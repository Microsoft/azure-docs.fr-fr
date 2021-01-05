---
title: Déployer Live Video Analytics sur Azure Stack Edge
description: Cet article répertorie les étapes permettant de déployer Live Video Analytics sur Azure Stack Edge.
ms.topic: how-to
ms.date: 09/09/2020
ms.openlocfilehash: f33b6fb0f0dc5c5b733a0fcb021e2792ce9c6ec6
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92019594"
---
# <a name="deploy-live-video-analytics-on-azure-stack-edge"></a>Déployer Live Video Analytics sur Azure Stack Edge

Cet article répertorie les étapes permettant de déployer Live Video Analytics sur Azure Stack Edge. Une fois l’appareil configuré et activé, il est prêt pour le déploiement de Live Video Analytics. 

Pour Live Video Analytics, nous allons procéder à un déploiement via IoT Hub, bien que les ressources Azure Stack Edge exposent une API Kubernetes, qui permet au client de déployer des solutions supplémentaires non IoT Hub compatibles qui peuvent interagir avec le service Live Video Analytics. 

> [!TIP]
> L’utilisation de l’API Kubernetes (K8s) pour le déploiement personnalisé est un cas particulier. Nous recommandons au client de créer des modules Edge et procéder au déploiement via IoT Hub vers chaque ressource Azure Stack Edge au lieu d’utiliser l’API Kubernetes. Dans cet article, nous allons vous montrer les étapes de déploiement du module Live Video Analytics à l’aide de IoT Hub.

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure dans lequel vous disposez des [privilèges de propriétaire](../../role-based-access-control/built-in-roles.md#owner).
* Une ressource [Azure Stack Edge](../../databox-online/azure-stack-edge-gpu-deploy-prep.md)
   
* [IoT Hub](../../iot-hub/iot-hub-create-through-portal.md)
* Un [principal de service](./create-custom-azure-resource-manager-role-how-to.md#create-service-principal) pour le module Live Video Analytics.

   Utilisez l’une des régions où IoT Hub est disponible : USA Est 2, USA Centre, USA Centre Nord, Japon Est, USA Ouest 2, USA Centre-Ouest, Canada Est, Royaume-Uni Sud, France Centre, France Sud, Suisse Nord, Suisse Ouest et Japon Ouest.
* Compte de stockage

    Nous vous recommandons d’utiliser des comptes de stockage à usage général v2 (GPv2).  
    En savoir plus sur le [compte de stockage v2 à usage général](../../storage/common/storage-account-upgrade.md?tabs=azure-portal).
* [Visual Studio Code](https://code.visualstudio.com/) sur votre machine de développement. Vérifiez que vous disposez de l’[extension Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).
* Vérifiez que le réseau auquel votre machine de développement est connectée autorise le protocole AMQP (Advanced Message Queueing Protocol) sur le port 5671. Cette configuration permet à Azure IoT Tools de communiquer avec Azure IoT Hub.

## <a name="configuring-azure-stack-edge-for-using-live-video-analytics"></a>Configuration d’Azure Stack Edge pour utiliser Live Video Analytics

Azure Stack Edge est une solution matérielle en tant que service ainsi qu’un appareil de computing en périphérie basé sur l’intelligence artificielle. Il est doté de fonctionnalités de transfert de données via le réseau. Pour en savoir plus, consultez [Azure Stack Edge et instructions d’installation détaillées](../../databox-online/azure-stack-edge-deploy-prep.md). Pour commencer, suivez les instructions des liens ci-dessous :

* [Création de ressources Service Azure Stack Edge/Data Box Gateway](../../databox-online/azure-stack-edge-deploy-prep.md)
* [Installation et configuration](../../databox-online/azure-stack-edge-deploy-install.md)
* [Connexion et activation](../../databox-online/azure-stack-edge-deploy-connect-setup-activate.md)

### <a name="attach-an-iot-hub-to-azure-stack-edge"></a>Attacher IoT Hub à Azure Stack Edge

1. Sur le [portail Azure](https://ms.portal.azure.com), accédez à votre ressource Azure Stack Edge, puis cliquez sur Vue d’ensemble. Dans le volet droit, sur la vignette Calculer, sélectionnez Bien démarrer.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/azure-stack-edge.png" alt-text="Azure Stack Edge":::
1. Sur la vignette Configurer le computing en périphérie, sélectionnez Configurer le calcul.
1. Sur le panneau Configurer le computing en périphérie, entrez ce qui suit :
    
    | Champ|Valeur|
    |---|---|
    |IoT Hub|Choisissez Nouveau ou Existant.<br/>Par défaut, un niveau Standard (S1) est utilisé pour créer une ressource IoT. Pour utiliser une ressource IoT de niveau gratuit, créez-en une, puis sélectionnez-la.<br/>Dans chaque cas, la ressource IoT Hub utilise les mêmes abonnement et groupe de ressources que la ressource Azure Stack Edge.|
    |Nom|Entrez un nom pour votre ressource IoT Hub.|

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/azure-stack-edge-get-started.png" alt-text="Azure Stack Edge":::
1. Sélectionnez **Create** (Créer). La création de ressources IoT Hub prend quelques minutes. Une fois la ressource IoT Hub créée, la vignette **Configurer le calcul** est mise à jour pour afficher la configuration du calcul. Pour vérifier que le rôle de computing en périphérie a été configuré, sélectionnez **Voir le computing** sur la vignette **Configurer le calcul**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/edge-compute-config.png" alt-text="Azure Stack Edge":::

    > [!NOTE]
    > Si la boîte de dialogue Configurer le calcul est fermée avant que l’IoT Hub soit associé à la ressource Azure Stack Edge, une ressource loT Hub est créée mais n’est pas affichée dans la configuration de calcul. Attendez quelques minutes pour recharger la page. La ressource s’affiche.
    
    Quand le rôle de calcul Edge est configuré sur l’appareil Edge, il crée deux appareils : un appareil IoT et un appareil IoT Edge. Ces deux appareils peuvent être visualisés dans la ressource IoT Hub. Un runtime IoT Edge est également exécuté sur l’appareil IoT Edge. À ce stade, seule la plateforme Linux est disponible pour votre appareil IoT Edge.
    
    Une fois toutes les informations remplies, la carte de computing en périphérie s’affiche :
    
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/configure-edge-compute.png" alt-text="Azure Stack Edge":::
 
### <a name="enable-compute-prerequisites-on-the-azure-stack-edge-local-ui"></a>Activer les conditions préalables de calcul sur l’interface utilisateur locale Azure Stack Edge

Avant de poursuivre, assurez-vous que :

* vous avez activé votre ressource Azure Stack Edge ;
* vous avez accès à un système client Windows exécutant PowerShell 5.0 ou une version ultérieure pour accéder à la ressource Azure Stack Edge.
* Pour déployer un cluster Kubernetes, vous devez configurer votre ressource Azure Stack Edge via son [interface utilisateur locale](../../databox-online/azure-stack-edge-deploy-connect-setup-activate.md#connect-to-the-local-web-ui-setup). 
    
    * Pour activer le calcul, accédez à la page Calcul de l’interface utilisateur locale de votre appareil.
    
        * Sélectionnez une interface réseau que vous souhaitez activer pour le calcul. Sélectionnez Activer. L’activation du calcul entraîne la création d’un commutateur virtuel sur votre appareil sur cette interface réseau.
        * Laissez les adresses IP du nœud de test Kubernetes et les adresses IP des services externes Kubernetes vides.
        * Sélectionnez Appliquer. Cette opération doit prendre environ 2 minutes.
        
        > [!div class="mx-imgBorder"]
        > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/azure-stack-edge-commercial.png" alt-text="Azure Stack Edge":::

        * Si DNS n’est pas configuré pour l’API Kubernetes et la ressource Azure Stack Edge, vous pouvez mettre à jour votre fichier hôte Windows.
        
            * Ouvrez un éditeur de texte en tant qu’administrateur
            * Ouvrez le fichier « to C:\Windows\System32\drivers\etc\hosts' »
            * Ajoutez le nom d’hôte et le nom du périphérique IPv4 de l’API Kubernetes au fichier. (Vous pouvez retrouver ces informations dans le portail Azure Stack Edge sous la section Périphériques.)
            * Enregistrer et fermer

### <a name="deploy-live-video-analytics-edge-module-using-azure-portal"></a>Déployer un module Live Video Analytics Edge à l’aide du portail Azure

Pour ce faire, nous allons effectuer uniquement certaines étapes du tutoriel [Déployer Live Video Analytics via IoT Hub](deploy-iot-edge-device.md).

1. Ignorez les étapes de création d’utilisateurs et de groupes et passez au module [déployer Live Video Analytics Edge](deploy-iot-edge-device.md#deploy-live-video-analytics-edge-module). Suivez les étapes qui s’y trouvent.
1. Dans les options de création de conteneurs, vous n’avez pas besoin de définir de variables d’environnement. Ignorez simplement cette étape.
1. Ouvrez l’onglet Options de création de conteneur.

   * Copiez et collez le JSON suivant dans la zone pour limiter la taille des fichiers journaux générés par le module.
    
      ```json
        "HostConfig": {
                    "LogConfig": {
                        "Type": "",
                        "Config": {
                        "max-size": "10m",
                        "max-file": "10"
                        }
                    },
                    "Binds": [
                        "/var/media/:/var/media/",
                        "/var/lib/azuremediaservices:/var/lib/azuremediaservices"
                    ]
                    }
      ```
    
      > [!NOTE]
      > Si vous utilisez le protocole gRPC avec le transfert de mémoire partagé, utilisez le mode IPC hôte pour accéder à la mémoire partagée entre Live Video Analytics et les solutions d’inférence.
   
      ```json
          "HostConfig": {
                        "LogConfig": {
                            "Type": "",
                            "Config": {
                            "max-size": "10m",
                            "max-file": "10"`
                            }
                        },
                        "Binds": [
                            "/var/media/:/var/media/",
                            "/var/lib/azuremediaservices:/var/lib/azuremediaservices"
                        ],
                        "IpcMode": "host",
                        "ShmSize": 1536870912
                    }
      ```

      > [!NOTE]
      > La section « Lier » du code JSON comporte 2 entrées. N’hésitez pas à mettre à jour les liaisons de périphérique, pensez bien à vous assurez que ces répertoires existent.
    
    * « /var/lib/azuremediaservices:/var/lib/azuremediaservices » : Celle-là permet de lier les données de configuration d’application persistantes du conteneur et de les stocker sur le périphérique.
    * « /var/media:/var/media » : Celle-ci lie les dossiers multimédias entre le périphérique et le conteneur. Elle est utilisée pour stocker les enregistrements vidéo lorsque vous exécutez une topologie de graphique média qui prend en charge le stockage des clips vidéo sur le périphérique.
        
1. Poursuivez les étapes répertoriées dans le document et renseignez les paramètres du jumeau de module.
1. Sélectionnez **Suivant** : Itinéraires pour passer à la section Itinéraires. Spécifiez des itinéraires.

    Conservez les itinéraires par défaut et sélectionnez Suivant : Vérifier + créer pour passer à la section Vérifier.
1. [Examiner et vérifier votre déploiement](deploy-iot-edge-device.md#review-deployment)

#### <a name="optional-setup-docker-volume-mounts"></a>(Facultatif) Configurer les montages de volume Docker

Si vous souhaitez afficher les données dans les répertoires de travail, procédez comme suit pour configurer les montages de volume Docker avant le déploiement. 

Ces étapes décrivent la création d’un utilisateur de passerelle et la configuration de partages de fichiers pour afficher le contenu du répertoire de travail Live Video Analytics et du dossier du média Live Video Analytics.

> [!NOTE]
> Les montages de liaisons sont pris en charge, mais les montages de volume permettent d’afficher les données et, le cas échéant, de les copier à distance. Il est possible d’utiliser à la fois des montages de liaison et de volume, mais ils ne peuvent pas pointer vers le même chemin d’accès de conteneur.

1. Ouvre le portail Azure et accédez à votre ressource Azure Stack Edge.
1. Créer un **utilisateur de passerelle** qui peut accéder aux partages.
    
    1. Dans le volet de navigation gauche, cliquez sur **Passerelle->Utilisateurs**.
    1. Cliquez sur **+ ajouter un utilisateur** pour définir le nom d’utilisateur et le mot de passe. (Recommandé : `lvauser`).
    1. Cliquez sur **Ajouter**.
    
1. Créer un **partage local** pour la persistance de Live Video Analytics.

    1. Cliquez sur **Passerelle->Partages**.
    1. Cliquez sur **+ Ajouter des partages**.
    1. Entrez un nom de partage. (Recommandé : `lva`).
    1. Conservez le type de partage SMB.
    1. Assurez-vous que l’option **utiliser le partage avec le computing en périphérie** est activée.
    1. Assurez-vous que l’option **Configurer en tant que partage local Edge** est activée.
    1. Dans détails de l’utilisateur, autorisez l’accès au partage à l’utilisateur créé récemment.
    1. Cliquez sur **Créer**.
        
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/local-share.png" alt-text="Azure Stack Edge":::
    
1. Créer un partage distant pour le stockage File Sync.

    1. Commencez par créer un compte de stockage Blob dans la même région.
    1. Cliquez sur **Passerelle->Partages**.
    1. Cliquez sur **+ Ajouter des partages**.
    1. Entrez un nom de partage. (Recommandé : média).
    1. Conservez le type de partage SMB.
    1. Assurez-vous que l’option **utiliser le partage avec le computing en périphérie** est activée.
    1. Assurez-vous que l’option **Configurer en tant que partage local Edge** n’est pas activée.
    1. Sélectionnez le compte de stockage récemment créé.
    1. Définissez un nom de conteneur.
    1. Définissez le type de stockage sur l’objet blob de blocs.
    1. Dans détails de l’utilisateur, autorisez l’accès au partage à l’utilisateur créé récemment.
    1. Cliquez sur **Créer**.    
    
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/remote-share.png" alt-text="Azure Stack Edge"
            }]
        }
    }
    ```

### <a name="verify-that-the-module-is-running"></a>Vérifiez que le module soit en cours d’exécution

L’étape finale consiste à vérifier que le module est connecté et qu’il s’exécute comme prévu. L’état du runtime du module doit être en cours d’exécution pour votre appareil IoT Edge dans la ressource IoT Hub.

Pour vérifier que le module est en cours d’exécution, effectuez les étapes suivantes :

1. Dans le portail Azure, revenez à votre ressource Azure Stack Edge
1. Sélectionnez la vignette « Modules ». Le panneau Modules apparaît. Dans la liste des modules, identifiez le module que vous avez déployé. L’état du runtime du module que vous avez ajouté doit être en cours d’exécution.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/deploy-azure-stack-edge-how-to/iot-edge-custom-module.png" alt-text="Azure Stack Edge":::

### <a name="configure-the-azure-iot-tools-extension"></a>Configurer l’extension Azure IoT Tools

Suivez ces instructions pour vous connecter à votre hub IoT à l’aide de l’extension Azure IoT Tools.

1. Dans Visual Studio Code, sélectionnez Affichage > Explorer. Vous pouvez aussi sélectionner Ctrl+Maj+E.
1. Dans l’angle en bas à gauche de l’onglet Explorateur, sélectionnez Azure IoT Hub.
1. Sélectionnez l’icône Plus d’options pour voir le menu contextuel. Sélectionnez ensuite Définir la chaîne de connexion IoT Hub.
1. Quand une zone d’entrée s’affiche, entrez votre chaîne de connexion IoT Hub. 

   * Pour obtenir la chaîne de connexion, accédez à votre IoT Hub sur le portail Azure puis cliquez sur Stratégies d’accès partagé dans le volet de navigation de gauche.
   * Cliquez sur iothubowner pour obtenir les clés d’accès partagé.
   * Copiez la clé primaire de la chaîne de connexion, puis collez-la dans la zone d’entrée du VSCode.

   La chaîne de connexion doit ressembler à :<br/>`HostName=xxx.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=xxx`
    
   Si la connexion aboutit, la liste des appareils de périphérie s’affiche. Votre appareil Azure Stack Edge doit s’afficher. Vous pouvez désormais gérer vos appareils IoT Edge et interagir avec Azure IoT Hub par le biais du menu contextuel. Pour afficher les modules déployés sur le périphérique, sous l’appareil Azure Stack, développez le nœud Modules.
    
## <a name="troubleshooting"></a>Dépannage

* Accès à l’API Kubernetes (kubectl).

    * Suivez la documentation pour configurer votre ordinateur afin d’avoir [accès au cluster Kubernetes](https://review.docs.microsoft.com/azure/databox-online/azure-stack-edge-j-series-create-kubernetes-cluster?toc=%2Fazure%2Fdatabox-online%2Fazure-stack-edge-gpu%2Ftoc.json&bc=%2Fazure%2Fdatabox-online%2Fazure-stack-edge-gpu%2Fbreadcrumb%2Ftoc.json&branch=release-tzl#debug-kubernetes-issues).
    * Tous les modules IoT Edge déployés utilisent l’espace de noms `iotedge`. Assurez-vous de l’inclure lorsque vous utiliser kubectl.
* Journaux de module

    L’outil `iotedge` n’est pas accessible pour obtenir des journaux. Vous devez utiliser [journaux kubectl](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs) pour afficher les journaux ou le canal dans un fichier. Exemple : <br/>  `kubectl logs deployments/mediaedge -n iotedge --all-containers`
* Métriques nœud et pod

    Utilisez [kubectl top](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#top) pour voir les métriques nœud et pod. (Cette fonctionnalité sera disponible dans la prochaine version d’Azure Stack Edge. >v2007)<br/>`kubectl top pods -n iotedge`
* Concernant la mise en réseau des modules pour la découverte de module sur Azure Stack Edge, il est nécessaire que le module dispose de la liaison avec le port hôte dans createOptions. Le module sera ensuite adressable sur `moduleName:hostport`.
    
    ```json
    "createOptions": {
        "HostConfig": {
            "PortBindings": {
                "8554/tcp": [ { "HostPort": "8554" } ]
            }
        }
    }
    ```
    
* Montages de volumes

    Un module ne peut pas démarrer si le conteneur tente de monter un volume dans un répertoire existant et qui n’est pas vide.
* Mémoire partagée

    La fonction de mémoire partagée sur les ressources Azure Stack Edge est prise en charge sur les pods dans tous les espaces de noms en utilisant l’hôte IPC.
    Configuration de la mémoire partagée sur un module Edge pour le déploiement via IoT Hub.
    
    ```
    ...
    "createOptions": {
        "HostConfig": {
            "IpcMode": "host"
        }
    ...
        
    (Advanced) Configuring shared memory on a K8s Pod or Deployment manifest for deployment via K8s API.
    spec:
        ...
        template:
        spec:
            hostIPC: true
        ...
    ```
    
* (Avancé) Colocalisation de pods

    Quand vous utilisez K8s pour déployer des solutions d’inférence personnalisées qui communiquent avec Live Video Analytics via gRPC, vous devez vous assurer que les pods sont déployés sur les mêmes nœuds que les modules Live Video Analytics.

    * Option 1 : utiliser l’affinité de nœud et les étiquettes de nœud intégrées pour la colocalisation.

    Actuellement, la configuration personnalisée NodeSelector ne semble pas être possible, car les utilisateurs ne disposent pas des accès nécessaires pour définir des étiquettes sur les nœuds. Toutefois, en fonction de la topologie du client et des conventions d’affectation de noms, il peut être en mesure d’utiliser des [étiquettes de nœud intégrées](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#built-in-node-labels). La section nodeAffinity référençant des ressources Azure Stack Edge avec Live Video Analytics peut être ajoutée au manifeste de pod d’inférence afin de réussir la colocalisation.
    * Option 2 : utiliser l’affinité de pod pour la colocalisation (recommandé).
Kubernetes prend en charge [l’affinité de pod](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity) qui peut planifier des pods sur le même nœud. La section podAffinity référençant les modules Live Video Analytics peut être ajoutée au manifeste de pod d’inférence afin de réussir la colocalisation.

    ```json   
    // Example Live Video Analytics module deployment match labels
    selector:
      matchLabels:
        net.azure-devices.edge.deviceid: dev-ase-1-edge
        net.azure-devices.edge.module: mediaedge
    
    // Example Inference deployment manifest pod affinity
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: net.azure-devices.edge.module
                operator: In
                values:
                - mediaedge
            topologyKey: "kubernetes.io/hostname"
    ```

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez utiliser le module pour analyser des flux vidéo en direct en invoquant des méthodes directes. [Appelez les méthodes directes](get-started-detect-motion-emit-events-quickstart.md#use-direct-method-calls) sur le module.