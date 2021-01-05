---
title: Mettre à l’échelle pour les performances
titleSuffix: Azure Cognitive Search
description: Découvrez les techniques et les bonnes pratiques pour optimiser les performances de la Recherche cognitive Azure et configurer une mise à l’échelle optimale.
manager: nitinme
author: LiamCavanagh
ms.author: liamca
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/14/2020
ms.openlocfilehash: 362d5f2046ff4e9ba52dd2e73433cc39e80f7a50
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93420595"
---
# <a name="scale-for-performance-on-azure-cognitive-search"></a>Mettre à l’échelle pour les performances de la Recherche cognitive Azure

Cet article décrit les meilleures pratiques applicables dans le cadre de scénarios avancés présentant des exigences complexes en matière d’extensibilité et de disponibilité.

## <a name="start-with-baseline-numbers"></a>Commencer avec des valeurs de référence

Avant d’entreprendre un déploiement à plus grande échelle, vous devez savoir à quoi ressemble une charge de requête standard. Les instructions ci-après peuvent vous aider à atteindre les valeurs de requête de référence.

1. Choisissez une latence cible (ou durée maximale) nécessaire à l’exécution d’une recherche standard .

1. Créez et testez une charge de travail réelle avec votre service de recherche en utilisant un jeu de données réaliste pour mesurer les taux de latence.

1. Commencez par un petit nombre de requêtes par seconde (RPS), puis augmentez graduellement le nombre d’exécutions dans le test jusqu’à ce que la latence des requêtes soit inférieure à la cible prédéfinie. Il s’agit d’un test d’évaluation important qui vous aidera à planifier la mise à l’échelle à mesure que l’utilisation de votre application s’intensifie.

1. Dans la mesure du possible, réutilisez les connexions HTTP. Si vous utilisez le Kit de développement logiciel (SDK) .NET Recherche cognitive Azure, cela signifie que vous devez réutiliser une instance ou une instance [SearchClient](/dotnet/api/azure.search.documents.searchclient) et, si vous utilisez l’API REST, vous devez réutiliser une instance HttpClient unique.

1. Variez la substance des requêtes afin que la recherche s’effectue sur différentes parties de votre index. Il est important de varier les requêtes, car si vous exécutez continuellement les mêmes requêtes de recherche, la mise en cache des données commencera à offrir de meilleures performances qu’avec un ensemble de requêtes plus disparates.

1. Variez la structure des requêtes afin d’obtenir différents types de requêtes. En effet, le niveau de performance varie selon les requêtes de recherche. Par exemple, une recherche de document ou une suggestion de recherche s’exécute plus rapidement qu’une requête comportant un nombre important de facettes et de filtres. La composition de test doit inclure diverses requêtes, approximativement dans les mêmes proportions que celles que vous rencontreriez en production.  

Lors de la création de ces charges de travail de test, certaines caractéristiques de la Recherche cognitive Azure doivent être prises en compte :

+ L’envoi (push) simultané d’un nombre excessif de requêtes de recherche risque de surcharger votre service. Dans ce cas, vous verrez les codes de réponse HTTP 503. Pour éviter l’obtention du code 503 lors du test, commencez avec différentes plages de requêtes de recherche pour visualiser les différences de taux de latence à mesure que vous ajoutez d’autres requêtes.

+ Le service Recherche cognitive Azure n’exécute pas de tâches d’indexation en arrière-plan. Si votre service gère les charges de travail de requête et d’indexation simultanément, prenez cela en compte en introduisant des travaux d’indexation dans vos tests de requête ou en explorant les options d’exécution des travaux d’indexation pendant les heures creuses.

> [!Tip]
> Vous pouvez simuler une charge de requête réaliste à l’aide des outils de test de charge. Essayez de [tester la charge avec Azure DevOps](/azure/devops/test/load-test/get-started-simple-cloud-load-test) ou utilisez l’une de ces [alternatives](/azure/devops/test/load-test/overview#alternatives).

## <a name="scale-for-high-query-volume"></a>Mettre à l’échelle pour un volume de requêtes élevé

Un service est surchargé quand les requêtes prennent trop de temps ou que le service commence à abandonner des requêtes. Dans ces cas de figure, vous pouvez résoudre le problème de l’une des manières suivantes :

+ **Ajouter des réplicas**  

  Chaque réplica est une copie de vos données, ce qui permet au service d’équilibrer la charge des requêtes sur plusieurs copies.  L’équilibrage de la charge et la réplication des données sont entièrement gérés par la Recherche cognitive Azure, mais vous pouvez à tout moment changer le nombre de réplicas alloués à votre service. Vous pouvez allouer jusqu'à 12 réplicas dans un service de recherche Standard, et 3 dans un service de recherche de base. Les réplicas peuvent être ajustés sur le [portail Azure](search-create-service-portal.md) ou via [PowerShell](search-manage-powershell.md).

+ **Créer un service à un niveau supérieur**  

  la Recherche cognitive Azure est proposée dans [plusieurs niveaux](https://azure.microsoft.com/pricing/details/search/), chacun d’eux offrant des niveaux de performances différents. Dans certains cas, le nombre de requêtes peut être trop élevé pour qu’elles puissent être traitées dans le niveau que vous avez choisi, même si les réplicas sont optimisés. Envisagez alors de passer à un niveau supérieur plus performant, tel que le niveau S3 Standard qui est conçu pour des scénarios impliquant un grand nombre de documents et des charges de requêtes extrêmement élevées.

## <a name="scale-for-slow-individual-queries"></a>Mettre à l’échelle pour les requêtes individuelles lentes

Une requête unique dont l’exécution prend trop de temps peut également entraîner des taux de latence élevés. Dans ce cas, l’ajout de réplicas n’offrira aucune amélioration. Vous pouvez tenter de résoudre ce problème de deux manières :

+ **Augmenter les partitions**

  Une partition divise les données entre des ressources de calcul supplémentaires. Deux partitions divisent les données en deux, trois partitions les divisent en trois, et ainsi de suite. L’un des avantages découlant de cette opération réside dans l’accélération potentielle de l’exécution des requêtes les plus lentes grâce au calcul parallèle. Nous avons constaté une parallélisation sur les requêtes qui affichent une faible sélectivité, telles que les requêtes portant sur de nombreux documents ou les facettes comptabilisant un grand nombre de documents. Dans la mesure où des calculs significatifs sont nécessaires pour évaluer la pertinence des documents ou pour comptabiliser les documents, l’ajout de partitions supplémentaires contribue à accélérer l’exécution des requêtes.  
   
  Il peut y avoir un maximum de 12 partitions dans le service de recherche de niveau Standard, et 1 partition dans le service de recherche de niveau De base. Les réplicas peuvent être ajustés sur le [portail Azure](search-create-service-portal.md) ou via [PowerShell](search-manage-powershell.md).

+ **Limiter les champs à cardinalité élevée**

  un champ à cardinalité élevée est un champ utilisable comme facette ou comme filtre et contenant un grand nombre de valeurs uniques. Ce champ consomme donc une quantité substantielle de ressources lors du calcul des résultats. Par exemple, la définition d’un champ ID produit ou Description en tant que champ utilisable comme facette ou comme filtre est considérée comme une cardinalité élevée, car la plupart des valeurs sont uniques pour chaque document. Dans la mesure du possible, limitez le nombre de champs à cardinalité élevée.

+ **Passer à un niveau de Recherche supérieur**  

  Choisir un niveau de Recherche cognitive Azure supérieur est une autre façon d’améliorer les performances des requêtes lentes. Chaque niveau supérieur se caractérise par un processeur plus rapide et par une plus grande quantité de mémoire. Ces deux aspects ont un impact positif sur les performances des requêtes.

## <a name="scale-for-availability"></a>Mettre à l’échelle pour la disponibilité

Les réplicas permettent non seulement de réduire la latence des requêtes, mais ils peuvent également offrir une plus grande disponibilité. Avec un seul réplica, attendez-vous à des temps d’arrêt périodiques en raison du redémarrage du serveur après les mises à jour logicielles ou lorsque d’autres événements de maintenance se produisent. Par conséquent, il est important d’évaluer si votre application requiert une haute disponibilité des recherches (requêtes) ainsi que des écritures (événements d’indexation). La Recherche cognitive Azure offre des options de contrat de niveau de service sur tous les services de recherche payants avec les attributs suivants :

+ Deux réplicas pour la haute disponibilité des charges de travail en lecture seule (requêtes)

+ Trois réplicas (ou plus) pour la haute disponibilité des charges de travail en lecture-écriture (requêtes et indexation)

Pour plus d’informations, consultez le [Contrat de niveau de service de la Recherche cognitive Azure](https://azure.microsoft.com/support/legal/sla/search/v1_0/).

Étant donné que les réplicas sont des copies de vos données, l’utilisation de plusieurs réplicas permet au service Recherche cognitive Azure de gérer les redémarrages et la maintenance des machines pour un réplica spécifique, alors que les requêtes continuent de s’exécuter sur d’autres réplicas. À l’inverse, si vous retirez des réplicas, vous constaterez une dégradation des performances de requête, en supposant que ces réplicas constituaient des ressources sous-exploitées.

## <a name="scale-for-geo-distributed-workloads-and-geo-redundancy"></a>Mettre à l’échelle pour les charges de travail géodistribuées et la géoredondance

Dans le cas des charges de travail géodistribuées, les utilisateurs éloignés du centre de données hôte font face à des taux de latence plus élevés. L’une des manières d’atténuer ce problème consiste à approvisionner plusieurs services de recherche dans des régions plus proches de ces utilisateurs.

La Recherche cognitive Azure n’offre actuellement pas de méthode automatisée de géo-réplication des index de Recherche cognitive Azure sur différentes régions, mais quelques techniques permettent d’implémenter et de gérer simplement ce processus. Elles sont présentées dans les sections suivantes.

L’objectif d’un ensemble géodistribué de services de recherche est de disposer de plusieurs index disponibles dans au moins deux régions, et de rediriger l’utilisateur vers le service Recherche cognitive Azure qui offre la plus faible latence, comme illustré dans cet exemple :

   ![Tableau croisé des services par région][1]

### <a name="keep-data-synchronized-across-multiple-services"></a>Garantir la synchronisation des données entre plusieurs services

Il existe deux méthodes pour synchroniser vos services de recherche distribués : [l’indexeur de Recherche cognitive Azure](search-indexer-overview.md) ou l’API Push (également appelée [API REST de Recherche cognitive Azure](/rest/api/searchservice/)).  

### <a name="use-indexers-for-updating-content-on-multiple-services"></a>Mettre à jour le contenu dans plusieurs services à l’aide d’indexeurs

Si vous utilisez déjà un indexeur sur un service unique, vous pouvez configurer un second indexeur sur un deuxième service pour qu’il utilise le même objet de source de données, en extrayant les données au même emplacement. Chacun des services de chaque région dispose de son propre indexeur et d’un index cible (votre index de recherche n’est pas partagé, ce qui signifie que les données sont dupliquées), mais chaque indexeur référence la même source de données.

Voici une vue d’ensemble de l’aspect d’une telle architecture.

   ![Source de données unique avec indexeur distribué et combinaisons de service][2]

### <a name="use-rest-apis-for-pushing-content-updates-on-multiple-services"></a>Envoyer (push) les mises à jour de contenu à plusieurs services par le biais d’API REST

Si vous utilisez l’API REST Recherche cognitive Azure pour [envoyer (push) le contenu de votre index Recherche cognitive Azure](/rest/api/searchservice/update-index), vous pouvez assurer la synchronisation continue de vos différents services de recherche en envoyant les modifications à tous ces services chaque fois qu’une mise à jour est nécessaire. Dans votre code, veillez à gérer les cas dans lesquels la mise à jour d’un service de recherche échoue, mais réussit pour d’autres services de recherche.

## <a name="leverage-azure-traffic-manager"></a>Exploiter Azure Traffic Manager

[Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) vous permet d’acheminer les requêtes vers plusieurs sites géo-localisés et pris en charge par plusieurs services de recherche. Traffic Manager offre l’avantage de pouvoir tester la Recherche cognitive Azure pour vous assurer qu’il est disponible et de rediriger les utilisateurs vers d’autres services de recherche en cas d’interruption du service. En outre, si vous acheminez des requêtes de recherche via des sites Web Azure, Azure Traffic Manager permet d’équilibrer les charges lorsque le site web est opérationnel mais pas la Recherche cognitive Azure. Voici un exemple d’architecture tirant parti de Traffic Manager.

   ![Tableau croisé des services par région, avec Traffic Manager central][3]

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les niveaux tarifaires et les limites de service de chacun d’eux, consultez [Limites de service](search-limits-quotas-capacity.md). Consultez la page [Planification des capacités](search-capacity-planning.md) pour en savoir plus sur les combinaisons de partitions et de réplicas.

Pour une discussion sur les performances et des démonstrations des techniques présentées dans cet article, regardez la vidéo suivante :

> [!VIDEO https://channel9.msdn.com/Events/Microsoft-Azure/AzureCon-2015/ACON319/player]
> 

<!--Image references-->
[1]: ./media/search-performance-optimization/geo-redundancy.png
[2]: ./media/search-performance-optimization/scale-indexers.png
[3]: ./media/search-performance-optimization/geo-search-traffic-mgr.png