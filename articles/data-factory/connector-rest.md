---
title: Copier des données depuis et vers un point de terminaison REST à l’aide d’Azure Data Factory
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données d’une source REST locale ou dans le cloud vers des banques de données réceptrices prises en charge, ou d’une banque de données source prise en charge vers un récepteur REST.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 12/08/2020
ms.author: jingwang
ms.openlocfilehash: 1b3ab569666ea413ba36da0dc00f6c37336c4443
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96931299"
---
# <a name="copy-data-from-and-to-a-rest-endpoint-by-using-azure-data-factory"></a>Copier des données depuis et vers un point de terminaison REST à l’aide d’Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité Copy dans Azure Data Factory pour copier des données depuis et vers un point de terminaison REST. Il s’appuie sur l’article [Activité de copie dans Azure Data Factory](copy-activity-overview.md), qui constitue une présentation de l’activité de copie.

Les différences entre ce connecteur REST, un [connecteur HTTP](connector-http.md) et le [connecteur Table web](connector-web-table.md) sont les suivantes :

- Le **connecteur REST** prend spécifiquement en charge la copie de données à partir d’API RESTful. 
- Le **connecteur HTTP** est générique pour récupérer des données à partir de n’importe quel point de terminaison HTTP, par exemple pour télécharger un fichier. Avant que ce connecteur REST soit disponible, vous utiliserez peut-être un connecteur HTTP pour copier des données à partir d’une API RESTful, ce qui est pris en charge mais moins fonctionnel comparé au connecteur REST.
- Le **connecteur Table web** extrait le contenu de tables d’une page web HTML.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Vous pouvez copier des données d’une source REST vers toute banque de données réceptrice prise en charge. Vous pouvez également copier des données de n’importe quelle banque de données source prise en charge vers un récepteur REST. Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs pour l’activité de copie, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur REST générique prend en charge ce qui suit :

- La copie de données à partir d’un point de terminaison REST à l’aide des méthodes **GET** ou **POST** et la copie de données vers un point de terminaison REST à l’aide des méthodes **POST**, **PUT** ou **PATCH**.
- Copie de données avec une des authentifications suivantes : **Anonyme**, **De base**, **Principal de service AAD** et **Identités managées pour ressources Azure**.
- La **[Pagination](#pagination-support)** dans les API REST.
- Pour REST en tant que source, la copie de la réponse JSON REST [en l’état](#export-json-response-as-is) ou son analyse à l’aide d’une [mise en correspondance du schéma](copy-activity-schema-and-type-mapping.md#schema-mapping). Seule la charge utile de réponse dans **JSON** est prise en charge.

> [!TIP]
> Pour tester une requête pour l’extraction de données avant de configurer le connecteur REST dans Data Factory, obtenez des informations à partir de la spécification d’API sur les exigences d’en-tête et de corps. Vous pouvez vous servir d’outils tels que Postman ou un navigateur web pour valider.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Bien démarrer

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory propres au connecteur REST.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié REST sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** doit être définie sur **RestService**. | Oui |
| url | URL de base du service REST. | Oui |
| enableServerCertificateValidation | Indique s'il convient de valider ou non le certificat TLS/SSL côté serveur lors de la connexion au point de terminaison. | Non<br /> (la valeur par défaut est **true**) |
| authenticationType | Type d’authentification utilisé pour se connecter au service REST. Les valeurs autorisées sont **Anonymous**, **Basic**, **AadServicePrincipal** et **ManagedServiceIdentity**. Pour d’autres propriétés et exemples, voir les sections correspondantes ci-dessous. | Oui |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion au magasin de données. Pour plus d’informations, consultez la section [Conditions préalables](#prerequisites). À défaut de spécification, cette propriété utilise Azure Integration Runtime par défaut. |Non |

### <a name="use-basic-authentication"></a>Utiliser une authentification de base

Définissez la propriété **authenticationType** sur **De base**. Outre les propriétés génériques décrites dans la section précédente, spécifiez les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| userName | Nom d’utilisateur à utiliser pour accéder au point de terminaison REST. | Oui |
| mot de passe | Mot de passe de l’utilisateur (valeur **userName**). Vous pouvez marquer ce champ en tant que type **SecureString** pour le stocker de manière sécurisée dans Data Factory. Vous pouvez également [référencer un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |

**Exemple**

```json
{
    "name": "RESTLinkedService",
    "properties": {
        "type": "RestService",
        "typeProperties": {
            "authenticationType": "Basic",
            "url" : "<REST endpoint>",
            "userName": "<user name>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-aad-service-principal-authentication"></a>Utiliser une authentification de principal de service AAD

Définissez la propriété **authenticationType** sur **AadServicePrincipal**. Outre les propriétés génériques décrites dans la section précédente, spécifiez les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| servicePrincipalId | Spécifiez l’ID de l’application Azure Active Directory. | Oui |
| servicePrincipalKey | Spécifiez la clé de l’application Azure Active Directory. Marquez ce champ en tant que **SecureString** afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| tenant | Spécifiez les informations de locataire (nom de domaine ou ID de locataire) dans lesquels se trouve votre application. Récupérez-le en pointant la souris dans le coin supérieur droit du Portail Azure. | Oui |
| aadResourceId | Spécifiez la ressource AAD pour laquelle vous demandez une autorisation, par exemple `https://management.core.windows.net`.| Oui |
| azureCloudType | Pour l’authentification du principal du service, spécifiez le type d’environnement cloud Azure auquel votre application AAD est inscrite. <br/> Les valeurs autorisées sont **AzurePublic**, **AzureChina**, **AzureUsGovernment** et **AzureGermany**. Par défaut, l’environnement cloud de la fabrique de données est utilisé. | Non |

**Exemple**

```json
{
    "name": "RESTLinkedService",
    "properties": {
        "type": "RestService",
        "typeProperties": {
            "url": "<REST endpoint e.g. https://www.example.com/>",
            "authenticationType": "AadServicePrincipal",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "value": "<service principal key>",
                "type": "SecureString"
            },
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "aadResourceId": "<AAD resource URL e.g. https://management.core.windows.net>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

### <a name="use-managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Utiliser des identités managées afin d’authentifier les ressources Azure

Définissez la propriété **authenticationType** sur **ManagedServiceIdentity**. Outre les propriétés génériques décrites dans la section précédente, spécifiez les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| aadResourceId | Spécifiez la ressource AAD pour laquelle vous demandez une autorisation, par exemple `https://management.core.windows.net`.| Oui |

**Exemple**

```json
{
    "name": "RESTLinkedService",
    "properties": {
        "type": "RestService",
        "typeProperties": {
            "url": "<REST endpoint e.g. https://www.example.com/>",
            "authenticationType": "ManagedServiceIdentity",
            "aadResourceId": "<AAD resource URL e.g. https://management.core.windows.net>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Cette section contient la liste des propriétés prises en charge par le jeu de données REST. 

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez [Jeux de données et services liés](concepts-datasets-linked-services.md). 

Pour copier des données à partir de REST, les propriétés suivantes sont prises en charge :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** du jeu de données doit être définie sur **RestResource**. | Oui |
| relativeUrl | URL relative de la ressource qui contient les données. Quand cette propriété n’est pas spécifiée, seule l’URL indiquée dans la définition du service lié est utilisée. Le connecteur HTTP copie les données à partir de l’URL combinée : `[URL specified in linked service]/[relative URL specified in dataset]`. | Non |

Si vous avez défini `requestMethod`, `additionalHeaders`, `requestBody` et `paginationRules` dans le jeu de données, il reste pris en charge tel quel, mais nous vous suggérons d’utiliser le nouveau modèle dans l’activité à l’avenir.

**Exemple :**

```json
{
    "name": "RESTDataset",
    "properties": {
        "type": "RestResource",
        "typeProperties": {
            "relativeUrl": "<relative url>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<REST linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Cette section fournit la liste des propriétés prises en charge par la source et le récepteur REST.

Pour obtenir la liste complète des sections et des propriétés permettant de définir des activités, consultez [Pipelines](concepts-pipelines-activities.md). 

### <a name="rest-as-source"></a>REST en tant que source

Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** de la source d’activité de copie doit être définie sur **RestSource**. | Oui |
| requestMethod | Méthode HTTP. Les valeurs autorisées sont **GET** (par défaut) et **POST**. | Non |
| additionalHeaders | En-têtes de requête HTTP supplémentaires. | Non |
| requestBody | Corps de la requête HTTP. | Non |
| paginationRules | Règles de pagination pour composer des requêtes de page suivantes. Pour plus de détails, voir la section [Prise en charge la pagination](#pagination-support). | Non |
| httpRequestTimeout | Délai d’expiration (valeur **TimeSpan**) pour l’obtention d’une réponse par la requête HTTP. Cette valeur correspond au délai d’expiration pour l’obtention d’une réponse, et non au délai d’expiration pour la lecture des données de la réponse. La valeur par défaut est **00:01:40**.  | Non |
| requestInterval | Durée d’attente avant d’envoyer la requête de page suivante. La valeur par défaut est **00:00:01** |  Non |

>[!NOTE]
>Le connecteur REST ignore tout en-tête « accepter » spécifié dans `additionalHeaders`. Comme le connecteur REST ne prend en charge que la réponse dans JSON, il génère automatiquement un en-tête de `Accept: application/json`.

**Exemple 1 : Utilisation de la méthode Get avec la pagination**

```json
"activities":[
    {
        "name": "CopyFromREST",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<REST input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "RestSource",
                "additionalHeaders": {
                    "x-user-defined": "helloworld"
                },
                "paginationRules": {
                    "AbsoluteUrl": "$.paging.next"
                },
                "httpRequestTimeout": "00:01:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

**Exemple 2 : Utilisation de la méthode Post**

```json
"activities":[
    {
        "name": "CopyFromREST",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<REST input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "RestSource",
                "requestMethod": "Post",
                "requestBody": "<body for POST REST request>",
                "httpRequestTimeout": "00:01:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="rest-as-sink"></a>REST en tant que récepteur

Les propriétés prises en charge dans la section **sink** (récepteur) de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** du récepteur de l’activité Copy doit être définie sur **RestSink**. | Oui |
| requestMethod | Méthode HTTP. Les valeurs autorisées sont **POST** (valeur par défaut), **PUT** et **PATCH**. | Non |
| additionalHeaders | En-têtes de requête HTTP supplémentaires. | Non |
| httpRequestTimeout | Délai d’expiration (valeur **TimeSpan**) pour l’obtention d’une réponse par la requête HTTP. Cette valeur correspond au délai d’expiration pour l’obtention d’une réponse, et non au délai d’expiration pour l’écriture des données. La valeur par défaut est **00:01:40**.  | Non |
| requestInterval | Intervalle de temps en millisecondes entre les différentes demandes. La valeur de l’intervalle de demande doit être un nombre compris entre [10, 60000]. |  Non |
| httpCompressionType | Type de compression HTTP à utiliser lors de l’envoi de données avec un niveau de compression optimal. Les valeurs autorisées sont **none** et **gzip**. | Non |
| writeBatchSize | Nombre d’enregistrements à écrire dans le récepteur REST par lot. La valeur par défaut est 10 000. | Non |

Le connecteur REST en tant que récepteur fonctionne avec les API REST qui acceptent JSON. Les données sont envoyées au format JSON avec le modèle suivant. Si nécessaire, vous pouvez utiliser le [mappage de schéma](copy-activity-schema-and-type-mapping.md#schema-mapping) de l’activité de copie pour remodeler les données sources de façon à les rendre conformes à la charge utile attendue par l’API REST.

```json
[
    { <data object> },
    { <data object> },
    ...
]
```

**Exemple :**

```json
"activities":[
    {
        "name": "CopyToREST",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<REST output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "RestSink",
                "requestMethod": "POST",
                "httpRequestTimeout": "00:01:40",
                "requestInterval": 10,
                "writeBatchSize": 10000,
                "httpCompressionType": "none",
            },
        }
    }
]
```

## <a name="pagination-support"></a>Prise en charge la pagination

Normalement, quand vous copiez des données à partir de l’API REST, celles-ci limite la taille de charge utile de réponse d’une même demande à un nombre raisonnable. Pour retourner une grande quantité de données, elle fractionne le résultat en plusieurs pages et exige des appelants qu’ils envoient des demandes consécutives pour obtenir la page suivante du résultat. En règle générale, la requête d’une page est dynamique et composée par les informations retournées à partir de la réponse de la page précédente.

Ce connecteur REST générique prend en charge les modèles de pagination suivants : 

* URL absolue ou relative de la requête suivante = valeur de propriété dans le corps de la réponse en cours
* URL absolue ou relative de la requête suivante = valeur d’en-tête dans les en-têtes de la réponse en cours
* Paramètre de requête de la demande suivante = valeur de propriété dans le corps de la réponse en cours
* Paramètre de requête de la demande suivante = valeur d’en-tête dans les en-têtes de la réponse en cours
* En-tête de la requête suivante = valeur de propriété dans le corps de la réponse en cours
* En-tête de la requête suivante = valeur d’en-tête dans les en-têtes de la réponse en cours

Les **règles de pagination** sont définies en tant que dictionnaire dans un jeu de données qui contient une ou plusieurs paires clé-valeur respectant la casse. La configuration sera utilisée pour générer la requête à partir de la deuxième page. Le connecteur arrête les itérations quand il obtient le code d’état HTTP 204 (Aucun contenu) ou quand une expression JSONPath dans « paginationRules » retourne la valeur null.

**Clés prises en charge** dans les règles de pagination :

| Clé | Description |
|:--- |:--- |
| AbsoluteUrl | Indique l’URL pour l’émission de la requête suivante. Il peut s’agit d’une **URL absolue ou relative**. |
| QueryParameters.*request_query_parameter* OU QueryParameters[’request_query_parameter’] | « request_query_parameter » est défini par l’utilisateur et fait référence à un nom de paramètre de requête dans l’URL de la requête HTTP suivante. |
| Headers.*request_header* OR Headers[’request_header’] | « request_header » est défini par l’utilisateur et fait référence à un nom d’en-tête dans la requête HTTP suivante. |

**Valeurs prises en charge** dans les règles de pagination :

| Valeur | Description |
|:--- |:--- |
| Headers.*response_header* OR Headers[’response_header’] | « response_header » est défini par l’utilisateur et fait référence à un nom d’en-tête dans la réponse HTTP actuelle, dont la valeur sera utilisée pour émettre la prochaine requête. |
| Expression JSONPath commençant par « $ » (représentant la racine du corps de la réponse) | Le corps de la réponse ne doit contenir qu’un seul objet JSON. L’expression JSONPath doit retourner une seule valeur primitive qui sera utilisée pour émettre la requête suivante. |

**Exemple :**

L’API Graph Facebook retourne une réponse dans la structure suivante. Dans ce cas, l’URL de la page suivante est représentée dans **_paging.next_* _ :

```json
{
    "data": [
        {
            "created_time": "2017-12-12T14:12:20+0000",
            "name": "album1",
            "id": "1809938745705498_1809939942372045"
        },
        {
            "created_time": "2017-12-12T14:14:03+0000",
            "name": "album2",
            "id": "1809938745705498_1809941802371859"
        },
        {
            "created_time": "2017-12-12T14:14:11+0000",
            "name": "album3",
            "id": "1809938745705498_1809941879038518"
        }
    ],
    "paging": {
        "cursors": {
            "after": "MTAxNTExOTQ1MjAwNzI5NDE=",
            "before": "NDMyNzQyODI3OTQw"
        },
        "previous": "https://graph.facebook.com/me/albums?limit=25&before=NDMyNzQyODI3OTQw",
        "next": "https://graph.facebook.com/me/albums?limit=25&after=MTAxNTExOTQ1MjAwNzI5NDE="
    }
}
```

La configuration de la source de l’activité de copie REST correspondante, en particulier `paginationRules`, se présente comme suit :

```json
"typeProperties": {
    "source": {
        "type": "RestSource",
        "paginationRules": {
            "AbsoluteUrl": "$.paging.next"
        },
        ...
    },
    "sink": {
        "type": "<sink type>"
    }
}
```

## <a name="use-oauth"></a>Utiliser OAuth
Cette section décrit comment utiliser un modèle de solution pour copier des données à partir d’un connecteur REST dans Azure Data Lake Storage au format JSON à l’aide d’OAuth. 

### <a name="about-the-solution-template"></a>À propos du modèle de solution

Le modèle comporte deux activités :
- L’activité _ *Web** récupère le jeton du porteur, puis le transmet à l’activité Copy suivante en tant qu’autorisation.
- L’activité **Copier** copie les données de REST vers Azure Data Lake Storage.

Le modèle définit deux paramètres :
- **SinkContainer** est le chemin d’accès au dossier racine dans lequel les données sont copiées dans votre Azure Data Lake Storage. 
- **SinkDirectory** est le chemin du répertoire sous la racine où les données sont copiées dans votre Azure Data Lake Storage. 

### <a name="how-to-use-this-solution-template"></a>Utiliser ce modèle de solution

1. Accédez au modèle **Copier à partir de REST ou HTTP à l’aide de OAuth**. Créez une connexion pour la connexion source. 
    ![Créer des connexions](media/solution-template-copy-from-rest-or-http-using-oauth/source-connection.png)

    Voici les principales étapes pour les nouveaux paramètres de service lié (REST) :
    
     1. Sous **URL de base**, spécifiez le paramètre d’URL pour votre propre service REST source. 
     2. Pour **Type d’authentification**, choisissez *Anonyme*.
        ![Nouvelle connexion REST](media/solution-template-copy-from-rest-or-http-using-oauth/new-rest-connection.png)

2. Créez une connexion pour la connexion de destination.  
    ![Nouvelle connexion Gen2](media/solution-template-copy-from-rest-or-http-using-oauth/destination-connection.png)

3. Sélectionnez **Utiliser ce modèle**.
    ![Utiliser ce modèle](media/solution-template-copy-from-rest-or-http-using-oauth/use-this-template.png)

4. Vous pouvez voir le pipeline créé comme indiqué dans l’exemple suivant :  ![La capture d’écran montre le pipeline créé à partir du modèle.](media/solution-template-copy-from-rest-or-http-using-oauth/pipeline.png)

5. Sélectionnez l’activité **Web**. Dans **Paramètres**, spécifiez l’**URL**, la **méthode**, les **en-têtes** et le **corps** correspondants pour récupérer le jeton du porteur OAuth à partir de l’API de connexion du service à partir duquel vous souhaitez copier des données. L’espace réservé dans le modèle illustre un exemple de norme OAuth Azure Active Directory (AAD). Remarque : l’authentification AAD est prise en charge en mode natif par le connecteur REST, qui ici n’est qu’un exemple de flux OAuth. 

    | Propriété | Description |
    |:--- |:--- |:--- |
    | URL |Spécifiez l’URL à partir de laquelle récupérer le jeton du porteur OAuth. Dans l’exemple, c’est https://login.microsoftonline.com/microsoft.onmicrosoft.com/oauth2/token |. 
    | Méthode | Méthode HTTP. Les valeurs autorisées sont **Post** et **Get**. | 
    | headers | L’en-tête est défini par l’utilisateur, qui fait référence à un nom d’en-tête dans la requête HTTP. | 
    | body | Corps de la requête HTTP. | 

    ![Pipeline](media/solution-template-copy-from-rest-or-http-using-oauth/web-settings.png)

6. Dans l’activité **Copier les données**, sélectionnez l’onglet *Source*, vous pouvez voir que le jeton du porteur (access_token) récupéré à partir de l’étape précédente est transmis à l’activité Copier les données en tant qu’**Autorisation** sous les en-têtes supplémentaires. Confirmez les paramètres pour les propriétés suivantes avant de démarrer une exécution de pipeline.

    | Propriété | Description |
    |:--- |:--- |:--- | 
    | Méthode de requête | Méthode HTTP. Les valeurs autorisées sont **Get** (par défaut) et **Post**. | 
    | En-têtes supplémentaires | En-têtes de requête HTTP supplémentaires.| 

   ![Copier l’authentification source](media/solution-template-copy-from-rest-or-http-using-oauth/copy-data-settings.png)

7. Sélectionnez **Déboguer**, entrez les **Paramètres**, puis sélectionnez **Terminer**.
   ![Exécution du pipeline](media/solution-template-copy-from-rest-or-http-using-oauth/pipeline-run.png) 

8. Lorsque l’exécution du pipeline s’effectue correctement, vous pouvez voir le résultat similaire à l’exemple suivant : ![Résultat de l’exécution du pipeline](media/solution-template-copy-from-rest-or-http-using-oauth/run-result.png) 

9. Cliquez sur l’icône de sortie de WebActivity dans la colonne **Actions**, vous voyez l’access_token retourné par le service.

   ![Sortie de jeton](media/solution-template-copy-from-rest-or-http-using-oauth/token-output.png) 

10. Cliquez sur l’icône d’entrée de CopyActivity dans la colonne **Actions**, vous voyez que l’access_token récupéré par WebActivity est transmis à CopyActivity à des fins d’authentification. 

    ![Entrée de jeton](media/solution-template-copy-from-rest-or-http-using-oauth/token-input.png)
        
    >[!CAUTION] 
    >Pour éviter la journalisation des jetons en texte brut, activez la sortie sécurisée dans l’activité Web et l’entrée sécurisée dans l’activité de copie.


## <a name="export-json-response-as-is"></a>Exporter la réponse JSON en l’état

Vous pouvez utiliser ce connecteur REST pour exporter une réponse JSON d’API REST en l’état à différentes banques basées sur un fichier. Pour obtenir une telle copie indépendante du schéma, ignorez la section « structure » (également appelée *schéma*) dans le mappage de schéma et de jeu de données dans l’activité de copie.

## <a name="schema-mapping"></a>Mappage de schéma

Pour copier des données d’un point de terminaison REST vers un récepteur tabulaire, consultez [Mappage de schéma](copy-activity-schema-and-type-mapping.md#schema-mapping).

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir la liste des magasins de données pris en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez [Magasins de données et formats pris en charge](copy-activity-overview.md#supported-data-stores-and-formats).
