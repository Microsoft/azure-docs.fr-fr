---
title: Utiliser le portail Azure pour créer un IoT Hub | Microsoft Azure
description: Comment créer, gérer et supprimer des IoT Hubs Azure via le portail Azure. Inclut des informations sur les niveaux de tarification, l’évolutivité, la sécurité et la configuration de la messagerie.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 09/06/2018
ms.author: robinsh
ms.custom:
- 'Role: Cloud Development'
ms.openlocfilehash: 0a54d0c85902915d2ee62acd8a1d38b8db8b221c
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92536043"
---
# <a name="create-an-iot-hub-using-the-azure-portal"></a>Création d’un IoT Hub à l’aide du portail Azure

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Cet article explique comment créer et gérer des hubs IoT avec le [portail Azure](https://portal.azure.com).

Pour utiliser les étapes de ce tutoriel, vous devez disposer d’un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-an-iot-hub"></a>Créer un hub IoT

[!INCLUDE [iot-hub-include-create-hub](../../includes/iot-hub-include-create-hub.md)]

## <a name="change-the-settings-of-the-iot-hub"></a>Modifier les paramètres du hub IoT

Vous pouvez modifier les paramètres d’un hub IoT existant après sa création, dans le volet IoT Hub.

![Capture d’écran montrant les paramètres pour le hub IoT](./media/iot-hub-create-through-portal/iot-hub-settings-panel.png)

Voici quelques-unes des propriétés que vous pouvez définir pour un hub IoT :

**Tarification et mise à l’échelle**  : Vous pouvez utiliser cette propriété pour migrer vers un autre niveau ou pour définir le nombre d’unités IoT Hub. 

**Surveillance des opérations**  : Activez ou désactivez les différentes catégories de surveillance, comme la journalisation pour les événements liés à des messages appareil-à-cloud ou cloud-à-appareil.

**Filtre IP**  : Spécifiez une plage d’adresses IP qui seront acceptées ou rejetées par le hub IoT.

**Propriétés**  : Fournit la liste des propriétés que vous pouvez copier et utiliser ailleurs, comme l’ID de ressource, le groupe de ressources, l’emplacement, etc.

### <a name="shared-access-policies"></a>Stratégies d’accès partagé

Vous pouvez également afficher ou modifier la liste des stratégies d’accès partagé en cliquant sur **Stratégies d’accès partagé** dans la section **Paramètres** . Ces stratégies définissent les autorisations pour que les appareils et les services se connectent au hub IoT. 

Cliquez sur **Ajouter** pour ouvrir le panneau **Ajouter une stratégie d’accès partagé** .  Vous pouvez entre le nom de la nouvelle stratégie et les autorisations que vous voulez associer à celle-ci, comme illustré dans la figure suivante :

![Capture d’écran montrant l’ajout d’une stratégie d’accès partagé](./media/iot-hub-create-through-portal/iot-hub-add-shared-access-policy.png)

* Les stratégies **Lecture de registre** et **Écriture de registre** accordent les autorisations en lecture et en écriture au registre d’identité. Ces autorisations sont utilisées par les services cloud principaux pour gérer les identités des appareils. Le choix de l'option en écriture active automatiquement l'option en lecture.

* La stratégie de **connexion de service** autorise l’accès aux points de terminaison de service. Cette autorisation est utilisée par les services cloud principaux pour envoyer et recevoir des messages à partir d’appareils. Elle permet également de mettre à jour et lire les données de jumeaux d’appareil et de jumeaux de module.

* La stratégie de **connexion d’appareil** accorde des autorisations pour envoyer et recevoir des messages à l’aide des points de terminaison côté appareil de IoT Hub. Cette autorisation est utilisée par les appareils afin d’envoyer et de recevoir des messages à partir d’un hub IoT. Elle permet aussi de mettre à jour et lire les données de jumeaux d’appareil et de jumeaux de module, ainsi que de mettre à jour les fichiers.

Cliquez sur **Créer** pour ajouter la stratégie créée à la liste existante.

Pour plus d’informations sur les accès accordés par des autorisations spécifiques, consultez [Autorisations IoT Hub](./iot-hub-devguide-security.md#iot-hub-permissions).

## <a name="register-a-new-device-in-the-iot-hub"></a>Inscrire un nouvel appareil dans le hub IoT

[!INCLUDE [iot-hub-include-create-device](../../includes/iot-hub-include-create-device.md)]

## <a name="message-routing-for-an-iot-hub"></a>Routage des messages pour un hub IoT

Cliquez sur **Routage des messages** sous **Messagerie** pour afficher le volet Routage des messages, où vous définissez les routes et les points de terminaison personnalisés pour le hub. Le [routage des messages](iot-hub-devguide-messages-d2c.md) vous permet de gérer la façon dont les données sont envoyées depuis vos appareils à vos points de terminaison. La première étape consiste à ajouter une nouvelle route. Vous pouvez ensuite ajouter un point de terminaison existant à la route ou créer un point de terminaison d’un des types pris en charge, comme Stockage Blob. 

![Volet Routage des messages](./media/iot-hub-create-through-portal/iot-hub-message-routing.png)

### <a name="routes"></a>Itinéraires

Routes est le premier onglet du volet Routage des messages. Pour ajouter une nouvelle route, cliquez sur + **Ajouter** . Vous voyez l’écran suivant. 

![Capture d’écran montrant l’ajout d’un itinéraire](./media/iot-hub-create-through-portal/iot-hub-add-route-storage-endpoint.png)

Nommez votre itinéraire. Le nom de l’itinéraire doit être unique au sein de la liste des routes pour ce hub. 

Pour **Point de terminaison** , vous pouvez en sélectionner un dans la liste déroulante ou en ajouter un nouveau. Dans cet exemple, un compte de stockage et un conteneur sont déjà disponibles. Pour les ajouter comme point de terminaison, cliquez sur + **Ajouter** en regard de la liste déroulante Point de terminaison et sélectionnez **Stockage Blob** . L’écran suivant montre l’endroit où le compte de stockage et le conteneur sont spécifiés.

![Capture d’écran montrant l’ajout d’un point de terminaison de stockage pour la règle de routage](./media/iot-hub-create-through-portal/iot-hub-routing-add-storage-endpoint.png)

Cliquez sur **Choisir un conteneur** pour sélectionner le compte de stockage et le conteneur. Une fois que vous avez sélectionné ces champs, vous revenez au volet Point de terminaison. Utilisez les valeurs par défaut pour le reste des champs, puis sélectionnez **Créer** pour créer le point de terminaison pour le compte de stockage et l’ajouter aux règles de routage.

Pour **Source de données** , sélectionnez Messages de télémétrie des appareils. 

Ensuite, ajoutez une requête de routage. Dans cet exemple, les messages qui ont une propriété d’application nommée `level` avec une valeur égale à `critical` sont routés vers le compte de stockage.

![Capture d’écran montrant l’enregistrement d’une nouvelle règle de routage](./media/iot-hub-create-through-portal/iot-hub-add-route.png)

Cliquez sur **Enregistrer** pour enregistrer la règle de routage. Vous revenez au volet Routage des messages, et votre nouvelle règle de routage s’affiche.

### <a name="custom-endpoints"></a>Points de terminaison personnalisés

Cliquez sur l’onglet **Points de terminaison personnalisés** . Vous voyez les points de terminaison personnalisés déjà créés. Vous pouvez ajouter ici de nouveaux points de terminaison ou supprimer des points de terminaison existants. 

> [!NOTE]
> Si vous supprimez une route, cela ne supprime pas les points de terminaison affectés à cette route. Pour supprimer un point de terminaison, cliquez sur l’onglet Points de terminaison personnalisés, sélectionnez le point de terminaison que vous voulez supprimer, puis cliquez sur Supprimer.
>

Pour plus d’informations sur les points de terminaison personnalisés, consultez [Référence - Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md).

Vous pouvez définir jusqu’à 10 points de terminaison personnalisés pour un hub IoT. 

Pour voir un exemple complet montrant comment utiliser des points de terminaison personnalisés avec le routage, consultez [Routage des messages avec IoT Hub](tutorial-routing.md).

## <a name="find-a-specific-iot-hub"></a>Rechercher un hub IoT spécifique

Voici deux façons de rechercher un hub IoT spécifique dans votre abonnement :

1. Si vous connaissez le groupe de ressources auquel appartient le hub IoT, cliquez sur **Groupes de ressources** , puis sélectionnez le groupe de ressources dans la liste. L’écran du groupe de ressources montre toutes les ressources de ce groupe, notamment les hubs IoT. Cliquez sur le hub que vous recherchez.

2. Cliquez sur **Toutes les ressources** . Dans le volet **Toutes les ressources** , une liste déroulante affiche par défaut `All types`. Cliquez sur la liste déroulante et décochez la case `Select all`. Recherchez `IoT Hub` et cochez la case correspondante. Cliquez sur la zone de liste déroulante pour la fermer : les entrées sont alors filtrées, affichant seulement vos hubs IoT.

## <a name="delete-the-iot-hub"></a>Supprimez IoT Hub

Pour supprimer un hub IoT, recherchez le hub IoT que vous voulez supprimer, puis cliquez sur le bouton **Supprimer** en dessous du nom du hub IoT.

## <a name="next-steps"></a>Étapes suivantes

Suivez ces liens pour en savoir plus sur la gestion de Azure IoT Hub :

* [Routage des messages avec IoT Hub](tutorial-routing.md)
* [Surveiller votre IoT Hub](monitor-iot-hub.md)
