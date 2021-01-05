---
author: cherylmc
ms.date: 12/06/2019
ms.service: expressroute
ms.topic: include
ms.author: cherylmc
ms.openlocfilehash: f04861d55c9cea3c79f4983f7be2e1f5a3c6d864
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018151"
---
Installez les dernières versions des modules PowerShell Azure Service Management (SM) et du module ExpressRoute. Vous ne pouvez pas utiliser l’environnement Azure CloudShell pour exécuter des modules SM.

1. Suivez les instructions de l’article [Installation du module Service Management](/powershell/azure/servicemanagement/install-azure-ps) pour installer le module Azure Service Management. Si le module Az ou RM est déjà installé, assurez-vous d’utiliser « -AllowClobber ».
2. Importez les modules installés. Lorsque vous utilisez l’exemple suivant, ajustez le chemin pour refléter l’emplacement et la version de vos modules PowerShell installés.

   ```powershell
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.3.0\Azure.psd1'
   Import-Module 'C:\Program Files\WindowsPowerShell\Modules\Azure\5.3.0\ExpressRoute\ExpressRoute.psd1'
   ```
3. Pour vous connecter à votre compte Azure, ouvrez votre console PowerShell avec des droits élevés et connectez-vous à votre compte. Utilisez l’exemple suivant pour vous aider à vous connecter à l’aide du module Service Management :

   ```powershell
   Add-AzureAccount
   ```