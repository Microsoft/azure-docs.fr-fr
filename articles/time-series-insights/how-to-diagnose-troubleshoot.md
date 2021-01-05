---
title: Diagnostiquer et résoudre les problèmes d’un environnement Gen2 - Azure Time Series Insights | Microsoft Docs
description: Découvrez comment diagnostiquer et résoudre les problèmes d’un environnement Gen2 Azure Time Series Insights.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 10/01/2020
ms.custom: seodec18
ms.openlocfilehash: eca2009ee0470dec9c9ce60d8754f8f7a71619d3
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97740536"
---
# <a name="diagnose-and-troubleshoot-an-azure-time-series-insights-gen2-environment"></a>Diagnostiquer et résoudre les problèmes d’un environnement Gen2 Azure Time Series Insights.

Cet article résume plusieurs problèmes courants que vous pourriez rencontrer lorsque vous travaillez avec votre environnement Gen2 Azure Time Series Insights. L’article décrit également les causes et solutions potentielles de chaque problème.

## <a name="problem-i-cant-find-my-environment-in-the-gen2-explorer"></a>Problème : Mon environnement est introuvable dans l’Explorateur Gen2

Ce problème peut se produire si vous n’êtes pas autorisé à accéder à l’environnement Time Series Insights. Les utilisateurs ont besoin d’un rôle d’accès de niveau lecteur pour visualiser leur environnement Time Series Insights. Pour vérifier les niveaux d’accès actuels et accorder un accès supplémentaire, accédez à la section **Stratégies d’accès aux données** sur la ressource Insights Time Series du [Portail Azure](https://portal.azure.com/).

  [![Vérifiez les stratégies d’accès aux données.](media/preview-troubleshoot/verify-data-access-policies.png)](media/preview-troubleshoot/verify-data-access-policies.png#lightbox)

## <a name="problem-no-data-is-seen-in-the-gen2-explorer"></a>Problème : Aucune donnée n’est visible dans l’Explorateur Gen2

Il existe plusieurs raisons pour lesquelles vos données pourraient ne pas apparaître dans l’[Explorateur Gen2 Azure Time Series Insights](https://insights.timeseries.azure.com/preview).

- Votre source d’événement ne reçoit peut-être pas de données.

    Vérifiez que votre source d’événement, qui est un event Hub ou un hub IoT, reçoit des données de vos balises ou instances. Pour ce faire, rendez-vous sur la page d’aperçu de votre ressource dans le portail Azure.

    [![Vue d’ensemble des métriques du tableau de bord des évaluations.](media/preview-troubleshoot/verify-dashboard-metrics.png)](media/preview-troubleshoot/verify-dashboard-metrics.png#lightbox)

- Vos données sources d’événements ne sont pas au format JSON.

    Time Series Insights ne prend en charge que les données JSON. Pour obtenir des exemples JSON, consultez [Formes JSON prises en charge](./concepts-json-flattening-escaping-rules.md).

- Il manque une autorisation requise pour votre clé de source d’événements.

  - Pour IoTHub, vous devez fournir la clé avec l’autorisation **Connexion de service**.

    [![Vérifiez les autorisations IoT Hub.](media/preview-troubleshoot/verify-correct-permissions.png)](media/preview-troubleshoot/verify-correct-permissions.png#lightbox)

    - Les stratégies **iothubowner** et **service** fonctionnent toutes les deux car elles disposent de l’autorisation **Connexion de service**.

  - Pour Event Hub, vous devez fournir la clé avec l’autorisation **Écouter**.
  
    [![Passez en revue les autorisations du hub d’événements.](media/preview-troubleshoot/verify-eh-permissions.png)](media/preview-troubleshoot/verify-eh-permissions.png#lightbox)

    - Les stratégies **Read** et **Manage** fonctionnent toutes les deux car elles disposent de l’autorisation **Écouter**.

- Votre groupe de consommateurs fourni n’est pas exclusif à Time Series Insights.

    Lors de l’enregistrement d’un IoT Hub ou Event Hub, spécifiez le groupe de consommateurs utilisé pour la lecture des données. Ce groupe de consommateurs doit être unique par environnement. Si le groupe de consommateurs est partagé, le concentrateur d’événements sous-jacent déconnecte automatiquement l’un des lecteurs au hasard. Fournissez un groupe de consommateurs unique auprès duquel Time Series Insights lira les informations.

- Votre propriété ID Time Series spécifiée au moment de l’approvisionnement est incorrecte, manquante ou nulle.

    Ce problème peut survenir si la propriété Time Series ID (ID de série chronologique) est mal configurée au moment de l’approvisionnement de l’environnement. Pour plus d’informations, consultez [Bonnes pratiques pour le choix d’un ID Time Series](./how-to-select-tsid.md). Pour le moment, vous ne pouvez pas mettre à jour un environnement Time Series Insights existant pour utiliser un ID de série chronologique différent.

## <a name="problem-some-data-shows-but-some-is-missing"></a>Problème : Certaines données s’affichent, mais d’autres manquent

Il se peut que vous envoyiez des données sans l’ID de série chronologique.

- Ce problème peut survenir lorsque vous envoyez des événements sans le champ ID de série chronologique dans la charge utile. Pour plus d’informations, consultez [Formes JSON prises en charge](./concepts-json-flattening-escaping-rules.md).
- Ce problème peut survenir parce que votre environnement est limité.

    > [!NOTE]
    > À l’heure actuelle, Time Series Insights prend en charge un taux d’ingestion maximal de 6 Mbit/s.

## <a name="problem-data-was-showing-but-now-ingestion-has-stopped"></a>Problème : Des données étaient affichées, mais la réception s’est arrêtée

- Votre clé de source d’événement a peut-être été générée à nouveau et votre environnement Gen2 a besoin de la nouvelle clé de source d’événement.

Ce problème se produit lorsque la clé fournie lors de la création de votre source d’événements n’est plus valide. Vous pouvez voir les données de télémétrie dans votre hub, mais aucun message n’a été reçu en entrée dans Time Series Insights. Si vous ne savez pas si la clé a été générée à nouveau, vous pouvez rechercher « Création ou mise à jour de règles d’autorisation d’espace de noms » dans le journal d’activité de votre Event Hubs ou « Création ou mise à jour d’une ressource IoT Hub » pour IoT Hub.

Pour mettre à jour votre environnement Gen2 Time Series Insights avec la nouvelle clé, ouvrez votre ressource de hub dans le portail Azure et copiez la nouvelle clé. Accédez à votre ressource TSI, puis cliquez sur Sources d’événements.

   [![Capture d’écran montrant la ressource T S I avec l’élément de menu Sources d’événements appelé.](media/preview-troubleshoot/update-hub-key-step-1.png)](media/preview-troubleshoot/update-hub-key-step-1.png#lightbox)

Sélectionnez la ou les sources d’événements à partir desquelles l’ingestion s’est arrêtée, collez la nouvelle clé, puis cliquez sur Enregistrer.

   [![Capture d’écran montrant la ressource T S I avec la clé de stratégie IoT Hub entrée.](media/preview-troubleshoot/update-hub-key-step-2.png)](media/preview-troubleshoot/update-hub-key-step-2.png#lightbox)

## <a name="problem-my-event-sources-timestamp-property-name-doesnt-work"></a>Problème : le nom de la propriété Timestamp de ma source d’événement ne fonctionne pas

Vérifiez que le nom et la valeur répondent aux critères suivants :

- Le nom de la propriété Timestamp est sensible à la casse.
- La valeur de la propriété Timestamp provenant de votre source d’événement, telle qu’une chaîne JSON, est au format `yyyy-MM-ddTHH:mm:ss.FFFFFFFK`. Exemple de chaîne : `"2008-04-12T12:53Z"`.

La méthode la plus simple pour vous assurer que votre nom de propriété Timestamp est capturé et fonctionne correctement consiste à utiliser l’Explorateur Gen2 Time Series Insights. Dans l’Explorateur Gen2 Time Series Insights, utilisez le graphique pour sélectionner une période après avoir indiqué le nom de la propriété Timestamp. Cliquez avec le bouton droit sur la sélection et sélectionnez l’option **Explorer les événements**. Le premier en-tête de colonne correspond au nom de votre propriété Timestamp. Il devrait contenir `($ts)` en regard du mot `Timestamp` plutôt que :

- `(abc)`, ce qui indique que Time Series Insights lit les valeurs de données sous forme de chaînes.
- L’**icône de calendrier**, qui indique que Time Series Insights lit la valeur de données sous forme de date/heure.
- `#`, ce qui indique que Time Series Insights lit les valeurs de données sous forme d’entier.

Si la propriété Timestamp n’est pas explicitement spécifiée, l’horodatage par défaut est l’heure de mise en file d’attente du hub IoT ou du hub d’événement.

## <a name="problem-i-cant-view-data-from-my-warm-store-in-the-explorer"></a>Problème : Je ne peux pas afficher les données de mon magasin Warm dans l’Explorateur

- Vous avez peut-être approvisionné votre magasin Warm récemment et les données sont toujours en cours d’acheminement.
- Vous avez peut-être supprimé votre magasin Warm, auquel cas vous risquez de perdre des données.

## <a name="problem-i-cant-view-or-edit-my-time-series-model"></a>Problème : Je n’arrive pas à afficher ou à modifier mon modèle Time Series

- Vous accédez peut-être à un environnement Time Series Insights S1 ou S2.

   Les modèles de séries chronologiques ne sont pris en charge que dans les environnements de paiement à l’utilisation. Pour plus d’informations sur la façon d’accéder à votre environnement S1 ou S2 à partir de l’Explorateur Gen2 Time Series Insights, consultez [Visualiser les données dans l’Explorateur](./concepts-ux-panels.md).

   [![Aucun événement dans l’environnement.](media/preview-troubleshoot/troubleshoot-no-events.png)](media/preview-troubleshoot/troubleshoot-no-events.png#lightbox)

- Il se peut que vous n’ayez pas les autorisations nécessaires pour afficher et modifier le modèle.

   Les utilisateurs ont besoin d’un accès de niveau contributeur pour modifier et visualiser leur modèle de série chronologique. Pour vérifier les niveaux d’accès actuels et accorder un accès supplémentaire, accédez à la section **Stratégies d’accès aux données** sur votre ressource Insights Time Series du Portail Azure.

## <a name="problem-all-my-instances-in-the-gen2-explorer-lack-a-parent"></a>Problème : Toutes mes instances dans l’Explorateur Gen2 n’ont pas de parent

Ce problème peut se produire si votre environnement n’a pas de hiérarchie de modèle Time Series définie. Pour plus d’informations, découvrez comment [utiliser des modèles Time Series](./time-series-insights-overview.md).

  [![Les instances non apparentées affichent un avertissement.](media/preview-troubleshoot/unparented-instances.png)](media/preview-troubleshoot/unparented-instances.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [utiliser des modèles Time Series](./time-series-insights-overview.md).

- En savoir plus sur les [formes JSON prises en charge](./concepts-json-flattening-escaping-rules.md).

- Passez en revue [la planification et les limites](./how-to-plan-your-environment.md) dans Azure Time Series Insights Gen2.