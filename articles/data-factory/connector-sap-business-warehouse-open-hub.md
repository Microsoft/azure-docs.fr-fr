---
title: Copie de données depuis SAP Business Warehouse par le biais d’Open Hub
description: Apprenez à utiliser l'activité de copie dans un pipeline Azure Data Factory pour copier des données de SAP Business Warehouse (BW) vers des banques de données réceptrices prises en charge via Open Hub.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 06/12/2020
ms.openlocfilehash: 930c7e7881a00cd0cb1f4abc6b219c0fbdeebac5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87533408"
---
# <a name="copy-data-from-sap-business-warehouse-via-open-hub-using-azure-data-factory"></a>Copier des données à partir de SAP Business Warehouse via Open Hub à l'aide d'Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article explique comment utiliser l'activité de copie d'Azure Data Factory pour copier des données à partir d'un SAP Business Warehouse (BW) via Open Hub. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md).

>[!TIP]
>Pour en savoir plus sur la prise en charge générale de l’intégration de données SAP par ADF, consultez le livre blanc [Intégration de données SAP à l’aide d’Azure Data Factory](https://github.com/Azure/Azure-DataFactory/blob/master/whitepaper/SAP%20Data%20Integration%20using%20Azure%20Data%20Factory.pdf) offrant une présentation détaillée sur chaque connecteur SAP, une comparaison et des conseils.

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur SAP Business Warehouse Open Hub est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier des données de SAP Business Warehouse vers une banque de données réceptrice prise en charge via Open Hub. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur SAP Business Warehouse Open Hub prend en charge ce qui suit :

- SAP Business Warehouse **version 7.01 ou ultérieure (dans une pile SAP Support Package Stack postérieure à 2015)** . SAP BW4/HANA n'est pas pris en charge par ce connecteur.
- Copie de données via la table OHD locale qui peut être DSO, InfoCube, MultiProvider, DataSource, etc.
- Copie de données en utilisant une authentification de base.
- Connexion à un serveur d’applications SAP ou un serveur de messagerie SAP.
- Récupération de données via RFC.

## <a name="sap-bw-open-hub-integration"></a>Intégration de SAP BW Open Hub 

Le service [SAP BW Open Hub](https://wiki.scn.sap.com/wiki/display/BI/Overview+of+Open+Hub+Service) permet d'extraire efficacement des données à partir de SAP BW. Le schéma suivant illustre l'un des flux typiques dont disposent les clients dans leur système SAP, et dans lequel les données circulent comme suit : SAP ECC -> PSA -> DSO -> Cube.

La destination SAP BW Open Hub (OHD) définit la cible vers laquelle les données SAP sont relayées. Tous les objets pris en charge par le processus de transfert de données (DTP) SAP peuvent être utilisés comme sources de données Open Hub, par exemple, DSO, InfoCube, DataSource, etc. Les types de destination Open Hub, où les données relayées sont stockées, peuvent être des tables de bases de données (locales ou distantes) et des fichiers plats. Ce connecteur SAP BW Open Hub prend en charge la copie de données de la table OHD locale dans BW. Si vous utilisez d'autres types, vous pouvez vous connecter directement à la base de données ou au système de fichiers à l'aide d'autres connecteurs.

![SAP BW Open Hub](./media/connector-sap-business-warehouse-open-hub/sap-bw-open-hub.png)

## <a name="delta-extraction-flow"></a>Flux d’extraction de delta

Le connecteur ADF SAP BW Open Hub offre deux propriétés facultatives, `excludeLastRequest` et `baseRequestId`, qui permettent de gérer le chargement de delta à partir d’Open Hub. 

- **excludeLastRequestId** : Indique s'il faut exclure les enregistrements de la dernière requête. La valeur par défaut est true. 
- **baseRequestId** : ID de requête pour le chargement delta. Une fois l’ID défini, seules les données dont l’ID de requête est supérieur à la valeur de cette propriété sont récupérées. 

En général, l’extraction de fournisseurs d’informations SAP vers Azure Data Factory (ADF) comprend 2 étapes : 

1. **Traitement de transfert de données SAP BW** Cette étape copie les données d’un fournisseur d’informations SAP BW vers une table SAP BW Open Hub. 

1. **Copie des données ADF** Au cours de cette étape, le connecteur ADF lit la table Open Hub. 

![Flux d’extraction de delta](media/connector-sap-business-warehouse-open-hub/delta-extraction-flow.png)

Dans la première étape, un traitement de transfert de données est exécuté. Chaque exécution crée un ID de demande SAP. L’ID de demande est stocké dans la table Open Hub, puis utilisé par le connecteur ADF pour identifier le delta. Les deux étapes s’exécutent de façon asynchrone : le traitement de transfert de données est déclenché par SAP, et la copie des données ADF via ADF. 

Par défaut, ADF ne lit pas le dernier delta à partir de la table Open Hub (l’option « exclude last request » et définie sur true). Ainsi, les données dans ADF ne sont pas synchronisées à 100 % avec les données de la table Open Hub (le dernier delta est manquant). Par contre, cette procédure garantit qu’aucune ligne n’est perdue en raison de l’extraction asynchrone. Cela fonctionne bien, même si ADF lit la table Open Hub alors que le traitement de transfert de données est toujours en train d’écrire dans la même table. 

Vous stockez généralement l’ID de requête max copié lors de la dernière exécution par ADF dans une banque de données intermédiaires (telle que le Stockage Blob Azure dans le diagramme ci-dessus). Par conséquent, la même demande n’est pas lue une deuxième fois par ADF lors de l’exécution suivante. Entre-temps, notez que les données ne sont pas automatiquement supprimées de la table Open Hub.

Pour un traitement de delta approprié, il ne peut pas y avoir d’ID de demande provenant de traitements de transfert de données différents dans la même table Open Hub. Vous ne devez donc pas créer plus d’un traitement de transfert de données pour chaque destination Open Hub. Si vous avec besoin d’extractions complète et de delta du même fournisseur d’informations, vous devez créer deux destinations Open Hub pour le même fournisseur d’informations. 

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce connecteur SAP Business Warehouse Open Hub, vous devez suivre les instructions ci-dessous :

- Configurez un runtime d'intégration auto-hébergé avec la version 3.13 ou ultérieure. Pour plus d’informations, consultez l’article [Runtime d’intégration autohébergé](create-self-hosted-integration-runtime.md).

- Téléchargez la version **64 bits de [SAP .NET Connector 3.0](https://support.sap.com/en/product/connectors/msnet.html)** à partir du site web de SAP et installez-la sur la machine dotée du runtime d'intégration auto-hébergé. Lors de l'installation, dans la fenêtre des étapes de configuration facultatives, sélectionnez l'option **Installer les assemblys dans le GAC**, comme illustré ci-dessous. 

    ![Installer SAP .NET Connector](./media/connector-sap-business-warehouse-open-hub/install-sap-dotnet-connector.png)

- L'utilisateur SAP utilisé dans le connecteur Data Factory BW doit disposer des autorisations suivantes : 

    - Autorisation relative à RFC et SAP BW. 
    - Autorisations relatives à l'activité « Exécuter » de l'objet d'autorisation « S_SDSAUTH ».

- Créez le type de destination Open Hub en le définissant sur **Table de base de données** et en cochant l'option « Clé technique ».  Il est également recommandé de ne pas cocher la case Suppression des données de la table, bien que cela ne soit pas obligatoire. Utilisez le DTP (exécution directe ou intégration dans la chaîne de processus existante) pour transférer les données de l'objet source (tel que le cube) que vous avez choisi vers la table de destination Open Hub.

## <a name="getting-started"></a>Prise en main

> [!TIP]
>
> Pour une procédure pas à pas décrivant l’utilisation du connecteur SAP BW Open Hub, voir [Charger des données depuis SAP Business Warehouse à l’aide d’Azure Data Factory](load-sap-bw-data.md).

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations détaillées sur les propriétés utilisées pour définir les entités Data Factory spécifiques au connecteur SAP Business Warehouse Open Hub.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié SAP Business Warehouse Open Hub sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **SapOpenHub** | Oui |
| server | Nom du serveur sur lequel réside l’instance SAP BW. | Oui |
| systemNumber | Numéro de système du système SAP BW.<br/>Valeur autorisée : nombre décimal à deux chiffres représenté sous forme de chaîne. | Oui |
| messageServer | Nom d’hôte du serveur de messagerie SAP.<br/>Permet de se connecter à un serveur de messagerie SAP. | Non |
| messageServerService | Nom de service ou numéro de port du serveur de messagerie.<br/>Permet de se connecter à un serveur de messagerie SAP. | Non |
| systemId | ID du système SAP où se trouve la table.<br/>Permet de se connecter à un serveur de messagerie SAP. | Non |
| logonGroup | Groupe de connexion du système SAP.<br/>Permet de se connecter à un serveur de messagerie SAP. | Non |
| clientId | ID client du client dans le système SAP W.<br/>Valeur autorisée : nombre décimal à trois chiffres représenté sous forme de chaîne. | Oui |
| langage | Langue utilisée par le système SAP. | Non (la valeur par défaut est **EN**)|
| userName | Nom de l’utilisateur ayant accès au serveur SAP. | Oui |
| mot de passe | Mot de passe pour l’utilisateur. Marquez ce champ en tant que SecureString afin de le stocker en toute sécurité dans Data Factory, ou [référencez un secret stocké dans Azure Key Vault](store-credentials-in-key-vault.md). | Oui |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion à la banque de données. Un Runtime d’intégration autohébergé est nécessaire comme indiqué dans [Prérequis](#prerequisites). |Oui |

**Exemple :**

```json
{
    "name": "SapBwOpenHubLinkedService",
    "properties": {
        "type": "SapOpenHub",
        "typeProperties": {
            "server": "<server name>",
            "systemNumber": "<system number>",
            "clientId": "<client id>",
            "userName": "<SAP user>",
            "password": {
                "type": "SecureString",
                "value": "<Password for SAP user>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données SAP BW Open Hub.

Pour copier des données depuis et vers SAP BW Open Hub, définissez la propriété type du jeu de données sur **SapOpenHubTable**. Les propriétés suivantes sont prises en charge.

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur **SapOpenHubTable**.  | Oui |
| openHubDestinationName | Nom de la destination Open Hub à partir de laquelle les données doivent être copiées. | Oui |

Si vous avez défini `excludeLastRequest` et `baseRequestId` dans le jeu de données, il reste pris en charge tel quel, mais nous vous suggérons d’utiliser le nouveau modèle dans la source d’activité à l’avenir.

**Exemple :**

```json
{
    "name": "SAPBWOpenHubDataset",
    "properties": {
        "type": "SapOpenHubTable",
        "typeProperties": {
            "openHubDestinationName": "<open hub destination name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<SAP BW Open Hub linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source SAP BW Open Hub.

### <a name="sap-bw-open-hub-as-source"></a>SAP BW Open Hub en tant que source

Pour copier des données à partir de SAP BW Open Hub, les propriétés prises en charge dans la section **source** de l’activité de copie sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété de **type** de la source d’activité de copie doit être définie sur **SapOpenHubSource**. | Oui |
| excludeLastRequest | Indique s'il faut exclure les enregistrements de la dernière requête. | Non (la valeur par défaut est **true**) |
| baseRequestId | ID de requête pour le chargement delta. Une fois ceci défini, seules les données dont le requestId est **supérieur à** la valeur de cette propriété sont récupérées.  | Non |

>[!TIP]
>Si votre table Open Hub ne contient que les données générées par un seul ID de requête, par exemple, vous exécutez toujours un plein chargement et écrasez les données existantes dans la table, ou si vous n'exécutez le DTP qu'une fois à des fins de test, n'oubliez pas de décocher l'option « excludeLastRequest » afin de copier les données.

Pour accélérer le chargement des données, vous pouvez définir [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) sur l’activité de copie pour charger des données à partir du hub ouvert SAP BW en parallèle. Par exemple, si vous définissez `parallelCopies` sur quatre, Data Factory exécute simultanément quatre appels RFC et chaque appel RFC récupère une partie des données de votre table de hubs ouverts SAP BW partitionnée par l’ID de la requête de PAO et l’ID du package. Cela s’applique lorsque le nombre d’ID de la requête de PAO unique et d’ID de package est supérieur à la valeur de `parallelCopies`. Lors de la copie de données dans un magasin de données basé sur des fichiers, il est également recommandé de les écrire dans un dossier sous forme fichiers multiples (spécifiez uniquement le nom du dossier). Ceci offre de meilleures performances qu’une écriture dans un fichier unique.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromSAPBWOpenHub",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<SAP BW Open Hub input dataset name>",
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
                "type": "SapOpenHubSource",
                "excludeLastRequest": true
            },
            "sink": {
                "type": "<sink type>"
            },
            "parallelCopies": 4
        }
    }
]
```

## <a name="data-type-mapping-for-sap-bw-open-hub"></a>Mappage de type de données pour SAP BW Open Hub

Lors de la copie de données à partir de SAP BW Open Hub, les mappages suivants sont utilisés entre les types de données SAP BW et les types de données intermédiaires d'Azure Data Factory. Pour découvrir comment l’activité de copie mappe le schéma et le type de données la source au récepteur, voir [Mappages de schémas et de types de données](copy-activity-schema-and-type-mapping.md).

| Type SAP ABAP | Type de données intermédiaires de Data Factory |
|:--- |:--- |
| C (Chaîne) | String |
| I (entier) | Int32 |
| F (Float) | Double |
| D (Date) | String |
| T (Heure) | String |
| P (« packed » DCB, Devise, Décimal, Qté) | Decimal |
| N (Numc) | String |
| X (données binaires et brutes) | String |

## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="troubleshooting-tips"></a>Conseils de dépannage

**Symptômes :** Si vous exécutez SAP BW sur HANA et que seul un sous-ensemble de données est copié à l’aide de l’activité de copie ADF (1 million de lignes), la cause possible est que vous avez activé l’option « Exécution SAP HANA » dans votre DTP, auquel cas ADF peut récupérer seulement le premier lot de données.

**Résolution :** Désactivez l’option « Exécution SAP HANA » dans DTP, retraitez les données, puis réessayez d’exécuter l’activité de copie.

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).
