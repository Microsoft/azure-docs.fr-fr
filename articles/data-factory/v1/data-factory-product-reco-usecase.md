---
title: Cas d'utilisation Data Factory - Recommandations de produits
description: Découvrez un cas d'utilisation implémenté à l'aide d’Azure Data Factory et d'autres services.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.assetid: 6f1523c7-46c3-4b8d-9ed6-b847ae5ec4ae
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: fc1e0398f98dc72d57039530dce9751af012f154
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495291"
---
# <a name="use-case---product-recommendations"></a>Cas d’utilisation - Recommandations de produits
Azure Data Factory est l’un des nombreux services permettant d’implémenter la solution d’accélérateurs Cortana Intelligence Suite.  Consultez la page [Cortana Intelligence Suite](https://www.microsoft.com/cortanaanalytics) pour plus de détails. Dans ce document, nous décrivons un cas d’utilisation courant que les utilisateurs Azure ont déjà résolu et implémenté à l’aide d’Azure Data Factory et d’autres services Cortana Intelligence.

## <a name="scenario"></a>Scénario
Les détaillants en ligne souhaitent généralement inciter leurs clients à acheter des produits en leur présentant les articles qu’ils sont les plus susceptibles d’apprécier et par conséquent d’acheter. Pour ce faire, les détaillants en ligne doivent personnaliser l’expérience en ligne des utilisateurs à l’aide des recommandations de produits personnalisées pour chaque utilisateur particulier. Ces recommandations personnalisées portent sur leurs données de comportement d’achat historiques et actuelles, les informations produit, les marques récemment ajoutées, et les données de segmentation produit et utilisateur.  En outre, les détaillants peuvent fournir des recommandations de produits basées sur l'analyse des habitudes d'utilisation globale de tous leurs utilisateurs.

L'objectif est d'optimiser le processus de conversion des visites en ventes effectives et d’augmenter ainsi le chiffre d’affaires.  Ils parviennent à effectuer cette conversion en proposant des recommandations de produits contextuelles basées sur les habitudes, les centres d’intérêt et les actions du client. Pour ce cas d’utilisation, nous prenons des détaillants en ligne comme exemple d’entreprises qui souhaitent optimiser pour leurs clients. Cependant, ces principes s'appliquent à toute entreprise cherchant à fidéliser ses clients autour de ses produits et services et à améliorer leur expérience d’achat grâce à des recommandations de produits personnalisées.

## <a name="challenges"></a>Défis
Les détaillants en ligne doivent relever de nombreux défis lorsqu’elles tentent d’implémenter ce type de cas d'utilisation. 

Tout d’abord, des données de différentes tailles et formes doivent être ingérées depuis plusieurs sources de données locales et dans le cloud. Ces données comprennent les données de produit, les données historiques de comportement client, et les données utilisateur lorsque ce dernier navigue sur le site du détaillant. 

Deuxièmement, les recommandations de produits personnalisées doivent être anticipées et évaluées de manière raisonnable et précise. Outre les informations sur les produits, les marques, les habitudes du client et les données du navigateur, les détaillants en ligne doivent également afficher les commentaires d’anciens clients afin d’aider l’utilisateur à mieux cibler les produits qui lui conviennent. 

Troisièmement, les recommandations doivent être immédiatement communiquées à l'utilisateur pour garantir une expérience d'achat et de navigation transparente, avec les informations les plus récentes et les plus pertinentes. 

Enfin, les détaillants doivent mesurer l'efficacité de leur approche par un suivi des ventes croisées et incitatives réalisées, et ajuster leurs prochaines recommandations.

## <a name="solution-overview"></a>Vue d'ensemble de la solution
Cet exemple de cas d’utilisation a été résolu et implémenté par de vrais utilisateurs Azure avec Azure Data Factory et d’autres services Cortana Intelligence, notamment [HDInsight](https://azure.microsoft.com/services/hdinsight/) et [Power BI](https://powerbi.microsoft.com/).

Le détaillant en ligne utilise un magasin d’objets blob Azure, un serveur SQL local, Azure SQL Database et un mini-Data Warehouse relationnel comme options de stockage des données tout au long du workflow.  Le magasin d'objets blob contient des informations sur les clients, des données sur les habitudes des clients et des informations sur les produits. Les informations sur les produits incluent des informations sur les marques de produits ainsi qu’un catalogue de produits stocké localement dans un Azure Synapse Analytics. 

Toutes les données sont combinées et chargées dans un système de recommandations de produit pour fournir des recommandations personnalisées selon les intérêts et les actions du client à mesure que l'utilisateur parcourt le catalogue de produits sur le site web. Le client peut également voir des produits liés à celui qu'il recherche, selon des modèles d'utilisation du site web qui ne sont spécifiques à aucun utilisateur.

![diagramme de cas d'utilisation](./media/data-factory-product-reco-usecase/diagram-1.png)

Des gigaoctets de fichiers journaux web bruts sont générés chaque jour sous forme de fichiers semi-structurés, à partir du site web du détaillant en ligne. Les fichiers journaux web bruts et les informations sur les clients et le catalogue de produits sont transmises régulièrement à un stockage d'objets blob Azure en utilisant le déplacement de données déployées globalement Data Factory comme service. Les fichiers journaux bruts de la journée sont partitionnés (par année et mois) dans le stockage d’objets blob pour un stockage à long terme.  [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/) sert à partitionner les fichiers journaux bruts dans le magasin d’objets blob et à traiter les journaux d’activité ingérés à l’échelle à l’aide de scripts Hive et Pig. Les données de journal d’activité web partitionnées sont alors traitées pour extraire les entrées nécessaires pour un système de recommandation, afin de générer les recommandations de produits personnalisées.

Le système de recommandation utilisé pour Machine Learning dans cet exemple est une plateforme de recommandation Machine Learning open source d’ [Apache Mahout](https://mahout.apache.org/).  N’importe quel modèle [Azure Machine Learning](https://azure.microsoft.com/services/machine-learning/) ou modèle personnalisé peut être appliqué au scénario.  Le modèle Mahout sert à prédire la similarité entre des éléments présents sur le site web en fonction de modèles d'utilisation globaux, et à générer des recommandations personnalisées selon un utilisateur individuel.

Enfin, le jeu de résultats des recommandations de produits personnalisées est déplacé vers un mini-Data Warehouse relationnel afin d’être utilisé par le site web du détaillant.  Le jeu de résultats peut également être accessible directement depuis le stockage d’objets blob par une autre application, ou déplacé vers des magasins supplémentaires pour d'autres consommateurs et cas d'utilisation.

## <a name="benefits"></a>Avantages
En optimisant leur stratégie de recommandation de produits et en l’alignant sur les objectifs commerciaux, la solution répond aux besoins de merchandising et de marketing du détaillant en ligne. En outre, ils peuvent faire fonctionner et gérer le flux de travail de recommandation de produits de manière efficace, fiable et économique. L’approche leur permet de mettre à jour facilement leur modèle et d’optimiser son efficacité sur la base des mesures de ventes concrétisées. Grâce à Azure Data Factory, ils ont pu se débarrasser de leur système de gestion manuelle des ressources de cloud, à la fois long et coûteux, pour passer à une gestion des ressources de cloud à la demande. Ainsi, ils ont été en mesure de gagner du temps et de l’argent, et de réduire le temps de déploiement de la solution. Ils ont pu visualiser et dépanner facilement les vues de lignage des données et l'intégrité du service opérationnel grâce à une interface utilisateur de surveillance et gestion Data Factory intuitive, disponible sur le portail Azure. Leur solution peut maintenant être planifiée et gérée de sorte que les données terminées sont produites et transmises de façon fiable aux utilisateurs. Par ailleurs, les données et les dépendances de traitement sont gérées automatiquement, sans intervention humaine.

Grâce à cette expérience d'achat personnalisée, le détaillant en ligne a créé une expérience client plus compétitive et plus attrayante, augmentant ainsi les ventes et la satisfaction globale des clients.

