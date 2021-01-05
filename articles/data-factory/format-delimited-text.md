---
title: Format de texte délimité dans Azure Data Factory
description: Cette rubrique décrit comment traiter le format de texte délimité dans Azure Data Factory.
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 12/07/2020
ms.author: jingwang
ms.openlocfilehash: 0fa68a8dbdcb1f2c0cc4af1b4df751fd81c0de14
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854629"
---
# <a name="delimited-text-format-in-azure-data-factory"></a>Format de texte délimité dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Suivez cet article si vous souhaitez **analyser des fichiers de texte délimité ou écrire des données au format de texte délimité**. 

Le format de texte délimité est pris en charge pour les connecteurs suivants : [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [Système de fichiers](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) et [SFTP](connector-sftp.md).

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données de texte délimité.

| Propriété         | Description                                                  | Obligatoire |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propriété de type du jeu de données doit être définie sur **DelimitedText**. | Oui      |
| location         | Paramètres d’emplacement du ou des fichiers. Chaque connecteur basé sur un fichier possède ses propres type d’emplacement et propriétés prises en charge sous `location`.  | Oui      |
| columnDelimiter  | Caractère(s) utilisé(s) pour séparer les colonnes dans un fichier. <br>La valeur par défaut est la **virgule`,`** . Lorsque le délimiteur de colonne est défini comme une chaîne vide, ce qui signifie aucun délimiteur, la ligne entière est considérée comme une seule colonne.<br>Actuellement, le délimiteur, chaîne vide ou multi-char par exemple, est uniquement pris en charge pour le flux de données de mappage, mais pas l’activité de copie.  | Non       |
| rowDelimiter     | Caractère unique ou « \r\n » utilisé pour séparer les lignes dans un fichier. <br>La valeur par défaut est l’une des valeurs suivantes **lors de la lecture : [« \r\n », « \r », « \n »]** et **« \n » ou « \r\n » en écriture** respectivement par le flux de données de mappage et l’activité de copie. <br>Lorsque le délimiteur de ligne est défini sur aucun délimiteur (chaîne vide), le délimiteur de colonne doit être également défini comme aucun délimiteur (chaîne vide), ce qui signifie le fait de traiter l’intégralité du contenu comme une valeur unique.<br>Actuellement, le délimiteur de ligne, chaîne vide par exemple, est uniquement pris en charge pour le flux de données de mappage, mais pas l’activité de copie. | Non       |
| quoteChar        | Le caractère unique pour mettre entre guillemets les valeurs de colonne si elle contient un délimiteur de colonne. <br>La valeur par défaut est **guillemets anglais doubles** `"`. <br>Lorsque `quoteChar` est défini comme une chaîne vide, cela signifie qu’il n’existe aucun caractère de citation, la valeur de colonne n’est pas mise entre guillemets, et `escapeChar` est utilisé pour soustraire le délimiteur de colonne et lui-même. | Non       |
| escapeChar       | Le caractère unique pour placer les guillemets dans une séquence d’échappement à l’intérieur d’une valeur entre guillemets.<br>La valeur par défaut est **Barre oblique inverse`\`** . <br>Lorsque `escapeChar` est défini comme une chaîne vide, `quoteChar` doit également être défini comme une chaîne vide, auquel cas vous devez vous assurer que toutes les valeurs de colonne ne contiennent pas de délimiteurs. | Non       |
| firstRowAsHeader | Spécifie s’il faut considérer/faire de la première ligne une ligne d’en-tête avec les noms des colonnes.<br>Les valeurs autorisées sont **True** et **False** (par défaut).<br>Lorsque la première ligne en tant qu’en-tête a la valeur false, notez que la sortie de l’activité de recherche et d’aperçu des données de l’interface utilisateur génère automatiquement des noms de colonnes tels que Prop_{n} (à partir de 0). L’activité de copie requiert un [mappage explicite](copy-activity-schema-and-type-mapping.md#explicit-mapping) de la source au récepteur, et recherche les colonnes par ordinal (à partir de 1). Et le flux de données de mappage répertorie et recherche les colonnes par leur nom, par exemple Column_{n} (à partir de 1).  | Non       |
| nullValue        | Spécifie la représentation sous forme de chaîne de la valeur null. <br>La valeur par défaut est une **chaîne vide**. | Non       |
| encodingName     | Le type de codage utilisé pour lire/écrire des fichiers de test. <br>Les valeurs autorisées sont les suivantes : « UTF-8 », « UTF-16 », « UTF-16BE », « UTF-32 », « UTF-32BE NE », « US-ASCII », « UTF-7 », « BIG5 », « EUC-JP », « EUC-KR », « GB2312 », « GB18030 », « JOHAB », « SHIFT-JIS », « CP875 », « CP866 », « IBM00858 », « IBM037 », « IBM273 », « IBM437 », « IBM500 », « IBM737 », « IBM775 », « IBM850 », » IBM852 », « IBM855 », « IBM857 », « IBM860 », « IBM861 », « IBM863 », « IBM864 », « IBM865 », « IBM869 », « IBM870 », « IBM01140 », « IBM01141 », « IBM01142 », « IBM01143 », « IBM01144 », « IBM01145 », « IBM01146 », « IBM01147 », « IBM01148 », « IBM01149 », « ISO-2022-JP », « ISO-2022-KR «, « ISO-8859-1 », « ISO-8859-2 », « ISO-8859-3", « ISO-8859-4 », « ISO-8859-5 », « ISO-8859-6 », « ISO-8859-7 », « ISO-8859-8 », « ISO-8859-9 », « ISO-8859-13 », « ISO-8859-15 », « WINDOWS-874 », « WINDOWS-1250 », « WINDOWS-1251 », « WINDOWS-1252 », « WINDOWS-1253 », » WINDOWS-1254 », « WINDOWS-1255 », « WINDOWS-1256 », « WINDOWS-1257 », « WINDOWS-1258 ».<br>Notez que le flux de données de mappage ne prend pas en charge le codage UTF-7. | Non       |
| compressionCodec | Le codec de compression utilisé pour lire/écrire des fichiers texte. <br>Les valeurs autorisées sont **bzip2**, **gzip**, **deflate**, **ZipDeflate**, **TarGzip**, **Tar**, **snappy** et **lz4**. La valeur par défaut n’est pas compressée. <br>**Remarque :** Pour l’instant, l’activité de copie ne prend pas en charge « snappy » ni « lz4 ». Le flux de données de mappage ne prend pas en charge « ZipDeflate », « TarGzip » ni « Tar ». <br>**Remarque :** Lorsque l’activité de copie est utilisée pour décompresser un ou plusieurs fichiers **ZipDeflate**/**TarGzip**/**Tar** et écrire dans un magasin de données récepteur basé sur des fichiers, les fichiers sont par défaut extraits dans le dossier `<path specified in dataset>/<folder named as source compressed file>/`. Utilisez `preserveZipFileNameAsFolder`/`preserveCompressionFileNameAsFolder` sur [source de l’activité de copie](#delimited-text-as-source) pour déterminer si le nom du ou des fichiers compressés doit être conservé comme structure de dossier. | Non       |
| compressionLevel | Le taux de compression. <br>Les valeurs autorisées sont **Optimal** ou **Fastest**.<br>- **Fastest (le plus rapide) :** l’opération de compression doit se terminer le plus rapidement possible, même si le fichier résultant n’est pas compressé de façon optimale.<br>- **Optimal** : l’opération de compression doit aboutir à une compression optimale, même si elle prend plus de temps. Pour plus d’informations, consultez la rubrique [Niveau de compression](/dotnet/api/system.io.compression.compressionlevel) . | Non       |

Voici un exemple de jeu de données de texte délimité sur Stockage Blob Azure :

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "escapeChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source et le récepteur de texte délimité.

### <a name="delimited-text-as-source"></a>Texte délimité en tant que source 

Les propriétés prises en charge dans la section **_\_source\*** * de l’activité de copie sont les suivantes.

| Propriété       | Description                                                  | Obligatoire |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | La propriété type de la source d’activité de copie doit être définie sur **DelimitedTextSource**. | Oui      |
| formatSettings | Un groupe de propriétés. Reportez-vous au tableau **Paramètres de lecture du texte délimité** ci-dessous. |  Non       |
| storeSettings  | Un groupe de propriétés sur la façon de lire les données d’un magasin de données. Chaque connecteur basé sur un fichier possède ses propres paramètres de lecture pris en charge sous `storeSettings`. | Non       |

Prise en charge des **paramètres de lecture du texte délimité** sous `formatSettings`:

| Propriété      | Description                                                  | Obligatoire |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | Le type de formatSettings doit être défini sur **DelimitedTextReadSettings**. | Oui      |
| skipLineCount | Indique le nombre de lignes **non vides** à ignorer lors de la lecture des données à partir des fichiers d’entrée. <br>Si skipLineCount et firstRowAsHeader sont spécifiés, les lignes sont d’abord ignorées, puis les informations d’en-têtes sont lues à partir du fichier d’entrée. | Non       |
| compressionProperties | Groupe de propriétés permettant de décompresser les données d’un codec de compression spécifique. | Non       |
| preserveZipFileNameAsFolder<br>(*sous `compressionProperties`->`type` en tant que `ZipDeflateReadSettings`* ) |  S’applique lorsque le jeu de données d’entrée est configuré avec la compression **ZipDeflate**. Indique si le nom du fichier zip source doit être conservé en tant que structure de dossiers lors de la copie.<br>– Lorsque la valeur est définie sur **true (par défaut)** , Data Factory écrit les fichiers décompressés dans `<path specified in dataset>/<folder named as source zip file>/`.<br>– Lorsque la valeur est définie sur **false**, Data Factory écrit les fichiers décompressés directement dans `<path specified in dataset>`. Assurez-vous de ne pas avoir de noms de fichiers dupliqués dans les différents fichiers zip sources afin d’éviter toute course ou tout comportement inattendu.  | Non |
| preserveCompressionFileNameAsFolder<br>(*sous `compressionProperties`->`type` en tant que `TarGZipReadSettings` ou `TarReadSettings`* )  | S'applique lorsque le jeu de données d'entrée est configuré avec la compression **TarGzip**/**Tar**. Indique si le nom du fichier source compressé doit être conservé en tant que structure de dossiers lors de la copie.<br>– Lorsque la valeur est définie sur **true (par défaut)** , Data Factory écrit les fichiers décompressés dans `<path specified in dataset>/<folder named as source compressed file>/`. <br>– Lorsque la valeur est définie sur **false**, Data Factory écrit les fichiers décompressés directement dans `<path specified in dataset>`. Assurez-vous de ne pas avoir de noms de fichiers dupliqués dans les différents fichiers sources afin d’éviter toute course ou tout comportement inattendu. | Non |

```json
"activities": [
    {
        "name": "CopyFromDelimitedText",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "DelimitedTextSource",
                "storeSettings": {
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                },
                "formatSettings": {
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 3,
                    "compressionProperties": {
                        "type": "ZipDeflateReadSettings",
                        "preserveZipFileNameAsFolder": false
                    }
                }
            },
            ...
        }
        ...
    }
]
```

### <a name="delimited-text-as-sink"></a>Texte délimité en tant que récepteur

Les propriétés prises en charge dans la section **_\_sink\*** * de l’activité de copie sont les suivantes.

| Propriété       | Description                                                  | Obligatoire |
| -------------- | ------------------------------------------------------------ | -------- |
| type           | La propriété type de la source d’activité de copie doit être définie sur **DelimitedTextSink**. | Oui      |
| formatSettings | Un groupe de propriétés. Reportez-vous au tableau **Paramètres d’écriture du texte délimité** ci-dessous. |    Non      |
| storeSettings  | Groupe de propriétés sur la méthode d’écriture de données dans un magasin de données. Chaque connecteur basé sur un fichier possède ses propres paramètres d’écriture pris en charge sous `storeSettings`.  | Non       |

Prise en charge des **paramètres d’écriture du texte délimité** sous `formatSettings`:

| Propriété      | Description                                                  | Obligatoire                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | Le type de formatSettings doit être défini sur **DelimitedTextWriteSettings**. | Oui                                                   |
| fileExtension | Extension de fichier utilisée pour nommer les fichiers de sortie, par exemple : `.csv`, `.txt`. Elle doit être spécifiée lorsque `fileName` n’est pas spécifié dans le jeu de données de sortie DelimitedText. Quand le nom de fichier est configuré dans le jeu de données de sortie, il est utilisé comme nom de fichier du récepteur et le paramètre d’extension de fichier est ignoré.  | Oui lorsque le nom de fichier n’est pas spécifié dans le jeu de données de sortie |
| maxRowsPerFile | Lorsque vous écrivez des données dans un dossier, vous pouvez choisir d’écrire dans plusieurs fichiers et de spécifier le nombre maximal de lignes par fichier.  | Non |
| fileNamePrefix | Applicable lorsque `maxRowsPerFile` est configuré.<br> Spécifiez le préfixe du nom de fichier lors de l’écriture de données dans plusieurs fichiers, ce qui a généré ce modèle : `<fileNamePrefix>_00000.<fileExtension>`. S’il n’est pas spécifié, le préfixe du nom de fichier est généré automatiquement. Cette propriété ne s’applique pas lorsque la source est un magasin basé sur des fichiers ou un [magasin de données partition-option-enabled](copy-activity-performance-features.md).  | Non |

## <a name="mapping-data-flow-properties"></a>Propriétés du mappage de flux de données

Dans les flux de données de mappage, vous pouvez lire et écrire des données au format de texte délimité dans les magasins de données suivants : [Stockage Blob Azure](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties).

### <a name="source-properties"></a>Propriétés de source

Le tableau ci-dessous répertorie les propriétés prises en charge par une source de texte délimité. Vous pouvez modifier ces propriétés sous l’onglet **Options de la source**.

| Nom | Description | Obligatoire | Valeurs autorisées | Propriété du script de flux de données |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Chemins génériques | Tous les fichiers correspondant au chemin générique seront traités. Remplace le chemin du dossier et du fichier défini dans le jeu de données. | non | String[] | wildcardPaths |
| Chemin racine de la partition | Pour les données de fichier qui sont partitionnées, vous pouvez entrer le chemin racine d’une partition pour pouvoir lire les dossiers partitionnés comme des colonnes. | non | String | partitionRootPath |
| Liste de fichiers | Si votre source pointe ou non vers un fichier texte qui liste les fichiers à traiter | non | `true` ou `false` | fileList |
| Lignes multilignes | Le fichier source contient-il des lignes qui s’étendent sur plusieurs lignes. Les valeurs multilignes doivent être entre guillemets. | aucun `true` ou `false` | multiLineRow |
| Colonne où stocker le nom du fichier | Crée une colonne avec le nom et le chemin du fichier source | non | String | rowUrlColumn |
| Après l’exécution | Supprime ou déplace les fichiers après le traitement. Le chemin du fichier commence à la racine du conteneur | non | Supprimer : `true` ou `false` <br> Déplacer : `['<from>', '<to>']` | purgeFiles <br> moveFiles |
| Filtrer par date de dernière modification | Choisir pour filtrer les fichiers en fonction de leur date de dernière modification | non | Timestamp | modifiedAfter <br> modifiedBefore |
| N’autoriser aucun fichier trouvé | Si la valeur est true, aucune erreur n’est levée si aucun fichier n’est trouvé | non | `true` ou `false` | ignoreNoFilesFound |

### <a name="source-example"></a>Exemple de source

L’image ci-dessous est un exemple de configuration de source de texte délimité dans des flux de données de mappage.

![Source DelimitedText](media/data-flow/delimited-text-source.png)

Le script de flux de données associé est le suivant :

```
source(
    allowSchemaDrift: true,
    validateSchema: false,
    multiLineRow: true,
    wildcardPaths:['*.csv']) ~> CSVSource
```

> [!NOTE]
> Les sources de flux de données prennent en charge un ensemble limité d’extensions métacaractères (globbing) Linux prises en charge par les systèmes de fichiers Hadoop

### <a name="sink-properties"></a>Propriétés du récepteur

Le tableau ci-dessous répertorie les propriétés prises en charge par un récepteur de texte délimité. Vous pouvez modifier ces propriétés sous l’onglet **Paramètres**.

| Nom | Description | Obligatoire | Valeurs autorisées | Propriété du script de flux de données |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Effacer le contenu du dossier | Si le dossier de destination est vidé avant l’écriture | non | `true` ou `false` | truncate |
| Option de nom de fichier | Format de nommage des données écrites. Par défaut, un fichier par partition au format `part-#####-tid-<guid>` | non | Modèle : String <br> Par partition : String[] <br> Comme les données de la colonne : String <br> Sortie dans un fichier unique : `['<fileName>']`  | filePattern <br> partitionFileNames <br> rowUrlColumn <br> partitionFileNames |
| Tout mettre entre guillemets | Placer toutes les valeurs entre guillemets | non | `true` ou `false` | quoteAll |

### <a name="sink-example"></a>Exemple de récepteur

L’image ci-dessous est un exemple de configuration de récepteur de texte délimité dans des flux de données de mappage.

![Récepteur DelimitedText](media/data-flow/delimited-text-sink.png)

Le script de flux de données associé est le suivant :

```
CSVSource sink(allowSchemaDrift: true,
    validateSchema: false,
    truncate: true,
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> CSVSink
```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
- [Mappage de flux de données](concepts-data-flow-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)
