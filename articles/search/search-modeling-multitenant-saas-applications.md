---
title: Multilocation et isolation de contenu
titleSuffix: Azure Cognitive Search
description: Découvrez les modèles de conception courants pour les applications SaaS mutualisées lors de l’utilisation de Recherche cognitive Azure.
manager: nitinme
author: LiamCavanagh
ms.author: liamca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/25/2020
ms.openlocfilehash: cd21197d6d1559b681ae622b974f6eb7ba95ad3d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91397366"
---
# <a name="design-patterns-for-multitenant-saas-applications-and-azure-cognitive-search"></a>Modèles de conception pour les applications SaaS mutualisées et Recherche cognitive Azure

Une application mutualisée est une application qui fournit les mêmes services et fonctionnalités à plusieurs clients qui ne peuvent pas voir ni partager les données d’un autre client. Ce document aborde les stratégies d’isolation de client pour les applications mutualisées conçues avec Recherche cognitive Azure.

## <a name="azure-cognitive-search-concepts"></a>Concepts de Recherche cognitive Azure
En tant que solution SaaS (search-as-a-service), [Recherche cognitive Azure](search-what-is-azure-search.md) permet aux développeurs d’ajouter des expériences de recherche enrichies dans les applications sans avoir à gérer d’infrastructure, ni devenir un expert en matière de récupération d’informations. Les données sont téléchargées vers le service, puis stockées dans le cloud. À l’aide de requêtes simples dans l’API Recherche cognitive Azure, les données peuvent ensuite être modifiées et faire l’objet de recherches. 

### <a name="search-services-indexes-fields-and-documents"></a>Rechercher des services, des index, des champs et des documents

Avant d’aborder les modèles de conception, il est important de comprendre certains concepts de base.

Lorsque vous utilisez Recherche cognitive Azure, vous vous abonnez à un *service de recherche*. Lorsque les données sont téléchargées vers Recherche cognitive Azure, elles sont stockées dans un *index* au sein du service de recherche. Un seul service peut contenir plusieurs index. Pour utiliser les concepts familiers des bases de données, le service de recherche peut être comparé à une base de données, tandis que les index au sein d’un service peuvent être comparés aux tables dans une base de données.

Chaque index au sein d’un service de recherche possède son propre schéma, qui est défini par un certain nombre de *champs*personnalisables. Les données sont ajoutées à un index Recherche cognitive Azure sous la forme de *documents*individuels. Chaque document doit être téléchargé dans un index spécifique et doit respecter le schéma de cet index. Lors de la recherche de données à l’aide de Recherche cognitive Azure, les requêtes de recherche en texte intégral sont exécutées sur un index spécifique.  Pour comparer ces concepts à ceux d’une base de données, les champs peuvent être comparés aux colonnes d’une table et les documents peuvent être comparés aux lignes.

### <a name="scalability"></a>Extensibilité
Tout service Recherche cognitive Azure dans le [niveau tarifaire](https://azure.microsoft.com/pricing/details/search/) Standard peut évoluer en deux dimensions : stockage et disponibilité.

* *partitions* pour augmenter le stockage d’un service de recherche.
* *réplicas* à un service pour augmenter le débit des requêtes qu’un service de recherche peut gérer.

L’ajout et la suppression de partitions et de réplicas lorsque nécessaire permet à la capacité du service de recherche d’augmenter en fonction de la quantité de données et du trafic demandés par l’application. Pour qu’un service de recherche obtienne un [Contrat de niveau de service](https://azure.microsoft.com/support/legal/sla/search/v1_0/)en lecture, deux réplicas sont nécessaires. Pour qu’un service obtienne un [Contrat de niveau de service](https://azure.microsoft.com/support/legal/sla/search/v1_0/)en lecture et en écriture, trois réplicas sont nécessaires.

### <a name="service-and-index-limits-in-azure-cognitive-search"></a>Limites du service et de l’index dans Recherche cognitive Azure
Il existe différents [niveaux tarifaires](https://azure.microsoft.com/pricing/details/search/) dans la Recherche cognitive Azure et chaque niveau présente des [limites et quotas](search-limits-quotas-capacity.md) différents. Certaines de ces limites se situent au niveau du service, certaines au niveau de l’index, et d’autres au niveau de la partition.

|  | De base | Standard1 | Standard2 | Standard3 | Standard3 HD |
| --- | --- | --- | --- | --- | --- |
| **Nombre maximal de réplicas par service** |3 |12 |12 |12 |12 |
| **Nombre maximal de partitions par service** |1 |12 |12 |12 |3 |
| **Nombre maximal d’unités de recherche (réplicas*partitions) par service** |3 |36 |36 |36 |36 (3 partitions max.) |
| **Stockage maximal par service** |2 Go |300 Go |1,2 To |2,4 To |600 Go |
| **Stockage maximal par partition** |2 Go |25 Go |100 Go |200 Go |200 Go |
| **Nombre maximal d’index par service** |5 |50 |200 |200 |3000 (1 000 index max. par partition) |

#### <a name="s3-high-density"></a>Haute densité S3
Dans le niveau tarifaire S3 de Recherche cognitive Azure, il existe une option pour le mode haute densité (HD) conçu spécifiquement pour les scénarios mutualisés. Dans de nombreux cas, il est nécessaire de prendre en charge un grand nombre de clients plus petits sous un seul service pour tirer parti des avantages de simplicité et de rentabilité.

S3 HD permet de réunir les nombreux index de petite taille sous la gestion d’un seul service de recherche en troquant la possibilité d’effectuer un scale-out des index à l’aide de partitions contre la possibilité d’héberger plus d’index dans un seul service.

Un service S3 est conçu pour héberger un nombre fixe d’index (200 maximum) et permet à chaque index d’évoluer horizontalement au fur et à mesure que de nouvelles partitions sont ajoutées au service. L’ajout de partitions à des services S3 HD augmente le nombre maximal d’index que le service peut héberger. La taille maximale idéale pour un index S3 HD est d’environ 50 à 80 Go, bien qu’il n’y ait pas de limite de taille matérielle sur chaque index imposé par le système.

## <a name="considerations-for-multitenant-applications"></a>Considérations relatives aux applications mutualisées
Les applications mutualisées doivent distribuer efficacement les ressources entre les clients tout en conservant un certain niveau de confidentialité entre les différents clients. Il existe quelques considérations à prendre en compte lors de la conception de l’architecture pour ce type d’application :

* *Isolation des locataires :* les développeurs d’applications doivent prendre les mesures appropriées pour s’assurer qu’aucun locataire ne bénéficie d’un accès non autorisé ou non désiré aux données d’autres locataires. Au-delà de la confidentialité des données, les stratégies d’isolation des clients nécessitent une gestion efficace des ressources partagées et la protection contre les voisins bruyants.
* *Coût des ressources cloud :* comme pour toute autre application, les solutions logicielles doivent rester compétitives au niveau du coût en tant que composant d’une application mutualisée.
* *Facilité des opérations :* lors du développement d’une architecture mutualisée, l’impact sur les opérations et la complexité de l’application est un facteur important. Recherche cognitive Azure propose un [Contrat de niveau de service à 99,9 %](https://azure.microsoft.com/support/legal/sla/search/v1_0/).
* *Envergure internationale :* les applications mutualisées devront peut-être servir efficacement des locataires qui sont répartis dans le monde entier.
* *Scalabilité :* Les développeurs d’applications doivent trouver l’équilibre entre le fait de maintenir un niveau de complexité des applications suffisamment faible et la conception de l’application de façon à ce qu’elle évolue avec le nombre de locataires, ainsi que la taille des données et la charge de travail des locataires.

Recherche cognitive Azure propose quelques limites qui peuvent être utilisées pour isoler les données et la charge de travail des clients.

## <a name="modeling-multitenancy-with-azure-cognitive-search"></a>Modélisation de la mutualisation avec Recherche cognitive Azure
Dans le cas d’un scénario mutualisé, le développeur de l’application consomme un ou plusieurs services de recherche et répartit les clients entre les services, les index ou les deux. Recherche cognitive Azure offre quelques modèles courants pour la modélisation d’un scénario mutualisé :

1. *Index par locataire :* chaque locataire a son propre index dans un service de recherche qui est partagé avec d’autres locataires.
2. *Service par locataire :* chaque locataire possède son propre service Recherche cognitive Azure dédié, pour un niveau de séparation maximal entre les données et la charge de travail.
3. *Combinaison des deux :* les locataires les plus volumineux et actifs se voient attribuer des services dédiés, tandis que les locataires plus petits se voient attribuer des index individuels au sein de services partagés.

## <a name="1-index-per-tenant"></a>1. Index par client

:::image type="content" source="media/search-modeling-multitenant-saas-applications/azure-search-index-per-tenant.png" alt-text="Une image du modèle d’index par client" border="false":::

Dans un modèle d’index par client, plusieurs clients occupent un seul service Recherche cognitive Azure où chaque client a son propre index.

Les clients bénéficient de l’isolation des données, car toutes les requêtes de recherche et les opérations sur les documents sont émises au niveau de l’index dans Recherche cognitive Azure. Dans la couche d’application, il est entendu qu’il est nécessaire de diriger le trafic des divers clients vers les index appropriés, tout en gérant les ressources au niveau du service pour tous les clients.

Un attribut clé du modèle d’index par client est la possibilité pour le développeur d’applications d’augmenter la capacité d’un service de recherche pour les clients de l’application. Si la répartition de la charge de travail est inégale entre les clients, la combinaison optimale des clients peut être distribuée entre les index d’un service de recherche pour prendre en charge un nombre de clients très actifs, gourmands en ressources, tout en gérant simultanément une longue file de clients moins actifs. Le compromis réside dans l’incapacité du modèle à gérer les situations où chaque client est simultanément très actif.

Le modèle d’index par client sert de base à un modèle de coût variable, où un service Recherche cognitive Azure entier est acheté d’avance, puis rempli avec des clients. Cela permet d’attribuer la capacité inutilisée aux essais et aux comptes gratuits.

Pour les applications avec une envergure internationale, le modèle d’index par client n’est peut-être pas le plus efficace. Si les clients d’une application sont répartis dans le monde entier, un service distinct peut être nécessaire pour chaque région, ce qui peut dupliquer les coûts pour chacun d’eux.

Recherche cognitive Azure permet la croissance des index individuels et du nombre total d’index. Si un niveau tarifaire approprié est choisi, des réplicas et des partitions peuvent être ajoutés au service de recherche entier lorsqu’un index individuel au sein du service devient trop important en termes de stockage ou de trafic.

Si le nombre total d’index devient trop important pour un seul service, un autre service doit être configuré pour prendre en charge les nouveaux clients. Si les index doivent être déplacés entre les services de recherche lorsque de nouveaux services sont ajoutés, les données de l’index doivent être copiées manuellement d’un index vers l’autre car le déplacement d’un index n’est pas autorisé dans Recherche cognitive Azure.

## <a name="2-service-per-tenant"></a>2. Service par client

:::image type="content" source="media/search-modeling-multitenant-saas-applications/azure-search-service-per-tenant.png" alt-text="Une image du modèle d’index par client" border="false":::

Dans une architecture de service par client, chaque client possède son propre service de recherche.

Dans ce modèle, l’application atteint le niveau maximal d’isolation pour ses clients. Chaque service bénéficie d’un débit et d’un stockage dédiés pour gérer les requêtes de recherche, ainsi que des clés d’API distinctes.

Pour les applications où chaque client a une grande envergure ou si la charge de travail varie peu d’un client à l’autre, le modèle de service par client constitue un choix efficace car les ressources ne sont pas partagées entre les charges de travail de différents clients.

Un modèle de service par client offre également l’avantage d’un modèle à coût prévisible et fixe. Il n’existe aucun investissement initial pour un service de recherche entier jusqu'à ce qu’il y ait un client pour le remplir. Cependant, le coût par client est supérieur au modèle d’index par client.

Le modèle de service par client constitue une solution efficace pour les applications d’envergure internationale. Avec des clients répartis géographiquement, il est facile de placer le service de chaque client dans la région appropriée.

Les défis liés à la mise à l’échelle de ce modèle se présentent lorsque la croissance des clients individuels surpasse celle de leur service. Actuellement, Recherche cognitive Azure ne prend pas en charge la mise à niveau du niveau de tarification d’un service de recherche, donc toutes les données doivent être copiées manuellement vers un nouveau service.

## <a name="3-mixing-both-models"></a>3. Combinaison des deux modèles
Un autre modèle pour la modélisation mutualisée est de combiner les stratégies d’index par client et de service par client.

En mélangeant les deux modèles, les clients les plus volumineux d’une application peuvent occuper des services dédiés, tandis que la longue file de clients moins actifs et plus petits peut occuper des index dans un service partagé. Ce modèle garantit que les clients les plus volumineux bénéficient toujours de performances élevées du service tout en protégeant les clients plus petits des voisins bruyants.

Cependant, l’implémentation de cette stratégie repose sur la capacité à prévoir quels clients nécessiteront un service dédié au lieu d’un index dans un service partagé. La complexité de l’application augmente avec la nécessité de gérer les deux modèles d’architecture mutualisée.

## <a name="achieving-even-finer-granularity"></a>Atteindre une granularité encore plus fine
Les modèles de conception ci-dessus pour les scénarios mutualisés dans Recherche cognitive Azure supposent qu’il existe une étendue uniforme, où chaque client constitue une instance entière d’une application. Toutefois, les applications peuvent parfois gérer plusieurs étendues plus petites.

Si les modèles service par client et index par client ne sont pas des étendues suffisamment petites, il est possible de modéliser un index pour atteindre un degré de granularité encore plus fin.

Pour qu’un seul index se comporte différemment pour des points de terminaison clients différents, il est possible d’ajouter un champ à un index qui désigne une certaine valeur pour chaque client possible. À chaque fois qu’un client appelle Recherche cognitive Azure pour interroger ou modifier un index, le code de l’application cliente spécifie la valeur appropriée pour ce champ à l’aide de la fonctionnalité [filter](./query-odata-filter-orderby-syntax.md) de Recherche cognitive Azure au moment de la requête.

Cette méthode peut être utilisée pour obtenir une fonctionnalité de comptes d’utilisateurs distincts, de niveaux d’autorisation distincts et même d’applications distinctes.

> [!NOTE]
> Utiliser l’approche décrite ci-dessus pour configurer un index unique pour servir plusieurs locataires affecte la pertinence des résultats de recherche. Les notes de pertinence de la recherche sont calculées au niveau de l’index et non au niveau du locataire. Ainsi, toutes les données des locataires sont incorporées dans les statistiques sous-jacentes des notes de pertinence, comme la fréquence des termes.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
Recherche cognitive Azure est un choix attrayant pour de nombreuses applications. Lorsque vous évaluez les différents modèles de conception pour les applications mutualisées, consultez les [différents niveaux tarifaires](https://azure.microsoft.com/pricing/details/search/) et les [limites de service](search-limits-quotas-capacity.md) respectives pour mieux adapter la Recherche cognitive Azure aux architectures et aux charges de travail de toutes tailles.

Toutes questions relatives à Recherche cognitive Azure et les scénarios partagés au sein d’une architecture mutualisée peuvent être envoyés vers azuresearch_contact@microsoft.com.