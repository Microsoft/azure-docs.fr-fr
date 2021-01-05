---
title: Partage de fichiers Azure pour des pools Azure Batch
description: Guide pratique pour monter un partage Azure Files à partir de nœuds de calcul dans un pool Linux ou Windows dans Azure Batch.
ms.topic: how-to
ms.date: 05/24/2018
ms.openlocfilehash: 6bbaba20512a17de563e74ba095057c5c3f71f6b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87986421"
---
# <a name="use-an-azure-file-share-with-a-batch-pool"></a>Utiliser un partage de fichiers Azure avec un pool Batch

[Azure Files](../storage/files/storage-files-introduction.md) offre des partages de fichiers managés dans le cloud qui sont accessibles par le biais du protocole SMB (Server Message Block). Cet article fournit des informations et des exemples de code sur le montage et l’utilisation d’un partage de fichiers Azure sur des nœuds de calcul de pool.

## <a name="considerations-for-use-with-batch"></a>Considérations relatives à l’utilisation de Batch

* Si vous utilisez Azure Files non Premium, envisagez d’utiliser un partage de fichiers Azure quand vous avez des pools qui exécutent un nombre relativement faible de tâches en parallèle. Passez en revue les [objectifs de performance et d’extensibilité](../storage/files/storage-files-scale-targets.md) pour déterminer si Azure Files (qui utilise un compte Stockage Azure) doit être utilisé étant donné la taille attendue de votre pool et le nombre de fichiers d’actifs. 

* Les partages de fichiers Azure sont [rentables](https://azure.microsoft.com/pricing/details/storage/files/) et redondants à l’échelle mondiale car ils peuvent être configurés avec la réplication des données d’une autre région. 

* Vous pouvez monter un partage de fichiers Azure simultanément à partir d’un ordinateur local. Toutefois, veillez à bien comprendre les [implications de la concurrence](../storage/common/storage-concurrency.md), en particulier lors de l’utilisation d’API REST.

* Consultez également les [considérations de planification](../storage/files/storage-files-planning.md) générales pour les partages de fichiers Azure.


## <a name="create-a-file-share"></a>Créer un partage de fichiers

[Créez un partage de fichiers](../storage/files/storage-how-to-create-file-share.md) dans un compte de stockage lié à votre compte Batch, ou dans un compte de stockage distinct.

## <a name="mount-an-azure-file-share-on-a-batch-pool"></a>Monter un partage de fichiers Azure sur un pool Batch

Reportez-vous à la documentation sur la façon de [monter un système de fichiers virtuel sur un pool Batch](virtual-file-mount.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour d’autres options de lecture et d’écriture de données dans Batch, consultez [Conserver les résultats des tâches et des travaux](batch-task-output.md).
* Consultez également la boîte à outils [Batch Shipyard](https://github.com/Azure/batch-shipyard), qui contient des [recettes Shipyard](https://github.com/Azure/batch-shipyard/tree/master/recipes) afin de déployer des systèmes de fichiers pour des charges de travail de conteneur Batch.
