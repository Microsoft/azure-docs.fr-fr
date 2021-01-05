---
title: Copier des données à partir de ServiceNow
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données de ServiceNow vers des banques de données réceptrices prises en charge.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 08/01/2019
ms.openlocfilehash: bc48f651a1adb099017e8f47d9fa6bcfa8078fa1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81415349"
---
# <a name="copy-data-from-servicenow-using-azure-data-factory"></a>Copier des données à partir de ServiceNow avec Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité de copie dans Azure Data Factory pour copier des données de ServiceNow. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur ServiceNow est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier les données depuis ServiceNow vers toute banque de données réceptrice prise en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Azure Data Factory fournit un pilote intégré qui permet la connexion. Vous n’avez donc pas besoin d’installer manuellement un pilote à l’aide de ce connecteur.

## <a name="getting-started"></a>Prise en main

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques du connecteur ServiceNow.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié ServiceNow sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **ServiceNow** | Oui |
| endpoint | Point de terminaison du serveur ServiceNow (`http://<instance>.service-now.com`).  | Oui |
| authenticationType | Type d’authentification à utiliser. <br/>Les valeurs autorisées sont les suivantes : **Basic**, **OAuth2** | Oui |
| username | Nom d’utilisateur utilisé pour la connexion au serveur ServiceNow pour l’authentification De base et OAuth2.  | Oui |
| mot de passe | Mot de passe correspondant au nom d’utilisateur pour l’authentification De base et OAuth2. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| clientId | ID client pour l’authentification OAuth2.  | Non |
| clientSecret | Secret client pour l’authentification OAuth2. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Non |
| useEncryptedEndpoints | Indique si les points de terminaison de la source de données sont chiffrés suivant le protocole HTTPS. La valeur par défaut est true.  | Non |
| useHostVerification | Indique si le nom d’hôte du certificat du serveur doit correspondre à celui du serveur en cas de connexion TLS. La valeur par défaut est true.  | Non |
| usePeerVerification | Indique s’il faut vérifier l’identité du serveur en cas de connexion TLS. La valeur par défaut est true.  | Non |

**Exemple :**

```json
{
    "name": "ServiceNowLinkedService",
    "properties": {
        "type": "ServiceNow",
        "typeProperties": {
            "endpoint" : "http://<instance>.service-now.com",
            "authenticationType" : "Basic",
            "username" : "<username>",
            "password": {
                 "type": "SecureString",
                 "value": "<password>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les [jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données ServiceNow.

Pour copier des données de ServiceNow, affectez la valeur **ServiceNowObject** à la propriété type du jeu de données. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur : **ServiceNowObject** | Oui |
| tableName | Nom de la table. | Non (si « query » dans la source de l’activité est spécifié) |

**Exemple**

```json
{
    "name": "ServiceNowDataset",
    "properties": {
        "type": "ServiceNowObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<ServiceNow linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source ServiceNow.

### <a name="servicenow-as-source"></a>ServiceNow en tant que source

Pour copier des données à partir de ServiceNow, définissez le type de source sur **ServiceNowSource** dans l’activité de copie. Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source d’activité de copie doit être définie sur : **ServiceNowSource** | Oui |
| query | Utiliser la requête SQL personnalisée pour lire les données. Par exemple : `"SELECT * FROM Actual.alm_asset"`. | Non (si « tableName » est spécifié dans dataset) |

Notez les points suivants au moment de spécifier le schéma et la colonne pour ServiceNow dans la requête, et **reportez-vous à la section [Conseils sur les performances](#performance-tips) pour en savoir plus sur l’implication des performances de copie**.

- **Schéma :** spécifiez le schéma avec la valeur `Actual` ou `Display` dans la requête ServiceNow, ce que vous pouvez considérer comme le paramètre de `sysparm_display_value` avec la valeur true ou false quand vous appelez les [API RESTful de ServiceNow](https://developer.servicenow.com/app.do#!/rest_api_doc?v=jakarta&id=r_AggregateAPI-GET). 
- **Colonne :** le nom de la colonne pour une valeur réelle sous le schéma `Actual` est `[column name]_value`, tandis que pour la valeur d’affichage, sous le schéma `Display`, le nom est `[column name]_display_value`. Le nom de colonne doit correspondre au schéma utilisé dans la requête.

**Exemple de requête :** 
`SELECT col_value FROM Actual.alm_asset` OR 
`SELECT col_display_value FROM Display.alm_asset`

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromServiceNow",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<ServiceNow input dataset name>",
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
                "type": "ServiceNowSource",
                "query": "SELECT * FROM Actual.alm_asset"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```
## <a name="performance-tips"></a>Conseils sur les performances

### <a name="schema-to-use"></a>Schéma à utiliser

ServiceNow possède deux schémas différents : **« Actual »** qui retourne des données réelles et **« Display »** qui retourne les valeurs d’affichage des données. 

Si votre requête contient un filtre, utilisez le schéma « Actual » pour bénéficier de meilleures performances de copie. Quand la requête porte sur le schéma « Actual », ServiceNow prend en charge le filtre en mode natif au moment d’extraire les données pour retourner uniquement le jeu de résultats filtré, alors qu’en interrogeant le schéma« Display », ADF récupère toutes les données et applique le filtre en interne.

### <a name="index"></a>Index

L’index de table ServiceNow peut contribuer à améliorer les performances des requêtes. Consultez [Create a table index](https://docs.servicenow.com/bundle/geneva-servicenow-platform/page/administer/table_administration/task/t_CreateCustomIndex.html).

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).
