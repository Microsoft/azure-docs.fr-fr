---
title: Activité web dans Azure Data Factory
description: Découvrez comment utiliser Activité Web, l’une des activités de flux de contrôle prises en charge par Azure Data Factory, pour appeler un point de terminaison REST à partir d’un pipeline.
services: data-factory
documentationcenter: ''
author: dcstwh
ms.author: weetok
manager: jroth
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 12/19/2018
ms.openlocfilehash: fbe37152f4ff1ce24754bc2d7b968c8e1c76ca10
ms.sourcegitcommit: ea17e3a6219f0f01330cf7610e54f033a394b459
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97387715"
---
# <a name="web-activity-in-azure-data-factory"></a>Activité Web dans Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]


Une activité web peut être utilisée pour appeler un point de terminaison REST personnalisé à partir d’un pipeline Data Factory. Vous pouvez transmettre des jeux de données et des services liés que l’activité peut utiliser et auxquels elle peut accéder.

> [!NOTE]
> L’activité web est également prise en charge pour appeler des URL hébergées sur un réseau virtuel privé en tirant parti du runtime d’intégration auto-hébergé. Le runtime d’intégration doit pouvoir visualiser le point de terminaison de l’URL. 

> [!NOTE]
> La taille maximale de la charge utile de la réponse en sortie est de 4 Mo.  

## <a name="syntax"></a>Syntaxe

```json
{
   "name":"MyWebActivity",
   "type":"WebActivity",
   "typeProperties":{
      "method":"Post",
      "url":"<URLEndpoint>",
      "connectVia": {
          "referenceName": "<integrationRuntimeName>",
          "type": "IntegrationRuntimeReference"
      }
      "headers":{
         "Content-Type":"application/json"
      },
      "authentication":{
         "type":"ClientCertificate",
         "pfx":"****",
         "password":"****"
      },
      "datasets":[
         {
            "referenceName":"<ConsumedDatasetName>",
            "type":"DatasetReference",
            "parameters":{
               ...
            }
         }
      ],
      "linkedServices":[
         {
            "referenceName":"<ConsumedLinkedServiceName>",
            "type":"LinkedServiceReference"
         }
      ]
   }
}

```

## <a name="type-properties"></a>Propriétés type

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | --------
name | Nom de l’activité web | String | Oui
type | Doit avoir la valeur **WebActivity**. | String | Oui
method | Méthode d’API REST pour le point de terminaison cible. | Chaîne. <br/><br/>Types pris en charge : « GET », « POST », « PUT » | Oui
url | Point de terminaison cible et chemin d’accès | Chaîne (ou expression avec resultType de chaîne). Si elle ne reçoit pas de réponse du point de terminaison, l’activité expire à 1 minute avec une erreur. | Oui
headers | En-têtes envoyés à la demande. Par exemple, pour définir la langue et le type sur une requête : `"headers" : { "Accept-Language": "en-us", "Content-Type": "application/json" }`. | Chaîne (ou expression avec resultType de chaîne) | Oui, l’en-tête Content-type est obligatoire. `"headers":{ "Content-Type":"application/json"}`
body | Représente la charge utile envoyée au point de terminaison.  | Chaîne (ou expression avec resultType de chaîne). <br/><br/>Voir le schéma de la charge utile de demande dans la section [Schéma de la charge utile](#request-payload-schema). | Obligatoire pour les méthodes POST/PUT.
Authentification | Méthode d’authentification utilisée pour appeler le point de terminaison. Les types pris en charge sont « De base » ou « ClientCertificate ». Pour en savoir plus, voir la section [Authentification](#authentication). Si l’authentification n’est pas obligatoire, excluez cette propriété. | Chaîne (ou expression avec resultType de chaîne) | Non
jeux de données | Liste des jeux de données transmis au point de terminaison. | Tableau de références de jeu de données. Peut être un tableau vide. | Oui
linkedServices | Liste des services liés transmise au point de terminaison. | Tableau des références de service lié. Peut être un tableau vide. | Oui
connectVia | Le [runtime d’intégration](./concepts-integration-runtime.md) à utiliser pour se connecter à la banque de données. Vous pouvez utiliser le runtime d’intégration Azure ou un runtime d’intégration auto-hébergé (si votre banque de données se trouve sur un réseau privé). Si cette propriété n’est pas spécifiée, le service utilise le runtime d’intégration Azure par défaut. | Référence du runtime d’intégration. | Non 

> [!NOTE]
> Les points de terminaison REST que l’activité web appelle doivent retourner une réponse de type JSON. Si elle ne reçoit pas de réponse du point de terminaison, l’activité expire à 1 minute avec une erreur.

Le tableau suivant affiche la configuration requise pour le contenu JSON :

| Type de valeur | Corps de la demande | Response body |
|---|---|---|
|Objet JSON | Pris en charge | Prise en charge |
|Tableau JSON | Prise en charge <br/>(À l’heure actuelle, les tableaux JSON ne fonctionnent pas en raison d’un bogue. Un correctif est en cours.) | Non pris en charge |
| Valeur JSON | Prise en charge | Non pris en charge |
| Type non-JSON | Non pris en charge | Non pris en charge |
||||

## <a name="authentication"></a>Authentification

Voici les types d’authentification pris en charge dans l’activité web.

### <a name="none"></a>None

Si l’authentification n’est pas requise, n’incluez pas la propriété « authentication ».

### <a name="basic"></a>De base

Spécifiez le nom d’utilisateur et le mot de passe à utiliser avec l’authentification de base.

```json
"authentication":{
   "type":"Basic",
   "username":"****",
   "password":"****"
}
```

### <a name="client-certificate"></a>Certificat client

Spécifiez le contenu encodé en Base64 d’un fichier PFX et le mot de passe.

```json
"authentication":{
   "type":"ClientCertificate",
   "pfx":"****",
   "password":"****"
}
```

### <a name="managed-identity"></a>Identité managée

Spécifiez l’uri de ressource pour lequel le jeton d’accès sera demandé à l’aide de l’identité managée pour la fabrique de données. Pour appeler l’API Gestion des ressources Azure, utilisez `https://management.azure.com/`. Pour plus d’informations sur le fonctionnement des identités managées, consultez la [page de vue d'ensemble des identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

```json
"authentication": {
    "type": "MSI",
    "resource": "https://management.azure.com/"
}
```

> [!NOTE]
> Si votre fabrique de données est configurée avec un référentiel git, vous devez stocker vos informations d’identification dans Azure Key Vault pour utiliser l’authentification de base ou de certificat client. Azure Data Factory ne stocke pas les mots de passe dans Git.

## <a name="request-payload-schema"></a>Schéma de charge utile de demande
Lorsque vous utilisez la méthode POST/PUT, la propriété body représente la charge utile envoyée au point de terminaison. Vous pouvez transmettre des services liés et des jeux de données dans la charge utile. Voici le schéma de la charge utile :

```json
{
    "body": {
        "myMessage": "Sample",
        "datasets": [{
            "name": "MyDataset1",
            "properties": {
                ...
            }
        }],
        "linkedServices": [{
            "name": "MyStorageLinkedService1",
            "properties": {
                ...
            }
        }]
    }
}
```

## <a name="example"></a>Exemple
Dans cet exemple, l’activité web dans le pipeline appelle un point de terminaison REST. Elle transmet un service lié Azure SQL et un jeu de données Azure SQL au point de terminaison. Le point de terminaison REST utilise la chaîne de connexion Azure SQL pour se connecter au serveur SQL logique et retourne le nom de l’instance du serveur SQL.

### <a name="pipeline-definition"></a>Définition de pipeline

```json
{
    "name": "<MyWebActivityPipeline>",
    "properties": {
        "activities": [
            {
                "name": "<MyWebActivity>",
                "type": "WebActivity",
                "typeProperties": {
                    "method": "Post",
                    "url": "@pipeline().parameters.url",
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "authentication": {
                        "type": "ClientCertificate",
                        "pfx": "*****",
                        "password": "*****"
                    },
                    "datasets": [
                        {
                            "referenceName": "MySQLDataset",
                            "type": "DatasetReference",
                            "parameters": {
                                "SqlTableName": "@pipeline().parameters.sqlTableName"
                            }
                        }
                    ],
                    "linkedServices": [
                        {
                            "referenceName": "SqlLinkedService",
                            "type": "LinkedServiceReference"
                        }
                    ]
                }
            }
        ],
        "parameters": {
            "sqlTableName": {
                "type": "String"
            },
            "url": {
                "type": "String"
            }
        }
    }
}

```

### <a name="pipeline-parameter-values"></a>Valeurs de paramètre de pipeline

```json
{
    "sqlTableName": "department",
    "url": "https://adftes.azurewebsites.net/api/execute/running"
}

```

### <a name="web-service-endpoint-code"></a>Code de point de terminaison de service web

```csharp

[HttpPost]
public HttpResponseMessage Execute(JObject payload)
{
    Trace.TraceInformation("Start Execute");

    JObject result = new JObject();
    result.Add("status", "complete");

    JArray datasets = payload.GetValue("datasets") as JArray;
    result.Add("sinktable", datasets[0]["properties"]["typeProperties"]["tableName"].ToString());

    JArray linkedServices = payload.GetValue("linkedServices") as JArray;
    string connString = linkedServices[0]["properties"]["typeProperties"]["connectionString"].ToString();

    System.Data.SqlClient.SqlConnection sqlConn = new System.Data.SqlClient.SqlConnection(connString);

    result.Add("sinkServer", sqlConn.DataSource);

    Trace.TraceInformation("Stop Execute");

    return this.Request.CreateResponse(HttpStatusCode.OK, result);
}

```

## <a name="next-steps"></a>Étapes suivantes
Consultez les autres activités de flux de contrôle prises en charge par Data Factory :

- [Activité d’exécution du pipeline](control-flow-execute-pipeline-activity.md)
- [Pour chaque activité](control-flow-for-each-activity.md)
- [Activité d’obtention des métadonnées](control-flow-get-metadata-activity.md)
- [Activité de recherche](control-flow-lookup-activity.md)
