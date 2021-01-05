---
title: Module PowerShell Windows Virtual Desktop - Azure
description: Comment installer et configurer le module PowerShell pour Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 04/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f2f01e2b58c997db08ad4427de7eef1ee3760c4a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016809"
---
# <a name="set-up-the-powershell-module-for-windows-virtual-desktop"></a>Configurer le module PowerShell pour Windows Virtual Desktop

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec une intégration Azure Resource Manager.

Le module PowerShell pour Windows Virtual Desktop est intégré au module Azure PowerShell. Cet article vous indique comment configurer le module PowerShell afin de pouvoir exécuter des cmdlets pour Windows Virtual Desktop.

## <a name="set-up-your-powershell-environment"></a>Configurer votre environnement PowerShell

Pour commencer à utiliser le module, commencez par installer la [dernière version de PowerShell Core](/powershell/scripting/install/installing-powershell#powershell-core). Actuellement, les cmdlets Windows Virtual Desktop fonctionnent uniquement avec PowerShell Core.

Ensuite, vous devez installer le module DesktopVirtualization à utiliser dans votre session PowerShell.

Exécutez la cmdlet PowerShell suivante en mode élevé pour installer le module :

```powershell
Install-Module -Name Az.DesktopVirtualization
```

>[!NOTE]
> Si cette cmdlet ne fonctionne pas, essayez de l’exécuter à nouveau avec des autorisations élevées.

Ensuite, exécutez la cmdlet suivante pour vous connecter à Azure :

```powershell
Connect-AzAccount
```

>[!IMPORTANT]
>Si vous vous connectez au portail USGov, exécutez cette applet de commande à la place :
> 
> ```powershell
> Connect-AzAccount -EnvironmentName AzureUSGovernment
> ```

La connexion à votre compte Azure nécessite un code qui est généré lorsque vous exécutez la cmdlet Connect. Pour vous connecter, accédez à <https://microsoft.com/devicelogin>, entrez le code, puis connectez-vous à l’aide de vos informations d’identification d’administrateur Azure.

```powershell
Account SubscriptionName TenantId Environment

------- ---------------- -------- -----------

Youradminupn subscriptionname AzureADTenantID AzureCloud
```

Cela vous permet de vous connecter directement à l’abonnement par défaut correspondant à vos informations d’identification d’administrateur.

## <a name="change-the-default-subscription"></a>Changer l’abonnement par défaut

Si vous souhaitez modifier l’abonnement par défaut une fois que vous êtes connecté, exécutez cette cmdlet :

```powershell
Select-AzSubscription -Subscription <preferredsubscriptionname>
```

Vous pouvez également en sélectionner un dans une liste à l’aide de l’applet de commande Out-GridView :

```powershell
Get-AzSubscription | Out-GridView -PassThru | Select-AzSubscription
```

Lorsque vous sélectionnez un nouvel abonnement à utiliser, vous n’avez pas besoin de spécifier l’ID de cet abonnement dans les cmdlets que vous exécutez par la suite. Par exemple, la cmdlet suivante récupère un hôte de session spécifique sans avoir besoin de l’ID d’abonnement :

```powershell
Get-AzWvdSessionHost -HostPoolName <hostpoolname> -Name <sessionhostname> -ResourceGroupName <resourcegroupname>
```

Vous pouvez également modifier des abonnements par cmdlet en ajoutant le nom d’abonnement souhaité en tant que paramètre. La cmdlet suivante est identique à l’exemple précédent, sauf que l’ID d’abonnement a été ajouté en tant que paramètre pour changer l’abonnement utilisé par la cmdlet.

```powershell
Get-AzWvdSessionHost -HostPoolName <hostpoolname> -Name <sessionhostname> -ResourceGroupName <resourcegroupname> -SubscriptionId <subscriptionGUID>
```

## <a name="get-locations"></a>Obtenir des emplacements

Le paramètre d’emplacement est obligatoire pour toutes les cmdlets **New-AzWVD** qui créent des objets.

Exécutez la cmdlet suivante pour obtenir la liste des emplacements pris en charge par votre abonnement :

```powershell
Get-AzLocation
```

La sortie pour **Get-AzLocation** se présente ainsi :

```powershell
Location : eastasia

DisplayName : East Asia

Providers : {Microsoft.RecoveryServices, Microsoft.ManagedIdentity,
Microsoft.SqlVirtualMachine, microsoft.insightsΓÇª}

Location : southeastasia

DisplayName : Southeast Asia

Providers : {Microsoft.RecoveryServices, Microsoft.ManagedIdentity,
Microsoft.SqlVirtualMachine, microsoft.insightsΓÇª}

Location : centralus

DisplayName : Central US

Providers : {Microsoft.RecoveryServices, Microsoft.DesktopVirtualization,
Microsoft.ManagedIdentity, Microsoft.SqlVirtualMachineΓÇª}

Location : eastus

DisplayName : East US

Providers : {Microsoft.RecoveryServices, Microsoft.DesktopVirtualization,
Microsoft.ManagedIdentity, Microsoft.SqlVirtualMachineΓÇª}
```

Une fois que vous connaissez l’emplacement de votre compte, vous pouvez l’utiliser dans une cmdlet. Par exemple, voici une cmdlet qui crée un pool d’hôtes à l’emplacement « southeastasia » :

```powershell
New-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -WorkspaceName <workspacename> -Location “southeastasia”
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez configuré votre module PowerShell, vous pouvez exécuter des cmdlets pour effectuer toutes sortes d’opérations dans Windows Virtual Desktop. Voici quelques-uns des endroits où vous pouvez utiliser votre module :

- Suivez nos [tutoriels sur Windows Virtual Desktop]() pour configurer votre propre environnement Windows Virtual Desktop.
- [Créer un pool d’hôtes avec PowerShell](create-host-pools-powershell.md)
- [Configurer la méthode d’équilibrage de charge de Windows Virtual Desktop](configure-host-pool-load-balancing.md)
- [Configurer le type d’affectation d’un pool d’hôtes de bureau personnel](configure-host-pool-personal-desktop-assignment-type.md)
- Et bien plus !

Si vous rencontrez des problèmes, consultez notre article sur la [résolution des problèmes PowerShell](troubleshoot-powershell.md) pour obtenir de l’aide.

