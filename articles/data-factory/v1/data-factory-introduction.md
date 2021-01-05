---
title: Introduction à Data Factory, un service d’intégration de données
description: Découvrez Azure Data Factory, un service cloud d’intégration de données qui gère et automatise le déplacement et la transformation des données.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.assetid: cec68cb5-ca0d-473b-8ae8-35de949a009e
ms.service: data-factory
ms.workload: data-services
ms.topic: overview
ms.date: 01/22/2018
ms.openlocfilehash: 101de32aea47bb3dd3c8a04434993c8614939c37
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96495563"
---
# <a name="introduction-to-azure-data-factory"></a>Présentation d’Azure Data Factory 
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-introduction.md)
> * [Version 2 (version actuelle)](../introduction.md)

> [!NOTE]
> Cet article s’applique à la version 1 d’Azure Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez l’article [Présentation de Data Factory V2](../introduction.md).


## <a name="what-is-azure-data-factory"></a>qu'est-ce qu'Azure Data Factory ?
Dans le monde du Big Data, comment les entreprises exploitent-elles les données existantes ? Est-il possible d’enrichir les données générées dans le cloud en utilisant des données de référence provenant de sources locales ou d’autres sources disparates ? 

Par exemple, une entreprise qui produit des jeux collecte des journaux d’activité générés par les jeux dans le cloud. Elle souhaite analyser ces journaux d’activité pour obtenir des informations détaillées sur les préférences des clients, les données démographiques, le comportement d’utilisation, et d’autres. L’entreprise souhaite également identifier les opportunités de vente incitative et de ventes croisées, développer des nouvelles fonctionnalités attrayantes, développer son activité et fournir une meilleure expérience à ses clients. 

Pour analyser ces journaux d’activité, l’entreprise doit utiliser des données de référence comme des informations sur le client, des informations sur les jeux et des informations sur la campagne marketing qui sont contenues dans un magasin de données local. L’entreprise souhaite donc ingérer des données de journal du magasin de données cloud et des données de référence du magasin de données local. 

Ensuite, elle souhaite traiter les données à l’aide de Hadoop dans le cloud (Azure HDInsight). Elle souhaite publier les données de résultat dans un entrepôt de données cloud comme Azure Synapse Analytics ou un magasin de données local tel que SQL Server. L’entreprise souhaite exécuter ce flux de travail une fois par semaine. 

L’entreprise a besoin d’une plateforme où elle peut créer un flux de travail pouvant ingérer des données venant de magasins de données locales ou sur le cloud. L’entreprise doit également être en mesure de transformer ou de traiter des données à l’aide de services de calcul existants tels que Hadoop, et publier les résultats dans un magasin de données local ou sur le cloud pour les applications BI à consommer. 

![Présentation de Data Factory](media/data-factory-introduction/what-is-azure-data-factory.png) 

Azure Data Factory est la plateforme pour ce genre de scénarios. Il s’agit d’un *service d’intégration de données basé sur le cloud qui vous permet de créer des flux de travail orientés données dans le cloud pour orchestrer et automatiser le déplacement des données et la transformation des données*. À l’aide de Azure Data Factory, vous pouvez effectuer les tâches suivantes : 

- Créer et planifier des flux de travail pilotés par les données (appelés pipelines) qui peuvent ingérer des données provenant de différents magasins de données.

- Traiter ou transformer les données à l’aide de services de calcul tels que Azure HDInsight Hadoop, Spark, Azure Data Lake Analytics et Azure Machine Learning.

-  Publier des données de sortie sur des magasins de données comme Azure Synapse Analytics pour que des applications décisionnelles (BI) puissent les utiliser.  

Il s’agit plus d’une plateforme d’extraction et de chargement (EL) et de transformation et chargement (TL) qu’une plateforme d’extraction, de transformation-et de chargement (ETL) traditionnelle. Les transformations traitent des données à l’aide de services de calcul plutôt qu’en ajoutant des colonnes dérivées, en comptant le nombre de lignes, en triant des données et ainsi de suite. 

Dans Azure Data Factory, les données utilisées et générées par les flux de travail sont actuellement des *données à tranches temporelles* (toutes les heures, tous les jours, toutes les semaines, et ainsi de suite). Par exemple, un pipeline peut lire des données d’entrée, traiter des données et générer des données de sortie une fois par jour. Vous ne pouvez également exécuter un flux de travail qu’une seule fois.  
  

## <a name="how-does-it-work"></a>Comment cela fonctionne-t-il ? 
Les pipelines (flux de travail orientés données) dans Azure Data Factory effectuent généralement les trois étapes suivantes :

![Trois étapes d’Azure Data Factory](media/data-factory-introduction/three-information-production-stages.png)

### <a name="connect-and-collect"></a>Se connecter et collecter
Les entreprises disposent de données de divers types situées dans des sources diverses. La première étape dans la création d’un système de production d’informations est la connexion à toutes les sources de données et de traitement requises. Ces sources incluent des services SaaS, les partages de fichiers, FTP et les services web. Déplacez ensuite les données souhaitées vers un emplacement centralisé pour un traitement ultérieur.

Sans Data Factory, les entreprises doivent concevoir des composants personnalisés chargés du déplacement des données ou écrire des services personnalisés pour intégrer ces sources de données et leur traitement. Il est coûteux et difficile d’intégrer et de gérer ces systèmes. Ces systèmes sont également souvent dépourvus des fonctionnalités de surveillance, d’alertes et de contrôles de niveau entreprise qu’un service entièrement géré peut offrir.

Avec Data Factory, vous pouvez utiliser la fonction de copie dans un pipeline de données. L’intérêt ? Déplacer les données des magasins de données locaux et dans le cloud dans un magasin de données centralisé dans le cloud, pour les besoins d’analyse ultérieurs. 

Par exemple, vous pouvez collecter des données dans Azure Data Lake Store, puis les transformer ultérieurement à l’aide d’un service de calcul Azure Data Lake Analytics. Ou, collectez des données dans un stockage Blob Azure, puis transformez-les ultérieurement à l’aide d’un cluster Azure HDInsight Hadoop.

### <a name="transform-and-enrich"></a>Transformer et enrichir
Une fois que les données sont présentes dans un magasin de données centralisé dans le cloud, traitez ou transférez-les à l’aide de services de calcul tels que HDInsight Hadoop, Spark, Data Lake Analytics et Machine Learning. Vous souhaitez générer de manière fiable des données transformées selon une planification facile à gérer et contrôlée afin de fournir aux environnements de production des données approuvées. 

### <a name="publish"></a>Publish 
Remettez les données transformées depuis le cloud vers des sources locales telles que SQL Server. Vous pouvez également les conserver dans vos sources de stockage sur le cloud pour une consommation par BI, des outils analytiques et d’autres applications.

## <a name="key-components"></a>Composants clés
Un abonnement Azure peut contenir une ou plusieurs instances Azure Data Factory (ou fabriques de données). Azure Data Factory s’articule autour de quatre composants clés. Ces composants fonctionnent ensemble et vous dotent de la plateforme sur laquelle composer des flux de travail orientés données constitués d’étapes de déplacement et de transformation des données. 

### <a name="pipeline"></a>Pipeline
Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline est un groupe d’activités. Ensemble, les activités d’un pipeline effectuent une tâche. 

Par exemple, un pipeline peut contenir un groupe d’activités qui ingère des données à partir d’un objet Blob Azure, puis exécute une requête Hive sur un cluster HDInsight pour partitionner les données. L’avantage de cette opération, c’est que le pipeline vous permet de gérer les activités en tant que groupe et non pas individuellement. Par exemple, vous pouvez déployer et planifier le pipeline, au lieu de planifier des activités séparément. 

### <a name="activity"></a>Activité
Un pipeline peut contenir une ou plusieurs activités. Les activités définissent les actions à effectuer sur les données. Par exemple, vous pouvez utiliser une activité de copie pour copier des données d’une banque de données vers une autre. De même, vous pouvez utiliser une activité Hive. Une activité Hive exécute une requête Hive sur un cluster Azure HDInsight afin de convertir ou d’analyser vos données. Data Factory prend en charge deux types d’activités : les activités de déplacement des données et les activités de transformation des données.

### <a name="data-movement-activities"></a>Activités de déplacement des données
L’activité de copie dans Data Factory permet de copier les données d’un magasin de données source vers un magasin de données récepteur. Les données de n’importe quelle source peuvent être écrites dans n’importe quel récepteur. Sélectionnez un magasin de données pour découvrir comment copier des données depuis/vers cette banque. Data Factory prend en charge les magasins de données suivants :

[!INCLUDE [data-factory-supported-data-stores](../../../includes/data-factory-supported-data-stores.md)]

Pour plus d’informations, consultez [Déplacer des données à l’aide de l’activité de copie](data-factory-data-movement-activities.md).

### <a name="data-transformation-activities"></a>Activités de transformation des données
[!INCLUDE [data-factory-transformation-activities](../../../includes/data-factory-transformation-activities.md)]

Pour plus d’informations, consultez [Déplacer des données à l’aide de l’activité de copie](data-factory-data-transformation-activities.md).

### <a name="custom-net-activities"></a>Activités .NET personnalisées
Créez une activité .NET personnalisée si vous devez déplacer des données vers ou à partir d’un magasin de données qui n’est pas prise en charge par l’activité de copie, ou transformer des données à l’aide de votre propre logique. Pour plus d’informations sur la façon de créer et d’utiliser une activité personnalisée, consultez [Utilisation des activités personnalisées dans un pipeline Azure Data Factory](data-factory-use-custom-activities.md).

### <a name="datasets"></a>Groupes de données
Une activité accepte ou non des jeux de données en tant qu’entrées et produit un ou plusieurs jeux de données en tant que sorties. Les jeux de données forment des structures de données au sein des magasins de données. Ces structures pointent ou référencent les données que vous souhaitez utiliser dans vos activités (telles que des entrées ou des sorties). 

Par exemple, un jeu de données d’objets Blob Azure spécifie le conteneur et le dossier du stockage Blob Azure à partir duquel le pipeline doit lire les données. Ou un jeu de données Azure SQL Table spécifie la table dans laquelle les données de sortie sont écrites par l’activité. 

### <a name="linked-services"></a>Services liés
Les services liés ressemblent à des chaînes de connexion. Ils définissent les informations de connexion nécessaires à Data Factory pour se connecter à des ressources externes. Voyez les choses de la façon suivante : un service lié définit la connexion à la source de données et un jeu de données représente la structure des données. 

Par exemple, un service lié Stockage Azure spécifie une chaîne de connexion avec laquelle se connecter au compte de stockage Azure. Un jeu de données blob Azure spécifie le conteneur d’objets blob et le dossier qui contient les données.   

Data Factory fait appel aux services liés pour deux raisons :

* Pour représenter un *magasin de données* qui inclut, sans s’y limiter, une base de données SQL Server, une base de données Oracle, un partage de fichiers ou un compte de stockage d’objets blob Azure. Pour obtenir la liste des banques de données prises en charge, consultez la section [Activités de déplacement des données](#data-movement-activities) .

* Pour représenter une *ressource de calcul* qui peut héberger l’exécution d’une activité. Par exemple, l’activité HDInsightHive s’exécute sur un cluster HDInsight Hadoop. Pour obtenir la liste des environnements de calcul pris en charge, consultez la section [Activités de transformation des données](#data-transformation-activities).

### <a name="relationship-between-data-factory-entities"></a>Relation entre des entités Data Factory

![Diagramme : Data Factory, un service cloud d’intégration de données - Concepts clés](./media/data-factory-introduction/data-integration-service-key-concepts.png)

## <a name="supported-regions"></a>Régions prises en charge
Actuellement, vous pouvez créer des fabriques de données dans les régions USA Ouest, USA Est et Europe Nord. Une fabrique de données peut toutefois accéder à des magasins de données et à des services de calcul situés dans d’autres régions Azure pour déplacer des données entre des magasins de données ou pour traiter des données à l’aide des services de calcul.

Azure Data Factory ne permet pas en soi de stocker des données. Il vous permet de créer des flux de travail pilotés par des données pour orchestrer le déplacement des données entre [les magasins de données pris en charge](#data-movement-activities). Il vous permet également de traiter des données à l’aide de [services de calcul](#data-transformation-activities) dans d’autres régions ou dans un environnement local. Il vous permet également de [surveiller et gérer des flux de travail](data-factory-monitor-manage-pipelines.md) au moyen de programmes et à l’aide des mécanismes de l’interface utilisateur.

Data Factory est disponible uniquement dans les régions Europe Nord, USA Est et USA Ouest. Toutefois, le service qui gère le déplacement des données dans Data Factory est disponible [globalement](data-factory-data-movement-activities.md#global) dans plusieurs régions. Si un magasin de données se trouve derrière un pare-feu, le déplacement des données est assuré au moyen d’une [passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) installée dans votre environnement local.

Supposons que vos environnements de calcul (cluster Azure HDInsight et Azure Machine Learning, par exemple) sont situés hors de la région Europe Ouest. Vous pouvez créer et utiliser une instance de Azure Data Factory dans la région Europe Nord. Vous pouvez ensuite l’utiliser pour planifier des travaux sur vos environnements Compute dans la région Europe Ouest. Quelques millisecondes suffisent à Data Factory pour déclencher la tâche dans votre environnement de calcul, mais l’heure d’exécution du travail dans votre environnement informatique ne change pas.

## <a name="get-started-with-creating-a-pipeline"></a>Prise en main de la création d’un pipeline
Vous pouvez utiliser un des outils ou API suivants pour créer des pipelines de données dans Azure Data Factory : 

- Visual Studio
- PowerShell
- API .NET
- API REST
- Modèle Azure Resource Manager

Pour découvrir comment créer des fabriques de données avec des pipelines de données, suivez les instructions pas à pas des didacticiels suivants :

| Didacticiel | Description |
| --- | --- |
| [Déplacer des données entre deux magasins de données cloud](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) |Créez une fabrique de données avec un pipeline qui déplace des données du stockage blob vers SQL Database. |
| [Transformer des données à l’aide du cluster Hadoop](data-factory-build-your-first-pipeline.md) |Générez votre première fabrique de données Azure avec un pipeline de données qui traite les données en exécutant le script Hive sur un cluster Azure HDInsight (Hadoop). |
| [Déplacer des données entre un magasin de données locale et un magasin de données cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md) |Créez une fabrique de données avec un pipeline qui déplace des données d’une base de données SQL Server vers un objet blob Azure. Dans le cadre de la procédure pas à pas, vous installez et configurez la passerelle de gestion des données sur votre ordinateur. |
