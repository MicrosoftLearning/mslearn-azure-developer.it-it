---
lab:
  topic: Azure Storage
  title: Creare risorse di Archiviazione BLOB con la libreria client .NET
  description: 'Informazioni su come usare la libreria client .NET di Archiviazione di Azure per creare contenitori, caricare ed elencare BLOB ed eliminare contenitori.'
---

# Creare risorse di Archiviazione BLOB con la libreria client .NET

In questo esercizio si crea un account di Archiviazione di Azure e si compila un'applicazione console .NET usando la libreria client di BLOB del servizio di archiviazione di Azure per creare contenitori, caricare file in Archiviazione BLOB, elencare BLOB e scaricare file. Si apprenderà come eseguire l'autenticazione con Azure, eseguire operazioni di Archiviazione BLOB a livello di codice e verificare i risultati nel portale di Azure.

Attività eseguite in questo esercizio:

* Preparare le risorse di Azure
* Creare un'app console per creare e scaricare i dati
* Eseguire l'app e verificare i risultati
* Pulire le risorse

Questo esercizio richiede circa **30** minuti.

## Creare un account di Archiviazione di Azure

In questa sezione dell'esercizio si creano le risorse necessarie in Azure con l'interfaccia della riga di comando di Azure.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

1. Creare un gruppo di risorse con le risorse necessarie per questo esercizio. Sostituire **myResourceGroup** con un nome da usare per il gruppo di risorse. Se necessario, è possibile sostituire **eastus2** con un'area nelle vicinanze. Se si ha già un gruppo di risorse da usare, procedere con il passaggio successivo.

    ```
    az group create --location eastus2 --name myResourceGroup
    ```

1. Molti comandi richiedono nomi univoci e usano gli stessi parametri. La creazione di alcune variabili ridurrà le modifiche necessarie per i comandi che creano le risorse. Eseguire i comandi seguenti per creare le variabili necessarie. Sostituire **myResourceGroup** con il nome usato per questo esercizio.

    ```
    resourceGroup=myResourceGroup
    location=eastus
    accountName=storageacct$RANDOM
    ```

1. Eseguire i comandi seguenti per creare l'account di Archiviazione di Azure. Ogni nome di account deve essere univoco. Il primo comando crea una variabile con un nome univoco per l'account di archiviazione. Registrare il nome dell'account dall'output del comando **echo**. 

    ```
    az storage account create --name $accountName \
        --resource-group $resourceGroup \
        --location $location \
        --sku Standard_LRS 
    
    echo $accountName
    ```

### Assegnare un ruolo al nome utente di Microsoft Entra

Per consentire all'app di creare risorse ed elementi, assegnare l'utente di Microsoft Entra al ruolo **Proprietario dei dati del BLOB di archiviazione**. Eseguire i passaggi seguenti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Eseguire il comando seguente per recuperare **userPrincipalName** dall'account. Rappresenta l'utente a cui verrà assegnato il ruolo.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Eseguire il comando seguente per recuperare l'ID della risorsa dell'account di archiviazione. L'ID della risorsa imposta l'ambito per l'assegnazione di ruolo su uno spazio dei nomi specifico.

    ```
    resourceID=$(az storage account show --name $accountName \
        --resource-group $resourceGroup \
        --query id --output tsv)
    ```
1. Eseguire il comando seguente per creare e assegnare il ruolo **Proprietario dei dati del BLOB di archiviazione**. Questo ruolo consente di gestire contenitori ed elementi.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Storage Blob Data Owner" \
        --scope $resourceID
    ```

## Creare un'app console .NET per creare contenitori ed elementi

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

1. Eseguire i comandi seguenti per creare una directory che conterrà il progetto e passare alla directory del progetto.

    ```
    mkdir azstor
    cd azstor
    ```

1. Creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti necessari nell'applicazione.

    ```
    dotnet add package Azure.Storage.Blobs
    dotnet add package Azure.Identity
    ```

1. Eseguire il comando seguente per creare una cartella **data** nel progetto. 

    ```
    mkdir data
    ```

Ora è il momento di aggiungere il codice per il progetto.

### Aggiungere il codice di avvio per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```
    code Program.cs
    ```

1. Sostituire i contenuti esistenti con il codice seguente. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    using Azure.Storage.Blobs;
    using Azure.Storage.Blobs.Models;
    using Azure.Identity;
    
    Console.WriteLine("Azure Blob Storage exercise\n");
    
    // Create a DefaultAzureCredentialOptions object to configure the DefaultAzureCredential
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Run the examples asynchronously, wait for the results before proceeding
    await ProcessAsync();
    
    Console.WriteLine("\nPress enter to exit the sample application.");
    Console.ReadLine();
    
    async Task ProcessAsync()
    {
        // CREATE A BLOB STORAGE CLIENT
        
    
    
        // CREATE A CONTAINER
        
    
    
        // CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE
        
    
    
        // UPLOAD THE FILE TO BLOB STORAGE
        
    
    
        // LIST BLOBS IN THE CONTAINER
        
    
    
        // DOWNLOAD THE BLOB TO A LOCAL FILE
        
    
    }
    ```

1. Premere **CTRL+S** per salvare le modifiche e continuare con il passaggio successivo.


## Aggiungere codice per completare il progetto

Nel resto dell'esercizio si aggiungerà codice nelle aree specificate per creare l'applicazione completa. 

1. Individuare il commento **// CREATE A BLOB STORAGE CLIENT**, quindi aggiungere il codice seguente direttamente sotto il commento. **BlobServiceClient** funge da punto di ingresso primario per la gestione di contenitori e BLOB in un account di archiviazione. Il client usa *DefaultAzureCredential* per l'autenticazione. Assicurarsi di sostituire **YOUR_ACCOUNT_NAME** con il nome registrato in precedenza.

    ```csharp
    // Create a credential using DefaultAzureCredential with configured options
    string accountName = "YOUR_ACCOUNT_NAME"; // Replace with your storage account name
    
    // Use the DefaultAzureCredential with the options configured at the top of the program
    DefaultAzureCredential credential = new DefaultAzureCredential(options);
    
    // Create the BlobServiceClient using the endpoint and DefaultAzureCredential
    string blobServiceEndpoint = $"https://{accountName}.blob.core.windows.net";
    BlobServiceClient blobServiceClient = new BlobServiceClient(new Uri(blobServiceEndpoint), credential);
    ```

1. Premere **CTRL+S** per salvare le modifiche e continuare con il passaggio successivo.

1. Individuare il commento **// CREATE A CONTAINER**, quindi aggiungere il codice seguente direttamente sotto il commento. La creazione di un contenitore include la creazione di un'istanza della classe **BlobServiceClient** e quindi la chiamata del metodo **CreateBlobContainerAsync** per creare il contenitore nell'account di archiviazione. Un valore GUID viene aggiunto al nome del contenitore per assicurarsi che sia univoco. Il metodo **CreateBlobContainerAsync** avrà esito negativo se il contenitore esiste già.

    ```csharp
    // Create a unique name for the container
    string containerName = "wtblob" + Guid.NewGuid().ToString();
    
    // Create the container and return a container client object
    Console.WriteLine("Creating container: " + containerName);
    BlobContainerClient containerClient = 
        await blobServiceClient.CreateBlobContainerAsync(containerName);
    
    // Check if the container was created successfully
    if (containerClient != null)
    {
        Console.WriteLine("Container created successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("Failed to create the container, exiting program.");
        return;
    }
    ```

1. Premere **CTRL+S** per salvare le modifiche e continuare con il passaggio successivo.

1. Trovare il commento **// CREATE A LOCAL FILE FOR UPLOAD TO BLOB STORAGE**, quindi aggiungere il codice seguente direttamente sotto il commento. Nella directory data verrà creato un file che verrà poi caricato nel contenitore.

    ```csharp
    // Create a local file in the ./data/ directory for uploading and downloading
    Console.WriteLine("Creating a local file for upload to Blob storage...");
    string localPath = "./data/";
    string fileName = "wtfile" + Guid.NewGuid().ToString() + ".txt";
    string localFilePath = Path.Combine(localPath, fileName);
    
    // Write text to the file
    await File.WriteAllTextAsync(localFilePath, "Hello, World!");
    Console.WriteLine("Local file created, press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Premere **CTRL+S** per salvare le modifiche e continuare con il passaggio successivo.

1. Individuare il commento **// UPLOAD THE FILE TO BLOB STORAGE**, quindi aggiungere il codice seguente direttamente sotto il commento. Il codice ottiene un riferimento a un oggetto **BlobClient** chiamando il metodo **GetBlobClient** sul contenitore creato nella sezione precedente. Carica quindi un file locale generato usando il metodo **UploadAsync**. Questo metodo crea il BLOB se non esiste o lo sovrascrive se esiste già.

    ```csharp
    // Get a reference to the blob and upload the file
    BlobClient blobClient = containerClient.GetBlobClient(fileName);
    
    Console.WriteLine("Uploading to Blob storage as blob:\n\t {0}", blobClient.Uri);
    
    // Open the file and upload its data
    using (FileStream uploadFileStream = File.OpenRead(localFilePath))
    {
        await blobClient.UploadAsync(uploadFileStream);
        uploadFileStream.Close();
    }
    
    // Verify if the file was uploaded successfully
    bool blobExists = await blobClient.ExistsAsync();
    if (blobExists)
    {
        Console.WriteLine("File uploaded successfully, press 'Enter' to continue.");
        Console.ReadLine();
    }
    else
    {
        Console.WriteLine("File upload failed, exiting program..");
        return;
    }
    ```

1. Premere **CTRL+S** per salvare le modifiche e continuare con il passaggio successivo.

1. Individuare il commento **// LIST BLOBS IN THE CONTAINER**, quindi aggiungere il codice seguente direttamente sotto il commento. Usare il metodo **GetBlobsAsync** per elencare i BLOB nel contenitore. In questo caso, al contenitore è stato aggiunto un solo BLOB, quindi l'operazione di elenco restituisce solo quel BLOB. 

    ```csharp
    Console.WriteLine("Listing blobs in container...");
    await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
    {
        Console.WriteLine("\t" + blobItem.Name);
    }
    
    Console.WriteLine("Press 'Enter' to continue.");
    Console.ReadLine();
    ```

1. Premere **CTRL+S** per salvare le modifiche e continuare con il passaggio successivo.

1. Individuare il commento **// DOWNLOAD THE BLOB TO A LOCAL FILE**, quindi aggiungere il codice seguente direttamente sotto il commento. Il codice usa il metodo **DownloadAsync** per scaricare il BLOB creato in precedenza nel file system locale. Il codice di esempio aggiunge il suffisso "DOWNLOADED" al nome del BLOB in modo che si possano vedere entrambi i file nel file system locale. 

    ```csharp
    // Adds the string "DOWNLOADED" before the .txt extension so it doesn't 
    // overwrite the original file
    
    string downloadFilePath = localFilePath.Replace(".txt", "DOWNLOADED.txt");
    
    Console.WriteLine("Downloading blob to: {0}", downloadFilePath);
    
    // Download the blob's contents and save it to a file
    BlobDownloadInfo download = await blobClient.DownloadAsync();
    
    using (FileStream downloadFileStream = File.OpenWrite(downloadFilePath))
    {
        await download.Content.CopyToAsync(downloadFileStream);
    }
    
    Console.WriteLine("Blob downloaded successfully to: {0}", downloadFilePath);
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

1. Espandere **> Archiviazione dati** nel riquadro di spostamento sinistro e selezionare **Contenitori**.

1. Selezionare il contenitore creato dall'applicazione. Sarà possibile visualizzare il BLOB caricato.

1. Eseguire i due comandi seguenti per passare alla directory **data** ed elencare i file caricati e scaricati.

    ```
    cd data
    ls
    ```

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, tutte le risorse esistenti esterne all'ambito di questo 

