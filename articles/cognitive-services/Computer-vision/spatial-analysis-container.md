---
title: 'Comment installer et exécuter le conteneur d’analyse spatiale : vision par ordinateur'
titleSuffix: Azure Cognitive Services
description: Le conteneur d’analyse spatiale vous permet de détecter des personnes et des distances.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 11/06/2020
ms.author: aahi
ms.openlocfilehash: 2f03746a6a5afc388db2beeff84b3ab4cbd393b5
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95014592"
---
# <a name="install-and-run-the-spatial-analysis-container-preview"></a>Installer et exécuter le conteneur d’analyse spatiale (préversion)

Le conteneur d’analyse spatiale vous permet d’analyser le flux vidéo en temps réel afin de comprendre les relations spatiales entre les personnes, leurs déplacements et les interactions avec des objets dans des environnements physiques. Les conteneurs conviennent particulièrement bien à certaines exigences de sécurité et de gouvernance des données.

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure - [En créer un gratuitement](https://azure.microsoft.com/free/cognitive-services)
* Une fois que vous avez votre abonnement Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="créez une ressource Vision par ordinateur"  target="_blank">créer une ressource Vision par ordinateur <span class="docon docon-navigate-external x-hidden-focus"></span></a> dans le portail Azure pour obtenir votre clé et votre point de terminaison. Une fois le déploiement effectué, cliquez sur **Accéder à la ressource**.
    * Vous aurez besoin de la clé et du point de terminaison de la ressource que vous créez pour exécuter le conteneur d’analyse spatiale. Vous utiliserez votre clé et votre point de terminaison ultérieurement.


### <a name="spatial-analysis-container-requirements"></a>Exigences du conteneur d’analyse spatiale

Pour exécuter le conteneur d’analyse spatiale, vous avez besoin d’un appareil de calcul avec une [GPU NVIDIA Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/). Nous vous recommandons d’utiliser [Azure Stack Edge](https://azure.microsoft.com/products/azure-stack/edge/) avec l’accélération GPU, bien que le conteneur s’exécute sur n’importe quel autre ordinateur de bureau présentant la configuration minimale requise. Nous ferons référence à cet appareil en tant qu’ordinateur hôte.

#### <a name="azure-stack-edge-device"></a>[Appareil Azure Stack Edge](#tab/azure-stack-edge)

Azure Stack Edge est une solution matérielle en tant que service ainsi qu’un appareil de computing en périphérie basé sur l’intelligence artificielle. Il est doté de fonctionnalités de transfert de données via le réseau. Pour obtenir des instructions détaillées sur la préparation et la configuration, consultez la [documentation Azure Stack Edge](../../databox-online/azure-stack-edge-deploy-prep.md).

#### <a name="desktop-machine"></a>[Ordinateur de bureau](#tab/desktop-machine)

#### <a name="minimum-hardware-requirements"></a>Conditions matérielles minimales requises

* RAM système de 4 Go
* 4 Go de RAM GPU
* Processeur 8 cœurs
* 1 GPU NVIDIA Tesla T4
* 20 GB d’espace HDD

#### <a name="recommended-hardware"></a>Matériel recommandé

* RAM système de 32 Go
* 16 Go de RAM GPU
* Processeur 8 cœurs
* 2 GPU NVIDIA Tesla T4
* 50 Go d’espace SSD

Dans cet article, vous allez télécharger et installer les packages logiciels suivants. L’ordinateur hôte doit être en mesure d’exécuter les opérations suivantes (voir les instructions ci-dessous) :

* [Pilotes graphiques NVIDIA](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html) et [NVIDIA CUDA Toolkit](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html)
* Configurations pour [NVIDIA MPS](https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf) (service multiprocessus).
* [Docker CE](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-docker-engine---community-1) et [NVIDIA-Docker2](https://github.com/NVIDIA/nvidia-docker) 
* Runtime [Azure IoT Edge](../../iot-edge/how-to-install-iot-edge.md).

---

| Condition requise | Description |
|--|--|
| Appareil photo | Le conteneur d’analyse spatiale n’est pas lié à une marque de caméra spécifique. La caméra doit : prendre en charge l’encodage RTSP (Real-Time Streaming Protocol) et H. 264, être accessible à l’ordinateur hôte et pouvoir assurer une diffusion en continu à 15FPS et avec une résolution de 1080p. |
| Système d’exploitation Linux | [Ubuntu Desktop 18.04 LTS](http://releases.ubuntu.com/18.04/) doit être installé sur l’ordinateur hôte.  |


## <a name="request-approval-to-run-the-container"></a>Demande d’approbation pour l’exécution du conteneur

Complétez le [formulaire de demande](https://aka.ms/csgate) et envoyez-le afin d’obtenir l’approbation d’exécuter le conteneur.

Le formulaire demande des informations sur vous, votre entreprise et le scénario d’utilisateur pour lequel vous allez utiliser le conteneur. Une fois le formulaire envoyé, l’équipe Azure Cognitive Services Azure l’examinera et vous informera de sa décision par courrier électronique.

> [!IMPORTANT]
> * Dans le formulaire, vous devez utiliser une adresse e-mail associée à un ID d’abonnement Azure.
> * La ressource Vision par ordinateur que vous utilisez pour exécuter le conteneur doit avoir été créée avec l’ID d’abonnement Azure approuvé.

Une fois que vous êtes approuvé, vous pourrez exécuter le conteneur après l’avoir téléchargé à partir de Microsoft Container Registry, décrit plus loin dans cet article.

Vous ne pourrez pas exécuter le conteneur si votre abonnement Azure n’a pas été approuvé.

## <a name="set-up-the-host-computer"></a>Configurer l’ordinateur hôte

Il est recommandé d’utiliser un appareil Azure Stack Edge pour votre ordinateur hôte. Cliquez sur **Ordinateur de bureau** si vous configurez un autre appareil.

#### <a name="azure-stack-edge-device"></a>[Appareil Azure Stack Edge](#tab/azure-stack-edge)

### <a name="configure-compute-on-the-azure-stack-edge-portal"></a>Configurer le calcul sur le portail Azure Stack Edge 
 
L’analyse spatiale utilise les fonctionnalités de calcul d’Azure Stack Edge pour exécuter une solution d’intelligence artificielle. Pour activer les fonctionnalités de calcul, assurez-vous que les conditions suivantes sont remplies : 

* Vous avez [connecté et activé](../../databox-online/azure-stack-edge-deploy-connect-setup-activate.md) votre appareil Azure Stack Edge. 
* Vous avez un système client Windows exécutant PowerShell 5.0 ou une version ultérieure pour accéder à l’appareil.  
* Pour déployer un cluster Kubernetes, vous devez configurer votre appareil Azure Stack Edge via l’**interface utilisateur locale** dans le [portail Azure](https://portal.azure.com/) : 
  1. Configurer la fonctionnalité de calcul sur votre appareil Azure Stack Edge. Pour activer le calcul, accédez à la page **Calcul** dans l’interface web de votre appareil. 
  2. Sélectionnez une interface réseau que vous souhaitez activer pour le calcul, puis cliquez sur **Activer**. Un commutateur virtuel est créé sur votre appareil sur cette interface réseau.
  3. Laissez les adresses IP du nœud de test Kubernetes et les adresses IP des services externes Kubernetes vides.
  4. Cliquez sur **Appliquer**. Cette opération peut prendre environ deux minutes. 

![Configurer le calcul](media/spatial-analysis/configure-compute.png)

### <a name="set-up-an-edge-compute-role-and-create-an-iot-hub-resource"></a>Configurer un rôle de calcul Edge et créer une ressource IoT Hub

Dans le [portail Azure](https://portal.azure.com/), accédez à votre ressource Azure Stack Edge. Sur la page **Vue d’ensemble** ou dans la liste de navigation, cliquez sur le bouton de calcul Edge **Démarrer**. Dans la vignette  **Configurer le calcul Edge** , cliquez sur **Configurer**. 

![Lien](media/spatial-analysis/configure-edge-compute-tile.png)

Dans la page  **Configurer le calcul Edge** , choisissez une ressource IoT Hub existante, ou choisissez d’en créer une nouvelle. Par défaut, un niveau tarifaire Standard (S1) est utilisé pour créer une ressource IoT Hub. Pour utiliser une ressource IoT Hub de niveau Gratuit, créez-en une, puis sélectionnez-la. La ressource IoT Hub utilise les mêmes abonnement et groupe de ressources que la ressource Azure Stack Edge 

Cliquez sur **Créer**. La création de ressources IoT Hub peut prendre quelques minutes. Une fois la ressource IoT Hub créée, la vignette  **Configurer le computing en périphérie** sera mise à jour pour afficher la nouvelle configuration. Pour vérifier que le rôle de computing Edge a été configuré, sélectionnez  **Configuration de la vue** sur la vignette  **Configurer le calcul** .

Quand le rôle de calcul Edge est configuré sur l’appareil Edge, il crée deux appareils : un appareil IoT et un appareil IoT Edge. Ces deux appareils peuvent être visualisés dans la ressource IoT Hub. Le runtime Azure IoT Edge sera déjà en cours d’exécution sur l’appareil IoT Edge.            

> [!NOTE]
> * Actuellement, seule la plateforme Linux est prise en charge pour les appareils IoT Edge. Pour obtenir de l’aide sur le dépannage de l’appareil Azure Stack Edge, consultez l’article [Journalisation et résolution des problèmes](spatial-analysis-logging.md).
> * Pour en savoir plus sur la configuration d’un appareil IoT Edge pour communiquer via un serveur proxy, consultez [Configurer un appareil IoT Edge pour communiquer via un serveur proxy](../../iot-edge/how-to-configure-proxy-support.md#azure-portal)

###  <a name="enable-mps-on-azure-stack-edge"></a>Activer MPS sur Azure Stack Edge 

1. Exécutez une session Windows PowerShell en tant qu’administrateur. 

2. Assurez-vous que le service Windows Remote Management est en cours d’exécution sur votre client. Dans le terminal PowerShell, utilisez la commande suivante 
    
    ```powershell
    winrm quickconfig
    ```
    
    Si vous voyez des avertissements concernant une exception de pare-feu, vérifiez votre type de connexion réseau et consultez la documentation [Windows Remote Management](/windows/win32/winrm/installation-and-configuration-for-windows-remote-management).

3. Attribuez une variable à l’adresse IP de l’appareil. 
    
    ```powershell
    $ip = "" Replace with the IP address of your device. 
    ```
    
4. Pour ajouter l’adresse IP de l’appareil à la liste des hôtes approuvés du client, utilisez la commande suivante : 
    
    ```powershell
    Set-Item WSMan:\localhost\Client\TrustedHosts $ip -Concatenate -Force 
    ```

5. Démarrez une session Windows PowerShell sur l’appareil. 

    ```powershell
    Enter-PSSession -ComputerName $ip -Credential $ip\EdgeUser -ConfigurationName Minishell 
    ```

6. Indiquez le mot de passe lorsque vous y êtes invité. Utilisez le mot de passe vous permettant de vous connecter à l’interface utilisateur web locale. Le mot de passe par défaut de cette interface est `Password1`.

Tapez `Start-HcsGpuMPS` pour démarrer le service MPS sur l’appareil. 

Pour obtenir de l’aide sur le dépannage de l’appareil Azure Stack Edge, consultez [Résolution des problèmes de l’appareil Azure Stack Edge](spatial-analysis-logging.md#troubleshooting-the-azure-stack-edge-device) 

#### <a name="desktop-machine"></a>[Ordinateur de bureau](#tab/desktop-machine)

Suivez ces instructions si votre ordinateur hôte n’est pas un appareil Azure Stack Edge.

#### <a name="install-nvidia-cuda-toolkit-and-nvidia-graphics-drivers-on-the-host-computer"></a>Installez NVIDIA CUDA Toolkit et les pilotes graphiques Nvidia sur l’ordinateur hôte

Utilisez le script Bash suivant pour installer les pilotes graphiques NVIDIA requis et CUDA Toolkit.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo add-apt-repository "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/ /"
sudo apt-get update
sudo apt-get -y install cuda
```

Redémarrez l’ordinateur et exécutez la commande suivante :

```bash
nvidia-smi
```

Vous devez voir la sortie suivante.

![Sortie du pilote NVIDIA](media/spatial-analysis/nvidia-driver-output.png)

### <a name="install-docker-ce-and-nvidia-docker2-on-the-host-computer"></a>Installez Docker CE et nvidia-docker2 sur l’ordinateur hôte

Installez Docker CE sur l’ordinateur hôte.

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg-agent software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

Installez le package logiciel *nvidia-docker-2*.

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update
sudo apt-get install -y docker-ce nvidia-docker2
sudo systemctl restart docker
```

## <a name="enable-nvidia-mps-on-the-host-computer"></a>Activez NVIDIA MPS sur l’ordinateur hôte

> [!TIP]
> * N’installez pas de pack d’administration si la capacité de calcul de votre GPU est inférieure à 7.x (pre Volta). Pour obtenir des informations de référence, consultez [Compatibilité CUDA](https://docs.nvidia.com/deploy/cuda-compatibility/index.html#support-title). 
> * Exécutez les instructions de MPS à partir d’une fenêtre de terminal sur l’ordinateur hôte. Pas à l’intérieur de votre instance de conteneur Docker.

Pour des performances et une utilisation optimales, configurez le ou les GPU de l’ordinateur hôte pour le [service de multitraitement (MPS) NVIDIA](https://docs.nvidia.com/deploy/pdf/CUDA_Multi_Process_Service_Overview.pdf). Exécutez les instructions de MPS à partir d’une fenêtre de terminal sur l’ordinateur hôte.

```bash
# Set GPU(s) compute mode to EXCLUSIVE_PROCESS
sudo nvidia-smi --compute-mode=EXCLUSIVE_PROCESS

# Cronjob for setting GPU(s) compute mode to EXCLUSIVE_PROCESS on boot
echo "SHELL=/bin/bash" > /tmp/nvidia-mps-cronjob
echo "PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin" >> /tmp/nvidia-mps-cronjob
echo "@reboot   root    nvidia-smi --compute-mode=EXCLUSIVE_PROCESS" >> /tmp/nvidia-mps-cronjob

sudo chown root:root /tmp/nvidia-mps-cronjob
sudo mv /tmp/nvidia-mps-cronjob /etc/cron.d/

# Service entry for automatically starting MPS control daemon
echo "[Unit]" > /tmp/nvidia-mps.service
echo "Description=NVIDIA MPS control service" >> /tmp/nvidia-mps.service
echo "After=cron.service" >> /tmp/nvidia-mps.service
echo "" >> /tmp/nvidia-mps.service
echo "[Service]" >> /tmp/nvidia-mps.service
echo "Restart=on-failure" >> /tmp/nvidia-mps.service
echo "ExecStart=/usr/bin/nvidia-cuda-mps-control -f" >> /tmp/nvidia-mps.service
echo "" >> /tmp/nvidia-mps.service
echo "[Install]" >> /tmp/nvidia-mps.service
echo "WantedBy=multi-user.target" >> /tmp/nvidia-mps.service

sudo chown root:root /tmp/nvidia-mps.service
sudo mv /tmp/nvidia-mps.service /etc/systemd/system/

sudo systemctl --now enable nvidia-mps.service
```

## <a name="configure-azure-iot-edge-on-the-host-computer"></a>Configurer Azure IoT Edge sur l’ordinateur hôte

Pour déployer le conteneur d’analyse spatiale sur l’ordinateur hôte, créez une instance d’un service [Azure IoT Hub](../../iot-hub/iot-hub-create-through-portal.md) à l’aide du niveau tarifaire Standard (S1) ou Gratuit (F0). Si votre ordinateur hôte est un Azure Stack Edge, utilisez les mêmes abonnement et groupe de ressources que la ressource Azure Stack Edge.

Utiliser l’interface de ligne de commande Azure pour créer un service Azure IoT Hub. Remplacez les paramètres le cas échéant. Vous pouvez également créer le service Azure IoT Hub sur le [portail Azure](https://portal.azure.com/).

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
az login
az account set --subscription <name or ID of Azure Subscription>
az group create --name "test-resource-group" --location "WestUS"

az iot hub create --name "test-iot-hub-123" --sku S1 --resource-group "test-resource-group"

az iot hub device-identity create --hub-name "test-iot-hub-123" --device-id "my-edge-device" --edge-enabled
```

Si l’ordinateur hôte n’est pas un appareil Azure Stack Edge, vous devez installer [Azure IoT Edge](../../iot-edge/how-to-install-iot-edge.md) version 1.0.9. Pour télécharger la version correcte, procédez comme suit :

Ubuntu Server 18.04 :
```bash
curl https://packages.microsoft.com/config/ubuntu/18.04/multiarch/prod.list > ./microsoft-prod.list
```

Copiez la liste générée.
```bash
sudo cp ./microsoft-prod.list /etc/apt/sources.list.d/
```

Installez la clé publique Microsoft GPG.

```bash
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo cp ./microsoft.gpg /etc/apt/trusted.gpg.d/
```

Mettez à jour les listes de packages sur votre appareil.

```bash
sudo apt-get update
```

Installez la version 1.0.9 :

```bash
sudo apt-get install iotedge=1.0.9* libiothsm-std=1.0.9*
```

Ensuite, inscrivez l’ordinateur hôte en tant qu’appareil IoT Edge dans votre instance IoT Hub à l’aide d’une [chaîne de connexion](../../iot-edge/how-to-manual-provision-symmetric-key.md?view=iotedge-2018-06).

Vous devez connecter l’appareil IoT Edge à votre service IoT Hub Azure. Vous devez copier la chaîne de connexion à partir de l’appareil IoT Edge que vous avez créé précédemment. Vous pouvez aussi exécuter la commande Azure CLI ci-dessous.

```bash
az iot hub device-identity show-connection-string --device-id my-edge-device --hub-name test-iot-hub-123
```

Sur l’ordinateur hôte, ouvrez `/etc/iotedge/config.yaml` pour le modifier. Remplacez `ADD DEVICE CONNECTION STRING HERE` par la chaîne de connexion. Enregistrez et fermez le fichier. Exécutez cette commande pour redémarrer le service IoT Edge sur l’ordinateur hôte.

```bash
sudo systemctl restart iotedge
```

Déployez le conteneur d’analyse spatiale comme un module IoT sur l’ordinateur hôte à partir du [portail Azure](../../iot-edge/how-to-deploy-modules-portal.md) ou de l’[interface de ligne de commande Azure](../../iot-edge/how-to-deploy-modules-cli.md). Si vous utilisez le portail, définissez l’URI de l’image sur l’emplacement de votre service Azure Container Registry. 

Procédez comme suit pour déployer le conteneur à l’aide de l’interface de ligne de commande Azure.

---

### <a name="iot-deployment-manifest"></a>Manifeste de déploiement IoT

Pour simplifier le déploiement de conteneurs sur plusieurs ordinateurs hôtes, vous pouvez créer un fichier de manifeste de déploiement pour spécifier les options de création de conteneurs et les variables d’environnement. Vous trouverez un exemple de manifeste de déploiement [pour Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179) et [d’autres ordinateurs de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json) sur GitHub.

Le tableau suivant présente les différentes variables d’environnement utilisées par le module IoT Edge. Vous pouvez également les définir dans le manifeste de déploiement lié ci-dessus, à l’aide de l’attribut `env` dans `spatialanalysis` :

| Nom du paramètre | Valeur | Description|
|---------|---------|---------|
| ARCHON_LOG_LEVEL | Info; Verbose | Niveau de journalisation, sélectionnez l’une des deux valeurs suivantes :|
| ARCHON_SHARED_BUFFER_LIMIT | 377487360 | Ne modifiez pas|
| ARCHON_PERF_MARKER| false| Affectez la valeur true à la journalisation des performances ; sinon, la valeur doit être false| 
| ARCHON_NODES_LOG_LEVEL | Info; Verbose | Niveau de journalisation, sélectionnez l’une des deux valeurs suivantes :|
| OMP_WAIT_POLICY | PASSIVE | Ne modifiez pas|
| QT_X11_NO_MITSHM | 1 | Ne modifiez pas|
| API_KEY | votre clé API| Récupérez cette valeur dans le portail Azure à partir de votre ressource Vision par ordinateur. Vous la trouverez dans la section **Clé et point de terminaison** de vos ressources. |
| BILLING_ENDPOINT | votre URI de point de terminaison| Récupérez cette valeur dans le portail Azure à partir de votre ressource Vision par ordinateur. Vous la trouverez dans la section **Clé et point de terminaison** de vos ressources.|
| CLUF | accepter | Cette valeur doit être définie sur *accepter* pour que le conteneur s’exécute |
| DISPLAY | :1 | Cette valeur doit être identique à la sortie de `echo $DISPLAY` sur l’ordinateur hôte. Les appareils Azure Stack Edge n’ont pas d’affichage. Ce paramètre n'est pas applicable|


> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas.  Pour plus d'informations, consultez [Facturation](#billing).

Une fois que vous avez mis à jour le manifeste de déploiement pour des [appareils Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179) ou un [appareil de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json) avec vos propres paramètres et sélection d’opérations, vous pouvez utiliser la commande [Azure CLI](../../iot-edge/how-to-deploy-modules-cli.md) ci-dessous pour déployer le conteneur en tant que module IoT Edge sur l’ordinateur hôte.

```azurecli
az login
az extension add --name azure-iot
az iot edge set-modules --hub-name "<IoT Hub name>" --device-id "<IoT Edge device name>" --content DeploymentManifest.json --subscription "<subscriptionId>"
```

|Paramètre  |Description  |
|---------|---------|
| `--hub-name` | Nom de votre Azure IoT Hub. |
| `--content` | Nom du fichier de déploiement. |
| `--target-condition` | Nom de votre appareil IoT Edge pour l’ordinateur hôte. |
| `-–subscription` | ID ou nom de l’abonnement. |

Cette commande démarre le déploiement. Accédez à la page de votre instance Azure IoT Hub dans le portail Azure pour afficher l’état du déploiement. L’état peut se présenter sous la forme *417 : la configuration de déploiement de l’appareil n’est pas définie* jusqu’à ce que l’appareil termine de télécharger les images du conteneur et démarre l’exécution.

## <a name="validate-that-the-deployment-is-successful"></a>Vérifiez que déploiement est réussi

Il existe plusieurs façons de confirmer que le conteneur s’exécute. Localisez la section *État du runtime* dans les **paramètres du module IoT Edge** pour le module d’analyse spatiale de votre instance Azure IoT Hub sur le portail Azure. Vérifiez que les champs **Valeur souhaitée** et **Valeur signalée** de la section *État du runtime* indiquent *En cours d’exécution*.

![Exemple de vérification du déploiement](./media/spatial-analysis/deployment-verification.png)

Une fois le déploiement terminé et le conteneur en cours d’exécution, l’**ordinateur hôte** commence à envoyer des événements à Azure IoT Hub. Si vous avez utilisé la version `.debug` des opérations, vous verrez une fenêtre de visualiseur pour chaque caméra que vous avez configurée dans le manifeste de déploiement. Vous pouvez maintenant définir les lignes et les zones que vous souhaitez analyser dans le manifeste de déploiement et suivre les instructions pour effectuer un nouveau déploiement. 

## <a name="configure-the-operations-performed-by-spatial-analysis"></a>Configurer les opérations effectuées par l’analyse spatiale

Vous devez utiliser [opérations d’analyse spatiale](spatial-analysis-operations.md) pour configurer le conteneur afin d’utiliser des caméras connectées et de configurer les opérations, entre autres. Pour chaque caméra que vous configurez, les opérations d’analyse spatiale génèrent un flux de sortie de messages JSON envoyés à votre instance Azure IoT Hub.

## <a name="redeploy-or-delete-the-deployment"></a>Redéployer ou supprimer le déploiement

Si vous devez mettre à jour le déploiement, vous devez vous assurer que vos déploiements précédents ont abouti, ou supprimer les déploiements d’appareils IoT Edge qui ne se sont pas terminés. Dans le cas contraire, ces déploiements se poursuivront, laissant le système dans un état incorrect. Vous pouvez utiliser le portail Azure ou l’[interface de ligne de commande Azure](/cli/azure/ext/azure-cli-iot-ext/iot/edge/deployment).

## <a name="use-the-output-generated-by-the-container"></a>Utiliser la sortie générée par le conteneur

Si vous souhaitez commencer à consommer la sortie générée par le conteneur, consultez les articles suivants :

*   Utilisez le kit de développement logiciel (SDK) Azure Event Hub pour le langage de programmation que vous avez choisi pour vous connecter au point de terminaison Azure IoT Hub et recevoir les événements. Pour plus d’informations, consultez [Lire des messages appareil-à-cloud à partir du point de terminaison intégré](../../iot-hub/iot-hub-devguide-messages-read-builtin.md). 
*   Configurez le routage des messages sur votre instance IoT Hub Azure pour envoyer les événements à d’autres points de terminaison ou enregistrer les événements dans votre stockage Blob Azure, etc. Pour plus d’informations, consultez [Routage des messages IoT Hub](../../iot-hub/iot-hub-devguide-messages-d2c.md). 

## <a name="running-spatial-analysis-with-a-recorded-video-file"></a>Exécution de l’analyse spatiale avec un fichier vidéo enregistré

Vous pouvez utiliser l’analyse spatiale avec des vidéos enregistrées ou en direct. Pour utiliser l’analyse spatiale pour une vidéo enregistrée, essayez d’enregistrer un fichier vidéo et de l’enregistrer en tant que fichier MP4. Créez un compte de stockage d’objets blob existant ou créez-en un nouveau. Ensuite, mettez à jour les paramètres de stockage d’objets blob suivants dans le Portail Azure :
    1. Modifiez **Transfert sécurisé requis** en **Désactivé**
    2. Modifiez **Autoriser l’accès public aux objets blob** en **Activé**

Accédez à la section **Conteneur**, puis créez un nouveau conteneur ou utilisez un conteneur existant. Téléchargez ensuite le fichier vidéo dans le conteneur. Développez les paramètres de fichier pour le fichier chargé, puis sélectionnez **Générer une SAS**. Veillez à définir une **Date d’expiration** suffisamment longue pour couvrir la période de tests. Définissez les **protocoles autorisés** sur *HTTP* (*HTTPS* n’est pas pris en charge).

Cliquez sur **Générer un jeton SAS et une URL** et copiez l’URL SAS de l’objet blob. Remplacez le `https` de départ par `http` et testez l’URL dans un navigateur qui prend en charge la lecture vidéo.

Remplacez `VIDEO_URL` dans le manifeste de déploiement pour votre [appareil Azure Stack Edge](https://go.microsoft.com/fwlink/?linkid=2142179) ou un autre [ordinateur de bureau](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/ComputerVision/spatial-analysis/DeploymentManifest_for_non_ASE_devices.json) par l’URL que vous avez créée, pour tous les graphiques. Définissez `VIDEO_IS_LIVE` sur `false`et redéployez le conteneur d’analyse spatiale avec le manifeste mis à jour. Reportez-vous à l’exemple ci-dessous.

Le module d’analyse spatiale va commencer à consommer un fichier vidéo et le relira automatiquement en permanence.


```json
"zonecrossing": {
  "operationId" : "cognitiveservices.vision.spatialanalysis-personcrossingpolygon",
  "version": 1,
  "enabled": true,
  "parameters": {
      "VIDEO_URL": "Replace http url here",
      "VIDEO_SOURCE_ID": "personcountgraph",
      "VIDEO_IS_LIVE": false,
        "VIDEO_DECODE_GPU_INDEX": 0,
      "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0 }",
      "SPACEANALYTICS_CONFIG": "{\"zones\":[{\"name\":\"queue\",\"polygon\":[[0.3,0.3],[0.3,0.9],[0.6,0.9],[0.6,0.3],[0.3,0.3]], \"threshold\":35.0}]}"
    }
  },

```

## <a name="troubleshooting"></a>Dépannage

Si vous rencontrez des problèmes lors du démarrage ou de l’exécution du conteneur, consultez [Télémétrie et résolution des problèmes](spatial-analysis-logging.md) pour connaître les étapes de problèmes courants. Cet article contient également des informations sur la génération et la collecte de journaux et sur la collecte de l’intégrité du système.

## <a name="billing"></a>Facturation

Le conteneur d’analyse spatiale envoie des informations de facturation à Azure à l’aide d’une ressource Vision par ordinateur sur votre compte Azure. L’utilisation de l’analyse spatiale en préversion publique est actuellement gratuite. 

Les conteneurs Azure Cognitive Services ne sont pas concédés sous licence dans le but de s’exécuter sans être connectés au point de terminaison de mesure/facturation. Vous devez configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au point de terminaison de facturation à tout moment. Les conteneurs Cognitive Services n’envoient pas de données client, telles que la vidéo ou l’image analysées, à Microsoft.


## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert des concepts et le flux de travail pour le téléchargement, l’installation et l’exécution du conteneur d’analyse spatiale. En résumé :

* L’analyse spatiale est un conteneur Linux pour Docker.
* Les images conteneurs sont téléchargées à partir de Microsoft Container Registry.
* Les images de conteneur s’exécutent en tant que modules IoT dans Azure IoT Edge.
* Comment configurer le conteneur et le déployer sur un ordinateur hôte.

## <a name="next-steps"></a>Étapes suivantes

* [Déployer une application web de comptage de personnes](spatial-analysis-web-app.md)
* [Configurer les opérations d’analyse spatiale](spatial-analysis-operations.md)
* [Journalisation et résolution des problèmes](spatial-analysis-logging.md)
* [Guide de positionnement de la caméra](spatial-analysis-camera-placement.md)
* [Guide de positionnement de zone et de ligne](spatial-analysis-zone-line-placement.md)
