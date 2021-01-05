---
title: Configurer le chargement de fichiers vers IoT Hub à l’aide d’Azure CLI | Microsoft Docs
description: Découvrez comment configurer le chargement des fichiers vers Azure IoT Hub à l’aide de l’interface Azure CLI multiplateforme.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: robinsh
ms.openlocfilehash: 4dbda13ffe04e0a4214b24ccaca2b8103a39b9f2
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536060"
---
# <a name="configure-iot-hub-file-uploads-using-azure-cli"></a>Configurer les chargements de fichiers IoT Hub à l’aide d’Azure CLI

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

Pour [charger des fichiers à partir d’un appareil](iot-hub-devguide-file-upload.md), vous devez d’abord associer un compte de stockage Azure à votre hub IoT. Vous pouvez utiliser un compte de stockage existant ou en créer un nouveau.

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

* [Azure CLI](/cli/azure/install-azure-cli).

* Un IoT Hub Azure. Si vous n’avez pas de hub IoT, vous pouvez utiliser la [commande `az iot hub create`](/cli/azure/iot/hub#az-iot-hub-create) pour en créer un, ou vous pouvez [Créer un IoT Hub à l’aide du portail](iot-hub-create-through-portal.md).

* Un compte de stockage Azure. Si vous n’avez pas de compte Stockage Azure, vous pouvez utiliser Azure CLI pour en créer un. Pour plus d’informations, consultez la rubrique [Création d’un compte de stockage](../storage/common/storage-account-create.md) .

## <a name="sign-in-and-set-your-azure-account"></a>Se connecter à votre compte Azure et le définir

Vous connecter à votre compte Azure et sélectionner votre abonnement.

1. Dans l’invite de commande, exécutez la [commande login](/cli/azure/get-started-with-azure-cli) :

    ```azurecli
    az login
    ```

    Suivez les instructions pour vous authentifier à l’aide du code et vous connecter à votre compte Azure via un navigateur web.

2. Si vous possédez plusieurs abonnements Azure, la connexion à Azure vous donne accès à tous les abonnements Azure associés à vos informations d’identification. Utilisez la [commande pour répertorier les comptes Azure](/cli/azure/account) ci-dessous :

    ```azurecli
    az account list
    ```

    Utilisez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser afin d’exécuter les commandes pour créer votre hub IoT. Vous pouvez utiliser le nom de l’abonnement ou l’ID de la sortie de la commande précédente :

    ```azurecli
    az account set --subscription {your subscription name or id}
    ```

## <a name="retrieve-your-storage-account-details"></a>Récupérez vos détails du compte de stockage

Les étapes suivantes supposent que vous avez créé votre compte de stockage à l’aide du modèle de déploiement de **Resource Manager** et non à partir du modèle de déploiement **classique**.

Pour configurer les chargements de fichiers en provenance de vos appareils, vous avez besoin de la chaîne de connexion d’un compte de stockage Azure. Le compte de stockage doit être situé dans le même abonnement que votre IoT Hub. Vous avez également besoin du nom d’un conteneur d’objets blob dans le compte de stockage. Utilisez la commande suivante pour récupérer vos clés de compte de stockage :

```azurecli
az storage account show-connection-string --name {your storage account name} \
  --resource-group {your storage account resource group}
```

Notez le nom de la valeur **connectionString**. Vous en aurez besoin dans les étapes suivantes.

Pour le chargement des fichiers, vous pouvez soit utiliser un conteneur d’objets blob existant, soit en créer un nouveau :

* Pour répertorier les conteneurs d’objets blob existants dans votre compte de stockage, utilisez la commande suivante :

    ```azurecli
    az storage container list --connection-string "{your storage account connection string}"
    ```

* Pour créer un conteneur d’objet blob dans votre compte de stockage, utilisez la commande suivante :

    ```azurecli
    az storage container create --name {container name} \
      --connection-string "{your storage account connection string}"
    ```

## <a name="file-upload"></a>Chargement de fichiers

Vous pouvez désormais configurer votre hub IoT pour activer la [fonctionnalité de chargement des fichiers vers le hub IoT](iot-hub-devguide-file-upload.md) à l’aide des informations de votre compte de stockage.

La configuration requiert les valeurs suivantes :

* **Conteneur de stockage**  : Un conteneur d’objets blob dans un compte de stockage Azure de votre abonnement Azure actuel à associer à votre IoT Hub. Vous avez extrait les informations de compte de stockage nécessaires dans la section précédente. IoT Hub génère automatiquement des URI SAS avec des autorisations d’écriture pour ce conteneur d’objets blob pour les appareils à utiliser lorsqu’ils chargent des fichiers.

* **Recevoir des notifications pour les fichiers chargés**  : activez ou désactivez les notifications de chargement de fichiers.

* **SAS TTL** : ce paramètre est la durée de vie des URI de signature d’accès partagé renvoyés à l’appareil par IoT Hub. Défini sur 1 heure par défaut.

* **Durée de vie par défaut des paramètres de notification de fichiers** : la durée de vie d’une notification de chargement avant son expiration. Défini sur 1 jour par défaut.

* **Nombre maximal de remises de notifications de fichier** : le nombre de tentatives de remise d’une notification de chargement de fichier par l’IoT Hub. Défini sur 10 par défaut.

Utilisez les commandes Azure CLI suivantes pour configurer les paramètres de chargement sur votre IoT Hub :

<!--Robinsh this is out of date, add cloud powershell -->

Dans un shell Bash, utilisez :

```azurecli
az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.connectionString="{your storage account connection string}"

az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.containerName="{your storage container name}"

az iot hub update --name {your iot hub name} \
  --set properties.storageEndpoints.'$default'.sasTtlAsIso8601=PT1H0M0S

az iot hub update --name {your iot hub name} \
  --set properties.enableFileUploadNotifications=true

az iot hub update --name {your iot hub name} \
  --set properties.messagingEndpoints.fileNotifications.maxDeliveryCount=10

az iot hub update --name {your iot hub name} \
  --set properties.messagingEndpoints.fileNotifications.ttlAsIso8601=PT1H0M0S
```

Vous pouvez consulter le fichier de configuration de téléchargement sur votre IoT Hub à l’aide de la commande suivante :

```azurecli
az iot hub show --name {your iot hub name}
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les fonctionnalités de chargement des fichiers dans IoT Hub, consultez [Charger des fichiers à partir d’un appareil](iot-hub-devguide-file-upload.md).

Suivez ces liens pour en savoir plus sur la gestion de Azure IoT Hub :

* [Gestion en bloc des appareils IoT](iot-hub-bulk-identity-mgmt.md)
* [Surveiller votre hub IoT](monitor-iot-hub.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Guide du développeur d’IoT Hub](iot-hub-devguide.md)
* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)
* [Sécuriser votre solution IoT de bout en bout](../iot-fundamentals/iot-security-ground-up.md)