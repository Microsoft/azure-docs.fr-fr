---
title: Exemple PowerShell - Exportez les applications avec des secrets et des certificats arrivant à expiration vers le locataire Azure Active Directory.
description: Exemple PowerShell qui exporte tous les secrets et certificats arrivant à expiration pour les applications spécifiées vers votre locataire Azure Active Directory.
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: sample
ms.date: 03/09/2021
ms.author: mtillman
ms.reviewer: mifarca
ms.openlocfilehash: 38b7b424e23c196f89004b443b8945d6f7bfe44b
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/14/2021
ms.locfileid: "112076777"
---
# <a name="export-apps-with-expiring-secrets-and-certificates"></a>Exporter les applications avec des secrets et des certificats arrivant à expiration

Cet exemple de script PowerShell exporte toutes les inscriptions d’applications avec des secrets et des certificats arrivant à expiration et leurs propriétaires pour les applications spécifiées, de votre annuaire vers un fichier CSV.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

Cet exemple requiert le [module AzureAD v2 PowerShell pour Graph](/powershell/azure/active-directory/install-adv2) (AzureAD) ou la [préversion du Module AzureAD v2 PowerShell pour Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0-preview&preserve-view=true) (AzureADPreview).

## <a name="sample-script"></a>Exemple de script

[!code-azurepowershell[main](~/powershell_scripts/application-management/export-apps-with-expiring-secrets.ps1 "Exports all apps with expiring secrets and certificates for the specified apps in your directory.")]

## <a name="script-explanation"></a>Explication du script

Le script peut être utilisé directement sans aucune modification. L’administrateur est invité à indiquer la date d’expiration, et s’il souhaite voir les secrets ou certificats ayant déjà expiré.

La commande « Add-Member » est chargée de créer les colonnes dans le fichier CSV.
La commande « New-Object » crée un objet à utiliser pour les colonnes dans l’exportation de fichier CSV.
Vous pouvez modifier la variable « $Path » directement dans PowerShell, avec un chemin de fichier CSV, si vous préférez que l’exportation ne soit pas interactive.

| Commande | Notes |
|---|---|
| [Get-AzureADApplication](/powershell/module/azuread/get-azureadapplication?view=azureadps-2.0&preserve-view=true) | Récupère une application dans votre annuaire. |
| [Get-AzureADApplicationOwner](/powershell/module/azuread/Get-AzureADApplicationOwner?view=azureadps-2.0&preserve-view=true) | Récupère les propriétaires d’une application dans votre annuaire. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le Module Azure AD PowerShell, consultez [Présentation du Module Azure AD PowerShell](/powershell/azure/active-directory/overview).

Pour d’autres exemples PowerShell de gestion d’applications, consultez [Exemples PowerShell pour la gestion d’applications Azure AD](../app-management-powershell-samples.md).
