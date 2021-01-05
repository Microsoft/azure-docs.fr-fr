---
title: Copier des données de Xero à l’aide d’Azure Data Factory
description: Découvrez comment utiliser l’activité de copie dans un pipeline Azure Data Factory pour copier des données de Xero vers des banques de données réceptrices prises en charge.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/29/2020
ms.author: jingwang
ms.openlocfilehash: 342d0aabe2222393f33aa4ce93646da9f29cf1fb
ms.sourcegitcommit: dd45ae4fc54f8267cda2ddf4a92ccd123464d411
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92926459"
---
# <a name="copy-data-from-xero-using-azure-data-factory"></a>Copier des données de Xero à l’aide d’Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité de copie dans Azure Data Factory pour copier des données de Xero. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md).

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur Xero est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier les données de Xero vers toute banque de données réceptrice prise en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur Xero prend en charge ce qui suit :

- [Application privée](https://developer.xero.com/documentation/getting-started/getting-started-guide) Xero mais pas d’application publique.
- Toutes les tables Xero (points de terminaison d’API) à l’exception de « Reports ».
- Authentification OAuth 1.0 et OAuth 2.0.

Azure Data Factory fournit un pilote intégré qui permet la connexion. Vous n’avez donc pas besoin d’installer manuellement un pilote à l’aide de ce connecteur.

## <a name="getting-started"></a>Prise en main

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques du connecteur Xero.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Xero sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **Xero** | Oui |
| connectionProperties | Un groupe de propriétés qui définit la façon de se connecter à Xero. | Oui |
| **_Sous `connectionProperties`:_* _ | | |
| host | Le point de terminaison du serveur Xero (`api.xero.com`).  | Oui |
| authenticationType | Les valeurs autorisées sont `OAuth_2.0` et `OAuth_1.0`. | Oui |
| consumerKey | Clé de consommateur associée à l’application Xero. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| privateKey | La clé privée provenant du fichier .pem qui a été généré pour votre application privée Xero, consultez [Créer une paire de clés publique/privée](https://developer.xero.com/documentation/auth-and-limits/create-publicprivate-key). Remarque : pour _ *générer privatekey.pem avec numbits à 512* * à l’aide de `openssl genrsa -out privatekey.pem 512`, 1024 n’est pas pris en charge. Inclut tout le texte du fichier .pem, y compris les fins de ligne Unix (\n), voir l’exemple ci-dessous.<br/>Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| tenantId | L’ID de locataire associé à votre application Xero. S’applique à l’authentification OAuth 2.0.<br>Découvrez comment obtenir l’ID de locataire dans la section [Check the tenants you’re authorized to access](https://developer.xero.com/documentation/oauth2/auth-flow). | Oui pour l’authentification OAuth 2.0 |
| refreshToken | S’applique à l’authentification OAuth 2.0.<br/>Le jeton d’actualisation OAuth 2.0 est associé à l’application Xero et utilisé pour actualiser le jeton d’accès ; celui-ci expire après 30 minutes. Découvrez comment fonctionne le flux d’autorisation Xero et comment obtenir le jeton d'actualisation dans [cet article](https://developer.xero.com/documentation/oauth2/auth-flow). Pour obtenir un jeton d’actualisation, vous devez demander l’[étendue offline_access](https://developer.xero.com/documentation/oauth2/scopes). <br/>**Limitation**  : Notez que Xero réinitialise le jeton d’actualisation après utilisation. Pour une charge de travail opérationnalisée, avant chaque exécution de l’activité de copie, vous devez définir un jeton d’actualisation valide pour l’utilisation d’ADF.<br/>Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui pour l’authentification OAuth 2.0 |
| useEncryptedEndpoints | Indique si les points de terminaison de la source de données sont chiffrés suivant le protocole HTTPS. La valeur par défaut est true.  | Non |
| useHostVerification | Indique si le nom d’hôte est requis dans le certificat de serveur et doit correspondre au nom d’hôte du serveur lors de la connexion via le protocole TLS. La valeur par défaut est true.  | Non |
| usePeerVerification | Indique s’il faut vérifier l’identité du serveur en cas de connexion TLS. La valeur par défaut est true.  | Non |

**Exemple : Authentification OAuth 2.0**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "connectionProperties": { 
                "host": "api.xero.com",
                "authenticationType":"OAuth_2.0", 
                "consumerKey": {
                    "type": "SecureString",
                    "value": "<consumer key>"
                },
                "privateKey": {
                    "type": "SecureString",
                    "value": "<private key>"
                },
                "tenantId": "<tenant ID>", 
                "refreshToken": {
                    "type": "SecureString",
                    "value": "<refresh token>"
                }, 
                "useEncryptedEndpoints": true, 
                "useHostVerification": true, 
                "usePeerVerification": true
            }            
        }
    }
}
```

**Exemple : Authentification OAuth 1.0**

```json
{
    "name": "XeroLinkedService",
    "properties": {
        "type": "Xero",
        "typeProperties": {
            "connectionProperties": {
                "host": "api.xero.com", 
                "authenticationType":"OAuth_1.0", 
                "consumerKey": {
                    "type": "SecureString",
                    "value": "<consumer key>"
                },
                "privateKey": {
                    "type": "SecureString",
                    "value": "<private key>"
                }, 
                "useEncryptedEndpoints": true,
                "useHostVerification": true,
                "usePeerVerification": true
            }
        }
    }
}
```

**Exemple de valeur de clé privée :**

Inclut tout le texte du fichier .pem, y compris les fins de ligne Unix (\n).

```
"-----BEGIN RSA PRIVATE KEY-----\nMII***************************************************P\nbu****************************************************s\nU/****************************************************B\nA*****************************************************W\njH****************************************************e\nsx*****************************************************l\nq******************************************************X\nh*****************************************************i\nd*****************************************************s\nA*****************************************************dsfb\nN*****************************************************M\np*****************************************************Ly\nK*****************************************************Y=\n-----END RSA PRIVATE KEY-----"
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les [jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données Xero.

Pour copier des données de Xero, affectez la valeur **XeroObject** à la propriété de type du jeu de données. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur : **XeroObject** | Oui |
| tableName | Nom de la table. | Non (si « query » dans la source de l’activité est spécifié) |

**Exemple**

```json
{
    "name": "XeroDataset",
    "properties": {
        "type": "XeroObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Xero linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source Xero.

### <a name="xero-as-source"></a>Xero en tant que source

Pour copier des données de Xero, définissez le type de source dans l’activité de copie sur **XeroSource**. Les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type de la source d’activité de copie doit être définie sur : **XeroSource** | Oui |
| query | Utiliser la requête SQL personnalisée pour lire les données. Par exemple : `"SELECT * FROM Contacts"`. | Non (si « tableName » est spécifié dans dataset) |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromXero",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Xero input dataset name>",
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
                "type": "XeroSource",
                "query": "SELECT * FROM Contacts"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

Notez ce qui suit lors de la spécification de la requête Xero :

- Les tables comportant des éléments complexes sont réparties dans plusieurs tables. Par exemple, les transactions bancaires ont une structure de données complexe « LineItems », les données de transaction bancaire sont donc mappées à la table `Bank_Transaction` et `Bank_Transaction_Line_Items`, avec `Bank_Transaction_ID` en tant que clé étrangère pour les relier.

- Les données de Xero sont disponibles via les deux schémas : `Minimal` (par défaut) et `Complete`. Le schéma complet contient des tables d’appel requises qui nécessitent des données supplémentaires (par exemple, la colonne ID) avant d’effectuer la requête souhaitée.

Les tables suivantes contiennent les mêmes informations dans le schéma Minimal et le schéma Complet. Pour réduire le nombre d’appels d’API, utilisez un schéma Minimal (par défaut).

- Bank_Transactions
- Contact_Groups 
- Contacts 
- Contacts_Sales_Tracking_Categories 
- Contacts_Phones 
- Contacts_Addresses 
- Contacts_Purchases_Tracking_Categories 
- Credit_Notes 
- Credit_Notes_Allocations 
- Expense_Claims 
- Expense_Claim_Validation_Errors
- Factures 
- Invoices_Credit_Notes
- Invoices_ Prepayments 
- Invoices_Overpayments 
- Manual_Journals 
- Overpayments 
- Overpayments_Allocations 
- Prepayments 
- Prepayments_Allocations 
- Receipts 
- Receipt_Validation_Errors 
- Tracking_Categories

Les tables suivantes peuvent uniquement être interrogées avec un schéma complet :

- Complete.Bank_Transaction_Line_Items 
- Complete.Bank_Transaction_Line_Item_Tracking 
- Complete.Contact_Group_Contacts 
- Complete.Contacts_Contact_ Persons 
- Complete.Credit_Note_Line_Items 
- Complete.Credit_Notes_Line_Items_Tracking 
- Complete.Expense_Claim_ Payments 
- Complete.Expense_Claim_Receipts 
- Complete.Invoice_Line_Items 
- Complete.Invoices_Line_Items_Tracking
- Complete.Manual_Journal_Lines 
- Complete.Manual_Journal_Line_Tracking 
- Complete.Overpayment_Line_Items 
- Complete.Overpayment_Line_Items_Tracking 
- Complete.Prepayment_Line_Items 
- Complete.Prepayment_Line_Item_Tracking 
- Complete.Receipt_Line_Items 
- Complete.Receipt_Line_Item_Tracking 
- Complete.Tracking_Category_Options

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Étapes suivantes
Consultez les [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats) pour obtenir la liste des banques de données prises en charge par l’activité de copie.
