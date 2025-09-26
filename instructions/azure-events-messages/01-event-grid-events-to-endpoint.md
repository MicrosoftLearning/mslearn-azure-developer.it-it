---
lab:
  topic: Azure events and messaging
  title: Instradare gli eventi a un endpoint personalizzato con Griglia di eventi di Azure
  description: Informazioni su come usare Griglia di eventi di Azure per instradare gli eventi a un endpoint personalizzato.
---

# Instradare gli eventi a un endpoint personalizzato con Griglia di eventi di Azure

In questo esercizio si crea un argomento di Griglia di eventi di Azure e un endpoint dell'app Web, quindi si compila un'applicazione console .NET che invia gli eventi personalizzati all'argomento di Griglia di eventi. Si apprenderà come configurare le sottoscrizioni di eventi, eseguire l'autenticazione con Griglia di eventi e verificare che gli eventi vengano instradati correttamente all'endpoint visualizzandoli nell'app Web.

Attività eseguite in questo esercizio:

* Creare risorse di Griglia di eventi di Azure
* Abilitare il provider di risorse di Griglia di eventi
* Creare un argomento in Griglia di eventi
* Creare un endpoint del messaggio
* Sottoscrivere l'argomento
* Inviare un evento con un'app console .NET
* Pulire le risorse

Questo esercizio richiede circa **30** minuti.

## Creare risorse di Griglia di eventi di Azure

In questa sezione dell'esercizio si creano le risorse necessarie in Azure con l'interfaccia della riga di comando di Azure.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

1. Creare un gruppo di risorse con le risorse necessarie per questo esercizio. Se si ha già un gruppo di risorse da usare, procedere con il passaggio successivo. Sostituire **myResourceGroup** con un nome da usare per il gruppo di risorse. Se necessario, è possibile sostituire **eastus** con un'area nelle vicinanze.

    ```bash
    az group create --name myResourceGroup --location eastus
    ```

1. Molti comandi richiedono nomi univoci e usano gli stessi parametri. La creazione di alcune variabili ridurrà le modifiche necessarie per i comandi che creano le risorse. Eseguire i comandi seguenti per creare le variabili necessarie. Sostituire **myResourceGroup** con il nome usato per questo esercizio. Se è stata modificata la posizione nel passaggio precedente, apportare la stessa modifica nella variabile **location**.

    ```bash
    let rNum=$RANDOM
    resourceGroup=myResourceGroup
    location=eastus
    topicName="mytopic-evgtopic-${rNum}"
    siteName="evgsite-${rNum}"
    siteURL="https://${siteName}.azurewebsites.net"
    ```

### Abilitare il provider di risorse di Griglia di eventi

Un provider di risorse di Azure è un servizio che definisce e gestisce tipi specifici di risorse in Azure. È ciò che Azure usa in background quando si distribuiscono o si gestiscono le risorse. Registrare il provider di risorse di Griglia di eventi con il comando **az provider register**. 

```bash
az provider register --namespace Microsoft.EventGrid
```

Per completare la registrazione possono essere necessari alcuni minuti. È possibile controllare lo stato con il comando seguente.

```bash
az provider show --namespace Microsoft.EventGrid --query "registrationState"
```

> **Nota:** Questo passaggio è necessario solo per le sottoscrizioni che non hanno usato in precedenza Griglia di eventi.

### Creare un argomento in Griglia di eventi

Creare un argomento usando il comando **az eventgrid topic create**. Il nome deve essere univoco perché fa parte del DNS.  

```bash
az eventgrid topic create --name $topicName \
    --location $location \
    --resource-group $resourceGroup
```

### Creare un endpoint del messaggio

Prima di sottoscrivere l'argomento personalizzato, è necessario creare l'endpoint per il messaggio dell'evento. L'endpoint richiede in genere azioni basate sui dati degli eventi. Lo script seguente usa un'app Web predefinita che visualizza i messaggi di evento. La soluzione distribuita include un piano di servizio app, un'app Web del servizio app e codice sorgente da GitHub.

1. Eseguire i comandi seguenti per creare un endpoint del messaggio. Il comando **echo** visualizzerà l'URL del sito per l'endpoint.

    ```bash
    az deployment group create \
        --resource-group $resourceGroup \
        --template-uri "https://raw.githubusercontent.com/Azure-Samples/azure-event-grid-viewer/main/azuredeploy.json" \
        --parameters siteName=$siteName hostingPlanName=viewerhost
    
    echo "Your web app URL: ${siteURL}"
    ```

    > **Nota:** Il completamento del comando può richiedere alcuni minuti.

1. Aprire una nuova scheda nel browser e passare all'URL generato alla fine dello script precedente per assicurarsi che l'app Web sia in esecuzione. Il sito dovrebbe essere visibile senza messaggi attualmente visualizzati.

    > **Suggerimento:** Lasciare in esecuzione il browser, usato per visualizzare gli aggiornamenti.

### Sottoscrivere l'argomento

È possibile sottoscrivere un argomento di Griglia di eventi per indicare a Griglia di eventi gli eventi di cui si vuole tenere traccia e dove inviare tali eventi. 

1. Sottoscrivere un argomento usando il comando **az eventgrid event-subscription create**. Lo script seguente recupera l'ID sottoscrizione dall'account e lo usa nella creazione della sottoscrizione di eventi.

    ```bash
    endpoint="${siteURL}/api/updates"
    topicId=$(az eventgrid topic show --resource-group $resourceGroup \
        --name $topicName --query "id" --output tsv)
    
    az eventgrid event-subscription create \
        --source-resource-id $topicId \
        --name TopicSubscription \
        --endpoint $endpoint
    ```

1. Visualizzare nuovamente l'app Web e notare che all'app è stato inviato un evento di convalida della sottoscrizione. Selezionare l'icona a forma di occhio per espandere i dati dell'evento. Griglia di eventi invia l'evento di convalida in modo che l'endpoint possa verificare che voglia ricevere i dati dell'evento. L'app Web include il codice per la convalida della sottoscrizione.

## Inviare un evento con un'applicazione console .NET

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Eseguire i comandi seguenti per creare una directory che conterrà il progetto e passare alla directory del progetto.

    ```bash
    mkdir eventgrid
    cd eventgrid
    ```

1. Creare l'applicazione console .NET.

    ```bash
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Azure.Messaging.EventGrid** e **dotenv.net** al progetto.

    ```bash
    dotnet add package Azure.Messaging.EventGrid
    dotnet add package dotenv.net
    ```

### Configurare l'applicazione console

In questa sezione si recupera l'endpoint dell'argomento e la chiave di accesso in modo che possano essere aggiunti a un file con estensione **env** per contenere tali segreti.

1. Eseguire i comandi seguenti per recuperare l'URL e la chiave di accesso per l'argomento creato in precedenza. Assicurarsi di registrare questi valori.

    ```bash
    az eventgrid topic show --name $topicName -g $resourceGroup --query "endpoint" --output tsv
    az eventgrid topic key list --name $topicName -g $resourceGroup --query "key1" --output tsv
    ```

1. Eseguire il comando seguente per creare il file con estensione **env** che conterrà i segreti e quindi aprirlo nell'editor di codice.

    ```bash
    touch .env
    code .env
    ```

1. Aggiungere il codice seguente al file con estensione **env**. Sostituire **YOUR_TOPIC_ENDPOINT** e **YOUR_TOPIC_ACCESS_KEY** con i valori registrati in precedenza.

    ```
    TOPIC_ENDPOINT="YOUR_TOPIC_ENDPOINT"
    TOPIC_ACCESS_KEY="YOUR_TOPIC_ACCESS_KEY"
    ```

1. Premere **CTRL+S** per salvare il file e quindi **CTRL+Q** per uscire dall'editor.

È ora possibile sostituire il codice del modello nel file **Program.cs** usando l'editor in Cloud Shell.

### Aggiungere il codice per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```bash
    code Program.cs
    ```

1. Sostituire il codice esistente con il codice seguente. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    using dotenv.net; 
    using Azure.Messaging.EventGrid; 
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Start the asynchronous process to send an Event Grid event
    ProcessAsync().GetAwaiter().GetResult();
    
    async Task ProcessAsync()
    {
        // Retrieve Event Grid topic endpoint and access key from environment variables
        var topicEndpoint = envVars["TOPIC_ENDPOINT"];
        var topicKey = envVars["TOPIC_ACCESS_KEY"];
        
        // Check if the required environment variables are set
        if (string.IsNullOrEmpty(topicEndpoint) || string.IsNullOrEmpty(topicKey))
        {
            Console.WriteLine("Please set TOPIC_ENDPOINT and TOPIC_ACCESS_KEY in your .env file.");
            return;
        }
    
        // Create an EventGridPublisherClient to send events to the specified topic
        EventGridPublisherClient client = new EventGridPublisherClient
            (new Uri(topicEndpoint),
            new Azure.AzureKeyCredential(topicKey));
    
        // Create a new EventGridEvent with sample data
        var eventGridEvent = new EventGridEvent(
            subject: "ExampleSubject",
            eventType: "ExampleEventType",
            dataVersion: "1.0",
            data: new { Message = "Hello, Event Grid!" }
        );
    
        // Send the event to Azure Event Grid
        await client.SendEventAsync(eventGridEvent);
        Console.WriteLine("Event sent successfully.");
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

1. Eseguire il comando seguente in Cloud Shell per avviare l'applicazione console. Verrà visualizzato il messaggio **Evento inviato correttamente.** quando il messaggio viene inviato.

    ```bash
    dotnet run
    ```

1. Visualizzare l'app Web per vedere l'evento appena inviato. Selezionare l'icona a forma di occhio per espandere i dati dell'evento.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.