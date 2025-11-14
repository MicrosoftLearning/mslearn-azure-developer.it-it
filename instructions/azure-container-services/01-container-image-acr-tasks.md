---
lab:
  topic: Azure container services
  title: Compilare ed eseguire un'immagine del contenitore con Attività del Registro Azure Container
  description: Informazioni su come usare i comandi dell'interfaccia della riga di comando di Azure per compilare ed eseguire immagini del contenitore con Attività del Registro Azure Container.
---

# Compilare ed eseguire un'immagine del contenitore con Attività del Registro Azure Container

In questo esercizio si compila un'immagine del contenitore dal codice dell'applicazione e si esegue il push di tale immagine in Registro Azure Container usando l'interfaccia della riga di comando di Azure. Si apprenderà come preparare l'app per la containerizzazione, creare un'istanza di Registro Azure Container e archiviare l'immagine del contenitore in Azure.

Attività eseguite in questo esercizio:

* Creare una risorsa del Registro Azure Container
* Compilare ed eseguire il push di un'immagine da un Dockerfile
* Verificare i risultati
* Eseguire l'immagine su Registro dei Contenitori di Azure

Questo esercizio richiede circa**20** minuti.

## Creare una risorsa del Registro Azure Container

1. Nel browser passare al portale di Azure[https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.

1. Usare il pulsante **[\>_]** a destra della barra di ricerca, nella parte superiore della pagina, per aprire una nuova sessione di Cloud Shell nel portale di Azure selezionando un ambiente***Bash***. Cloud Shell fornisce un'interfaccia della riga di comando in un riquadro nella parte inferiore del portale di Azure. Se viene richiesto di selezionare un account di archiviazione per salvare in modo permanente i file, selezionare**Nessun account di archiviazione richiesto**, selezionare la sottoscrizione e quindi**Applica**.

    > **Nota**: se in precedenza è stata creata una sessione Cloud Shell che usa un ambiente*PowerShell*, passare a***Bash***.

1. Creare un gruppo di risorse con le risorse necessarie per questo esercizio. Sostituire**myResourceGroup** con un nome da usare per il gruppo di risorse. Se necessario, è possibile sostituire**eastus** con un'area nelle vicinanze. Se si ha già un gruppo di risorse da usare, procedere con il passaggio successivo.

    ```
    az group create --location eastus --name myResourceGroup
    ```

1. Eseguire il comando seguente per creare un registro contenitori di base. Il nome del registro deve essere univoco in Azure e deve contenere da 5 a 50 caratteri numerici e minuscoli. Sostituire**myResourceGroup** con il nome usato in precedenza e**myContainerRegistry** con un valore univoco.

    ```bash
    az acr create --resource-group myResourceGroup \
        --name myContainerRegistry --sku Basic
    ```

    > **Nota:** Il comando crea un registro*Basic*, ovvero un'opzione ottimizzata in termini di costo per sviluppatori che iniziano a usare Registro Azure Container.

## Compilare ed eseguire il push di un'immagine da un Dockerfile

Successivamente, si compila e si esegue il push di un'immagine in base a un Dockerfile.

1. Eseguire il comando seguente per creare il Dockerfile. Il Dockerfile contiene una singola riga che fa riferimento all'immagine*hello-world* ospitata in Microsoft Container Registry.

    ```bash
    echo FROM mcr.microsoft.com/hello-world > Dockerfile
    ```

1. Eseguire il comando**az acr build** seguente, che consente di compilare l'immagine e, dopo la corretta compilazione, ne esegue il push nel registro. Sostituire**myContainerRegistry** con il nome creato in precedenza.

    ```bash
    az acr build --image sample/hello-world:v1  \
        --registry myContainerRegistry \
        --file Dockerfile .
    ```

    Di seguito è riportato un esempio abbreviato dell'output del comando precedente che mostra le ultime righe con i risultati finali. Come si può vedere, nel campo*repository* è elencata l'immagine*sample/hello-word*.

    ```
    - image:
        registry: myContainerRegistry.azurecr.io
        repository: sample/hello-world
        tag: v1
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      runtime-dependency:
        registry: mcr.microsoft.com
        repository: hello-world
        tag: latest
        digest: sha256:92c7f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e899a
      git: {}
    
    
    Run ID: cf1 was successful after 11s
    ```

## Verificare i risultati

1. Eseguire il comando seguente per elencare i repository nel registro. Sostituire**myContainerRegistry** con il nome creato in precedenza.

    ```bash
    az acr repository list --name myContainerRegistry --output table
    ```

    Output:

    ```
    Result
    ----------------
    sample/hello-world
    ```

1. Eseguire il comando seguente per elencare i tag nel repository**sample/hello-world**. Sostituire**myContainerRegistry** con il nome usato in precedenza.

    ```bash
    az acr repository show-tags --name myContainerRegistry \
        --repository sample/hello-world --output table
    ```

    Output:

    ```
    Result
    --------
    v1
    ```

## Eseguire l'immagine nel Registro Azure Container

1. Eseguire l'immagine del contenitore*sample/hello-world:v1* dal registro contenitori con il comando**az acr run**. Nell'esempio seguente viene usato **$Registry** per specificare il registro in cui eseguire il comando. Sostituire**myContainerRegistry** con il nome usato in precedenza.

    ```bash
    az acr run --registry myContainerRegistry \
        --cmd '$Registry/sample/hello-world:v1' /dev/null
    ```

    Il parametro**cmd** di questo esempio esegue il contenitore nella configurazione predefinita, ma**cmd** supporta altri parametri**docker run** o anche altri comandi**docker**. 

    L'output di esempio seguente è abbreviato:

    ```
    Packing source code into tar to upload...
    Uploading archived source code from '/tmp/run_archive_ebf74da7fcb04683867b129e2ccad5e1.tar.gz'...
    Sending context (1.855 KiB) to registry: mycontainerre...
    Queued a run with ID: cab
    Waiting for an agent...
    2019/03/19 19:01:53 Using acb_vol_60e9a538-b466-475f-9565-80c5b93eaa15 as the home volume
    2019/03/19 19:01:53 Creating Docker network: acb_default_network, driver: 'bridge'
    2019/03/19 19:01:53 Successfully set up Docker network: acb_default_network
    2019/03/19 19:01:53 Setting up Docker configuration...
    2019/03/19 19:01:54 Successfully set up Docker configuration
    2019/03/19 19:01:54 Logging in to registry: mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Successfully logged into mycontainerregistry008.azurecr.io
    2019/03/19 19:01:55 Executing step ID: acb_step_0. Working directory: '', Network: 'acb_default_network'
    2019/03/19 19:01:55 Launching container with name: acb_step_0
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    2019/03/19 19:01:56 Successfully executed container: acb_step_0
    2019/03/19 19:01:56 Step ID: acb_step_0 marked as successful (elapsed time in seconds: 0.843801)
    
    Run ID: cab was successful after 6s
    ```

## Pulire le risorse

Dopo aver completato l'esercizio, è consigliabile eliminare le risorse cloud create per evitare un utilizzo non necessario delle risorse.

1. Nel browser passare al portale di Azure[https://portal.azure.com](https://portal.azure.com). Accedere con le credenziali di Azure, se richiesto.
1. Passare al gruppo di risorse creato e visualizzare il contenuto delle risorse usate in questo esercizio.
1. Sulla barra degli strumenti selezionare**Elimina gruppo di risorse**.
1. Immettere il nome del gruppo di risorse e confermarne l'eliminazione.

> **ATTENZIONE:** Se si elimina un gruppo di risorse, vengono eliminate tutte le risorse contenute in esso. Se si sceglie un gruppo di risorse esistente per questo esercizio, verranno eliminate anche tutte le risorse esistenti esterne all'ambito di questo esercizio.
