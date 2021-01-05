---
title: Vue d'ensemble des Clusters Azure Stream Analytics (préversion)
description: Découvrez l'offre de cluster Stream Analytics dédiée à un seul locataire.
author: sidramadoss
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: overview
ms.custom: mvc
ms.date: 09/22/2020
ms.openlocfilehash: f67df45b6b663584e848983251031146cdf8df60
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95247691"
---
# <a name="overview-of-azure-stream-analytics-cluster-preview"></a>Vue d'ensemble du Cluster Azure Stream Analytics (préversion)

L'offre de cluster Azure Stream Analytics permet de déployer un seul locataire pour les scénarios de streaming complexes et exigeants. À grande échelle, les clusters Stream Analytics peuvent traiter plus de 200 Mo/seconde en temps réel. Les travaux Stream Analytics exécutés sur des clusters dédiés peuvent tirer parti de toutes les fonctionnalités de l'offre Standard et incluent la prise en charge de la connectivité par liaison privée à vos entrées et sorties.

Les clusters Stream Analytics sont facturés sur la base des unités de streaming, qui représentent la quantité de ressources processeur et de mémoire allouée à votre cluster. Une unité de streaming est identique pour les offres Standard et Dedicated. Vous pouvez acheter 36, 72, 108, 144, 180 ou 216 unités de streaming par cluster. Un cluster Stream Analytics peut servir de plateforme de streaming à votre organisation et être partagé par différentes équipes travaillant sur des cas d'usage distincts.

## <a name="what-are-stream-analytics-clusters"></a>Que sont les clusters Stream Analytics ?

Les clusters Stream Analytics sont alimentés par le même moteur que les travaux Stream Analytics exécutés dans un environnement multi-locataire. Le cluster dédié à un seul locataire présente les fonctionnalités suivantes :

* Hébergement à locataire unique ne subissant pas le bruit d’autres locataires. Vos ressources sont « isolées » les unes des autres et sont plus performantes lors des pics de trafic.

* Faites passer votre cluster à un nombre d'unités de streaming compris entre 36 et 216 à mesure que vos besoins en streaming augmentent.

* Prise en charge d'un réseau virtuel qui permet à vos travaux Stream Analytics de se connecter en toute sécurité à d'autres ressources par le biais de points de terminaison privés.

* Possibilité de créer des fonctions C# définies par l'utilisateur et des désérialiseurs personnalisés dans n'importe quelle région.

* Aucuns frais de maintenance, ce qui vous permet de concentrer vos efforts sur la création de solutions d'analyse en temps réel.

## <a name="how-to-get-started"></a>Pour commencer

Vous pouvez [créer un cluster Stream Analytics](create-cluster.md) via le [portail Azure](https://aka.ms/asaclustercreateportal). Pour toute question, ou si vous avez besoin d'aide concernant l'intégration, vous pouvez contacter l'[équipe Stream Analytics](mailto:askasa@microsoft.com).

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="how-do-i-choose-between-a-stream-analytics-cluster-and-a-stream-analytics-job"></a>Comment choisir entre un cluster Stream Analytics et un travail Stream Analytics ?

Pour commencer, le plus simple est de créer et de développer un travail Stream Analytics afin de vous familiariser avec le service et de voir comment il peut répondre à vos besoins en matière d'analyse.

À eux seuls, les travaux Stream Analytics ne prennent pas en charge les réseaux virtuels. Si vos entrées ou sorties sont sécurisées derrière un pare-feu ou un réseau virtuel Azure, vous disposez des deux options suivantes :

* Si votre ordinateur local a accès aux ressources d'entrée et de sortie sécurisées par un réseau virtuel (par exemple, Azure Event Hubs ou Azure SQL Database), vous pouvez [installer les outils Azure Stream Analytics pour Visual Studio](stream-analytics-tools-for-visual-studio-install.md) sur votre ordinateur local. Vous pouvez développer et [tester des travaux Stream Analytics localement](stream-analytics-live-data-local-testing.md) sur votre appareil sans aucuns frais. Lorsque vous êtes prêt à utiliser Stream Analytics dans votre architecture, vous pouvez créer un cluster Stream Analytics, configurer des points de terminaison privés, et exécuter vos travaux à grande échelle.

* Vous pouvez créer un cluster Stream Analytics, configurer le cluster avec les points de terminaison privés nécessaires à votre pipeline, et exécuter vos travaux Stream Analytics sur le cluster.

### <a name="what-performance-can-i-expect"></a>À quelles performances puis-je m'attendre ?

Une unité de streaming est identique pour les offres Standard et Dedicated. Un travail individuel qui utilise un cluster complet de 36 unités de streaming peut atteindre un débit d'environ 36 Mo/seconde avec une latence de l'ordre de la milliseconde. Le nombre exact dépend du format des événements et du type d'analyse. Dans la mesure où le cluster Stream Analytics est dédié, il offre des garanties de performances plus fiables. Les travaux exécutés sur votre cluster n'appartiennent qu'à vous.

### <a name="can-i-scale-my-cluster"></a>Puis-je mettre mon cluster à l'échelle ?

Oui. Vous pouvez facilement configurer la capacité de votre cluster, ce qui vous permet d'effectuer [un scale-up ou un scale-down](scale-cluster.md) en fonction de vos besoins afin de répondre à l'évolution de la demande.

### <a name="can-i-run-my-existing-jobs-on-these-new-clusters-ive-created"></a>Puis-je exécuter mes travaux existants sur les nouveaux clusters que j'ai créés ?

Oui. Vous pouvez lier vos travaux existants au cluster Stream Analytics que vous venez de créer et l'exécuter comme vous le faites habituellement. Il n'est pas nécessaire de recréer les travaux Stream Analytics existants à partir de zéro.

### <a name="how-much-will-these-clusters-cost-me"></a>Combien me coûteront ces clusters ?

Vos clusters Stream Analytics sont facturés en fonction de la capacité choisie en termes d'unités de streaming. Les clusters sont facturés à l'heure, et aucuns frais supplémentaires ne s'appliquent aux travaux exécutés dans ces clusters. Consultez la [page de tarification du service Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/) pour accéder aux mises à jour de facturation des points de terminaison privés.

### <a name="which-inputs-and-outputs-can-i-privately-connect-to-from-my-stream-analytics-cluster"></a>À quelles entrées et sorties puis-je me connecter en privé à partir de mon cluster Stream Analytics ?

Stream Analytics prend en charge différents types d'entrées et de sorties. Vous pouvez [créer dans votre cluster des points de terminaison privés](private-endpoints.md) qui permettent aux travaux d'accéder aux ressources d'entrée et de sortie. Actuellement, Azure SQL Database, Stockage Azure, Azure Data Lake Storage Gen2, Azure Event Hub et Azure Service Bus sont des services pris en charge, et d’autres types seront bientôt ajoutés. 

## <a name="next-steps"></a>Étapes suivantes

Vous disposez maintenant d'une vue d'ensemble du cluster Azure Stream Analytics. Vous pouvez ensuite créer votre cluster et exécuter votre travail Stream Analytics : 

* [Créer un cluster Stream Analytics](create-cluster.md)
* [Gérer les points de terminaison privés dans un cluster Azure Stream Analytics](private-endpoints.md)
* [Gérer des travaux Stream Analytics dans un cluster Stream Analytics](manage-jobs-cluster.md)
* [Création d’un travail Stream Analytics](stream-analytics-quick-create-portal.md)
