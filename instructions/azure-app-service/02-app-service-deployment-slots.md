---
lab:
  topic: Azure App Service
  title: Scambiare gli slot di distribuzione nel Servizio app di Azure
  description: 'Informazioni su come scambiare gli slot di distribuzione nel Servizio app di Azure. In questo esercizio si distribuisce una semplice app in Servizio app, si apporta una piccola modifica all''app e la si distribuisce in uno slot di staging, infine si scambiano gli slot in modo che l''app aggiornata sia in produzione.'
---

# Scambiare gli slot di distribuzione nel Servizio app di Azure

In questo esercizio si distribuisce un sito Web HTML statico in Servizio app di Azure, si crea uno slot di distribuzione di staging, si apportano modifiche al codice e le si distribuisce nello slot di staging, quindi si scambiano gli slot di staging e di produzione per promuovere le modifiche all'ambiente di produzione. Si apprenderà come usare gli slot di distribuzione per gli aggiornamenti sicuri delle applicazioni e le distribuzioni blu-verde.

Attività eseguite in questo esercizio:

* Scaricare e distribuire l'app di esempio in Servizio app di Azure.
* Creare uno slot di distribuzione di staging
* Apportare una modifica all'app di esempio e distribuirla nello slot di staging.
* Scambiare gli slot di staging e di produzione predefiniti per spostare le modifiche nello slot di produzione.

Questo esercizio richiede circa **30** minuti.

## Scaricare ed eseguire l'app di esempio

In questa sezione si scarica l'app di esempio e si impostano le variabili per semplificare l'immissione dei comandi, quindi si crea una risorsa del Servizio app di Azure e si distribuisce un sito HTML statico usando i comandi dell'interfaccia della riga di comando di Azure.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Nella barra degli strumenti di Cloud Shell scegliere **Vai alla versione classica** dal menu **Impostazioni**. Questa operazione è necessaria per usare l'editor di codice.

1. Eseguire il comando **git** seguente per clonare il repository dell'app di esempio.

    ```bash
    git clone https://github.com/Azure-Samples/html-docs-hello-world.git
    ```

1. Impostare le variabili per contenere i nomi del gruppo di risorse e dell'app eseguendo i comandi seguenti. È possibile sostituire il valore **rg-mywebapp** per **resourceGroup** se si ha un gruppo di risorse che si vuole usare. Prendere nota del valore di **appName** visualizzato dopo l'esecuzione dei comandi, che sarà necessario più avanti in questo esercizio.

    ```bash
    resourceGroup=rg-mywebapp

    appName=mywebapp$RANDOM
    echo $appName
    ```

1. Passare alla directory contenente il codice di esempio ed eseguire il comando **az webapp up**. **Nota:** L'esecuzione del comando può impiegare alcuni minuti.

    ```bash
    cd html-docs-hello-world

    az webapp up -g $resourceGroup -n $appName --sku P0V3 --html
    ```

    Ora che la distribuzione è terminata, è possibile visualizzare l'app Web.

1. Nel portale di Azure passare all'app Web distribuita. È possibile immettere il nome annotato in precedenza nella barra di ricerca **Cerca risorse, servizi e documentazione (G + /)** e selezionare la risorsa dall'elenco.

1. Selezionare il collegamento all'app Web che si trova nel campo **Dominio predefinito** della sezione **Funzionalità essenziali**. Il collegamento aprirà il sito in una nuova scheda.

## Distribuire il codice aggiornato in uno slot di distribuzione

In questa sezione si crea uno slot di distribuzione, si modifica il codice HTML nell'app e si distribuisce il codice aggiornato nel nuovo slot di distribuzione.

### Creare uno slot di distribuzione 

1. Tornare alla scheda contenente il portale di Azure e Cloud Shell.

1. Immettere il comando seguente in Cloud Shell per creare uno slot di distribuzione denominato *staging*.

    ```bash
    az webapp deployment slot create -n $appName -g $resourceGroup --slot staging
    ```

1. Attendere il completamento del comando e quindi selezionare **Distribuzione > Slot di distribuzione** nel menu a sinistra per visualizzare gli slot di distribuzione per l'app Web. Si noti che il nome del nuovo slot contiene *-staging* aggiunto al nome dell'app Web

### Aggiornare il codice e distribuirlo nello slot di staging

1. In Cloud Shell digitare **code index.html** per aprire l'editor. Individuare il tag di intestazione **\<h1\>** e modificare *Servizio app di Azure - Sito HTML statico di esempio* in *Slot di staging del Servizio app di Azure* o qualsiasi altro elemento desiderato.

1. Usare i comandi **ctrl-s** per salvare e **ctrl-q** per uscire.

1. In Cloud Shell eseguire il comando seguente per creare un file ZIP del progetto aggiornato. Per il passaggio successivo è necessario un file ZIP o un file WAR (Web Application Resource).

    ```bash
    zip -r stagingcode.zip .
    ```

1. Eseguire il comando seguente in Cloud Shell per distribuire gli aggiornamenti nello slot di staging.

    ```bash
    az webapp deploy -g $resourceGroup -n $appName --src-path ./stagingcode.zip --slot staging
    ```

1. Selezionare **Distribuzione > Slot di distribuzione** nel menu a sinistra dell'app Web e quindi selezionare lo slot di staging creato in precedenza.

1. Selezionare il collegamento nel campo **Dominio predefinito** della sezione **Funzionalità essenziali**. Il collegamento aprirà il sito Web per lo slot di staging in una nuova scheda.

## Scambiare gli slot di staging e di produzione

È possibile eseguire uno scambio nel portale di Azure con l'opzione **Scambio** della barra degli strumenti. L'opzione **Scambio** verrà visualizzata nella barra degli strumenti se si seleziona **Panoramica** o **Distribuzione > Slot di distribuzione** nel menu a sinistra dell'app Web.

1. Nel portale di Azure selezionare **Scambio** nella barra degli strumenti per aprire il pannello **Scambio**.

1. Esaminare le impostazioni nel pannello Scambio. L'**Origine** deve mostrare lo slot **-staging** e la **Destinazione** deve mostrare lo slot di produzione predefinito.

    ![Screenshot del pannello Scambio.](./media/02/app-service-swap-panel.png)

1. Selezionare **Avvia scambio** e attendere il completamento dell'operazione. È possibile tenere traccia del completamento nel pannello **Notifiche** che è possibile aprire selezionando l'icona a forma di campanello nella parte superiore del portale.

1. Per verificare lo scambio, passare all'app Web distribuita. Immettere il nome dell'app Web creato in precedenza (ad esempio, *mywebapp12360*) nella barra di ricerca **Cerca risorse, servizi e documentazione (G + /)** e quindi selezionare la risorsa dall'elenco.

1. Selezionare il collegamento all'app Web che si trova nel campo **Dominio predefinito** della sezione **Funzionalità essenziali**. Il collegamento aprirà il sito (slot di produzione) in una nuova scheda.

1. Verificare le modifiche; potrebbe essere necessario aggiornare la pagina affinché vengano visualizzate.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
