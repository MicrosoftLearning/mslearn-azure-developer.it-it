---
lab:
  topic: Azure container services
  title: Distribuire un contenitore in Istanze di Azure Container usando i comandi dell'interfaccia della riga di comando di Azure
  description: Informazioni su come usare i comandi dell'interfaccia della riga di comando di Azure per distribuire un contenitore in Istanze di Azure Container.
---

# Distribuire un contenitore in Istanze di Azure Container usando i comandi dell'interfaccia della riga di comando di Azure

In questo esercizio si distribuisce ed esegue un contenitore in Istanze di Azure Container usando l'interfaccia della riga di comando di Azure. Si apprenderà come creare un gruppo di contenitori, specificare le impostazioni del contenitore e verificare che l'applicazione in contenitori sia in esecuzione nel cloud.

Attività eseguite in questo esercizio:

* Creare risorse dell'istanza di Azure Container in Azure
* Creare e distribuire un contenitore
* Verificare che il contenitore sia in esecuzione

Questo esercizio richiede circa **15** minuti.

## Creare un gruppo di risorse

1. Nel browser passare al portale di Azure [https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente ***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare **Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi **Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente *PowerShell*, passare a ***Bash***.

1. Creare un gruppo di risorse con le risorse necessarie per questo esercizio. Sostituire **myResourceGroup** con un nome da usare per il gruppo di risorse. Se necessario, è possibile sostituire **eastus** con un'area nelle vicinanze. Se si ha già un gruppo di risorse da usare, procedere con il passaggio successivo.

    ```
    az group create --location eastus --name myResourceGroup
    ```

## Creare e distribuire un contenitore

Si crea un contenitore specificando un nome, un'immagine Docker e un gruppo di risorse di Azure nel comando **az container create**. Esporre il contenitore in Internet specificando un'etichetta del nome DNS.

1. Eseguire il comando seguente per creare un nome DNS usato per esporre il contenitore a Internet. Il nome DNS seve essere univoco, eseguire questo comando da Cloud Shell per creare una variabile contenente un nome univoco.

    ```bash
    DNS_NAME_LABEL=aci-example-$RANDOM
    ```

1. Eseguire il comando seguente per creare un'istanza del contenitore. Sostituire **myResourceGroup** e **myLocation** con i valori usati in precedenza. Il completamento dell'installazione può richiedere alcuni minuti.

    ```bash
    az container create --resource-group myResourceGroup \
        --name mycontainer \
        --image mcr.microsoft.com/azuredocs/aci-helloworld \
        --ports 80 \
        --dns-name-label $DNS_NAME_LABEL --location myLocation \
        --os-type Linux \
        --cpu 1 \
        --memory 1.5 
    ```

    Nel comando precedente **$DNS_NAME_LABEL** specifica il nome DNS. Il nome dell'immagine, **mcr.microsoft.com/azuredocs/aci-helloworld**, si riferisce a un'immagine Docker che esegue un'applicazione Web Node.js di base.

Passare alla sezione successiva al termine del comando **az container create**.

## Verificare che il contenitore sia in esecuzione

È possibile controllare lo stato di compilazione dei contenitori con il comando **az container show**. 

1. Eseguire il comando seguente per controllare lo stato del provisioning del contenitore creato. Sostituire **myResourceGroup** con il valore usato in precedenza.

    ```bash
    az container show --resource-group myResourceGroup \
        --name mycontainer \
        --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" \
        --out table 
    ```

    Verranno visualizzati il nome di dominio completo (FQDN) del contenitore e il relativo stato di provisioning. Ecco un esempio.

    ```
    FQDN                                    ProvisioningState
    --------------------------------------  -------------------
    aci-wt.eastus.azurecontainer.io         Succeeded
    ```

    > **Nota:** Se il contenitore si trova nello stato **Creating** (Creazione in corso), attendere alcuni istanti ed eseguire di nuovo il comando finché non viene visualizzato lo stato **Succeeded** (Completato).

1. Aprire un browser e passare al nome di dominio completo del contenitore per vederlo in esecuzione. È possibile che venga visualizzato un avviso che indica che il sito non è sicuro.

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare **Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
