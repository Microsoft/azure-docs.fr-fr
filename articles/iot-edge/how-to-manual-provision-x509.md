---
title: Provisionner avec des certificats X.509 - Azure IoT Edge | Microsoft Docs
description: Après son installation, provisionnez un appareil IoT Edge avec ses certificats d’identité et authentifiez-le auprès d’IoT Hub
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 10/06/2020
ms.author: kgremban
ms.openlocfilehash: abb3aa9ca7c9697fef1cf456964154249f0d69f3
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913974"
---
# <a name="set-up-an-azure-iot-edge-device-with-x509-certificate-authentication"></a>Configurer un appareil Azure IoT Edge avec une authentification par certificat X.509

Cet article décrit les étapes à effectuer pour inscrire un nouvel appareil IoT Edge auprès d’IoT Hub et configurer l’appareil en vue d’une authentification avec des certificats X.509.

Les étapes de cet article suivent un processus appelé provisionnement manuel, où vous connectez manuellement chaque appareil à son hub IoT. L’alternative est le provisionnement automatique à l’aide du service IoT Hub Device Provisioning. Cette méthode est particulièrement utile quand vous devez provisionner beaucoup d’appareils.

<!--TODO: Add auto-provision info/links-->

Avec le provisionnement manuel, vous avez le choix entre deux méthodes d’authentification des appareils IoT Edge :

* **Par clé symétrique**  : quand vous créez une identité d’appareil dans IoT Hub, le service crée deux clés. Vous placez l’une des clés sur l’appareil, lequel présente la clé à IoT Hub au moment de l’authentification.

  Cette méthode d’authentification est plus rapide pour commencer, mais moins sûre que l’autre.

* **Par certificat autosigné X.509**  : vous créez deux certificats d’identité X.509, que vous placez sur l’appareil. Quand vous créez une identité d’appareil dans IoT Hub, vous fournissez les empreintes numériques des deux certificats. Au moment de son authentification auprès d’IoT Hub, l’appareil présente ses certificats, et IoT Hub vérifie que ces derniers correspondent aux empreintes numériques.

  Cette méthode d’authentification étant plus sûre, elle est recommandée dans les scénarios de production.

Cet article décrit en détail le processus d’inscription et de provisionnement d’un appareil avec une authentification par certificat X.509. Pour savoir comment configurer un appareil avec des clés symétriques, consultez [Configurer un appareil Azure IoT Edge avec une authentification par clé symétrique](how-to-manual-provision-symmetric-key.md).

## <a name="prerequisites"></a>Prérequis

Avant d’effectuer les étapes décrites dans cet article, vérifiez que vous avez un appareil sur lequel est installé le runtime IoT Edge. Si ce n’est pas le cas, suivez les étapes décrites dans [Installer ou désinstaller le runtime Azure IoT Edge](how-to-install-iot-edge.md).

Le provisionnement manuel avec des certificats X.509 requiert IoT Edge version 1.0.10 ou ultérieure.

## <a name="create-certificates-and-thumbprints"></a>Créer des certificats et des empreintes numériques

Le certificat d’identité d’appareil est un certificat feuille qui se connecte par le biais d’une chaîne de certificats de confiance au certificat d’autorité de certification X.509 supérieur. Le nom commun (CN) du certificat d’identité d’appareil doit être défini sur l’ID d’appareil que vous souhaitez que l’appareil ait dans votre hub IoT.

Les certificats d’identité d’appareil sont utilisés uniquement pour provisionner l’appareil IoT Edge et l’authentifier auprès d’Azure IoT Hub. Ils ne signent pas de certificats, contrairement aux certificats d’autorité de certification que l’appareil IoT Edge présente aux modules ou aux appareils de nœud terminal à des fins de vérification. Pour plus d’informations, consultez les [détails sur l’utilisation des certificats par Azure IoT Edge](iot-edge-certs.md).

Après avoir créé le certificat d’identité d’appareil, vous devez avoir deux fichiers : un fichier .cer ou .pem, qui contient la partie publique du certificat, et un fichier .cer ou .pem, qui comporte la clé privée du certificat.

Les fichiers suivants sont nécessaires pour le provisionnement manuel avec X.509 :

* Deux ensembles de certificats d’identité d’appareil et de certificats de clé privée. Un ensemble de fichiers de certificat/clé est fourni au runtime IoT Edge.
* Empreintes numériques tirées des deux certificats d’identité d’appareil. Les valeurs d’empreinte numérique sont constituées de 40 caractères hexadécimaux pour les hachages SHA-1 ou de 64 caractères hexadécimaux pour les hachages SHA-256. Les deux empreintes numériques sont fournies pour IoT Hub lors de l’enregistrement de l’appareil.

Si vous n’avez pas de certificats disponibles, vous pouvez [créer des certificats de démonstration pour tester les fonctionnalités de l’appareil IoT Edge](how-to-create-test-certificates.md). Suivez les instructions de cet article pour configurer des scripts de création de certificat, créer un certificat d’autorité de certification racine, puis créer deux certificats d’identité d’appareil IoT Edge.

L’une des méthodes permettant de récupérer l’empreinte numérique d’un certificat consiste à utiliser la commande openssl suivante :

```cmd
openssl x509 -in <certificate filename>.pem -text -fingerprint
```

## <a name="register-a-new-device"></a>Inscrire un nouvel appareil

Chaque appareil qui se connecte à un hub IoT a un ID d’appareil qui sert au suivi des communications cloud-à-appareil ou appareil-à-cloud. Vous configurez un appareil avec ses informations de connexion, qui comprennent le nom d’hôte du hub IoT, l’ID d’appareil ainsi que les informations que l’appareil utilise pour s’authentifier auprès d’IoT Hub.

Pour l’authentification par certificat X.509, ces informations sont fournies sous la forme d’ *empreintes numériques* extraites de vos certificats d’identité d’appareil. Ces empreintes numériques, fournies à IoT Hub au moment de l’inscription de l’appareil, permettent au service de reconnaître l’appareil qui se connecte.

Vous avez le choix entre plusieurs outils pour inscrire un nouvel appareil IoT Edge dans IoT Hub et charger ses empreintes numériques de certificat.

# <a name="portal"></a>[Portail](#tab/azure-portal)

### <a name="prerequisites-for-the-azure-portal"></a>Conditions préalables pour le portail Azure

Un [hub IoT](../iot-hub/iot-hub-create-through-portal.md) gratuit ou standard dans votre abonnement Azure.

### <a name="create-an-iot-edge-device-in-the-azure-portal"></a>Créer un appareil IoT Edge dans le portail Azure

Dans votre IoT Hub dans le portail Azure, les appareils IoT Edge sont créés et gérés séparément des appareils IoT qui ne sont pas compatibles avec la périphérie.

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre IoT Hub.

1. Dans le volet de gauche, sélectionnez **IoT Edge** dans le menu, puis sélectionnez **Ajouter un appareil IoT Edge**.

   ![Ajouter un appareil IoT Edge à partir du portail Azure](./media/how-to-manual-provision-symmetric-key/portal-add-iot-edge-device.png)

1. Dans la page **Créer un appareil** , fournissez les informations suivantes :

   * Créez un ID d’appareil descriptif. Notez cet ID d’appareil, car vous en aurez besoin dans la prochaine section.
   * Sélectionnez **X.509 autosigné** comme type d’authentification.
   * Fournissez les empreintes numériques des certificats d’identité principal et secondaire. Les valeurs d’empreinte numérique sont constituées de 40 caractères hexadécimaux pour les hachages SHA-1 ou de 64 caractères hexadécimaux pour les hachages SHA-256.

1. Sélectionnez **Enregistrer**.

### <a name="view-iot-edge-devices-in-the-azure-portal"></a>Afficher des appareils IoT Edge dans le portail Azure

Tous les appareils compatibles avec Edge qui se connectent à votre hub IoT sont répertoriés dans la page **IoT Edge**.

![Afficher tous les appareils IoT Edge de votre hub IoT](./media/how-to-manual-provision-symmetric-key/portal-view-devices.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="prerequisites-for-the-azure-cli"></a>Conditions préalables pour Azure CLI

* Un [hub IoT](../iot-hub/iot-hub-create-using-cli.md) dans votre abonnement Azure.
* [Azure CLI](/cli/azure/install-azure-cli) dans votre environnement. Vous devez utiliser Azure CLI version 2.0.70 ou ultérieure. Utilisez `az --version` pour valider. Cette version prend en charge les commandes d’extension az et introduit l’infrastructure de la commande Knack.
* [Extension IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension).

### <a name="create-an-iot-edge-device-with-the-azure-cli"></a>Créer un appareil IoT Edge avec Azure CLI

Utilisez la commande [az iot hub device-identity create](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-create) pour créer une identité d’appareil dans votre hub IoT. Par exemple :

   ```azurecli
   az iot hub device-identity create --device-id [device id] --hub-name [hub name] --edge-enabled --auth-method x509_thumbprint --primary-thumbprint [SHA thumbprint] --secondary-thumbprint [SHA thumbprint]
   ```

Cette commande comprend plusieurs paramètres :

* `--device-id` ou `-d` : fournissez un nom descriptif unique à votre hub IoT. Notez cet ID d’appareil, car vous en aurez besoin dans la prochaine section.
* `hub-name` ou `-n` : indiquez le nom de votre hub IoT.
* `--edge-enabled` ou `--ee` : déclarez que l’appareil est un appareil IoT Edge.
* `--auth-method` ou `--am` : déclarez le type d’autorisation que l’appareil utilisera. Ici, nous utilisons des empreintes numériques de certificat X.509.
* `--primary-thumbprint` ou `--ptp` : fournissez une empreinte numérique de certificat X.509 à utiliser comme clé primaire.
* `--secondary-thumbprint` ou `--stp` : fournissez une empreinte numérique de certificat X.509 à utiliser comme clé secondaire.

### <a name="view-iot-edge-devices-with-the-azure-cli"></a>Affichez des appareils IoT Edge avec Azure CLI

Utilisez la commande [az iot hub device-identity list](/cli/azure/ext/azure-iot/iot/hub/device-identity#ext-azure-iot-az-iot-hub-device-identity-list) pour voir tous les appareils dans votre hub IoT. Par exemple :

   ```azurecli
   az iot hub device-identity list --hub-name [hub name]
   ```

Ajoutez l’indicateur `--edge-enabled` ou `--ee` pour lister uniquement les appareils IoT Edge de votre hub IoT.

Tout appareil inscrit en tant qu’appareil IoT Edge a la propriété **capabilities.iotEdge** définie sur **true**.

--- 

## <a name="configure-an-iot-edge-device"></a>Configurer un appareil IoT Edge

Une fois que l’appareil IoT Edge a une identité dans IoT Hub, vous devez configurer l’appareil avec son identité cloud et ses certificats d’identité.

Sur un appareil Linux, vous fournissez ces informations en modifiant un fichier config.yaml. Sur un appareil Linux, vous fournissez ces informations en exécutant un script PowerShell.

# <a name="linux"></a>[Linux](#tab/linux)

1. Sur l’appareil IoT Edge, ouvrez le fichier de configuration.

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

1. Recherchez la section des configurations de provisionnement dans le fichier. 

1. Commentez la section **Manual provisioning configuration using a connection string**.

1. Décommentez la section **Manual provisioning configuration using an X.509 identity certificate**. Assurez-vous que la ligne **d’approvisionnement :** n’est pas précédée d’une espace et que les éléments imbriqués sont en retrait de deux espaces.

   ```yml
   # Manual provisioning configuration using a connection string
   provisioning:
     source: "manual"
     authentication:
       method: "x509"
       iothub_hostname: "<REQUIRED IOTHUB HOSTNAME>"
       device_id: "<REQUIRED DEVICE ID PROVISIONED IN IOTHUB>"
       identity_cert: "<REQUIRED URI TO DEVICE IDENTITY CERTIFICATE>"
       identity_pk: "<REQUIRED URI TO DEVICE IDENTITY PRIVATE KEY>"
     dynamic_reprovisioning: false
   ```

1. Mettez à jour les champs suivants :

   * **iothub_hostname**  : nom d’hôte de l’hub IoT auquel l’appareil doit se connecter. Par exemple, `{IoT hub name}.azure-devices.net`.
   * **device_id**  : ID que vous avez indiqué lors de l’inscription de l’appareil.
   * **identity_cert**  : URI vers un certificat d’identité sur l’appareil. Par exemple, `file:///path/identity_certificate.pem`.
   * **identity_pk**  : URI vers le fichier de clé privée pour le certificat d’identité fourni. Par exemple, `file:///path/identity_key.pem`.

1. Enregistrez et fermez le fichier.

   `CTRL + X`, `Y`, `Enter`

1. Après avoir entré les informations de provisionnement dans le fichier de configuration, redémarrez le démon :

   ```bash
   sudo systemctl restart iotedge
   ```

# <a name="windows"></a>[Windows](#tab/windows)

1. Sur l’appareil IoT Edge, exécutez PowerShell en tant qu’administrateur.

2. Utilisez la commande [Initialize-IoTEdge](reference-windows-scripts.md#initialize-iotedge) pour configurer le runtime IoT Edge sur votre ordinateur.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -ManualX509
   ```

   * Si vous utilisez des conteneurs Linux, ajoutez le paramètre `-ContainerOs` à l’indicateur. Soyez cohérent avec l’option de conteneur que vous avez choisie dans la commande `Deploy-IoTEdge` exécutée précédemment.

      ```powershell
      . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
      Initialize-IoTEdge -ManualX509 -ContainerOs Linux
      ```

   * Si vous avez téléchargé le script IoTEdgeSecurityDaemon.ps1 sur votre appareil pour une installation hors connexion ou l’installation d’une version spécifique, veillez à référencer la copie locale du script.

      ```powershell
      . <path>/IoTEdgeSecurityDaemon.ps1
      Initialize-IoTEdge -ManualX509
      ```

3. À l’invite, fournissez les informations suivantes :

   * **IotHubHostName**  : nom d’hôte de l’hub IoT auquel l’appareil doit se connecter. Par exemple, `{IoT hub name}.azure-devices.net`.
   * **DeviceId**  : ID que vous avez indiqué lors de l’inscription de l’appareil.
   * **X509IdentityCertificate**  : chemin absolu d’un certificat d’identité sur l’appareil. Par exemple, `C:\path\identity_certificate.pem`.
   * **X509IdentityPrivateKey**  : chemin absolu du fichier de clé privée pour le certificat d’identité fourni. Par exemple, `C:\path\identity_key.pem`.

Quand vous provisionnez un appareil manuellement, vous pouvez utiliser des paramètres supplémentaires pour changer le processus, à savoir :

* Diriger le trafic pour qu’il transite par un serveur proxy,
* Déclarer une image conteneur d’agent Edge spécifique, et fournir des informations d’identification si l’image se trouve dans un registre privé

Pour plus d’informations sur ces paramètres supplémentaires, consultez [Scripts PowerShell pour IoT Edge sur Windows](reference-windows-scripts.md).

---

[!INCLUDE [Verify and troubleshoot installation](../../includes/iot-edge-verify-troubleshoot-install.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez à [Déployer des modules IoT Edge](how-to-deploy-modules-portal.md) pour savoir comment déployer des modules sur votre appareil.