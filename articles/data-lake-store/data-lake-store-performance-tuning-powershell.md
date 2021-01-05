---
title: Optimisation des performances d’Azure Data Lake Storage Gen1 - PowerShell
description: Conseils en matière d’amélioration des performances lors de l’utilisation d’Azure PowerShell avec Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 01/09/2018
ms.author: twooley
ms.openlocfilehash: 4ac2bbb21fd1a987d544a536d0f52628824e0bf4
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97723794"
---
# <a name="performance-tuning-guidance-for-using-powershell-with-azure-data-lake-storage-gen1"></a>Recommandations en matière d’optimisation des performances pour l’utilisation de PowerShell avec Azure Data Lake Storage Gen1

Cet article décrit les propriétés que vous pouvez ajuster pour optimiser les performances lors de l’utilisation de PowerShell avec Data Lake Storage Gen1.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="performance-related-properties"></a>Propriétés associées aux performances

| Propriété            | Default | Description |
|---------------------|---------|-------------|
| PerFileThreadCount  | 10      | Ce paramètre vous permet de choisir le nombre de threads parallèles pour charger ou télécharger chaque fichier. Cette valeur représente le nombre maximal de threads pouvant être alloués par fichier, mais il est possible que vous obteniez un nombre de threads inférieur en fonction du scénario (par exemple, si vous chargez un fichier de 1 Ko, vous n’obtenez qu’un seul thread, même si vous en demandez 20).  |
| ConcurrentFileCount | 10      | Ce paramètre est spécifique au chargement ou au téléchargement des dossiers. Il détermine le nombre de fichiers simultanés pouvant être chargés ou téléchargés. Cette valeur représente le nombre maximal de fichiers simultanés pouvant être chargés ou téléchargés en une seule opération, mais il se peut que vous en obteniez moins en fonction du scénario (par exemple, si vous chargez deux fichiers, vous obtenez deux chargements de fichiers simultanés, même si vous en demandez 15). |

**Exemple :**

Cette commande télécharge les fichiers à partir de Data Lake Storage Gen1 sur le disque local de l’utilisateur à l’aide de 20 threads par fichier et 100 fichiers simultanés.

```PowerShell
Export-AzDataLakeStoreItem -AccountName "Data Lake Storage Gen1 account name" `
    -PerFileThreadCount 20 `
    -ConcurrentFileCount 100 `
    -Path /Powershell/100GB `
    -Destination C:\Performance\ `
    -Force `
    -Recurse
```

## <a name="how-to-determine-property-values"></a>Comment déterminer les valeurs de propriété

La question suivante que vous pouvez vous poser porte sur la procédure permettant de déterminer la valeur à fournir pour les propriétés associées aux performances. Voici quelques conseils à suivre.

* **Étape 1 : Déterminer le nombre total de threads** : commencez par calculer le nombre total de threads à utiliser. En règle générale, vous devez utiliser six threads pour chaque noyau physique.

    `Total thread count = total physical cores * 6`

    **Exemple :**

    Supposons que vous exécutez les commandes PowerShell à partir d’une machine virtuelle D14 avec 16 noyaux

    `Total thread count = 16 cores * 6 = 96 threads`

* **Étape 2 : Calculer PerFileThreadCount** : nous calculons PerFileThreadCount en fonction de la taille des fichiers. Pour les fichiers inférieurs à 2,5 Go, il est inutile de modifier ce paramètre, car la valeur par défaut de 10 est suffisante. Pour les fichiers supérieurs à 2,5 Go, vous devez utiliser 10 threads en tant que base pour les 2,5 premiers Go, puis ajouter 1 thread chaque fois que la taille du fichier augmente de 256 Mo. Si vous copiez un dossier contenant différentes tailles de fichiers, envisagez de regrouper ces fichiers par tailles similaires. Les différentes tailles de fichiers ne permettent pas d’obtenir des performances optimales. S’il n’est pas possible de regrouper les tailles de fichiers similaires, vous devez définir PerFileThreadCount en fonction de la plus grande taille de fichier.

    `PerFileThreadCount = 10 threads for the first 2.5 GB + 1 thread for each additional 256 MB increase in file size`

    **Exemple :**

    Supposons que vous disposiez de 100 fichiers de 1 à 10 Go. Pour l’équation, nous utilisons la taille de fichier la plus grande, 10 Go, qui se lit comme suit.

    `PerFileThreadCount = 10 + ((10 GB - 2.5 GB) / 256 MB) = 40 threads`

* **Étape 3 : Calculer ConcurrentFilecount** : utilisez le nombre total de threads et PerFileThreadCount pour calculer ConcurrentFileCount sur la base de l’équation suivante :

    `Total thread count = PerFileThreadCount * ConcurrentFileCount`

    **Exemple :**

    Basé sur les exemples de valeurs utilisés

    `96 = 40 * ConcurrentFileCount`

    Ainsi, **ConcurrentFileCount** vaut **2.4**, que nous pouvons arrondir à **2**.

## <a name="further-tuning"></a>Paramétrage supplémentaire

Il se peut qu’un paramétrage supplémentaire soit requis en raison des différentes tailles de fichiers à utiliser. Le calcul précédent fonctionne bien si l’ensemble ou la plupart des fichiers sont plus volumineux et plus proches de la plage de 10 Go. Par contre, s’il y a beaucoup de tailles de fichiers différentes et que la plupart des fichiers sont parmi les plus petits, vous pouvez réduire PerFileThreadCount. En réduisant PerFileThreadCount, nous pouvons augmenter ConcurrentFileCount. Par conséquent, si nous partons du principe que la plupart des fichiers sont plus petits (dans la plage de 5 Go), nous pouvons refaire nos calculs :

`PerFileThreadCount = 10 + ((5 GB - 2.5 GB) / 256 MB) = 20`

Ainsi, **ConcurrentFileCount** devient égal à 96/20, soit 4,8, arrondi à **4**.

Vous pouvez continuer à ajuster ces paramètres en augmentant et en diminuant le nombre **PerFileThreadCount** en fonction de la répartition des tailles de fichiers.

### <a name="limitation"></a>Limitation

* **Le nombre de fichiers est inférieur à ConcurrentFileCount** : Si le nombre de fichiers chargés est inférieur à la valeur **ConcurrentFileCount** calculée, vous devez réduire la valeur **ConcurrentFileCount** pour qu’elle soit égale au nombre de fichiers. Vous pouvez utiliser les threads restants pour augmenter **PerFileThreadCount**.

* **Trop de threads** : Si vous augmentez trop le nombre de threads sans augmenter la taille du cluster, vous courez le risque d’une diminution des performances. Il peut y avoir des problèmes de conflit lors du changement de contexte sur le processeur.

* **Accès concurrentiels insuffisants** : Si les accès concurrentiels sont insuffisants, le cluster risque d’être trop petit. Vous pouvez augmenter le nombre de nœuds dans votre cluster pour autoriser plus d’accès simultanés.

* **Erreurs de limitation** : Il se peut que vous rencontriez des erreurs de limitation si le nombre d’accès concurrentiels est trop élevé. En cas d’erreurs de limitation, vous devez réduire le nombre d’accès simultanés ou nous contacter.

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser Azure Data Lake Storage Gen1 pour le Big Data](data-lake-store-data-scenarios.md) 
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)
* [Utiliser Azure Data Lake Analytics avec Data Lake Storage Gen1](../data-lake-analytics/data-lake-analytics-get-started-portal.md)
* [Utiliser Azure HDInsight avec Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md)

