---
title: Provisionner un appareil à l’aide d’une attestation de clé symétrique - Azure IoT Edge
description: Utiliser l’attestation de clé symétrique pour tester l’approvisionnement automatique des appareils pour Azure IoT Edge avec le service Device Provisioning
author: kgremban
manager: philmea
ms.author: kgremban
ms.reviewer: mrohera
ms.date: 4/3/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: 10ed546e8f05f4a93e4523c7870f79d41aa1f622
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92045990"
---
# <a name="create-and-provision-an-iot-edge-device-using-symmetric-key-attestation"></a>Créer et approvisionner un appareil IoT Edge à l’aide de l’attestation de clé symétrique

Les appareils Azure IoT Edge peuvent être provisionnés automatiquement à l’aide du [service Device Provisioning](../iot-dps/index.yml) tout comme les appareils qui ne sont pas compatibles avec Edge. Si vous ne connaissez pas le processus de provisionnement automatique, révisez la présentation du [provisionnement](../iot-dps/about-iot-dps.md#provisioning-process) avant de poursuivre.

Cet article vous montre comment créer une inscription individuelle au service Device Provisioning à l’aide de l’attestation de clé symétrique sur un appareil IoT Edge, en procédant comme suit :

* Création d’une instance du service IoT Hub Device Provisioning.
* Création d’une inscription individuelle pour l’appareil.
* Installation du runtime IoT Edge et de la connexion à IoT Hub.

L’attestation de clé symétrique est une approche simple pour authentifier un appareil avec une instance du service Device Provisioning. Cette méthode d’attestation représente une expérience « Hello world » pour les développeurs qui découvrent le provisionnement d’appareils ou n’ont pas d’exigences de sécurité strictes. L’attestation d’appareil avec un [Module de plateforme sécurisée (TPM)](../iot-dps/concepts-tpm-attestation.md) ou un [certificat X.509](../iot-dps/concepts-x509-attestation.md) est plus sécurisée, et doit être utilisée lorsque les exigences de sécurité sont plus strictes.

## <a name="prerequisites"></a>Prérequis

* Un IoT Hub actif
* Un appareil physique ou virtuel

## <a name="set-up-the-iot-hub-device-provisioning-service"></a>Configurer le service IoT Hub Device Provisioning

Créez une nouvelle instance du service IoT Hub Device Provisioning dans Azure et liez-la à votre hub IoT. Vous pouvez suivre les instructions dans [Configurer le service IoT Hub Device Provisioning](../iot-dps/quick-setup-auto-provision.md).

Après avoir lancé l’exécution du service Device Provisioning, copiez la valeur de **Étendue de l’ID** à partir de la page de présentation. Vous utilisez cette valeur lorsque vous configurez le runtime IoT Edge.

## <a name="choose-a-unique-registration-id-for-the-device"></a>Choisir un ID d’inscription unique pour l’appareil

Un ID d’inscription unique doit être défini pour identifier chaque appareil. Vous pouvez utiliser l’adresse MAC, le numéro de série ou toute information unique provenant de l’appareil.

Dans cet exemple, nous utilisons une combinaison d’une adresse MAC et d’un numéro de série, forment la chaîne suivante pour un identifiant d’inscription : `sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6`.

Créez un ID d’inscription unique pour votre appareil. Les caractères valides sont les caractères alphanumériques minuscules et les tirets (« - »).

## <a name="create-a-dps-enrollment"></a>Créer une inscription au service Device Provisioning

Utilisez l’ID d’inscription de votre appareil pour créer une inscription individuelle au service Device Provisioning.

Lorsque vous créez une inscription auprès du service Device Provisioning, vous avez la possibilité de déclarer un **État initial du jumeau d’appareil**. Dans le jumeau d’appareil, vous pouvez définir des balises pour regrouper les appareils en fonction des métriques dont vous avez besoin dans votre solution, comme la région, l’environnement, l’emplacement ou le type d’appareil. Ces balises sont utilisées pour créer [des déploiements automatiques](how-to-deploy-at-scale.md).

> [!TIP]
> Les inscriptions de groupe sont également possibles lors de l’utilisation de l’attestation de clé symétrique et impliquent les mêmes décisions que les inscriptions individuelles.

1. Dans le [Portail Microsoft Azure](https://portal.azure.com), accédez à votre instance du service IoT Hub Device Provisioning.

1. Sous **Paramètres**, sélectionnez **Gérer les inscriptions**.

1. Sélectionnez **Ajouter une inscription individuelle** et suivez ces étapes pour configurer l’inscription :  

   1. Pour **Mécanisme**, sélectionnez **Clé symétrique**.

   1. Cochez la case **Générer automatiquement les clés**.

   1. Indiquez l’**ID d’inscription** que vous avez créé pour votre appareil.

   1. Fournissez un **ID d’appareil loT Hub** pour votre appareil si vous le souhaitez. Vous pouvez utiliser l’ID d’appareil pour cibler un appareil individuel lors du déploiement de module. Si vous ne fournissez pas un ID d’appareil, l’ID d’inscription est utilisé.

   1. Sélectionnez **Vrai** pour déclarer que cette inscription est un appareil IoT Edge. Pour une inscription de groupe, tous les appareils doivent être de IoT Edge, ou aucun d’entre eux ne peut l’être.

   > [!TIP]
   > Dans l'interface de ligne de commande Azure, vous pouvez créer une [inscription](/cli/azure/ext/azure-iot/iot/dps/enrollment) ou un [groupe d'inscriptions](/cli/azure/ext/azure-iot/iot/dps/enrollment-group) et utiliser l'indicateur **compatible avec Edge** pour spécifier qu'un appareil, ou un groupe d'appareils, est un appareil IoT Edge.

   1. Acceptez la valeur par défaut de la stratégie d’allocation du service Device Provisioning pour déterminer la façon dont **vous souhaitez affecter des appareils aux hubs** ou choisissez une autre valeur spécifique à cette inscription.

   1. Choisissez le **Hub IoT** lié auquel vous voulez connecter votre appareil. Vous pouvez choisir plusieurs hubs : l’appareil sera affecté à l’un d’entre eux en fonction de la stratégie d’allocation sélectionnée.

   1. Choisissez la façon dont **vous souhaitez que les données de l’appareil soient traitées lors du réapprovisionnement** lorsque les appareils demandent la configuration suite à la première fois.

   1. Ajoutez une valeur de balise à l’**État initial du jumeau d’appareil** si vous le souhaitez. Vous pouvez utiliser des balises pour cibler des groupes d’appareils lors du déploiement de module. Par exemple :

      ```json
      {
         "tags": {
            "environment": "test"
         },
         "properties": {
            "desired": {}
         }
      }
      ```

   1. Vérifiez que **Activer l’entrée** est définie sur **Activer**.

   1. Sélectionnez **Enregistrer**.

Maintenant qu’une inscription existe pour cet appareil, le runtime IoT Edge peut provisionner automatiquement l’appareil lors de l’installation. Veillez à copier la valeur de **Clé primaire** de votre inscription pour l’utiliser lors de l’installation du runtime IoT Edge ou si vous comptez créer des clés d’appareil à utiliser avec une inscription de groupe.

## <a name="derive-a-device-key"></a>Dériver une clé d’appareil

> [!NOTE]
> Cette section n’est obligatoire que si vous utilisez une inscription de groupe.

Chaque appareil utilise sa clé d’appareil dérivée avec votre ID d’inscription unique pour effectuer l’attestation de clé symétrique avec l’inscription lors de l’approvisionnement. Pour générer la clé de l’appareil, utilisez la clé que vous avez copiée à partir de votre inscription au service Device Provisioning pour calculer un [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) de l’ID d’inscription unique pour l’appareil et convertir le résultat au format Base64.

N’incluez pas la clé primaire ou secondaire de votre inscription dans le code de votre appareil.

### <a name="linux-workstations"></a>Stations de travail Linux

Si vous utilisez une station de travail Linux, vous pouvez utiliser openssl pour générer votre clé d’appareil dérivée, comme indiqué dans l’exemple suivant.

Remplacez la valeur **KEY** par la **clé primaire** que vous avez notée précédemment.

Remplacez la valeur de **REG_ID** avec l’ID d’inscription de votre appareil.

```bash
KEY=8isrFI1sGsIlvvFSSFRiMfCNzv21fjbE/+ah/lSh3lF8e2YG1Te7w1KpZhJFFXJrqYKi9yegxkqIChbqOS9Egw==
REG_ID=sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
echo -n $REG_ID | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64
```

```bash
Jsm0lyGpjaVYVP2g3FnmnmG9dI/9qU24wNoykUmermc=
```

### <a name="windows-based-workstations"></a>Stations de travail Windows

Si vous utilisez une station de travail Windows, utilisez PowerShell pour générer votre clé d’appareil dérivée, comme indiqué dans l’exemple suivant.

Remplacez la valeur **KEY** par la **clé primaire** que vous avez notée précédemment.

Remplacez la valeur de **REG_ID** avec l’ID d’inscription de votre appareil.

```powershell
$KEY='8isrFI1sGsIlvvFSSFRiMfCNzv21fjbE/+ah/lSh3lF8e2YG1Te7w1KpZhJFFXJrqYKi9yegxkqIChbqOS9Egw=='
$REG_ID='sn-007-888-abc-mac-a1-b2-c3-d4-e5-f6'

$hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
$hmacsha256.key = [Convert]::FromBase64String($KEY)
$sig = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID))
$derivedkey = [Convert]::ToBase64String($sig)
echo "`n$derivedkey`n"
```

```powershell
Jsm0lyGpjaVYVP2g3FnmnmG9dI/9qU24wNoykUmermc=
```

## <a name="install-the-iot-edge-runtime"></a>Installer le runtime IoT Edge

Le runtime IoT Edge est déployé sur tous les appareils IoT Edge. Ses composants s’exécutent dans des conteneurs et vous permettent de déployer des conteneurs supplémentaires sur l’appareil, pour que vous puissiez exécuter du code en périphérie.

Suivez la procédure décrite dans [Installer le runtime Azure IoT Edge](how-to-install-iot-edge.md), puis revenez à cet article pour approvisionner l’appareil.

## <a name="configure-the-device-with-provisioning-information"></a>Configuration de l’appareil avec des informations d’approvisionnement

Une fois que le runtime est installé sur votre appareil, configurez l’appareil avec les informations qu’il utilise pour se connecter au service Device Provisioning (DPS) et à IoT Hub.

Préparez les informations suivantes :

* Valeur **Étendue de l’ID** du service Device Provisioning
* L’**ID d’inscription** de l’appareil que vous avez créé
* La valeur de **Clé primaire** que vous avez copiée lors de l’inscription DPS

> [!TIP]
> Pour les inscriptions de groupe, vous devez avoir la [clé dérivée](#derive-a-device-key) de chaque appareil au lieu de la clé d’inscription DPS.

### <a name="linux-device"></a>Appareil Linux

1. Ouvrez le fichier config sur l’appareil IoT Edge.

   ```bash
   sudo nano /etc/iotedge/config.yaml
   ```

1. Recherchez la section relative aux configurations d’approvisionnement dans le fichier. Supprimez les marques de commentaire pour les lignes de l’approvisionnement par clé symétrique de DPS et assurez-vous que les autres lignes d’approvisionnement sont commentées.

   La ligne `provisioning:` ne doit pas être précédée d’un espace blanc et les éléments imbriqués doivent être en retrait de deux espaces.

   ```yml
   # DPS TPM provisioning configuration
   provisioning:
     source: "dps"
     global_endpoint: "https://global.azure-devices-provisioning.net"
     scope_id: "<SCOPE_ID>"
     attestation:
       method: "symmetric_key"
       registration_id: "<REGISTRATION_ID>"
       symmetric_key: "<SYMMETRIC_KEY>"
   ```

1. Mettez à jour les valeurs de `scope_id`, `registration_id` et `symmetric_key` avec vos informations relatives à l’appareil et à DPS.

1. Redémarrez le runtime IoT Edge afin qu’il récupère toutes les modifications de configuration que vous avez effectuées sur l’appareil.

   ```bash
   sudo systemctl restart iotedge
   ```

### <a name="windows-device"></a>Appareil Windows

1. Ouvrez une fenêtre PowerShell en mode administrateur. Veillez à utiliser une session AMD64 de PowerShell lors de l'installation d'IoT Edge, plutôt que PowerShell (x86).

1. La commande **Initialize-IoTEdge** configure le runtime IoT Edge sur votre ordinateur. La commande opère par défaut un approvisionnement manuel avec les conteneurs Windows. Vous devez donc ajouter l’indicateur `-DpsSymmetricKey` pour utiliser l’approvisionnement automatique avec l’authentification par clé symétrique.

   Remplacez les valeurs d’espace réservé pour `{scope_id}`, `{registration_id}` et `{symmetric_key}` par les données que vous avez collectées précédemment.

   Ajoutez le paramètre `-ContainerOs Linux` si vous utilisez des conteneurs Linux sur Windows.

   ```powershell
   . {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
   Initialize-IoTEdge -DpsSymmetricKey -ScopeId {scope ID} -RegistrationId {registration ID} -SymmetricKey {symmetric key}
   ```

## <a name="verify-successful-installation"></a>Vérifier la réussite de l’installation

Si le runtime a démarré correctement, vous pouvez accéder à votre IoT Hub et commencer à déployer des modules IoT Edge sur votre appareil. Utilisez les commandes suivantes sur votre appareil pour vérifier que le runtime a été installé et démarré correctement.

### <a name="linux-device"></a>Appareil Linux

Vérifiez l’état du service IoT Edge.

```cmd/sh
systemctl status iotedge
```

Consultez les journaux d’activité de service.

```cmd/sh
journalctl -u iotedge --no-pager --no-full
```

Répertoriez les modules en cours d’exécution.

```cmd/sh
iotedge list
```

### <a name="windows-device"></a>Appareil Windows

Vérifiez l’état du service IoT Edge.

```powershell
Get-Service iotedge
```

Consultez les journaux d’activité de service.

```powershell
. {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; Get-IoTEdgeLog
```

Répertoriez les modules en cours d’exécution.

```powershell
iotedge list
```

Vous pouvez vérifier que l’inscription individuelle que vous avez créée dans le service Device Provisioning a été utilisée. Dans le Portail Azure, accédez à l’instance du service Device Provisioning. Ouvrez les détails de l’inscription pour l’inscription individuelle que vous avez créée. Notez que l’état de l’inscription est **Affecté** et que l’ID de l’appareil figure dans la liste.

## <a name="next-steps"></a>Étapes suivantes

Le processus d’inscription auprès du service Device Provisioning vous permet de définir l’ID d’appareil et les balises du jumeau d’appareil en même temps que vous provisionnez le nouvel appareil. Vous pouvez utiliser ces valeurs pour cibler des appareils individuels ou des groupes d’appareils avec la gestion d’appareils automatique. En savoir plus sur [Déployer et surveiller des modules IoT Edge à grande échelle à l’aide du portail Azure](how-to-deploy-at-scale.md) ou [d’Azure CLI](how-to-deploy-cli-at-scale.md).