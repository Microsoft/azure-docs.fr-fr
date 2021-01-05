---
title: Exécuter des scripts dans une machine virtuelle Azure Windows
description: Cette rubrique décrit comment exécuter des scripts dans une machine virtuelle Windows
services: automation
ms.service: virtual-machines
author: bobbytreed
ms.author: robreed
ms.date: 05/02/2018
ms.topic: how-to
manager: carmonm
ms.openlocfilehash: c2999d5bd6d780d5815f22d18e6e8cdd580987af
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87020166"
---
# <a name="run-scripts-in-your-windows-vm"></a>Exécuter des scripts dans votre machine virtuelle Windows

Pour automatiser les tâches ou résoudre les problèmes, vous devrez peut-être exécuter des commandes dans une machine virtuelle. L’article suivant donne une brève vue d’ensemble des fonctionnalités qui sont disponibles pour exécuter des scripts et des commandes sur vos machines virtuelles.

## <a name="custom-script-extension"></a>Extension de script personnalisé

[L’extension de script personnalisé](../extensions/custom-script-windows.md) est principalement utilisée pour l’installation de logiciels et la configuration après déploiement.

* Téléchargez et exécutez des scripts dans des machines virtuelles Azure.
* Il est possible d’utiliser les modèles Azure Resource Manager, la CLI Azure, l’API REST, PowerShell ou le portail Azure.
* Des fichiers de script peuvent être téléchargés à partir de Stockage Azure ou de GitHub, ou fournis à partir de votre ordinateur lors de l’exécution à partir du portail Azure.
* Exécutez le script PowerShell sur des ordinateurs Windows et le script Bash sur des ordinateurs Linux.
* Utile pour la configuration de post-déploiement, l’installation de logiciels ou toute autre tâche de configuration ou d’administration.

## <a name="run-command"></a>Commande d’exécution

La fonctionnalité [Commande d’exécution](run-command.md) permet l’administration de la machine virtuelle et de l’application, ainsi que le dépannage, à l’aide de scripts. Elle est disponible même lorsque la machine virtuelle n’est pas accessible, par exemple, si le pare-feu invité n’a pas le port RDP ou SSH ouvert.

* Exécutez des scripts dans des machines virtuelles Azure.
* L’exécution peut se faire à l’aide du [portail Azure](run-command.md), de [l’API REST](/rest/api/compute/virtual%20machines%20run%20commands/runcommand), de la [CLI Azure](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) ou de [PowerShell](/powershell/module/az.compute/invoke-azvmruncommand)
* Exécutez rapidement un script et affichez la sortie, puis répétez autant que nécessaire dans le portail Azure.
* Le script peut être tapé directement, ou vous pouvez exécuter l’un des scripts intégrés.
* Exécutez le script PowerShell sur des ordinateurs Windows et le script Bash sur des ordinateurs Linux.
* Utile pour l’administration de machine virtuelle et d’application, et pour l’exécution de scripts sur des machines virtuelles inaccessibles.

## <a name="hybrid-runbook-worker"></a>Runbook Worker hybride

Le [Runbook Worker hybride](../../automation/automation-hybrid-runbook-worker.md) fournit la gestion générale de la machine, de l’application et de l’environnement avec des scripts personnalisés de l’utilisateur stockés dans un compte Automation.

* Exécutez des scripts dans des machines Azure et non-Azure.
* L’exécution peut se faire à l’aide du portail Azure, de la CLI Azure, de l’API REST, de PowerShell ou du webhook.
* Scripts stockés et gérés dans un compte Automation.
* Exécuter des runbooks PowerShell, de flux de travail PowerShell, Python ou graphiques
* Aucune limite de temps sur la durée d’exécution du script.
* Plusieurs scripts peuvent s’exécuter simultanément.
* Sortie de script complète retourné et stockée.
* Historique des travaux disponible pendant 90 jours.
* Les scripts peuvent s’exécuter en tant que système local ou avec des informations d’identification fournies par l’utilisateur.
* [Installation manuelle](../../automation/automation-windows-hrw-install.md) requise

## <a name="serial-console"></a>Console série

La [Console série](../troubleshooting/serial-console-windows.md) offre un accès direct à une machine virtuelle, semblable au fait de disposer d’un clavier connecté à la machine virtuelle.

* Exécutez des commandes dans des machines Azure.
* L’exécution peut se faire à l’aide d’une console basée sur texte vers la machine virtuelle dans le portail Azure.
* Connectez-vous à la machine avec un compte d’utilisateur local.
* Utile lorsque l’accès à la machine virtuelle est nécessaire quel que soit l’état du réseau ou du système d’exploitation de la machine.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les différentes fonctionnalités qui sont disponibles pour exécuter des scripts et des commandes sur vos machines virtuelles.

* [Extension de script personnalisé](../extensions/custom-script-windows.md)
* [Commande d’exécution](run-command.md)
* [Runbook Worker hybride](../../automation/automation-hybrid-runbook-worker.md)
* [Console série](../troubleshooting/serial-console-windows.md)
