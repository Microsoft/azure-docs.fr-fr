---
title: Transformer des données à l’aide d’un script U-SQL – Azure
description: Découvrez comment traiter ou transformer les données en exécutant des scripts U-SQL sur le service de calcul Azure Data Lake Analytics – version 1.
services: data-factory
documentationcenter: ''
ms.assetid: e17c1255-62c2-4e2e-bb60-d25274903e80
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/01/2017
author: nabhishek
ms.author: abnarain
ms.custom: devx-track-csharp
manager: anandsub
robots: noindex
ms.openlocfilehash: a5e53cab30f1adca05652a3b3b7541e12ebebbdb
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92631459"
---
# <a name="transform-data-by-running-u-sql-scripts-on-azure-data-lake-analytics"></a>Transformer des données en exécutant des scripts U-SQL sur Azure Data Lake Analytics 
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-usql-activity.md)
> * [Version 2 (version actuelle)](../transform-data-using-data-lake-analytics.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Activité U-SQL dans V2](../transform-data-using-data-lake-analytics.md).

Un pipeline dans une fabrique de données Azure traite les données dans les services de stockage liés à l'aide des services de calcul liés. Il contient une séquence d'activités dans laquelle chaque activité effectue une opération de traitement spécifique. Cet article décrit l’ **activité U-SQL de Data Lake Analytics** qui exécute un script **U-SQL** sur un service lié de calcul **Azure Data Lake Analytics** . 

Créez un compte Azure Data Lake Analytics avant de créer un pipeline avec une activité U-SQL Data Lake Analytics. Pour plus d’informations sur Azure Data Lake Analytics, consultez [Prise en main d’Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-get-started-portal.md).

Consultez le [didacticiel Concevez votre premier pipeline](data-factory-build-your-first-pipeline.md) pour connaître les étapes détaillées de création d'une fabrique de données, de services liés, de jeux de données et d'un pipeline. Utilisez les extraits de code JSON avec Data Factory Editor ou Visual Studio ou Azure PowerShell pour créer les entités Data Factory.

## <a name="supported-authentication-types"></a>Types d’authentification pris en charge
L’activité U-SQL prend en charge les types d’authentification ci-dessous pour Data Lake Analytics :
* Authentification d’un principal du service
* Utilisation de l’authentification des informations d’identification utilisateur (OAuth) 

Nous vous recommandons d’utiliser l’authentification de principal du service, en particulier pour une exécution de U-SQL planifiée. Un comportement d’expiration de jeton peut se produire avec l’authentification par informations d’identification utilisateur. Pour les détails de configuration, consultez la section [Propriétés du service lié](#azure-data-lake-analytics-linked-service).

## <a name="azure-data-lake-analytics-linked-service"></a>Service lié Analytique Azure Data Lake
Vous créez un service lié **Analytique Azure Data Lake** pour lier un service de calcul Analytique Azure Data Lake Analytics à une fabrique de données Azure. L’activité U-SQL Analytique Data Lake dans le pipeline fait référence à ce service lié. 

Le tableau suivant décrit les propriétés génériques utilisées dans la définition JSON. Vous pouvez ensuite choisir entre une authentification par principal de service et par informations d’identification utilisateur.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| **type** |La propriété de type doit être définie sur : **AzureDataLakeAnalytics** . |Oui |
| **accountName** |Nom du compte du service Analytique Azure Data Lake. |Oui |
| **dataLakeAnalyticsUri** |URI du service Analytique Azure Data Lake. |Non |
| **subscriptionId** |ID d’abonnement Azure |Non (si non spécifié, l’abonnement de la fabrique de données est utilisé). |
| **resourceGroupName** |Nom du groupe de ressources Azure |Non (si non spécifié, le groupe de ressources de la fabrique de données est utilisé). |

### <a name="service-principal-authentication-recommended"></a>Authentification d’un principal du service (recommandée)
Pour utiliser l’authentification d’un principal du service, inscrivez une entité d’application dans Azure Active Directory (Azure AD) et lui accorder l’accès à Data Lake Store. Consultez la page [Authentification de service à service](../../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md) pour des instructions détaillées. Prenez note des valeurs suivantes, qui vous permettent de définir le service lié :
* ID de l'application
* Clé de l'application 
* ID client

Utilisez l’authentification par principal de service en spécifiant les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| **servicePrincipalId** | Spécifiez l’ID client de l’application. | Oui |
| **servicePrincipalKey** | Spécifiez la clé de l’application. | Oui |
| **client** | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Vous pouvez le récupérer en pointant la souris dans le coin supérieur droit du portail Azure. | Oui |

**Exemple : Authentification d’un principal de service**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>Authentification des informations d’identification utilisateur
Vous pouvez également utiliser l’authentification par informations d’identification utilisateur pour Data Lake Analytics en spécifiant les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| **autorisation** | Cliquez sur le bouton **Autoriser** dans Data Factory Editor et saisissez vos informations d’identification, ce qui affecte l’URL d’autorisation générée automatiquement à cette propriété. | Oui |
| **sessionId** | ID de session OAuth issu de la session d’autorisation OAuth. Chaque ID de session est unique et ne peut être utilisé qu’une seule fois. Ce paramètre est généré automatiquement lorsque vous utilisez Data Factory Editor. | Oui |

**Exemple : Authentification des informations d’identification utilisateur**
```json
{
    "name": "AzureDataLakeAnalyticsLinkedService",
    "properties": {
        "type": "AzureDataLakeAnalytics",
        "typeProperties": {
            "accountName": "adftestaccount",
            "dataLakeAnalyticsUri": "azuredatalakeanalytics.net",
            "authorization": "<authcode>",
            "sessionId": "<session ID>", 
            "subscriptionId": "<optional, subscription id of ADLA>",
            "resourceGroupName": "<optional, resource group name of ADLA>"
        }
    }
}
```

#### <a name="token-expiration"></a>Expiration du jeton
Le code d’autorisation que vous avez généré à l’aide du bouton **Autoriser** expire au bout d’un certain temps. Consultez le tableau suivant pour connaître les délais d’expiration associés aux différents types de comptes d’utilisateur. Vous pouvez rencontrer le message d’erreur suivant lorsque **le jeton d’authentification expire** : Erreur de l’opération des informations d’identification : invalid_grant - AADSTS70002 : Erreur de validation des informations d’identification. AADSTS70008 : L’autorisation d’accès fournie a expiré ou est révoquée. ID de trace : d18629e8-af88-43c5-88e3-d8419eb1fca1 ID de corrélation : fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Horodatage : 2015-12-15 21:09:31Z

| Type d’utilisateur | Expire après |
|:--- |:--- |
| Comptes d’utilisateurs NON gérés par Azure Active Directory (@hotmail.com, @live.com, etc.) |12 heures |
| Comptes d’utilisateurs gérés par Azure Active Directory (AAD) |14 jours après la dernière exécution de tranche de données. <br/><br/>90 jours, si une tranche basée sur un service lié OAuth est exécutée au moins une fois tous les 14 jours. |

Pour éviter ou résoudre cette erreur, accordez une nouvelle autorisation à l’aide du bouton **Autoriser** au moment de **l’expiration du jeton** , puis redéployer le service lié. Vous pouvez également générer par programmation des valeurs pour les propriétés **sessionId** et **authorization** à l’aide du code suivant :

```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```

Pour plus d’informations sur les classes Data Factory utilisées dans le code, consultez les rubriques [AzureDataLakeStoreLinkedService, classe](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice), [AzureDataLakeAnalyticsLinkedService, classe](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice) et [AuthorizationSessionGetResponse, classe](/dotnet/api/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse). Ajoutez une référence à : Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll pour la classe WindowsFormsWebAuthenticationDialog. 

## <a name="data-lake-analytics-u-sql-activity"></a>Activité U-SQL Data Lake Analytics
L'extrait de code JSON suivant définit un pipeline avec une activité U-SQL Data Lake Analytics. La définition d'activité comporte une référence au service lié Azure Data Lake Analytics créé précédemment.   

```json
{
    "name": "ComputeEventsByRegionPipeline",
    "properties": {
        "description": "This is a pipeline to compute events for en-gb locale and date less than 2012/02/19.",
        "activities": 
        [
            {
                "type": "DataLakeAnalyticsU-SQL",
                "typeProperties": {
                    "scriptPath": "scripts\\kona\\SearchLogProcessing.txt",
                    "scriptLinkedService": "StorageLinkedService",
                    "degreeOfParallelism": 3,
                    "priority": 100,
                    "parameters": {
                        "in": "/datalake/input/SearchLog.tsv",
                        "out": "/datalake/output/Result.tsv"
                    }
                },
                "inputs": [
                    {
                        "name": "DataLakeTable"
                    }
                ],
                "outputs": 
                [
                    {
                        "name": "EventsByRegionTable"
                    }
                ],
                "policy": {
                    "timeout": "06:00:00",
                    "concurrency": 1,
                    "executionPriorityOrder": "NewestFirst",
                    "retry": 1
                },
                "scheduler": {
                    "frequency": "Day",
                    "interval": 1
                },
                "name": "EventsByRegion",
                "linkedServiceName": "AzureDataLakeAnalyticsLinkedService"
            }
        ],
        "start": "2015-08-08T00:00:00Z",
        "end": "2015-08-08T01:00:00Z",
        "isPaused": false
    }
}
```

Le tableau suivant indique les noms et les descriptions des propriétés qui sont spécifiques à cette activité. 

| Propriété            | Description                              | Obligatoire                                 |
| :------------------ | :--------------------------------------- | :--------------------------------------- |
| type                | La propriété de type doit être définie sur **DataLakeAnalyticsU-SQL** . | Oui                                      |
| linkedServiceName   | Référence au Azure Data Lake Analytics enregistré en tant que service lié dans Data Factory | Oui                                      |
| scriptPath          | Chemin d'accès au dossier qui contient le script SQL-U. Le nom de fichier respecte la casse. | Non (si vous utilisez le script)                   |
| scriptLinkedService | Service lié qui lie le stockage qui contient le script à la fabrique de données | Non (si vous utilisez le script)                   |
| script              | Spécifiez un script en ligne au lieu de spécifier scriptPath et scriptLinkedService. Par exemple : `"script": "CREATE DATABASE test"`. | Non (si vous utilisez scriptPath et scriptLinkedService) |
| degreeOfParallelism | Le nombre maximal de nœuds utilisés simultanément pour exécuter le travail. | Non                                       |
| priority            | Détermine les travaux parmi tous ceux qui sont en file d'attente qui doivent être sélectionnés pour s'exécuter en premier. Plus le numéro est faible, plus la priorité est élevée. | Non                                       |
| parameters          | Paramètres du script U-SQL          | Non                                       |
| runtimeVersion      | Version du runtime du moteur U-SQL à utiliser | Non                                       |
| compilationMode     | <p>Mode de compilation d’U-SQL. Doit avoir l’une des valeurs suivantes :</p> <ul><li>**Semantic :** exécuter uniquement les vérifications sémantiques et les contrôles d’intégrité nécessaires.</li><li>**Full :** effectuer la compilation complète, notamment la vérification de la syntaxe, l’optimisation, la génération de code, et ainsi de suite.</li><li>**SingleBox :** effectuer la compilation complète, avec le paramètre TargetType défini sur SingleBox.</li></ul><p>Si vous ne spécifiez pas de valeur pour cette propriété, le serveur détermine le mode de compilation optimal. </p> | Non                                       |

Vous trouverez la définition du script dans la section [Définition du script SearchLogProcessing.txt](#sample-u-sql-script) . 

## <a name="sample-input-and-output-datasets"></a>Exemples de jeux de données d'entrée et de sortie
### <a name="input-dataset"></a>Jeu de données d'entrée
Dans cet exemple, les données d'entrée se trouvent dans un magasin Azure Data Lake (fichier SearchLog.tsv dans le dossier datalake/input). 

```json
{
    "name": "DataLakeTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}    
```

### <a name="output-dataset"></a>Jeu de données de sortie
Dans cet exemple, les données de sortie générées par le script U-SQL sont stockées dans un magasin Azure Data Lake (dossier datalake/output). 

```json
{
    "name": "EventsByRegionTable",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
            "frequency": "Day",
            "interval": 1
        }
    }
}
```

### <a name="sample-data-lake-store-linked-service"></a>Exemple de service lié Data Lake Store
Voici la définition de l’exemple de service lié Azure Data Lake Store utilisé par les jeux de données d’entrée/de sortie. 

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
        }
    }
}
```

Consultez l’article [Déplacer des données vers et depuis Azure Data Lake Store](data-factory-azure-datalake-connector.md) pour obtenir une description des propriétés JSON. 

## <a name="sample-u-sql-script"></a>Exemple de script SQL-U

```
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
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

Les valeurs des paramètres **\@in** et **\@out** dans le script U-SQL sont passées dynamiquement par ADF en utilisant la section « parameters ». Consultez la section « parameters » dans la définition du pipeline.

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
    "in": "$$Text.Format('/datalake/input/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)",
    "out": "$$Text.Format('/datalake/output/{0:yyyy-MM-dd HH:mm:ss}.tsv', SliceStart)"
}
```

Dans ce cas, les fichiers d’entrée sont toujours récupérés à partir du dossier /datalake/input et les fichiers de sortie sont générés dans le dossier /datalake/output. Les noms de fichiers sont dynamiques et basés sur l’heure de début de la tranche horaire.