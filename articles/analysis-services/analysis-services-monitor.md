---
title: Surveiller les indicateurs de performance de serveur de Azure Analysis Services | Microsoft Docs
description: Découvrez comment Analysis Services utilise Azure Metrics Explorer, un outil gratuit du portail permettant d’effectuer le monitorage des performances et de l’intégrité des serveurs.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 03/04/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: aaa3a6d128fe7dd466f6f60ab515f05fa38ba63b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84690374"
---
# <a name="monitor-server-metrics"></a>Surveiller les indicateurs de performances du serveur

Analysis Services fournit des mesures dans Azure Metrics Explore, un outil gratuit du portail vous permettant de surveiller les performances et l’intégrité de vos serveurs. Par exemple, vous pouvez surveiller l’utilisation de la mémoire et du processeur, le nombre de connexions clientes et la consommation de ressources de requête. Analysis Services utilise la même infrastructure de surveillance que la plupart des autres services Azure. Pour en savoir plus, consultez la section [Bien démarrer avec Azure Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md).

Pour effectuer des diagnostics plus approfondis, suivre des performances et identifier les tendances de plusieurs ressources de service au sein d’un groupe de ressources ou d’un abonnement, utilisez [Azure Monitor](../azure-monitor/overview.md). Azure Monitor (service) peut entraîner un service facturable.


## <a name="to-monitor-metrics-for-an-analysis-services-server"></a>Pour surveiller les indicateurs de performance sur un serveur Analysis Services

1. Dans le portail Azure, sélectionnez **Indicateurs de performances**.

    ![Surveiller dans le portail Azure](./media/analysis-services-monitor/aas-monitor-portal.png)

2. Dans **Métrique**, sélectionnez les métriques à inclure sur votre graphique. 

    ![Graphique de surveillance](./media/analysis-services-monitor/aas-monitor-chart.png)

<a id="#server-metrics"></a>

## <a name="server-metrics"></a>Indicateurs de performances de serveur

Ce tableau permet de déterminer les meilleurs indicateurs de performances pour votre scénario d’analyse. Seuls ceux appartenant à la même unité peuvent être affichés sur un même graphique.

|Métrique|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|
|---|---|---|---|---|
|CommandPoolJobQueueLength|Longueur de la file d’attente des travaux du pool de commandes|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads de commandes.|
|CurrentConnections|Connexion : connexions actuelles|Count|Average|Nombre actuel de connexions client établies.|
|CurrentUserSessions|Sessions utilisateur actuelles|Count|Average|Nombre actuel de sessions utilisateur établies.|
|mashup_engine_memory_metric|Mémoire du moteur M|Octets|Average|Utilisation de la mémoire par les processus de moteur mashup|
|mashup_engine_qpu_metric|QPU du moteur M|Count|Average|Utilisation des QPU par les processus de moteur mashup|
|memory_metric|Mémoire|Octets|Average|Mémoire. Plage de 0 à 25 Go pour S1, de 0 à 50 Go pour S2 et de 0 à 100 Go pour S4|
|memory_thrashing_metric|Vidage de mémoire|Pourcentage|Average|Vidage de mémoire moyenne.|
|CleanerCurrentPrice|Mémoire : prix actuel du nettoyage|Count|Average|Prix actuel de la mémoire, $/octet/temps, normalisé à 1000.|
|CleanerMemoryNonshrinkable|Mémoire : mémoire de nettoyage non réductible|Octets|Average|Quantité de mémoire, en octets, qui ne doit pas être vidée par le nettoyage en arrière-plan.|
|CleanerMemoryShrinkable|Mémoire : mémoire de nettoyage réductible|Octets|Average|Quantité de mémoire, en octets, qui doit être vidée par le nettoyage en arrière-plan.|
|MemoryLimitHard|Mémoire : limite de mémoire physique|Octets|Average|Limite de mémoire physique, du fichier de configuration.|
|MemoryLimitHigh|Mémoire : limite de mémoire élevée|Octets|Average|Limite de mémoire élevée, du fichier de configuration.|
|MemoryLimitLow|Mémoire : limite de mémoire basse|Octets|Average|Limite de mémoire basse, du fichier de configuration.|
|MemoryLimitVertiPaq|Mémoire : limite de mémoire VertiPaq|Octets|Average|Limite en mémoire, du fichier de configuration.|
|MemoryUsage|Mémoire : utilisation de la mémoire|Octets|Average|Utilisation de la mémoire du processus serveur telle qu’utilisée dans le calcul du coût de la mémoire de nettoyage. Équivaut au compteur Process\PrivateBytes, plus la taille des données mappées en mémoire, en ignorant la mémoire mappée ou allouée par le moteur d’analyse de mémoire (VertiPaq) dépassant la limite de mémoire du moteur.|
|private_bytes_metric|Octets privés |Octets|Average|Quantité totale de mémoire allouée par le processus du moteur Analysis Services et les processus de conteneur mashup, à l’exclusion de la mémoire partagée avec d’autres processus.|
|virtual_bytes_metric|Octets virtuels |Octets|Average|Taille actuelle de l’espace d’adressage virtuel utilisé par le processus du moteur Analysis Services et les processus de conteneur mashup.|
|mashup_engine_private_bytes_metric|Octets privés du moteur M |Octets|Average|Quantité totale de mémoire allouée par les processus de conteneur mashup, à l’exclusion de la mémoire partagée avec d’autres processus.|
|mashup_engine_virtual_bytes_metric|Octets virtuels du moteur M |Octets|Average|Taille actuelle de l’espace d’adressage virtuel utilisé par les processus de conteneur mashup.|
|Quota|Mémoire : quota|Octets|Average|Quota de mémoire actuel, en octets. Le quota de mémoire est également appelé réserve de mémoire ou d’allocation.|
|QuotaBlocked|Mémoire : quota bloqué|Count|Average|Nombre actuel de requêtes de quota qui sont bloquées en attendant la libération d’autres quotas de mémoire.|
|VertiPaqNonpaged|Mémoire : réserve non paginée VertiPaq|Octets|Average|Octets de mémoire verrouillée dans la plage de travail pour utilisation par le moteur en mémoire.|
|VertiPaqPaged|Mémoire : réserve paginée VertiPaq|Octets|Average|Octets de mémoire paginée utilisée pour les données en mémoire.|
|ProcessingPoolJobQueueLength|Longueur de la file d’attente des travaux du pool de traitement|Count|Average|Nombre de travaux autres que d’E/S contenus dans la file d’attente du pool de threads de traitement.|
|RowsConvertedPerSec|Traitement : lignes converties par seconde|CountPerSecond|Average|Taux de lignes converties lors du traitement.|
|RowsReadPerSec|Traitement : lignes lues par seconde|CountPerSecond|Average|Taux de lignes lues à partir de toutes les bases de données relationnelles.|
|RowsWrittenPerSec|Traitement : lignes écrites par seconde|CountPerSecond|Average|Taux de lignes écrites lors du traitement.|
|qpu_metric|QPU|Count|Average|QPU. Plage de 0 à 100 pour S1, de 0 à 200 pour S2 et de 0 à 400 pour S4|
|QueryPoolBusyThreads|Threads occupés du pool de threads de requêtes|Count|Average|Nombre de threads occupés dans le pool de threads de requêtes.|
|SuccessfullConnectionsPerSec|Connexions réussies par seconde|CountPerSecond|Average|Taux de connexions terminées réussies.|
|CommandPoolBusyThreads|Threads : threads occupés du pool commandes|Count|Average|Nombre de threads occupés dans le pool de threads de commandes.|
|CommandPoolIdleThreads|Threads : threads inactifs du pool commande|Count|Average|Nombre de threads inactifs dans le pool de threads de commandes.|
|LongParsingBusyThreads|Threads : threads d’analyse longue occupés|Count|Average|Nombre de threads occupés dans le pool de threads d’analyse longue.|
|LongParsingIdleThreads|Threads : threads d’analyse longue inactifs|Count|Average|Nombre de threads inactifs dans le pool de threads d’analyse longue.|
|LongParsingJobQueueLength|Threads : durée de file d’attente des travaux d’analyse longue|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads d’analyse longue.|
|ProcessingPoolIOJobQueueLength|Threads : longueur de la file d’attente des travaux d’E/S du pool de traitement|Count|Average|Nombre de travaux d’E/S contenus dans la file d’attente du pool de threads de traitement.|
|ProcessingPoolBusyIOJobThreads|Threads : traitement des threads de travail d’E/S occupés du pool|Count|Average|Nombre de threads pour les travaux d’E/S en cours d’exécution dans le pool de threads de traitement.|
|ProcessingPoolBusyNonIOThreads|Threads : traitement des threads de travail autres qu’E/S occupés du pool|Count|Average|Nombre de threads pour les travaux autres que d’E/S en cours d’exécution dans le pool de threads de traitement.|
|ProcessingPoolIdleIOJobThreads|Threads : traitement des threads de travail d’E/S ignorés du pool|Count|Average|Nombre de threads inactifs pour les travaux d’E/S le pool de threads de traitement.|
|ProcessingPoolIdleNonIOThreads|Threads : traitement des threads de travail autres qu’E/S inactifs du pool|Count|Average|Nombre de threads inactifs le pool de threads de traitement dédiés aux travaux autres qu’E/S.|
|QueryPoolIdleThreads|Threads : threads inactifs du pool de requêtes|Count|Average|Nombre de threads inactifs pour les travaux d’E/S le pool de threads de traitement.|
|QueryPoolJobQueueLength|Threads : longueur de file d’attente de travaux du pool de requêtes|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads de requêtes.|
|ShortParsingBusyThreads|Threads : threads d’analyse courte occupés|Count|Average|Nombre de threads occupés dans le pool de threads d’analyse courte.|
|ShortParsingIdleThreads|Threads : threads d’analyse courte inactifs|Count|Average|Nombre de threads inactifs dans le pool de threads d’analyse courte.|
|ShortParsingJobQueueLength|Threads : durée de file d’attente des travaux d’analyse courte|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads d’analyse courte.|
|TotalConnectionFailures|Nombre total d’échecs de connexion|Count|Average|Total des échecs de tentatives de connexion.|
|TotalConnectionRequests|Nombre total de demandes de connexion|Count|Average|Nombre total de demandes de connexion. |

## <a name="next-steps"></a>Étapes suivantes
[Vue d’ensemble d’Azure Monitor](../azure-monitor/overview.md)      
[Bien démarrer avec Azure Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md)      
[Indicateurs de performances dans l’API REST Azure Monitor](/rest/api/monitor/metrics)
