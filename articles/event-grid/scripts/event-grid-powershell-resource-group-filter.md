---
title: Azure PowerShell - S’abonner à un groupe de ressources
description: Cet article fournit un exemple de script Azure PowerShell qui montre comment s’abonner à des événements Event Grid pour un groupe de ressources et filtrer pour trouver une ressource.
ms.devlang: powershell
ms.topic: sample
ms.date: 07/08/2020
ms.openlocfilehash: f3aeec208ba6b80643e50307b89f590977bfb446
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87460760"
---
# <a name="subscribe-to-events-for-a-resource-group-and-filter-for-a-resource-with-powershell"></a>S’inscrire aux événements d’un groupe de ressources et filtrer pour trouver une ressource avec PowerShell

Ce script crée un abonnement Event Grid aux événements d’un groupe de ressources. Il utilise un filtre pour obtenir uniquement les événements d’une ressource spécifiée dans le groupe de ressources.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script---stable"></a>Exemple de script : stable

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/event-grid/filter-events/filter-events.ps1 "Filter events")]

## <a name="sample-script---preview-module"></a>Exemple de script - module en préversion

[!INCLUDE [requires-azurerm](../../../includes/requires-azurerm.md)]

L’exemple de script en préversion requiert le module Event Grid. Pour l’installer, exécutez `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`

[!code-powershell[main](../../../powershell_scripts/event-grid/filter-events-preview/filter-events-preview.ps1 "Filter events")]

## <a name="script-explanation"></a>Explication du script

Ce script utilise la commande suivante pour créer l’abonnement aux événements. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzEventGridSubscription](/powershell/module/az.eventgrid/new-azeventgridsubscription) | Créer un abonnement Event Grid. |

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation des applications gérées, consultez [Vue d’ensemble des applications gérées Azure](../overview.md).
* Pour plus d’informations sur PowerShell, consultez la [documentation Azure PowerShell](/powershell/azure/get-started-azureps).
