---
lab:
  topic: Secure solutions in Azure
  title: Creare e recuperare segreti da Azure Key Vault
  description: Informazioni su come creare un insieme di credenziali delle chiavi e creare e recuperare segreti con l'interfaccia della riga di comando di Azure e anche a livello di codice.
---

# Creare e recuperare segreti da Azure Key Vault

In questo esercizio si crea un insieme di credenziali delle chiavi di Azure, si archiviano i segreti usando l'interfaccia della riga di comando di Azure e si compila un'applicazione console .NET in grado di creare e recuperare segreti dall'insieme di credenziali delle chiavi. Si apprenderà come configurare l'autenticazione, gestire i segreti a livello di codice e pulire le risorse al termine delle attività.  

Attività eseguite in questo esercizio:

* Creare risorse di Azure Key Vault
* Archiviare un segreto in un insieme di credenziali delle chiavi usando l'interfaccia della riga di comando di Azure
* Creare un'app console .NET per creare e recuperare segreti
* Pulire le risorse

Questo esercizio richiede circa **30** minuti.

## Creare risorse di Azure Key Vault e aggiungere un segreto

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
    keyVaultName=mykeyvaultname$RANDOM
    ```

1. Eseguire il comando seguente per ottenere il nome dell'insieme di credenziali delle chiavi e registrare il nome. Sarà necessario più avanti nell'esercizio.

    ```
    echo $keyVaultName
    ```

1. Eseguire il comando seguente per creare una risorsa Azure Key Vault. L'esecuzione potrebbe richiedere alcuni minuti.

    ```
    az keyvault create --name $keyVaultName \
        --resource-group $resourceGroup --location $location
    ```

### Assegnare un ruolo al nome utente di Microsoft Entra

Per creare e recuperare un segreto, assegnare all'utente Microsoft Entra il ruolo **Responsabile dei segreti di Key Vault**. In questo modo si concede all'account utente l'autorizzazione per impostare, eliminare ed elencare i segreti. In uno scenario tipico è possibile separare le azioni di creazione/lettura assegnando il ruolo **Responsabile dei segreti di Key Vault** a un gruppo e il ruolo **Utente dei segreti di Key Vault** (che può ottenere ed elencare i segreti) a un altro gruppo.

1. Eseguire il comando seguente per recuperare **userPrincipalName** dall'account. Rappresenta l'utente a cui verrà assegnato il ruolo.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Eseguire il comando seguente per recuperare l'ID della risorsa dell'insieme di credenziali delle chiavi. L'ID della risorsa imposta l'ambito per l'assegnazione di ruolo su un insieme di credenziali delle chiavi specifico.

    ```
    resourceID=$(az keyvault show --resource-group $resourceGroup \
        --name $keyVaultName --query id --output tsv)
    ```

1. Eseguire il comando seguente per creare e assegnare il ruolo **Responsabile dei segreti di Key Vault**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "Key Vault Secrets Officer" \
        --scope $resourceID
    ```

Aggiungere quindi un segreto all'insieme di credenziali delle chiavi creato.

### Aggiungere e recuperare un segreto con l'interfaccia della riga di comando di Azure

1. Eseguire il comando seguente per creare un segreto. 

    ```
    az keyvault secret set --vault-name $keyVaultName \
        --name "MySecret" --value "My secret value"
    ```

1. Eseguire il comando seguente per recuperare il segreto per verificare che sia stato impostato.

    ```
    az keyvault secret show --name "MySecret" --vault-name $keyVaultName
    ```

    Questo comando restituisce un valore JSON. L'ultima riga contiene la password in testo normale. 

    ```json
    "value": "My secret value"
    ```

## Creare un'app console .NET per archiviare e recuperare segreti

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Eseguire i comandi seguenti per creare una directory che conterrà il progetto e passare alla directory del progetto.

    ```
    mkdir keyvault
    cd keyvault
    ```

1. Creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Azure.Identity** e **Azure.Security.KeyVault.Secrets** al progetto.

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.Security.KeyVault.Secrets
    ```

### Aggiungere il codice di avvio per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```
    code Program.cs
    ```

1. Sostituire i contenuti esistenti con il codice seguente. Assicurarsi di sostituire **YOUR-KEYVAULT-NAME** con il nome effettivo dell'insieme di credenziali delle chiavi.

    ```csharp
    using Azure.Identity;
    using Azure.Security.KeyVault.Secrets;
    
    // Replace YOUR-KEYVAULT-NAME with your actual Key Vault name
    string KeyVaultUrl = "https://YOUR-KEYVAULT-NAME.vault.azure.net/";
    
    
    // ADD CODE TO CREATE A CLIENT
    
    
    
    // ADD CODE TO CREATE A MENU SYSTEM
    
    
    
    // ADD CODE TO CREATE A SECRET
    
    
    
    // ADD CODE TO LIST SECRETS
    
    
    ```

1. Premi **CTRL+S** per salvare le modifiche.

### Aggiungere codice per completare l'applicazione

È ora possibile aggiungere il codice per completare l'applicazione.

1. Individuare il commento **// ADD CODE TO CREATE A CLIENT** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Configure authentication options for connecting to Azure Key Vault
    DefaultAzureCredentialOptions options = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create the Key Vault client using the URL and authentication credentials
    var client = new SecretClient(new Uri(KeyVaultUrl), new DefaultAzureCredential(options));
    ```

1. Individuare il commento **// ADD CODE TO CREATE A MENU SYSTEM** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    // Main application loop - continues until user types 'quit'
    while (true)
    {
        // Display menu options to the user
        Console.Clear();
        Console.WriteLine("\nPlease select an option:");
        Console.WriteLine("1. Create a new secret");
        Console.WriteLine("2. List all secrets");
        Console.WriteLine("Type 'quit' to exit");
        Console.Write("Enter your choice: ");
    
        // Read user input and convert to lowercase for easier comparison
        string? input = Console.ReadLine()?.Trim().ToLower();
        
        // Check if user wants to exit the application
        if (input == "quit")
        {
            Console.WriteLine("Goodbye!");
            break;
        }
    
        // Process the user's menu selection
        switch (input)
        {
            case "1":
                // Call the method to create a new secret
                await CreateSecretAsync(client);
                break;
            case "2":
                // Call the method to list all existing secrets
                await ListSecretsAsync(client);
                break;
            default:
                // Handle invalid input
                Console.WriteLine("Invalid option. Please enter 1, 2, or 'quit'.");
                break;
        }
    }
    ```

1. Individuare il commento **// ADD CODE TO CREATE A SECRET** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    async Task CreateSecretAsync(SecretClient client)
    {
        try
        {
            Console.Clear();
            Console.WriteLine("\nCreating a new secret...");
            
            // Get the secret name from user input
            Console.Write("Enter secret name: ");
            string? secretName = Console.ReadLine()?.Trim();
    
            // Validate that the secret name is not empty
            if (string.IsNullOrEmpty(secretName))
            {
                Console.WriteLine("Secret name cannot be empty.");
                return;
            }
            
            // Get the secret value from user input
            Console.Write("Enter secret value: ");
            string? secretValue = Console.ReadLine()?.Trim();
    
            // Validate that the secret value is not empty
            if (string.IsNullOrEmpty(secretValue))
            {
                Console.WriteLine("Secret value cannot be empty.");
                return;
            }
    
            // Create a new KeyVaultSecret object with the provided name and value
            var secret = new KeyVaultSecret(secretName, secretValue);
            
            // Store the secret in Azure Key Vault
            await client.SetSecretAsync(secret);
    
            Console.WriteLine($"Secret '{secretName}' created successfully!");
            Console.WriteLine("Press Enter to continue...");
            Console.ReadLine();
        }
        catch (Exception ex)
        {
            // Handle any errors that occur during secret creation
            Console.WriteLine($"Error creating secret: {ex.Message}");
        }
    }
    ```

1. Individuare il commento **// ADD CODE TO LIST SECRETS** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare il codice e i commenti.

    ```csharp
    async Task ListSecretsAsync(SecretClient client)
    {
        try
        {
            Console.Clear();
            Console.WriteLine("Listing all secrets in the Key Vault...");
            Console.WriteLine("----------------------------------------");
    
            // Get an async enumerable of all secret properties in the Key Vault
            var secretProperties = client.GetPropertiesOfSecretsAsync();
            bool hasSecrets = false;
    
            // Iterate through each secret property to retrieve full secret details
            await foreach (var secretProperty in secretProperties)
            {
                hasSecrets = true;
                try
                {
                    // Retrieve the actual secret value and metadata using the secret name
                    var secret = await client.GetSecretAsync(secretProperty.Name);
                    
                    // Display the secret information to the console
                    Console.WriteLine($"Name: {secret.Value.Name}");
                    Console.WriteLine($"Value: {secret.Value.Value}");
                    Console.WriteLine($"Created: {secret.Value.Properties.CreatedOn}");
                    Console.WriteLine("----------------------------------------");
                }
                catch (Exception ex)
                {
                    // Handle errors for individual secrets (e.g., access denied, secret not found)
                    Console.WriteLine($"Error retrieving secret '{secretProperty.Name}': {ex.Message}");
                    Console.WriteLine("----------------------------------------");
                }
            }
    
            // Inform user if no secrets were found in the Key Vault
            if (!hasSecrets)
            {
                Console.WriteLine("No secrets found in the Key Vault.");
            }
        }
        catch (Exception ex)
        {
            // Handle general errors that occur during the listing operation
            Console.WriteLine($"Error listing secrets: {ex.Message}");
        
        }
        Console.WriteLine("Press Enter to continue...");
        Console.ReadLine();
    }
    ```

1. Premere **CTRL+S** per salvare il file e quindi **CTRL+Q** per uscire dall'editor.

## Accedere in Azure ed eseguire l'app

1. In Cloud Shell immettere il comando seguente per accedere ad Azure.

    ```
    az login
    ```

    **<font color="red">È necessario accedere ad Azure, anche se la sessione di Cloud Shell è già autenticata.</font>**

    > **Nota**: nella maggior parte degli scenari, il semplice uso di *az login* sarà sufficiente. Tuttavia, in caso di sottoscrizioni in più tenant, potrebbe essere necessario specificare il tenant usando il parametro *--tenant*. Per dettagli, vedere [Accedere ad Azure in modo interattivo usando l'interfaccia della riga di comando di Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).

1. Eseguire il comando seguente per avviare l'app console. L'app visualizzerà il sistema di menu per l'applicazione. 

    ```
    dotnet run
    ```

1. È stato creato un segreto all'inizio di questo esercizio, immettere **2** per recuperarlo e visualizzarlo.

1. Immettere **1** e immettere un nome e un valore del segreto per creare un nuovo segreto.

1. Elencare di nuovo i segreti per visualizzare la nuova aggiunta.

Immettere **quit** quando si è terminato di utilizzare l'applicazione.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
