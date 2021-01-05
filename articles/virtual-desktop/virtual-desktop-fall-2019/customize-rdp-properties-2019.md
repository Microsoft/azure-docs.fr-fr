---
title: Personnaliser les propriétés de RDP avec Windows Virtual Desktop (classique) PowerShell - Azure
description: Comment personnaliser les propriétés de RDP pour Windows Virtual Desktop (classique) avec des applets de commande PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 5110e97e52939ea2115bb839768cc7ab96802961
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020706"
---
# <a name="customize-remote-desktop-protocol-properties-for-a--windows-virtual-desktop-classic-host-pool"></a>Personnaliser les propriétés du protocole RDP pour un pool d’hôtes de Windows Virtual Desktop (classique)

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager. Si vous essayez de gérer des objets Windows Virtual Desktop Azure Resource Manager, consultez [cet article](../customize-rdp-properties.md).

La personnalisation des propriétés du protocole RDP (Remote Desktop Protocol) d’un pool d’hôtes, comme l’utilisation de plusieurs écrans et la redirection audio, vous permet d’offrir une expérience optimale pour vos utilisateurs en fonction de leurs besoins. Vous pouvez personnaliser les propriétés de RDP dans Windows Virtual Desktop avec le paramètre **-CustomRdpProperty** dans l’applet de commande **Set-RdsHostPool**.

Pour obtenir la liste complète des propriétés prises en charge et leur valeur par défaut, consultez [Paramètres de fichier RDP pris en charge](/windows-server/remote/remote-desktop-services/clients/rdp-files?context=%2fazure%2fvirtual-desktop%2fcontext%2fcontext).

Tout d’abord, si vous ne l’avez pas déjà fait, [téléchargez et importez le module PowerShell Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) à utiliser dans votre session PowerShell. Exécutez ensuite l’applet de commande suivante pour vous connecter à votre compte :

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
```

## <a name="default-rdp-properties"></a>Propriétés RDP par défaut

Par défaut, les fichiers RDP publiés contiennent les propriétés suivantes :

|Propriétés RDP | Bureaux | RemoteApps |
|---|---| --- |
| Mode à plusieurs écrans | activé | N/A |
| Redirections de lecteur activées | Lecteurs, presse-papiers, imprimantes, ports COM, périphériques USB et cartes à puce| Lecteurs, presse-papiers et imprimantes |
| Mode audio distant | Lire localement | Lire localement |

Toutes les propriétés personnalisées que vous définissez pour le pool hôte remplacent ces valeurs par défaut.

## <a name="add-or-edit-a-single-custom-rdp-property"></a>Ajouter ou modifier une seule propriété RDP personnalisée

Pour ajouter ou modifier une seule propriété RDP personnalisée, exécutez l’applet de commande PowerShell suivante :

```powershell
Set-RdsHostPool -TenantName <tenantname> -Name <hostpoolname> -CustomRdpProperty "<property>"
```

> [!div class="mx-imgBorder"]
> ![Capture d'écran de la cmdlet PowerShell Get-RDSRemoteApp, dans laquelle les noms Name et FriendlyName sont mis en surbrillance pour modifier une propriété RDP personnalisée.](../media/singlecustomrdpproperty.png)

## <a name="add-or-edit-multiple-custom-rdp-properties"></a>Ajouter ou modifier plusieurs propriétés RDP personnalisées

Pour ajouter ou modifier plusieurs propriétés RDP personnalisées, exécutez les applets de commande PowerShell suivantes en fournissant les propriétés RDP personnalisées sous forme de chaîne avec des points-virgules comme séparateur :

```powershell
$properties="<property1>;<property2>;<property3>"
Set-RdsHostPool -TenantName <tenantname> -Name <hostpoolname> -CustomRdpProperty $properties
```

> [!div class="mx-imgBorder"]
> ![Capture d'écran de la cmdlet PowerShell Set-RDSRemoteApp, dans laquelle les noms Name et FriendlyName sont mis en surbrillance pour modifier une propriété RDP personnalisée.](../media/multiplecustomrdpproperty.png)

## <a name="reset-all-custom-rdp-properties"></a>Réinitialiser toutes les propriétés RDP personnalisées

Vous pouvez réinitialiser des propriétés RDP personnalisées à leurs valeurs par défaut en suivant les instructions de [Ajouter ou modifier une seule propriété RDP personnalisée](#add-or-edit-a-single-custom-rdp-property), ou vous pouvez réinitialiser toutes les propriétés RDP personnalisées pour un pool d’hôtes en exécutant l’applet de commande PowerShell suivante :

```powershell
Set-RdsHostPool -TenantName <tenantname> -Name <hostpoolname> -CustomRdpProperty ""
```

> [!div class="mx-imgBorder"]
> ![Capture d’écran de l’applet de commande PowerShell Get-RDSRemoteApp avec le nom et le nom convivial en évidence.](../media/resetcustomrdpproperty.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez personnalisé les propriétés RDP pour un pool d’hôtes donné, connectez-vous à un client Windows Virtual Desktop pour les tester dans le cadre d’une session utilisateur. Ces deux procédures suivantes vous indiquent comment vous connecter à une session à l’aide du client de votre choix :

- [Se connecter avec le client Windows Desktop](connect-windows-7-10-2019.md)
- [Se connecter avec le client web](connect-web-2019.md)