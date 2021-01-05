---
title: Utilisation de Azure PowerShell pour configurer le chargement du fichier | Microsoft Docs
description: Comment utiliser les applets de commande Azure PowerShell pour configurer votre IoT Hub afin d’activer les téléchargements de fichiers à partir d’appareils connectés. Comprend des informations sur la configuration du compte de stockage Azure de destination.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: robinsh
ms.openlocfilehash: e2b106e64aed08b0586575d4d77602329454a673
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536009"
---
# <a name="configure-iot-hub-file-uploads-using-powershell"></a>Configurer les chargements de fichiers IoT Hub à l’aide de Powershell

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

Pour utiliser la [fonctionnalité de chargement de fichiers dans IoT Hub](iot-hub-devguide-file-upload.md), vous devez d’abord associer un compte de stockage Azure à votre IoT Hub. Vous pouvez utiliser un compte de stockage existant ou en créer un nouveau.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.

* [Cmdlets Azure PowerShell](/powershell/azure/install-Az-ps).

* Un IoT Hub Azure. Si vous n’avez pas de IoT Hub, vous pouvez utiliser [la cmdlet New-AzIoTHub](/powershell/module/az.iothub/new-aziothub) afin d’en créer un ou utiliser le portail pour [créer un IoT Hub](iot-hub-create-through-portal.md).

* Un compte de stockage Azure. Si vous n’avez pas de compte de stockage Azure, vous pouvez utiliser [les cmdlets PowerShell de stockage Azure](/powershell/module/az.storage/) afin d’en créer un ou utiliser le portail pour [créer un compte de stockage](../storage/common/storage-account-create.md).

## <a name="sign-in-and-set-your-azure-account"></a>Se connecter à votre compte Azure et le définir

Vous connecter à votre compte Azure et sélectionner votre abonnement.

1. À l’invite PowerShell, exécutez la cmdlet **Connect-AzAccount**  :

    ```powershell
    Connect-AzAccount
    ```

2. Si vous possédez plusieurs abonnements Azure, la connexion à Azure vous donne accès à tous les abonnements Azure associés à vos informations d’identification. Utilisez la commande suivante pour répertorier les abonnements Azure que vous pouvez utiliser :

    ```powershell
    Get-AzSubscription
    ```

    Utilisez la commande suivante pour sélectionner l’abonnement que vous souhaitez utiliser afin d’exécuter les commandes pour gérer votre IoT Hub. Vous pouvez utiliser le nom de l’abonnement ou l’ID de la sortie de la commande précédente :

    ```powershell
    Select-AzSubscription `
        -SubscriptionName "{your subscription name}"
    ```

## <a name="retrieve-your-storage-account-details"></a>Récupérez vos détails du compte de stockage

Les étapes suivantes supposent que vous avez créé votre compte de stockage à l’aide du modèle de déploiement de **Resource Manager** et non à partir du modèle de déploiement **classique**.

Pour configurer les chargements de fichiers en provenance de vos appareils, vous avez besoin de la chaîne de connexion d’un compte de stockage Azure. Le compte de stockage doit être situé dans le même abonnement que votre IoT Hub. Vous avez également besoin du nom d’un conteneur d’objets blob dans le compte de stockage. Utilisez la commande suivante pour récupérer vos clés de compte de stockage :

```powershell
Get-AzStorageAccountKey `
  -Name {your storage account name} `
  -ResourceGroupName {your storage account resource group}
```

Prenez note de la valeur de clé de compte de stockage **key1** . Vous en aurez besoin dans les étapes suivantes.

Vous pouvez utiliser un conteneur d’objets blob existant pour les chargements de fichiers ou en créer un nouveau :

* Pour répertorier les conteneurs d’objets blob existants dans votre compte de stockage, utilisez les commandes suivantes :

    ```powershell
    $ctx = New-AzStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    Get-AzStorageContainer -Context $ctx
    ```

* Pour créer un conteneur d’objet blob dans votre compte de stockage, utilisez les commandes suivantes :

    ```powershell
    $ctx = New-AzStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    New-AzStorageContainer `
        -Name {your new container name} `
        -Permission Off `
        -Context $ctx
    ```

## <a name="configure-your-iot-hub"></a>Configuration de votre IoT Hub

Vous pouvez désormais configurer votre IoT Hub pour [charger des fichiers vers le IoT Hub](iot-hub-devguide-file-upload.md) à l’aide des informations de votre compte de stockage.

La configuration requiert les valeurs suivantes :

* **Conteneur de stockage**  : conteneur d’objets blob dans un compte de stockage Azure de votre abonnement Azure actuel à associer à votre hub IoT. Vous avez extrait les informations de compte de stockage nécessaires dans la section précédente. IoT Hub génère automatiquement des URI SAS avec des autorisations d’écriture pour ce conteneur d’objets blob pour les appareils à utiliser lorsqu’ils chargent des fichiers.

* **Recevoir des notifications pour les fichiers chargés**  : activez ou désactivez les notifications de chargement de fichiers.

* **Durée de vie de SAS**  : ce paramètre est la durée de vie des URI de signature d’accès partagé retournés à l’appareil par IoT Hub. Défini sur 1 heure par défaut.

* **Durée de vie par défaut des paramètres de notification de fichiers**  : durée de vie d’une notification de chargement de fichier avant son expiration. Défini sur 1 jour par défaut.

* **Nombre maximal de remises de notifications de fichier**  : nombre de tentatives de remise d’une notification de chargement de fichier par le hub IoT. Défini sur 10 par défaut.

Utilisez l’applet de commande PowerShell suivant pour configurer les paramètres de chargement sur votre IoT Hub :

```powershell
Set-AzIotHub `
    -ResourceGroupName "{your iot hub resource group}" `
    -Name "{your iot hub name}" `
    -FileUploadNotificationTtl "01:00:00" `
    -FileUploadSasUriTtl "01:00:00" `
    -EnableFileUploadNotifications $true `
    -FileUploadStorageConnectionString "DefaultEndpointsProtocol=https;AccountName={your storage account name};AccountKey={your storage account key};EndpointSuffix=core.windows.net" `
    -FileUploadContainerName "{your blob container name}" `
    -FileUploadNotificationMaxDeliveryCount 10
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les fonctionnalités de chargement des fichiers dans IoT Hub, consultez [Charger des fichiers à partir d’un appareil](iot-hub-devguide-file-upload.md).

Suivez ces liens pour en savoir plus sur la gestion de Azure IoT Hub :

* [Gestion en bloc des appareils IoT](iot-hub-bulk-identity-mgmt.md)
* [Surveiller IoT Hub](monitor-iot-hub.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Guide du développeur d’IoT Hub](iot-hub-devguide.md)
* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)
* [Sécuriser votre solution IoT de bout en bout](../iot-fundamentals/iot-security-ground-up.md)