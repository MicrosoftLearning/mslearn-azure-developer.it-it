---
lab:
  topic: Azure Cosmos DB
  title: Creare risorse in Azure Cosmos DB for NoSQL usando .NET
  description: Informazioni su come creare risorse di database e contenitori in Azure Cosmos DB con Microsoft .NET SDK v3.
---

# Creare risorse in Azure Cosmos DB for NoSQL usando .NET

In questo esercizio si crea un account di Azure Cosmos DB e si compila un'applicazione console .NET che usa Microsoft Azure Cosmos DB SDK per creare un database, un contenitore e un elemento di esempio. Si apprenderà come configurare l'autenticazione, eseguire operazioni di database a livello di codice e verificare i risultati nel portale di Azure.

Attività eseguite in questo esercizio:

* Creare un account Azure Cosmos DB
* Creare un'app console che crea un database, un contenitore e un elemento
* Eseguire l'app console e verificare i risultati

Questo esercizio richiede circa **30** minuti.

## Creare un account Azure Cosmos DB

In questa sezione dell'esercizio si crea un gruppo di risorse e un account di Azure Cosmos DB. Si registrano anche l'endpoint e la chiave di accesso per l'account.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

1. Creare un gruppo di risorse con le risorse necessarie per questo esercizio. Se si ha già un gruppo di risorse da usare, procedere con il passaggio successivo. Sostituire **myResourceGroup** con un nome da usare per il gruppo di risorse. Se necessario, è possibile sostituire **eastus** con un'area nelle vicinanze.

    ```
    az group create --location eastus --name myResourceGroup
    ```

1. Molti comandi richiedono nomi univoci e usano gli stessi parametri. La creazione di alcune variabili ridurrà le modifiche necessarie per i comandi che creano le risorse. Eseguire i comandi seguenti per creare le variabili necessarie. Sostituire **myResourceGroup** con il nome usato per questo esercizio.

    ```
    resourceGroup=myResourceGroup
    accountName=cosmosexercise$RANDOM
    ```

1. Eseguire i comandi seguenti per creare l'account di Azure Cosmos DB. Ogni nome di account deve essere univoco. 

    ```
    az cosmosdb create --name $accountName \
        --resource-group $resourceGroup
    ```

1.  Eseguire il comando seguente per recuperare **documentEndpoint** per l'account di Azure Cosmos DB. Registrare l'endpoint dai risultati del comando. Sarà necessario più avanti nell'esercizio.

    ```
    az cosmosdb show --name $accountName \
        --resource-group $resourceGroup \
        --query "documentEndpoint" --output tsv
    ```

1. Recuperare la chiave primaria per l'account con il comando riportato di seguito. Registrare la chiave primaria dai risultati del comando. Sarà necessaria più avanti nell'esercizio.

    ```
    az cosmosdb keys list --name $accountName \
        --resource-group $resourceGroup \
        --query "primaryMasterKey" --output tsv
    ```

## Creare risorse dati e un elemento con un'applicazione console .NET

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Creare una cartella per il progetto e passare alla cartella.

    ```bash
    mkdir cosmosdb
    cd cosmosdb
    ```

1. Creare un'app console .NET.

    ```bash
    dotnet new console
    ```

### Configurare l'applicazione console

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Microsoft.Azure.Cosmos**, **Newtonsoft.Json** e **dotenv.net** al progetto.

    ```bash
    dotnet add package Microsoft.Azure.Cosmos --version 3.*
    dotnet add package Newtonsoft.Json --version 13.*
    dotnet add package dotenv.net
    ```

1. Eseguire il comando seguente per creare il file con estensione **env** che conterrà i segreti e quindi aprirlo nell'editor di codice.

    ```bash
    touch .env
    code .env
    ```

1. Aggiungere il codice seguente al file con estensione **env**. Sostituire **YOUR_DOCUMENT_ENDPOINT** e **YOUR_ACCOUNT_KEY** con i valori registrati in precedenza.

    ```
    DOCUMENT_ENDPOINT="YOUR_DOCUMENT_ENDPOINT"
    ACCOUNT_KEY="YOUR_ACCOUNT_KEY"
    ```

1. Premere **CTRL+S** per salvare il file e quindi **CTRL+Q** per uscire dall'editor.

A questo punto è il momento di sostituire il codice del modello nel file **Program.cs** usando l'editor in Cloud Shell.

### Aggiungere il codice iniziale per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```bash
    code Program.cs
    ```

1. Sostituire qualsiasi codice esistente con il frammento di codice seguente. 

    Il codice fornisce la struttura complessiva dell'app. Esaminare i commenti nel codice per comprenderne il funzionamento. Per completare l'applicazione, si aggiungerà codice nelle aree specificate più avanti nell'esercizio. 

    ```csharp
    using Microsoft.Azure.Cosmos;
    using dotenv.net;
    
    string databaseName = "myDatabase"; // Name of the database to create or use
    string containerName = "myContainer"; // Name of the container to create or use
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    string cosmosDbAccountUrl = envVars["DOCUMENT_ENDPOINT"];
    string accountKey = envVars["ACCOUNT_KEY"];
    
    if (string.IsNullOrEmpty(cosmosDbAccountUrl) || string.IsNullOrEmpty(accountKey))
    {
        Console.WriteLine("Please set the DOCUMENT_ENDPOINT and ACCOUNT_KEY environment variables.");
        return;
    }
    
    // CREATE THE COSMOS DB CLIENT USING THE ACCOUNT URL AND KEY
    
    
    try
    {
        // CREATE A DATABASE IF IT DOESN'T ALREADY EXIST
    
    
        // CREATE A CONTAINER WITH A SPECIFIED PARTITION KEY
    
    
        // DEFINE A TYPED ITEM (PRODUCT) TO ADD TO THE CONTAINER
    
    
        // ADD THE ITEM TO THE CONTAINER
    
    
    }
    catch (CosmosException ex)
    {
        // Handle Cosmos DB-specific exceptions
        // Log the status code and error message for debugging
        Console.WriteLine($"Cosmos DB Error: {ex.StatusCode} - {ex.Message}");
    }
    catch (Exception ex)
    {
        // Handle general exceptions
        // Log the error message for debugging
        Console.WriteLine($"Error: {ex.Message}");
    }
    
    // This class represents a product in the Cosmos DB container
    public class Product
    {
        public string? id { get; set; }
        public string? name { get; set; }
        public string? description { get; set; }
    }
    ```

Aggiungere quindi il codice nelle aree specificate dei progetti per creare il client, il database, il contenitore e aggiungere un elemento di esempio al contenitore.

### Aggiungere codice per creare il client ed eseguire operazioni 

1. Aggiungere il codice seguente nello spazio dopo il commento **// CREATE THE COSMOS DB CLIENT USING THE ACCOUNT URL AND KEY**. Questo codice definisce il client usato per connettersi all'account di Azure Cosmos DB.

    ```csharp
    CosmosClient client = new(
        accountEndpoint: cosmosDbAccountUrl,
        authKeyOrResourceToken: accountKey
    );
    ```

    >Nota: È consigliabile usare **DefaultAzureCredential** dalla libreria *Azure Identity*. Ciò può richiedere alcuni requisiti di configurazione aggiuntivi in Azure a seconda della configurazione della sottoscrizione. 

1. Aggiungere il codice seguente nello spazio dopo il commento **// CREATE A DATABASE IF IT DOESN'T ALREADY EXIST**. 

    ```csharp
    Database database = await client.CreateDatabaseIfNotExistsAsync(databaseName);
    Console.WriteLine($"Created or retrieved database: {database.Id}");
    ```

1. Aggiungere il codice seguente nello spazio dopo il commento **// CREATE A CONTAINER WITH A SPECIFIED PARTITION KEY**. 

    ```csharp
    Container container = await database.CreateContainerIfNotExistsAsync(
        id: containerName,
        partitionKeyPath: "/id"
    );
    Console.WriteLine($"Created or retrieved container: {container.Id}");
    ```

1. Aggiungere il codice seguente nello spazio dopo il commento **// DEFINE A TYPED ITEM (PRODUCT) TO ADD TO THE CONTAINER**. In questo modo si definisce l'elemento aggiunto al contenitore.

    ```csharp
    Product newItem = new Product
    {
        id = Guid.NewGuid().ToString(), // Generate a unique ID for the product
        name = "Sample Item",
        description = "This is a sample item in my Azure Cosmos DB exercise."
    };
    ```

1. Aggiungere il codice seguente nello spazio dopo il commento **// ADD THE ITEM TO THE CONTAINER**. 

    ```csharp
    ItemResponse<Product> createResponse = await container.CreateItemAsync(
        item: newItem,
        partitionKey: new PartitionKey(newItem.id)
    );

    Console.WriteLine($"Created item with ID: {createResponse.Resource.id}");
    Console.WriteLine($"Request charge: {createResponse.RequestCharge} RUs");
    ```

1. Dopo aver completato il codice, salvare lo stato di avanzamento usando **CTRL+S** per salvare il file e **CTRL+Q** per uscire dall'editor.

1. Eseguire il comando seguente in Cloud Shell per testare se sono presenti eventuali errori nel progetto. Se vengono rilevati errori, aprire il file *Program.cs* nell'editor e verificare se manca codice o se il codice è stato incollato in modo errato.

    ```
    dotnet build
    ```

Dopo aver completato il progetto, è possibile eseguire l'applicazione e verificare i risultati nel portale di Azure.

## Eseguire l'applicazione e verificare i risultati

1. Eseguire il comando `dotnet run` se si usa Cloud Shell. L'output sarà simile all'esempio seguente.

    ```
    Created or retrieved database: myDatabase
    Created or retrieved container: myContainer
    Created item: c549c3fa-054d-40db-a42b-c05deabbc4a6
    Request charge: 6.29 RUs
    ```

1. Nel portale di Azure passare alla risorsa di Azure Cosmos DB creata in precedenza. Nel riquadro di spostamento a sinistra selezionare **Esplora dati**. In **Esplora dati** selezionare **myDatabase**, quindi espandere **myContainer**. È possibile visualizzare l'elemento creato selezionando **Elementi**.

    ![Screenshot che mostra la posizione degli elementi in Esplora dati.](./media/01/cosmos-data-explorer.png)

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
