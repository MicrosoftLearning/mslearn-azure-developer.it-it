---
lab:
  topic: Azure events and messaging
  title: Inviare e ricevere messaggi dall'archiviazione code di Azure
  description: Informazioni su come inviare e ricevere messaggi dall'archiviazione code di Azure con l'SDK .NET Azure.StorageQueues.
---

# Inviare e ricevere messaggi dall'archiviazione code di Azure

In questo esercizio si creano e si configurano le risorse di Archiviazione code di Azure, quindi si crea un'app .NET per inviare e ricevere messaggi usando l'SDK **Azure.Storage.Queues**. Si apprenderà come effettuare il provisioning delle risorse di archiviazione, gestire i messaggi della coda e pulire l'ambiente al termine delle attività. 

Attività eseguite in questo esercizio:

* Creare risorse di archiviazione code di Azure
* Assegnare un ruolo al nome utente di Microsoft Entra
* Creare un'app console .NET per inviare e ricevere messaggi
* Pulire le risorse

Questo esercizio richiede circa **30** minuti.

## Creare risorse di archiviazione code di Azure

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
    storAcctName=storactname$RANDOM
    ```

1. Sarà necessario il nome assegnato all'account di archiviazione più avanti in questo esercizio. Eseguire il comando seguente e osservare l'output.

    ```
    echo $storAcctName
    ```

1. Eseguire il comando seguente per creare un account di archiviazione usando la variabile creata in precedenza. Il completamento dell'operazione richiede alcuni minuti.

    ```bash
    az storage account create --resource-group $resourceGroup \
        --name $storAcctName --location $location --sku Standard_LRS
    ```

### Assegnare un ruolo al nome utente di Microsoft Entra

Per consentire all'app di inviare e ricevere messaggi, assegnare all'utente Microsoft Entra il ruolo **Collaboratore ai dati della coda di archiviazione**. In questo modo si concede all'account utente l'autorizzazione per creare code e inviare/ricevere messaggi usando il controllo degli accessi in base al ruolo di Azure. Eseguire i passaggi seguenti in Cloud Shell.

1. Eseguire il comando seguente per recuperare **userPrincipalName** dall'account. Rappresenta l'utente a cui verrà assegnato il ruolo.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Eseguire il comando seguente per recuperare l'ID della risorsa dell'account di archiviazione. L'ID della risorsa imposta l'ambito per l'assegnazione di ruolo su uno spazio dei nomi specifico.

    ```
    resourceID=$(az storage account show --resource-group $resourceGroup \
        --name $storAcctName --query id --output tsv)
    ```

1. Eseguire il comando seguente per creare e assegnare il ruolo **Collaboratore ai dati della coda di archiviazione**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Queue Data Contributor" \
        --scope $resourceID
    ```

## Creare un'app console .NET per inviare e ricevere messaggi

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Eseguire i comandi seguenti per creare una directory che conterrà il progetto e passare alla directory del progetto.

    ```
    mkdir queuestor
    cd queuestor
    ```

1. Creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Azure.Storage.Queues** e **Azure.Identity** al progetto.

    ```
    dotnet add package Azure.Storage.Queues
    dotnet add package Azure.Identity
    ```

### Aggiungere il codice di avvio per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```
    code Program.cs
    ```

1. Sostituire i contenuti esistenti con il codice seguente. Assicurarsi di esaminare i commenti nel codice e sostituire **<YOUR-STORAGE-ACCT-NAME>** con il nome dell'account di archiviazione registrato in precedenza.

    ```csharp
    using Azure;
    using Azure.Identity;
    using Azure.Storage.Queues;
    using Azure.Storage.Queues.Models;
    using System;
    using System.Threading.Tasks;
    
    // Create a unique name for the queue
    // TODO: Replace the <YOUR-STORAGE-ACCT-NAME> placeholder 
    string queueName = "myqueue-" + Guid.NewGuid().ToString();
    string storageAccountName = "<YOUR-STORAGE-ACCT-NAME>";
    
    // ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE
    
    
    
    // ADD CODE TO SEND AND LIST MESSAGES
    
    
    
    // ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES
    
    
    
    // ADD CODE TO DELETE MESSAGES AND THE QUEUE
    
    
    ```

1. Premi **CTRL+S** per salvare le modifiche.

### Aggiungere il codice per creare un client della coda e creare una coda

È ora possibile aggiungere il codice per creare il client di archiviazione code e creare una coda.

1. Individuare il commento **// ADD CODE TO CREATE A QUEUE CLIENT AND CREATE A QUEUE** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Create a DefaultAzureCredentialOptions object to exclude certain credentials
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Instantiate a QueueClient to create and interact with the queue
    QueueClient queueClient = new QueueClient(
        new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"),
        new DefaultAzureCredential(options));
    
    Console.WriteLine($"Creating queue: {queueName}");
    
    // Create the queue
    await queueClient.CreateAsync();
    
    Console.WriteLine("Queue created, press Enter to add messages to the queue...");
    Console.ReadLine();
    ```

1. Premere **CTRL+s** per salvare il file, quindi continuare con l'esercizio.

### Aggiungere il codice per inviare ed elencare messaggi in una coda

1. Individuare il commento **// ADD CODE TO SEND AND LIST MESSAGES** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Send several messages to the queue with the SendMessageAsync method.
    await queueClient.SendMessageAsync("Message 1");
    await queueClient.SendMessageAsync("Message 2");
    
    // Send a message and save the receipt for later use
    SendReceipt receipt = await queueClient.SendMessageAsync("Message 3");
    
    Console.WriteLine("Messages added to the queue. Press Enter to peek at the messages...");
    Console.ReadLine();
    
    // Peeking messages lets you view the messages without removing them from the queue.
    
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to update a message in the queue...");
    Console.ReadLine();
    ```

1. Premere **CTRL+s** per salvare il file, quindi continuare con l'esercizio.

### Aggiungere il codice per aggiornare un messaggio ed elencare i risultati

1. Individuare il commento **// ADD CODE TO UPDATE A MESSAGE AND LIST MESSAGES** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Update a message with the UpdateMessageAsync method and the saved receipt
    await queueClient.UpdateMessageAsync(receipt.MessageId, receipt.PopReceipt, "Message 3 has been updated");
    
    Console.WriteLine("Message three updated. Press Enter to peek at the messages again...");
    Console.ReadLine();
    
    
    // Peek messages from the queue to compare updated content
    foreach (var message in (await queueClient.PeekMessagesAsync(maxMessages: 10)).Value)
    {
        Console.WriteLine($"Message: {message.MessageText}");
    }
    
    Console.WriteLine("\nPress Enter to delete messages from the queue...");
    Console.ReadLine();
    ```

1. Premere **CTRL+s** per salvare il file, quindi continuare con l'esercizio.

### Aggiungere il codice per inviare messaggi alla coda

1. Individuare il commento **// ADD CODE TO DELETE MESSAGES AND THE QUEUE** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Delete messages from the queue with the DeleteMessagesAsync method.
    foreach (var message in (await queueClient.ReceiveMessagesAsync(maxMessages: 10)).Value)
    {
        // "Process" the message
        Console.WriteLine($"Deleting message: {message.MessageText}");
    
        // Let the service know we're finished with the message and it can be safely deleted.
        await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
    }
    Console.WriteLine("Messages deleted from the queue.");
    Console.WriteLine("\nPress Enter key to delete the queue...");
    Console.ReadLine();
    
    // Delete the queue with the DeleteAsync method.
    Console.WriteLine($"Deleting queue: {queueClient.Name}");
    await queueClient.DeleteAsync();
    
    Console.WriteLine("Done");
    ```

1. Premere **CTRL+S** per salvare il file e quindi **CTRL+Q** per uscire dall'editor.

## Accedere in Azure ed eseguire l'app

1. Nel riquadro della riga di comando di Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
    az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, vedere [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).

1. Eseguire il comando seguente per avviare l'app console. L'app verrà sospesa molte volte durante l'esecuzione in attesa che venga premuto un tasto per continuare. In questo modo è possibile visualizzare i messaggi nel portale di Azure.

    ```
    dotnet run
    ```

1. Nel portale di Azure passare all'account di Archiviazione di Azure creato. 

1. Espandere **> Archiviazione dati** nel riquadro di spostamento sinistro e selezionare **Code**.

1. Selezionare la coda creata dall'applicazione in modo da poter visualizzare i messaggi inviati e monitorare le operazioni dell'applicazione.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, tutte le risorse esistenti esterne all'ambito di questo 

