---
title: Exécuter des scripts shell dans une machine virtuelle Linux sur Azure
description: Cette rubrique explique comment exécuter des scripts au sein d’une machine virtuelle Linux Azure à l’aide de la fonctionnalité Run Command
services: automation
ms.service: virtual-machines
author: bobbytreed
ms.author: robreed
ms.date: 04/26/2019
ms.topic: how-to
manager: carmonm
ms.openlocfilehash: 5baa6d57bd3895640f1654cf7a5ebca52f101cbe
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91970569"
---
# <a name="run-shell-scripts-in-your-linux-vm-by-using-run-command"></a>Exécuter des scripts shell dans votre machine virtuelle Linux avec Run Command

La fonctionnalité Run Command utilise l’agent de machine virtuelle (VM) pour exécuter des scripts shell au sein d’une machine virtuelle Linux Azure. Vous pouvez utiliser ces scripts pour la gestion générale des ordinateurs ou des applications. Ils peuvent vous aider à diagnostiquer et corriger rapidement les problèmes de réseau et d’accès aux machines virtuelles et à rétablir l’état de la machine virtuelle.

## <a name="benefits"></a>Avantages

Vous pouvez accéder à vos machines virtuelles de plusieurs façons. Run Command peut exécuter à distance des scripts sur vos machines virtuelles à l’aide de l’agent de machine virtuelle. Vous utilisez Run Command via le portail Azure, une [API REST](/rest/api/compute/virtual%20machines%20run%20commands/runcommand) ou [Azure CLI](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) pour des machines virtuelles Linux.

Cette fonctionnalité est utile dans tous les scénarios où vous souhaitez exécuter un script sur une machine virtuelle. C’est l’une des seules manières de détecter et de corriger un problème sur une machine virtuelle qui n’a pas de port RDP ou SSH ouvert en raison d’une configuration incorrecte du réseau ou de l’utilisateur administratif.

## <a name="restrictions"></a>Restrictions

Les restrictions suivantes s’appliquent lorsque vous utilisez Run Command :

* La sortie est limitée aux derniers 4 096 octets.
* La durée minimale d’exécution d’un script est d’environ 20 secondes.
* Les scripts s’exécutent par défaut en tant qu’utilisateur avec élévation de privilèges sur Linux.
* Vous pouvez exécuter un script à la fois.
* Les scripts qui demandent des informations (mode interactif) ne sont pas pris en charge.
* Vous ne pouvez pas annuler un script en cours d’exécution.
* La durée maximale d’exécution d’un script est de 90 minutes. Après cela, le script expire.
* Une connectivité sortante à partir de la machine virtuelle est nécessaire pour retourner les résultats du script.

> [!NOTE]
> Pour fonctionner correctement, Run Command a besoin d’une connectivité (port 443) aux IP publiques Azure. Si l’extension n’a pas accès à ces points de terminaison, il se peut que les scripts s’exécutent correctement, mais qu’ils ne retournent pas les résultats. Si vous bloquez le trafic sur la machine virtuelle, vous pouvez utiliser des [balises de service](../../virtual-network/network-security-groups-overview.md#service-tags) pour autoriser le trafic à destination des IP publiques Azure en utilisant la balise `AzureCloud`.

## <a name="available-commands"></a>Commandes disponibles

Ce tableau affiche la liste des commandes disponibles pour les machines virtuelles Linux. Vous pouvez utiliser la commande **RunShellScript** pour exécuter n’importe quel script personnalisé de votre choix. Quand vous utilisez Azure CLI ou PowerShell pour exécuter une commande, la valeur que vous fournissez pour le paramètre `--command-id` ou `-CommandId` doit être l’une des valeurs listées ci-dessous. Quand vous spécifiez une valeur qui n’est pas une commande disponible, vous recevez cette erreur :

```error
The entity was not found in this Azure location
```

|**Nom**|**Description**|
|---|---|
|**RunShellScript**|Exécute un script shell Linux.|
|**ifconfig**| Obtient la configuration de toutes les interfaces réseau.|

## <a name="azure-cli"></a>Azure CLI

L’exemple suivant utilise la commande [az vm run-command](/cli/azure/vm/run-command?view=azure-cli-latest#az-vm-run-command-invoke) pour exécuter un script shell sur une machine virtuelle Linux Azure.

```azurecli-interactive
az vm run-command invoke -g myResourceGroup -n myVm --command-id RunShellScript --scripts "apt-get update && apt-get install -y nginx"
```

> [!NOTE]
> Si vous souhaitez exécuter des commandes en tant qu’autre utilisateur, entrez `sudo -u` pour spécifier un compte d’utilisateur.

## <a name="azure-portal"></a>Portail Azure

Accédez à une machine virtuelle dans le [Portail Azure](https://portal.azure.com) et sélectionnez **Run Command** sous **OPÉRATIONS**. Une liste des commandes pouvant s’exécuter sur la machine virtuelle s’affiche.

![Liste des commandes](./media/run-command/run-command-list.png)

Choisissez une commande à exécuter. Certaines des commandes peuvent avoir des paramètres d’entrée facultatifs ou obligatoires. Pour ces commandes, les paramètres sont présentés en tant que champs de texte vous permettant de fournir les valeurs d’entrée. Pour chaque commande, vous pouvez voir le script en cours d’exécution en développant **Afficher le script**. La commande **RunShellScript** est différente des autres commandes, car elle vous permet de fournir votre propre script personnalisé.

> [!NOTE]
> Les commandes intégrées ne sont pas modifiables.

Une fois que vous avez choisi la commande, sélectionnez **Exécuter** pour exécuter le script. Une fois le script terminé, il renvoie la sortie et toutes les erreurs dans la fenêtre de sortie. La capture d’écran suivante montre un exemple de sortie de l’exécution de la commande **ifconfig**.

![Sortie du script d’une commande Run](./media/run-command/run-command-script-output.png)

### <a name="powershell"></a>PowerShell

L’exemple suivant utilise l’applet de commande [Invoke-AzVMRunCommand](/powershell/module/az.compute/invoke-azvmruncommand) pour exécuter un script PowerShell sur une machine virtuelle Azure. Pour la cmdlet, le script référencé dans le paramètre `-ScriptPath` doit se situer au même emplacement qu'elle.

```powershell-interactive
Invoke-AzVMRunCommand -ResourceGroupName '<myResourceGroup>' -Name '<myVMName>' -CommandId 'RunShellScript' -ScriptPath '<pathToScript>' -Parameter @{"arg1" = "var1";"arg2" = "var2"}
```

## <a name="limiting-access-to-run-command"></a>Limitation de l’accès à la commande Run

La liste des commandes d’exécution ou l’affichage des détails d’une commande nécessite l’autorisation `Microsoft.Compute/locations/runCommands/read`. Le rôle intégré [Lecteur](../../role-based-access-control/built-in-roles.md#reader) et les niveaux supérieurs disposent de cette autorisation.

L’exécution d’une commande nécessite l’autorisation `Microsoft.Compute/virtualMachines/runCommand/action`. Le rôle [Contributeur de machines virtuelles](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) et les niveaux supérieurs disposent de cette autorisation.

Vous pouvez utiliser un des [rôles intégrés](../../role-based-access-control/built-in-roles.md) ou créer un [rôle personnalisé](../../role-based-access-control/custom-roles.md) afin d’exécuter Run Command.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les autres méthodes d’exécution à distance des commandes et des scripts dans votre machine virtuelle, consultez [Exécuter des scripts dans votre machine virtuelle Linux](run-scripts-in-vm.md).