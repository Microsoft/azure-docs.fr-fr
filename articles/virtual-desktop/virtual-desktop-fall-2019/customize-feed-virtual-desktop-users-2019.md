---
title: Personnaliser le flux pour les utilisateurs de Windows Virtual Desktop (classique) – Azure
description: Comment personnaliser le flux pour les utilisateurs de Windows Virtual Desktop (classique) avec des cmdlets PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: cd7496690ec88fbe4297386c32d1b8a2c3234577
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91540759"
---
# <a name="customize-feed-for-windows-virtual-desktop-classic-users"></a>Personnaliser le flux pour les utilisateurs de Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../customize-feed-for-virtual-desktop-users.md).

Vous pouvez personnaliser le flux pour que les ressources d’application distante et de Bureau à distance apparaissent de façon reconnaissable pour vos utilisateurs.

Tout d’abord, si vous ne l’avez pas déjà fait, [téléchargez et importez le module PowerShell Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) à utiliser dans votre session PowerShell. Exécutez ensuite l’applet de commande suivante pour vous connecter à votre compte :

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="customize-the-display-name-for-a-remoteapp"></a>Personnaliser le nom d’affichage pour une application distante

Vous pouvez changer le nom d’affichage pour une application distante publiée en définissant le nom convivial. Par défaut, le nom convivial est le même que celui du programme RemoteApp.

Pour récupérer une liste des applications distantes publiées pour un groupe d’applications, exécutez l’applet de commande PowerShell suivante :

```powershell
Get-RdsRemoteApp -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname>
```

> [!div class="mx-imgBorder"]
> ![Capture d'écran de la cmdlet PowerShell Get-RDSRemoteApp, dans laquelle les noms Name et FriendlyName sont mis en surbrillance pour personnaliser le nom d'affichage.](../media/get-rdsremoteapp.png)

Pour affecter un nom convivial à une application distante, exécutez l’applet de commande PowerShell suivante :

```powershell
Set-RdsRemoteApp -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname> -Name <existingappname> -FriendlyName <newfriendlyname>
```

> [!div class="mx-imgBorder"]
> ![Capture d'écran de la cmdlet PowerShell Set-RDSRemoteApp, dans laquelle les noms Name et New FriendlyName sont mis en surbrillance pour personnaliser le nom d'affichage.](../media/set-rdsremoteapp.png)

## <a name="customize-the-display-name-for-a-remote-desktop"></a>Personnaliser le nom d’affichage pour un Bureau à distance

Vous pouvez changer le nom d’affichage pour un Bureau à distance publié en définissant un nom convivial. Si vous avez créé manuellement un pool d’hôtes et un groupe d’applications de Bureau via PowerShell, le nom convivial par défaut est « Bureau de session ». Si vous avez créé un pool d’hôtes et un groupe d’applications de Bureau via le modèle Azure Resource Manager de GitHub ou l’offre de la Place de marché Azure, le nom convivial par défaut est le même que le nom du pool d’hôtes.

Pour récupérer la ressource Bureau à distance, exécutez l’applet de commande PowerShell suivante :

```powershell
Get-RdsRemoteDesktop -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname>
```

> [!div class="mx-imgBorder"]
> ![Capture d’écran de l’applet de commande PowerShell Get-RDSRemoteApp avec le nom et le nom convivial en évidence.](../media/get-rdsremotedesktop.png)

Pour affecter un nom convivial à la ressource Bureau à distance, exécutez l’applet de commande PowerShell suivante :

```powershell
Set-RdsRemoteDesktop -TenantName <tenantname> -HostPoolName <hostpoolname> -AppGroupName <appgroupname> -FriendlyName <newfriendlyname>
```

> [!div class="mx-imgBorder"]
> ![Capture d’écran de l’applet de commande PowerShell Get-RDSRemoteApp avec le nom et le nouveau nom convivial en évidence.](../media/set-rdsremotedesktop.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez personnalisé le flux pour les utilisateurs, connectez-vous à un client Windows Virtual Desktop pour faire des tests. Pour cela, passez aux rubriques de procédures Se connecter à Windows Virtual Desktop :

 * [Se connecter à partir de Windows 10 ou Windows 7](connect-windows-7-10-2019.md)
 * [Se connecter à partir d’un navigateur web](connect-web-2019.md)
