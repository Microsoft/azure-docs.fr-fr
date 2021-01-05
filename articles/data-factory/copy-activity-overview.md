---
title: Activité de copie dans Azure Data Factory
description: En savoir plus sur l’activité de copie dans Azure Data Factory. Vous pouvez l’utiliser pour copier des données à partir d’un magasin de données source pris en charge vers un magasin de données de récepteur pris en charge.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/12/2020
ms.author: jingwang
ms.openlocfilehash: 0b10a4de78c44e4c0a113a1f1a46c316b13a1f78
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96902156"
---
# <a name="copy-activity-in-azure-data-factory"></a>Activité de copie dans Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version de Data Factory que vous utilisez :"]
> * [Version 1](v1/data-factory-data-movement-activities.md)
> * [Version actuelle](copy-activity-overview.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Dans Azure Data Factory, vous pouvez utiliser l’activité de copie pour copier des données entre des banques de données locales et dans cloud. Une fois que vous avez copié les données, vous pouvez utiliser d’autres activités pour les transformer et les analyser ultérieurement. Vous pouvez également utiliser l’activité de copie pour publier les résultats de transformation et d’analyse pour l’aide à la décision (BI) et l’utilisation d’application.

![Rôle de l’activité de copie](media/copy-activity-overview/copy-activity.png)

L’activité de copie est exécutée sur un [runtime d’intégration](concepts-integration-runtime.md). Vous pouvez utiliser différents types de runtimes d’intégration pour différents scénarios de copie de données :

* Lorsque vous copiez des données entre deux magasins de données accessibles publiquement via Internet à partir de n’importe quelle adresse IP, vous pouvez utiliser le runtime d’intégration Azure pour l’activité de copie. Ce runtime d’intégration est sécurisé, fiable, évolutif et [disponible dans le monde entier](concepts-integration-runtime.md#integration-runtime-location).
* Lorsque vous copiez des données vers et à partir de banques de données situées localement ou dans un réseau avec contrôle d’accès (par exemple, un réseau virtuel Azure), vous devez configurer un runtime d’intégration auto-hébergé.

Un runtime d’intégration doit être associé à chaque magasin de données source et récepteur. Pour plus d’informations sur la façon dont l’activité de copie détermine le runtime d’intégration à utiliser, consultez [Choix du runtime d’intégration](concepts-integration-runtime.md#determining-which-ir-to-use).

Pour copier des données d’une source vers un récepteur, le service qui exécute l’activité de copie effectue les étapes suivantes :

1. Lit les données d’une banque de données source.
2. Effectue les opérations de sérialisation/désérialisation, de compression/décompression, de mappage de colonnes, et ainsi de suite. Il effectue ces opérations en se basant sur les configurations du jeu de données d’entrée, du jeu de données de sortie et de l’activité de copie.
3. Écrit les données dans la banque de données réceptrice/de destination.

![Vue d’ensemble de l’activité de copie](media/copy-activity-overview/copy-activity-overview.png)

## <a name="supported-data-stores-and-formats"></a>Banques de données et formats pris en charge

[!INCLUDE [data-factory-v2-supported-data-stores](../../includes/data-factory-v2-supported-data-stores.md)]

### <a name="supported-file-formats"></a>Formats de fichiers pris en charge

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Vous pouvez utiliser l’activité de copie pour copier des fichiers en l'état entre deux banques de données de fichiers, auquel cas les données sont copiées efficacement sans sérialisation ou désérialisation. En outre, vous avez la possibilité d'analyser ou de générer des fichiers d’un format donné. Vous pouvez notamment effectuer ce qui suit :

* Copier des données à partir d'une base de données SQL Server et écrire dans Azure Data Lake Storage Gen2 au format Parquet.
* Copier des fichiers au format texte (CSV) à partir d’un système de fichiers local et les écrire dans le stockage d’objets BLOB Azure au format Avro.
* Copier des fichiers compressés à partir d’un système de fichiers local, les décompresser à la volée et écrire les fichiers extraits dans Azure Data Lake Storage Gen2.
* Copier des données au format de texte compressé Gzip (CSV) à partir du stockage Blob Azure et les écrire dans Azure SQL Database.
* Beaucoup d’autres activités qui nécessitent la sérialisation/désérialisation ou la compression/décompression.

## <a name="supported-regions"></a>Régions prises en charge

Le service qui permet l’activité de copie est disponible mondialement, dans les régions et zones géographiques répertoriées dans [Emplacements du runtime d’intégration Azure](concepts-integration-runtime.md#integration-runtime-location). La topologie globalement disponible garantit le déplacement efficace des données en évitant généralement les sauts entre régions. Consultez [Produits par région](https://azure.microsoft.com/regions/#services) pour vérifier la disponibilité de Data Factory et le déplacement des données dans une région spécifique.

## <a name="configuration"></a>Configuration

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

En règle générale, pour utiliser l’activité de copie dans Azure Data Factory, vous devez :

1. **Créer des services liés pour le magasin de données source et le magasin de données récepteur.** Vous trouverez la liste des connecteurs pris en charge dans la section [Magasins de données et formats pris en charge](#supported-data-stores-and-formats) dans cet article. Reportez-vous à la section « Propriétés du service lié » de l’article relatif au connecteur pour obtenir des informations sur la configuration et les propriétés prises en charge. 
2. **Créer des jeux de données pour la source et le récepteur.** Pour plus d’informations sur la configuration et les propriétés prises en charge, reportez-vous aux sections « Propriétés du jeu données » des articles relatifs au connecteur source et récepteur.
3. **Créer un pipeline avec l’activité de copie.** La section suivante fournit un exemple.

### <a name="syntax"></a>Syntaxe

Le modèle suivant d’activité de copie contient une liste complète des propriétés prises en charge. Spécifiez celles qui correspondent à votre scénario.

```json
"activities":[
    {
        "name": "CopyActivityTemplate",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<source dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<sink dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>",
                <properties>
            },
            "sink": {
                "type": "<sink type>"
                <properties>
            },
            "translator":
            {
                "type": "TabularTranslator",
                "columnMappings": "<column mapping>"
            },
            "dataIntegrationUnits": <number>,
            "parallelCopies": <number>,
            "enableStaging": true/false,
            "stagingSettings": {
                <properties>
            },
            "enableSkipIncompatibleRow": true/false,
            "redirectIncompatibleRowSettings": {
                <properties>
            }
        }
    }
]
```

#### <a name="syntax-details"></a>Détails de la syntaxe

| Propriété | Description | Requis ? |
|:--- |:--- |:--- |
| type | Pour une activité de copie, définissez sur `Copy` | Oui |
| inputs | Spécifiez le jeu de données que vous avez créé qui pointe vers les données sources. L’activité de copie ne prend en charge qu’une seule entrée. | Oui |
| outputs | Spécifiez le jeu de données que vous avez créé qui pointe vers les données de récepteur. L’activité de copie ne prend en charge qu’une seule sortie. | Oui |
| typeProperties | Spécifiez les propriétés pour configurer l’activité de copie. | Oui |
| source | Spécifiez le type de source de copie et les propriétés correspondantes pour la récupération des données.<br/>Pour plus d’informations, consultez la section « Propriétés de l’activité de copie » de l’article sur le connecteur répertorié dans [Magasins de données et formats pris en charge](#supported-data-stores-and-formats). | Oui |
| sink | Spécifiez le type de récepteur de copie et les propriétés correspondantes pour l’écriture des données.<br/>Pour plus d’informations, consultez la section « Propriétés de l’activité de copie » de l’article sur le connecteur répertorié dans [Magasins de données et formats pris en charge](#supported-data-stores-and-formats). | Oui |
| translator | Spécifiez des mappages de colonnes explicites de la source au récepteur. Cette propriété s’applique lorsque le comportement de copie par défaut ne répond pas à vos besoins.<br/>Pour plus d’informations, consultez [Mappage de schéma dans l’activité de copie](copy-activity-schema-and-type-mapping.md). | Non |
| dataIntegrationUnits | Spécifiez une mesure qui représente la quantité d’énergie que le [runtime d’intégration Azure](concepts-integration-runtime.md) utilise pour la copie des données. Ces unités étaient auparavant appelées unités de déplacement de données Cloud. <br/>Pour plus d’informations, consultez [Unités d’intégration de données](copy-activity-performance-features.md#data-integration-units). | Non |
| parallelCopies | Spécifiez le parallélisme que l’activité de copie doit utiliser lors de la lecture des données de la source et l’écriture des données vers le récepteur.<br/>Pour plus d’informations, voir [Copie en parallèle](copy-activity-performance-features.md#parallel-copy). | Non |
| conserves | Spécifiez s’il faut conserver les métadonnées/ACL lors de la copie des données. <br/>Pour plus d’informations, consultez [Conserver les métadonnées](copy-activity-preserve-metadata.md). |Non |
| enableStaging<br/>stagingSettings | Spécifiez s’il faut effectuer une copie intermédiaire des données intermédiaires dans le stockage Blob au lieu de copier directement les données de la source vers le récepteur.<br/>Pour plus d’informations sur les scénarios utiles et les détails de la configuration, consultez [Copie intermédiaire](copy-activity-performance-features.md#staged-copy). | Non |
| enableSkipIncompatibleRow<br/>redirectIncompatibleRowSettings| Choisissez comment gérer les lignes incompatibles lorsque vous copiez des données de la source vers le récepteur.<br/>Pour plus d’informations, consultez [Tolérance aux pannes](copy-activity-fault-tolerance.md). | Non |

## <a name="monitoring"></a>Surveillance

Vous pouvez surveiller l’exécution de l’activité de copie dans Azure Data Factory visuellement ou par programmation. Pour plus d’informations, consultez [Surveiller l’activité de copie](copy-activity-monitoring.md).

## <a name="incremental-copy"></a>Copie incrémentielle

Data Factory vous permet de copier de façon incrémentielle des données delta d’un magasin de données source vers un magasin de données récepteur. Pour plus d’informations, consultez [Didacticiel : Copier les données de façon incrémentielle](tutorial-incremental-copy-overview.md).

## <a name="performance-and-tuning"></a>Performances et réglage

L’expérience de [surveillance de l’activité de copie](copy-activity-monitoring.md) affiche les statistiques des performances de copie pour chaque exécution d’activité. Le [Guide des performances et de l’évolutivité de l’activité de copie](copy-activity-performance.md) décrit les facteurs clés qui affectent les performances du déplacement des données par le biais de l’activité de copie dans Azure Data Factory. Il répertorie également les valeurs de performances observées pendant le test et explique comment optimiser les performances de l’activité de copie.

## <a name="resume-from-last-failed-run"></a>Reprendre à partir de la dernière exécution ayant échoué

L’activité de copie prend en charge la reprise après l’échec de la dernière exécution lorsque vous copiez une grande taille de fichiers tels quels avec un format binaire entre les magasins basés sur des fichiers et que vous choisissez de conserver la hiérarchie des dossiers/fichiers de la source au récepteur, par exemple pour migrer des données d’Amazon S3 vers Azure Data Lake Storage Gen2. Elle s’applique aux connecteurs basés sur les fichiers suivants : [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [Système de fichiers](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md) et [SFTP](connector-sftp.md).

Vous pouvez tirer parti de la reprise de l’activité de copie des deux manières suivantes :

- **Nouvelle tentative de niveau d’activité :** Vous pouvez définir le nombre de nouvelles tentatives sur l’activité de copie. Pendant l’exécution du pipeline, en cas d’échec de l’exécution de l’activité de copie, la nouvelle tentative automatique suivante démarre à partir du point d’échec de la dernière évaluation.
- **Réexécuter à partir d’une activité ayant échoué :** Une fois l’exécution du pipeline terminée, vous pouvez également déclencher une réexécution à partir de l’activité ayant échoué dans la vue d’analyse de l’interface utilisateur ADF ou par programmation. Si l’activité qui a échoué est une activité de copie, le pipeline n’est pas exécuté à nouveau à partir de cette activité, mais reprend également à partir du point d’échec de l’exécution précédente.

    ![Reprendre la copie](media/copy-activity-overview/resume-copy.png)

Quelques points à noter :

- La reprise se produit au niveau du fichier. Si l’activité de copie échoue lors de la copie d’un fichier, lors de la prochaine exécution, ce fichier spécifique sera recopié.
- Pour que la reprise fonctionne correctement, ne modifiez pas les paramètres de l’activité de copie entre les nouvelles exécutions.
- Lorsque vous copiez des données à partir d’Amazon S3, d’un objet Blob Azure, d’Azure Data Lake Storage Gen2 et de Google Cloud Storage, l’activité de copie peut reprendre à partir d’un nombre arbitraire de fichiers copiés. Alors que pour le reste des connecteurs basés sur des fichiers en tant que source, l’activité de copie prend en charge la reprise à partir d’un nombre limité de fichiers, généralement une plage de dizaines de milliers de fichier, et varie en fonction de la longueur des chemins d’accès aux fichiers. les fichiers au-delà de ce nombre seront copiés à nouveau lors de la réexécution.

Pour les autres scénarios que la copie de fichiers binaires, la réexécution de l’activité de copie commence dès le début.

## <a name="preserve-metadata-along-with-data"></a>Conserver les métadonnées avec les données

Lors de la copie des données de la source vers le récepteur, dans des scénarios tels que la migration d'un lac de données, vous pouvez également choisir de conserver les métadonnées et ACL avec les données à l’aide de l’activité de copie. Pour plus d’informations, consultez [Conserver les métadonnées](copy-activity-preserve-metadata.md).

## <a name="schema-and-data-type-mapping"></a>Mappage du schéma et du type de données

Pour plus d’informations sur la façon dont l’activité de copie met en correspondance vos données sources et votre récepteur, consultez [Mappage de type de données et de schéma](copy-activity-schema-and-type-mapping.md).

## <a name="add-additional-columns-during-copy"></a>Ajouter des colonnes supplémentaires pendant la copie

En plus de copier des données d’une banque de données source vers un récepteur, vous pouvez également configurer l’ajout de colonnes de données supplémentaires à copier dans le récepteur. Par exemple :

- Lors de la copie à partir d’une source basée sur un fichier, enregistrez le chemin d’accès relatif du fichier dans une colonne supplémentaire pour savoir de quel fichier proviennent les données.
- Dupliquez la colonne source spécifiée comme une autre colonne. 
- Ajoutez une colonne avec l’expression ADF, pour joindre des variables système ADF telles que le nom ou l’ID du pipeline, ou stocker une autre valeur dynamique provenant de la sortie de l’activité en amont.
- Ajoutez une colonne avec une valeur statique pour répondre à votre besoin de consommation en aval.

Vous pouvez trouver la configuration suivante dans l’onglet source de l’activité de copie. Vous pouvez également mapper ces colonnes supplémentaires dans l’activité de copie [Mappage de schéma](copy-activity-schema-and-type-mapping.md#schema-mapping) comme d’habitude en utilisant les noms de colonne que vous avez définis. 

![Ajouter des colonnes supplémentaires dans l’activité de copie](./media/copy-activity-overview/copy-activity-add-additional-columns.png)

>[!TIP]
>Cette fonctionnalité fonctionne avec le modèle de jeu de données le plus récent. Si vous ne voyez pas cette option dans l’interface utilisateur, essayez de créer un nouveau jeu de données.

Pour le configurer par programmation, ajoutez la propriété `additionalColumns` dans la source de l’activité de copie :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| additionalColumns | Ajoutez des colonnes de données supplémentaires à copier dans le récepteur.<br><br>Chaque objet sous le tableau `additionalColumns` représente une colonne supplémentaire. `name` définit le nom de la colonne et `value` indique la valeur des données de cette colonne.<br><br>Les valeurs de données autorisées sont :<br>-  **`$$FILEPATH`**  : une variable réservée indique de stocker le chemin d’accès relatif des fichiers sources dans le chemin d’accès du dossier spécifié dans le jeu de données. Appliquer à la source basée sur un fichier.<br>-  **`$$COLUMN:<source_column_name>`**  : un modèle de variable réservée indique de dupliquer la colonne source spécifiée comme une autre colonne.<br>- **Expression**<br>- **Valeur statique** | Non |

**Exemple :**

```json
"activities":[
    {
        "name": "CopyWithAdditionalColumns",
        "type": "Copy",
        "inputs": [...],
        "outputs": [...],
        "typeProperties": {
            "source": {
                "type": "<source type>",
                "additionalColumns": [
                    {
                        "name": "filePath",
                        "value": "$$FILEPATH"
                    },
                    {
                        "name": "newColName",
                        "value": "$$COLUMN:SourceColumnA"
                    },
                    {
                        "name": "pipelineName",
                        "value": {
                            "value": "@pipeline().Pipeline",
                            "type": "Expression"
                        }
                    },
                    {
                        "name": "staticValue",
                        "value": "sampleValue"
                    }
                ],
                ...
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="auto-create-sink-tables"></a>Créer automatiquement des tables de récepteur

Lors de la copie de données dans SQL Database/Azure Synapse Analytics, si la table de destination n’existe pas, l’activité de copie prend en charge sa création automatique en fonction des données sources. Elle vise à vous aider à commencer rapidement à charger les données et à évaluer SQL Database/Azure Synapse Analytics. Après l’ingestion des données, vous pouvez examiner et ajuster le schéma de la table de récepteur en fonction de vos besoins.

Cette fonctionnalité est prise en charge lors de la copie de données à partir de n’importe quelle source vers les magasins de données récepteurs suivants. Vous pouvez trouver l’option sur l’*interface utilisateur de création ADF* –> *Récepteur d’activité de copie* –> *Option de table* –> *Créer automatiquement une table*, ou via la propriété `tableOption` dans la charge utile de récepteur d’activité de copie.

- [Azure SQL Database](connector-azure-sql-database.md)
- [Azure SQL Database Managed Instance](connector-azure-sql-managed-instance.md)
- [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md)
- [SQL Server](connector-sql-server.md)

![Créer des tables de récepteur](media/copy-activity-overview/create-sink-table.png)

## <a name="fault-tolerance"></a>Tolérance de panne

Par défaut, l’activité de copie arrête la copie des données et retourne un échec lorsque les lignes de données sources sont incompatibles avec les lignes de données du récepteur. Pour que la copie aboutisse, vous pouvez configurer l’activité de copie afin d’ignorer et de journaliser les lignes incompatibles et de copier uniquement les données compatibles. Pour plus d’informations, consultez [Tolérance de panne de l’activité de copie](copy-activity-fault-tolerance.md).

## <a name="data-consistency-verification"></a>Vérification de la cohérence des données

Lorsque vous déplacez des données du magasin source au magasin de destination, l’activité de copie d’Azure Data Factory vous offre la possibilité de faire une vérification supplémentaire de la cohérence des données pour vous assurer que les données sont non seulement copiées du magasin source au magasin de destination, mais également que leur cohérence entre les deux magasins de données est vérifiée. Une fois que des fichiers incohérents ont été détectés pendant le déplacement des données, vous pouvez soit abandonner l’activité de copie, soit continuer à copier le reste en activant le paramètre de tolérance de panne afin d’ignorer les fichiers incohérents. Vous pouvez récupérer les noms de fichiers ignorés en activant le paramètre de journal de session dans l’activité de copie. Consultez [Vérification de la cohérence des données dans l’activité de copie](copy-activity-data-consistency.md) pour plus d’informations.

## <a name="session-log"></a>Journal de session
Vous pouvez journaliser les noms de fichiers copiés, ce qui peut vous aider à mieux vous assurer que les données sont non seulement bien copiées du magasin source vers le magasin de destination, mais qu’elles sont également cohérentes entre les deux magasins en examinant les journaux de session de l’activité de copie. Consultez [Journal de session dans l’activité de copie](copy-activity-log.md) pour plus de détails.

## <a name="next-steps"></a>Étapes suivantes
Voir les procédures de démarrage rapide, didacticiels et exemples suivants :

- [Copier des données d’un emplacement vers un autre dans le même compte de stockage Blob Azure](quickstart-create-data-factory-dot-net.md)
- [Copier des données de stockage Blob Azure vers Azure SQL Database](tutorial-copy-data-dot-net.md)
- [Copier des données d'une base de données SQL Server vers Azure](tutorial-hybrid-copy-powershell.md)
