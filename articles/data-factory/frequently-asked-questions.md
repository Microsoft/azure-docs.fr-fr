---
title: 'Azure Data Factory : Forum aux questions '
description: Réponses aux questions les plus fréquentes concernant Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 02/10/2020
ms.openlocfilehash: 2d1b9f0e25c460b7f26c31c3d2c6ebe51d958017
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96485012"
---
# <a name="azure-data-factory-faq"></a>Forum Aux Questions Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article fournit des réponses aux questions fréquemment posées sur Azure Data Factory.  

## <a name="what-is-azure-data-factory"></a>qu'est-ce qu'Azure Data Factory ? 
Data Factory est un service ERL cloud d’intégration de données entièrement géré qui automatise le déplacement et la transformation des données. À l’instar d’une usine qui utilise des machines visant à transformer des matières premières en produits manufacturés, Azure Data Factory orchestre des services existants qui collectent des données brutes et les transforment en informations prêtes à l’emploi. 

Grâce à Azure Data Factory, vous pouvez créer des flux de travail pilotés par les données pour déplacer des données entre des magasins locaux et cloud. Et vous pouvez traiter et transformer des données avec des flux de données. ADF prend également en charge des moteurs de calcul externe pour des transformations codées à la main en utilisant des services de calcul tels qu’Azure HDInsight, Azure Databricks et le runtime d’intégration SQL Server Integration Services (SSIS). 

Avec Data Factory, vous pouvez exécuter votre traitement de données sur un service cloud Azure ou dans votre propre environnement de calcul auto-hébergé comme SSIS, SQL Server ou Oracle. Après avoir créé un pipeline qui exécute l’action dont vous avez besoin, vous pouvez planifier son exécution périodique (par exemple toutes les heures, tous les jours ou toutes les semaines), planifier une fenêtre de temps ou déclencher le pipeline à partir de l’occurrence d’un événement. Pour plus d’informations, consultez [Présentation d’Azure Data Factory](introduction.md).

### <a name="control-flows-and-scale"></a>Flux de contrôle et mise à l’échelle 
Pour prendre en charge les divers flux d’intégration et modèles dans l’entrepôt de données moderne, Data Factory met en œuvre une modélisation flexible de pipeline de données. Cela implique des paradigmes de programmation de flux de contrôle complets, notamment l’exécution conditionnelle, la création de branches vers des pipelines de données et la possibilité de transmettre explicitement des paramètres dans et entre ces flux. Le flux de contrôle comprend également la transformation des données via la distribution des activités vers des moteurs d’exécution externes et des fonctionnalités de flux de données, notamment le déplacement des données à grande échelle via l’activité de copie.

Data Factory vous donne la liberté de modéliser n’importe quel style de flux nécessaire pour l’intégration des données, qui peut être distribué à la demande ou de façon répétée selon une planification. Voici quelques flux courants permis par ce modèle :   

- Flux de contrôle :
    - Les activités peuvent être chaînées dans une séquence au sein d’un pipeline.
    - Les activités peuvent comprendre des branches dans un pipeline.
    - Paramètres :
        - Les paramètres peuvent être définis au niveau du pipeline et les arguments transmis quand vous appelez le pipeline à la demande ou à partir d’un déclencheur.
        - Les activités peuvent consommer les arguments transmis au pipeline.
    - Transmission d’un état personnalisé :
        - Les sorties de l’activité, notamment l’état, peuvent être consommées par une activité ultérieure du pipeline.
    - Bouclage des conteneurs :
        - L’activité foreach effectue une itération sur une collection spécifiée d’activités dans une boucle. 
- Flux basés sur un déclencheur :
    - Les pipelines peuvent être déclenchés à la demande ou selon un temps horloge.
- Flux delta :
    - Vous pouvez utiliser des paramètres afin de définir la marque de borne haute pour la copie delta lors du déplacement des tables de dimension ou de référence à partir d’un magasin relationnel, localement ou dans le cloud, pour charger les données dans le lac. 

Pour plus d’informations, consultez [Didacticiel : Flux de contrôle](tutorial-control-flow.md).

### <a name="data-transformed-at-scale-with-code-free-pipelines"></a>Données transformées à grande échelle avec des pipelines sans code
La nouvelle expérience des outils basés sur un navigateur permet la création et le déploiement de pipelines sans code avec une expérience web interactive et moderne.

Pour les développeurs de données visuelles et les ingénieurs de données, l’interface utilisateur web de Data Factory constitue l’environnement de conception sans code à utiliser pour créer les pipelines. Elle est entièrement intégrée à l’environnement Git de Visual Studio Online et fournit l’intégration du développement itératif et des fonctionnalités CI/CD avec des options de débogage.

### <a name="rich-cross-platform-sdks-for-advanced-users"></a>Kits SDK multiplateformes complets pour les utilisateurs avancés
Data Factory V2 fournit un ensemble enrichi de SDK que vous pouvez utiliser pour créer, gérer et superviser des pipelines à l’aide de votre IDE favori, notamment :
* Kit de développement logiciel (SDK) Python
* Interface de ligne de commande PowerShell
* Kit de développement logiciel (SDK) C#

Les utilisateurs peuvent également utiliser les API REST documentées pour interagir avec Data Factory V2.

### <a name="iterative-development-and-debugging-by-using-visual-tools"></a>Développement et débogage itératifs avec des outils visuels
Les outils visuels d’Azure Data Factory autorisent le développement et le débogage itératifs. Vous pouvez créer vos pipelines et effectuer des séries de tests avec la fonctionnalité **Déboguer** dans le canevas du pipeline sans écrire une seule ligne de code. Vous pouvez voire les résultats de vos séries de tests dans la fenêtre **Sortie** de votre canevas de pipeline. Après la réussite de votre série de tests, vous pouvez ajouter d’autres activités à votre pipeline et continuer le débogage de façon itérative. Vous pouvez aussi annuler votre série de tests alors qu’elle est en cours. 

Vous n’êtes pas obligé de publier les modifications que vous avez apportées aux données sur le service Data Factory avant de sélectionner **Déboguer**. Ceci est pratique dans les scénarios où vous voulez être sûr que les nouveaux ajouts ou modifications fonctionnent comme prévu avant de mettre à jour les flux de travail de votre fabrique de données dans des environnements de développement, de test ou de production. 

### <a name="ability-to-deploy-ssis-packages-to-azure"></a>Capacité à déployer des packages SSIS sur Azure 
Si vous voulez déplacer vos charges de travail SSIS, vous pouvez créer une fabrique de données et provisionner un runtime d’intégration Azure-SSIS. Un runtime d’intégration Azure-SSIS est un cluster entièrement managé de machines virtuelles Azure (nœuds) dédiées à l’exécution de vos packages SSIS dans le cloud. Pour obtenir des instructions pas à pas, consultez le didacticiel [Déployer des packages SSIS vers Azure](./tutorial-deploy-ssis-packages-azure.md). 
 
### <a name="sdks"></a>Kits SDK
Si vous êtes un utilisateur avancé et que vous recherchez une interface de programmation, Data Factory offre un ensemble étendu de SDK que vous pouvez utiliser pour créer, gérer ou superviser les pipelines avec votre IDE favori. La prise en charge des langages inclut .NET, PowerShell, Python et REST.

### <a name="monitoring"></a>Surveillance
Vous pouvez surveiller vos fabriques de données via PowerShell, un SDK ou les outils de surveillance visuels dans l’interface utilisateur du navigateur. Vous pouvez superviser et gérer des flux personnalisés à la demande, basés sur des déclencheurs et pilotés par le temps, de façon efficiente et efficace. Annulez des tâches existantes, visualisez les échecs en un coup d’œil, explorez les détails des messages d’erreur et déboguez les problèmes dans un seul et même volet, sans avoir à changer de contexte ou à effectuer des allers-retours entre les écrans. 

### <a name="new-features-for-ssis-in-data-factory"></a>Nouvelles fonctionnalités pour SSIS dans Data Factory
Depuis la préversion publique initiale de 2017, les fonctionnalités suivantes pour SSIS ont été ajoutées à Data Factory :

-    Prise en charge de trois configurations/variantes supplémentaires d’Azure SQL Database pour héberger la base de données SSIS (SSISDB) des projets/packages :
-    SQL Database avec points de terminaison de service de réseau virtuel
-    Instance managée SQL
-    Pool élastique
-    Prise en charge d’un réseau virtuel Azure Resource Manager sur un réseau virtuel classique qui sera dépréciée dans le futur : ceci vous permet d’injecter/joindre votre runtime d’intégration Azure-SSIS à un réseau virtuel configuré pour SQL Database avec des points de terminaison de service de réseau virtuel/MI/accès aux données locales. Pour plus d’informations, consultez également [Joindre un runtime d’intégration Azure-SSIS à un réseau virtuel](join-azure-ssis-integration-runtime-virtual-network.md).
-    Prise en charge de l’authentification Azure Active Directory (AAD) et de l’authentification SQL pour se connecter à la SSISDB : ceci vous permet d’utiliser l’authentification Azure AD avec votre identité managée Data Factory pour des ressources Azure
-    Prise en charge de votre licence SQL Server existante pour réaliser des économies substantielles avec l’option Azure Hybrid Benefit
-    Prise en charge de l’édition Entreprise du runtime d’intégration Azure-SSIS, qui vous permet d’utiliser des fonctionnalités avancées/Premium, un programme d’installation personnalisé pour installer des composants/extensions supplémentaires, et un écosystème de partenaires. Pour plus d’informations, consultez également [Enterprise Edition, Custom Setup, and 3rd Party Extensibility for SSIS in ADF](https://blogs.msdn.microsoft.com/ssis/2018/04/27/enterprise-edition-custom-setup-and-3rd-party-extensibility-for-ssis-in-adf/). 
-    Une intégration plus étroite de SSIS dans Data Factory, qui vous permet d’appeler/déclencher des activités étendues d’exécution de package SSIS dans des pipelines Data Factory et de les planifier via SSMS. Pour plus d’informations, consultez [Modernize and extend your ETL/ELT workflows with SSIS activities in ADF pipelines](https://blogs.msdn.microsoft.com/ssis/2018/05/23/modernize-and-extend-your-etlelt-workflows-with-ssis-activities-in-adf-pipelines/).


## <a name="what-is-the-integration-runtime"></a>Qu’est-ce que le runtime d’intégration ?
Le runtime d’intégration est l’infrastructure de calcul utilisée par Azure Data Factory pour fournir les fonctionnalités d’intégration de données suivantes entre différents environnements réseau :

- **Déplacement des données** : pour le mouvement des données, le runtime d’intégration déplace les données entre les magasins de données source et de destination, tout en fournissant une prise en charge des connecteurs intégrés, de la conversion de format, du mappage de colonnes et des transferts de données performants et scalables.
- **Répartir les activités** : pour la transformation, le runtime d’intégration offre la possibilité d’exécuter des packages SSIS en mode natif.
- **Exécuter des packages SSIS** : le runtime d’intégration exécute en mode natif les packages SSIS dans un environnement de calcul Azure managé. Le runtime d’intégration prend également en charge la répartition et la supervision des activités de transformation exécutées sur un large éventail de services de calcul, comme Azure HDInsight, Azure Machine Learning, SQL Database et SQL Server.

Vous pouvez déployer une ou plusieurs instances du runtime d’intégration en fonction de vos besoins pour déplacer ou transformer des données. Le runtime d’intégration peut s’exécuter sur un réseau public Azure ou un réseau privé (localement, réseau virtuel Azure ou cloud privé virtuel Amazon Web Services). 

Pour plus d’informations, consultez [Runtime d’intégration dans Azure Data Factory](concepts-integration-runtime.md).

## <a name="what-is-the-limit-on-the-number-of-integration-runtimes"></a>Quelle est la limite au nombre de runtimes d’intégration ?
Il n’existe aucune limite quant au nombre d’instances de runtime d’intégration autorisées dans une fabrique de données. Il existe toutefois une limite relative au nombre de cœurs de machines virtuelles que le runtime d’intégration peut utiliser par abonnement pour l’exécution de package SSIS. Pour plus d’informations, consultez la section [Limites de Data Factory](../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits).

## <a name="what-are-the-top-level-concepts-of-azure-data-factory"></a>Quels sont les concepts principaux d’Azure Data Factory ?
Un abonnement Azure peut contenir une ou plusieurs instances Azure Data Factory (ou fabriques de données). Azure Data Factory s’articule autour de quatre composants clés. Ils fonctionnent ensemble en tant que plateforme sur laquelle composer des flux de travail pilotés par les données et constitués d’étapes de déplacement et de transformation de données.

### <a name="pipelines"></a>Pipelines
Une fabrique de données peut avoir un ou plusieurs pipelines. Un pipeline est un regroupement logique d’activités nécessaires pour effectuer une unité de travail. Ensemble, les activités d’un pipeline effectuent une tâche. Par exemple, un pipeline peut contenir un groupe d’activités qui ingèrent des données à partir d’un objet blob Azure, puis exécute une requête Hive sur un cluster HDInsight pour partitionner les données. L’avantage est que vous pouvez utiliser un pipeline pour gérer les activités en tant qu’ensemble au lieu de devoir gérer chaque activité individuellement. Vous pouvez chaîner les activités dans un pipeline pour qu’elles fonctionnent de manière séquentielle, ou vous pouvez les exécuter indépendamment, en parallèle.

### <a name="data-flows"></a>Flux de données
Les flux de données sont des objets que vous créez visuellement dans Data Factory et qui transforment les données à l’échelle sur des services Spark de back-end. Vous n’avez pas besoin de comprendre la programmation ni les éléments internes de Spark. Concevez simplement votre intention de transformation des données à l’aide de graphiques (mappage) ou de feuilles de calcul (wrangling).

### <a name="activities"></a>Activités
Les activités représentent une étape de traitement dans un pipeline. Par exemple, vous pouvez utiliser une activité Copy pour copier des données d’un magasin de données vers un autre. De même, vous pouvez utiliser une activité Hive qui exécute une requête Hive sur un cluster Azure HDInsight afin de convertir ou d’analyser vos données. Data Factory prend en charge trois types d’activités : les activités de déplacement des données, les activités de transformation des données et les activités de contrôle.

### <a name="datasets"></a>Groupes de données
Les jeux de données représentent les structures des données dans les magasins. Ils pointent vers ou référencent simplement en tant qu’entrées ou sorties les données que vous voulez utiliser dans vos activités. 

### <a name="linked-services"></a>Services liés
Les services liés ressemblent à des chaînes de connexion. Ils définissent les informations de connexion nécessaires à Data Factory pour se connecter à des ressources externes. Voyez les choses de la façon suivante : un service lié définit la connexion à la source de données et un jeu de données représente la structure des données. Par exemple, un service lié Stockage Azure spécifie la chaîne de connexion pour se connecter au compte de stockage Azure. Et un jeu de données blob Azure spécifie le conteneur d’objets blob et le dossier qui contient les données.

Les services liés ont deux objectifs dans Data Factory :

- Représenter un *magasin de données* qui inclut, mais sans s’y limiter, une instance SQL Server, une instance de base de données Oracle, un partage de fichiers ou un compte de stockage d’objets blob Azure. Pour obtenir la liste des magasins de données pris en charge, consultez [Activité de copie dans Azure Data Factory](copy-activity-overview.md).
- Pour représenter une *ressource de calcul* qui peut héberger l’exécution d’une activité. Par exemple, l’activité HDInsight Hive s’exécute sur un cluster HDInsight Hadoop. Pour obtenir la liste des activités de transformation et des environnements de calcul pris en charge, consultez [Transformer des données dans Azure Data Factory](transform-data.md).

### <a name="triggers"></a>Déclencheurs
Les déclencheurs représentent des unités de traitement qui déterminent le moment où une exécution de pipeline est lancée. Il existe différents types de déclencheurs pour différents types d’événements. 

### <a name="pipeline-runs"></a>Exécutions de pipeline
Une exécution du pipeline est une instance d’exécution d’un pipeline. Généralement, vous instanciez une exécution de pipeline en transmettant des arguments aux paramètres définis dans le pipeline. Vous pouvez transmettre les arguments manuellement ou dans la définition du déclencheur.

### <a name="parameters"></a>Paramètres
Les paramètres sont des paires clé-valeur dans une configuration en lecture seule. Vous définissez des paramètres dans un pipeline, et vous transmettez les arguments pour les paramètres définis lors de l’exécution à partir d’un contexte d’exécution. Le contexte d’exécution est créé par un déclencheur ou à partir d’un pipeline que vous exécutez manuellement. Les activités contenues dans le pipeline utilisent les valeurs des paramètres.

Un jeu de données est un paramètre fortement typé et une entité que vous pouvez réutiliser ou référencer. Une activité peut référencer des jeux de données et utiliser les propriétés définies dans la définition du jeu de données.

Un service lié est également un paramètre fortement typé qui contient des informations de connexion à un magasin de données ou à un environnement de calcul. Il s’agit également d’une entité que vous pouvez réutiliser ou référencer.

### <a name="control-flows"></a>Flux de contrôle
Les flux de contrôle orchestrent des activités de pipeline telles que le chaînage des activités en une séquence, la création de branche, des paramètres que vous définissez au niveau du pipeline, et des arguments que vous transmettez lors de l’appel du pipeline à la demande ou à partir d’un déclencheur. Les flux de contrôle incluent également la transmission d’états personnalisés et le bouclage des conteneurs (autrement dit, des itérateurs foreach).


Pour plus d’informations sur les concepts de Data Factory, consultez les articles suivants :

- [Jeux de données et services liés](concepts-datasets-linked-services.md)
- [Pipelines et activités](concepts-pipelines-activities.md)
- [Runtime d’intégration](concepts-integration-runtime.md)

## <a name="what-is-the-pricing-model-for-data-factory"></a>Quel est le modèle de tarification de Data Factory ?
Pour plus d’informations sur la tarification d’Azure Data Factory, consultez [Détails sur la tarification de Data Factory](https://azure.microsoft.com/pricing/details/data-factory/).

## <a name="how-can-i-stay-up-to-date-with-information-about-data-factory"></a>Comment se tenir informé sur Data Factory ?
Pour obtenir les informations les plus récentes sur Azure Data Factory, consultez les sites suivants :

- [Blog](https://azure.microsoft.com/blog/tag/azure-data-factory/)
- [Page d’accueil de la documentation](./index.yml)
- [Page d’accueil du produit](https://azure.microsoft.com/services/data-factory/)

## <a name="technical-deep-dive"></a>Présentation technique approfondie 

### <a name="how-can-i-schedule-a-pipeline"></a>Comment planifier un pipeline ? 
Vous pouvez utiliser le déclencheur Planificateur ou Fenêtre de temps pour planifier un pipeline. Ce déclencheur utilise un planning horaire, qui peut planifier les pipelines périodiquement ou à l’aide de modèles récurrents basés sur le calendrier (par exemple, le lundi à 18h00 et le jeudi à 21h00). Pour plus d’informations, consultez l’article [Exécution de pipelines et déclencheurs](concepts-pipeline-execution-triggers.md).

### <a name="can-i-pass-parameters-to-a-pipeline-run"></a>Puis-je passer des paramètres à une exécution de pipeline ?
Oui, les paramètres sont des concepts importants dans Data Factory. Vous pouvez les définir au niveau du pipeline et transmettre des arguments lors de l’exécution du pipeline à la demande ou à l’aide d’un déclencheur.  

### <a name="can-i-define-default-values-for-the-pipeline-parameters"></a>Puis-je définir des valeurs par défaut pour les paramètres des pipelines ? 
Oui. Vous pouvez définir des valeurs par défaut pour les paramètres des pipelines. 

### <a name="can-an-activity-in-a-pipeline-consume-arguments-that-are-passed-to-a-pipeline-run"></a>Une activité dans un pipeline peut-elle consommer des arguments transmis à une exécution de pipeline ? 
Oui. Chaque activité du pipeline peut consommer la valeur de paramètre transmise au pipeline et exécutée avec la construction `@parameter`. 

### <a name="can-an-activity-output-property-be-consumed-in-another-activity"></a>Une propriété de sortie d’activité peut-elle être consommée dans une autre activité ? 
Oui. Une sortie d’activité peut être consommée dans une activité suivante avec la construction `@activity`.
 
### <a name="how-do-i-gracefully-handle-null-values-in-an-activity-output"></a>Comment gérer correctement les valeurs Null dans une sortie d’activité ? 
Vous pouvez utiliser la construction `@coalesce` dans les expressions pour gérer correctement les valeurs Null. 

## <a name="mapping-data-flows"></a>Mappage des flux de données

### <a name="i-need-help-troubleshooting-my-data-flow-logic-what-info-do-i-need-to-provide-to-get-help"></a>J’ai besoin d’aide pour dépanner ma logique de flux de données. Quelles informations dois-je fournir pour obtenir une aide ?

Quand Microsoft fournit une aide ou une assistance au dépannage pour des flux de données, veuillez indiquer le script DataFlow. Il s’agit du script code-behind de votre graphique de Data Flow. À partir de l’interface utilisateur ADF, ouvrez votre flux de données, puis cliquez sur le bouton « Script » dans le coin supérieur droit. Copiez et collez ce script ou enregistrez-le dans un fichier texte.

### <a name="how-do-i-access-data-by-using-the-other-90-dataset-types-in-data-factory"></a>Comment accéder à des données à l’aide des 90 autres types de jeux de données dans Data Factory ?

La fonctionnalité de flux de données de mappage autorise actuellement Azure SQL Database, Azure Synapse Analytics, les fichiers texte délimités à partir de Stockage Blob Azure ou Azure Data Lake Storage Gen2, et les fichiers Parquet à partir du stockage d’objets blob ou Data Lake Storage Gen2 en mode natif pour la source et le récepteur. 

Utilisez l’activité de copie pour indexer les données des autres connecteurs, puis exécutez une activité de flux de données pour transformer les données une fois qu’elles ont été indexées. Par exemple, votre pipeline copiera tout d’abord dans le stockage d’objets blob, puis une activité de flux de données utilisera un jeu de données dans la source pour transformer ces données.

### <a name="is-the-self-hosted-integration-runtime-available-for-data-flows"></a>Le runtime d’intégration auto-hébergé est-il disponible pour les flux de données ?

Le runtime d’intégration auto-hébergé est une construction de pipeline ADF que vous pouvez utiliser avec l’activité de copie pour acquérir ou déplacer des données vers et depuis des sources de données et des récepteurs basés sur des machines virtuelles ou locaux. Effectuez d’abord une indexation des données avec une copie, puis le transfert de données pour la transformation, puis une copie ultérieure si vous devez replacer ces données transformées dans le magasin local.

### <a name="does-the-data-flow-compute-engine-serve-multiple-tenants"></a>Le moteur de calcul du flux de données gère-t-il plusieurs locataires ?
Les clusters ne sont jamais partagés. Nous garantissons l’isolement de chaque travail exécuté dans des exécutions de production. Dans le cas d’un scénario de débogage, une personne obtient un cluster, et tous les débogages lancés par cet utilisateur sont dirigés vers ce cluster.

## <a name="wrangling-data-flows"></a>Flux de wrangling data

### <a name="what-are-the-supported-regions-for-wrangling-data-flow"></a>Quelles sont les régions prises en charge pour le flux de wrangling data ?

Le flux de wrangling data est actuellement pris en charge dans les fabriques de données créées dans les régions suivantes :

* Australie Est
* Centre du Canada
* Inde centrale
* USA Est
* USA Est 2
* Japon Est
* Europe Nord
* Asie Sud-Est
* États-Unis - partie centrale méridionale
* Sud du Royaume-Uni
* Centre-USA Ouest
* Europe Ouest
* USA Ouest
* USA Ouest 2

### <a name="what-are-the-limitations-and-constraints-with-wrangling-data-flow"></a>Quelles sont les limitations et les contraintes imposées par l’utilisation d’un flux de wrangling data ?

Les noms de jeux de données peuvent uniquement contenir des caractères alphanumériques. Les banques de données suivantes sont prises en charge :

* Jeu de données DelimitedText dans le Stockage Blob Azure à l’aide de l’authentification par clé de compte
* Jeu de données DelimitedText dans Azure Data Lake Storage Gen2 utilisant la clé de compte ou l’authentification du principal de service
* Jeu de données DelimitedText dans Azure Data Lake Storage Gen1 utilisant l’authentification du principal de service
* Azure SQL Database et Data Warehouse utilisant l’authentification SQL. Reportez-vous aux types SQL pris en charge. Il n’existe aucune prise en charge de Polybase ou de phase intermédiaire pour l’entrepôt de données.

À ce stade, l’intégration du service Key Vault lié n’est pas prise en charge dans les flux de wrangling data.

### <a name="what-is-the-difference-between-mapping-and-wrangling-data-flows"></a>Quelle est la différence entre le mappage et les flux de wrangling data ?

Le mappage de flux de données permet de transformer des données à grande échelle sans aucun codage. Vous pouvez concevoir une tâche de transformation de données dans le canevas de flux de données en créant une série de transformations. Commencez par un certain nombre de transformations sources, suivies par des étapes de transformation de données. Réalisez votre flux de données avec un récepteur qui servira de destination à vos résultats. Le mappage du flux de données est très utile lors du mappage et de la transformation des données avec des schémas connus et inconnus dans les récepteurs et les sources.

Les flux de wrangling data vous permettent d’effectuer une préparation et une exploration agiles des données à l’aide de l’éditeur web hybride Power Query en ligne à grande échelle via l’exécution de Spark. Si votre organisation fait partie des entreprises de plus en plus nombreuses à adopter la technologie des lacs de données, vous devez parfois simplement explorer un jeu de données ou créer un jeu de données dans le lac. Vous n’effectuez pas de mappage vers une cible connue. Les flux de wrangling data sont utilisés dans des scénarios analytiques moins formels et basés sur des modèles.

### <a name="what-is-the-difference-between-power-platform-dataflows-and-wrangling-data-flows"></a>Quelle est la différence entre les flux de données Power Platform et les flux de wrangling data ?

Les Dataflows Power Platform permettent aux utilisateurs d’importer et de transformer des données provenant d'un large éventail de sources de données dans le Common Data Service et Azure Data Lake afin de créer des applications PowerApps, des rapports Power BI ou des automatisations Flow. Les Dataflows Power Platform utilisent les expériences éprouvées de préparation de données Power Query, similaires à Power BI et Excel. Les Dataflows Power Platform permettent également une réutilisation aisée au sein d'une organisation et gèrent automatiquement l’orchestration (par exemple, actualiser automatiquement les flux de données qui dépendent d’un autre flux de données lorsque l’ancien flux de données est actualisé).

Azure Data Factory (ADF) est un service d’intégration de données géré qui permet aux ingénieurs de données et aux intégrateurs de données citoyens de créer des flux de travail hybrides complexes d’extraction-transformation-chargement (ETL) et d’extraction-chargement-transformation (ELT). Le flux de wrangling data dans ADF offre aux utilisateurs un environnement sans code et serverless, qui simplifie la préparation des données dans le cloud et s’adapte à n’importe quelle taille de données. Aucune gestion d’infrastructure n’est requise. Il utilise la technologie de préparation des données Power Query (également utilisée dans Power Platform flux, Excel, Power BI) pour préparer et mettre en forme les données. Conçus pour gérer l’ensemble des complexités et des défis d’échelle de l’intégration du Big Data, les flux de wrangling data permettent aux utilisateurs de préparer rapidement des données à grande échelle via l’exécution de Spark. Les utilisateurs peuvent créer des pipelines de données résilients dans un environnement visuel accessible avec notre interface basée sur un navigateur et laisser ADF gérer les complexités de l’exécution de Spark. Établissez des calendriers pour vos pipelines et surveillez les exécutions de vos flux de données à partir du portail de supervision ADF. Gérez facilement les contrats SLA de disponibilité des données avec les alertes et la supervision enrichie de la disponibilité dans ADF, ainsi que les fonctionnalités d’intégration et de déploiement continus prédéfinies pour enregistrer et gérer vos flux dans un environnement managé. Établissez des alertes et affichez des plans d’exécution pour valider que votre logique fonctionne comme prévu lorsque vous optimisez vos flux de données.

### <a name="supported-sql-types"></a>Types SQL pris en charge

Le flux de wrangling data prend en charge les types de données suivantes dans SQL. Vous obtiendrez une erreur de validation pour l’utilisation d’un type de données qui n’est pas pris en charge.

* short
* double
* real
* float
* char
* NCHAR
* varchar
* NVARCHAR
* entier
* int
* bit
* boolean
* SMALLINT
* TINYINT
* bigint
* long
* text
* Date
* DATETIME
* datetime2
* smalldatetime
* timestamp
* UNIQUEIDENTIFIER
* Xml

D’autres types de données seront pris en charge à l’avenir.

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des instructions pas à pas pour créer une fabrique de données, consultez les tutoriels suivants :

- [Démarrage rapide : Créer une fabrique de données](quickstart-create-data-factory-dot-net.md)
- [Tutoriel : Copier des données dans le cloud](tutorial-copy-data-dot-net.md)