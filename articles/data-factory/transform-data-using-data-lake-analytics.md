---
title: Transformer des données à l’aide d’un script U-SQL
description: Découvrez comment traiter ou transformer les données en exécutant des scripts U-SQL sur le service de calcul Azure Data Lake Analytics.
services: data-factory
documentationcenter: ''
ms.author: abnarain
author: nabhishek
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/01/2018
ms.openlocfilehash: d890f73ea8b2294755b14055cb11904d50160cc4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96014004"
---
# <a name="process-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>Traiter des données en exécutant des scripts U-SQL sur Azure Data Lake Analytics 
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-usql-activity.md)
> * [Version actuelle](transform-data-using-data-lake-analytics.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Un pipeline dans une fabrique de données Azure traite les données dans les services de stockage liés à l'aide des services de calcul liés. Il contient une séquence d'activités dans laquelle chaque activité effectue une opération de traitement spécifique. Cet article décrit l’**activité U-SQL de Data Lake Analytics** qui exécute un script **U-SQL** sur un service lié de calcul **Azure Data Lake Analytics**. 

Créez un compte Azure Data Lake Analytics avant de créer un pipeline avec une activité U-SQL Data Lake Analytics. Pour plus d’informations sur Azure Data Lake Analytics, consultez [Prise en main d’Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md).


## <a name="azure-data-lake-analytics-linked-service"></a>Service lié Azure Data Lake Analytics
Vous créez un service lié **Analytique Azure Data Lake** pour lier un service de calcul Analytique Azure Data Lake Analytics à une fabrique de données Azure. L’activité U-SQL Analytique Data Lake dans le pipeline fait référence à ce service lié. 

Le tableau suivant décrit les propriétés génériques utilisées dans la définition JSON. 

| Propriété                 | Description                              | Obligatoire                                 |
| ------------------------ | ---------------------------------------- | ---------------------------------------- |
| **type**                 | La propriété de type doit être définie sur : **AzureDataLakeAnalytics**. | Oui                                      |
| **accountName**          | Nom du compte du service Analytique Azure Data Lake.  | Oui                                      |
| **dataLakeAnalyticsUri** | URI du service Analytique Azure Data Lake.           | Non                                       |
| **subscriptionId**       | ID d’abonnement Azure                    | Non                                       |
| **resourceGroupName**    | Nom du groupe de ressources Azure                | Non                                       |

### <a name="service-principal-authentication"></a>Authentification d’un principal du service
Le service lié d’Azure Data Lake Analytics a besoin d’une authentification de principal de service pour se connecter au service Azure Data Lake Analytics. Pour utiliser l’authentification d’un principal de service, inscrivez une entité d’application auprès d’Azure Active Directory (Azure AD) et accordez-lui l’accès aux services Data Lake Analytics et Data Lake Store qu’elle utilise. Consultez la page [Authentification de service à service](../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md) pour des instructions détaillées. Prenez note des valeurs suivantes, qui vous permettent de définir le service lié :

* ID de l'application
* Clé de l'application 
* ID client

Accordez l’autorisation de principal de service à Azure Data Lake Analytics à l’aide de [l’Assistant Ajout d’un utilisateur](../data-lake-analytics/data-lake-analytics-manage-use-portal.md#add-a-new-user).

Utilisez l’authentification par principal de service en spécifiant les propriétés suivantes :

| Propriété                | Description                              | Obligatoire |
| :---------------------- | :--------------------------------------- | :------- |
| **servicePrincipalId**  | Spécifiez l’ID client de l’application.     | Oui      |
| **servicePrincipalKey** | Spécifiez la clé de l’application.           | Oui      |
| **client**              | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Vous pouvez le récupérer en pointant la souris dans le coin supérieur droit du portail Azure. | Oui      |

**Exemple : Authentification d’un principal de service**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "<account name>",
            "dataLakeAnalyticsUri": "<azure data lake analytics URI>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "value": "<service principal key>",
                "type": "SecureString"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }       
    }
}
```

Pour en savoir plus sur le service lié, consultez [Services liés de calcul](compute-linked-services.md).

## <a name="data-lake-analytics-u-sql-activity"></a>Activité U-SQL Data Lake Analytics
L'extrait de code JSON suivant définit un pipeline avec une activité U-SQL Data Lake Analytics. La définition d'activité comporte une référence au service lié Azure Data Lake Analytics créé précédemment. Pour exécuter un script U-SQL Data Lake Analytics, Data Factory soumet le script que vous avez spécifié au service Data Lake Analytics tandis que les entrées et les sorties nécessaires sont définies dans le script, afin que Data Lake Analytics procède à l’extraction et à la sortie. 

```json
{
    "name": "ADLA U-SQL Activity",
    "description": "description",
    "type": "DataLakeAnalyticsU-SQL",
    "linkedServiceName": {
        "referenceName": "<linked service name of Azure Data Lake Analytics>",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "scriptLinkedService": {
            "referenceName": "<linked service name of Azure Data Lake Store or Azure Storage which contains the U-SQL script>",
            "type": "LinkedServiceReference"
        },
        "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
        "degreeOfParallelism": 3,
        "priority": 100,
        "parameters": {
            "in": "/datalake/input/SearchLog.tsv",
            "out": "/datalake/output/Result.tsv"
        }
    }   
}
```

Le tableau suivant indique les noms et les descriptions des propriétés qui sont spécifiques à cette activité. 

| Propriété            | Description                              | Obligatoire |
| :------------------ | :--------------------------------------- | :------- |
| name                | Nom de l’activité dans le pipeline     | Oui      |
| description         | Texte décrivant l’activité.  | Non       |
| type                | Pour l’activité U-SQL Data Lake Analytics, le type d’activité est **DataLakeAnalyticsU-SQL**. | Oui      |
| linkedServiceName   | Service lié à Azure Data Lake Analytics. Pour en savoir plus sur ce service lié, consultez l’article [Services liés de calcul](compute-linked-services.md).  |Oui       |
| scriptPath          | Chemin d'accès au dossier qui contient le script SQL-U. Le nom de fichier respecte la casse. | Oui      |
| scriptLinkedService | Service lié qui lie **Azure Data Lake Store** ou le **stockage Azure** qui contient le script à la fabrique de données | Oui      |
| degreeOfParallelism | Le nombre maximal de nœuds utilisés simultanément pour exécuter le travail. | Non       |
| priority            | Détermine les travaux parmi tous ceux qui sont en file d'attente qui doivent être sélectionnés pour s'exécuter en premier. Plus le numéro est faible, plus la priorité est élevée. | Non       |
| parameters          | Paramètres à transmettre au script U-SQL.    | Non       |
| runtimeVersion      | Version du runtime du moteur U-SQL à utiliser. | Non       |
| compilationMode     | <p>Mode de compilation d’U-SQL. Doit avoir l’une des valeurs suivantes : **Semantic :** Exécuter uniquement les vérifications sémantiques et les contrôles d’intégrité nécessaires. **Full :** Effectuer la compilation complète, y compris la vérification de la syntaxe, l’optimisation, la génération de code, etc. **SingleBox :** effectuer la compilation complète, avec le paramètre TargetType défini sur SingleBox. Si vous ne spécifiez pas de valeur pour cette propriété, le serveur détermine le mode de compilation optimal. | Non |

Vous trouverez la définition du script dans la section [SearchLogProcessing.txt](#sample-u-sql-script). 

## <a name="sample-u-sql-script"></a>Exemple de script U-SQL

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int,
            Urls            string,
            ClickedUrls     string
    FROM @in
    USING Extractors.Tsv(nullEscape:"#NULL#");

@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";

@rs1 =
    SELECT Start, Region, Duration
    FROM @rs1
    WHERE Start <= DateTime.Parse("2012/02/19");

OUTPUT @rs1   
    TO @out
      USING Outputters.Tsv(quoting:false, dateTimeFormat:null);
```

Dans l’exemple de script ci-dessus, l’entrée et la sortie du script sont définies dans les paramètres **\@in** et **\@out**. Les valeurs des paramètres **\@in** et **\@out** dans le script U-SQL sont passées dynamiquement par Data Factory en utilisant la section « parameters ». 

Vous pouvez aussi spécifier d’autres propriétés comme degreeOfParallelism et priority dans votre définition de pipeline pour les travaux qui s’exécutent au niveau du service Azure Data Lake Analytics.

## <a name="dynamic-parameters"></a>Paramètres dynamiques
Dans l’exemple de définition de pipeline, des valeurs codées en dur sont affectées aux paramètres de sortie. 

```json
"parameters": {
    "in": "/datalake/input/SearchLog.tsv",
    "out": "/datalake/output/Result.tsv"
}
```

Il est possible d’utiliser des paramètres dynamiques à la place. Par exemple : 

```json
"parameters": {
    "in": "/datalake/input/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/data.tsv",
    "out": "/datalake/output/@{formatDateTime(pipeline().parameters.WindowStart,'yyyy/MM/dd')}/result.tsv"
}
```

Dans ce cas, les fichiers d’entrée sont toujours récupérés à partir du dossier /datalake/input et les fichiers de sortie sont générés dans le dossier /datalake/output. Les noms de fichiers sont dynamiques en fonction de l’heure de début de la fenêtre lorsque le pipeline est déclenché.  

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants qui expliquent comment transformer des données par d’autres moyens : 

* [Activité Hive](transform-data-using-hadoop-hive.md)
* [Activité pig](transform-data-using-hadoop-pig.md)
* [Activité MapReduce](transform-data-using-hadoop-map-reduce.md)
* [Activité de diffusion en continu Hadoop](transform-data-using-hadoop-streaming.md)
* [Activité Spark](transform-data-using-spark.md)
* [Activité personnalisée .NET](transform-data-using-dotnet-custom-activity.md)
* [Activité Exécution par lots Azure Machine Learning studio (classique)](transform-data-using-machine-learning.md)
* [Activité de procédure stockée](transform-data-using-stored-procedure.md)