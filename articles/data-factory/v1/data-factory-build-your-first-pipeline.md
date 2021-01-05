---
title: 'Tutoriel Data Factory : Premier pipeline de données '
description: Ce didacticiel Azure Data Factory vous montre comment créer et planifier une fabrique de données qui traite les données à l’aide du script Hive sur un cluster Hadoop.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.openlocfilehash: a79872c11d76a74932fe232fa3f7818edc58ffcc
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96496209"
---
# <a name="tutorial-build-your-first-pipeline-to-transform-data-using-hadoop-cluster"></a>Tutoriel : Générer votre premier pipeline pour transformer les données à l’aide du cluster Hadoop
> [!div class="op_single_selector"]
> * [Vue d’ensemble et composants requis](data-factory-build-your-first-pipeline.md)
> * [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
> * [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
> * [modèle Azure Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
> * [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)


> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Démarrage rapide : Créer une fabrique de données à l’aide d’Azure Data Factory](../quickstart-create-data-factory-dot-net.md).

Dans ce didacticiel, vous allez générer votre première fabrique de données Azure avec un pipeline de données. Le pipeline transforme les données d’entrée en exécutant un script Hive sur un cluster Azure HDInsight (Hadoop) pour produire des données de sortie.  

Cet article fournit une vue d’ensemble et la configuration requise pour le didacticiel. Une fois les prérequis satisfaits, vous pouvez effectuer le tutoriel à l’aide de l’un des outils/SDK suivants : Visual Studio, PowerShell, modèle Resource Manager, API REST. Sélectionnez l’une des options de la liste déroulante au début ou les liens à la fin de cet article pour suivre ce didacticiel en utilisant l’une ou l’autre des possibilités.    

## <a name="tutorial-overview"></a>Vue d’ensemble du didacticiel
Dans ce tutoriel, vous effectuerez les étapes suivantes :

1. Création d'une **fabrique de données**. Une fabrique de données peut contenir un ou plusieurs pipelines de données qui déplacent et transforment des données.

    Dans ce didacticiel, vous créez un pipeline dans la fabrique de données.
2. Création d'un **pipeline**. Un pipeline peut contenir une ou plusieurs activités (exemples : activité de copie, activité HDInsight Hive). Cet exemple utilise l’activité Hive d’HDInsight, qui exécute un script Hive sur un cluster HDInsight Hadoop. Le script crée d’abord une table qui fait référence aux données de journaux web bruts stockées dans le stockage d’objets blob Azure, puis partitionne les données brutes par année et par mois.

    Dans ce didacticiel, le pipeline utilise l’activité Hive pour transformer les données en exécutant une requête Hive sur un cluster Azure HDInsight Hadoop.
3. Créer des **services liés**. Vous créez un service lié pour lier un magasin de données ou un service de calcul à la fabrique de données. Un magasin de données comme Azure Storage conserve les données d’entrée/de sortie d’activités dans le pipeline. Un service de calcul comme un cluster HDInsight Hadoop traite/transforme des données.

    Dans ce tutoriel, vous allez créer deux services liés : **Stockage Azure** et **HDInsight Azure**. Le service lié du stockage Azure relie un compte de stockage Azure qui contient les données d’entrée/de sortie de la fabrique de données. Le service lié Azure HDInsight relie un cluster Azure HDInsight qui est utilisé pour transformer les données de la fabrique de données.
3. Créer des **jeux de données** d’entrée et de sortie. Un jeu de données d’entrée représente l’entrée d’une activité dans le pipeline, tandis qu’un jeu de données de sortie représente la sortie de l’activité.

    Dans ce didacticiel, les jeux de données d’entrée et de sortie indiquent des emplacements de données d’entrée et de sortie dans le stockage Blob Azure. Le service lié du stockage Azure indique quel compte de stockage Azure est utilisé. Un jeu de données d’entrée spécifie l’emplacement des fichiers d’entrée, tandis qu’un jeu de données de sortie indique l’emplacement des fichiers de sortie.


Pour obtenir une présentation détaillée d’Azure Data Factory, consultez l’article [Présentation d’Azure Data Factory](data-factory-introduction.md).

Voici la **vue schématique** de l’exemple de fabrique de données que vous créez dans ce didacticiel. **MyFirstPipeline** a une activité de type Hive qui utilise le jeu de données **AzureBlobInput** comme une entrée et génère le jeu de données **AzureBlobOutput** en tant que sortie.

![Vue Diagramme dans le didacticiel Data Factory](media/data-factory-build-your-first-pipeline/data-factory-tutorial-diagram-view.png)


Dans ce didacticiel, le dossier **inputdata** du conteneur d’objets blob Azure **adfgetstarted** contient un fichier nommé input.log. Ce fichier journal comporte des entrées de trois mois : janvier, février et mars 2016. Voici les échantillons de lignes pour chaque mois du fichier d’entrée.

```
2016-01-01,02:01:09,SAMPLEWEBSITE,GET,/blogposts/mvc4/step2.png,X-ARR-LOG-ID=2ec4b8ad-3cf0-4442-93ab-837317ece6a1,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,53175,871
2016-02-01,02:01:10,SAMPLEWEBSITE,GET,/blogposts/mvc4/step7.png,X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,30184,871
2016-03-01,02:01:10,SAMPLEWEBSITE,GET,/blogposts/mvc4/step7.png,X-ARR-LOG-ID=d7472a26-431a-4a4d-99eb-c7b4fda2cf4c,80,-,1.54.23.196,Mozilla/5.0+(Windows+NT+6.3;+WOW64)+AppleWebKit/537.36+(KHTML,+like+Gecko)+Chrome/31.0.1650.63+Safari/537.36,-,http://weblogs.asp.net/sample/archive/2007/12/09/asp-net-mvc-framework-part-4-handling-form-edit-and-post-scenarios.aspx,\N,200,0,0,30184,871
```

Quand le fichier est traité par le pipeline avec l’activité Hive d’HDInsight, l’activité exécute un script Hive sur le cluster HDInsight qui partitionne les données d’entrée par année et par mois. Le script crée trois dossiers de sortie qui contiennent un fichier avec les entrées de chaque mois.  

```
adfgetstarted/partitioneddata/year=2016/month=1/000000_0
adfgetstarted/partitioneddata/year=2016/month=2/000000_0
adfgetstarted/partitioneddata/year=2016/month=3/000000_0
```

Dans les échantillons de lignes ci-dessus, la première (avec 2016-01-01) est écrite dans le fichier 000000_0 dans le dossier month=1. De même, la deuxième est écrite dans le fichier du dossier month=2 et la troisième est écrite dans le fichier du dossier month=3.  

## <a name="prerequisites"></a>Prérequis
Avant de commencer ce didacticiel, vous devez disposer des éléments suivants :

1. **Un abonnement Azure** : si vous n’en avez pas, vous pouvez créer un compte en quelques minutes pour une évaluation gratuite. Consultez l’article [Évaluation gratuite](https://azure.microsoft.com/pricing/free-trial/) pour savoir comment obtenir un compte d’évaluation gratuite.
2. **Stockage Azure** : dans ce didacticiel, vous utilisez un compte de stockage Azure pour stocker les données. Si vous ne possédez pas de compte de stockage Azure, consultez l’article [Créer un compte de stockage](../../storage/common/storage-account-create.md) . Après avoir créé le compte de stockage, notez le **nom du compte** et la **clé d’accès**. Pour plus d’informations sur la façon de récupérer les clés d’accès au compte de stockage, consultez [Gérer les clés d’accès au compte de stockage](../../storage/common/storage-account-keys-manage.md).
3. Téléchargez et lisez le fichier de requête Hive (**HQL**) à l’adresse : [https://adftutorialfiles.blob.core.windows.net/hivetutorial/partitionweblogs.hql](https://adftutorialfiles.blob.core.windows.net/hivetutorial/partitionweblogs.hql). Cette requête transforme les données d’entrée pour produire des données de sortie.
4. Téléchargez et lisez l’exemple de fichier d’entrée (**input.log**) à l’adresse : [https://adftutorialfiles.blob.core.windows.net/hivetutorial/input.log](https://adftutorialfiles.blob.core.windows.net/hivetutorial/input.log).
5. Créez un conteneur de blobs nommé **adfgetstarted** dans votre stockage Blob Azure.
6. Chargez le fichier **partitionweblogs.hql** dans le dossier **script** du conteneur **adfgetstarted**. Utilisez des outils tels que [l’Explorateur de stockage Microsoft Azure](https://storageexplorer.com/).
7. Chargez le fichier **input.log** dans le dossier **inputdata** du conteneur **adfgetstarted**.

Si vous disposez de tout ce qui est nécessaire, sélectionnez l’un des outils/kits de développement logiciel (SDK) ci-dessous pour suivre ce didacticiel :

- [Visual Studio](data-factory-build-your-first-pipeline-using-vs.md)
- [PowerShell](data-factory-build-your-first-pipeline-using-powershell.md)
- [modèle Azure Resource Manager](data-factory-build-your-first-pipeline-using-arm.md)
- [REST API](data-factory-build-your-first-pipeline-using-rest-api.md)

Visual Studio propose une méthode utilisant l’interface utilisateur graphique pour créer vos fabriques de données. Quant aux options fournies par l’API REST, le modèle Resource Manager et PowerShell, elles vous permettent de créer vos fabriques de données via des scripts et des programmes.

> [!NOTE]
> Dans ce didacticiel, le pipeline de données transforme les données d’entrée pour produire des données de sortie. Il ne copie pas les données d’un magasin de données source vers un magasin de données de destination. Pour suivre un tutoriel sur la copie des données à l’aide d’Azure Data Factory, consultez [Tutoriel : Copier des données de Stockage Blob vers SQL Database](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md)
>
> Vous pouvez chaîner deux activités (une après l’autre) en configurant le jeu de données de sortie d’une activité en tant que jeu de données d’entrée de l’autre activité. Pour plus d’informations, voir [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md).
