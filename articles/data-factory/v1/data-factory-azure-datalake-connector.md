---
title: Copier des données vers et depuis Azure Data Lake Storage Gen1
description: Découvrez comment copier des données depuis et vers Data Lake Store à l’aide d’Azure Data Factory
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: 25b1ff3c-b2fd-48e5-b759-bb2112122e30
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
ms.custom: devx-track-csharp
robots: noindex
ms.openlocfilehash: 16cef1fb35efcbe12a4054304e3f354c03b37227
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637647"
---
# <a name="copy-data-to-and-from-data-lake-storage-gen1-by-using-data-factory"></a>Copier des données vers et depuis Data Lake Storage Gen1 à l’aide de Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-azure-datalake-connector.md)
> * [Version 2 (version actuelle)](../connector-azure-data-lake-store.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Azure Data Lake Storage Gen1 connector in V2](../connector-azure-data-lake-store.md) (Connecteur Azure Data Lake Storage Gen1 dans V2).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données vers et depuis Azure Data Lake Storage Gen1 (appelé auparavant Azure Data Lake Store). Il s’appuie sur l’article [Activités de déplacement des données](data-factory-data-movement-activities.md), une vue d’ensemble du déplacement de données avec l’activité de copie.

## <a name="supported-scenarios"></a>Scénarios pris en charge
Vous pouvez copier les données **à partir d’Azure Data Lake Store** dans les magasins de données suivants :

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

Vous pouvez copier les données des magasins de données suivants **dans Azure Data Lake Store** :

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!NOTE]
> Créez un compte Data Lake Store avant de créer un pipeline avec activité de copie. Pour plus d’informations, consultez [Prise en main d’Azure Data Lake Store](../../data-lake-store/data-lake-store-get-started-portal.md).

## <a name="supported-authentication-types"></a>Types d’authentification pris en charge
Le connecteur Data Lake Store prend en charge les types d’authentification suivants :
* Authentification d’un principal du service
* Utilisation de l’authentification des informations d’identification utilisateur (OAuth)

Nous vous recommandons d’utiliser l’authentification de principal du service, en particulier pour une copie planifiée des données. Un comportement d’expiration de jeton peut se produire avec l’authentification par informations d’identification utilisateur. Pour les détails de configuration, consultez la section [Propriétés du service lié](#linked-service-properties).

## <a name="get-started"></a>Bien démarrer
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données vers ou à partir d’Azure Data Lake Store à l’aide de différents outils/API.

Le moyen le plus simple de créer un pipeline pour copier des données consiste à utiliser **l’Assistant de copie** . Pour suivre un tutoriel sur la création d’un pipeline à l’aide de l’Assistant de copie, consultez [Tutoriel : Créer un pipeline à l’aide de l’Assistant de copie](data-factory-copy-data-wizard-tutorial.md).

Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio** , **Azure PowerShell** , **modèle Azure Resource Manager** , **.NET API** et **API REST** . Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md).

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création d'une **fabrique de données** . Une fabrique de données peut contenir un ou plusieurs pipelines.
2. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données. Par exemple, si vous copiez des données d’un stockage Blob Azure dans Azure Data Lake Store, vous créez deux services liés pour lier votre compte de stockage Azure et Azure Data Lake Store à votre fabrique de données. Pour plus d’informations sur les propriétés de service lié qui sont spécifiques à Azure Data Lake Store, consultez la section [Propriétés du service lié](#linked-service-properties).
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie. Dans l’exemple mentionné dans la dernière étape, vous créez un jeu de données pour spécifier le conteneur d’objets blob et le dossier qui contient les données d’entrée. Ensuite, vous créez un autre jeu de données pour spécifier le chemin du dossier/fichier dans l’instance de Data Lake Store qui contient les données copiées à partir du stockage Blob. Pour les propriétés du jeu de données qui sont spécifiques à Azure Data Lake Store, consultez la section [propriétés du jeu de données](#dataset-properties).
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie. Dans l’exemple mentionné plus haut, vous utilisez BlobSource comme source et AzureDataLakeStoreSink comme récepteur pour l’activité de copie. De la même façon, si vous copiez des données d’Azure Data Lake Store vers le Stockage Blob Azure, vous utilisez AzureDataLakeStoreSource et BlobSink dans l’activité de copie. Pour plus d’informations sur les propriétés de l’activité de copie qui sont spécifiques à Azure Data Lake Store, consultez la section [Propriétés de l’activité de copie](#copy-activity-properties). Pour plus d’informations sur l’utilisation d’un magasin de données comme source ou comme récepteur, cliquez sur le lien de la section précédente de votre magasin de données.

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON. Pour obtenir des exemples comportant des définitions JSON pour les entités Data Factory utilisées pour copier les données vers ou à partir d’Azure Data Lake Store, consultez la section [Exemples JSON](#json-examples-for-copying-data-to-and-from-data-lake-store) de cet article.

Les sections suivantes offrent des informations détaillées sur les propriétés JSON utilisées pour définir les entités Data Factory propres à Data Lake Store.

## <a name="linked-service-properties"></a>Propriétés du service lié
Un service lié lie un magasin de données à une fabrique de données. Pour lier votre magasin de données Data Lake Store à votre fabrique de données, vous devez créer un service lié de type **AzureDataLakeStore** . Le tableau suivant décrit les éléments JSON spécifiques pour des services Data Lake Store liés. Vous pouvez choisir entre une authentification par principal de service et par informations d’identification utilisateur.

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| **type** | La propriété type doit être définie sur : **AzureDataLakeStore** . | Oui |
| **dataLakeStoreUri** | Informations à propos du compte Azure Data Lake Store. Cette information prend un des formats suivants : `https://[accountname].azuredatalakestore.net/webhdfs/v1` ou `adl://[accountname].azuredatalakestore.net/`. | Oui |
| **subscriptionId** | ID d’abonnement Azure auquel appartient le compte Data Lake Store. | Requis pour le récepteur |
| **resourceGroupName** | Nom du groupe de ressources Azure auquel appartient le compte Data Lake Store. | Requis pour le récepteur |

### <a name="service-principal-authentication-recommended"></a>Authentification d’un principal du service (recommandée)
Pour utiliser l’authentification d’un principal du service, inscrivez une entité d’application dans Azure Active Directory (Azure AD) et lui accorder l’accès à Data Lake Store. Consultez la page [Authentification de service à service](../../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md) pour des instructions détaillées. Prenez note des valeurs suivantes, qui vous permettent de définir le service lié :
* ID de l'application
* Clé de l'application
* ID client

> [!IMPORTANT]
> Veillez à accorder l’autorisation appropriée au principal de service dans Azure Data Lake Store :
>- **Pour utiliser Data Lake Store en tant que source** , accordez au moins l’autorisation d’accès aux données **Lecture + Exécution** pour répertorier et copier le contenu d’un dossier, ou l’autorisation **Lecture** pour copier un seul fichier. Aucune exigence sur le contrôle d’accès au niveau du compte.
>- **Pour utiliser Data Lake Store en tant que récepteur** , accordez au moins l’autorisation d’accès aux données **Écriture + Exécution** pour créer des éléments enfants dans le dossier. Et si vous utilisez Azure IR pour autoriser la copie (la source et le récepteur sont tous les deux dans le cloud) et permettre la détection par Data Factory de la région de Data Lake Store, accordez au moins le rôle **Lecteur** dans le contrôle d’accès au compte (IAM). Si vous souhaitez éviter ce rôle IAM, choisissez de [spécifier executionLocation](data-factory-data-movement-activities.md#global) avec l’emplacement de votre Data Lake Store dans l’activité de copie.
>- Si vous **utilisez l’Assistant Copie pour créer des pipelines** , accordez au moins le rôle **Lecteur** dans le contrôle d’accès au compte (IAM). De plus, accordez au moins l’autorisation **Lecture + Exécution** à la racine de Data Lake Store (« / ») et à ses enfants. Sinon, le message « Les informations d’identification fournies ne sont pas valides » peut s’afficher.

Utilisez l’authentification par principal de service en spécifiant les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| **servicePrincipalId** | Spécifiez l’ID client de l’application. | Oui |
| **servicePrincipalKey** | Spécifiez la clé de l’application. | Oui |
| **client** | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Vous pouvez le récupérer en pointant la souris dans le coin supérieur droit du portail Azure. | Oui |

**Exemple : Authentification d’un principal de service**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>Authentification des informations d’identification utilisateur
Vous pouvez également utiliser l’authentification des informations d’identification utilisateur pour la copie vers ou à partir de Data Lake Store en spécifiant les propriétés ci-dessous :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| **autorisation** | Cliquez sur le bouton **Autoriser** dans Data Factory Editor et saisissez vos informations d’identification, ce qui affecte l’URL d’autorisation générée automatiquement à cette propriété. | Oui |
| **sessionId** | ID de session OAuth issu de la session d’autorisation OAuth. Chaque ID de session est unique et ne peut être utilisé qu’une seule fois. Ce paramètre est généré automatiquement lorsque vous utilisez Data Factory Editor. | Oui |

> [!IMPORTANT]
> Veillez à accorder l’autorisation appropriée à l’utilisateur dans Azure Data Lake Store :
>- **Pour utiliser Data Lake Store en tant que source** , accordez au moins l’autorisation d’accès aux données **Lecture + Exécution** pour répertorier et copier le contenu d’un dossier, ou l’autorisation **Lecture** pour copier un seul fichier. Aucune exigence sur le contrôle d’accès au niveau du compte.
>- **Pour utiliser Data Lake Store en tant que récepteur** , accordez au moins l’autorisation d’accès aux données **Écriture + Exécution** pour créer des éléments enfants dans le dossier. Et si vous utilisez Azure IR pour autoriser la copie (la source et le récepteur sont tous les deux dans le cloud) et permettre la détection par Data Factory de la région de Data Lake Store, accordez au moins le rôle **Lecteur** dans le contrôle d’accès au compte (IAM). Si vous souhaitez éviter ce rôle IAM, choisissez de [spécifier executionLocation](data-factory-data-movement-activities.md#global) avec l’emplacement de votre Data Lake Store dans l’activité de copie.
>- Si vous **utilisez l’Assistant Copie pour créer des pipelines** , accordez au moins le rôle **Lecteur** dans le contrôle d’accès au compte (IAM). De plus, accordez au moins l’autorisation **Lecture + Exécution** à la racine de Data Lake Store (« / ») et à ses enfants. Sinon, le message « Les informations d’identification fournies ne sont pas valides » peut s’afficher.

**Exemple : Authentification des informations d’identification utilisateur**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

#### <a name="token-expiration"></a>Expiration du jeton
Le code d’autorisation que vous générez à l’aide du bouton **Autoriser** expire au bout d’un certain temps. Le message suivant signifie que le jeton d’authentification a expiré :

Erreur de l’opération des informations d’identification : invalid_grant - AADSTS70002 : Erreur de validation des informations d’identification. AADSTS70008 : L’autorisation d’accès fournie a expiré ou est révoquée. ID de trace : d18629e8-af88-43c5-88e3-d8419eb1fca1 ID de corrélation : fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Horodatage : 2015-12-15 21-09-31Z.

Le tableau suivant présente les délais d’expiration associés aux différents types de comptes d’utilisateur :

| Type d’utilisateur | Expire après |
|:--- |:--- |
| Comptes d’utilisateurs *non* gérés par Azure Active Directory (par exemple @hotmail.com ou @live.com) |12 heures |
| Comptes d’utilisateurs gérés par Azure Active Directory |14 jours après la dernière exécution de tranche de données <br/><br/>90 jours, si une tranche basée sur un service lié OAuth est exécutée au moins une fois tous les 14 jours |

Si vous modifiez votre mot de passe avant cette date d’expiration du jeton, le jeton expirera immédiatement. Vous verrez le message mentionné précédemment dans cette section.

Vous pouvez autoriser à nouveau le compte à l’aide du bouton **Autoriser** à l’expiration du jeton, puis redéployer le service lié. Vous pouvez également générer des valeurs pour les propriétés **sessionId** et **authorization** à l’aide du code suivant :


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
Pour plus d’informations sur les classes Data Factory utilisées dans le code, consultez les rubriques [AzureDataLakeStoreLinkedService, classe](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice), [AzureDataLakeAnalyticsLinkedService, classe](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice) et [AuthorizationSessionGetResponse, classe](/dotnet/api/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse). Ajoutez une référence à la version `2.9.10826.1824` de `Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll` pour la classe `WindowsFormsWebAuthenticationDialog` utilisée dans le code.

## <a name="troubleshooting-tips"></a>Conseils de dépannage

**Symptôme :** Lors de la copie de données **dans** Azure Data Lake Store, si votre activité de copie échoue avec l’erreur suivante :

  ```
  Failed to detect the region for Azure Data Lake account {your account name}. Please make sure that the Resource Group name: {resource group name} and subscription ID: {subscription ID} of this Azure Data Lake Store resource are correct.
  ```

**Cause racine :** Il existe deux causes possibles :

1. Le `resourceGroupName` et/ou `subscriptionId` spécifié dans le service lié Azure Data Lake Store est incorrect
2. L’utilisateur ou le principal de service n’a pas l’autorisation nécessaire

**Résolution :**

1. Vérifiez que le `subscriptionId` et `resourceGroupName` que vous spécifiez dans le service lié `typeProperties` sont bien ceux appartenant à votre compte Data Lake.

2. Veillez à accorder au moins le rôle **Lecteur** à l’utilisateur ou au principal de service sur le compte Data Lake. Voici comment faire :

    1. Accédez au portail Azure -> votre compte Data Lake Store.
    2. Cliquez sur **Contrôle d’accès (IAM)** dans le panneau du Data Lake Store.
    3. Cliquez sur **Ajouter une attribution de rôle** .
    4. Affectez à **Rôle** la valeur **Lecteur** et sélectionnez l’utilisateur ou le principal de service que vous utilisez pour la copie afin d’octroyer l’accès.

3. Si vous ne souhaitez pas accorder le rôle de **Lecteur** à l’utilisateur ou au principal de service, l’alternative consiste à [spécifier explicitement un emplacement d’exécution](data-factory-data-movement-activities.md#global) dans l’activité de copie avec l’emplacement de votre Data Lake Store. Exemple :

    ```json
    {
      "name": "CopyToADLS",
      "type": "Copy",
      ......
      "typeProperties": {
        "source": {
          "type": "<source type>"
        },
        "sink": {
          "type": "AzureDataLakeStoreSink"
        },
        "exeuctionLocation": "West US"
      }
    }
    ```

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour spécifier un jeu de données afin de représenter les données d’entrée dans un Data Lake Store, vous devez définir la propriété **type** du jeu de données sur **AzureDataLakeStore** . Définissez la propriété **linkedServiceName** du jeu de données sur le nom du service lié Data Lake Store. Pour obtenir une liste complète des sections et propriétés JSON disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections d’un jeu de données en JSON, comme la **structure** , la **disponibilité** et la **stratégie** sont similaires pour tous les types de jeux de données (par exemple, Azure SQL Database, Azure Blob et Azure Table). La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations notamment sur l'emplacement et le format des données dans le magasin de données.

La section **typeProperties** correspondant au jeu de données de type **AzureDataLakeStore** contient les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| **folderPath** |Chemin d’accès au conteneur et au dossier dans Data Lake Store. |Oui |
| **fileName** |Le nom du fichier dans Azure Data Lake Store. La propriété **fileName** est facultative et sensible à la casse. <br/><br/>Si vous spécifiez **fileName** , l’activité (y compris la copie) fonctionne sur le fichier spécifique.<br/><br/>Lorsque **fileName** n’est pas spécifié, la copie inclut tous les fichiers dans le paramètre **folderPath** du jeu de données d’entrée.<br/><br/>Lorsque **fileName** n'est pas spécifié pour un jeu de données de sortie et que **preserveHierarchy** n'est pas spécifié dans le récepteur d'activité, le nom du fichier généré est au format `Data._Guid_.txt`. Par exemple : Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt. |Non |
| **partitionedBy** |La propriété **partitionedBy** est facultative. Vous pouvez l'utiliser pour spécifier un chemin dynamique et le nom de fichier pour les données de série chronologique. Par exemple, **folderPath** peut être paramétré pour toutes les heures de données. Consultez La propriété partitionedBy pour obtenir plus d’informations et des exemples. |Non |
| **format** | Les types de formats suivants sont pris en charge : **TextFormat** , **JsonFormat** , **AvroFormat** , **OrcFormat** et **ParquetFormat** . Définissez la propriété **type** située sous **Format** sur l’une de ces valeurs. Pour en savoir plus, voir les sections [Format Text](data-factory-supported-file-and-compression-formats.md#text-format), [Format JSON](data-factory-supported-file-and-compression-formats.md#json-format), [Format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [Format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format) dans l’article [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md). <br><br> Si vous souhaitez copier des fichiers en l’état entre des magasins de fichiers (copie binaire), ignorez la section `format` dans les deux définitions de jeu de données d’entrée et de sortie. |Non |
| **compression** | Spécifiez le type et le niveau de compression pour les données. Les types pris en charge sont : **GZip** , **Deflate** , **BZip2** et **ZipDeflate** . Les niveaux pris en charge sont **Optimal** et **Fastest** . Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

### <a name="the-partitionedby-property"></a>La propriété partitionedBy
Vous pouvez spécifier des propriétés **folderPath** et un **fileName** dynamiques pour les données de série chronologique avec la propriété **partitionedBy** , les fonctions Data Factory et les variables système. Pour plus de détails, consultez l’article [Azure Data Factory - Variables système et fonctions](data-factory-functions-variables.md).


Dans l’exemple suivant, `{Slice}` est remplacé par la valeur de la variable système Data Factory `SliceStart` au format spécifié (`yyyyMMddHH`). Le nom `SliceStart` fait référence à l’heure de début de la tranche. La propriété `folderPath` est différente pour chaque secteur, comme dans `wikidatagateway/wikisampledataout/2014100103` ou `wikidatagateway/wikisampledataout/2014100104`.

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

Dans l’exemple suivant, l’année, le mois, le jour et l’heure de `SliceStart` sont extraits dans des variables distinctes qui sont utilisées par les propriétés `folderPath` et `fileName` :
```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
[
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```
Consultez les articles [Jeux de données dans Azure Data Factory](data-factory-create-datasets.md) et [Planification et exécution dans Data Factory](data-factory-scheduling-and-execution.md) pour mieux comprendre les jeux de données de série chronologique, la planification et les segments.


## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d’entrée et de sortie et la stratégie sont disponibles pour tous les types d’activités.

Les propriétés disponibles dans la section **typeProperties** d’une activité varient pour chaque type d’activité. Pour une activité de copie, elles dépendent des types de sources et récepteurs.

**AzureDataLakeStoreSource** prend en charge les propriétés suivantes dans la section **typeProperties**  :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| **recursive** |Indique si les données sont lues de manière récursive à partir des sous-dossiers ou uniquement du dossier spécifié. |True (valeur par défaut), False |Non |

**AzureDataLakeStoreSink** prend en charge les propriétés suivantes dans la section **typeProperties**  :

| Propriété | Description | Valeurs autorisées | Obligatoire |
| --- | --- | --- | --- |
| **copyBehavior** |Spécifie le comportement de copie. |<b>PreserveHierarchy</b> : conserve la hiérarchie des fichiers dans le dossier cible. Le chemin d’accès relatif du fichier source vers le dossier source est identique au chemin d’accès relatif du fichier cible vers le dossier cible.<br/><br/><b>FlattenHierarchy</b> : tous les fichiers du dossier source sont créés dans le premier niveau du dossier cible. Les fichiers cibles sont créés avec un nom généré automatiquement.<br/><br/><b>MergeFiles</b> : fusionne tous les fichiers du dossier source dans un seul fichier. Si le nom d’objet blob ou de fichier est spécifié, le nom de fichier fusionné est le nom spécifié. Dans le cas contraire, le nom de fichier est généré automatiquement. |Non |

### <a name="recursive-and-copybehavior-examples"></a>exemples de valeurs recursive et copyBehavior
Cette section décrit le comportement résultant de l’opération de copie pour différentes combinaisons de valeurs recursive et copyBehavior.

| recursive | copyBehavior | Comportement résultant |
| --- | --- | --- |
| true |preserveHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré de la même manière que la source<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5. |
| true |flattenHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier5 |
| true |mergeFiles |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible Dossier1 est créé et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 + Fichier3 + Fichier4 + Fichier5 est fusionné dans un fichier avec le nom de fichier généré automatiquement |
| false |preserveHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit : <br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/><br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |flattenHierarchy |Pour un dossier source nommé Dossier1 et structuré comme suit :<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;nom généré automatiquement pour Fichier2<br/><br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |
| false |mergeFiles |Pour un dossier source nommé Dossier1 et structuré comme suit :<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Fichier2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Sousdossier1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Fichier5<br/><br/>le dossier cible nommé Dossier1 est créé et structuré comme suit<br/><br/>Folder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Le contenu de Fichier1 + Fichier2 est fusionné dans un fichier avec le nom de fichier généré automatiquement. nom généré automatiquement pour Fichier1<br/><br/>Sous-dossier1, où Fichier3, Fichier4 et Fichier5 ne sont pas sélectionnés. |

## <a name="supported-file-and-compression-formats"></a>Formats de fichier et de compression pris en charge
Pour plus d’informations, voir [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md).

## <a name="json-examples-for-copying-data-to-and-from-data-lake-store"></a>Exemples JSON pour copier des données vers et depuisData Lake Store
Les exemples suivants fournissent des exemples de définitions JSON. Vous pouvez utiliser ces exemples de définitions pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Les exemples indiquent comment copier des données vers et depuis Data Lake Store et Azure Blob Storage. Toutefois, les données peuvent être copiées _directement_ à partir de n’importe quelle source, vers tout récepteur pris en charge. Pour plus d’informations, consultez la section « Banques de données et formats pris en charge » de l’article [Déplacer des données à l’aide de l’activité de copie](data-factory-data-movement-activities.md).

### <a name="example-copy-data-from-azure-blob-storage-to-azure-data-lake-store"></a>Exemple : copie de données du stockage Blob Azure vers Azure Data Lake Store
L’exemple de code dans cette section montre :

* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Un service lié de type [AzureDataLakeStore](#linked-service-properties).
* un [jeu de données](data-factory-create-datasets.md) d'entrée de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureDataLakeStore](#dataset-properties).
* Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) et [AzureDataLakeStoreSink](#copy-activity-properties).

L’exemple montre la copie des données de série horaire depuis un stockage d’objets blob vers Azure Data Lake Store toutes les heures.

**Service lié Azure Storage**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**Service lié Azure Data Lake Store**

```JSON
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

> [!NOTE]
> Pour les détails de configuration, consultez la section [Propriétés du service lié](#linked-service-properties).
>

**Jeu de données d'entrée d'objet Blob Azure**

Dans l’exemple suivant, les données sont récupérées à partir d'un nouvel objet Blob toutes les heures (`"frequency": "Hour", "interval": 1`). Le nom du chemin d'accès et du fichier de dossier pour l'objet blob sont évalués dynamiquement en fonction de l'heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois et le jour de début. Le nom de fichier utilise la partie heure de l’heure de début. Le paramètre `"external": true` informe le service Data Factory que cette table est externe à la Data Factory et non produite par une activité dans la Data Factory.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ]
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Jeu de données de sortie Azure Data Lake Store**

L’exemple suivant copie des données dans un magasin Data Lake Store. De nouvelles données sont copiées dans Data Lake Store toutes les heures.

```JSON
{
    "name": "AzureDataLakeStoreOutput",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Activité de copie dans un pipeline avec une source Blob et un récepteur Data Lake Store**

Dans l’exemple suivant, le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie. L’activité de copie est planifiée pour s’exécuter toutes les heures. Dans la définition JSON du pipeline, le type `source` est défini sur `BlobSource` et le type `sink` est défini sur `AzureDataLakeStoreSink`.

```json
{
    "name":"SamplePipeline",
    "properties":
    {
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":
        [
            {
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureDataLakeStoreOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "AzureDataLakeStoreSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

### <a name="example-copy-data-from-azure-data-lake-store-to-an-azure-blob"></a>Exemple : copie de données d’Azure Data Lake Store vers un objet blob Azure
L’exemple de code dans cette section montre :

* Un service lié de type [AzureDataLakeStore](#linked-service-properties).
* Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [AzureDataLakeStore](#dataset-properties).
* Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [AzureDataLakeStoreSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

Le code copie des données appartenant à une série horaire depuis un magasin Data Lake vers un objet blob Azure toutes les heures.

**Service lié Azure Data Lake Store**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        }
    }
}
```

> [!NOTE]
> Pour les détails de configuration, consultez la section [Propriétés du service lié](#linked-service-properties).
>

**Service lié Azure Storage**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Jeu de données Azure Data Lake de sortie**

Dans cet exemple, la définition de `"external"` sur `true` informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à la Data Factory et non produit par une activité dans la Data Factory.

```json
{
    "name": "AzureDataLakeStoreInput",
    "properties":
    {
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
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```
**Jeu de données de sortie d’objet Blob Azure**

Dans l’exemple suivant, les données sont écrites dans un nouvel objet Blob toutes les heures (`"frequency": "Hour", "interval": 1`). Le chemin d’accès du dossier pour l’objet blob est évalué dynamiquement en fonction de l’heure de début du segment en cours de traitement. Le chemin d’accès du dossier utilise l’année, le mois, le jour et la partie heure de l’heure de début.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Activité de copie dans un pipeline avec une source Azure Data Lake Store et un récepteur Blob**

Dans l’exemple suivant, le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie. L’activité de copie est planifiée pour s’exécuter toutes les heures. Dans la définition JSON du pipeline, le type `source` est défini sur `AzureDataLakeStoreSource` et le type `sink` est défini sur `BlobSink`.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[
            {
                "name": "AzureDakeLaketoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureDataLakeStoreInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureDataLakeStoreSource",
                    },
                    "sink": {
                        "type": "BlobSink"
                    }
                },
                "scheduler": {
                    "frequency": "Hour",
                    "interval": 1
                },
                "policy": {
                    "concurrency": 1,
                    "executionPriorityOrder": "OldestFirst",
                    "retry": 0,
                    "timeout": "01:00:00"
                }
            }
        ]
    }
}
```

Dans la définition d’activité de copie, vous pouvez également mapper des colonnes du jeu de données source aux colonnes du jeu de données récepteur. Pour plus d’informations, consultez [Mappage de colonnes de jeux de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) et les manières de les optimiser.