---
title: 'Tutoriel : Créer une hiérarchie d’appareils IoT Edge - Azure IoT Edge'
description: Ce tutoriel montre comment créer une structure hiérarchique d’appareils IoT Edge à l’aide de passerelles.
author: v-tcassi
manager: philmea
ms.author: v-tcassi
ms.date: 11/10/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
monikerRange: '>=iotedge-2020-11'
ms.openlocfilehash: c1dba383f259e35b143688b2db68f05f1a67def6
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938194"
---
# <a name="tutorial-create-a-hierarchy-of-iot-edge-devices-preview"></a>Tutoriel : Créer une hiérarchie d’appareils IoT Edge (préversion)

Déployez des nœuds Azure IoT Edge sur des réseaux organisés en couches hiérarchiques. Chaque couche d’une hiérarchie est un appareil de passerelle qui gère les messages et les requêtes des appareils situés dans la couche du dessous.

>[!NOTE]
>Cette fonctionnalité nécessite IoT Edge version 1.2, qui est en préversion publique, exécutant des conteneurs Linux.

Vous pouvez structurer une hiérarchie d’appareils de sorte que seule la couche supérieure soit connectée au cloud, et que les couches inférieures puissent uniquement communiquer avec les couches Nord et Sud adjacentes. Cette structure en couches réseau est la base de la plupart des réseaux industriels, qui respectent la [norme ISA-95](https://en.wikipedia.org/wiki/ANSI/ISA-95).

L’objectif de ce tutoriel est de créer une hiérarchie d’appareils IoT Edge qui simule un environnement de production. À la fin, vous déploierez le [module Simulated Temperature Sensor](https://azuremarketplace.microsoft.com/marketplace/apps/azure-iot.simulated-temperature-sensor) sur un appareil de couche inférieure sans accès à Internet en téléchargeant des images conteneur à travers la hiérarchie.

Pour atteindre cet objectif, ce tutoriel vous guide dans la création d’une hiérarchie d’appareils IoT Edge, le déploiement de conteneurs de runtime IoT Edge sur vos appareils et la configuration locale de vos appareils. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Créer et définir les relations dans une hiérarchie d’appareils IoT Edge.
> * Configurer le runtime IoT Edge sur les appareils de votre hiérarchie.
> * Installer des certificats cohérents dans votre hiérarchie d’appareils.
> * Ajouter des charges de travail aux appareils de votre hiérarchie.
> * Utiliser un module de proxy d’API pour router de manière sécurisée le trafic HTTP sur un port unique à partir de vos appareils de couche inférieure.

Dans ce tutoriel, les couches réseau suivantes sont définies :

* **Couche supérieure** : les appareils IoT Edge de cette couche peuvent se connecter directement au cloud.

* **Couche inférieure** : les appareils IoT Edge de cette couche ne peuvent pas se connecter directement au cloud. Ils doivent passer par un ou plusieurs appareils IoT Edge intermédiaires pour envoyer et recevoir des données.

Ce tutoriel utilise une hiérarchie à deux appareils pour des raisons de simplicité. Un appareil, **topLayerDevice**, représente un appareil de la couche supérieure de la hiérarchie, qui peut se connecter directement au cloud. Cet appareil sera également appelé **appareil parent**. L’autre appareil, **lowerLayerDevice**, représente un appareil de la couche inférieure de la hiérarchie, qui ne peut pas se connecter directement au cloud. Cet appareil sera également appelé **appareil enfant**. Vous pouvez ajouter des appareils de couche inférieure supplémentaires pour représenter votre environnement de production. La configuration de tous les appareils de couche inférieure supplémentaires suivra la configuration de **lowerLayerDevice**.

## <a name="prerequisites"></a>Prérequis

Pour créer une hiérarchie d’appareils IoT Edge, vous aurez besoin des éléments suivants :

* Un ordinateur (Windows ou Linux) avec une connectivité Internet
* Un compte Azure avec un abonnement valide. Si vous n’avez pas [d’abonnement Azure](../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) de niveau gratuit ou standard dans Azure
* Azure CLI v2.3.1 avec l’extension Azure IoT v0.10.6 ou une version ultérieure installée. Ce tutoriel utilise [Azure Cloud Shell](../cloud-shell/overview.md). Si vous ne connaissez pas Azure Cloud Shell, [consultez ce guide de démarrage rapide pour plus d’informations](./quickstart-linux.md#prerequisites).
* Deux appareils Linux à configurer en tant qu’appareils IoT Edge. Si vous n’avez pas d’appareils disponibles, vous pouvez créer deux machines virtuelles Azure en remplaçant le texte de l’espace réservé dans la commande suivante, que vous exécuterez deux fois :

   ```azurecli-interactive
   az vm create \
    --resource-group <REPLACE_WITH_RESOURCE_GROUP> \
    --name <REPLACE_WITH_UNIQUE_NAMES_FOR_EACH_VM> \
    --image UbuntuLTS \
    --admin-username azureuser \
    --admin-password <REPLACE_WITH_PASSWORD>
   ```

Vous pouvez également tester ce scénario en suivant l’[exemple Azure IoT Edge for Industrial IoT](https://aka.ms/iotedge-nested-sample), qui déploie des machines virtuelles Azure en tant qu’appareils préconfigurés pour simuler un environnement d’usine.

## <a name="configure-your-iot-edge-device-hierarchy"></a>Configurer votre hiérarchie d’appareils IoT Edge

### <a name="create-a-hierarchy-of-iot-edge-devices"></a>Créer une hiérarchie d’appareils IoT Edge

La première étape, la création de vos appareils IoT Edge, peut être effectuée par le biais du portail Azure ou d’Azure CLI. Ce tutoriel crée une hiérarchie de deux appareils IoT Edge : **topLayerDevice** et son enfant **lowerLayerDevice**.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://ms.portal.azure.com/), accédez à votre hub IoT.

1. Dans le menu du volet gauche, sous **Gestion automatique des appareils**, sélectionnez **IoT Edge**.

1. Sélectionnez **+ Ajouter un appareil IoT Edge**. Cet appareil sera l’appareil de la couche supérieure. Entrez donc un ID d’appareil unique approprié. Sélectionnez **Enregistrer**.

1. Sélectionnez **+ Ajouter un appareil IoT Edge** à nouveau. Cet appareil sera l’appareil de périphérie de couche inférieure. Entrez donc un ID d’appareil unique approprié.

1. Choisissez **Définir un appareil parent**, choisissez votre appareil de couche supérieure dans la liste des appareils, puis sélectionnez **OK**. Sélectionnez **Enregistrer**.

   ![Définition du parent de l’appareil de couche inférieure](./media/tutorial-nested-iot-edge/set-parent-device.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Dans [Azure Cloud Shell](https://shell.azure.com/), entrez la commande suivante pour créer un appareil IoT Edge dans votre hub. Cet appareil sera l’appareil de la couche supérieure. Entrez donc un ID d’appareil unique approprié :

   ```azurecli-interactive
   az iot hub device-identity create --device-id {top_layer_device_id} --edge-enabled --hub-name {hub_name}
   ```

1. Entrez la commande suivante pour créer votre appareil IoT Edge enfant et créer la relation parent-enfant entre les appareils :

    ```azurecli-interactive
    az iot hub device-identity create --device-id {lower_layer_device_id} --edge-enabled --pd {top_layer_device_id} --hub-name {iothub_name}
    ```

---

Prenez note de la chaîne de connexion de chaque appareil IoT Edge. Vous les utiliserez ultérieurement.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://ms.portal.azure.com/), accédez à la section **IoT Edge** de votre hub IoT.

1. Cliquez sur l’ID de l’un des appareils dans la liste des appareils.

1. Sélectionnez **Copier** dans le champ **Chaîne de connexion principale** et enregistrez-la à l’emplacement de votre choix.

1. Répétez cette opération pour tous les autres appareils.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Dans [Azure Cloud Shell](https://shell.azure.com/), pour chaque appareil, entrez la commande suivante afin de récupérer la chaîne de connexion de votre appareil, et enregistrez-la à l’emplacement de votre choix :

   ```azurecli-interactive
   az iot hub device-identity connection-string show --device-id {device_id} --hub-name {hub_name}
   ```

---

### <a name="create-certificates"></a>Créer des certificats

Tous les appareils d’un [scénario de passerelle](iot-edge-as-gateway.md) ont besoin d’un certificat partagé afin que vous puissiez configurer des connexions sécurisées entre eux. Effectuez les étapes suivantes pour créer des certificats de démonstration pour les deux appareils de ce scénario.

Pour créer des certificats de démonstration sur un appareil Linux, vous devez cloner les scripts de génération et les configurer pour qu’ils s’exécutent localement dans bash.

1. Clonez le référentiel Git IoT Edge, qui contient des scripts pour générer des certificats de démonstration.

   ```bash
   git clone https://github.com/Azure/iotedge.git
   ```

1. Accédez au répertoire dans lequel vous souhaitez travailler. Tous les fichiers de clés et de certificats seront créés dans ce répertoire.

1. Copiez les fichiers de configuration et de script du dépôt IoT Edge cloné dans votre répertoire de travail.

   ```bash
   cp <path>/iotedge/tools/CACertificates/*.cnf .
   cp <path>/iotedge/tools/CACertificates/certGen.sh .
   ```

1. Créez le certificat d’autorité de certification racine et un certificat intermédiaire.

   ```bash
   ./certGen.sh create_root_and_intermediate
   ```

   Cette commande de script crée plusieurs fichiers de certificat et de clé, mais nous utilisons le fichier suivant comme **certificat d’autorité de certification racine**, pour la hiérarchie de passerelle :

   * `<WRKDIR>/certs/azure-iot-test-only.root.ca.cert.pem`  

1. Créez deux jeux de clés privées et de certificats d’autorité de certification d’appareils IoT Edge avec la commande suivante : un jeu pour l’appareil de couche supérieure et l’autre pour l’appareil de couche inférieure. Fournissez des noms explicites pour les certificats d’autorité de certification, afin de les distinguer l’un de l’autre.

   ```bash
   ./certGen.sh create_edge_device_ca_certificate "top-layer-device"
   ./certGen.sh create_edge_device_ca_certificate "lower-layer-device"
   ```

   Cette commande de script crée plusieurs fichiers de certificat et de clé, mais nous utilisons le certificat et la paire de clés suivants sur chaque appareil IoT Edge et à des fins de référence dans le fichier config.yaml :

   * `<WRKDIR>/certs/iot-edge-device-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>/private/iot-edge-device-<CA cert name>.key.pem`

Chaque appareil a besoin d’une copie du certificat d’autorité de certification racine et d’une copie de son propre certificat d’autorité de certification d’appareil et de sa clé privée. Vous pouvez utiliser un lecteur USB ou une [copie sécurisée des fichiers](https://www.ssh.com/ssh/scp/) pour déplacer les certificats sur chaque appareil.

1. Une fois les certificats transférés, installez l’autorité de certification racine pour chaque appareil.

   ```bash
   sudo cp <path>/azure-iot-test-only.root.ca.cert.pem /usr/local/share/ca-certificates/azure-iot-test-only.root.ca.cert.pem.crt
   sudo update-ca-certificates
   ```

   Cette commande doit générer un certificat qui a été ajouté dans /etc/ssl/certs.

### <a name="install-iot-edge-on-the-devices"></a>Installer IoT Edge sur les appareils

Installez IoT Edge en suivant ces étapes sur les deux appareils.

1. Mettez à jour les listes de packages sur votre appareil.

   ```bash
   sudo apt-get update
   ```

1. Installez le moteur Moby.

   ```bash
   sudo apt-get install moby-engine
   ```

1. Installez le hsmlib et le démon IoT Edge. Pour voir les ressources propres à d’autres distributions Linux, [consultez la version GitHub](https://github.com/Azure/azure-iotedge/releases/tag/1.2.0-rc1). <!-- Update with proper image links on release -->

   ```bash
   curl -L https://github.com/Azure/azure-iotedge/releases/download/1.2.0-rc1/libiothsm-std_1.2.0.rc1-1-1_debian9_amd64.deb -o libiothsm-std.deb
   curl -L https://github.com/Azure/azure-iotedge/releases/download/1.2.0-rc1/iotedge_1.2.0_rc1-1_debian9_amd64.deb -o iotedge.deb
   sudo dpkg -i ./libiothsm-std.deb
   sudo dpkg -i ./iotedge.deb
   ```

### <a name="configure-the-iot-edge-runtime"></a>Configurer le runtime IoT Edge

Configurez le runtime IoT Edge en suivant ces étapes sur vos appareils. La configuration du runtime IoT Edge pour vos appareils nécessite quatre étapes, toutes accomplies en modifiant le fichier de configuration IoT Edge :

1. Provisionnez manuellement chaque appareil en ajoutant sa chaîne de connexion au fichier de configuration.

1. Terminez la configuration des certificats de votre appareil en faisant pointer le fichier de configuration sur le certificat d’autorité de certification d’appareil, la clé privée de l’autorité de certification d’appareil et le certificat d’autorité de certification racine.

1. Amorcez le système à l’aide de l’agent IoT Edge.

1. Mettez à jour le paramètre **hostname** pour votre appareil de **couche supérieure**, et mettez à jour les paramètres **hostname** et **parent_hostname** pour vos appareils de **couche inférieure**.

Effectuez ces étapes et redémarrez le service IoT Edge pour configurer vos appareils.

1. Sur chaque appareil, ouvrez le fichier de configuration IoT Edge.

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

1. Recherchez les configurations de provisionnement du fichier et décommentez la section **Manual provisioning configuration using a connection string**.

1. Mettez à jour la valeur de **device_connection_string** avec la chaîne de connexion à partir de votre appareil IoT Edge. Assurez-vous que les commentaires des autres sections de provisionnement sont supprimés. Assurez-vous que la ligne **approvisionnement :** n’est pas précédée d’une espace et que les éléments imbriqués sont en retrait de deux espaces.

   ```yml
   # Manual provisioning configuration using a connection string
   provisioning:
     source: "manual"
     device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
     dynamic_reprovisioning: false
   ```

   >[!TIP]
   >Pour coller le contenu du Presse-papiers dans Nano, appuyez sur `Shift+Right Click` ou sur `Shift+Insert`.

1. Recherchez la section **certificates**. Supprimez les marques de commentaire et mettez à jour les trois champs de certificat pour qu’ils pointent vers les certificats que vous avez créés dans la section précédente et déplacés vers l’appareil IoT Edge. Fournissez les chemins d’URI de fichier, qui sont au format `file:///<path>/<filename>`.

   ```yml
   certificates:
     device_ca_cert: "<File URI path to the device CA certificate unique to this device.>"
     device_ca_pk: "<File URI path to the device CA private key unique to this device.>"
     trusted_ca_certs: "<File URI path to the root CA certificate shared by all devices in the gateway hierarchy.>"
   ```

1. Pour votre appareil de **couche supérieure**, recherchez le paramètre **hostname**. Mettez à jour la valeur pour qu’elle corresponde au nom de domaine complet (FQDN) ou à l’adresse IP de l’appareil IoT Edge. Utilisez la valeur que vous choisissez de manière cohérente sur tous les appareils de votre hiérarchie.

   ```yml
   hostname: <device fqdn or IP>
   ```

1. Pour les appareils IoT Edge des **couches inférieures**, mettez à jour le fichier de configuration afin qu’il pointe vers le nom de domaine complet ou l’adresse IP de l’appareil parent, en fonction de ce qui figure dans le champ **hostname** de l’appareil parent. Pour les appareils IoT Edge de la **couche supérieure**, laissez ce paramètre en commentaire.

   ```yml
   parent_hostname: <parent device fqdn or IP>
   ```

   > [!NOTE]
   > Pour les hiérarchies comportant plusieurs couches inférieures, mettez à jour le champ *parent_hostname* avec le nom de domaine complet de l’appareil de la couche située juste au-dessus.

1. Pour votre appareil de **couche supérieure**, recherchez la section **agent** yaml et mettez à jour la valeur de l’image avec la version appropriée de l’agent IoT Edge. Ici, nous allons faire pointer l’agent IoT Edge de la couche supérieure sur l’Azure Container Registry avec la préversion publique d’image de l’agent IoT Edge disponible.

   ```yml
   agent:
     name: "edgeAgent"
     type: "docker"
     env: {}
     config:
       image: "mcr.microsoft.com/azureiotedge-agent:1.2.0-rc2"
       auth: {}
   ```

1. Pour les appareils IoT Edge des **couches inférieures**, mettez à jour le nom de domaine de la valeur d’image pour qu’il pointe vers le nom de domaine complet ou l’adresse IP de l’appareil parent, suivi du port du proxy d’API, 8000. Vous ajouterez le module proxy d’API dans la section suivante.

   ```yml
   agent:
     name: "edgeAgent"
     type: "docker"
     env: {}
     config:
       image: "<parent_device_fqdn_or_ip>:8000/azureiotedge-agent:1.2.0-rc2"
       auth: {}
   ```

1. Enregistrez et fermez le fichier.

   `CTRL + X`, `Y`, `Enter`

1. Après avoir entré les informations de provisionnement dans le fichier de configuration, redémarrez le démon :

   ```bash
   sudo systemctl restart iotedge
   ```

## <a name="deploy-modules-to-the-top-layer-device"></a>Déployer des modules sur l’appareil de couche supérieure

Les étapes restantes pour terminer la configuration du runtime IoT Edge et déployer des charges de travail s’effectuent à partir du cloud par le biais du portail Azure ou d’Azure CLI.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le [portail Azure](https://ms.portal.azure.com/):

1. Accédez à votre IoT Hub.

1. Dans le menu du volet gauche, sous **Gestion automatique des appareils**, sélectionnez **IoT Edge**.

1. Cliquez sur l’ID de votre appareil de périphérie de **couche supérieure** dans la liste des appareils.

1. Dans la barre supérieure, sélectionnez **Définir des modules**.

1. Sélectionnez **Paramètres du runtime** en regard de l’icône d’engrenage.

1. Sous **Edge Hub**, dans le champ d’image, entrez `mcr.microsoft.com/azureiotedge-hub:1.2.0-rc2`.

   ![Modifier l’image du hub Edge](./media/tutorial-nested-iot-edge/edge-hub-image.png)

1. Ajoutez les variables d’environnement suivantes à votre module Edge Hub :

    | Nom | Valeur |
    | - | - |
    | `experimentalFeatures__enabled` | `true` |
    | `experimentalFeatures__nestedEdgeEnabled` | `true` |

   ![Modifier les variables d’environnement du hub Edge](./media/tutorial-nested-iot-edge/edge-hub-environment-variables.png)

1. Sous **Agent Edge**, dans le champ d’image, entrez `mcr.microsoft.com/azureiotedge-agent:1.2.0-rc2`. Sélectionnez **Enregistrer**.

1. Ajoutez le module de registre Docker à votre appareil de couche supérieure. Sélectionnez **+ Ajouter** et choisissez **Module IoT Edge** dans la liste déroulante. Spécifiez le nom `registry` pour votre module de registre Docker, et entrez `registry:latest` pour l’URI de l’image. Ensuite, ajoutez des variables d’environnement et créez des options pour faire pointer votre module de registre local sur le registre de conteneurs Microsoft afin de télécharger des images conteneur à partir de ce registre et de servir ces images à registry:5000.

1. Sous l’onglet des variables d’environnement, entrez la paire nom-valeur de variable d’environnement suivante :

    | Nom | Valeur |
    | - | - |
    | `REGISTRY_PROXY_REMOTEURL` | `https://mcr.microsoft.com` |

1. Sous l’onglet des options de création de conteneur, entrez le code JSON suivant :

   ```json
   {
    "HostConfig": {
        "PortBindings": {
            "5000/tcp": [
                {
                    "HostPort": "5000"
                }
            ]
         }
      }
   }
   ```

1. Ensuite, ajoutez le module de proxy d’API à votre appareil de couche supérieure. Sélectionnez **+ Ajouter** et choisissez **Module de la Place de marché** dans la liste déroulante. Recherchez `IoT Edge API Proxy` et sélectionnez le module. Le proxy d’API IoT Edge utilise le port 8000, et il est configuré pour utiliser par défaut votre module de registre nommé `registry` sur le port 5000.

1. Sélectionnez **Vérifier + créer**, puis **Créer** pour terminer le déploiement. Le runtime IoT Edge de votre appareil de couche supérieure, qui a accès à Internet, extraira et exécutera les configurations en **préversion publique** pour le hub IoT Edge et l’agent IoT Edge.

   ![Déploiement complet contenant le hub Edge, l’agent Edge, le module de registre et le module de proxy d’API](./media/tutorial-nested-iot-edge/complete-top-layer-deployment.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Dans [Azure Cloud Shell](https://shell.azure.com/), entrez la commande suivante pour créer un fichier deployment.json :

   ```azurecli-interactive
   code deploymentTopLayer.json
   ```

1. Copiez le contenu du code JSON ci-dessous dans votre fichier deployment.json, enregistrez-le et fermez-le.

   ```json
   {
       "modulesContent": {
           "$edgeAgent": {
               "properties.desired": {
                   "modules": {
                       "dockerContainerRegistry": {
                           "settings": {
                               "image": "registry:latest",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"5000/tcp\":[{\"HostPort\":\"5000\"}]}}}"
                           },
                           "type": "docker",
                           "version": "1.0",
                           "env": {
                               "REGISTRY_PROXY_REMOTEURL": {
                                   "value": "https://mcr.microsoft.com"
                           },
                           "status": "running",
                           "restartPolicy": "always"
                       },
                       "IoTEdgeAPIProxy": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-api-proxy",
                               "createOptions": "{\"HostConfig\": {\"PortBindings\": {\"8000/tcp\": [{\"HostPort\":\"8000\"}]}}}"
                           },
                           "type": "docker",
                           "env": {
                               "NGINX_DEFAULT_PORT": {
                                   "value": "8000"
                               },
                               "DOCKER_REQUEST_ROUTE_ADDRESS": {
                                   "value": "registry:5000"
                               },
                               "BLOB_UPLOAD_ROUTE_ADDRESS": {
                                   "value": "AzureBlobStorageonIoTEdge:11002"
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always",
                           "version": "1.0"
                       }
                   },
                   "runtime": {
                       "settings": {
                           "minDockerVersion": "v1.25",
                       },
                       "type": "docker"
                   },
                   "schemaVersion": "1.1",
                   "systemModules": {
                       "edgeAgent": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-agent:1.2.0-rc2",
                               "createOptions": ""
                           },
                           "type": "docker"
                       },
                       "edgeHub": {
                           "settings": {
                               "image": "mcr.microsoft.com/azureiotedge-hub:1.2.0-rc2",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
                           },
                           "type": "docker",
                           "env": {
                               "experimentalFeatures__enabled": {
                                   "value": "true"
                               },
                               "experimentalFeatures__nestedEdgeEnabled": {
                                   "value": "true"
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always"
                       }
                   }
               }
           },
           "$edgeHub": {
               "properties.desired": {
                   "routes": {
                       "route": "FROM /messages/* INTO $upstream"
                   },
                   "schemaVersion": "1.1",
                   "storeAndForwardConfiguration": {
                       "timeToLiveSecs": 7200
                   }
               }
           }
       }
   }
   ```

1. Entrez la commande suivante pour créer un déploiement sur votre appareil de périphérie de couche supérieure :

   ```azurecli-interactive
   az iot edge set-modules --device-id <top_layer_device_id> --hub-name <iot_hub_name> --content ./deploymentTopLayer.json
   ```

---

## <a name="deploy-modules-to-the-lower-layer-device"></a>Déployer des modules sur l’appareil de couche inférieure

Vous pouvez utiliser le portail Azure et Azure CLI pour déployer des charges de travail à partir du cloud vers vos appareils de **couche inférieure**.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Dans le [portail Azure](https://ms.portal.azure.com/):

1. Accédez à votre IoT Hub.

1. Dans le menu du volet gauche, sous **Gestion automatique des appareils**, sélectionnez **IoT Edge**.

1. Cliquez sur l’ID de votre appareil de couche inférieure dans la liste des appareils IoT Edge.

1. Dans la barre supérieure, sélectionnez **Définir des modules**.

1. Sélectionnez **Paramètres du runtime** en regard de l’icône d’engrenage.

1. Sous **Edge Hub**, dans le champ d’image, entrez `$upstream:8000/azureiotedge-hub:1.2.0-rc2`.

1. Ajoutez les variables d’environnement suivantes à votre module Edge Hub :

    | Nom | Valeur |
    | - | - |
    | `experimentalFeatures__enabled` | `true` |
    | `experimentalFeatures__nestedEdgeEnabled` | `true` |

1. Sous **Agent Edge**, dans le champ d’image, entrez `$upstream:8000/azureiotedge-agent:1.2.0-rc2`. Sélectionnez **Enregistrer**.

1. Ajoutez le module de capteur de température. Sélectionnez **+ Ajouter** et choisissez **Module de la Place de marché** dans la liste déroulante. Recherchez `Simulated Temperature Sensor` et sélectionnez le module.

1. Sous **Modules IoT Edge**, sélectionnez le module `Simulated Temperature Sensor` que vous venez d’ajouter et mettez à jour son URI d’image pour qu’il pointe sur `$upstream:8000/azureiotedge-simulated-temperature-sensor:1.0`.

1. Sélectionnez **Enregistrer**, **Vérifier + créer**, puis **Créer** pour terminer le déploiement.

   ![Déploiement complet contenant le hub Edge, l’agent Edge et le capteur de température simulé](./media/tutorial-nested-iot-edge/complete-lower-layer-deployment.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Dans [Azure Cloud Shell](https://shell.azure.com/), entrez la commande suivante pour créer un fichier deployment.json :

   ```azurecli-interactive
   code deploymentLowerLayer.json
   ```

1. Copiez le contenu du code JSON ci-dessous dans votre fichier deployment.json, enregistrez-le et fermez-le.

   ```json
   {
       "modulesContent": {
           "$edgeAgent": {
               "properties.desired": {
                   "modules": {
                       "simulatedTemperatureSensor": {
                           "settings": {
                               "image": "$upstream:8000/azureiotedge-simulated-temperature-sensor:1.0",
                               "createOptions": ""
                           },
                           "type": "docker",
                           "status": "running",
                           "restartPolicy": "always",
                           "version": "1.0"
                       }
                   },
                   "runtime": {
                       "settings": {
                           "minDockerVersion": "v1.25",
                       },
                       "type": "docker"
                   },
                   "schemaVersion": "1.1",
                   "systemModules": {
                       "edgeAgent": {
                           "settings": {
                               "image": "$upstream:8000/azureiotedge-agent:1.2.0-rc2",
                               "createOptions": ""
                           },
                           "type": "docker"
                       },
                       "edgeHub": {
                           "settings": {
                               "image": "$upstream:8000/azureiotedge-hub:1.2.0-rc2",
                               "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
                           },
                           "type": "docker",
                           "env": {
                               "experimentalFeatures__enabled": {
                                   "value": "true"
                               },
                               "experimentalFeatures__nestedEdgeEnabled": {
                                   "value": "true"
                               }
                           },
                           "status": "running",
                           "restartPolicy": "always"
                       }
                   }
               }
           },
           "$edgeHub": {
               "properties.desired": {
                   "routes": {
                       "route": "FROM /messages/* INTO $upstream"
                   },
                   "schemaVersion": "1.1",
                   "storeAndForwardConfiguration": {
                       "timeToLiveSecs": 7200
                   }
               }
           }
       }
   }
   ```

1. Entrez la commande suivante pour créer un déploiement de modules sur votre appareil de périphérie de couche inférieure :

   ```azurecli-interactive
   az iot edge set-modules --device-id <lower_layer_device_id> --hub-name <iot_hub_name> --content ./deploymentLowerLayer.json

---

Notice that the image URI that we used for the simulated temperature sensor module pointed to `$upstream:8000` instead of to a container registry. We configured this device to not have direct connections to the cloud, because it's in a lower layer. To pull container images, this device requests the image from its parent device instead. At the top layer, the API proxy module routes this container request to the registry module, which handles the image pull.

On the device details page for your lower layer IoT Edge device, you should now see the temperature sensor module listed along the system modules as **Specified in deployment**. It may take a few minutes for the device to receive its new deployment, request the container image, and start the module. Refresh the page until you see the temperature sensor module listed as **Reported by device**.

## View generated data

The **Simulated Temperature Sensor** module that you pushed generates sample environment data. It sends messages that include ambient temperature and humidity, machine temperature and pressure, and a timestamp.

You can watch the messages arrive at your IoT hub by using the [Azure IoT Hub extension for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit).

You can also view these messages through the [Azure Cloud Shell](https://shell.azure.com/):

   ```azurecli-interactive
   az iot hub monitor-events -n <iothub_name> -d <lower-layer-device-name>
   ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer les ressources Azure et les configurations locales que vous avez créées dans cet article pour éviter des frais.

Pour supprimer les ressources :

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Groupes de ressources**.

2. Sélectionnez le nom du groupe de ressources contenant vos ressources de test de IoT Edge. 

3. Passez en revue la liste des ressources contenues dans votre groupe de ressources. Si vous souhaitez toutes les supprimer, vous pouvez sélectionner **Supprimer le groupe de ressources**. Si vous souhaitez en supprimer seulement quelques-unes, vous pouvez cliquer sur chaque ressource pour les supprimer individuellement. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez configuré deux appareils IoT Edge en tant que passerelles et défini l’un des deux comme appareil parent de l’autre. Ensuite, vous avez démontré l’extraction d’une image conteneur sur l’appareil enfant par le biais d’une passerelle.

Vous pouvez passer à d’autres didacticiels pour savoir comment Azure IoT Edge peut trouver d’autres solutions pour votre entreprise.

> [!div class="nextstepaction"]
> [Déployer un modèle Azure Machine Learning en tant que module](tutorial-deploy-machine-learning.md)