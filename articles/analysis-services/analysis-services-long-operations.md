---
title: Meilleures pratiques en matière d'opérations de longue durée dans Azure Analysis Services | Microsoft Docs
description: Cet article décrit les meilleures pratiques en matière d'opérations de longue durée.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: owend
ms.openlocfilehash: 4e069effae0cb7f834b2c3dac696d05304d841a7
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92014832"
---
# <a name="best-practices-for-long-running-operations"></a>Meilleures pratiques en matière d'opérations de longue durée

Dans Azure Analysis Services, un *nœud* représente une machine virtuelle hôte sur laquelle une ressource de serveur s'exécute. Certaines opérations, telles que les requêtes longues, les opérations d’actualisation et la synchronisation de la montée en puissance des requêtes, peuvent échouer si une ressource de serveur est déplacée vers un autre nœud. Dans ce scénario, les messages d’erreur courants sont les suivants :

- « Une erreur s’est produite lors de la tentative de localisation d’une requête XMLA longue. La requête a peut-être été interrompue en raison d'une mise à niveau du service ou d'un redémarrage du serveur. »
- « La tâche portant l'ID <guid>pour le modèle <database> a été annulée en raison d’une erreur du service (inactivité) avec le message « Annulation de la demande d’actualisation, car celle-ci était bloquée sans mise à jour. Il s'agit d'un problème de service interne. Soumettez à nouveau le travail ou envoyez un ticket pour obtenir de l’aide si ce problème se produit de façon répétée. »

Les opérations de longue durée peuvent être interrompues pour de nombreuses raisons. Par exemple, les mises à jour dans Azure, telles que : 
- Correctifs de système d'exploitation. 
- Mises à jour de sécurité
- Mises à jour de service Azure Analysis Services
- Mises à jour Service Fabric Service Fabric est un composant de plateforme utilisé par un certain nombre de services de cloud computing Microsoft, y compris Azure Analysis Services.

En plus des mises à jour qui se produisent dans le service, un déplacement naturel des services entre les nœuds intervient en raison de l’équilibrage de charge. Les déplacements de nœuds relèvent d'un point attendu d'un service cloud. Azure Analysis Services tente de réduire les impacts des déplacements des nœuds, même s'il est impossible de les annuler intégralement. 

En plus des déplacements de nœuds, d’autres défaillances peuvent se produire. Par exemple, un système de base de données de source de données peut être hors connexion ou la connectivité réseau peut avoir été perdue. Au cours de l’actualisation, si une partition contient 10 millions lignes et qu’une défaillance se produit à la 9 millionième ligne, il est impossible de redémarrer l’actualisation au point de défaillance. Le service doit redémarrer depuis le début. 

## <a name="refresh-rest-api"></a>Actualiser l’API REST

Les interruptions de service peuvent s'avérer difficiles pour les opérations de longue durée telles que l’actualisation des données. Azure Analysis Services intègre une API REST permettant d'atténuer les impacts négatifs des interruptions de service. Pour plus d'informations, consultez [Actualisation asynchrone avec l’API REST](analysis-services-async-refresh.md).
 
En plus de l’API REST, vous pouvez utiliser d’autres approches pour réduire les problèmes susceptibles de survenir lors des opérations d’actualisation de longue durée. L’objectif principal consiste à éviter de devoir redémarrer l’opération d’actualisation du début, en procédant plutôt à des actualisations dans de plus petits lots pouvant être validés par étapes. 
 
L’API REST permet un tel redémarrage, sans pour autant offrir une flexibilité totale en termes de création et de suppression de partition. Si un scénario requiert des opérations de gestion de données complexes, la logique de votre solution doit inclure une certaine forme de traitement par lot. Par exemple, en utilisant des transactions pour traiter des données dans plusieurs lots distincts, un échec peut ne pas entraîner un redémarrage depuis le début, mais depuis un point de contrôle intermédiaire. 
 
## <a name="scale-out-query-replicas"></a>Monter en charge des réplicas de requête

Qu'elles utilisent une logique REST ou personnalisée, les requêtes d’application cliente peuvent renvoyer des résultats incohérents ou intermédiaires lors du traitement des lots. Si les requêtes d'application cliente doivent renvoyer des données cohérentes et que les données du modèle se trouvent dans un état intermédiaire, vous pouvez utiliser la [montée en charge](analysis-services-scale-out.md) avec des réplicas de requête en lecture seule.

Lorsque vous utilisez des réplicas de requête en lecture seule et si les actualisations interviennent par lots, les utilisateurs d'application cliente peuvent continuer d'interroger le précédent instantané des données sur les réplicas en lecture seule. Une fois les actualisations terminées, une opération de synchronisation peut intervenir pour mettre à jour les réplicas en lecture seule.


## <a name="next-steps"></a>Étapes suivantes

[Actualisation asynchrone avec l’API REST](analysis-services-async-refresh.md)  
[Scale-out d’Azure Analysis Services](analysis-services-scale-out.md)  
[Haute disponibilité Analysis Services](analysis-services-bcdr.md)  
[Guide du mécanisme de nouvelle tentative relatif aux services Azure](/azure/architecture/best-practices/retry-service-specific)