---
lab:
  topic: Azure events and messaging
  title: Inviare e ricevere messaggi dal bus di servizio di Azure
  description: Informazioni su come inviare e ricevere messaggi dal bus di servizio di Azure con l'SDK .NET Azure.Messaging.ServiceBus.
---

# Inviare e ricevere messaggi dal bus di servizio di Azure

In questo esercizio si creano e si configurano le risorse del bus di servizio di Azure, quindi si crea un'app .NET per inviare e ricevere messaggi usando l'SDK **Azure.Messaging.ServiceBus**. Si apprenderà come effettuare il provisioning di uno spazio dei nomi e di una coda del bus di servizio, assegnare autorizzazioni e interagire con i messaggi a livello di codice. 

Attività eseguite in questo esercizio:

* Creare risorse del bus di servizio di Azure
* Assegnare un ruolo al nome utente di Microsoft Entra
* Creare un'app console .NET per inviare e ricevere messaggi
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
    namespaceName=svcbusns$RANDOM
    ```

1. Sarà necessario il nome assegnato allo spazio dei nomi più avanti in questo esercizio. Eseguire il comando seguente e osservare l'output.

    ```
    echo $namespaceName
    ```

### Creare uno spazio dei nomi e una coda del bus di servizio di Azure

1. Creare uno spazio dei nomi di messaggistica del bus di servizio. Il comando seguente creerà uno spazio dei nomi usando la variabile creata in precedenza. Il completamento dell'operazione richiede alcuni minuti.

    ```bash
    az servicebus namespace create \
        --resource-group $resourceGroup \
        --name $namespaceName \
        --location $location
    ```

1. Dopo aver creato uno spazio dei nomi, è necessario creare una coda per contenere i messaggi. Eseguire il comando seguente per creare una coda denominata **myqueue**.

    ```bash
    az servicebus queue create --resource-group $resourceGroup \
        --namespace-name $namespaceName \
        --name myqueue
    ```

### Assegnare un ruolo al nome utente di Microsoft Entra

Per consentire all'app di inviare e ricevere messaggi, assegnare all'utente Microsoft Entra il ruolo **Proprietario dei dati del bus di servizio di Azure** a livello di spazio dei nomi del bus di servizio. In questo modo si concede all'account utente l'autorizzazione per gestire e accedere a code e argomenti usando il controllo degli accessi in base al ruolo di Azure. Eseguire i passaggi seguenti in Cloud Shell.

1. Eseguire il comando seguente per recuperare **userPrincipalName** dall'account. Rappresenta l'utente a cui verrà assegnato il ruolo.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Eseguire il comando seguente per recuperare l'ID della risorsa dello spazio dei nomi del bus di servizio. L'ID della risorsa imposta l'ambito per l'assegnazione di ruolo su uno spazio dei nomi specifico.

    ```
    resourceID=$(az servicebus namespace show --name $namespaceName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. Eseguire il comando seguente per creare e assegnare il ruolo **Proprietario dei dati del bus di servizio di Azure**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Azure Service Bus Data Owner" \
        --scope $resourceID
    ```

## Creare un'app console .NET per inviare e ricevere messaggi

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Eseguire i comandi seguenti per creare una directory che conterrà il progetto e passare alla directory del progetto.

    ```
    mkdir svcbus
    cd svcbus
    ```

1. Creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Azure.Messaging.ServiceBus** e **Azure.Identity** al progetto.

    ```
    dotnet add package Azure.Messaging.ServiceBus
    dotnet add package Azure.Identity
    ```

### Aggiungere il codice di avvio per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```
    code Program.cs
    ```

1. Sostituire i contenuti esistenti con il codice seguente. Assicurarsi di esaminare i commenti nel codice e sostituire **<YOUR-NAMESPACE>** con lo spazio dei nomi del bus di servizio registrato in precedenza.

    ```csharp
    using Azure.Messaging.ServiceBus;
    using Azure.Identity;
    using System.Timers;
    
    
    // TODO: Replace <YOUR-NAMESPACE> with your Service Bus namespace
    string svcbusNameSpace = "<YOUR-NAMESPACE>.servicebus.windows.net";
    string queueName = "myQueue";
    
    
    // ADD CODE TO CREATE A SERVICE BUS CLIENT
    
    
    
    // ADD CODE TO SEND MESSAGES TO THE QUEUE
    
    
    
    // ADD CODE TO PROCESS MESSAGES FROM THE QUEUE
    
    
    
    // Dispose client after use
    await client.DisposeAsync();
    ```

1. Premi **CTRL+S** per salvare le modifiche.

### Aggiungere il codice per inviare messaggi alla coda

È ora possibile aggiungere codice per creare il client del bus di servizio e inviare un batch di messaggi alla coda.

1. Individuare il commento **// ADD CODE TO CREATE A SERVICE BUS CLIENT** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a Service Bus client using the namespace and DefaultAzureCredential
    // The DefaultAzureCredential will use the Azure CLI credentials, so ensure you are logged in
    ServiceBusClient client = new(svcbusNameSpace, new DefaultAzureCredential(options));
    ```

1. Individuare il commento **// ADD CODE TO SEND MESSAGES TO THE QUEUE** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Create a sender for the specified queue
    ServiceBusSender sender = client.CreateSender(queueName);
    
    // create a batch 
    using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();
    
    // number of messages to be sent to the queue
    const int numOfMessages = 3;
    
    for (int i = 1; i <= numOfMessages; i++)
    {
        // try adding a message to the batch
        if (!messageBatch.TryAddMessage(new ServiceBusMessage($"Message {i}")))
        {
            // if it is too large for the batch
            throw new Exception($"The message {i} is too large to fit in the batch.");
        }
    }
    
    try
    {
        // Use the producer client to send the batch of messages to the Service Bus queue
        await sender.SendMessagesAsync(messageBatch);
        Console.WriteLine($"A batch of {numOfMessages} messages has been published to the queue.");
    }
    finally
    {
        // Calling DisposeAsync on client types is required to ensure that network
        // resources and other unmanaged objects are properly cleaned up.
        await sender.DisposeAsync();
    }
    
    Console.WriteLine("Press any key to continue");
    Console.ReadKey();
    ```

1. Premere **CTRL+s** per salvare il file, quindi continuare con l'esercizio.

### Aggiungere codice per elaborare i messaggi nella coda

1. Individuare il commento **// ADD CODE TO PROCESS MESSAGES FROM THE QUEUE** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Create a processor that we can use to process the messages in the queue
    ServiceBusProcessor processor = client.CreateProcessor(queueName, new ServiceBusProcessorOptions());
    
    // Idle timeout in milliseconds, the idle timer will stop the processor if there are no more 
    // messages in the queue to process
    const int idleTimeoutMs = 3000;
    System.Timers.Timer idleTimer = new(idleTimeoutMs);
    idleTimer.Elapsed += async (s, e) =>
    {
        Console.WriteLine($"No messages received for {idleTimeoutMs / 1000} seconds. Stopping processor...");
        await processor.StopProcessingAsync();
    };
    
    try
    {
        // add handler to process messages
        processor.ProcessMessageAsync += MessageHandler;
    
        // add handler to process any errors
        processor.ProcessErrorAsync += ErrorHandler;
    
        // start processing 
        idleTimer.Start();
        await processor.StartProcessingAsync();
    
        Console.WriteLine($"Processor started. Will stop after {idleTimeoutMs / 1000} seconds of inactivity.");
        // Wait for the processor to stop
        while (processor.IsProcessing)
        {
            await Task.Delay(500);
        }
        idleTimer.Stop();
        Console.WriteLine("Stopped receiving messages");
    }
    finally
    {
        // Dispose processor after use
        await processor.DisposeAsync();
    }
    
    // handle received messages
    async Task MessageHandler(ProcessMessageEventArgs args)
    {
        string body = args.Message.Body.ToString();
        Console.WriteLine($"Received: {body}");
    
        // Reset the idle timer on each message
        idleTimer.Stop();
        idleTimer.Start();
    
        // complete the message. message is deleted from the queue. 
        await args.CompleteMessageAsync(args.Message);
    }
    
    // handle any errors when receiving messages
    Task ErrorHandler(ProcessErrorEventArgs args)
    {
        Console.WriteLine(args.Exception.ToString());
        return Task.CompletedTask;
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

1. Eseguire il comando seguente per avviare l'app console. L'app si fermerà in diverse fasi e chiederà di premere un tasto per continuare. In questo modo è possibile visualizzare i messaggi nel portale di Azure.

    ```
    dotnet run
    ```

    

1. Nel portale di Azure passare allo spazio dei nomi del bus di servizio creato. 

1. Selezionare **myqueue** nella parte inferiore della finestra **Panoramica**.

1. Selezionare **Service Bus Explorer** nel riquadro di spostamento sinistro.

1. Selezionare **Visualizza in anteprima dall'inizio** e i tre messaggi verranno visualizzati dopo alcuni secondi.

1. In Cloud Shell premere qualsiasi tasto per continuare e l'applicazione elaborerà i tre messaggi. 
 
1. Tornare al portale dopo che l'applicazione ha completato l'elaborazione dei messaggi. Selezionare **Visualizza in anteprima dall'inizio** e notare che nella coda non sono presenti messaggi.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.

