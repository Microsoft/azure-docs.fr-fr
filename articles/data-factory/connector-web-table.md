---
title: Copier des données de Table web à l’aide d’Azure Data Factory
description: Découvrez le connecteur Table web d’Azure Data Factory, qui vous permet de copier des données d’une Table web vers des banques de données prises en charge par Data Factory en tant que récepteurs.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 08/01/2019
ms.author: jingwang
ms.openlocfilehash: 0eb4d37342685c13027a69bb6cb85f618fa63f20
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81410213"
---
# <a name="copy-data-from-web-table-by-using-azure-data-factory"></a>Copier des données de Table web à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-web-table-connector.md)
> * [Version actuelle](connector-web-table.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment utiliser l’activité de copie dans Azure Data Factory pour copier des données d’une base de données de Table web. Il s’appuie sur l’article [Vue d’ensemble de l’activité de copie](copy-activity-overview.md).

Les différences entre ce connecteur Table web, le [connecteur REST](connector-rest.md) et le [connecteur HTTP](connector-http.md) sont les suivantes :

- Le **connecteur Table web** extrait le contenu de tables d’une page web HTML.
- Le **connecteur REST** prend spécifiquement en charge la copie de données à partir d’API RESTful.
- Le **connecteur HTTP** est générique pour récupérer des données à partir de n’importe quel point de terminaison HTTP, par exemple pour télécharger un fichier. 

## <a name="supported-capabilities"></a>Fonctionnalités prises en charge

Ce connecteur de table web est pris en charge pour les activités suivantes :

- [Activité Copy](copy-activity-overview.md) avec [prise en charge de la matrice source/du récepteur](copy-activity-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)

Vous pouvez copier des données d’une base de données de Table web vers toute banque de données réceptrice prise en charge. Pour obtenir la liste des banques de données prises en charge en tant que sources ou récepteurs par l’activité de copie, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).

Plus précisément, ce connecteur Table web prend en charge l’**extraction de contenu de table d’une page HTML**.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce connecteur de table web, vous devez configurer un Runtime d’intégration autohébergé. Pour plus d’informations, consultez l’article [Runtime d’intégration autohébergé](create-self-hosted-integration-runtime.md).

## <a name="getting-started"></a>Prise en main

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Les sections suivantes fournissent des informations sur les propriétés utilisées pour définir les entités Data Factory spécifiques du connecteur Table web.

## <a name="linked-service-properties"></a>Propriétés du service lié

Les propriétés prises en charge pour le service lié Table web sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type doit être définie sur : **Web** |Oui |
| url | URL de la source web |Oui |
| authenticationType | Valeur autorisée : **Anonymous**. |Oui |
| connectVia | [Runtime d’intégration](concepts-integration-runtime.md) à utiliser pour la connexion à la banque de données. Un Runtime d’intégration autohébergé est nécessaire comme indiqué dans [Prérequis](#prerequisites). |Oui |

**Exemple :**

```json
{
    "name": "WebLinkedService",
    "properties": {
        "type": "Web",
        "typeProperties": {
            "url" : "https://en.wikipedia.org/wiki/",
            "authenticationType": "Anonymous"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article sur les [jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données de l’objet Table web.

Pour copier des données à partir de Table web, affectez la valeur **WebTable** à la propriété type du jeu de données. Les propriétés prises en charge sont les suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété type du jeu de données doit être définie sur : **WebTable** | Oui |
| path |URL relative de la ressource qui contient la table. |Non. Quand le chemin d’accès n’est pas spécifié, seule l’URL spécifiée dans la définition du service lié est utilisée. |
| index |Index de la table dans la ressource. Pour savoir comment obtenir l’index d’une table dans une page HTML, consultez la section [Obtenir l’index d’une table dans une page HTML](#get-index-of-a-table-in-an-html-page) . |Oui |

**Exemple :**

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Web linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source de Table web.

### <a name="web-table-as-source"></a>Table web en tant que source

Pour copier des données de Table web, définissez le type de source dans l’activité de copie sur **WebSource**. Aucune autre propriété n’est prise en charge.

**Exemple :**

```json
"activities":[
    {
        "name": "CopyFromWebTable",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Web table input dataset name>",
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
                "type": "WebSource"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="get-index-of-a-table-in-an-html-page"></a>Obtenir l’index d’une table dans une page HTML

Pour obtenir l’index d’une table que vous devez configurer dans les [propriétés du jeu de données](#dataset-properties), vous pouvez utiliser, par exemple, Excel 2016 comme suit :

1. Lancez **Excel 2016** et basculez vers l’onglet **Données**.
2. Cliquez sur **Nouvelle requête** dans la barre d’outils, pointez sur **À partir d’autres sources** et cliquez sur **À partir du web**.

    ![Menu Power Query](./media/copy-data-from-web-table/PowerQuery-Menu.png)
3. Dans la boîte de dialogue **À partir du web**, entrez l’**URL** que vous utiliseriez dans le service lié JSON (par exemple https://en.wikipedia.org/wiki/), ainsi que le chemin à spécifier pour le jeu de données (par exemple : AFI%27s_100_Years...100_Movies), puis cliquez sur **OK**.

    ![Boîte de dialogue À partir du web](./media/copy-data-from-web-table/FromWeb-DialogBox.png)

    URL utilisée dans cet exemple : https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies
4. Si la boîte de dialogue **Accéder au contenu web** apparaît, sélectionnez **l’URL** et **l’authentification** adéquates, puis cliquez sur **Se connecter**.

   ![Boîte de dialogue Accéder au contenu web](./media/copy-data-from-web-table/AccessWebContentDialog.png)
5. Cliquez sur un élément de **table** dans l’arborescence pour afficher le contenu de la table, puis sur le bouton **Modifier** du bas.  

   ![Boîte de dialogue Navigateur](./media/copy-data-from-web-table/Navigator-DialogBox.png)
6. Dans la fenêtre **Éditeur de requête**, cliquez sur **Éditeur avancé** dans la barre d’outils.

    ![Bouton Éditeur avancé](./media/copy-data-from-web-table/QueryEditor-AdvancedEditorButton.png)
7. Dans la boîte de dialogue Éditeur avancé, le numéro en regard de « Source » est l’index.

    ![Éditeur avancé - Index](./media/copy-data-from-web-table/AdvancedEditor-Index.png)

Si vous utilisez Excel 2013, utilisez [Microsoft Power Query pour Excel](https://www.microsoft.com/download/details.aspx?id=39379) pour obtenir l’index. Pour plus d’informations, consultez l’article [Se connecter à une page web](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8) . Les étapes sont identiques si vous utilisez [Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/).


## <a name="lookup-activity-properties"></a>Propriétés de l’activité Lookup

Pour en savoir plus sur les propriétés, consultez [Activité Lookup](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir la liste des banques de données prises en charge en tant que sources et récepteurs par l’activité de copie dans Azure Data Factory, consultez le tableau [banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).
