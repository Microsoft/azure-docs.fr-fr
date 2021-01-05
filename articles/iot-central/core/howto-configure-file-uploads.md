---
title: Charger les fichiers d’un appareil sur Stockage Azure | Microsoft Docs
description: Comment configurer les chargements de fichiers de vos appareils vers le cloud. Une fois que vous avez configuré les chargements de fichiers, implémentez-les sur vos appareils.
services: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 12/23/2020
ms.topic: how-to
ms.service: iot-central
ms.openlocfilehash: ed2eff4d6ccbb9f85dfaf7049fa3cc18711bae0f
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97796902"
---
# <a name="upload-files-from-your-devices-to-the-cloud"></a>Charger des fichiers de vos appareils vers le cloud

*Cette rubrique s’applique aux administrateurs et aux développeurs d’appareils.*

IoT Central vous permet de charger des médias et d’autres fichiers à partir d’appareils connectés et vers le stockage cloud. Vous configurez la capacité de chargement de fichiers dans votre application IoT Central, puis vous implémentez les chargements de fichiers dans le code de votre appareil.

## <a name="prerequisites"></a>Prérequis

Vous devez être administrateur dans votre application IoT Central pour configurer les chargements de fichiers.

Vous avez besoin d’un compte et d’un conteneur de stockage Azure pour stocker les fichiers chargés. Si vous n’avez pas de compte de stockage ni de conteneur à utiliser, créez un [compte de stockage dans le portail Azure](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM).

## <a name="configure-device-file-uploads"></a>Configurer les chargements de fichiers d’appareil

Pour configurer les chargements de fichiers d’appareil :

1. Accédez à la section **Administration** de votre application.

1. Sélectionnez **Chargement de fichier d’appareil**.

1. Sélectionnez le compte de stockage et le conteneur à utiliser. Si le compte de stockage se trouve dans un abonnement Azure différent de votre application, entrez une chaîne de connexion au compte de stockage.

1. Si nécessaire, ajustez le délai d’expiration du chargement qui définit la durée de validité d’une requête de chargement. Les valeurs possibles sont comprises entre 1 et 24 heures.

1. Sélectionnez **Enregistrer**. Lorsque l’état indique **Configuré**, vous êtes prêt à charger des fichiers à partir d’appareils.

:::image type="content" source="media/howto-configure-file-uploads/file-upload-configuration.png" alt-text="Configurer le chargement de fichiers dans l’application":::

## <a name="disable-device-file-uploads"></a>Désactiver les chargements de fichiers d’appareil

Si vous souhaitez désactiver les chargements de fichiers d’appareil dans votre application IoT Central :

1. Accédez à la section **Administration** de votre application.

1. Sélectionnez **Chargement de fichier d’appareil**.

1. Sélectionnez **Supprimer**.

## <a name="upload-a-file-from-a-device"></a>Charger un fichier à partir d’un appareil

IoT Central utilise la capacité de chargement de fichiers d’IoT Hub pour permettre aux appareils de charger des fichiers. Pour obtenir un exemple de code qui vous montre comment charger des fichiers à partir d’un appareil, consultez l’[exemple de chargement de fichiers d’appareil avec IoT Central](/samples/iot-for-all/iotc-file-upload-device/iotc-file-upload-device/).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment configurer et implémenter les chargements de fichiers d’appareil dans IoT Central, la prochaine étape consiste à en savoir plus sur d’autres chargements de fichiers d’appareil :

- [Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (.NET)](../../iot-hub/iot-hub-csharp-csharp-file-upload.md)
- [Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (Java)](../../iot-hub/iot-hub-java-java-file-upload.md)
- [Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (Node.js)](../../iot-hub/iot-hub-node-node-file-upload.md)
- [Charger des fichiers sur le cloud à partir d’un appareil avec IoT Hub (Python)](../../iot-hub/iot-hub-python-python-file-upload.md)