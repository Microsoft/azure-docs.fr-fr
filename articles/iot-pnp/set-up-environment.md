---
title: Configurer les ressources IoT nécessaires pour IoT Plug-and-Play | Microsoft Docs
description: Créez un hub IoT et une instance du service Device Provisioning à utiliser avec les tutoriels et les guides de démarrage rapide IoT Plug-and-Play.
author: dominicbetts
ms.author: dobett
ms.date: 08/11/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 6e6c090c02798103a30cc68f2ca28e8c4f7f2f17
ms.sourcegitcommit: b8eba4e733ace4eb6d33cc2c59456f550218b234
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "96004260"
---
# <a name="set-up-your-environment-for-the-iot-plug-and-play-quickstarts-and-tutorials"></a>Configurer votre environnement pour les tutoriels et les guides de démarrage rapide IoT Plug-and-Play

Avant de pouvoir suivre des tutoriels et guides de démarrage rapide IoT Plug-and-Play, vous devez configurer un hub IoT et le service Device Provisioning (DPS) dans votre abonnement Azure. Vous aurez également besoin de copies locales des fichiers de modèle utilisés par les exemples d’applications et l’outil Explorateur Azure IoT.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Pour éviter d’avoir à installer Azure CLI localement, vous pouvez configurer les services cloud à l’aide d’Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resources"></a>Créer les ressources

Créez un groupe de ressources Azure pour les ressources :

```azurecli-interactive
az group create --name my-pnp-resourcegroup --location centralus
```

Créez un hub IoT. La commande suivante utilise le nom `my-pnp-hub` comme exemple de nom du hub IoT à créer. Remplacez `my-pnp-hub` par un nom unique de votre choix pour votre hub IoT :

```azurecli-interactive
az iot hub create --name my-pnp-hub --resource-group my-pnp-resourcegroup --sku F1 --partition-count 2
```

Créez une instance du service Device Provisioning (DPS). La commande suivante utilise le nom `my-pnp-dps` comme exemple de nom de l’instance DPS à créer. Remplacez `my-pnp-dps` par un nom unique de votre choix pour votre instance DPS :

```azurecli-interactive
az iot dps create --name my-pnp-dps --resource-group my-pnp-resourcegroup
```

Pour lier l’instance DPS à votre hub IoT, utilisez les commandes suivantes. Remplacez `my-pnp-dps` et `my-pnp-hub` par les noms uniques choisis précédemment :

```azurecli-interactive
hubConnectionString=$(az iot hub connection-string show -n my-pnp-hub --key primary --query connectionString -o tsv)
az iot dps linked-hub create --dps-name my-pnp-dps --resource-group my-pnp-resourcegroup --location centralus --connection-string $hubConnectionString
```

## <a name="retrieve-the-settings"></a>Récupérer les paramètres

Certains guides de démarrage rapide et tutoriels utilisent la chaîne de connexion pour votre hub IoT. Vous avez également besoin de la chaîne de connexion pour configurer l’outil Explorateur Azure IoT. Récupérez la chaîne de connexion et notez-la. Remplacez `my-pnp-hub` par le nom unique choisi pour votre hub IoT :

```azurecli-interactive
az iot hub connection-string show -n my-pnp-hub --key primary --query connectionString
```

La plupart des guides de démarrage rapide et des tutoriels utilisent l’*Étendue de l’ID* configurée pour le service Device Provisioning. Récupérez la valeur de l’étendue de l’ID et notez-la. Remplacez `my-pnp-dps` par le nom unique choisi pour votre instance DPS :

```azurecli-interactive
az iot dps show --name my-pnp-dps --query properties.idScope
```

Tous les guides de démarrage rapide et les tutoriels utilisent une inscription d’appareil DPS. Utilisez la commande suivante pour créer une *inscription d’appareil individuel* `my-pnp-device` dans votre instance DPS. Remplacez `my-pnp-dps` par le nom unique choisi pour votre instance DPS. Notez l’ID de l’inscription et la valeur de clé primaire à utiliser dans les tutoriels et les guides de démarrage rapide :

```azurecli-interactive
az iot dps enrollment create --attestation-type symmetrickey --dps-name my-pnp-dps --resource-group my-pnp-resourcegroup --enrollment-id my-pnp-device --device-id my-pnp-device --query '{registrationID:registrationId,primaryKey:attestation.symmetricKey.primaryKey}'
```

## <a name="create-environment-variables"></a>Créer des variables d’environnement

Créez cinq variables d’environnement pour configurer les exemples dans les tutoriels et les guides de démarrage rapide afin qu’ils utilisent le service Device Provisioning (DPS) pour se connecter à votre hub IoT :

* **IOTHUB_DEVICE_SECURITY_TYPE** : la valeur `DPS`.
* **IOTHUB_DEVICE_DPS_ID_SCOPE** : l’étendue de l’ID du service DPS que vous avez notée précédemment.
* **IOTHUB_DEVICE_DPS_DEVICE_ID** : la valeur `my-pnp-device`.
* **IOTHUB_DEVICE_DPS_DEVICE_KEY** : la clé primaire d’inscription que vous avez notée précédemment.
* **IOTHUB_DEVICE_DPS_ENDPOINT** : la valeur `global.azure-devices-provisioning.net`

Les exemples du service nécessitent les variables d’environnement suivantes pour identifier le hub et l’appareil auxquels se connecter :

* **IOTHUB_CONNECTION_STRING** : la chaîne de connexion de hub IoT que vous avez notée précédemment.
* **IOTHUB_DEVICE_ID** : `my-pnp-device`.

Par exemple, dans un interpréteur de commandes Bash Linux :

```bash
export IOTHUB_DEVICE_SECURITY_TYPE="DPS"
export IOTHUB_DEVICE_DPS_ID_SCOPE="<Your ID scope>"
export IOTHUB_DEVICE_DPS_DEVICE_ID="my-pnp-device"
export IOTHUB_DEVICE_DPS_DEVICE_KEY="<Your enrolment primary key>"
export IOTHUB_DEVICE_DPS_ENDPOINT="global.azure-devices-provisioning.net"
export IOTHUB_CONNECTION_STRING="<Your IoT hub connection string>"
export IOTHUB_DEVICE_ID="my-pnp-device"
```

Par exemple, sur la ligne de commande Windows :

```cmd
set IOTHUB_DEVICE_SECURITY_TYPE=DPS
set IOTHUB_DEVICE_DPS_ID_SCOPE=<Your ID scope>
set IOTHUB_DEVICE_DPS_DEVICE_ID=my-pnp-device
set IOTHUB_DEVICE_DPS_DEVICE_KEY=<Your enrolment primary key>
set IOTHUB_DEVICE_DPS_ENDPOINT=global.azure-devices-provisioning.net
set IOTHUB_CONNECTION_STRING=<Your IoT hub connection string>
set IOTHUB_DEVICE_ID=my-pnp-device
```

## <a name="download-the-model-files"></a>Télécharger les fichiers de modèle

Les guides de démarrage rapide et les tutoriels utilisent des exemples de fichiers de modèle pour les appareils à thermostat ou de contrôle de température. Pour télécharger les exemples de fichiers de modèle :

1. Créez un dossier nommé *models* sur votre ordinateur local.

1. Cliquez avec le bouton droit sur [TemperatureController.json](https://raw.githubusercontent.com/Azure/opendigitaltwins-dtdl/master/DTDL/v2/samples/TemperatureController.json) et enregistrez le fichier JSON dans le dossier *models*.

1. Cliquez avec le bouton droit sur [Thermostat.json](https://raw.githubusercontent.com/Azure/opendigitaltwins-dtdl/master/DTDL/v2/samples/Thermostat.json) et enregistrez le fichier JSON dans le dossier *models*.

## <a name="install-the-azure-iot-explorer"></a>Installer l’explorateur Azure IoT

Les guides de démarrage rapide et les tutoriels utilisent l’outil **Explorateur Azure IoT**. Accédez aux [versions de l’explorateur Azure IOT](https://github.com/Azure/azure-iot-explorer/releases) et développez la liste des ressources pour la version la plus récente. Téléchargez et installez la version la plus récente de l’application pour votre système d’exploitation.

La première fois que vous exécutez l’outil, vous êtes invité à entrer la chaîne de connexion du hub IoT. Utilisez la chaîne de connexion que vous avez notée précédemment.

Configurez l’outil pour utiliser les fichiers de modèle précédemment téléchargés. Dans la page d’accueil de l’outil, sélectionnez **Paramètres IoT Plug-and-Play**, puis **+ Ajouter > Dossier local**. Sélectionnez le dossier de *modèles* que vous avez créé précédemment. Sélectionnez ensuite **Enregistrer** pour enregistrer les paramètres.

Pour plus d’informations, consultez [Installer et utiliser l’explorateur Azure IoT](howto-use-iot-explorer.md).

## <a name="remove-the-resources"></a>Supprimer les ressources

Vous pouvez utiliser le hub IoT et l’instance DPS pour l’ensemble des tutoriels et des guides de démarrage rapide IoT Plug-and-Play. Ainsi, il vous suffit d’effectuer les étapes de cet article une seule fois. Quand vous avez terminé, vous pouvez supprimer les ressources de votre abonnement à l’aide de la commande suivante :

```azurecli-interactive
az group delete --name my-pnp-resourcegroup
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré votre environnement, vous pouvez essayer l’un des guides de démarrage rapide ou tutoriels, comme celui-ci :

> [!div class="nextstepaction"]
> [Connecter un exemple d’application pour appareil IoT Plug-and-Play à IoT Hub](quickstart-connect-device.md)
