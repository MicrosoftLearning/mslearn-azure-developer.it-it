---
lab:
  topic: Azure authentication and authorization
  title: Recuperare le informazioni sul profilo utente con Microsoft Graph SDK
  description: Informazioni su come recuperare le informazioni sul profilo utente da Microsoft Graph.
---

# Recuperare le informazioni sul profilo utente con Microsoft Graph SDK

In questo esercizio si crea un'app .NET per l'autenticazione con Microsoft Entra ID e si richiede un token di accesso, quindi si chiama l'API Microsoft Graph per recuperare e visualizzare le informazioni del profilo utente. Si apprenderà come configurare le autorizzazioni e interagire con Microsoft Graph dall'applicazione.

Attività eseguite in questo esercizio:

* Registrare un'applicazione con Microsoft Identity Platform
* Creare un'applicazione console .NET che implementa l'autenticazione interattiva e usa la classe **GraphServiceClient** per recuperare le informazioni del profilo utente.

Questo esercizio richiede circa **15** minuti.

## Prima di iniziare

Per completare l'esercizio, è necessario avere:

* Una sottoscrizione di Azure. Se non si dispone ancora di una sottoscrizione, è possibile [registrarsi per ottenere una](https://azure.microsoft.com/).

* [Visual Studio Code](https://code.visualstudio.com/) in una delle [piattaforme supportate](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

* [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) o versione successiva.

* [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) per Visual Studio Code.

## Registrare una nuova applicazione

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Nel portale cercare e selezionare **Registrazioni app**. 

1. Selezionare **+ Nuova registrazione** e quando viene visualizzata la pagina **Registra un'applicazione** immettere le informazioni di registrazione dell'applicazione:

    | Campo | Valore |
    |--|--|
    | **Nome** | Immetti `myGraphApplication`  |
    | **Tipi di account supportati** | Selezionare **Account solo in questa directory dell'organizzazione** |
    | **URI di reindirizzamento (facoltativo)** | Selezionare **Client pubblico/nativo (per dispositivi mobili e desktop)** e immettere `http://localhost` nella casella a destra. |

1. Selezionare **Registrazione**. Microsoft Entra ID assegna un ID applicazione univoco all'app. Quindi, viene visualizzata la pagina **Panoramica** dell'applicazione. 

1. Nella sezione **Informazioni di base** della pagina **Panoramica** registrare l'**ID applicazione (client)** e l'**ID directory (tenant)**. Le informazioni sono necessarie per l'applicazione.

    ![Screenshot che mostra la posizione dei campi da copiare.](./media/01-app-directory-id-location.png)
 
## Creare un'app console .NET per inviare e ricevere messaggi

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti nell'ambiente locale.

1. Creare una cartella con nome **graphapp**, o il nome che si preferisce, per il progetto.

1. Avviare **Visual Studio Code**, selezionare **File > Apri cartella** e selezionare la cartella del progetto.

1. Selezionare **Visualizza > Terminale** per aprire un terminale.

1. Eseguire il comando seguente nel terminale VS Code per creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Azure.Identity**, **Microsoft.Graph** e **dotenv.net** al progetto.

    ```
    dotnet add package Azure.Identity
    dotnet add package Microsoft.Graph
    dotnet add package dotenv.net
    ```

### Configurare l'applicazione console

In questa sezione si crea e si modifica un file con estensione **env** in cui saranno contenuti i segreti registrati in precedenza. 

1. Selezionare **File > Nuovo file** e creare un file con estensione *env* nella cartella del progetto.

1. Aprire il file con estensione **env** e aggiungere il codice seguente. Sostituire **YOUR_CLIENT_ID**e **YOUR_TENANT_ID** con i valori registrati in precedenza.

    ```
    CLIENT_ID="YOUR_CLIENT_ID"
    TENANT_ID="YOUR_TENANT_ID"
    ```

1. Premere **CTRL+S** per salvare il file.

### Aggiungere il codice di avvio per il progetto

1. Aprire il file *Program.cs* e sostituire il contenuto esistente con il codice riportato di seguito. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    using Microsoft.Graph;
    using Azure.Identity;
    using dotenv.net;
    
    // Load environment variables from .env file (if present)
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Read Azure AD app registration values from environment
    string clientId = envVars["CLIENT_ID"];
    string tenantId = envVars["TENANT_ID"];
    
    // Validate that required environment variables are set
    if (string.IsNullOrEmpty(clientId) || string.IsNullOrEmpty(tenantId))
    {
        Console.WriteLine("Please set CLIENT_ID and TENANT_ID environment variables.");
        return;
    }
    
    // ADD CODE TO DEFINE SCOPE AND CONFIGURE AUTHENTICATION
    
    
    
    // ADD CODE TO CREATE GRAPH CLIENT AND RETRIEVE USER PROFILE
    
    
    ```

1. Premi **CTRL+S** per salvare le modifiche.

### Aggiungere codice per completare l'applicazione

1. Individuare il commento **// ADD CODE TO DEFINE SCOPE AND CONFIGURE AUTHENTICATION** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    // Define the Microsoft Graph permission scopes required by this app
    var scopes = new[] { "User.Read" };
    
    // Configure interactive browser authentication for the user
    var options = new InteractiveBrowserCredentialOptions
    {
        ClientId = clientId, // Azure AD app client ID
        TenantId = tenantId, // Azure AD tenant ID
        RedirectUri = new Uri("http://localhost") // Redirect URI for auth flow
    };
    var credential = new InteractiveBrowserCredential(options);
    ```

1. Individuare il commento **// ADD CODE TO CREATE GRAPH CLIENT AND RETRIEVE USER PROFILE** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    // Create a Microsoft Graph client using the credential
    var graphClient = new GraphServiceClient(credential);
    
    // Retrieve and display the user's profile information
    Console.WriteLine("Retrieving user profile...");
    await GetUserProfile(graphClient);
    
    // Function to get and print the signed-in user's profile
    async Task GetUserProfile(GraphServiceClient graphClient)
    {
        try
        {
            // Call Microsoft Graph /me endpoint to get user info
            var me = await graphClient.Me.GetAsync();
            Console.WriteLine($"Display Name: {me?.DisplayName}");
            Console.WriteLine($"Principal Name: {me?.UserPrincipalName}");
            Console.WriteLine($"User Id: {me?.Id}");
        }
        catch (Exception ex)
        {
            // Print any errors encountered during the call
            Console.WriteLine($"Error retrieving profile: {ex.Message}");
        }
    }
    ```

1. Premere **CTRL+S** per salvare il file.

## Eseguire l'applicazione

Ora che l'app è stata completata, è il momento di eseguirla. 

1. Avviare l'applicazione eseguendo il comando seguente:

    ```
    dotnet run
    ```

1. L'app apre il browser predefinito e viene chiesto di selezionare l'account con cui si vuole eseguire l'autenticazione. Se vengono elencati più account, selezionare quello associato al tenant usato nell'app.

1. Se è la prima volta che si esegue l'autenticazione all'app registrata, si riceve una notifica di tipo **Autorizzazioni richieste** che chiede di approvare l'app per l'accesso e la lettura del profilo e il mantenimento dell'accesso ai dati a cui è stato concesso l'accesso. Selezionare **Accetto**.

    ![Screenshot che mostra la notifica relativa alle autorizzazioni richieste](./media/01-granting-permission.png)

1. Nella console verranno visualizzati i risultati simili all'esempio seguente.

    ```
    Retrieving user profile...
    Display Name: <Your account display name>
    Principal Name: <Your principal name>
    User Id: 9f5...
    ```

1. Avviare l'applicazione una seconda volta. Si noterà che non si riceve più la notifica di tipo **Autorizzazioni richieste**. L'autorizzazione concessa in precedenza è stata memorizzata nella cache.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
