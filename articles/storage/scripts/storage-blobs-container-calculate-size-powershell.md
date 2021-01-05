---
title: Calculer la taille d’un conteneur d’objets blob avec PowerShell
titleSuffix: Azure Storage
description: Calculez la taille d’un conteneur dans le stockage d’objets blob Azure en effectuant le total des tailles de chacun de ses objets blob.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.devlang: powershell
ms.topic: sample
ms.date: 12/04/2019
ms.author: tamram
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: de275bcca1644750532809b35ae85d954d3cac6d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89076903"
---
# <a name="calculate-the-size-of-a-blob-container-with-powershell"></a>Calculer la taille d’un conteneur d’objets blob avec PowerShell

Ce script calcule la taille d’un conteneur dans le stockage d’objets blob Azure en effectuant le total des tailles des objets blob inclus dans le conteneur.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

> [!IMPORTANT]
> Ce script PowerShell fournit une estimation de taille pour le conteneur et ne doit pas être utilisé pour les calculs de facturation. Pour un script qui calcule la taille du conteneur pour la facturation, consultez [Calculer la taille totale d’un conteneur de stockage d’objets blob](../scripts/storage-blobs-container-calculate-billing-size-powershell.md).

## <a name="sample-script"></a>Exemple de script

[!code-powershell[main](../../../powershell_scripts/storage/calculate-container-size/calculate-container-size.ps1 "Calculate container size")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Exécutez la commande suivante pour supprimer le groupe de ressources, le conteneur et toutes les ressources associées.

```powershell
Remove-AzResourceGroup -Name bloblisttestrg
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes pour calculer la taille du conteneur de stockage d’objets blob. Chaque élément du tableau renvoie à une documentation spécifique de la commande.

| Commande | Notes |
|---|---|
| [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount) | Obtient un compte de stockage spécifié ou tous les comptes de stockage dans un groupe de ressources ou l’abonnement. |
| [Get-AzStorageBlob](/powershell/module/az.storage/Get-AzStorageBlob) | Répertorie les objets blob inclus dans un conteneur. |

## <a name="next-steps"></a>Étapes suivantes

Pour un script qui calcule la taille du conteneur pour la facturation, consultez [Calculer la taille totale d’un conteneur de stockage d’objets blob](../scripts/storage-blobs-container-calculate-billing-size-powershell.md).

Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).

Vous trouverez des exemples de scripts PowerShell de stockage supplémentaires dans les [exemples PowerShell pour Stockage Azure](../blobs/storage-samples-blobs-powershell.md).
