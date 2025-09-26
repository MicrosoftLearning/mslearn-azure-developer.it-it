---
lab:
  topic: Azure events and messaging
  title: Inviare e recuperare eventi da Hub eventi di Azure
  description: Informazioni su come inviare e recuperare eventi da Hub eventi di Azure con l'SDK .NET Azure.Messaging.EventHubs.
---

# Inviare e recuperare eventi da Hub eventi di Azure

In questo esercizio si creano risorse di Hub eventi di Azure e si crea un'app console .NET per inviare e ricevere eventi usando l'SDK **Azure.Messaging.EventHubs**. Si apprenderà come effettuare il provisioning delle risorse cloud, interagire con Hub eventi e pulire l'ambiente al termine delle attività.

Attività eseguite in questo esercizio:

* Creare un gruppo di risorse
* Creare risorse di Hub eventi di Azure
* Creare un'app console .NET per inviare e recuperare eventi
* Pulire le risorse

Questo esercizio richiede circa **30** minuti.

## Creare risorse di Hub eventi di Azure

In questa sezione dell'esercizio si creano le risorse necessarie in Azure con l'interfaccia della riga di comando di Azure.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

1. Creare un gruppo di risorse con le risorse necessarie per questo esercizio. Se si ha già un gruppo di risorse da usare, procedere con il passaggio successivo. Sostituire **myResourceGroup** con un nome da usare per il gruppo di risorse. Se necessario, è possibile sostituire **eastus** con un'area nelle vicinanze.

    ```
    az group create --name myResourceGroup --location eastus
    ```

1. Molti comandi richiedono nomi univoci e usano gli stessi parametri. La creazione di alcune variabili ridurrà le modifiche necessarie per i comandi che creano le risorse. Eseguire i comandi seguenti per creare le variabili necessarie. Sostituire **myResourceGroup** con il nome usato per questo esercizio. Se è stata modificata la posizione nel passaggio precedente, apportare la stessa modifica nella variabile **location**.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    namespaceName=eventhubsns$RANDOM
    ```

### Creare uno spazio dei nomi e un hub eventi di Hub eventi di Azure

Uno spazio dei nomi di Hub eventi di Azure è un contenitore logico per le risorse dell'hub eventi di Azure. Fornisce un contenitore di ambito univoco in cui è possibile creare uno o più hub eventi, usati per inserire, elaborare e archiviare grandi volumi di dati degli eventi. Le istruzioni seguenti vengono eseguite in Cloud Shell. 

1. Eseguire il comando seguente per creare uno spazio dei nomi di Hub eventi.

    ```
    az eventhubs namespace create --name $namespaceName --resource-group $resourceGroup -l $location
    ```

1. Eseguire il comando seguente per creare un hub eventi denominato **myEventHub** nello spazio dei nomi di Hub eventi. 

    ```
    az eventhubs eventhub create --name myEventHub --resource-group $resourceGroup \
      --namespace-name $namespaceName
    ```

### Assegnare un ruolo al nome utente di Microsoft Entra

Per consentire all'app di inviare e ricevere messaggi, assegnare all'utente Microsoft Entra il ruolo **Proprietario dei dati di Hub eventi di Azure** a livello di spazio dei nomi di Hub eventi. In questo modo si concede all'account utente l'autorizzazione per gestire e accedere a code e argomenti usando il controllo degli accessi in base al ruolo di Azure. Eseguire i passaggi seguenti in Cloud Shell.

1. Eseguire il comando seguente per recuperare **userPrincipalName** dall'account. Rappresenta l'utente a cui verrà assegnato il ruolo.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Eseguire il comando seguente per recuperare l'ID della risorsa dello spazio dei nomi di Hub eventi. L'ID della risorsa imposta l'ambito per l'assegnazione di ruolo su uno spazio dei nomi specifico.

    ```
    resourceID=$(az eventhubs namespace show --resource-group $resourceGroup \
        --name $namespaceName --query id --output tsv)
    ```
1. Eseguire il comando seguente per creare e assegnare il ruolo **Proprietario dei dati di Hub eventi di Azure**, che consente di inviare e recuperare eventi.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Azure Event Hubs Data Owner" \
        --scope $resourceID
    ```

## Inviare e recuperare eventi con un'applicazione console .NET

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Eseguire i comandi seguenti per creare una directory che conterrà il progetto e passare alla directory del progetto.

    ```
    mkdir eventhubs
    cd eventhubs
    ```

1. Creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Azure.Messaging.EventHubs** e **Azure.Identity** al progetto.

    ```
    dotnet add package Azure.Messaging.EventHubs
    dotnet add package Azure.Identity
    ```

A questo punto è il momento di sostituire il codice del modello nel file **Program.cs** usando l'editor in Cloud Shell.

### Aggiungere il codice di avvio per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```
    code Program.cs
    ```

1. Sostituire i contenuti esistenti con il codice seguente. Assicurarsi di esaminare i commenti nel codice e sostituire **YOUR_EVENT_HUB_NAMESPACE** con lo spazio dei nomi dell'hub eventi.

    ```csharp
    using Azure.Messaging.EventHubs;
    using Azure.Messaging.EventHubs.Producer;
    using Azure.Messaging.EventHubs.Consumer;
    using Azure.Identity;
    using System.Text;
    
    // TO-DO: Replace YOUR_EVENT_HUB_NAMESPACE with your actual Event Hub namespace
    string namespaceURL = "YOUR_EVENT_HUB_NAMESPACE.servicebus.windows.net";
    string eventHubName = "myEventHub"; 
    
    // Create a DefaultAzureCredentialOptions object to exclude certain credentials
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Number of events to be sent to the event hub
    int numOfEvents = 3;
    
    // CREATE A PRODUCER CLIENT AND SEND EVENTS
    
    
    
    // CREATE A CONSUMER CLIENT AND RECEIVE EVENTS
    
    
    ```

1. Premi **CTRL+S** per salvare le modifiche.

### Aggiungere codice per completare l'applicazione

In questa sezione si aggiunge codice per creare i client producer e consumer per inviare e ricevere eventi.

1. Individuare il commento **// CREATE A PRODUCER CLIENT AND SEND EVENTS** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    // Create a producer client to send events to the event hub
    EventHubProducerClient producerClient = new EventHubProducerClient(
        namespaceURL,
        eventHubName,
        new DefaultAzureCredential(options));
    
    // Create a batch of events 
    using EventDataBatch eventBatch = await producerClient.CreateBatchAsync();
    
    
    // Adding a random number to the event body and sending the events. 
    var random = new Random();
    for (int i = 1; i <= numOfEvents; i++)
    {
        int randomNumber = random.Next(1, 101); // 1 to 100 inclusive
        string eventBody = $"Event {randomNumber}";
        if (!eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes(eventBody))))
        {
            // if it is too large for the batch
            throw new Exception($"Event {i} is too large for the batch and cannot be sent.");
        }
    }
    
    try
    {
        // Use the producer client to send the batch of events to the event hub
        await producerClient.SendAsync(eventBatch);
    
        Console.WriteLine($"A batch of {numOfEvents} events has been published.");
        Console.WriteLine("Press Enter to retrieve and print the events...");
        Console.ReadLine();
    }
    finally
    {
        await producerClient.DisposeAsync();
    }
    ```

1. Premi **CTRL+S** per salvare le modifiche.

1. Individuare il commento **// CREATE A CONSUMER CLIENT AND RETRIEVE EVENTS** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    // Create an EventHubConsumerClient
    await using var consumerClient = new EventHubConsumerClient(
        EventHubConsumerClient.DefaultConsumerGroupName,
        namespaceURL,
        eventHubName,
        new DefaultAzureCredential(options));
    
    Console.Clear();
    Console.WriteLine("Retrieving all events from the hub...");
    
    // Get total number of events in the hub by summing (last - first + 1) for all partitions
    // This count is used to determine when to stop reading events
    long totalEventCount = 0;
    string[] partitionIds = await consumerClient.GetPartitionIdsAsync();
    foreach (var partitionId in partitionIds)
    {
        PartitionProperties properties = await consumerClient.GetPartitionPropertiesAsync(partitionId);
        if (!properties.IsEmpty && properties.LastEnqueuedSequenceNumber >= properties.BeginningSequenceNumber)
        {
            totalEventCount += (properties.LastEnqueuedSequenceNumber - properties.BeginningSequenceNumber + 1);
        }
    }
    
    // Start retrieving events from the event hub and print to the console
    int retrievedCount = 0;
    await foreach (PartitionEvent partitionEvent in consumerClient.ReadEventsAsync(startReadingAtEarliestEvent: true))
    {
        if (partitionEvent.Data != null)
        {
            string body = Encoding.UTF8.GetString(partitionEvent.Data.Body.ToArray());
            Console.WriteLine($"Retrieved event: {body}");
            retrievedCount++;
            if (retrievedCount >= totalEventCount)
            {
                Console.WriteLine("Done retrieving events. Press Enter to exit...");
                Console.ReadLine();
                return;
            }
        }
    }
    ```

1. Premere **CTRL+S** per salvare il file e quindi **CTRL+Q** per uscire dall'editor.

## Accedere in Azure ed eseguire l'app

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
    az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, vedere [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).

1. Avviare l'applicazione eseguendo il comando seguente:

    ```
    dotnet run
    ```

    Dopo alcuni secondi verrà visualizzato un output simile all'esempio seguente:
    
    ```
    A batch of 3 events has been published.
    Press Enter to retrieve and print the events...
    
    Retrieving all events from the hub...
    Retrieved event: Event 4
    Retrieved event: Event 96
    Retrieved event: Event 74
    Done retrieving events. Press Enter to exit...
    ```

L'applicazione invia sempre tre eventi all'hub, ma recupera tutti gli eventi nell'hub. Se si esegue l'applicazione più volte viene recuperato un numero crescente di eventi. I numeri casuali usati per la creazione di eventi consentono di identificare eventi diversi.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio. 
