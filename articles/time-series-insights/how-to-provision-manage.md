---
title: Provisionner et gérer un environnement Gen2 - Azure Time Series | Microsoft Docs
description: Apprenez à provisionner et gérer un environnement Azure Time Series Insights Gen2.
author: shipra1mishra
ms.author: shmishr
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 10/02/2020
ms.custom: seodec18
ms.openlocfilehash: 7c38c57a8480ef2addde494b94d70bd2eb679373
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016766"
---
# <a name="provision-and-manage-azure-time-series-insights-gen2"></a>Approvisionner et gérer Azure Time Series Insights Gen2

Cet article explique comment créer et gérer un environnement Azure Time Series Insights Gen2 via le [portail Azure](https://portal.azure.com/).

## <a name="overview"></a>Vue d’ensemble

Lorsque vous provisionnez un environnement Azure Time Series Insights Gen2, vous créez ces ressources Azure :

* Un environnement Azure Time Series Insights Gen2 qui suit le modèle de tarification avec paiement à l’accès
* Un compte Azure Storage
* Un magasin chaud facultatif pour des requêtes plus rapides et illimitées

> [!TIP]
>
> * Découvrez [comment planifier votre environnement](./how-to-plan-your-environment.md).
> * Lisez les articles expliquant comment [Ajouter une source de hub d’événements](./how-to-ingest-data-event-hub.md) et comment [Ajouter une source de hub IoT](./how-to-ingest-data-iot-hub.md).

Vous apprendrez à :

1. Associez chaque environnement Azure Time Series Insights Gen2 à une source d’événement. Vous devez également fournir une propriété ID d’horodatage et un groupe de consommateurs unique pour garantir que l’environnement a accès aux événements appropriés.

1. Une fois l’approvisionnement terminé, vous pouvez modifier vos stratégies d’accès et d’autres attributs de l’environnement pour qu’ils correspondent à vos besoins.

   > [!NOTE]
   > La première étape est facultative lors du provisionnement d’un environnement. Si vous ignorez cette étape, vous devez attacher une source d’événement à l’environnement ultérieurement afin que les données puissent circuler dans l’environnement et être accessibles par des requêtes.

## <a name="create-the-environment"></a>Créer l’environnement

Pour créer un environnement Azure Time Series Insights Gen2 :

1. Créez une ressource Azure Time Series Insights sous *Internet des objets* sur le [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Gen2 (L1)** comme **Niveau**. Fournissez un nom d’environnement et choisissez le groupe d’abonnements et le groupe de ressources à utiliser. Sélectionnez ensuite un emplacement pris en charge pour héberger l’environnement.

   [![Créez une instance Azure Time Series Insights.](media/v2-update-manage/create-and-manage-configuration.png)](media/v2-update-manage/create-and-manage-configuration.png#lightbox)

1. Entrez un ID Time Series.

    > [!NOTE]
    >
    > * L’ID de série chronologique *respecte la casse* et est *immuable*. (il ne peut pas être modifié une fois défini).
    > * Les ID de série chronologique peuvent comprendre jusqu’à *trois* clés. Considérez cela comme une clé primaire dans une base de données, qui représente de façon unique chaque capteur d’appareil qui envoie des données à votre environnement. Il peut s’agir d’une propriété ou d’une combinaison de jusqu’à trois propriétés.
    > * En savoir plus sur la façon de [choisir un ID de série chronologique](./how-to-select-tsid.md)

1. Créez un compte de Stockage Azure en sélectionnant un nom de compte de stockage ainsi qu’un type de compte et en spécifiant un choix de [réplication](../storage/common/redundancy-migration.md?tabs=portal). Un compte Stockage Azure est alors automatiquement créé. Par défaut, un [compte à usage général v2](../storage/common/storage-account-overview.md) sera créé. Ce compte est créé dans la même région que l’environnement Azure Time Series Insights Gen2 que vous avez sélectionné précédemment.
Vous pouvez également apporter votre propre stockage (BYOS) via le [modèle ARM](./time-series-insights-manage-resources-using-azure-resource-manager-template.md) lorsque vous créez un nouvel environnement Azure Time Series Gen2.

1. **(Facultatif)** Activez le magasin chaud pour votre environnement si vous voulez des requêtes plus rapides et illimitées sur les données les plus récentes de votre environnement. Vous pouvez également créer ou supprimer un magasin chaud par le biais de l’option **Configuration du stockage** dans le volet de navigation gauche, après avoir créé un environnement Azure Time Series Insights Gen2.

1. **(Facultatif)** Vous pouvez ajouter une source d’événement maintenant. Vous pouvez également attendre que l’instance soit provisionnée.

   * Azure Time Series Insights prend en charge [Azure IoT Hub](./how-to-ingest-data-iot-hub.md) et [Azure Event Hubs](./how-to-ingest-data-event-hub.md) en tant que source d’événement. Si vous ne pouvez ajouter qu’une seule source d’événement lors de la création d’un environnement, vous pouvez en ajouter une autre ultérieurement.

     Vous pouvez sélectionner un groupe de consommateurs existant ou bien en créer un nouveau lors de l’ajout de la source d’événement. Notez que la source de l’événement requiert un groupe de consommateurs unique pour votre environnement pour y lire des données.

   * Choisissez la propriété Timestamp appropriée. Par défaut, Azure Time Series Insights utilise l’heure de placement du message dans la file d’attente pour chaque source d’événement.

     > [!TIP]
     > Il est possible que l’heure de placement du message dans la file d’attente ne soit pas le meilleur paramètre configuré à utiliser dans les scénarios de traitement par lots des événements ou de chargement des données historiques. Le cas échéant, veillez à vérifier votre choix d’utiliser ou non une propriété Timestamp.

     [![Onglet de configuration de la source d’événement](media/v2-update-manage/create-and-manage-event-source.png)](media/v2-update-manage/create-and-manage-event-source.png#lightbox)

1. Vérifiez que votre environnement a été provisionné et configuré comme vous le souhaitez.

    [![Onglet Vérifier + créer](media/v2-update-manage/create-and-manage-review-and-confirm.png)](media/v2-update-manage/create-and-manage-review-and-confirm.png#lightbox)

## <a name="manage-the-environment"></a>Gérer l’environnement

Vous pouvez gérer votre environnement Azure Time Series Insights Gen2 à l’aide du portail Azure. Il y a quelques différences essentielles entre les environnements Gen2 et les environnements Gen1 S1 ou S2 en disponibilité générale à ne pas oublier lorsque vous gérez votre environnement par le biais du portail Azure :

* Le panneau **Vue d’ensemble** du portail Azure Gen2 présente les changements suivants :

  * La capacité est supprimée car elle ne s’applique pas aux environnements Gen2.
  * La propriété d’**ID de série chronologique** est ajoutée. Elle détermine la façon dont vos données sont partitionnées.
  * Les jeux de données de référence sont supprimés.
  * L’URL affichée vous dirige vers [l’Explorateur Azure Time Series Insights](./concepts-ux-panels.md).
  * Le nom de votre compte de stockage Azure est répertorié.

* Le panneau **Configurer** du portail Azure est supprimé, car les unités d'échelle ne s’appliquent pas aux environnements Azure Time Series Insights Gen2. Toutefois, vous pouvez utiliser **Configuration du stockage** pour configurer le magasin chaud récemment introduit.

* Le panneau **Données de référence** du portail Azure est supprimé dans Azure Time Series Insights Gen2 parce que le concept de données de référence a été remplacé par le [Modèle de série chronologique (TSM)](./concepts-model-overview.md).

[![Environnement Azure Time Series Insights Gen2 dans le portail Azure](media/v2-update-manage/create-and-manage-overview-confirm.png)](media/v2-update-manage/create-and-manage-overview-confirm.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les environnements Azure Time Series Insights en disponibilité générale et Gen2, consultez [Planifier votre environnement](./how-to-plan-your-environment.md).

* Découvrez comment [ajouter une source Event Hub](./how-to-ingest-data-event-hub.md).

* Configurez une [source de hub IoT](./how-to-ingest-data-iot-hub.md).