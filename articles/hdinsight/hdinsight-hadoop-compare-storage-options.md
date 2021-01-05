---
title: Comparer les options de stockage à utiliser avec les clusters Azure HDInsight
description: Fournit une vue d’ensemble des types de stockage et de leur fonctionnement avec Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: seoapr2020
ms.date: 04/21/2020
ms.openlocfilehash: 34eeeed2b3c44336cd4aa1219d54b1811c6988f5
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94952316"
---
# <a name="compare-storage-options-for-use-with-azure-hdinsight-clusters"></a>Comparer les options de stockage à utiliser avec les clusters Azure HDInsight

Vous pouvez choisir parmi plusieurs services de stockage Azure lors de la création de clusters HDInsight :

* [Stockage Blob Azure avec HDInsight](./overview-azure-storage.md)
* [Azure Data Lake Storage Gen2 avec HDInsight](./overview-data-lake-storage-gen2.md)
* [Azure Data Lake Storage Gen1 avec HDInsight](./overview-data-lake-storage-gen1.md)

Cet article fournit une vue d’ensemble de ces types de stockage, ainsi que de leurs caractéristiques uniques.

## <a name="storage-types-and-features"></a>Types de stockage et fonctionnalités

Le tableau suivant récapitule les services de stockage Azure pris en charge avec les différentes versions de HDInsight :

| Service de stockage | Type de compte | Type d’espace de noms | Services pris en charge | Niveaux de performances pris en charge | Niveaux d’accès pris en charge | Version de HDInsight | Type de cluster |
|---|---|---|---|---|---|---|---|
|Azure Data Lake Storage Gen2| Universel v2 | Hiérarchique (système de fichiers) | Objet blob | standard | Chaud, froid, archive | 3.6+ | Tout sauf Spark 2.1 et 2.2|
|Stockage Azure| Universel v2 | Object | Objet blob | standard | Chaud, froid, archive | 3.6+ | Tous |
|Stockage Azure| Universel v1 | Object | Objet blob | standard | N/A | Tous | Tous |
|Stockage Azure| Stockage Blob** | Object | Objet blob de blocs | standard | Chaud, froid, archive | Tous | Tous |
|Azure Data Lake Storage Gen1| N/A | Hiérarchique (système de fichiers) | N/A | N/A | N/A | 3.6 uniquement | Tout sauf HBase |
|Stockage Azure| Objet blob de blocs| Object | Objet blob de blocs | Premium | N/A| 3.6+ | HBase uniquement avec écritures accélérées|
|Azure Data Lake Storage Gen2| Objet blob de blocs| Hiérarchique (système de fichiers) | Objet blob de blocs | Premium | N/A| 3.6+ | HBase uniquement avec écritures accélérées|

** Pour les clusters HDInsight, seuls les comptes de stockage secondaires peuvent être de type BlobStorage. Par ailleurs, les objets blob de pages ne font pas partie des options de stockage prises en charge.

Pour plus d’informations sur les types de comptes de stockage Azure, consultez l’article [Vue d’ensemble des comptes de stockage Azure](../storage/common/storage-account-overview.md).

Pour plus d’informations sur les niveaux d’accès du stockage Azure, consultez [Stockage Blob Azure : niveaux de stockage Premium (préversion), chaud, froid et archive](../storage/blobs/storage-blob-storage-tiers.md)

Vous pouvez créer des clusters à l’aide de combinaisons de services pour le stockage principal et le stockage secondaire facultatif. Le tableau suivant résume les configurations de stockage de cluster actuellement prises en charge dans HDInsight :

| Version de HDInsight | Stockage principal | Stockage secondaire | Prise en charge |
|---|---|---|---|
| 3.6 et 4.0 | Universel V1, Universel V2 | Universel V1, Universel V2, BlobStorage (objets blob de blocs) | Oui |
| 3.6 et 4.0 | Universel V1, Universel V2 | Data Lake Storage Gen2 | Non |
| 3.6 et 4.0 | Data Lake Storage Gen2* | Data Lake Storage Gen2 | Oui |
| 3.6 et 4.0 | Data Lake Storage Gen2* | Universel V1, Universel V2, BlobStorage (objets blob de blocs) | Oui |
| 3.6 et 4.0 | Data Lake Storage Gen2 | Data Lake Storage Gen 1 | Non |
| 3.6 | Data Lake Storage Gen 1 | Data Lake Storage Gen 1 | Oui |
| 3.6 | Data Lake Storage Gen 1 | Universel V1, Universel V2, BlobStorage (objets blob de blocs) | Oui |
| 3.6 | Data Lake Storage Gen 1 | Data Lake Storage Gen2 | Non |
| 4.0 | Data Lake Storage Gen 1 | Quelconque | Non |
| 4.0 | Universel V1, Universel V2 | Data Lake Storage Gen 1 | Non |

* = Il peut s’agir d’un ou plusieurs Data Lake Storage Gen2, à condition qu’ils soient tous configurés pour utiliser la même identité managée pour l’accès au cluster.

> [!NOTE]
> Le stockage principal Data Lake Storage Gen2 n’est pas pris en charge pour les clusters Spark 2.1 ou 2.2.

## <a name="data-replication"></a>Réplication des données

Azure HDInsight ne stocke pas les données client. Les principaux moyens de stockage d’un cluster sont les comptes de stockage qui lui sont associés. Vous pouvez associer votre cluster à un compte de stockage existant ou créer un compte de stockage pendant le processus de création du cluster. Si un nouveau compte est créé, il est créé en tant que compte de stockage localement redondant (LRS) et répond aux exigences en matière de résidence des données dans la région, notamment celles spécifiées dans [Centre de confidentialité Azure](https://azuredatacentermap.azurewebsites.net).

Vous pouvez vérifier que HDInsight est correctement configuré pour stocker les données dans une seule région en veillant à ce que le compte de stockage associé à votre compte HDInsight soit LRS ou une autre option de stockage mentionnée sur le site de [Centre de confidentialité Azure](https://azuredatacentermap.azurewebsites.net).
 
## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de Stockage Azure dans HDInsight](./overview-azure-storage.md)
* [Vue d’ensemble d’Azure Data Lake Storage Gen1 dans HDInsight](./overview-data-lake-storage-gen1.md)
* [Vue d’ensemble d’Azure Data Lake Storage Gen2 dans HDInsight](./overview-data-lake-storage-gen2.md)
* [Présentation d’Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md)
* [Introduction à Azure Storage](../storage/common/storage-introduction.md)
