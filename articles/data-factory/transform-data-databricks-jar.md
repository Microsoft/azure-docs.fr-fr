---
title: Transformer des données avec Databricks Jar
description: Découvrez comment traiter ou transformer des données en exécutant une activité Databricks Jar.
services: data-factory
documentationcenter: ''
ms.assetid: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.author: abnarain
author: nabhishek
manager: shwang
ms.date: 03/15/2018
ms.openlocfilehash: 6b010000a674e351051c664dd5eeacd40e802439
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81414613"
---
# <a name="transform-data-by-running-a-jar-activity-in-azure-databricks"></a>Transformer des données en exécutant une activité Jar dans Azure Databricks
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

L’activité Azure Databricks Jar dans un [pipeline Data Factory](concepts-pipelines-activities.md) exécute un fichier Spark Jar dans votre cluster Azure Databricks. Cet article s’appuie sur l’article  [Activités de transformation des données](transform-data.md) , qui présente une vue d’ensemble de la transformation des données et les activités de transformation prises en charge. Azure Databricks est une plateforme gérée pour exécuter Apache Spark.

Pour une présentation de onze minutes et la démonstration de cette fonctionnalité, regardez la vidéo suivante :

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Friday/Execute-Jars-and-Python-scripts-on-Azure-Databricks-using-Data-Factory/player]

## <a name="databricks-jar-activity-definition"></a>Définition de l’activité Databricks Jar

Voici l’exemple de définition JSON d’une activité Databricks Jar :

```json
{
    "name": "SparkJarActivity",
    "type": "DatabricksSparkJar",
    "linkedServiceName": {
        "referenceName": "AzureDatabricks",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "mainClassName": "org.apache.spark.examples.SparkPi",
        "parameters": [ "10" ],
        "libraries": [
            {
                "jar": "dbfs:/docs/sparkpi.jar"
            }
        ]
    }
}

```

## <a name="databricks-jar-activity-properties"></a>Propriétés de l’activité Databricks Jar

Le tableau suivant décrit les propriétés JSON utilisées dans la définition JSON :

|Propriété|Description|Obligatoire|
|:--|---|:-:|
|name|Nom de l'activité dans le pipeline.|Oui|
|description|Texte décrivant l’activité.|Non|
|type|Pour l’activité Databricks Jar, le type d’activité est DatabricksSparkJar.|Oui|
|linkedServiceName|Nom du service lié Databricks sur lequel s’exécute l’activité Jar. Pour en savoir plus sur ce service lié, consultez l’article  [Services liés de calcul](compute-linked-services.md) .|Oui|
|mainClassName|Nom complet de la classe contenant la méthode principale à exécuter. Cette classe doit être contenue dans un fichier JAR fourni en tant que bibliothèque.|Oui|
|parameters|Paramètres qui sont transmis à la méthode principale.  C’est un tableau de chaînes.|Non|
|libraries|Liste de bibliothèques à installer sur le cluster qui exécute le travail. Il peut s’agir d’un tableau de < chaîne, objet >|Oui (au moins une contenant la méthode mainClassName)|

> [!NOTE]
> **Problème connu** : quand vous utilisez le même [cluster interactif](compute-linked-services.md#example---using-existing-interactive-cluster-in-databricks) pour exécuter simultanément plusieurs activités Databricks Jar (sans redémarrage du cluster), vous pouvez rencontrer un problème connu avec Databricks. En effet, dans ce cas, les paramètres de la première activité sont également utilisés par les activités suivantes. Des paramètres incorrects sont alors transmis aux travaux suivants. Pour limiter ce risque, utilisez plutôt un [cluster de travail](compute-linked-services.md#example---using-new-job-cluster-in-databricks). 

## <a name="supported-libraries-for-databricks-activities"></a>Bibliothèques prises en charge pour les activités Databricks

Dans la définition d’activité Databricks ci-dessus, vous précisez ces types de bibliothèques : *jar*, *egg*, *maven*, *pypi*,  *cran*.

```json
{
    "libraries": [
        {
            "jar": "dbfs:/mnt/libraries/library.jar"
        },
        {
            "egg": "dbfs:/mnt/libraries/library.egg"
        },
        {
            "maven": {
                "coordinates": "org.jsoup:jsoup:1.7.2",
                "exclusions": [ "slf4j:slf4j" ]
            }
        },
        {
            "pypi": {
                "package": "simplejson",
                "repo": "http://my-pypi-mirror.com"
            }
        },
        {
            "cran": {
                "package": "ada",
                "repo": "https://cran.us.r-project.org"
            }
        }
    ]
}

```

Pour plus d’informations, consultez la [documentation Databricks](https://docs.azuredatabricks.net/api/latest/libraries.html#managedlibrarieslibrary) pour les types de bibliothèques.

## <a name="how-to-upload-a-library-in-databricks"></a>Comment charger une bibliothèque dans Databricks

#### <a name="using-databricks-workspace-ui"></a>[Interface utilisateur de l’espace de travail Databricks](https://docs.azuredatabricks.net/user-guide/libraries.html#create-a-library)

Pour obtenir le chemin dbfs de la bibliothèque ajoutée par le biais de l’interface utilisateur, vous pouvez utiliser [CLI Databricks (installation)](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#install-the-cli). 

En général, les bibliothèques Jar sont stockées sous dbfs:/FileStore/jars lors de l’utilisation de l’interface utilisateur. Vous pouvez répertorier toutes les répertorier à l’aide de l’interface CLI : *databricks fs ls dbfs:/FileStore/job-jars* 



#### <a name="copy-library-using-databricks-cli"></a>[Copier une bibliothèque avec CLI Databricks](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#copy-a-file-to-dbfs)
Utilisez l’interface CLI Databricks [(étapes d’installation)](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#install-the-cli). 

Exemple - Copie de JAR dans dbfs: *dbfs cp SparkPi-assembly-0.1.jar dbfs:/docs/sparkpi.jar*
