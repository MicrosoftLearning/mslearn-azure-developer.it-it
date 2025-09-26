---
lab:
  topic: Azure authentication and authorization
  title: Implementare l'autenticazione interattiva con MSAL.NET
  description: Informazioni su come implementare l'autenticazione interattiva usando MSAL.NET SDK e acquisire un token.
---

# Implementare l'autenticazione interattiva con MSAL.NET

In questo esercizio si registra un'applicazione in Microsoft Entra ID, quindi si crea un'applicazione console .NET che usa MSAL.NET per eseguire l'autenticazione interattiva e acquisire un token di accesso per Microsoft Graph. Si apprenderà come configurare gli ambiti di autenticazione e gestire il consenso utente e si scoprirà in che modo i token vengono memorizzati nella cache per le esecuzioni successive. 

Attività eseguite in questo esercizio:

* Registrare un'applicazione con Microsoft Identity Platform
* Creare un'app console .NET che implementa la classe **PublicClientApplicationBuilder** per configurare l'autenticazione.
* Acquisire un token in modo interattivo usando l'autorizzazione **user.read** di Microsoft Graph.

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
    | **Nome** | Immetti `myMsalApplication`  |
    | **Tipi di account supportati** | Selezionare **Account solo in questa directory dell'organizzazione** |
    | **URI di reindirizzamento (facoltativo)** | Selezionare **Client pubblico/nativo (per dispositivi mobili e desktop)** e immettere `http://localhost` nella casella a destra. |

1. Selezionare **Registrazione**. Microsoft Entra ID assegna un ID applicazione univoco all'app. Quindi, viene visualizzata la pagina **Panoramica** dell'applicazione. 

1. Nella sezione **Informazioni di base** della pagina **Panoramica** registrare l'**ID applicazione (client)** e l'**ID directory (tenant)**. Le informazioni sono necessarie per l'applicazione.

    ![Screenshot che mostra la posizione dei campi da copiare.](./media/01-app-directory-id-location.png)
 
## Creare un'app console .NET per acquisire un token

Ora che le risorse necessarie sono state distribuite in Azure, il passaggio successivo consiste nel configurare l'applicazione console. I passaggi seguenti vengono eseguiti nell'ambiente locale.

1. Creare una cartella con nome **authapp**, o il nome che si preferisce, per il progetto.

1. Avviare **Visual Studio Code**, selezionare **File > Apri cartella** e selezionare la cartella del progetto.

1. Selezionare **Visualizza > Terminale** per aprire un terminale.

1. Eseguire il comando seguente nel terminale VS Code per creare l'applicazione console .NET.

    ```
    dotnet new console
    ```

1. Eseguire i comandi seguenti per aggiungere i pacchetti **Microsoft.Identity.Client** e **dotenv.net** al progetto.

    ```
    dotnet add package Microsoft.Identity.Client
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

1. Premi **CTRL+S** per salvare le modifiche.

### Aggiungere il codice di avvio per il progetto

1. Aprire il file *Program.cs* e sostituire il contenuto esistente con il codice riportato di seguito. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    using Microsoft.Identity.Client;
    using dotenv.net;
    
    // Load environment variables from .env file
    DotEnv.Load();
    var envVars = DotEnv.Read();
    
    // Retrieve Azure AD Application ID and tenant ID from environment variables
    string _clientId = envVars["CLIENT_ID"];
    string _tenantId = envVars["TENANT_ID"];
    
    // ADD CODE TO DEFINE SCOPES AND CREATE CLIENT 
    
    
    
    // ADD CODE TO ACQUIRE AN ACCESS TOKEN
    
    
    ```

1. Premi **CTRL+S** per salvare le modifiche.

### Aggiungere codice per completare l'applicazione

1. Individuare il commento **// ADD CODE TO DEFINE SCOPES AND CREATE CLIENT** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    // Define the scopes required for authentication
    string[] _scopes = { "User.Read" };
    
    // Build the MSAL public client application with authority and redirect URI
    var app = PublicClientApplicationBuilder.Create(_clientId)
        .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
        .WithDefaultRedirectUri()
        .Build();
    ```

1. Individuare il commento **// ADD CODE TO ACQUIRE AN ACCESS TOKEN** e aggiungere il codice seguente direttamente dopo il commento. Assicurarsi di esaminare i commenti nel codice.

    ```csharp
    // Attempt to acquire an access token silently or interactively
    AuthenticationResult result;
    try
    {
        // Try to acquire token silently from cache for the first available account
        var accounts = await app.GetAccountsAsync();
        result = await app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
    }
    catch (MsalUiRequiredException)
    {
        // If silent token acquisition fails, prompt the user interactively
        result = await app.AcquireTokenInteractive(_scopes)
                    .ExecuteAsync();
    }
    
    // Output the acquired access token to the console
    Console.WriteLine($"Access Token:\n{result.AccessToken}");
    ```

1. Premere **CTRL+S** per salvare il file e quindi **CTRL+Q** per uscire dall'editor.

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
    Access Token:
    eyJ0eXAiOiJKV1QiLCJub25jZSI6IlZF.........
    ```

1. Avviare l'applicazione una seconda volta. Si noterà che non si riceve più la notifica di tipo **Autorizzazioni richieste**. L'autorizzazione concessa in precedenza è stata memorizzata nella cache.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
