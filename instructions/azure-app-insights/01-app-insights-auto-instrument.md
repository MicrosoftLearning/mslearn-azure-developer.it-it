---
lab:
  topic: Application Insights
  title: Monitorare un'applicazione con la strumentazione automatica
  description: 'Informazioni su come monitorare un''applicazione in Application Insights senza modificare il codice configurando la strumentazione automatica '
---

# Monitorare un'applicazione con la strumentazione automatica

In questo esercizio si crea un'app Web del servizio app di Azure con Application Insights abilitato, si configura la strumentazione automatica senza modificare il codice, si crea e si distribuisce un'applicazione Blazor e quindi si visualizzano le metriche dell'applicazione e i dati di errore in Application Insights. L'implementazione di un monitoraggio e di un'osservabilità completi delle applicazioni, senza dover apportare modifiche al codice, semplifica le distribuzioni e le migrazioni.

Attività eseguite in questo esercizio:

* Creare una risorsa dell'app Web con Application Insights abilitato
* Configurare la strumentazione per l'app Web.
* Creare una nuova app Blazor e distribuirla nella risorsa dell'app Web.
* Visualizzare l'attività dell'applicazione in Application Insights
* Pulire le risorse

Il completamento di questo esercizio richiede circa **20** minuti.

## Creare risorse in Azure

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Selezionare **+ Crea una risorsa** che si trova nell'intestazione **Servizi di Azure** nella parte superiore della home page. 
1. Nella barra di ricerca **Cerca nel marketplace** immettere *app Web* e premere **INVIO** per iniziare la ricerca.
1. Nel riquadro App Web selezionare il menu a discesa **Crea** e quindi selezionare **App Web**.

    ![Screenshot del riquadro App Web.](./media/create-web-app-tile.png)

Selezionando **Crea** verrà aperto un modello con alcune schede da compilare con le informazioni sulla distribuzione. I passaggi seguenti illustrano le modifiche da apportare nelle schede pertinenti.

1. Compilare la scheda **Informazioni di base** con le informazioni riportate nella tabella seguente:

    | Impostazione | Azione |
    |--|--|
    | **Abbonamento** | Mantenere il valore predefinito. |
    | **Gruppo di risorse** | Selezionare Crea nuovo, immettere `rg-WebApp`, quindi scegliere OK. È anche possibile selezionare un gruppo di risorse esistente, se si preferisce. |
    | **Nome** | Immettere un nome univoco, ad esempio **YOUR-INITIALS-monitorapp**. Sostituire **YOUR-INITIALS** con le proprie iniziali o un altro valore. Il nome deve essere univoco, quindi potrebbe essere necessario apportare alcune modifiche. |
    | Dispositivo di scorrimento sotto l'impostazione **Nome** | Selezionare il dispositivo di scorrimento per disattivarlo. Questo dispositivo di scorrimento viene visualizzato solo in alcune configurazioni di Azure. |
    | **Pubblicazione** | Selezionare l'opzione **Codice**. |
    | **Stack di runtime** | Nel menu a discesa selezionare **.NET 8 (LTS)**. |
    | **Sistema operativo** | Selezionare **Windows**. |
    | **Area** | Mantenere la selezione predefinita o scegliere un'area nelle vicinanze. |
    | **Piano Windows** | Mantenere la selezione predefinita. |
    | **Piano tariffario** | Selezionare il menu a discesa e scegliere il piano **F1 gratuito**. |

1. Selezionare o passare alla scheda **Monitoraggio e protezione** e immettere le informazioni riportate nella tabella seguente:

    | Impostazione | Azione |
    |--|--|
    | **Abilita Application Insights** | Selezionare **Sì**. |
    | **Application Insights** | Selezionare **Crea nuovo** e verrà visualizzata una finestra di dialogo. Immettere `autoinstrument-insights` nel campo **Nome** della finestra di dialogo. Selezionare quindi **OK** per accettare il nome. |
    | **Area di lavoro** | Immettere `Workspace` se il campo non è già compilato e bloccato. |

1. Selezionare **Rivedi e crea** per esaminare i dettagli della distribuzione. Selezionare quindi **Crea** per creare le risorse.

Il completamento della distribuzione richiederà alcuni minuti. Al termine, selezionare il pulsante **Vai alla risorsa**.

### Configurare le impostazioni di strumentazione

Per abilitare il monitoraggio senza modifiche al codice, è necessario configurare la strumentazione per l'app a livello di servizio.

1. Nel menu di spostamento a sinistra espandere **Monitoraggio** e selezionare **Application Insights**.

1. Individuare la sezione **Instrumentare l'applicazione** e selezionare **.NET Core**.

1. Selezionare **Consigliato** nella sezione **Livello raccolta**.

1. Selezionare **Applica** e quindi confermare le modifiche.

1. Nel menu di spostamento a sinistra selezionare **Panoramica**.

## Creare e distribuire un'app Blazor

In questa sezione dell'esercizio si crea un'app Blazor in Cloud Shell e la si distribuisce nell'app Web creata. Tutti i passaggi descritti in questa sezione vengono eseguiti in Cloud Shell.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Eseguire i comandi seguenti per creare una directory per l'app Blazor e passare a tale directory.

    ```
    mkdir blazor
    cd blazor
    ```

1. Eseguire il comando seguente per creare una nuova app Blazor nella cartella.

    ```
    dotnet new blazor
    ```

1. Eseguire il comando seguente per compilare l'applicazione per assicurarsi che non siano presenti problemi durante la creazione.

    ```
    dotnet build
    ```

### Distribuire l'app nel Servizio app

Per distribuire l'app è prima necessario pubblicarla con il comando **dotnet publish** e quindi creare un file con estensione *zip* per la distribuzione.

1. Eseguire il comando seguente per pubblicare l'app nella directory *publish*.

    ```
    dotnet publish -c Release -o ./publish
    ```

1. Eseguire i comandi seguenti per creare un file con estensione *zip* dell'app pubblicata. Il file con estensione *zip* si trova nella directory radice dell'applicazione.

    ```
    cd publish
    zip -r ../app.zip .
    cd ..
    ```

1. Eseguire il comando seguente per distribuire l'app in Servizio app. Sostituire **YOUR-WEB-APP-NAME** e **YOUR-RESOURCE-GROUP** con i valori usati durante la creazione delle risorse di Servizio app in precedenza nell'esercizio.

    ```
    az webapp deploy --name YOUR-WEB-APP-NAME \
        --resource-group YOUR-RESOURCE-GROUP \
        --src-path ./app.zip
    ```

1. Al termine della distribuzione, selezionare il collegamento nel campo **Dominio predefinito** che si trova nella sezione **Funzionalità essenziali** per aprire l'app in una nuova scheda del browser.

A questo punto è possibile visualizzare alcune metriche dell'applicazione di base in Application Insights. Non chiudere questa scheda, verrà usata nel resto dell'esercizio.

## Visualizzare le metriche in Application Insights

Tornare alla scheda con il portale di Azure e passare alla risorsa di Application Insights creata in precedenza. Nella scheda **Panoramica** vengono visualizzati alcuni grafici di base:

* Richieste non riuscite
* Tempo di risposta del server
* Richieste server
* Disponibilità

In questa sezione verranno eseguite alcune azioni nell'app Web e quindi si tornerà a questa pagina per visualizzare l'attività. La creazione di report delle attività viene ritardata, quindi potrebbero essere necessari alcuni minuti prima che vengano visualizzate nei grafici.

Eseguire i passaggi seguenti nell'app Web.

1. Spostarsi tra le opzioni **Home**, **+ Contatore** e **Meteo** nel menu dell'app Web.

1. Aggiornare la pagina Web più volte per generare i dati **Tempo di risposta del server** e **Richieste del server**.

1. Per creare alcuni errori, selezionare il pulsante **Home** e quindi aggiungere l'URL con **/failures**. Questa route non esiste nell'app Web e genererà un errore. Aggiornare la pagina più volte per generare i dati dell'errore.

1. Tornare alla scheda in cui è in esecuzione Application Insights e attendere un minuto o due affinché le informazioni vengano visualizzate nei grafici. 

1. Nel riquadro di spostamento a sinistra espandere la sezione **Ricerca causa** e selezionare **Errori**. Viene visualizzato il conteggio delle richieste non riuscite insieme a informazioni più dettagliate sui codici di risposta degli errori.

Esplorare altre opzioni di creazione di report per avere un'idea di quali altri tipi di informazioni sono disponibili. 

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
