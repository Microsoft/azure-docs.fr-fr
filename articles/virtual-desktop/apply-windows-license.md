---
title: Appliquer la licence Windows pour machines virtuelles hôtes de session – Azure
description: Décrit comment appliquer la licence Windows pour des machines virtuelles Windows Virtual Desktop.
author: ChristianMontoya
ms.topic: how-to
ms.date: 08/14/2019
ms.author: chrimo
ms.openlocfilehash: 5f3749be36f5f035e49fcb862f92180e4902101f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88010138"
---
# <a name="apply-windows-license-to-session-host-virtual-machines"></a>Appliquer la licence Windows pour machines virtuelles hôtes de session

Les clients disposant d’une licence appropriée pour exécuter des charges de travail Windows Virtual Desktop sont éligibles pour appliquer une licence Windows à leurs machines virtuelles hôtes de session et les exécuter sans avoir à payer d’autre licence. Pour plus d’informations, voir [Tarification de Windows Virtual Desktop](https://azure.microsoft.com/pricing/details/virtual-desktop/).

## <a name="ways-to-use-your-windows-virtual-desktop-license"></a>Méthodes d’utilisation de votre licence Windows Virtual Desktop
La gestion des licences Windows Virtual Desktop vous permet d’appliquer une licence à toute machine virtuelle Windows ou Windows Server inscrite en tant qu’hôte de session dans un pool d’hôtes, et recevant des connexions utilisateur. Cette licence ne s’applique pas aux machines virtuelles qui s’exécutent en tant que serveurs de partage de fichiers, contrôleurs de domaine, etc.

Il existe plusieurs façons d’utiliser la licence Windows Virtual Desktop :
- Vous pouvez créer un pool d’hôtes et ses machines virtuelles hôtes de session à l’aide de l’[offre de la Place de marché Azure](./create-host-pools-azure-marketplace.md). La licence est automatiquement appliquée aux machines virtuelles créées de cette manière.
- Vous pouvez créer un pool d’hôtes et ses machines virtuelles hôtes de session à l’aide du [Modèle Azure Resource Manager sur GitHub](./virtual-desktop-fall-2019/create-host-pools-arm-template.md). La licence est automatiquement appliquée aux machines virtuelles créées de cette manière.
- Vous pouvez appliquer une licence à une machine virtuelle hôte de session existante. Pour ce faire, suivez d’abord les instructions fournies dans [Créer un pool d’hôtes avec PowerShell](./create-host-pools-powershell.md) afin de créer un pool d’hôtes et les machines virtuelles associées, puis revenez à cet article pour découvrir comment appliquer la licence.

## <a name="apply-a-windows-license-to-a-session-host-vm"></a>Appliquer une licence Windows à une machine virtuelle hôte de session
Vérifiez que vous avez [installé et configuré la dernière version d’Azure PowerShell](/powershell/azure/). Exécutez la cmdlet PowerShell suivante pour appliquer la licence Windows :

```powershell
$vm = Get-AzVM -ResourceGroup <resourceGroupName> -Name <vmName>
$vm.LicenseType = "Windows_Client"
Update-AzVM -ResourceGroupName <resourceGroupName> -VM $vm
```

## <a name="verify-your-session-host-vm-is-utilizing-the-licensing-benefit"></a>Vérifier que votre machine virtuelle hôte de session utilise l’avantage des licences
Après avoir déployé votre machine virtuelle, exécutez cette cmdlet pour vérifier le type de licence :
```powershell
Get-AzVM -ResourceGroupName <resourceGroupName> -Name <vmName>
```

Une machine virtuelle hôte de session à laquelle la licence Windows est appliquée affiche quelque chose comme ceci :

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Client
```

Les machines virtuelles auxquelles aucune licence Windows n’est appliquée affichent quelque chose comme ceci :

```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

Exécutez la cmdlet suivante pour afficher la liste de toutes les machines virtuelles hôtes de session auxquelles la licence Windows est appliquée dans votre abonnement Azure :

```powershell
$vms = Get-AzVM
$vms | Where-Object {$_.LicenseType -like "Windows_Client"} | Select-Object ResourceGroupName, Name, LicenseType
```
