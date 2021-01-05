---
title: Utilisation du portail Azure pour configurer le téléchargement du fichier | Microsoft Docs
description: Utilisation du portail Azure pour configurer votre IoT Hub en vue d’activer les transferts de fichiers à partir d’appareils connectés. Comprend des informations sur la configuration du compte de stockage Azure de destination.
author: robinsh
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/03/2017
ms.author: robinsh
ms.openlocfilehash: da28bfa31c74ff33a200967267500033dd6a9b1b
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92535873"
---
# <a name="configure-iot-hub-file-uploads-using-the-azure-portal"></a>Configurer les chargements de fichiers IoT Hub à l’aide du portail Azure

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

## <a name="file-upload"></a>Chargement de fichiers

Pour utiliser la [fonctionnalité de chargement de fichiers dans IoT Hub](iot-hub-devguide-file-upload.md), vous devez commencer par associer un compte de stockage Azure à votre Hub. Sélectionnez **Chargement de fichier** pour afficher une liste de propriétés de chargement de fichiers pour l’IoT Hub en cours de modification.

![Afficher les paramètres de chargement de fichier IoT Hub dans le portail](./media/iot-hub-configure-file-upload/file-upload-settings.png)

* **Conteneur de stockage**  : Utilisez le portail Azure pour sélectionner un conteneur d’objets blob dans un compte Azure Storage de votre abonnement Azure actuel à associer à votre IoT Hub. Si nécessaire, vous pouvez créer un compte Azure Storage dans le panneau **Comptes de stockage** et un conteneur d’objets blob dans le panneau **Conteneurs**. IoT Hub génère automatiquement des URI SAS avec des autorisations d’écriture pour ce conteneur d’objets blob pour les appareils à utiliser lorsqu’ils chargent des fichiers.

   ![Afficher les conteneurs de stockage pour le chargement de fichiers dans le portail](./media/iot-hub-configure-file-upload/file-upload-container-selection.png)

* **Recevoir des notifications pour les fichiers chargés**  : Activez ou désactivez les notifications de chargement de fichiers via le bouton bascule.

* **Durée de vie de SAS**  : ce paramètre est la durée de vie des URI de signature d’accès partagé retournés à l’appareil par IoT Hub. Il est défini par défaut sur une heure, mais il peut être personnalisé avec d’autres valeurs à l’aide du curseur.

* **Durée de vie par défaut des paramètres de notification de fichiers**  : durée de vie d’une notification de chargement de fichier avant son expiration. Il est défini par défaut sur un jour, mais il peut être personnalisé avec d’autres valeurs à l’aide du curseur.

* **Nombre maximal de remises de notifications de fichier**  : nombre de tentatives de remise d’une notification de chargement de fichier par le hub IoT. Il est défini par défaut sur 10, mais il peut être personnalisé avec d’autres valeurs à l’aide du curseur.

   ![Configurer le chargement de fichier IoT Hub dans le portail](./media/iot-hub-configure-file-upload/file-upload-selected-container.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les fonctionnalités de chargement de fichiers d’IoT Hub, consultez la section [Charger des fichiers à partir d’un appareil](iot-hub-devguide-file-upload.md) dans le guide du développeur IoT Hub.

Suivez ces liens pour en savoir plus sur la gestion de Azure IoT Hub :

* [Gestion en bloc des appareils IoT](iot-hub-bulk-identity-mgmt.md)
* [Surveiller IoT Hub](monitor-iot-hub.md)

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Guide du développeur d’IoT Hub](iot-hub-devguide.md)
* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge](../iot-edge/quickstart-linux.md)
* [Sécuriser votre solution IoT de bout en bout](../iot-fundamentals/iot-security-ground-up.md)