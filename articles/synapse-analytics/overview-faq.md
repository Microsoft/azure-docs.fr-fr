---
title: Questions fréquentes (FAQ) - Azure Synapse Analytics
description: FAQ sur Azure Synapse Analytics
services: synapse-analytics
author: saveenr
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: overview
ms.date: 10/25/2020
ms.author: saveenr
ms.reviewer: jrasnick
ms.openlocfilehash: 671ca73cfa898be532521599d1211d2a8081eb4b
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97563260"
---
# <a name="azure-synapse-analytics-frequently-asked-questions"></a>Questions fréquentes (FAQ) sur Azure Synapse Analytics

Ce guide recense les questions les plus fréquemment posées sur Azure Synapse Analytics.

## <a name="general"></a>Général

### <a name="q-how-can-i-use-rbac-roles-to-secure-my-workspace"></a>Q : Comment puis-je utiliser des rôles RBAC pour sécuriser mon espace de travail ?

A : Azure Synapse présente un certain nombre de rôles et d’étendues à affecter, qui simplifieront la sécurisation de votre espace de travail.

Rôles RBAC Synapse :
* Administrateur Synapse
* Administrateur Synapse SQL
* Administrateur Synapse Spark
* Contributeur Synapse (préversion)
* Éditeur d’artefact Synapse (préversion)
* Utilisateur d’artefact Synapse (préversion)
* Opérateur de capacité de calcul Synapse (préversion)
* Utilisateur d’informations d’identification Synapse (préversion)

Pour sécuriser votre espace de travail Synapse, attribuez les rôles RBAC à ces étendues RBAC :
* Workspaces
* Spark pools
* Runtimes d’intégration
* Services liés
* Informations d'identification

De plus, avec les pools SQL dédiés, vous disposez des mêmes fonctionnalités de sécurité que celles que vous connaissez et aimez.

### <a name="q-how-do-i-control-cont-dedicated-sql-pools-serverless-sql-pools-and-serverless-spark-pools"></a>Q : Comment contrôler les pools SQL dédiés, les pools SQL serverless et les pools Spark serverless ?

A : Azure Synapse travaille avec l’analyse des coûts et les alertes de coût intégrées qui sont disponibles au niveau de l’abonnement Azure, ce qui constitue une base de départ.

- Pools SQL dédiés : vous bénéficiez d’une visibilité directe sur le coût et la maîtrise du coût, car vous créez et indiquez les tailles des pools SQL dédiés. Vous pouvez mieux contrôler quels utilisateurs peuvent créer ou mettre à l’échelle les pools SQL dédiés avec les rôles RBAC Azure.

- Pools SQL serverless : vous disposez de contrôles de supervision et de gestion des coûts qui vous permettent de limiter les dépenses aux niveaux quotidien, hebdomadaire et mensuel. Pour plus d’informations, consultez [Gestion des coûts du pool SQL serverless](./sql/data-processed.md). 

- Pools Spark serverless : vous pouvez limiter les personnes autorisées à créer des pools Spark avec des rôles RBAC Synapse.  

### <a name="q-will-synapse-workspace-support-folder-organization-of-objects-and-granularity-at-ga"></a>Q : Est-ce que les espaces de travail Synapse vont prendre en charge l’organisation en dossiers des objets et la précision lors de la disponibilité générale ?

A : Les espaces de travail Synapse prennent en charge les dossiers définis par l’utilisateur.

### <a name="q-can-i-link-more-than-one-power-bi-workspace-to-a-single-azure-synapse-workspace"></a>Q : Puis-je lier plusieurs espaces de travail Power BI à un seul espace de travail Azure Synapse ?
    
A : Actuellement, vous ne pouvez lier qu’un seul espace de travail Power BI à un espace de travail Azure synapse. 

### <a name="q-is-synapse-link-to-cosmos-db-ga"></a>Q : Est-ce que Synapse Link pour Cosmos DB est en disponibilité générale ?

A : Synapse Link pour Apache Spark est en disponibilité générale. Synapse Link pour le pool SQL serverless est en préversion publique.

### <a name="q-does-azure-synapse-workspace-support-cicd"></a>Q : L’espace de travail Azure Synapse prend-il en charge CI/CD ? 

A : Oui. Tous les artefacts de pipeline, les notebooks, scripts SQL et définitions de travaux Spark résideront dans Git. Toutes les définitions de pool seront stockées dans Git sous forme de modèles ARM. Les objets de pool SQL dédiés (schémas, tables, vues, etc.) seront gérés avec des projets de base de données, avec prise en charge CI/CD.

## <a name="pipelines"></a>Pipelines

### <a name="q-how-do-i-ensure-i-know-what-credential-is-being-used-to-run-a-pipeline"></a>Q : Comment savoir quelles informations d’identification sont utilisées pour exécuter un pipeline ? 

A : Chaque activité d’un pipeline Synapse est exécutée avec des informations d’identification spécifiées à l’intérieur du service lié.

### <a name="q-are-ssis-irs-supported-in-synapse-integrate"></a>Q : Les instances SSIS IR sont-elles prises en charge dans Synapse Integrate ?

A : Pas pour l'instant. 

### <a name="q-how-do-i-migrate-existing-pipelines-from-azure-data-factory-to-an-azure-synapse-workspace"></a>Q : Comment migrer des pipelines existants d’Azure Data Factory vers un espace de travail Azure Synapse ?

A : À ce stade, vous devez recréer manuellement vos pipelines Azure Data Factory, et les artefacts associés, en exportant le fichier JSON depuis le pipeline d’origine et en l’important dans votre espace de travail Synapse.

## <a name="apache-spark"></a>Apache Spark

### <a name="q-what-is-the-difference-between-apache-spark-for-synapse-and-apache-spark"></a>Q : Quelle différence y a-t-il entre Apache Spark pour Synapse et Apache Spark ?

A : Apache Spark pour Synapse n’est autre qu’Apache Spark doté d’une prise en charge supplémentaire pour les intégrations à d’autres services (AAD, AzureML, etc.), avec des bibliothèques supplémentaires (mssparktuils, Hummingbird) et des configurations de performances prédéfinies.

Toute charge de travail en cours d’exécution sur Apache Spark s’exécutera sur Apache Spark pour Azure Synapse sans modification. 

### <a name="q-what-versions-of-spark-are-available"></a>Q : Quelles versions de Spark sont disponibles ?

A : Azure Synapse Apache Spark prend entièrement en charge Spark 2.4. Pour obtenir la liste complète des principaux composants et des versions actuellement prises en charge, consultez [Prise en charge des versions Apache Spark](./spark/apache-spark-version-support.md).

### <a name="q-is-there-an-equivalent-of-dbutils-in-azure-synapse-spark"></a>Q : Existe-t-il un équivalent de DButils dans Azure Synapse Spark ?

A : Oui, Azure Synapse Apache Spark fournit la bibliothèque **mssparkutils**. Pour obtenir une documentation complète de l’utilitaire, consultez [Présentation des utilitaires Microsoft Spark](./spark/microsoft-spark-utilities.md).

### <a name="q-how-do-i-set-session-parameters-in-apache-spark"></a>Q : Comment définir des paramètres de session dans Apache Spark ?

A : Pour définir des paramètres de session, utilisez %%configure magic disponible. Un redémarrage de session est nécessaire pour que les paramètres soient pris en compte. 

### <a name="q-how-do-i-set-cluster-level-parameters-in-a-serverless-spark-pool"></a>Q : Comment définir des paramètres au niveau du cluster dans un pool Spark serverless ?

A : Pour définir des paramètres au niveau du cluster, vous pouvez fournir un fichier spark.conf destiné au pool Spark. Ce pool respectera ensuite les paramètres passés dans le fichier de configuration. 

### <a name="q-can-i-run-a-multi-user-spark-cluster-in-azure-synapse-analytics"></a>Q : Puis-je exécuter un cluster Spark multiutilisateur dans Azure Synapse Analytics ?
 
A : Azure Synapse fournit des moteurs spécialement conçus pour des cas d’usage spécifiques. Apache Spark pour Synapse est conçu comme un service de travaux, et non comme un modèle de cluster. Il existe deux scénarios dans lesquels un modèle de cluster multiutilisateur est demandé.

**Scénario 1 : Accès de nombreux utilisateurs à un cluster pour traiter des données à des fins décisionnelles.**

Le moyen le plus simple d’accomplir cette tâche consiste à préparer les données avec Spark, puis à tirer parti des fonctionnalités de service de Synapse SQL, afin qu’elles puissent connecter Power BI à ces jeux de données.

**Scénario 2 : Présence de plusieurs développeurs sur un seul cluster pour faire des économies.**
 
Pour développer ce scénario, vous devez donner à chaque développeur un pool Spark serverless configuré pour utiliser un petit nombre de ressources Spark. Comme les pools Spark serverless ne coûtent rien tant qu’ils ne sont pas utilisés activement, vous réduisez les coûts lorsqu’il y a plusieurs développeurs. Du fait que les pools partagent des métadonnées (tables Spark), ils peuvent facilement travailler ensemble.

### <a name="q-how-do-i-include-manage-and-install-libraries"></a>Q : Comment inclure, gérer et installer des bibliothèques ?

A :  Vous pouvez installer des packages externes par le biais d’un fichier requirements.txt lors de la création du pool Spark, à partir de l’espace de travail Synapse ou du portail Azure. Consultez [Gérer des bibliothèques pour Apache Spark dans Azure Synapse Analytics](./spark/apache-spark-azure-portal-add-libraries.md).

## <a name="dedicated-sql-pools"></a>Pools SQL dédiés

### <a name="q-what-are-the-functional-differences-between-dedicated-sql-pools-and-serverless-pools"></a>Q : Quelles sont les différences fonctionnelles entre les pools SQL dédiés et les pools serverless ?

A : Vous trouverez la liste complète des différences dans [Différences entre les fonctionnalités T-SQL dans Synapse SQL](./sql/overview-features.md).

### <a name="q-now-that-azure-synapse-is-ga-how-do-i-move-my-dedicated-sql-pools-that-were-previously-standalone-into-azure-synapse"></a>Q : Azure Synapse étant en disponibilité générale, comment déplacer mes pools SQL dédiés, précédemment autonomes, dans Azure Synapse ? 

A : Il n’y a pas de « déplacement » ni de « migration ». Vous pouvez choisir d’activer les nouvelles fonctionnalités de l’espace de travail sur vos pools existants. Si vous le faites, il n’y aura aucun changement cassant. En revanche, vous aurez la possibilité d’utiliser de nouvelles fonctionnalités, telles que Synapse Studio, Spark et des pools SQL serverless.

### <a name="q-what-is-the-default-deployment-of-dedicated-sql-pools-now"></a>Q : Quel est le déploiement par défaut des pools SQL dédiés maintenant? 

A : Par défaut, tous les nouveaux pools SQL dédiés sont déployés dans un espace de travail. Toutefois, si vous avez en besoin, vous pouvez toujours créer un pool SQL dédié (anciennement SQL DW) dans un facteur de forme autonome. 


### <a name="q-what-are-the-functional-differences-between-dedicated-sql-pools-and-serverless-sql-pools"></a>Q : Quelles sont les différences fonctionnelles entre les pools SQL dédiés et les pools SQL serverless ?

A : Vous trouverez la liste complète des différences dans [Différences entre les fonctionnalités T-SQL dans Synapse SQL](./sql/overview-features.md).

## <a name="next-steps"></a>Étapes suivantes

* [Bien démarrer avec Azure Synapse Analytics](get-started.md)
* [Créer un espace de travail](quickstart-create-workspace.md)
* [Utiliser un pool SQL serverless](quickstart-sql-on-demand.md)
