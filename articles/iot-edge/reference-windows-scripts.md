---
title: Scripts Windows pour Azure IoT Edge | Microsoft Docs
description: Informations de référence sur les scripts IoT Edge PowerShell pour installer, désinstaller ou mettre à jour sur des appareils Windows
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 10/06/2020
ms.topic: reference
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: a082ccb62103ab5bd027bf49b9ee05bc48c63115
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91979192"
---
# <a name="powershell-scripts-for-iot-edge-on-windows"></a>Scripts PowerShell pour IoT Edge sur Windows

Découvrez les scripts PowerShell qui installent, mettent à jour ou désinstallent IoT Edge sur des appareils Windows.

Les commandes décrites dans cet article proviennent du fichier `IoTEdgeSecurityDaemon.ps1` qui est publié avec chaque [version d’IoT Edge](https://github.com/Azure/azure-iotedge/releases). La version la plus récente du script est toujours accessible via l’adresse aka.ms/iotedge-win.

Vous pouvez exécuter n’importe quelle commande à l’aide de la cmdlet `Invoke-WebRequest` pour accéder à la dernière version du script. Par exemple :

```powershell
. {Invoke-WebRequest -useb https://aka.ms/iotedge-win} | Invoke-Expression; `
Deploy-IoTEdge
```

Vous pouvez également télécharger ce script ou la version du script d’une version spécifique afin d’exécuter les commandes. Par exemple :

```powershell
. <path>\IoTEdgeSecurityDaemon.ps1
Deploy-IoTEdge
```

Le script fourni est signé pour augmenter la sécurité. Vous pouvez vérifier la signature en téléchargeant le script sur votre appareil, puis en exécutant la commande PowerShell suivante :

```powershell
Get-AuthenticodeSignature "C:\<path>\IotEdgeSecurityDaemon.ps1"
```

L’état de sortie est **Valide** si la signature est vérifiée.

## <a name="deploy-iotedge"></a>Deploy-IoTEdge

La commande Deploy-IoTEdge télécharge et déploie le démon de sécurité IoT Edge et ses dépendances. La commande de déploiement accepte ces paramètres communs, entre autres. Pour obtenir la liste complète, utilisez la commande `Get-Help Deploy-IoTEdge -full`.  

| Paramètre | Valeurs acceptées | Commentaires |
| --------- | --------------- | -------- |
| **ContainerOs** | **Windows** ou **Linux** | Si aucun système d’exploitation de conteneur n’est spécifié, Windows est la valeur par défaut.<br><br>Pour des conteneurs Windows, IoT Edge utilise le moteur de conteneur Moby inclus dans l’installation. Pour des conteneurs Linux, vous devez installer un moteur de conteneur avant de commencer l’installation. |
| **Proxy** | URL du proxy | Incluez ce paramètre si votre appareil doit passer par un serveur proxy pour accéder à internet. Pour plus d’informations, voir [Configurer un appareil IoT Edge de façon à ce qu’il communique via un serveur proxy](how-to-configure-proxy-support.md). |
| **OfflineInstallationPath** | Chemin du répertoire | Si ce paramètre est inclus, le programme d’installation vérifie la présence dans le répertoire listé des fichiers cab IoT Edge et MSI du Runtime VC. Tous les fichiers introuvables dans le répertoire sont téléchargés. Si les deux fichiers sont dans le répertoire, vous pouvez installer IoT Edge sans connexion Internet. Vous pouvez également utiliser ce paramètre pour utiliser une version spécifique. |
| **InvokeWebRequestParameters** | Table de hachage de paramètres et valeurs | Pendant l’installation, plusieurs requêtes web sont effectuées. Ce champ permet de définir des paramètres pour ces requêtes HTTP. Ce paramètre est utile pour configurer des informations d’identification pour des serveurs proxy. Pour plus d’informations, voir [Configurer un appareil IoT Edge de façon à ce qu’il communique via un serveur proxy](how-to-configure-proxy-support.md). |
| **RestartIfNeeded** | Aucun | Cet indicateur permet au script de déploiement de redémarrer l’ordinateur sans invite, si nécessaire. |

## <a name="initialize-iotedge"></a>Initialize-IoTEdge

La commande Initialize-IoTEdge configure IoT Edge avec votre chaîne de connexion d’appareil et les détails opérationnels. La plupart des informations générées par cette commande sont ensuite stockées dans le fichier iotedge\config.yaml. La commande d’initialisation accepte ces paramètres communs, entre autres. Pour obtenir la liste complète, utilisez la commande `Get-Help Initialize-IoTEdge -full`.

| Paramètre | Valeurs acceptées | Commentaires |
| --------- | --------------- | -------- |
| **ManualConnectionString** | None | **Paramètre booléen**. **Valeur par défaut**. Si aucun type d’approvisionnement n’est spécifié, la valeur par défaut est l’approvisionnement manuel avec une chaîne de connexion.<br><br>Déclare que vous allez fournir une chaîne de connexion d’appareil pour approvisionner l’appareil manuellement. |
| **ManualX509** | None | **Paramètre booléen**. Si aucun type d’approvisionnement n’est spécifié, la valeur par défaut est l’approvisionnement manuel avec une chaîne de connexion.<br><br>Déclare que vous allez fournir un certificat d’identité et une clé privée pour approvisionner l’appareil manuellement.
| **DpsTpm** | None | **Paramètre booléen**. Si aucun type d’approvisionnement n’est spécifié, la valeur par défaut est l’approvisionnement manuel avec une chaîne de connexion.<br><br>Déclare que vous allez fournir l’ID de l’étendue d’un Service Device Provisioning (DPS) et l’ID d’inscription de votre appareil à provisionner via DPS.  |
| **DpsSymmetricKey** | None | **Paramètre booléen**. Si aucun type d’approvisionnement n’est spécifié, la valeur par défaut est l’approvisionnement manuel avec une chaîne de connexion.<br><br>Déclare que vous allez fournir l’ID d’étendue d’un Service Device Provisioning (DPS) et l’ID d’inscription de votre appareil à approvisionner via DPS, ainsi qu’une clé symétrique pour l’attestation. |
| **DpsX509** | None | **Paramètre booléen**. Si aucun type d’approvisionnement n’est spécifié, la valeur par défaut est l’approvisionnement manuel avec une chaîne de connexion.<br><br>Déclare que vous allez fournir l’ID d’étendue d’un Service Device Provisioning (DPS) et l’ID d’inscription de votre appareil à approvisionner via DPS, ainsi qu’un certificat d’identité X.509 et une clé privée pour l’attestation.  |
| **DeviceConnectionString** | Chaîne de connexion d’un appareil IoT Edge inscrit dans un IoT Hub, entre apostrophes | **Obligatoire** pour l’approvisionnement manuel avec une chaîne de connexion. Si vous ne fournissez pas de chaîne de connexion dans les paramètres de script, vous serez invité à la spécifier. |
| **IotHubHostName** | Nom d’hôte du hub IoT auquel un appareil se connecte. | **Obligatoire** pour l’approvisionnement manuel avec des certificats X.509. Prend le format de *{hub name}.azure-devices.net*. |
| **DeviceId** | ID d’appareil provenant d’une identité d’appareil inscrite dans IoT Hub. | **Obligatoire** pour l’approvisionnement manuel avec des certificats X.509. |
| **ScopeId** | ID de l’étendue d’une instance de Service Device Provisioning associée à votre IoT Hub. | **Obligatoire** requis pour l’approvisionnement DPS. Si vous ne fournissez pas d’ID d’étendue dans les paramètres de script, vous serez invité à le spécifier durant l’installation. |
| **RegistrationID** | ID d’inscription généré par votre appareil | **Obligatoire** pour l’approvisionnement DPS si vous utilisez TPM ou une attestation de clé symétrique. **Facultatif** si vous utilisez une attestation de certificat X.509. |
| **X509IdentityCertificate** | Chemin d’accès de l’URI du certificat d’identité de l’appareil X.509 sur l’appareil. | **Obligatoire** pour l’approvisionnement manuel ou DPS si vous utilisez l’attestation de certificat X.509. |
| **X509IdentityPrivateKey** | Chemin d’accès de l’URI à la clé de certificat d’identité de l’appareil X.509 sur l’appareil. | **Obligatoire** pour l’approvisionnement manuel ou DPS si vous utilisez l’attestation de certificat X.509. |
| **SymmetricKey** | Clé symétrique utilisée pour provisionner l’identité de l’appareil IoT Edge lors de l’utilisation de DPS | **Obligatoire** pour l’approvisionnement DPS si vous utilisez une attestation de clé symétrique. |
| **ContainerOs** | **Windows** ou **Linux** | Si aucun système d’exploitation de conteneur n’est spécifié, Windows est la valeur par défaut.<br><br>Pour des conteneurs Windows, IoT Edge utilise le moteur de conteneur Moby inclus dans l’installation. Pour des conteneurs Linux, vous devez installer un moteur de conteneur avant de commencer l’installation. |
| **DeviceCACertificate** | Chemin d’accès de l’URI au certificat d’autorité de certification d’appareil X.509 sur l’appareil. | Peut également être configuré dans le fichier `C:\ProgramData\iotedge\config.yaml`. Pour plus d’informations, consultez [Gérer des certificats sur un appareil IoT Edge](how-to-manage-device-certificates.md). |
| **DeviceCAPrivateKey** | Chemin d’accès de l’URI à la clé privée d’autorité de certification d’appareil X.509 sur l’appareil. | Peut également être configuré dans le fichier `C:\ProgramData\iotedge\config.yaml`. Pour plus d’informations, consultez [Gérer des certificats sur un appareil IoT Edge](how-to-manage-device-certificates.md). |
| **InvokeWebRequestParameters** | Table de hachage de paramètres et valeurs | Pendant l’installation, plusieurs requêtes web sont effectuées. Ce champ permet de définir des paramètres pour ces requêtes HTTP. Ce paramètre est utile pour configurer des informations d’identification pour des serveurs proxy. Pour plus d’informations, voir [Configurer un appareil IoT Edge de façon à ce qu’il communique via un serveur proxy](how-to-configure-proxy-support.md). |
| **AgentImage** | URI d’image de l’agent IoT Edge | Par défaut, une nouvelle installation d’IoT Edge utilise la dernière balise de déploiement pour l’image d’agent IoT Edge. Ce paramètre permet de définir une balise spécifique pour la version d’image, ou de fournir votre propre image d’agent. Pour plus d’informations, voir [Comprendre les balises IoT Edge](how-to-update-iot-edge.md#understand-iot-edge-tags). |
| **Nom d’utilisateur** | Nom d’utilisateur du registre de conteneurs | Utilisez ce paramètre uniquement si vous définissez le paramètre -AgentImage sur un conteneur dans un registre privé. Fournissez un nom d’utilisateur ayant accès au registre. |
| **Mot de passe** | Chaîne de mot de passe sécurisé | Utilisez ce paramètre uniquement si vous définissez le paramètre -AgentImage sur un conteneur dans un registre privé. Fournissez le mot de passe pour l’accès au registre. |

## <a name="update-iotedge"></a>Update-IoTEdge

| Paramètre | Valeurs acceptées | Commentaires |
| --------- | --------------- | -------- |
| **ContainerOs** | **Windows** ou **Linux** | Si aucun système d’exploitation de conteneur n’est spécifié, Windows est la valeur par défaut. Pour des conteneurs Windows, un moteur de conteneur est inclus dans l’installation. Pour des conteneurs Linux, vous devez installer un moteur de conteneur avant de commencer l’installation. |
| **Proxy** | URL du proxy | Incluez ce paramètre si votre appareil doit passer par un serveur proxy pour accéder à internet. Pour plus d’informations, voir [Configurer un appareil IoT Edge de façon à ce qu’il communique via un serveur proxy](how-to-configure-proxy-support.md). |
| **InvokeWebRequestParameters** | Table de hachage de paramètres et valeurs | Pendant l’installation, plusieurs requêtes web sont effectuées. Ce champ permet de définir des paramètres pour ces requêtes HTTP. Ce paramètre est utile pour configurer des informations d’identification pour des serveurs proxy. Pour plus d’informations, voir [Configurer un appareil IoT Edge de façon à ce qu’il communique via un serveur proxy](how-to-configure-proxy-support.md). |
| **OfflineInstallationPath** | Chemin du répertoire | Si ce paramètre est inclus, le programme d’installation vérifie la présence dans le répertoire listé des fichiers cab IoT Edge et MSI du Runtime VC. Tous les fichiers introuvables dans le répertoire sont téléchargés. Si les deux fichiers sont dans le répertoire, vous pouvez installer IoT Edge sans connexion Internet. Vous pouvez également utiliser ce paramètre pour utiliser une version spécifique. |
| **RestartIfNeeded** | Aucun | Cet indicateur permet au script de déploiement de redémarrer l’ordinateur sans invite, si nécessaire. |

## <a name="uninstall-iotedge"></a>Uninstall-IoTEdge

| Paramètre | Valeurs acceptées | Commentaires |
| --------- | --------------- | -------- |
| **Force** | Aucun | Cet indicateur force la désinstallation si la précédente tentative de désinstallation a échoué.
| **RestartIfNeeded** | Aucun | Cet indicateur permet au script de désinstallation de redémarrer l’ordinateur sans invite, si nécessaire. |

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment utiliser ces commandes dans les articles suivants :

* [Installer ou désinstaller le runtime Azure IoT Edge](how-to-install-iot-edge.md)
* [Approvisionner un appareil Azure IoT Edge avec une authentification par clé symétrique](how-to-manual-provision-symmetric-key.md)
* [Approvisionner un appareil Azure IoT Edge avec une authentification par certificat X.509](how-to-manual-provision-x509.md)
