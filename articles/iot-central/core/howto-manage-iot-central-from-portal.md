---
title: Gérer IoT Central à l’aide du Portail Azure | Microsoft Docs
description: Cet article explique comment créer et gérer des applications IoT Central sur le Portail Azure.
services: iot-central
ms.service: iot-central
author: dominicbetts
ms.author: dobett
ms.date: 02/11/2020
ms.topic: how-to
manager: philmea
ms.openlocfilehash: 0802169898cad51e9a5d208e546a004f7ea7b3c0
ms.sourcegitcommit: b4f303f59bb04e3bae0739761a0eb7e974745bb7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/02/2020
ms.locfileid: "91653330"
---
# <a name="manage-iot-central-from-the-azure-portal"></a>Gérer IoT Central à l’aide du Portail Azure

[!INCLUDE [iot-central-selector-manage](../../../includes/iot-central-selector-manage.md)]

Au lieu de créer et de gérer des applications IoT Central sur le site web du [Gestionnaire d'applications Azure IoT Central](https://aka.ms/iotcentral), vous pouvez utiliser le [portail Azure](https://portal.azure.com) pour gérer vos applications.

## <a name="create-iot-central-applications"></a>Créer des applications IoT Central

Pour créer une application, accédez au [portail Azure](https://ms.portal.azure.com), puis sélectionnez **Créer une ressource**.

Dans la barre **Rechercher sur la Place de marché**, tapez *IoT Central* :

![Portail de gestion : recherche](media/howto-manage-iot-central-from-portal/image0a1.png)

Sélectionnez la vignette **Application IoT Central** dans les résultats de la recherche :

![Portail de gestion : résultats de la recherche](media/howto-manage-iot-central-from-portal/image0b1.png)

À présent, sélectionnez **Créer** :

![Portail de gestion : Ressource IoT Central](media/howto-manage-iot-central-from-portal/image0c1.png)

Renseignez tous les champs dans le formulaire. Ce formulaire ressemble à celui que vous devez compléter pour créer des applications sur le site web du [Gestionnaire d'applications Azure IoT Central](https://aka.ms/iotcentral). Pour plus d’informations, consultez le guide de démarrage rapide [Créer une application IoT Central](quick-deploy-iot-central.md).

![Créer un formulaire IoT Central](media/howto-manage-iot-central-from-portal/image6a.png)

L’**emplacement** correspond à la [zone géographique](https://azure.microsoft.com/global-infrastructure/geographies/) où vous souhaitez créer votre application. D'une façon générale, il est recommandé de choisir l'emplacement qui est physiquement le plus proche de vos appareils pour obtenir des performances optimales. Azure IoT Central est actuellement disponible dans les zones géographiques suivantes : **Australie**, **Asie-Pacifique**, **Europe**, **États-Unis**, **Royaume-Uni** et **Japon**. Une fois que vous aurez choisi un emplacement, vous ne pourrez plus déplacer votre application vers un autre emplacement.

Après avoir renseigné tous les champs, sélectionnez **Créer**.

## <a name="manage-existing-iot-central-applications"></a>Gérer des applications IoT Central existantes

Si vous disposez déjà d’une application Azure IoT Central, vous pouvez la supprimer ou la déplacer vers un abonnement ou un groupe de ressources différent dans le portail Azure.

> [!NOTE]
> Vous ne voyez pas les applications créées avec le plan Gratuit dans le portail Azure, car celles-ci ne sont pas associées à votre abonnement.

Pour commencer, sélectionnez **Toutes les ressources** dans le portail. Sélectionnez **Afficher les types masqués**, puis commencez à taper le nom de votre application dans la zone **Filtrer par nom** pour la trouver. Sélectionnez ensuite sur l’application IoT Central que vous souhaitez gérer.

Pour accéder à l’application, sélectionnez l’**URL de l’application IoT Central** :

![Capture d’écran montrant la page « Vue d’ensemble » avec l’option « URL de l’application IoT Central » mise en évidence.](media/howto-manage-iot-central-from-portal/image3.png)

Pour déplacer l’application vers un autre groupe de ressources, sélectionnez le lien **Modifier** à côté du groupe de ressources. Dans la page **Déplacer des ressources**, sélectionnez le groupe de ressources vers lequel vous souhaitez déplacer cette application :

![Capture d’écran montrant la page « Vue d’ensemble » avec l’option « Groupe de ressources (modifier) » mise en surbrillance.](media/howto-manage-iot-central-from-portal/image4a.png)

Pour déplacer l’application vers un autre abonnement, sélectionnez **Modifier** en regard de l’abonnement. Dans la page **Déplacer des ressources**, choisissez l’abonnement vers lequel vous souhaitez déplacer cette application :

![Portail de gestion : gestion des ressources](media/howto-manage-iot-central-from-portal/image5a.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez appris à gérer des applications Azure IoT Central dans le portail Azure, voici l’étape suivante suggérée :

> [!div class="nextstepaction"]
> [Administrer votre application](howto-administer.md)