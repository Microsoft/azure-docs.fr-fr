---
title: Azure Cloud Shell pour les utilisateurs Windows | Microsoft Docs
description: Guide pour les utilisateurs qui ne sont pas familiers avec les systèmes Linux
services: azure
documentationcenter: ''
author: maertendMSFT
manager: hemantm
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 08/03/2018
ms.author: damaerte
ms.openlocfilehash: 109067899884c04cbd283b5eef1d7e0089556b07
ms.sourcegitcommit: ab94795f9b8443eef47abae5bc6848bb9d8d8d01
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/27/2020
ms.locfileid: "96301719"
---
# <a name="powershell-in-azure-cloud-shell-for-windows-users"></a>PowerShell dans Azure Cloud Shell pour les utilisateurs Windows

En mai 2018, des modifications ont été [annoncées](https://azure.microsoft.com/blog/pscloudshellrefresh/) pour PowerShell dans Azure Cloud Shell.
L’expérience PowerShell dans Azure Cloud Shell exécute désormais [PowerShell Core 6](https://github.com/powershell/powershell) dans un environnement Linux.
Ce changement peut entraîner une expérience PowerShell dans Cloud Shell différente de celle dans Windows.

## <a name="file-system-case-sensitivity"></a>Respect de la casse du système de fichiers

Le système de fichiers ne respecte pas la casse dans Windows, alors qu’il la respecte sur Linux.
`file.txt` et `FILE.txt` étaient précédemment considérés comme le même fichier, mais ils sont désormais traités comme des fichiers différents.
La casse appropriée doit être utilisée lors de l’utilisation de `tab-completing` dans le système de fichiers.
Les expériences spécifiques à PowerShell, comme les noms, paramètres et valeurs de l’applet de commande `tab-completing`, ne respectent pas la casse.

## <a name="windows-powershell-aliases-vs-linux-utilities"></a>Alias Windows PowerShell et utilitaires Linux

Certains alias PowerShell existants ont les mêmes noms que des commandes Linux intégrées, telles que `cat`, `ls`, `sort`, `sleep`, etc. Dans PowerShell Core 6, les alias qui entrent en conflit avec les commandes Linux intégrées ont été supprimés.
Voici les alias courants qui ont été supprimés, ainsi que les commandes équivalentes :  

|Alias supprimé   |Commande équivalente   |
|---|---|
|`cat`    | `Get-Content` |
|`curl`   | `Invoke-WebRequest` |
|`diff`   | `Compare-Object` |
|`ls`     | `dir` <br> `Get-ChildItem` |
|`mv`     | `Move-Item`   |
|`rm`     | `Remove-Item` |
|`sleep`  | `Start-Sleep` |
|`sort`   | `Sort-Object` |
|`wget`   | `Invoke-WebRequest` |

## <a name="persisting-home"></a>Persistance de $HOME

Au départ, les utilisateurs pouvaient uniquement conserver les scripts et autres fichiers dans leur lecteur Cloud.
À présent, le répertoire $HOME de l’utilisateur est également conservé entre les sessions.

## <a name="powershell-profile"></a>Profil PowerShell

Par défaut, aucun profil PowerShell d’utilisateur n’est créé.
Pour créer votre profil, créez un répertoire `PowerShell` sous `$HOME/.config`.

```azurepowershell-interactive
mkdir (Split-Path $profile.CurrentUserAllHosts)
```

Dans `$HOME/.config/PowerShell`, vous pouvez créer vos fichiers de profil : `profile.ps1` et/ou `Microsoft.PowerShell_profile.ps1`.

## <a name="whats-new-in-powershell-core-6"></a>Nouveautés dans PowerShell Core 6

Pour plus d’informations sur les nouveautés dans PowerShell Core 6, consultez les [documents sur PowerShell](/powershell/scripting/whats-new/what-s-new-in-powershell-70?view=powershell-7.1) et le billet de blog [Getting Started with PowerShell Core](https://blogs.msdn.microsoft.com/powershell/2017/06/09/getting-started-with-powershell-core-on-windows-mac-and-linux/).