---
title: Guide pratique pour activer la journalisation d’Azure Key Vault
description: Découvrez comment activer la journalisation d’Azure Key Vault, qui enregistre les informations dans un compte de stockage Azure que vous fournissez.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 10/01/2020
ms.author: mbaldwin
ms.openlocfilehash: 5e0007f3b0dad8a68e9d81cebbe9fe24b5a7db3c
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93285655"
---
# <a name="how-to-enable-key-vault-logging"></a>Guide pratique pour activer la journalisation de Key Vault

Une fois que vous avez créé un ou plusieurs coffres de clés, vous voulez probablement contrôler qui accède à ces derniers, par quel moyen et quand. Pour plus d’informations sur la fonctionnalité, consultez [Journalisation de Key Vault](logging.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, vous devez disposer des éléments suivants :

* Un coffre de clés existant que vous utilisez déjà.  
* Azure CLI ou Azure PowerShell.
* Espace de stockage suffisant sur Azure pour vos journaux d’activité de coffre de clés.

Si vous choisissez d’installer et d’utiliser l’interface CLI localement, vous aurez besoin de la version Azure CLI 2.0.4 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli). Pour vous connecter à Azure à l’aide de l’interface CLI, vous pouvez taper la commande suivante :

```azurecli-interactive
az login
```

Si vous choisissez d’installer et d’utiliser PowerShell localement, vous aurez besoin du module Azure PowerShell version 1.0.0 ou ultérieure. Tapez `$PSVersionTable.PSVersion` pour connaître la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

```powershell-interactive
Connect-AzAccount
```

## <a name="connect-to-your-key-vault-subscription"></a>Se connecter à l’abonnement Key Vault

La première étape de la configuration de la journalisation des clés consiste à se connecter à l’abonnement contenant le coffre de clés. Cette étape est particulièrement importante si plusieurs abonnements sont associés à votre compte.

Avec Azure CLI, vous pouvez afficher tous vos abonnements à l’aide de la commande [az account list](/cli/azure/account?view=azure-cli-latest#az_account_list), puis vous connecter à l’un d’eux à l’aide de la commande [az account set](/cli/azure/account?view=azure-cli-latest#az_account_set) :

```azurecli-interactive
az account list

az account set --subscription "<subscriptionID>"
```

Avec Azure PowerShell, vous pouvez lister vos abonnements à l’aide de l’applet de commande [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription?view=azps-4.7.0), puis vous connecter à l’un d’eux à l’aide de l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext?view=azps-4.7.0) : 

```powershell-interactive
Get-AzSubscription

Set-AzContext -SubscriptionId "<subscriptionID>"
```

## <a name="create-a-storage-account-for-your-logs"></a>Créer un compte de stockage pour vos journaux

Même si vous pouvez utiliser un compte de stockage existant pour vos journaux, nous allons en créer un dédié à vos journaux Key Vault. 

Pour simplifier davantage la gestion, nous allons utiliser le groupe de ressources qui contient le coffre de clés. Dans le [guide de démarrage rapide pour Azure CLI](quick-create-cli.md) et le [guide de démarrage rapide pour Azure PowerShell](quick-create-powershell.md), ce groupe de ressources est nommé **myResourceGroup** et la localisation est *eastus*. Remplacez ces valeurs par les vôtres, comme il convient. 

Nous devrons également fournir un nom de compte de stockage. Ce nom doit être unique et comprendre entre 3 et 24 caractères, uniquement des lettres en minuscules et des nombres.  Enfin, nous allons créer un compte de stockage de la référence SKU « Standard_LRS ».

Avec Azure CLI, utilisez la commande [az storage account create](/cli/azure/storage/account?view=azure-cli-latest#az_storage_account_create).

```azurecli-interactive
az storage account create --name "<your-unique-storage-account-name>" -g "myResourceGroup" --sku "Standard_LRS"
```

Avec Azure PowerShell, utilisez l’applet de commande [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount?view=azps-4.7.0). Vous devrez fournir la localisation qui correspond au groupe de ressources.

```powershell
 New-AzStorageAccount -ResourceGroupName myResourceGroup -Name "<your-unique-storage-account-name>" -Type "Standard_LRS" -Location "eastus"
```

Dans les deux cas, notez l’ID du compte de stockage. L’opération Azure CLI retourne l’ID dans la sortie. Pour obtenir l’ID avec Azure PowerShell, utilisez [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount?view=azps-4.7.0) et attribuez la sortie à une variable $sa. Vous pouvez alors voir le compte de stockage avec $sa.id. (La propriété « $sa.Context » sera également utilisée, plus loin dans cet article.)

```powershell-interactive
$sa = Get-AzStorageAccount -Name "<your-unique-storage-account-name>" -ResourceGroup "myResourceGroup"
$sa.id
```

L’ID du compte de stockage a le format « /subscriptions/<ID de votre abonnement>/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/<nom unique de votre compte de stockage> ».

> [!NOTE]
> Si vous décidez d’utiliser un compte de stockage existant, vous devez utiliser le même abonnement que pour votre coffre de clés ainsi que le modèle de déploiement Azure Resource Manager plutôt que le modèle de déploiement classique.

## <a name="obtain-your-key-vault-resource-id"></a>Obtenir l’ID de ressource du coffre de clés

Dans le [guide de démarrage rapide pour CLI](quick-create-cli.md) et le [guide de démarrage rapide pour PowerShell](quick-create-powershell.md), vous avez créé une clé avec un nom unique.  Réutilisez ce nom dans les étapes ci-dessous.  Si vous ne vous souvenez pas du nom de votre coffre de clés, vous pouvez utiliser la commande Azure CLI [az keyvault list](/cli/azure/keyvault?view=azure-cli-latest#az_keyvault_list) ou l’applet de commande Azure PowerShell [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault?view=azps-4.7.0) pour lister les coffres de clés.

Utilisez le nom de votre coffre de clés pour trouver son ID de ressource.  Avec Azure CLI, utilisez la commande [az keyvault show](/cli/azure/keyvault?view=azure-cli-latest#az_keyvault_show).

```azurecli-interactive
az keyvault show --name "<your-unique-keyvault-name>"
```

Avec Azure PowerShell, utilisez l’applet de commande [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault?view=azps-4.7.0).

```powershell-interactive
Get-AzKeyVault -VaultName "<your-unique-keyvault-name>"
```

L’ID de ressource de votre coffre de clés a le format « /subscriptions/<ID de votre abonnement>/resourceGroups/myResourceGroup/providers/Microsoft.KeyVault/vaults/<nom unique de votre coffre de clés> ». Notez-le pour l’étape suivante.

## <a name="enable-logging-using-azure-powershell"></a>Activer la journalisation à l’aide d’Azure PowerShell

Pour activer la journalisation pour Key Vault, nous allons utiliser la commande Azure CLI [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings?view=azure-cli-latest) ou l’applet de commande [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting?view=azps-4.7.0) ainsi que l’ID du compte de stockage et l’ID de ressource du coffre de clés.

```azurecli-interactive
az monitor diagnostic-settings create --storage-account "<storage-account-id>" --resource "<key-vault-resource-id>" --name "Key vault logs" --logs '[{"category": "AuditEvent","enabled": true}]' --metrics '[{"category": "AllMetrics","enabled": true}]'
```

Avec Azure PowerShell, nous allons utiliser l’applet de commande [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting?view=azps-4.7.0), avec l’indicateur **-Enabled** défini sur **$true** et la catégorie définie sur `AuditEvent` (unique catégorie pour la journalisation de Key Vault) :

```powershell-interactive
Set-AzDiagnosticSetting -ResourceId "<key-vault-resource-id>" -StorageAccountId $sa.id -Enabled $true -Category "AuditEvent"
```

Si vous le souhaitez, vous pouvez définir une stratégie de conservation pour vos journaux, afin que les anciens journaux soient automatiquement supprimés après un laps de temps spécifié. Par exemple, vous pouvez définir une stratégie de conservation qui supprime automatiquement les journaux datant de plus de 90 jours.

<!-- With the Azure CLI, use the [az monitor diagnostic-settings update](/cli/azure/monitor/diagnostic-settings?view=azure-cli-latest#az_monitor_diagnostic_settings_update) command. 

```azurecli-interactive
az monitor diagnostic-settings update 
```
-->

Avec Azure PowerShell, utilisez l’applet de commande [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting?view=azps-4.7.0). 

```powershell-interactive
Set-AzDiagnosticSetting "<key-vault-resource-id>" -StorageAccountId $sa.id -Enabled $true -Category AuditEvent -RetentionEnabled $true -RetentionInDays 90
```

Éléments journalisés :

* Toutes les requêtes d’API REST authentifiées, ce qui inclut les requêtes ayant échoué suite à des autorisations d’accès, des erreurs système ou des requêtes incorrectes.
* Les opérations sur le coffre de clés proprement dit, notamment la création, la suppression, la définition des stratégies d’accès au coffre de clés et la mise à jour des attributs de coffre de clés (par exemple, les balises).
* Les opérations sur les clés et secrets dans le coffre de clés, à savoir :
  * Création, modification ou suppression de ces clés ou secrets.
  * Signature, vérification, chiffrement, déchiffrement, inclusion dans un wrapper et retrait d’un wrapper de clés, obtention des secrets, et liste de clés et secrets (et leurs versions).
* les requêtes non authentifiées qui génèrent une réponse 401. Il s’agit notamment des requêtes dépourvues de jeton du porteur, dont le format est incorrect, qui ont expiré ou qui comportent un jeton non valide.  
* Les événements de notification Event Grid concernant l’expiration proche, l’expiration et la modification de la stratégie d’accès au coffre (l’événement de nouvelle version n’est pas consigné). Les événements sont consignés, quel que soit l’abonnement aux événements créé sur le coffre de clés. Pour plus d’informations, consultez [Schéma d’événement Event Grid pour Key Vault](../../event-grid/event-schema-key-vault.md)

## <a name="access-your-logs"></a>Accéder à vos journaux d’activité

Les journaux Key Vault sont stockés dans le conteneur « insights-logs-auditevent » du compte de stockage que vous avez fourni. Pour visualiser les journaux, vous devez télécharger des objets blob.

Tout d’abord, obtenez tous les objets blob du conteneur.  Avec Azure CLI, utilisez la commande [az storage blob list](/cli/azure/storage/blob?view=azure-cli-latest#az_storage_blob_list).

```azurecli-interactive
az storage blob list --account-name "<your-unique-storage-account-name>" --container-name "insights-logs-auditevent"
```

Avec Azure PowerShell, utilisez l’applet de commande [Get-AzStorageBlob](/powershell/module/az.storage/get-azstorageblob?view=azps-4.7.0) pour lister tous les objets blob de ce conteneur :

```powershell
Get-AzStorageBlob -Container $container -Context $sa.Context
```

Comme vous le verrez dans la sortie de la commande Azure CLI ou de l’applet de commande Azure PowerShell, le nom des objets blob a le format `resourceId=<ARM resource ID>/y=<year>/m=<month>/d=<day of month>/h=<hour>/m=<minute>/filename.json`. Les valeurs de date et d’heure utilisent UTC.

Étant donné que vous pouvez utiliser le même compte de stockage pour collecter les journaux de plusieurs ressources, l’ID de ressource complet dans le nom de l’objet blob est utile si vous voulez uniquement accéder aux objets blob dont vous avez besoin ou télécharger ces derniers. Mais avant cela, nous aborderons le téléchargement de tous les objets blob.

Avec Azure CLI, utilisez la commande [az storage blob download](/cli/azure/storage/blob?view=azure-cli-latest#az_storage_blob_download), en lui fournissant les noms des objets blob et le chemin du fichier dans lequel vous souhaitez enregistrer les résultats.

```azurecli-interactive
az storage blob download --container-name "insights-logs-auditevent" --file <path-to-file> --name "<blob-name>" --account-name "<your-unique-storage-account-name>"
```

Avec Azure PowerShell, utilisez l’applet de commande [Gt-AzStorageBlobs](/powershell/module/az.storage/get-azstorageblob?view=azps-4.7.0), pour obtenir la liste des objets blob, associée à l’applet de commande [Get-AzStorageBlobContent](/powershell/module/az.storage/get-azstorageblobcontent?view=azps-4.7.0), pour télécharger les journaux vers le chemin de votre choix.

```powershell-interactive
$blobs = Get-AzStorageBlob -Container $container -Context $sa.Context | Get-AzStorageBlobContent -Destination "<path-to-file>"
```

Quand vous exécutez cette seconde applet de commande dans PowerShell, le délimiteur **/** présent dans les noms d’objet blob crée une structure de dossiers complète sous le dossier de destination. Vous allez utiliser cette structure pour télécharger et stocker les objets blob sous forme de fichiers.

Pour télécharger les objets blob de façon sélective, utilisez des caractères génériques. Par exemple :

* Si vous disposez de plusieurs coffres de clés et souhaitez télécharger les journaux d’activité d’un seul d’entre eux nommé CONTOSOKEYVAULT3 :

  ```powershell
  Get-AzStorageBlob -Container $container -Context $sa.Context -Blob '*/VAULTS/CONTOSOKEYVAULT3
  ```

* Si vous disposez de plusieurs groupes de ressources et souhaitez télécharger les journaux d’activité d’un seul d’entre eux, utilisez `-Blob '*/RESOURCEGROUPS/<resource group name>/*'`:

  ```powershell
  Get-AzStorageBlob -Container $container -Context $sa.Context -Blob '*/RESOURCEGROUPS/CONTOSORESOURCEGROUP3/*'
  ```

* Si vous souhaitez télécharger tous les journaux du mois de janvier 2019, utilisez `-Blob '*/year=2019/m=01/*'` :

  ```powershell
  Get-AzStorageBlob -Container $container -Context $sa.Context -Blob '*/year=2016/m=01/*'
  ```

Vous êtes maintenant prêt à commencer les recherches dans le contenu des journaux d’activité. Mais avant de procéder à cette opération, vous devez connaître deux autres commandes :

Pour plus d’informations sur la lecture des journaux, consultez [Journalisation de Key Vault : Interpréter vos journaux Key Vault](logging.md#interpret-your-key-vault-logs).

## <a name="use-azure-monitor-logs"></a>Utiliser les journaux d’activité Azure Monitor

Vous pouvez utiliser la solution Key Vault dans les journaux Azure Monitor pour consulter les journaux `AuditEvent` de Key Vault. Dans les journaux Azure Monitor, vous utilisez des requêtes de journaux pour analyser les données et obtenir les informations dont vous avez besoin.

Pour plus d’informations, notamment sur la procédure de configuration correspondante, consultez [Azure Key Vault dans Azure Monitor](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des informations conceptuelles, notamment sur l’interprétation des journaux Key Vault, consultez [Journalisation de Key Vault](logging.md).
- Pour accéder à un tutoriel utilisant Azure Key Vault dans une application web .NET, consultez l’article [Utilisation d’Azure Key Vault à partir d’une application web](tutorial-net-create-vault-azure-web-app.md).
- Pour les références de programmation, consultez le [guide du développeur de coffre de clés Azure](developers-guide.md).