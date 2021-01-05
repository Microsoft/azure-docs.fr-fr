---
title: Événements Azure Service Fabric
description: En savoir plus sur les événements Service Fabric fournis pour vous aider à surveiller votre cluster Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 11/21/2018
ms.author: srrengar
ms.openlocfilehash: 638b650e485ad3e83bd6021639a7e55b540d9cdc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75451728"
---
# <a name="service-fabric-events"></a>Événements de Service Fabric 

La plateforme Service Fabric écrit plusieurs événements structurés pour les activités opérationnelles clés qui se déroulent au sein de votre cluster. Cela va des mises à niveau de cluster aux décisions de placement des réplicas. Chaque événement exposé par Service Fabric est mappé à l’une des entités suivantes dans le cluster :
* Cluster
* Application
* Service
* Partition
* Réplica 
* Conteneur

Pour afficher la liste complète des événements exposés par la plateforme : [Liste des événements Service Fabric](service-fabric-diagnostics-event-generation-operational.md).

Voici quelques exemples de scénarios pour lesquels vous devez voir des événements dans votre cluster. 
* Événements de cycle de vie de nœud : À mesure que les nœuds augmentent, diminuent, effectuent un scale-in/out, sont activés/désactivés ou redémarrés, ces événements sont exposés vous montrant ce qui s’est passé. Ils vous aident à identifier s’il existe un problème lié à la machine proprement dite ou si une API a été appelée par le biais de Service Fabric pour modifier l’état d’un nœud.
* Mise à niveau de cluster : Lorsque le cluster est mis à niveau (modification de la configuration ou de la version de Service Fabric), vous pouvez observer le lancement de la mise à niveau, le passage par chacun de vos domaines de mise à niveau et l’achèvement (ou restauration). 
* Mises à niveau de l’application : Comme pour les mises à niveau de cluster, un ensemble complet d’événements se produisent au cours de la mise à niveau. Ces événements peuvent être utiles pour comprendre quand une mise à niveau a été planifiée, l’état actuel d’une mise à niveau et la séquence globale d’événements. Cela est utile pour vérifier les mises à niveau qui ont été déployées avec succès ou si une restauration a été déclenchée.
* Déploiement/suppression de service/d’application : Des événements sont créés ou supprimés pour chaque application, service et conteneur. Ils sont utiles lors d’un scale-in/out, par exemple pour augmenter le nombre de réplicas.
* Déplacements de partition (reconfiguration) : à chaque fois qu’une partition avec état subit une reconfiguration (modification dans le jeu de réplicas), un événement est consigné. Cela est utile si vous essayez de comprendre la fréquence à laquelle votre jeu de réplicas de partition est modifié ou bascule, ou de suivre le nœud qui exécutait votre réplica principal à un moment dans le temps.
* Événements de chaos : lors de l’utilisation du service [Chaos](service-fabric-controlled-chaos.md) de Service Fabric, vous voyez des événements à chaque démarrage ou arrêt du service ou lorsqu’une erreur est injectée dans le système.
* Événements d’intégrité : Service Fabric expose des événements d’intégrité à chaque fois qu’un rapport d’intégrité de type Avertissement ou Erreur est créé, qu’une entité repasse à un état d’intégrité OK ou qu’un rapport d’intégrité arrive à expiration. Ces événements sont très utiles pour effectuer le suivi de l’historique des statistiques d’intégrité d’une entité. 

## <a name="how-to-access-events"></a>Accès à des événements

Vous pouvez accéder aux événements Service Fabric de différentes façons :
* Les événements sont journalisés via des canaux standard tels que les journaux des événements ETW/Windows et peuvent être visualisés par n’importe quel outil de supervision qui les prend en charge, tels que les journaux Azure Monitor. Par défaut, pour les clusters créés dans le portail, les diagnostics sont activés et l’agent Diagnostics Azure pour Windows envoie les événements au Stockage Table Azure, mais vous devez encore intégrer cela à votre ressource Log Analytics. Apprenez-en davantage sur la configuration de l’[agent Diagnostics Azure](service-fabric-diagnostics-event-aggregation-wad.md) pour modifier la configuration des diagnostics de votre cluster afin de collecter plus de journaux et de compteurs de performances, et l’[intégration des journaux Log Monitor](service-fabric-diagnostics-event-analysis-oms.md).
* API Rest du service EventStore qui vous permettent d’interroger le cluster, directement ou via la bibliothèque de client Service Fabric. Consultez [Query EventStore APIs for cluster events](service-fabric-diagnostics-eventstore-query.md) (Interroger les API EventStore pour rechercher les événements de cluster).

## <a name="next-steps"></a>Étapes suivantes
* Pour plus d’informations sur la surveillance du cluster, consultez [Monitoring du cluster et de la plateforme](service-fabric-diagnostics-event-generation-infra.md).
* Pour en savoir plus sur le service EventStore, consultez [Vue d’ensemble du service EventStore](service-fabric-diagnostics-eventstore.md).
