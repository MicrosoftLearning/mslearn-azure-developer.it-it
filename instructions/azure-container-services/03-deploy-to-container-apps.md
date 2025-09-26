---
lab:
  topic: Azure container services
  title: Distribuire un contenitore in App contenitore di Azure con l'interfaccia della riga di comando di Azure
  description: Informazioni su come usare i comandi dell'interfaccia della riga di comando di Azure per creare un ambiente sicuro di App contenitore di Azure e distribuire un contenitore.
---

# Distribuire un contenitore in App contenitore di Azure con l'interfaccia della riga di comando di Azure

In questo esercizio si distribuisce un'applicazione in contenitori in App contenitore di Azure usando l'interfaccia della riga di comando di Azure. Si apprenderà come creare un ambiente di app contenitore, distribuire il contenitore e verificare che l'applicazione sia in esecuzione in Azure.

Attività eseguite in questo esercizio:

* Creare risorse in Azure
* Creare un ambiente di App contenitore di Azure
* Distribuire un'app contenitore nell'ambiente

Questo esercizio richiede circa **15** minuti.

## Creare un gruppo di risorse e preparare l'ambiente di Azure

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Creare un gruppo di risorse con le risorse necessarie per questo esercizio. Sostituire **myResourceGroup** con un nome da usare per il gruppo di risorse. Se necessario, è possibile sostituire **eastus** con un'area nelle vicinanze. Se si ha già un gruppo di risorse da usare, procedere con il passaggio successivo.

    ```azurecli
    az group create --location eastus --name myResourceGroup
    ```

1. Eseguire il comando seguente per assicurarsi che sia installata la versione più recente dell'estensione App contenitore di Azure per l'interfaccia della riga di comando.

    ```azurecli
    az extension add --name containerapp --upgrade
    ```

### Registrare gli spazi dei nomi

È necessario registrare due spazi dei nomi per App contenitore di Azure ed è possibile assicurarsi che siano registrati tramite i passaggi seguenti. Il completamento di ogni registrazione può richiedere alcuni minuti se non sono già configurati nella sottoscrizione. 

1. Registrare lo spazio dei nomi **Microsoft.App**. 

    ```bash
    az provider register --namespace Microsoft.App
    ```

1. Registrare il provider **Microsoft.OperationalInsights** per l'area di lavoro Log Analytics di Monitoraggio di Azure, se non è stato usato in precedenza:

    ```bash
    az provider register --namespace Microsoft.OperationalInsights
    ```

## Creare un ambiente di App contenitore di Azure

Un ambiente in App contenitore di Azure crea un limite sicuro intorno a un gruppo di app contenitore. Le app contenitore distribuite nello stesso ambiente vengono distribuite nella stessa rete virtuale e scrivono i log nella stessa area di lavoro Log Analytics.

1. Creare un ambiente con il comando **az containerapp env create**. Sostituire **myResourceGroup** e **myLocation** con i valori usati in precedenza. Il completamento dell'installazione può richiedere alcuni minuti.

    ```bash
    az containerapp env create \
        --name my-container-env \
        --resource-group myResourceGroup \
        --location myLocation
    ```

## Distribuire un'app contenitore nell'ambiente

Al termine della distribuzione dell'ambiente dell'app contenitore, è possibile distribuire un'immagine del contenitore nell'ambiente.

1. Distribuire un'immagine del contenitore di app di esempio con il comando **containerapp create**. Sostituire **myResourceGroup** con il valore usato in precedenza.

    ```bash
    az containerapp create \
        --name my-container-app \
        --resource-group myResourceGroup \
        --environment my-container-env \
        --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
        --target-port 80 \
        --ingress 'external' \
        --query properties.configuration.ingress.fqdn
    ```

    Impostando **--ingress** su **external** si rende l'app contenitore disponibile per le richieste pubbliche. Il comando restituisce un collegamento per accedere all'app.

    ```
    Container app created. Access your app at <url>
    ```

Per verificare la distribuzione, selezionare l'URL restituito dal comando **az containerapp create** per verificare che l'app contenitore sia in esecuzione.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
