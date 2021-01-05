---
title: Effectuer le provisionnement avec des clés symétriques - Azure IoT Edge | Microsoft Docs
description: Après son installation, provisionnez un appareil IoT Edge avec des clés symétriques à l’aide de sa chaîne de connexion et authentifiez-le auprès d’IoT Hub
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 10/06/2020
ms.author: kgremban
ms.openlocfilehash: f5371539c1b45c14b519729c7c07003bf74847a0
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043865"
---
# <a name="set-up-an-azure-iot-edge-device-with-symmetric-key-authentication"></a>Configurer un appareil Azure IoT Edge avec une authentification par clé symétrique

Cet article décrit les étapes à effectuer pour inscrire un nouvel appareil IoT Edge auprès d’IoT Hub et configurer l’appareil en vue d’une authentification avec des clés symétriques.

Les étapes de cet article suivent un processus appelé provisionnement manuel, où vous connectez manuellement chaque appareil à son hub IoT. L’alternative est le provisionnement automatique à l’aide du service IoT Hub Device Provisioning. Cette méthode est particulièrement utile quand vous devez provisionner beaucoup d’appareils.

<!--TODO: Add auto-provision info/links-->

Avec le provisionnement manuel, vous avez le choix entre deux méthodes d’authentification des appareils IoT Edge :

* **Par clé symétrique** : quand vous créez une identité d’appareil dans IoT Hub, le service crée deux clés. Vous placez l’une des clés sur l’appareil, lequel présente la clé à IoT Hub au moment de l’authentification.

  Cette méthode d’authentification est plus rapide pour commencer, mais moins sûre que l’autre.

* **Par certificat autosigné X.509** : vous créez deux certificats d’identité X.509, que vous placez sur l’appareil. Quand vous créez une identité d’appareil dans IoT Hub, vous fournissez les empreintes numériques des deux certificats. Au moment de son authentification auprès d’IoT Hub, l’appareil présente ses certificats, et IoT Hub vérifie que ces derniers correspondent aux empreintes numériques.

  Cette méthode d’authentification étant plus sûre, elle est recommandée dans les scénarios de production.

Cet article décrit en détail le processus d’inscription et de provisionnement d’un appareil avec une authentification par clé symétrique. Pour savoir comment configurer un appareil avec des certificats X.509, consultez [Configurer un appareil Azure IoT Edge avec une authentification par certificat X.509](how-to-manual-provision-x509.md).

## <a name="prerequisites"></a>Prérequis

Avant d’effectuer les étapes décrites dans cet article, vérifiez que vous avez un appareil sur lequel est installé le runtime IoT Edge. Si ce n’est pas le cas, suivez les étapes décrites dans [Installer ou désinstaller le runtime Azure IoT Edge](how-to-install-iot-edge.md).

## <a name="register-a-new-device"></a>Inscrire un nouvel appareil

Chaque appareil qui se connecte à un hub IoT a un ID d’appareil qui sert au suivi des communications cloud-à-appareil ou appareil-à-cloud. Vous configurez un appareil avec ses informations de connexion, qui comprennent le nom d’hôte du hub IoT, l’ID d’appareil ainsi que les informations que l’appareil utilise pour s’authentifier auprès d’IoT Hub.

Avec l’authentification par clé symétrique, ces informations sont collectées dans une *chaîne de connexion* que vous pouvez récupérer sur IoT Hub, avant de les placer sur votre appareil IoT Edge.

Selon vos préférences, vous avez le choix entre plusieurs outils pour inscrire un nouvel appareil IoT Edge dans IoT Hub et récupérer sa chaîne de connexion.

# <a name="portal"></a>[Portail](#tab/azure-portal)

### <a name="prerequisites-for-the-azure-portal"></a>Conditions préalables pour le portail Azure

Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) gratuit ou standard dans votre abonnement Azure.

### <a name="create-an-iot-edge-device-in-the-azure-portal"></a>Créer un appareil IoT Edge dans le portail Azure

Dans votre IoT Hub dans le portail Azure, les appareils IoT Edge sont créés et gérés séparément des appareils IoT qui ne sont pas compatibles avec la périphérie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre IoT Hub.

1. Dans le volet de gauche, sélectionnez **IoT Edge** dans le menu, puis sélectionnez **Ajouter un appareil IoT Edge**.

   ![Ajouter un appareil IoT Edge à partir du portail Azure](./media/how-to-manual-provision-symmetric-key/portal-add-iot-edge-device.png)

1. Dans la page **Créer un appareil**, fournissez les informations suivantes :

   * Créez un ID d’appareil descriptif.
   * Sélectionnez **Clé symétrique** comme type d’authentification.
   * Utilisez les paramètres par défaut pour générer automatiquement les clés d’authentification et connecter le nouvel appareil à votre hub.

1. Sélectionnez **Enregistrer**.

### <a name="view-iot-edge-devices-in-the-azure-portal"></a>Afficher des appareils IoT Edge dans le portail Azure

Tous les appareils compatibles avec Edge qui se connectent à votre hub IoT sont répertoriés dans la page **IoT Edge**.

![Utiliser le portail Azure pour voir tous les appareils IoT Edge dans votre hub IoT](./media/how-to-manual-provision-symmetric-key/portal-view-devices.png)

### <a name="retrieve-the-connection-string-in-the-azure-portal"></a>Récupérer la chaîne de connexion sur le Portail Azure

Pour configurer votre appareil, vous avez besoin de la chaîne de connexion qui établit un lien entre votre appareil physique et son identité dans le hub IoT.

1. Dans la page **IoT Edge** du portail, cliquez sur l’ID de l’appareil dans la liste des appareils IoT Edge.
2. Copiez la valeur de **Chaîne de connexion principale** ou **Chaîne de connexion secondaire**.

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

### <a name="prerequisites-for-visual-studio-code"></a>Conditions préalables pour Visual Studio Code

* Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) gratuit ou standard dans votre abonnement Azure
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) pour Visual Studio Code.

### <a name="sign-in-to-access-your-iot-hub"></a>Se connecter pour accéder à votre hub ioT

Vous pouvez utiliser les extensions Azure IoT pour Visual Studio Code afin d’effectuer des opérations avec votre IoT Hub. Pour que ces opérations fonctionnent, vous devez vous connecter à votre compte Azure, puis sélectionner votre IoT Hub.

1. Dans Visual Studio Code, ouvrez la vue **Explorateur**.
1. Au bas de l’Explorateur, développez la section **Azure IoT Hub**.

   ![Développer la section Appareils Azure IoT Hub](./media/how-to-manual-provision-symmetric-key/azure-iot-hub-devices.png)

1. Cliquez sur **...** dans l’en-tête de section **Azure IoT Hub**. Si vous ne voyez pas les points de suspension, cliquez ou pointez sur l’en-tête.
1. Choisissez **Sélectionner un hub IoT**.
1. Si vous n’êtes pas connecté à votre compte Azure, suivez les invites de connexion.
1. Sélectionnez votre abonnement Azure.
1. Sélectionnez votre hub IoT.

### <a name="create-an-iot-edge-device-with-visual-studio-code"></a>Créer un appareil IoT Edge avec Visual Studio Code

1. Dans l’Explorateur VS Code, développez la section **Appareils Azure IoT Hub**.
1. Cliquez sur **...** dans l’en-tête de section **Appareils Azure IoT Hub**. Si vous ne voyez pas les points de suspension, cliquez ou pointez sur l’en-tête.
1. Sélectionnez **Créer un appareil IoT Edge**.
1. Dans la zone de texte qui s’ouvre, donnez un ID à votre appareil.

Dans l’écran de sortie, vous voyez le résultat de la commande. Les informations de l’appareil apparaissent, notamment l’ID d’appareil (**deviceId**) que vous avez fourni et la chaîne de connexion (**connectionString**) qui vous permet de connecter votre appareil physique à votre hub IoT.

Dans l’écran de sortie, vous voyez le résultat de la commande. Les informations de l’appareil apparaissent, notamment l’ID d’appareil (**deviceId**) que vous avez fourni et la chaîne de connexion (**connectionString**) qui vous permet de connecter votre appareil physique à votre hub IoT.

### <a name="view-iot-edge-devices-with-visual-studio-code"></a>Afficher des appareils IoT Edge avec Visual Studio Code

Tous les appareils qui se connectent à votre hub IoT sont listés dans la section **Azure IoT Hub** de l’Explorateur Visual Studio Code. Les appareils IoT Edge se distinguent des appareils non Edge par leur icône et par le fait que les modules **$edgeAgent** et **$edgeHub** sont développés sur chacun d’eux.

![Utiliser VS Code pour voir tous les appareils IoT Edge dans votre hub IoT](./media/how-to-manual-provision-symmetric-key/view-devices.png)

### <a name="retrieve-the-connection-string-with-visual-studio-code"></a>Récupérer la chaîne de connexion avec Visual Studio Code

Pour configurer votre appareil, vous avez besoin de la chaîne de connexion qui établit un lien entre votre appareil physique et son identité dans le hub IoT.

1. Cliquez avec le bouton droit sur l’ID de votre appareil dans la section **Azure IoT Hub**.
1. Sélectionnez **Copier la chaîne de connexion de l’appareil**.

   La chaîne de connexion est copiée dans le Presse-papiers.

Vous pouvez également sélectionner **Obtenir les informations de l’appareil** dans le menu contextuel pour voir les informations de l’appareil, notamment la chaîne de connexion, dans la fenêtre de sortie.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="prerequisites-for-the-azure-cli"></a>Conditions préalables pour Azure CLI

* Un [hub IoT](../iot-hub/iot-hub-create-using-cli.md) dans votre abonnement Azure.
* [Azure CLI](/cli/azure/install-azure-cli) dans votre environnement. Vous devez utiliser Azure CLI version 2.0.70 ou ultérieure. Utilisez `az --version` pour valider. Cette version prend en charge les commandes d’extension az et introduit l’infrastructure de la commande Knack.
* [Extension IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension).

### <a name="create-an-iot-edge-device-with-the-azure-cli"></a>Créer un appareil IoT Edge avec Azure CLI

Utilisez la commande [az iot hub device-identity create](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-create) pour créer une identité d’appareil dans votre hub IoT. Par exemple :

   ```azurecli
   az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled
   ```

Cette commande inclut trois paramètres :

* `--device-id` ou `-d` : fournissez un nom descriptif unique à votre hub IoT.
* `hub-name` ou `-n` : indiquez le nom de votre hub IoT.
* `--edge-enabled` ou `--ee` : déclarez que l’appareil est un appareil IoT Edge.

   ![az iot hub device-identity create output](./media/how-to-manual-provision-symmetric-key/create-edge-device-cli.png)

### <a name="view-iot-edge-devices-with-the-azure-cli"></a>Affichez des appareils IoT Edge avec Azure CLI

Utilisez la commande [az iot hub device-identity list](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-list) pour voir tous les appareils dans votre hub IoT. Par exemple :

   ```azurecli
   az iot hub device-identity list --hub-name [hub name]
   ```

Tout appareil inscrit en tant qu’appareil IoT Edge a la propriété **capabilities.iotEdge** définie sur **true**.

### <a name="retrieve-the-connection-string-with-the-azure-cli"></a>Récupérer la chaîne de connexion avec Azure CLI

Pour configurer votre appareil, vous avez besoin de la chaîne de connexion qui établit un lien entre votre appareil physique et son identité dans le hub IoT. Utilisez la commande [az iot hub device-identity show-connection-string](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-show-connection-string) pour retourner la chaîne de connexion d’un appareil unique :

   ```azurecli
   az iot hub device-identity show-connection-string --device-id [device id] --hub-name [hub name]
   ```

La valeur du paramètre `device-id` respecte la casse. Ne copiez pas les guillemets qui entourent la chaîne de connexion.

---

## <a name="provision-an-iot-edge-device"></a>Provisionner un appareil IoT Edge

Une fois que l’appareil IoT Edge a une identité dans IoT Hub et une chaîne de connexion qu’il peut utiliser pour l’authentification, vous devez provisionner l’appareil proprement dit avec ces informations.

Sur un appareil Linux, vous fournissez la chaîne de connexion en modifiant un fichier config.yaml. Sur un appareil Windows, vous fournissez la chaîne de connexion en exécutant un script PowerShell.

# <a name="linux"></a>[Linux](#tab/linux)

Sur l’appareil IoT Edge, ouvrez le fichier de configuration.

```bash
sudo nano /etc/iotedge/config.yaml
```

Recherchez les configurations de provisionnement du fichier et décommentez la section **Manual provisioning configuration using a connection string**. 

Mettez à jour la valeur de **device_connection_string** avec la chaîne de connexion à partir de votre appareil IoT Edge. Assurez-vous que les commentaires des autres sections de provisionnement sont supprimés. Assurez-vous que la ligne **approvisionnement :** n’est pas précédée d’une espace et que les éléments imbriqués sont en retrait de deux espaces.

```yml
# Manual provisioning configuration using a connection string
provisioning:
  source: "manual"
  device_connection_string: "<ADD DEVICE CONNECTION STRING HERE>"
  dynamic_reprovisioning: false
```

Pour coller le contenu du Presse-papiers dans Nano, appuyez sur `Shift+Right Click` ou sur `Shift+Insert`.

Enregistrez et fermez le fichier.

   `CTRL + X`, `Y`, `Enter`

Après avoir entré les informations de provisionnement dans le fichier de configuration, redémarrez le démon :

```bash
sudo systemctl restart iotedge
```

# <a name="windows"></a>[Windows](#tab/windows)

1. Sur l’appareil IoT Edge, exécutez PowerShell en tant qu’administrateur.

2. Utilisez la commande [Initialize-IoTEdge](reference-windows-scripts.md#initialize-iotedge) pour configurer le runtime IoT Edge sur votre ordinateur. Par défaut, la commande est une commande d’approvisionnement manuel avec des conteneurs Windows.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -ManualConnectionString -ContainerOs Windows
   ```

   * Si vous utilisez des conteneurs Linux, ajoutez le paramètre `-ContainerOs` à l’indicateur. Soyez cohérent avec l’option de conteneur que vous avez choisie dans la commande `Deploy-IoTEdge` exécutée précédemment.

      ```powershell
      . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
      Initialize-IoTEdge -ContainerOs Linux
      ```

   * Si vous avez téléchargé le script IoTEdgeSecurityDaemon.ps1 sur votre appareil pour une installation hors connexion ou l’installation d’une version spécifique, veillez à référencer la copie locale du script.

      ```powershell
      . <path>/IoTEdgeSecurityDaemon.ps1
      Initialize-IoTEdge -ManualX509
      ```

3. Lorsque vous y êtes invité, spécifiez la chaîne de connexion de l’appareil que vous avez récupérée dans la section précédente. La chaîne de connexion de l’appareil associe l’appareil physique à un ID d’appareil dans IoT Hub et fournit des informations d’authentification.

   La chaîne de connexion se présente au format suivant et ne doit pas contenir de guillemets : `HostName={IoT hub name}.azure-devices.net;DeviceId={device name};SharedAccessKey={key}`

Quand vous provisionnez un appareil manuellement, vous pouvez utiliser des paramètres supplémentaires pour changer le processus, à savoir :

* Diriger le trafic pour qu’il transite par un serveur proxy,
* Déclarer une image conteneur d’agent Edge spécifique, et fournir des informations d’identification si l’image se trouve dans un registre privé

Pour plus d’informations sur ces paramètres supplémentaires, consultez [Scripts PowerShell pour IoT Edge sur Windows](reference-windows-scripts.md).

---

[!INCLUDE [Verify and troubleshoot installation](../../includes/iot-edge-verify-troubleshoot-install.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez à [Déployer des modules IoT Edge](how-to-deploy-modules-portal.md) pour savoir comment déployer des modules sur votre appareil.