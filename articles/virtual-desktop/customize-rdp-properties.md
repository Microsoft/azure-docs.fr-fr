---
title: Personnaliser les propriétés de RDP avec PowerShell – Azure
description: Comment personnaliser les propriétés de RDP pour Windows Virtual Desktop avec des applets de commande PowerShell.
author: Heidilohr
ms.topic: how-to
ms.date: 10/09/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: cc3a08f383368b189e41bebd204707f2483e77c0
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95018305"
---
# <a name="customize-remote-desktop-protocol-rdp-properties-for-a-host-pool"></a>Personnaliser les propriétés de Remote Desktop Protocol (RDP) pour un pool d’hôtes

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop avec des objets Windows Virtual Desktop Azure Resource Manager. Si vous utilisez la version Windows Virtual Desktop (classique) sans objets Azure Resource Manager, consultez [cet article](./virtual-desktop-fall-2019/customize-rdp-properties-2019.md).

La personnalisation des propriétés du protocole RDP (Remote Desktop Protocol) d’un pool d’hôtes, comme l’utilisation de plusieurs écrans et la redirection audio, vous permet d’offrir une expérience optimale pour vos utilisateurs en fonction de leurs besoins. Vous pouvez personnaliser les propriétés RDP dans Windows Virtual Desktop en utilisant le portail Azure ou le paramètre *-CustomRdpProperty* dans la cmdlet **Update-AzWvdHostPool**.

Pour obtenir la liste complète des propriétés prises en charge et leur valeur par défaut, consultez [Paramètres de fichier RDP pris en charge](/windows-server/remote/remote-desktop-services/clients/rdp-files?context=%2fazure%2fvirtual-desktop%2fcontext%2fcontext).

## <a name="default-rdp-file-properties"></a>Propriétés de fichier RDP par défaut

Les fichiers RDP ont les propriétés suivantes par défaut :

|Propriété RDP|Sur l’ordinateur de bureau|En tant qu’application distante|
|---|---|---|
|Mode à plusieurs écrans|Désactivé|activé|
|Redirections de lecteur activées|Lecteurs, presse-papiers, imprimantes, ports COM et cartes à puce|Lecteurs, presse-papiers et imprimantes|
|Mode audio distant|Lire localement|Lire localement|

## <a name="prerequisites"></a>Prérequis

Avant de commencer, suivez les instructions fournies dans [Configurer le module PowerShell Windows Virtual Desktop](powershell-module.md) pour configurer votre module PowerShell et vous connecter à Azure.

## <a name="configure-rdp-properties-in-the-azure-portal"></a>Configurer les propriétés RDP dans le portail Azure

Pour configurer les propriétés RDP dans le portail Azure :

1. Connectez-vous à Azure sur <https://portal.azure.com>.
2. Entrez **Windows Virtual Desktop** dans la barre de recherche.
3. Sous Services, sélectionnez **Windows Virtual Desktop**.
4. Dans la page Windows Virtual Desktop, dans le menu sur le côté gauche de l’écran, sélectionnez **pools d’hôtes**.
5. Sélectionnez le **nom du pool d’hôtes** que vous souhaitez mettre à jour.
6. Dans le menu situé sur le côté gauche de l’écran, sélectionnez **Propriétés RDP**.
7. Définissez la propriété de votre choix.
   - Vous pouvez également ouvrir l’onglet **Avancé** et ajouter vos propriétés RDP dans un format séparé par des points-virgules, comme les exemples PowerShell des sections suivantes.
8. Lorsque vous avez terminé, sélectionnez **Enregistrer** pour enregistrer vos paramètres.

Les sections suivantes expliquent comment modifier manuellement des propriétés RDP personnalisées dans PowerShell.

## <a name="add-or-edit-a-single-custom-rdp-property"></a>Ajouter ou modifier une seule propriété RDP personnalisée

Pour ajouter ou modifier une seule propriété RDP personnalisée, exécutez l’applet de commande PowerShell suivante :

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty <property>
```

Pour vérifier si la cmdlet que vous venez d’exécuter a mis à jour la propriété, exécutez la cmdlet suivante :

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <customRDPpropertystring>
```

Par exemple, pour rechercher la propriété « audiocapturemode » sur un pool d’hôtes nommé 0301HP, vous devez entrer la cmdlet suivante :

```powershell
Get-AzWvdHostPool -ResourceGroupName 0301rg -Name 0301hp | format-list Name, CustomRdpProperty

Name              : 0301HP
CustomRdpProperty : audiocapturemode:i:1;
```

## <a name="add-or-edit-multiple-custom-rdp-properties"></a>Ajouter ou modifier plusieurs propriétés RDP personnalisées

Pour ajouter ou modifier plusieurs propriétés RDP personnalisées, exécutez les applets de commande PowerShell suivantes en fournissant les propriétés RDP personnalisées sous forme de chaîne avec des points-virgules comme séparateur :

```powershell
$properties="<property1>;<property2>;<property3>"
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty $properties
```

Vous pouvez vérifier que la propriété RDP a été ajoutée en exécutant la cmdlet suivante :

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <customRDPpropertystring>
```

Sur la base de notre exemple de cmdlet précédent, si vous configurez plusieurs propriétés RDP sur le pool d’hôtes 0301HP, votre cmdlet doit ressembler à ceci :

```powershell
Get-AzWvdHostPool -ResourceGroupName 0301rg -Name 0301hp | format-list Name, CustomRdpProperty

Name              : 0301HP
CustomRdpProperty : audiocapturemode:i:1;audiomode:i:0;
```

## <a name="reset-all-custom-rdp-properties"></a>Réinitialiser toutes les propriétés RDP personnalisées

Vous pouvez réinitialiser des propriétés RDP personnalisées à leurs valeurs par défaut en suivant les instructions de [Ajouter ou modifier une seule propriété RDP personnalisée](#add-or-edit-a-single-custom-rdp-property), ou vous pouvez réinitialiser toutes les propriétés RDP personnalisées pour un pool d’hôtes en exécutant l’applet de commande PowerShell suivante :

```powershell
Update-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> -CustomRdpProperty ""
```

Pour vous assurer que vous avez correctement supprimé le paramètre, entrez la cmdlet suivante :

```powershell
Get-AzWvdHostPool -ResourceGroupName <resourcegroupname> -Name <hostpoolname> | format-list Name, CustomRdpProperty

Name              : <hostpoolname>
CustomRdpProperty : <CustomRDPpropertystring>
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez personnalisé les propriétés RDP pour un pool d’hôtes donné, connectez-vous à un client Windows Virtual Desktop pour les tester dans le cadre d’une session utilisateur. Les guides pratiques suivants indiquent comment se connecter à une session à l’aide du client choisi :

- [Se connecter avec le client Windows Desktop](connect-windows-7-10.md)
- [Se connecter avec le client web](connect-web.md)
- [Se connecter avec le client Android](connect-android.md)
- [Se connecter avec le client macOS](connect-macos.md)
- [Se connecter avec le client iOS](connect-ios.md)