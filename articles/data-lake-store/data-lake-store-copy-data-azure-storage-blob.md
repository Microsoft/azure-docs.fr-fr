---
title: Copier des données de blobs Stockage Azure vers Data Lake Storage Gen1
description: Utiliser l’outil AdlCopy pour copier des données d’objets blob de Stockage Azure vers Azure Data Lake Storage Gen1
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 84ee65b05af4393f49696875bda41df39e283d5d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85980087"
---
# <a name="copy-data-from-azure-storage-blobs-to-azure-data-lake-storage-gen1"></a>Copier des données d’objets blob de Stockage Azure vers Azure Data Lake Storage Gen1

> [!div class="op_single_selector"]
> * [Utilisation de DistCp](data-lake-store-copy-data-wasb-distcp.md)
> * [Utilisation d’AdlCopy](data-lake-store-copy-data-azure-storage-blob.md)
>
>

Data Lake Storage Gen1 fournit un outil en ligne de commande nommé [AdlCopy](https://www.microsoft.com/download/details.aspx?id=50358), qui permet de copier des données à partir des sources suivantes :

* De blobs Stockage Azure vers Azure Data Lake Storage Gen1. Vous ne pouvez pas utiliser AdlCopy pour copier des données d’Azure Data Lake Storage Gen1 vers des blobs Stockage Azure.
* Entre deux comptes Data Lake Storage Gen1.

Vous pouvez également utiliser l’outil AdlCopy dans deux modes différents :

* **Autonome** : l’outil utilise des ressources Data Lake Storage Gen1 pour effectuer la tâche.
* **À l’aide d’un compte Data Lake Analytics**: les unités affectées à votre compte Data Lake Analytics permettent d’effectuer l’opération de copie. Vous pouvez utiliser cette option lorsque vous souhaitez que les tâches de copie s’effectuent de manière prévisible.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer cet article, vous devez disposer des éléments suivants :

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* Un conteneur de **blobs Stockage Azure** avec quelques données.
* **Un compte Data Lake Storage Gen1**. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md).
* **Compte Data Lake Analytics (facultatif)**  : voir [Prise en main d’Azure Data Lake Analytics](../data-lake-analytics/data-lake-analytics-get-started-portal.md) pour obtenir des instructions sur la création d’un compte Data Lake Analytics.
* **Outil AdlCopy**. Installez [l’outil AdlCopy](https://www.microsoft.com/download/details.aspx?id=50358).

## <a name="syntax-of-the-adlcopy-tool"></a>Syntaxe de l’outil AdlCopy

L’outil AdlCopy utilise la syntaxe suivante.

```console
AdlCopy /Source <Blob or Data Lake Storage Gen1 source> /Dest <Data Lake Storage Gen1 destination> /SourceKey <Key for Blob account> /Account <Data Lake Analytics account> /Units <Number of Analytics units> /Pattern
```

Les paramètres de la syntaxe sont décrits ci-après :

| Option | Description |
| --- | --- |
| Source |Spécifie l’emplacement de la source de données dans l’objet blob Azure Storage. La source peut être un conteneur d’objets blob, un objet blob ou un autre compte Azure Data Lake Storage Gen1. |
| Dest |Spécifie la destination Data Lake Storage Gen1 vers laquelle effectuer la copie. |
| SourceKey |Spécifie la clé d’accès de stockage pour la source d’objet blob Azure Storage. Cela est nécessaire uniquement si la source est un conteneur d’objets blob ou un objet blob. |
| Compte |**Facultatif**. Utilisez ce paramètre si vous souhaitez utiliser un compte Azure Data Lake Analytics pour exécuter la tâche de copie. Si vous utilisez l’option /Account dans la syntaxe, mais sans spécifier de compte Data Lake Analytics, AdlCopy utilise un compte par défaut pour exécuter la tâche. En outre, si vous utilisez cette option, vous devez ajouter la source (Azure Storage Blob) et la destination (Data Lake Storage Gen1) comme sources de données de votre compte Data Lake Analytics. |
| Units |Spécifie le nombre d’unités Data Lake Analytics utilisées pour la tâche de copie. Cette option est obligatoire si vous utilisez l’option **/Account** pour spécifier le compte Data Lake Analytics. |
| Modèle |Spécifie un modèle d’expression régulière qui indique les objets blob ou les fichiers à copier. AdlCopy respecte la casse pour la mise en correspondance. Le modèle par défaut lorsqu’aucun modèle n’est spécifié consiste à copier tous les éléments. La spécification de plusieurs modèles de fichiers n’est pas prise en charge. |

## <a name="use-adlcopy-as-standalone-to-copy-data-from-an-azure-storage-blob"></a>Utilisation d’AdlCopy (en mode autonome) pour copier des données à partir d’un objet blob Azure Storage

1. Ouvrez une invite de commandes et accédez au répertoire où vous avez installé AdlCopy, généralement `%HOMEPATH%\Documents\adlcopy`.
1. Exécutez la commande suivante pour copier un objet blob spécifique du conteneur source vers un dossier Azure Data Lake Storage Gen1 :

    ```console
    AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>
    ```

    Par exemple :

    ```console
    AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/WebsiteLogSampleData/SampleLog/909f2b.log /dest swebhdfs://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==
    ```

    >[!NOTE]
    >La syntaxe ci-dessus spécifie le fichier à copier vers un dossier dans le compte Data Lake Storage Gen1. L’outil de AdlCopy crée un dossier si le nom de dossier spécifié n’existe pas.

    Vous êtes invité à entrer les informations d’identification de l’abonnement Azure sous lequel vous disposez d’un compte Azure Data Lake Storage Gen1. Vous devez voir une sortie similaire à ce qui suit :

    ```output
    Initializing Copy.
    Copy Started.
    100% data copied.
    Finishing Copy.
    Copy Completed. 1 file copied.
    ```

1. Vous pouvez également copier tous les objets blob d’un conteneur donné vers le compte Azure Data Lake Storage Gen1 à l’aide de la commande suivante :

    ```console
    AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/ /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container>  
    ```      

    Par exemple :

    ```console
    AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest adl://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ==
    ```

### <a name="performance-considerations"></a>Considérations relatives aux performances

Si vous copiez à partir d’un compte de stockage Blob Azure, vous pouvez être limité au cours de la copie du côté du stockage d’objets blob. Cela dégradera les performances de votre tâche de copie. Pour en savoir plus sur les limites du stockage Blob Azure, consultez les limites du stockage Azure sur [Abonnement Azure et limites du service](../azure-resource-manager/management/azure-subscription-service-limits.md).

## <a name="use-adlcopy-as-standalone-to-copy-data-from-another-data-lake-storage-gen1-account"></a>Utilisation d’AdlCopy (en mode autonome) pour copier des données à partir d’un autre compte Azure Data Lake Storage Gen1

Vous pouvez également utiliser AdlCopy pour copier des données entre deux Azure Data Lake Storage Gen1.

1. Ouvrez une invite de commandes et accédez au répertoire où vous avez installé AdlCopy, généralement `%HOMEPATH%\Documents\adlcopy`.
1. Exécutez la commande suivante pour copier un fichier spécifique d’un compte Data Lake Storage Gen1 vers un autre.

    ```console
    AdlCopy /Source adl://<source_adlsg1_account>.azuredatalakestore.net/<path_to_file> /dest adl://<dest_adlsg1_account>.azuredatalakestore.net/<path>/
    ```

    Par exemple :

    ```console
    AdlCopy /Source adl://mydatastorage.azuredatalakestore.net/mynewfolder/909f2b.log /dest adl://mynewdatalakestorage.azuredatalakestore.net/mynewfolder/
    ```

   > [!NOTE]
   > La syntaxe ci-dessus spécifie le fichier à copier vers un dossier dans le compte Data Lake Storage Gen1 de destination. L’outil de AdlCopy crée un dossier si le nom de dossier spécifié n’existe pas.
   >
   >

    Vous êtes invité à entrer les informations d’identification de l’abonnement Azure sous lequel vous disposez d’un compte Azure Data Lake Storage Gen1. Vous devez voir une sortie similaire à ce qui suit :

    ```output
    Initializing Copy.
    Copy Started.|
    100% data copied.
    Finishing Copy.
    Copy Completed. 1 file copied.
    ```
1. La commande suivante copie tous les fichiers depuis un dossier spécifique dans le compte Data Lake Storage Gen1 source vers un dossier dans le compte Data Lake Storage Gen1 de destination.

    ```console
    AdlCopy /Source adl://mydatastorage.azuredatalakestore.net/mynewfolder/ /dest adl://mynewdatalakestorage.azuredatalakestore.net/mynewfolder/
    ```

### <a name="performance-considerations"></a>Considérations relatives aux performances

Lorsque vous utilisez AdlCopy comme outil autonome, la copie est exécutée sur les ressources managées par Azure et partagées. Les performances que vous pouvez obtenir dans cet environnement dépendent de la charge du système et des ressources disponibles. Ce mode est idéal pour les petits transferts sur une base ad hoc. Aucun paramètre ne doit être réglé lorsque vous utilisez AdlCopy comme outil autonome.

## <a name="use-adlcopy-with-data-lake-analytics-account-to-copy-data"></a>Utilisation d’AdlCopy (avec un compte Data Lake Analytics) pour copier des données

Vous pouvez également utiliser votre compte Data Lake Analytics pour exécuter la tâche AdlCopy afin de copier les données d’objets blob de Stockage Azure vers Data Lake Storage Gen1. Vous utilisez généralement cette option lorsque les données à déplacer sont situées dans une plage de plusieurs gigaoctets et téraoctets et que vous souhaitez que le débit des performances soit amélioré et prévisible.

Pour utiliser votre compte Data Lake Analytics avec AdlCopy pour copier des données depuis un objet blob Azure Storage, vous devez ajouter la source (objet blob Azure Storage) comme source de données de votre compte Data Lake Analytics. Pour obtenir des instructions sur l’ajout de sources de données à votre compte Data Lake Analytics, consultez [Gérer les sources de donnés d’un compte Data Lake Analytics](../data-lake-analytics/data-lake-analytics-manage-use-portal.md#manage-data-sources).

> [!NOTE]
> Si vous copiez à partir d’un compte Azure Data Lake Storage Gen1 (comme source) à l’aide d’un compte Data Lake Analytics, il est inutile d’associer le compte Data Lake Storage Gen1 au compte Data Lake Analytics. Il n’est nécessaire d’associer le magasin (store) source au compte Data Lake Analytics que lorsque la source est un compte Azure Storage.
>
>

Exécutez la commande suivante pour copier à partir d’un objet blob Azure Storage vers un compte Data Lake Storage Gen1 l’aide du compte Data Lake Analytics :

```console
AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Account <data_lake_analytics_account> /Units <number_of_data_lake_analytics_units_to_be_used>
```

Par exemple :

```console
AdlCopy /Source https://mystorage.blob.core.windows.net/mycluster/example/data/gutenberg/ /dest swebhdfs://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Account mydatalakeanalyticaccount /Units 2
```

De même, exécutez la commande ci-après pour copier tous les fichiers à partir d’un dossier spécifique du compte Data Lake Storage Gen1 source vers un dossier du compte Data Lake Storage Gen1 de destination à l’aide du compte Data Lake Analytics :

```console
AdlCopy /Source adl://mysourcedatalakestorage.azuredatalakestore.net/mynewfolder/ /dest adl://mydestdatastorage.azuredatalakestore.net/mynewfolder/ /Account mydatalakeanalyticaccount /Units 2
```

### <a name="performance-considerations"></a>Considérations relatives aux performances

Lors de la copie de données de l’ordre des téraoctets, l’utilisation d’AdlCopy avec votre propre compte Azure Data Lake Analytics offre des performances meilleures et plus prévisibles. Le paramètre qui doit être réglé est le nombre d’unités d’Azure Data Lake Analytics à utiliser pour la tâche de copie. L’augmentation du nombre d’unités augmente les performances de votre tâche de copie. Chaque fichier copié peut utiliser une unité au plus. Spécifier plus d’unités que le nombre de fichiers copiés n’améliore pas les performances.

## <a name="use-adlcopy-to-copy-data-using-pattern-matching"></a>Utilisation d’AdlCopy pour copier des données à l’aide de critères spéciaux

Dans cette section, vous apprenez à utiliser AdlCopy pour copier des données depuis une source (dans notre exemple ci-dessous, nous utilisons un objet Azure Storage Blob) vers un compte Data Lake Storage Gen1 de destination à l’aide de critères spéciaux. Par exemple, vous pouvez utiliser la procédure ci-dessous pour copier tous les fichiers portant l’extension .csv à partir de l’objet blob source vers la destination.

1. Ouvrez une invite de commandes et accédez au répertoire où vous avez installé AdlCopy, généralement `%HOMEPATH%\Documents\adlcopy`.
1. Exécutez la commande suivante pour copier tous les fichiers portant l’extension *.csv à partir d’un objet blob spécifique du conteneur source vers un dossier Data Lake Storage Gen1 :

    ```console
    AdlCopy /source https://<source_account>.blob.core.windows.net/<source_container>/<blob name> /dest swebhdfs://<dest_adlsg1_account>.azuredatalakestore.net/<dest_folder>/ /sourcekey <storage_account_key_for_storage_container> /Pattern *.csv
    ```

    Par exemple :

    ```console
    AdlCopy /source https://mystorage.blob.core.windows.net/mycluster/HdiSamples/HdiSamples/FoodInspectionData/ /dest adl://mydatalakestorage.azuredatalakestore.net/mynewfolder/ /sourcekey uJUfvD6cEvhfLoBae2yyQf8t9/BpbWZ4XoYj4kAS5Jf40pZaMNf0q6a8yqTxktwVgRED4vPHeh/50iS9atS5LQ== /Pattern *.csv
    ```

## <a name="billing"></a>Facturation

* Si vous utilisez l’outil AdlCopy de manière autonome, vous êtes facturé en fonction des coûts de sortie correspondant au déplacement des données, si le compte Azure Storage source n’est pas situé dans la même région que le compte Data Lake Storage Gen1.
* Si vous utilisez l’outil AdlCopy avec votre compte Data Lake Analytics, les [tarifs de facturation Data Lake Analytics standard](https://azure.microsoft.com/pricing/details/data-lake-analytics/) s’appliquent.

## <a name="considerations-for-using-adlcopy"></a>Considérations sur l’utilisation d’AdlCopy

* AdlCopy (pour la version 1.0.5) prend en charge la copie de données à partir de sources qui contiennent collectivement plusieurs milliers de fichiers et dossiers. Cependant, si vous rencontrez des problèmes lors de la copie d’un jeu de données volumineux, vous pouvez distribuer les fichiers/dossiers dans différents sous-dossiers et utiliser comme source le chemin d’accès à ces sous-dossiers.

## <a name="performance-considerations-for-using-adlcopy"></a>Considérations de performances sur l’utilisation d’AdlCopy

AdlCopy prend en charge la copie de données contenant des milliers de fichiers et dossiers. Toutefois, si vous rencontrez des problèmes lors de la copie d’un jeu de données volumineux, vous pouvez distribuer les fichiers et dossiers dans des sous-dossiers plus petits. AdlCopy est conçu pour les copies ad hoc. Si vous tentez de copier des données sur une base périodique, vous envisagez d’utiliser [Azure Data Factory](../data-factory/connector-azure-data-lake-store.md), qui fournit une gestion complète des opérations de copie.

## <a name="release-notes"></a>Notes de publication

* 1.0.13 : si vous copiez des données vers le même compte Azure Data Lake Storage Gen1 à l’aide de plusieurs commandes AdlCopy, il n’est plus nécessaire de ré-entrer vos informations d’identification pour chaque exécution. Adlcopy met alors en cache ces informations sur plusieurs exécutions.

## <a name="next-steps"></a>Étapes suivantes

* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)
