---
title: Fichier include
description: Fichier include
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: include
ms.date: 11/06/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 644669ea27938e385e11f3b1911a23ab30829a95
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96028275"
---
Cette fonctionnalité est en préversion. Pour l’utiliser, vous devez installer une extension ou un module en préversion.

### <a name="install-extension-for-azure-cli"></a>Installer l'extension pour l'interface de ligne de commande Azure

L'interface de ligne de commande Azure requiert l'[extension Event Grid](/cli/azure/azure-cli-extensions-list).

Dans [CloudShell](../articles/cloud-shell/quickstart.md) :

* Si vous avez installé l’extension précédemment, mettez-la à jour : `az extension update -n eventgrid`
* Si vous n’avez pas installé l’extension précédemment, faites-le : `az extension add -n eventgrid`

Pour une installation locale :

1. [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli). Vérifiez que la dernière version est installée en exécutant `az --version`.
1. Désinstallez les précédentes versions de l’extension `az extension remove -n eventgrid`
1. Installez l’extension `eventgrid` avec `az extension add -n eventgrid`

### <a name="install-module-for-powershell"></a>Installez le module pour PowerShell

PowerShell requiert le [module AzureRM.EventGrid](https://www.powershellgallery.com/packages/AzureRM.EventGrid/0.4.1-preview).

Dans [CloudShell](../articles/cloud-shell/quickstart-powershell.md) :

* Installez le module `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`

Pour une installation locale :

1. Ouvrez la console PowerShell en tant qu’administrateur
1. Installez le module `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`

Si le paramètre `-AllowPrerelease` n’est pas disponible, procédez comme suit :

1. Exécutez `Install-Module PowerShellGet -Force`
1. Exécutez `Update-Module PowerShellGet`
1. Fermez la console PowerShell
1. Redémarrez PowerShell en tant qu’administrateur
1. Installez le module `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`