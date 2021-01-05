---
title: Créer un environnement – Azure Time Series Insights | Microsoft Docs
description: Découvrez comment utiliser le Portail Azure pour créer un environnement Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/29/2020
ms.custom: seodec18
ms.openlocfilehash: 61af7922318514a7b86a349d1970c59d4d168d85
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95023324"
---
# <a name="create-a-new-azure-time-series-insights-gen1-environment-in-the-azure-portal"></a>Créer un nouvel environnement Azure Time Series Insights Gen1 dans le Portail Azure

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet article décrit comment créer un environnement Azure Time Series Insights via le Portail Azure.

Azure Time Series Insights vous permet de commencer à visualiser et interroger les données qui circulent dans Azure IoT Hubs et Event Hubs en quelques minutes, ce qui vous permet d’interroger de grands volumes de données de série chronologique en quelques secondes.  Cette solution a été conçue pour l’échelle de l’Internet des objets et peut gérer plusieurs téraoctets de données.

## <a name="steps-to-create-the-environment"></a>Étapes pour créer l’environnement

Procédez comme suit pour créer un environnement :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez le bouton **+ Créer une ressource**.

1. Sélectionnez la catégorie **Internet des objets**, puis **Time Series Insights**.

   [![Créer l’environnement Azure Time Series Insights](media/time-series-insights-get-started/tsi-create-new-environment.png)](media/time-series-insights-get-started/tsi-create-new-environment.png#lightbox)

1. Sur la page **Time Series Insights**, sélectionnez **Créer**.

1. Renseignez les paramètres requis. Le tableau suivant décrit chaque paramètre :

   [![Créer le groupe de ressources Azure Time Series Insights](media/time-series-insights-get-started/tsi-configure-and-create.png)](media/time-series-insights-get-started/tsi-configure-and-create.png#lightbox)

   Paramètre|Valeur suggérée|Description
   ---|---|---
   Nom de l’environnement | Un nom unique | Ce nom représente l’environnement dans [l’explorateur time series](https://insights.timeseries.azure.com)
   Subscription | Votre abonnement | Si vous avez plusieurs abonnements, choisissez l’abonnement qui contient votre source d’événements de préférence. Azure Time Series Insights peut détecter automatiquement les ressources Azure IoT Hub et Event Hub associées à un même abonnement.
   Resource group | Créer un nouveau groupe ou en choisir un existant | Un groupe de ressources correspond à une collection de ressources Azure utilisées ensemble. Vous pouvez choisir un groupe de ressources existant, par exemple, celui qui contient votre concentrateur d’événements ou un IoT Hub. Ou vous pouvez en créer un nouveau si cette ressource n’est pas liée aux autres ressources.
   Location | Le plus près possible de votre source d’événements | De préférence, choisissez le même emplacement que pour le centre de données qui contient vos données de source d’événements, dans le but d’éviter les coûts de bande passante supplémentaires entre régions et zones et l’augmentation de la latence lors du déplacement des données en dehors de la région.
   Niveau tarifaire | S1 | Choisissez le débit nécessaire. Pour des coûts inférieurs et une capacité de démarrage, sélectionnez S1.
   Capacité | 1 | La capacité est le multiplicateur qui s’applique à la vitesse d’entrée, à la capacité de stockage et au coût associé à la référence (SKU) sélectionnée.  Vous pouvez modifier la capacité d’un environnement après sa création. Pour obtenir des coûts plus bas, sélectionnez une capacité de 1.
  
1. Sélectionnez **Créer** pour commencer le processus d’approvisionnement. Cela peut prendre quelques minutes.

1. Sélectionnez le symbole de **Notifications** (en forme de cloche) pour surveiller le processus de déploiement.

   [![Regarder les notifications](media/time-series-insights-get-started/tsi-deploy-notifications.png)](media/time-series-insights-get-started/tsi-deploy-notifications.png#lightbox)

1. Dans la **Vue d’ensemble** de la ressource, vérifiez les paramètres de configuration de votre déploiement.

   [![Créer l’épingle Azure Time Series Insights sur le tableau de bord](media/time-series-insights-get-started/tsi-verify-deployment.png)](media/time-series-insights-get-started/tsi-verify-deployment.png#lightbox)

1. **(Facultatif)** Sélectionnez l’**icône en forme d’épingle** située en haut à droite pour accéder plus facilement à votre environnement Azure Time Series Insights la prochaine fois.

## <a name="next-steps"></a>Étapes suivantes

* [Définissez des stratégies d’accès aux données](./concepts-access-policies.md) pour sécuriser votre environnement.

* [Ajoutez une source d’événement Event Hub](./how-to-ingest-data-event-hub.md) à votre environnement Azure Time Series Insights.

* [Envoyez des événements](time-series-insights-send-events.md) à la source d’événement.

* Affichez votre environnement dans [l’Explorateur Time Series Insights](https://insights.timeseries.azure.com).