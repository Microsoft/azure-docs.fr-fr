---
title: Limitations d’Azure Cloud Shell | Microsoft Docs
description: Vue d’ensemble des limites d’Azure Cloud Shell
services: azure
documentationcenter: ''
author: maertendMSFT
manager: timlt
tags: azure-resource-manager
ms.assetid: ''
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: na
ms.topic: article
ms.date: 02/15/2018
ms.author: damaerte
ms.openlocfilehash: 092dccab82326bb9983f11ff64fe50aee7b1084d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74951477"
---
# <a name="limitations-of-azure-cloud-shell"></a>Limitations d’Azure Cloud Shell

Azure Cloud Shell a les limitations connues suivantes :

## <a name="general-limitations"></a>Limitations générales

### <a name="system-state-and-persistence"></a>État du système et persistance

La machine qui fournit votre session Cloud Shell est temporaire. En effet, elle est recyclée lorsque votre session reste inactive pendant 20 minutes. Cloud Shell requiert qu’un partage de fichiers Azure soit monté. Par conséquent, votre abonnement doit être en mesure de configurer des ressources de stockage pour accéder à Cloud Shell. Autres éléments à prendre en compte :

* Avec le stockage monté, seules les modifications apportées à votre répertoire `$Home` sont conservées.
* Les partages de fichiers Azure peuvent être montés uniquement depuis votre [région affectée](persisting-shell-storage.md#mount-a-new-clouddrive).
  * Dans Bash, exécutez `env` pour rechercher votre région définie en tant que `ACC_LOCATION`.

### <a name="browser-support"></a>Prise en charge des navigateurs

Cloud Shell prend en charge les dernières versions de Microsoft Edge, Microsoft Internet Explorer, Google Chrome, Mozilla Firefox et Apple Safari. Safari en mode privé n’est pas pris en charge.

### <a name="copy-and-paste"></a>Copier et coller

[!INCLUDE [copy-paste](../../includes/cloud-shell-copy-paste.md)]

### <a name="for-a-given-user-only-one-shell-can-be-active"></a>Pour un utilisateur donné, un seul interpréteur de commandes peut être actif

Les utilisateurs peuvent lancer uniquement un seul type d’interpréteur de commandes à la fois, soit **Bash** soit **PowerShell**. Toutefois, plusieurs instances de Bash ou de PowerShell peuvent s’exécuter simultanément. Le fait de basculer entre Bash et PowerShell via le menu entraîne le redémarrage de Cloud Shell, ce qui met fin aux sessions existantes. Vous pouvez également exécuter Bash dans PowerShell en entrant `bash`, et exécuter PowerShell dans Bash en entrant `pwsh`.

### <a name="usage-limits"></a>Limites d’utilisation

Cloud Shell est destiné aux cas d’usage interactif. De fait, les sessions non interactives longues se terminent sans avertissement.

## <a name="bash-limitations"></a>Limites de Bash

### <a name="user-permissions"></a>Autorisations utilisateur

Les autorisations sont définies en tant qu’utilisateurs standards sans accès sudo. Les installations en dehors du répertoire `$Home` ne sont pas conservées.

### <a name="editing-bashrc-or-profile"></a>Modifier .bashrc ou $PROFILE

Soyez prudent lorsque vous modifiez le fichier $PROFILE de PowerShell ou .bashrc car cela peut entraîner des erreurs inattendues dans Cloud Shell.

## <a name="powershell-limitations"></a>Limites de PowerShell

### <a name="azuread-module-name"></a>Nom du module `AzureAD`

Le nom du module `AzureAD` est actuellement `AzureAD.Standard.Preview`, le module fournit les mêmes fonctionnalités.

### <a name="sqlserver-module-functionality"></a>Fonctionnalités du module `SqlServer`

Le module `SqlServer` inclus dans Cloud Shell n’offre qu’une prise en charge préliminaire pour PowerShell Core. En particulier, `Invoke-SqlCmd` n’est pas encore disponible.

### <a name="default-file-location-when-created-from-azure-drive"></a>Emplacement du fichier par défaut lors de sa création à partir du lecteur Azure :

Les utilisateurs ne peuvent pas créer de fichiers sous le lecteur Azure: à l'aide de cmdlets PowerShell. Si les utilisateurs créent des fichiers à l’aide d’autres outils, tels que vim ou nano, les fichiers sont enregistrés dans le dossier `$HOME` par défaut. 

### <a name="gui-applications-are-not-supported"></a>Les applications de l’interface graphique utilisateur ne sont pas prises en charge.

Si l'utilisateur exécute une commande susceptible de créer une boîte de dialogue Windows, un message d'erreur apparaît. Par exemple : `Unable to load DLL 'IEFRAME.dll': The specified module could not be found. (Exception from HRESULT: 0x8007007E)`.

### <a name="large-gap-after-displaying-progress-bar"></a>Écart important après avoir affiché la barre de progression

Si l’utilisateur exécute une action qui affiche une barre de progression, comme le renseignement d’un onglet sur le lecteur `Azure:`, il est possible que le curseur ne soit pas défini correctement et un écart s’affiche là où la barre de progression se situait précédemment.

## <a name="next-steps"></a>Étapes suivantes

[Dépannage de Cloud Shell](troubleshooting.md) <br>
[Démarrage rapide pour Bash](quickstart.md) <br>
[Démarrage rapide pour PowerShell](quickstart-powershell.md)
