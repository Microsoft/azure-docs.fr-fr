---
title: Format XML dans Azure Data Factory
description: Cette rubrique décrit comment traiter le format XML dans Azure Data Factory.
author: linda33wj
manager: shwang
ms.reviewer: craigg
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 10/29/2020
ms.author: jingwang
ms.openlocfilehash: 1555d8b97f89b567cb1769c0ba10ecf50bca4366
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352594"
---
# <a name="xml-format-in-azure-data-factory"></a>Format XML dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Suivez cet article lorsque vous souhaitez **analyser les fichiers XML**. 

Le format XML est pris en charge pour les connecteurs suivants : [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [Système de fichiers](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) et [SFTP](connector-sftp.md). Il est pris en charge en tant que source, mais pas en tant que récepteur.

## <a name="dataset-properties"></a>Propriétés du jeu de données

Pour obtenir la liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Jeux de données](concepts-datasets-linked-services.md). Cette section fournit la liste des propriétés prises en charge par le jeu de données XML.

| Propriété         | Description                                                  | Obligatoire |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propriété type du jeu de données doit être définie sur **Xml**. | Oui      |
| location         | Paramètres d’emplacement du ou des fichiers. Chaque connecteur basé sur un fichier possède ses propres type d’emplacement et propriétés prises en charge sous `location`. **Consultez les détails dans l’article du connecteur -> section des propriétés du jeu de données**. | Oui      |
| encodingName     | Le type de codage utilisé pour lire/écrire des fichiers de test. <br>Les valeurs autorisées sont les suivantes : « UTF-8 », « UTF-16 », « UTF-16BE », « UTF-32 », « UTF-32BE », « US-ASCII », « UTF-7 », « BIG5 », « EUC-JP », « EUC-KR », « GB2312 », « GB18030 », « JOHAB », « SHIFT-JIS », « CP875 », « CP866 », « IBM00858 », « IBM037 », « IBM273 », « IBM437 », « IBM500 », « IBM737 », « IBM775 », « IBM850 », « IBM852 », « IBM855 », « IBM857 », « IBM860 », « IBM861 », « IBM863 », « IBM864 », « IBM865 », « IBM869 », « IBM870 », « IBM01140 », « IBM01141 », « IBM01142 », « IBM01143 », « IBM01144 », « IBM01145 », « IBM01146 », « IBM01147 », « IBM01148 », « IBM01149 », « ISO-2022-JP », « ISO-2022-KR », « ISO-8859-1 », « ISO-8859-2 », « ISO-8859-3 », « ISO-8859-4 », « ISO-8859-5 », « ISO-8859-6 », « ISO-8859-7 », « ISO-8859-8 », « ISO-8859-9 », « ISO-8859-13 », « ISO-8859-15 », « WINDOWS-874 », « WINDOWS-1250 », « WINDOWS-1251 », « WINDOWS-1252 », « WINDOWS-1253 », » WINDOWS-1254 », « WINDOWS-1255 », « WINDOWS-1256 », « WINDOWS-1257 », « WINDOWS-1258 ».| Non       |
| nullValue | Spécifie la représentation sous forme de chaîne de la valeur null.<br/>La valeur par défaut est une **chaîne vide**. | Non |
| compression | Groupe de propriétés pour configurer la compression de fichier. Configurez cette section lorsque vous souhaitez effectuer la compression/décompression lors de l’exécution de l’activité. | Non |
| type<br>(*sous `compression`* ) | Le codec de compression utilisé pour lire/écrire des fichiers XML. <br>Les valeurs autorisées sont **bzip2**, **gzip**, **deflate**, **ZipDeflate**, **TarGzip**, **Tar**, **snappy** et **lz4**. La valeur par défaut n’est pas compressée.<br>**Remarque :** Pour l’instant, l’activité de copie ne prend pas en charge « snappy » ni « lz4 ». Le flux de données de mappage ne prend pas en charge « ZipDeflate », « TarGzip » ni « Tar ».<br>**Remarque :** Lorsque l’activité de copie est utilisée pour décompresser un ou des fichiers **ZipDeflate**/**TarGzip**/**Tar** et écrire dans un magasin de données récepteur basé sur des fichiers, les fichiers sont par défaut extraits dans le dossier `<path specified in dataset>/<folder named as source compressed file>/`. Utilisez `preserveZipFileNameAsFolder`/`preserveCompressionFileNameAsFolder` sur [source de l’activité de copie](#xml-as-source) pour déterminer si le nom du ou des fichiers compressés doit être conservé comme structure de dossier. | Non.  |
| level<br/>(*sous `compression`* ) | Le taux de compression. <br>Les valeurs autorisées sont **Optimal** ou **Fastest**.<br>- **Fastest (le plus rapide) :** l’opération de compression doit se terminer le plus rapidement possible, même si le fichier résultant n’est pas compressé de façon optimale.<br>- **Optimal** : l’opération de compression doit aboutir à une compression optimale, même si elle prend plus de temps. Pour plus d’informations, consultez la rubrique [Niveau de compression](/dotnet/api/system.io.compression.compressionlevel) . | Non       |

Voici un exemple de jeu de données XML sur Stockage Blob Azure :

```json
{
    "name": "XMLDataset",
    "properties": {
        "type": "Xml",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compression": {
                "type": "ZipDeflate"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Pipelines](concepts-pipelines-activities.md). Cette section fournit la liste des propriétés prises en charge par la source XML.

Découvrez comment mapper des données XML et le magasin/format de données récepteur à partir du [mappage de schéma](copy-activity-schema-and-type-mapping.md). Lorsque vous affichez un aperçu des fichiers XML, les données sont affichées avec la hiérarchie JSON et vous utilisez le chemin JSON pour pointer vers les champs.

### <a name="xml-as-source"></a>XML en tant que source

Les propriétés prises en charge dans la section **_\_source\*** * de l’activité de copie sont les suivantes. Pour plus d’informations, consultez [Comportement du connecteur XML](#xml-connector-behavior).

| Propriété      | Description                                                  | Obligatoire |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propriété type de la source de l’activité de copie doit être définie sur **XmlSource**. | Oui      |
| formatSettings | Un groupe de propriétés. Reportez-vous au tableau **Paramètres de lecture XML** ci-dessous. | Non       |
| storeSettings | Un groupe de propriétés sur la façon de lire les données d’un magasin de données. Chaque connecteur basé sur un fichier possède ses propres paramètres de lecture pris en charge sous `storeSettings`. **Consultez les détails dans l’article du connecteur -> section des propriétés de l’activité de copie**. | Non       |

**Paramètres de lecture XML** pris en charge sous `formatSettings` :

| Propriété      | Description                                                  | Obligatoire |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | Le type de formatSettings doit être défini sur **XmlReadSettings**. | Oui      |
| validationMode | Spécifie s’il faut valider le schéma XML.<br>Les valeurs autorisées sont **none** (aucun) (valeur par défaut, pas de validation), **xsd** (validation avec XSD), **dtd** (validation avec DTD). | Non |
| espaces de noms | Indique s’il faut activer l’espace de noms lors de l’analyse des fichiers XML. Valeurs autorisées : **true** (par défaut) et **false**. | Non |
| namespacePrefixes | Mappage URI d’espace de noms à préfixe utilisé pour nommer les champs lors de l’analyse du fichier xml.<br/>Si un fichier XML a un espace de noms et que celui-ci est activé, par défaut, le nom du champ est le même que dans le document XML.<br>Si un élément est défini pour l’URI d’espace de noms dans ce mappage, le nom du champ est `prefix:fieldName`. | Non |
| detectDataType | Indique s’il faut détecter les types de données entier, double et booléen. Valeurs autorisées : **true** (par défaut) et **false**.| Non |
| compressionProperties | Groupe de propriétés permettant de décompresser les données d’un codec de compression spécifique. | Non       |
| preserveZipFileNameAsFolder<br>(*sous `compressionProperties`->`type` en tant que `ZipDeflateReadSettings`* )  | S’applique lorsque le jeu de données d’entrée est configuré avec la compression **ZipDeflate**. Indique si le nom du fichier zip source doit être conservé en tant que structure de dossiers lors de la copie.<br>– Lorsque la valeur est définie sur **true (par défaut)** , Data Factory écrit les fichiers décompressés dans `<path specified in dataset>/<folder named as source zip file>/`.<br>– Lorsque la valeur est définie sur **false**, Data Factory écrit les fichiers décompressés directement dans `<path specified in dataset>`. Assurez-vous de ne pas avoir de noms de fichiers dupliqués dans les différents fichiers zip sources afin d’éviter toute course ou tout comportement inattendu.  | Non |
| preserveCompressionFileNameAsFolder<br>(*sous `compressionProperties`->`type` en tant que `TarGZipReadSettings` ou `TarReadSettings`* ) | S'applique lorsque le jeu de données d'entrée est configuré avec la compression **TarGzip**/**Tar**. Indique si le nom du fichier source compressé doit être conservé en tant que structure de dossiers lors de la copie.<br>– Lorsque la valeur est définie sur **true (par défaut)** , Data Factory écrit les fichiers décompressés dans `<path specified in dataset>/<folder named as source compressed file>/`. <br>– Lorsque la valeur est définie sur **false**, Data Factory écrit les fichiers décompressés directement dans `<path specified in dataset>`. Assurez-vous de ne pas avoir de noms de fichiers dupliqués dans les différents fichiers sources afin d’éviter toute course ou tout comportement inattendu. | Non |

## <a name="mapping-data-flow-properties"></a>Propriétés du mappage de flux de données

Dans les flux de données de mappage, vous pouvez lire et écrire des données au format XML dans les magasins de données suivants : [Stockage Blob Azure](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) et [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties). Vous pouvez pointer vers des fichiers XML à l’aide d’un jeu de données XML ou d’un [jeu de données inline](data-flow-source.md#inline-datasets).

### <a name="source-properties"></a>Propriétés de source

Le tableau ci-dessous liste les propriétés prises en charge par une source XML. Vous pouvez modifier ces propriétés sous l’onglet **Options de la source**. Pour plus d’informations, consultez [Comportement du connecteur XML](#xml-connector-behavior). Lorsque vous utilisez un jeu de données inlined, vous verrez des paramètres de fichier supplémentaires qui sont les mêmes que les propriétés décrites dans la section des [propriétés du jeu de données](#dataset-properties). 

| Nom | Description | Obligatoire | Valeurs autorisées | Propriété du script de flux de données |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Chemins génériques | Tous les fichiers correspondant au chemin générique seront traités. Remplace le chemin du dossier et du fichier défini dans le jeu de données. | Non | String[] | wildcardPaths |
| Chemin racine de la partition | Pour les données de fichier qui sont partitionnées, vous pouvez entrer le chemin racine d’une partition pour pouvoir lire les dossiers partitionnés comme des colonnes. | Non | String | partitionRootPath |
| Liste de fichiers | Si votre source pointe ou non vers un fichier texte qui liste les fichiers à traiter | Non | `true` ou `false` | fileList |
| Colonne où stocker le nom du fichier | Crée une colonne avec le nom et le chemin du fichier source | Non | String | rowUrlColumn |
| Après l’exécution | Supprime ou déplace les fichiers après le traitement. Le chemin du fichier commence à la racine du conteneur | Non | Supprimer : `true` ou `false` <br> Déplacer : `['<from>', '<to>']` | purgeFiles <br>moveFiles |
| Filtrer par date de dernière modification | Pour filtrer les fichiers en fonction de leur date de dernière modification | Non | Timestamp | modifiedAfter <br>modifiedBefore |
| Mode de validation | Spécifie s’il faut valider le schéma XML. | Non | `None` (valeur par défaut, pas de validation)<br>`xsd` (validation avec XSD)<br>`dtd` (validation avec DTD). | validationMode |
| Espaces de noms | Indique s’il faut activer l’espace de noms lors de l’analyse des fichiers XML. | Non | `true` (par défaut) ou `false` | espaces de noms |
| Paires espace de noms/préfixe | Mappage URI d’espace de noms à préfixe utilisé pour nommer les champs lors de l’analyse du fichier xml.<br/>Si un fichier XML a un espace de noms et que celui-ci est activé, par défaut, le nom du champ est le même que dans le document XML.<br>Si un élément est défini pour l’URI d’espace de noms dans ce mappage, le nom du champ est `prefix:fieldName`. | Non | Tableau avec modèle`['URI1'->'prefix1','URI2'->'prefix2']` | namespacePrefixes |
| N’autoriser aucun fichier trouvé | Si la valeur est true, aucune erreur n’est levée si aucun fichier n’est trouvé | non | `true` ou `false` | ignoreNoFilesFound |

### <a name="xml-source-script-example"></a>Exemple de script source XML

Le script ci-dessous est un exemple de configuration de source XML dans des flux de données de mappage utilisant le mode du jeu de données.

```
source(allowSchemaDrift: true,
    validateSchema: false,
    validationMode: 'xsd',
    namespaces: true) ~> XMLSource
```

Le script ci-dessous est un exemple de configuration de source XML utilisant le mode du jeu de données inline.

```
source(allowSchemaDrift: true,
    validateSchema: false,
    format: 'xml',
    fileSystem: 'filesystem',
    folderPath: 'folder',
    validationMode: 'xsd',
    namespaces: true) ~> XMLSource
```

## <a name="xml-connector-behavior"></a>Comportement du connecteur XML

Notez les éléments suivants quand vous utilisez XML en tant que source.

- Attributs XML :

    - Les attributs d’un élément sont analysés en tant que sous-champs de l’élément dans la hiérarchie.
    - Le nom du champ d’attribut suit le modèle `@attributeName`.

- Validation de schéma XML :

    - Vous pouvez choisir de ne pas valider le schéma ou de valider le schéma à l’aide de XSD ou DTD.
    - Lorsque vous utilisez XSD ou DTD pour valider des fichiers XML, XSD/DTD doit être référencé à l’intérieur des fichiers XML via un chemin relatif.

- Gestion d’espace de noms :

    - L’espace de noms peut être désactivé lors de l’utilisation de flux de données. Dans ce cas, les attributs qui définissent l’espace de noms seront analysés comme des attributs normaux.
    - Lorsque l’espace de noms est activé, les noms de l’élément et des attributs suivent les modèles `namespaceUri,elementName` et `namespaceUri,@attributeName` par défaut. Vous pouvez définir un préfixe d’espace de noms pour chaque URI d’espace de noms dans la source. Dans ce cas, les noms de l’élément et des attributs suivent le modèle `definedPrefix:elementName` ou `definedPrefix:@attributeName` à la place.

- Colonne de valeur :

    - Si un élément XML comporte à la fois une valeur de texte simple et des attributs/éléments enfants, la valeur de texte simple est analysée comme valeur d’une « colonne de valeur » avec le nom de champ intégré `_value_`. Elle hérite également de l’espace de noms de l’élément, le cas échéant.

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
- [Mappage de flux de données](concepts-data-flow-overview.md)
- [Activité de recherche](control-flow-lookup-activity.md)
- [Activité GetMetadata](control-flow-get-metadata-activity.md)