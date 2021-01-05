---
title: Ingestion de données Azure HPC Cache - msrsync
description: Comment utiliser msrsync pour déplacer des données vers une cible de stockage Blob dans Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 10/30/2019
ms.author: v-erkel
ms.openlocfilehash: 323ecd6a2dd001c3c8df1b2ec15f0ae8402ec70c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87092403"
---
# <a name="azure-hpc-cache-data-ingest---msrsync-method"></a>Ingestion de données Azure HPC Cache - msrsync

Cet article fournit des instructions détaillées sur l’utilisation de l’utilitaire ``msrsync`` pour copier des données vers un conteneur de stockage Blob Azure en vue d’une utilisation avec Azure HPC Cache.

Pour plus d’informations sur le déplacement des données vers le stockage Blob pour Azure HPC Cache, consultez [Déplacer les données vers le stockage Blob Azure](hpc-cache-ingest.md).

L’outil ``msrsync`` peut être utilisé pour déplacer des données vers une cible de stockage back-end pour Azure HPC Cache. Cet outil est conçu pour optimiser l’utilisation de la bande passante en exécutant plusieurs processus ``rsync`` parallèles. Il est disponible à partir de GitHub sur https://github.com/jbd/msrsync.

``msrsync`` divise le répertoire source en « compartiments » distincts, puis exécute des processus ``rsync`` individuels sur chaque compartiment.

Des tests préliminaires à l’aide d’une machine virtuelle à quatre cœurs ont montré plus d’efficacité lors de l’utilisation de 64 processus. Utilisez l’option ``msrsync````-p`` pour définir le nombre de processus sur 64.

Notez que ``msrsync`` peut uniquement écrire vers et depuis des volumes locaux. La source et la destination doivent être accessibles en tant que montages locaux sur la station de travail qui est utilisée pour émettre la commande.

Suivez les instructions suivantes pour utiliser ``msrsync`` afin de remplir de données le stockage Blob Azure avec Azure HPC Cache :

1. Installez ``msrsync`` et ses composants requis (``rsync`` et Python 2.6 ou version ultérieure)
1. Déterminez le nombre total de fichiers et répertoires à copier.

   Par exemple, utilisez l’utilitaire ``prime.py`` avec des arguments ```prime.py --directory /path/to/some/directory``` (disponibles en téléchargeant <https://github.com/Azure/Avere/blob/master/src/clientapps/dataingestor/prime.py>).

   Si vous n’utilisez pas ``prime.py``, vous pouvez calculer le nombre d’éléments avec l’outil GNU ``find`` de la façon suivante :

   ```bash
   find <path> -type f |wc -l         # (counts files)
   find <path> -type d |wc -l         # (counts directories)
   find <path> |wc -l                 # (counts both)
   ```

1. Divisez le nombre d’éléments par 64 pour déterminer le nombre d’éléments par processus. Utilisez ce nombre avec l’option ``-f`` pour définir la taille des compartiments quand vous exécutez la commande.

1. Émettez la commande ``msrsync`` pour copier les fichiers :

   ```bash
   msrsync -P --stats -p64 -f<ITEMS_DIV_64> --rsync "-ahv --inplace" <SOURCE_PATH> <DESTINATION_PATH>
   ```

   Par exemple, cette commande est conçue pour déplacer 11 000 fichiers dans 64 processus de /test/source-repository vers /mnt/hpccache/repository :

   ``mrsync -P --stats -p64 -f170 --rsync "-ahv --inplace" /test/source-repository/ /mnt/hpccache/repository``
