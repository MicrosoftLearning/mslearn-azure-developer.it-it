---
lab:
  topic: Secure solutions in Azure
  title: Recuperare le impostazioni di configurazione dalla Configurazione app di Azure
  description: Informazioni su come creare una risorsa di Configurazione app di Azure e impostare le informazioni di configurazione con l'interfaccia della riga di comando di Azure. Usare quindi **ConfigurationBuilder** per recuperare le impostazioni per l'applicazione.
---

# Recuperare le impostazioni di configurazione dalla Configurazione app di Azure

In questo esercizio si crea una risorsa di Configurazione app di Azure, si archiviano le impostazioni di configurazione usando l'interfaccia della riga di comando di Azure e si compila un'applicazione console .NET che usa **ConfigurationBuilder** per recuperare i valori di configurazione. Si apprenderà come organizzare le impostazioni con chiavi gerarchiche e autenticare l'applicazione per accedere ai dati di configurazione basati sul cloud.

Attività eseguite in questo esercizio:

* Creare una risorsa di Configurazione app di Azure
* Archiviare le informazioni di configurazione della stringa di connessione
* Creare un'app console .NET per recuperare le informazioni di configurazione
* Pulire le risorse

Il completamento di questo esercizio richiede circa **15** minuti.

## Creare una risorsa di Configurazione app di Azure e aggiungere le informazioni di configurazione

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
    appConfigName=appconfigname$RANDOM
    ```

1. Eseguire il comando seguente per ottenere il nome della risorsa di Configurazione app. Registrare il nome, necessario più avanti nell'esercizio.

    ```
    echo $appConfigName
    ```

1. Eseguire il comando seguente per assicurarsi che il provider **Microsoft.AppConfiguration** sia registrato per la sottoscrizione.

    ```
    az provider register --namespace Microsoft.AppConfiguration
    ```

1. Per completare la registrazione possono essere necessari alcuni minuti. Eseguire il comando seguente per verificare lo stato della registrazione. Procedere con il passaggio successivo quando i risultati restituiscono **Registrato**.

    ```
    az provider show --namespace Microsoft.AppConfiguration --query "registrationState"
    ```

1. Eseguire il comando seguente per creare una risorsa di Configurazione app di Azure. L'esecuzione potrebbe richiedere alcuni minuti.

    ```
    az appconfig create --location $location \
        --name $appConfigName \
        --resource-group $resourceGroup
        --sku Free
    ```

    >**Suggerimento:** Se si verifica un problema durante la creazione della risorsa AppConfig a causa di restrizioni di quota usando il valore SKU **Gratuito**, usare invece **Developer**.
    

### Assegnare un ruolo al nome utente di Microsoft Entra

Per recuperare le informazioni di configurazione, è necessario assegnare all'utente Microsoft Entra il **Ruolo con autorizzazioni di lettura per i dati di Configurazione dell'app**. 

1. Eseguire il comando seguente per recuperare **userPrincipalName** dall'account. Rappresenta l'utente a cui verrà assegnato il ruolo.

    ```
    userPrincipal=$(az rest --method GET --url https://graph.microsoft.com/v1.0/me \
        --headers 'Content-Type=application/json' \
        --query userPrincipalName --output tsv)
    ```

1. Eseguire il comando seguente per recuperare l'ID della risorsa del servizio Configurazione app. L'ID della risorsa imposta l'ambito per l'assegnazione di ruolo.

    ```
    resourceID=$(az appconfig show --resource-group $resourceGroup \
        --name $appConfigName --query id --output tsv)
    ```

1. Eseguire il comando seguente per creare e assegnare il **Ruolo con autorizzazioni di lettura per i dati di Configurazione dell'app**.

    ```
    az role assignment create --assignee $userPrincipal \
        --role "App Configuration Data Reader" \
        --scope $resourceID
    ```

Aggiungere quindi una stringa di connessione segnaposto a Configurazione app.

### Aggiungere informazioni di configurazione con l'interfaccia della riga di comando di Azure

In Configurazione app di Azure una chiave come **Dev:conStr** è una chiave gerarchica o con spazi dei nomi. I due punti (:) fungono da delimitatore e creano una gerarchia logica, dove:

* **Dev** rappresenta il prefisso dello spazio dei nomi o dell'ambiente (che indica che questa configurazione è per l'ambiente di sviluppo)
* **conStr** rappresenta il nome della configurazione

Questa struttura gerarchica consente di organizzare le impostazioni di configurazione per ambiente, funzionalità o componente dell'applicazione, semplificando la gestione e il recupero delle impostazioni correlate.

Eseguire il comando seguente per archiviare la stringa di connessione segnaposto. 

```
az appconfig kv set --name $appConfigName \
    --key Dev:conStr \
    --value connectionString \
    --yes
```

Questo comando restituisce un valore JSON. L'ultima riga contiene il valore in testo normale. 

```json
"value": "connectionString"
```

## Creare un'app console .NET per recuperare le informazioni di configurazione

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti in Cloud Shell.

>**Suggerimento:** Ridimensionare Cloud Shell per visualizzare altre informazioni e il codice trascinando il bordo superiore. È anche possibile usare i pulsanti Riduci a icona e Ingrandisci per spostarsi tra Cloud Shell e l'interfaccia principale del portale.

1. Eseguire i comandi seguenti per creare una directory che conterrà il progetto e passare alla directory del progetto.

    ```
    mkdir appconfig
    cd appconfig
    ```

1. Creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Azure.Identity** e **Microsoft.Extensions.Configuration.AzureAppConfiguration** al progetto.

    ```
    dotnet add package Azure.Identity
    dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
    ```

### Aggiungere il codice per il progetto

1. Eseguire il comando seguente in Cloud Shell per iniziare a modificare l'applicazione.

    ```
    code Program.cs
    ```

1. Sostituire i contenuti esistenti con il codice seguente. Assicurarsi di sostituire **YOUR_APP_CONFIGURATION_NAME** con il nome registrato in precedenza e leggere i commenti nel codice.

    ```csharp
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    using Azure.Identity;
    
    // Set the Azure App Configuration endpoint, replace YOUR_APP_CONFIGURATION_NAME
    // with the name of your actual App Configuration service
    
    string endpoint = "https://YOUR_APP_CONFIGURATION_NAME.azconfig.io"; 
    
    // Configure which authentication methods to use
    // DefaultAzureCredential tries multiple auth methods automatically
    DefaultAzureCredentialOptions credentialOptions = new()
    {
        ExcludeEnvironmentCredential = true,
        ExcludeManagedIdentityCredential = true
    };
    
    // Create a configuration builder to combine multiple config sources
    var builder = new ConfigurationBuilder();
    
    // Add Azure App Configuration as a source
    // This connects to Azure and loads configuration values
    builder.AddAzureAppConfiguration(options =>
    {
        
        options.Connect(new Uri(endpoint), new DefaultAzureCredential(credentialOptions));
    });
    
    // Build the final configuration object
    try
    {
        var config = builder.Build();
        
        // Retrieve a configuration value by key name
        Console.WriteLine(config["Dev:conStr"]);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error connecting to Azure App Configuration: {ex.Message}");
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

1. Eseguire il comando seguente per avviare l'app console. L'app visualizzerà il valore **connectionString** assegnato all'impostazione **Dev:conStr** in precedenza nell'esercizio.

    ```
    dotnet run
    ```

    L'app visualizzerà il valore **connectionString** assegnato all'impostazione **Dev:conStr** in precedenza nell'esercizio.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
