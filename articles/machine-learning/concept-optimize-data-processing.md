---
title: Optimiser le traitement des données
titleSuffix: Azure Machine Learning
description: Découvrez les meilleures pratiques pour l’optimisation des vitesses de traitement de données et les intégrations prises en charge par Azure Machine Learning pour le traitement de données à grande échelle.
services: machine-learning
ms.service: machine-learning
ms.author: sgilley
author: sdgilley
ms.subservice: core
ms.reviewer: nibaccam
ms.topic: conceptual
ms.date: 06/26/2020
ms.custom: data4ml
ms.openlocfilehash: 77edac14ef13901725eed656835e1a937d4f4ddf
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93360817"
---
# <a name="optimize-data-processing-with-azure-machine-learning"></a>Optimiser le traitement des données avec Azure Machine Learning

Dans cet article, vous allez découvrir les meilleures pratiques pour vous aider à optimiser les vitesses de traitement des données, localement et à grande échelle.

Azure Machine Learning est intégré aux packages open source et aux infrastructures pour le traitement des données. Vous pouvez améliorer les vitesses de traitement de données localement et à grande échelle grâce à l’utilisation de ces intégrations et à la mise en application des suggestions des meilleures pratiques citées dans cet article.

## <a name="parquet-and-csv-file-formats"></a>Formats de fichiers CSV et Parquet

Les fichiers de valeurs séparées par des virgules (CSV) sont des formats de fichiers courants dans le cadre du traitement de données. Toutefois, les formats de fichier Parquet sont recommandés pour des tâches de Machine Learning.

[Les fichiers Parquet](https://parquet.apache.org/) stockent les données dans un format binaire en colonnes. Ce format est utile si le fractionnement des données en plusieurs fichiers est nécessaire. De plus, ce format vous permet de cibler le champ approprié pour vos essais de Machine Learning. Au lieu de devoir lire un fichier de données de 20 Go, vous pouvez réduire cette charge de données en sélectionnant les colonnes nécessaires pour entraîner votre modèle de Machine Learning. Les fichiers Parquet peuvent également être compressés afin de réduire la puissance de traitement et réduire l’espace occupé.

Les fichiers CSV sont couramment utilisés pour importer et exporter des données, car ils sont faciles à modifier et à lire dans Excel. Les données dans des fichiers CSV sont stockées en tant que chaînes sous forme de lignes et les fichiers peuvent être compressés afin d’alléger les charges de transfert de données. Il est possible d’étendre les CSV par un facteur d’environ 2 à 10 et les CSV compressés peuvent avoir une expansion encore plus importante. Et ce, dans le but que le volume partagé de CSV de 5 Go dans la mémoire s’étende correctement sur les 8 Go de RAM dont vous disposez sur votre ordinateur. Ce comportement de compression est susceptible d’augmenter la latence du transfert de données, ce qui n’est pas l’idéal si vous avez de grandes quantités de données à traiter. 

## <a name="pandas-dataframe"></a>Tramedonnées Pandas

Les [dataframes Pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/overview.html) sont couramment utilisés pour la manipulation et l’analyse des données. `Pandas` est adapté aux tailles de données inférieures à 1 Go, mais les durées de traitement des dataframes `pandas` ralentissent lorsque la taille des fichiers atteint environ 1 Go. Ce ralentissement s’explique par la différence entre la taille de vos données dans le stockage et la taille des données dans un dataframe. Par exemple, les données des fichiers CSV peuvent décupler de volume dans un dataframe ; ainsi, un fichier CSV de 1 Go peut occuper 10 Go dans un dataframe.

`Pandas` est mono-thread, ce qui signifie que les opérations sont effectuées une par une sur un seul processeur. Il est possible de mettre en parallèle les charges de travail avec plusieurs processeurs virtuels sur une seule instance de calcul Azure Machine Learning à l’aide de packages, tels que [Modin](https://modin.readthedocs.io/en/latest/), qui inclut `Pandas` à l’aide d’un serveur principal distribué.

Pour mettre vos tâches en parallèle avec `Modin` et [Dask](https://dask.org), remplacez simplement cette ligne de code `import pandas as pd` par `import modin.pandas as pd`.

## <a name="dataframe-out-of-memory-error"></a>Dataframe : erreur de mémoire insuffisante 

Généralement, une erreur de *mémoire insuffisante* se produit lorsqu’un dataframe s’étend au de-là de la RAM disponible sur votre machine. Ce concept s’applique également à une architecture distribuée, telle que `Modin` ou `Dask`.  Autrement dit, votre opération tente de charger le dataframe en mémoire sur chaque nœud de votre cluster, mais la RAM disponible n’est pas suffisante pour cela.

Augmenter la RAM pour y faire tenir le dataframe est une solution. Nous vous recommandons d’utiliser une taille de calcul et une puissance de traitement représentant le double de la taille de RAM. Ainsi, si votre dataframe est de 10 Go, utilisez la cible de calcul avec au moins 20 Go de RAM pour que le dataframe tienne aisément dans la mémoire et soit traitée. 

Pour plusieurs processeurs virtuels, gardez à l’esprit qu’une partition est nécessaire pour contenir la RAM allouée à chaque processeur virtuel sur la machine. Autrement dit, si vous avez 16 Go de RAM pour 4 processeurs virtuels, vous aurez besoin d'environ 2 Go de dataframe pour chaque processeur virtuel.

### <a name="local-vs-remote"></a>Local vs distant

Vous remarquerez peut-être que certaines commandes de dataframe pandas s'exécutent plus rapidement sur votre PC local que sur une machine virtuelle distante que vous avez approvisionnée avec Azure Machine Learning. Un fichier d'échange est généralement activé sur votre PC local, ce qui vous permet de charger plus de données que ne peut en contenir la mémoire physique ; autrement dit, votre disque dur est utilisé comme extension de votre RAM. Actuellement, les machines virtuelles Azure Machine Learning s'exécutent sans fichier d'échange et ne peuvent donc charger que la quantité de données correspondant à la RAM physique disponible. 

Pour les travaux de calcul intensif, nous vous recommandons de choisir une machine virtuelle de plus grande taille afin d'améliorer la vitesse de traitement.

Découvrez-en plus sur les [séries et tailles de machines virtuelles disponibles](concept-compute-target.md#supported-vm-series-and-sizes) pour Azure Machine Learning. 

Concernant les spécifications de la RAM, consultez les pages des séries de machines virtuelles correspondantes, comme la [série Dv2-Dsv2](../virtual-machines/dv2-dsv2-series-memory.md) ou la [série NC](../virtual-machines/nc-series.md).

### <a name="minimize-cpu-workloads"></a>Réduire les charges de travail de l’UC

Si vous ne pouvez pas augmenter la RAM de votre ordinateur, vous pouvez appliquer les techniques suivantes pour réduire les charges de travail de l’UC et optimiser les temps de traitement. Ces suggestions concernent aussi bien les systèmes uniques que les systèmes distribués.

Technique | Description
----|----
Compression | Utilisez une représentation différente pour vos données, d’une manière qui utilise moins de mémoire et qui n’a pas d’impact significatif sur les résultats de votre calcul.<br><br>*Exemple :* Au lieu de stocker des entrées sous la forme d’une chaîne d’environ 10 octets ou plus par entrée, stockez-les sous la forme d’une valeur booléenne, True ou False, qu’il est possible de stocker dans 1 octet.
Segmentation | Chargez des données en mémoire dans des sous-ensembles (blocs), en traitant les données un sous-ensemble à la fois ou plusieurs sous-ensembles en parallèle. Cette méthode fonctionne mieux si vous devez traiter toutes les données, mais ne devez pas charger toutes les données en mémoire de façon simultanée. <br><br>*Exemple :* Au lieu de traiter une année complète de données à la fois, chargez et traitez les données un mois à la fois.
Indexation | Appliquez et utilisez un index, soit un résumé qui vous indique où trouver les données qui vous intéressent. L’indexation est utile lorsque vous devez simplement utiliser un sous-ensemble de données plutôt que l’ensemble complet<br><br>*Exemple :* Si vous avez une année complète de données de ventes triées par mois, un index vous permet de rechercher rapidement le mois que vous souhaitez traiter.

## <a name="scale-data-processing"></a>Traitement des données à l’échelle

Si les suggestions précédentes ne suffisent pas et que vous n’arrivez pas à obtenir une machine virtuelle adaptée à vos données, il vous est possible d’ 

* utiliser un dataframe, tel que `Spark` ou `Dask`, pour traiter les données « mémoire insuffisante ». Dans cette option, le dataframe est chargé dans la RAM, partition après partition, puis traité, le résultat étant collecté à la fin.  

* Effectuez un scale-out sur un cluster en utilisant une architecture distribuée. Dans cette option, les charges de traitement de données sont divisées et traitées sur plusieurs UC qui travaillent en parallèle, le résultat final étant collecté à la fin.

### <a name="recommended-distributed-frameworks"></a>Architectures distribuées recommandées

Le tableau suivant recommande des architectures distribuées intégrées à Azure Machine Learning en fonction de votre préférence de code ou de la taille des données.

Expérience ou taille des données | Recommandation
------|------
Si vous maîtrisez `Pandas`| Dataframe `Modin` ou `Dask`
Si vous préférez `Spark` | `PySpark`
Pour moins de 1 Go de données | `Pandas` en local **ou** une instance de calcul Azure Machine Learning à distance
Pour plus de 10 Go de données| Déplacer vers un cluster à l’aide de `Ray`, `Dask` ou `Spark`

Vous pouvez créer des clusters `Dask` sur un cluster de calcul Azure ML à l’aide du package [dask-cloudprovider](https://cloudprovider.dask.org/en/latest/#azure). Vous pouvez également exécuter `Dask` localement sur une instance de calcul.

## <a name="next-steps"></a>Étapes suivantes

* [Options d’ingestion des données avec Azure Machine Learning](concept-data-ingestion.md).
* [Créer et inscrire des jeux de données](how-to-create-register-datasets.md).
