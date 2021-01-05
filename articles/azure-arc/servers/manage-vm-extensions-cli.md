---
title: Activer l’extension de machine virtuelle à l’aide d’Azure CLI
description: Cet article explique comment déployer des extensions de machine virtuelle sur des serveurs Azure Arc exécutés dans des environnements cloud hybrides avec Azure CLI.
ms.date: 11/20/2020
ms.topic: conceptual
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3fa8273b15518c182aefa038e67d85773d500b30
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94991449"
---
# <a name="enable-azure-vm-extensions-using-the-azure-cli"></a>Activer l’extension de machine virtuelle Azure à l’aide d’Azure CLI

Cet article explique comment déployer et désinstaller des extensions de machine virtuelle, prises en charge par les serveurs Azure Arc, sur une machine hybride Linux ou Windows via Azure CLI.

[!INCLUDE [Azure CLI Prepare your environment](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="install-the-azure-cli-extension"></a>Installer l’extension Azure CLI

Les commandes ConnectedMachine ne sont pas fournies avec Azure CLI. Avant d’utiliser Azure CLI pour gérer les extensions de machine virtuelle sur votre serveur hybride géré par des serveurs Arc, vous devez charger l’extension ConnectedMachine. Exécutez la commande suivante pour l’obtenir :

```azurecli
az extension add --name connectedmachine
```

## <a name="enable-extension"></a>Activer l’extension

Pour activer une extension de machine virtuelle sur votre serveur Arc, utilisez [az connectedmachine extension create](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_create) avec les paramètres `--machine-name`, `--extension-name`, `--location`, `--type`, `settings` et `--publisher`.

L’exemple suivant active l’extension de machine virtuelle Log Analytics sur un serveur Linux avec Arc :

```azurecli
az connectedmachine extension create --machine-name "myMachineName" --name "OmsAgentforLinux" --location "eastus" --type "CustomScriptExtension" --publisher "Microsoft.EnterpriseCloud.Monitoring" --settings "{\"workspaceId\":\"workspaceId"}" --protected-settings "{\workspaceKey\":"\workspaceKey"} --type-handler-version "1.10" --resource-group "myResourceGroup"
```

L’exemple suivant active l’extension de script personnalisé sur un serveur Arc :

```azurecli
az connectedmachine extension create --machine-name "myMachineName" --name "CustomScriptExtension" --location "eastus" --type "CustomScriptExtension" --publisher "Microsoft.Compute" --settings "{\"commandToExecute\":\"powershell.exe -c \\\"Get-Process | Where-Object { $_.CPU -gt 10000 }\\\"\"}" --type-handler-version "1.10" --resource-group "myResourceGroup"
```

L’exemple suivant active l’extension de machine virtuelle Key Vault (préversion) sur un serveur avec Arc :

```azurecli
az connectedmachine extension create --resource-group "resourceGroupName" --machine-name "myMachineName" --location "regionName" --publisher "Microsoft.Azure.KeyVault" --type "KeyVaultForLinux or KeyVaultForWindows" --name "KeyVaultForLinux or KeyVaultForWindows" --settings '{"secretsManagementSettings": { "pollingIntervalInS": "60", "observedCertificates": ["observedCert1"] }, "authenticationSettings": { "msiEndpoint": "http://localhost:40342/metadata/identity" }}'
```

## <a name="list-extensions-installed"></a>Lister les extensions installées

Pour obtenir une liste des extensions de machine virtuelle sur votre serveur Arc, utilisez [az connectedmachine extension list](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_list) avec les paramètres `--machine-name` et `--resource-group`.

Exemple :

```azurecli
az connectedmachine extension list --machine-name "myMachineName" --resource-group "myResourceGroup"
```

Par défaut, la sortie des commandes Azure CLI est au format JSON (JavaScript Object Notation). Pour remplacer la sortie par défaut pour une liste ou une table, par exemple, utilisez [az configure --output](/cli/azure/reference-index). Vous pouvez également ajouter `--output` à n’importe quelle commande pour modifier ponctuellement le format de sortie.

L’exemple suivant illustre la sortie JSON partielle obtenue avec la commande `az connectedmachine extension -list` :

```json
[
  {
    "autoUpgradingMinorVersion": "false",
    "forceUpdateTag": null,
    "id": "/subscriptions/subscriptionId/resourceGroups/resourceGroupName/providers/Microsoft.HybridCompute/machines/SVR01/extensions/DependencyAgentWindows",
    "location": "eastus",
    "name": "DependencyAgentWindows",
    "namePropertiesInstanceViewName": "DependencyAgentWindows",
```

## <a name="remove-an-installed-extension"></a>Supprimer une extension installée

Pour supprimer une extension de machine virtuelle installée sur votre serveur Arc, utilisez [az connectedmachine extension delete](/cli/azure/ext/connectedmachine/connectedmachine/extension#ext_connectedmachine_az_connectedmachine_extension_delete) avec les paramètres `--extension-name`, `--machine-name` et `--resource-group`.

Par exemple, pour supprimer l’extension de machine virtuelle Log Analytics pour Linux, exécutez la commande suivante :

```azurecli
az connectedmachine extension delete --machine-name "myMachineName" --name "OmsAgentforLinux" --resource-group "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

- Vous pouvez déployer, gérer et supprimer des extensions de machine virtuelle à l’aide d’[Azure PowerShell](manage-vm-extensions-powershell.md), du [portail Azure](manage-vm-extensions-portal.md) ou de [modèles Azure Resource Manager](manage-vm-extensions-template.md).

- Des informations de dépannage sont accessibles dans le [guide Résoudre les problèmes liés aux extensions de machine virtuelle](troubleshoot-vm-extensions.md).

- Pour plus d’informations sur les commandes, consultez l’article sur la [vue d’ensemble](/cli/azure/ext/connectedmachine/connectedmachine/extension) de l’extension de machine virtuelle Azure CLI.
