---
title: Créer un cache Azure HPC Cache
description: Comment créer une instance de cache Azure HPC Cache
author: ekpgh
ms.service: hpc-cache
ms.topic: how-to
ms.date: 09/30/2020
ms.author: v-erkel
ms.openlocfilehash: 77bd5e3b7a258ef83e5de4ec645ea70578fb9dfb
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657089"
---
# <a name="create-an-azure-hpc-cache"></a>Créer un cache Azure HPC Cache

Utilisez le portail Azure ou Azure CLI pour créer votre cache.

![capture d’écran de la vue d’ensemble du cache dans le portail Azure, avec le bouton Créer en bas](media/hpc-cache-home-page.png)

Cliquez sur l’image ci-dessous pour regarder une [vidéo de démonstration](https://azure.microsoft.com/resources/videos/set-up-hpc-cache/) montrant comment créer un cache et ajouter une cible de stockage.

[![Miniature de vidéo : Azure HPC Cache : Configuration (cliquez pour accéder à la page vidéo)](media/video-4-setup.png)](https://azure.microsoft.com/resources/videos/set-up-hpc-cache/)

## <a name="portal"></a>[Portail](#tab/azure-portal)

## <a name="define-basic-details"></a>Définir les détails de base

![capture d’écran de la page Détails du projet dans le portail Azure](media/hpc-cache-create-basics.png)

Dans **Détails du projet**, sélectionnez l’abonnement et le groupe de ressources qui doit héberger le cache.

Dans **Détails sur le service**, définissez le nom du cache et les autres attributs suivants :

* Emplacement : sélectionnez l’une des [régions prises en charge](hpc-cache-overview.md#region-availability).
* Réseau virtuel : vous pouvez sélectionner un réseau virtuel existant ou en créer un.
* Sous-réseau : choisissez ou créez un sous-réseau avec au moins 64 adresses IP (/24). Ce sous-réseau doit être utilisé uniquement pour cette instance Azure HPC Cache.

## <a name="set-cache-capacity"></a>Définir la capacité du cache
<!-- referenced from GUI - update aka.ms link if you change this header text -->

Dans la page **Cache**, vous devez définir la capacité de votre cache. Les valeurs définies ici déterminent la quantité de données que votre cache peut contenir et la rapidité avec laquelle il peut traiter les requêtes des clients.

La capacité affecte également le coût du cache.

Choisissez la capacité en définissant les deux valeurs suivantes :

* Taux de transfert de données maximal pour le cache (débit), en Go/seconde
* Quantité de stockage allouée aux données mises en cache, en To

Choisissez l’une des valeurs de débit et des tailles de stockage du cache disponibles.

Gardez à l’esprit que le taux de transfert de données réel dépend de la charge de travail, des vitesses réseau et du type de cible de stockage. Les valeurs que vous choisissez définissent le débit maximal pour l’ensemble du système de mise en cache. Toutefois, certaines d’entre elles sont utilisées pour les tâches de surcharge. Par exemple, si un client demande un fichier qui n’est pas déjà stocké dans le cache, ou si le fichier est marqué comme étant obsolète, votre cache utilise une partie de son débit pour l’extraire du stockage back-end.

Azure HPC Cache gère les fichiers mis en cache et préchargés pour maximiser les taux de réussite du cache. Le contenu du cache est évalué en permanence, et les fichiers sont déplacés vers un stockage à long terme quand ils sont moins fréquemment sollicités. Choisissez une taille de stockage du cache qui peut contenir facilement l’ensemble actif de fichiers de travail ainsi que de l’espace supplémentaire pour les métadonnées et d’autres surcharges.

![capture d’écran de la page de dimensionnement du cache](media/hpc-cache-create-capacity.png)

## <a name="enable-azure-key-vault-encryption-optional"></a>Activer le chiffrement Azure Key Vault (facultatif)

Si votre cache se trouve dans une région qui prend en charge les clés de chiffrement gérées par le client, la page **Clés de chiffrement de disque** s’affiche entre les onglets **Cache** et **Balises**. Consultez [Disponibilité régionale](hpc-cache-overview.md#region-availability) pour en savoir plus sur la prise en charge des régions.

Si vous souhaitez gérer les clés de chiffrement utilisées pour votre stockage de cache, fournissez vos informations Azure Key Vault sur la page **Clés de chiffrement de disque**. Le coffre de clés doit se trouver dans la même région et dans le même abonnement que le cache.

Vous pouvez ignorer cette section si vous n’avez pas besoin de clés gérées par le client. Par défaut, Azure chiffre les données avec des clés gérées par Microsoft. En savoir plus sur le [chiffrement de stockage Azure](../storage/common/storage-service-encryption.md).

> [!NOTE]
>
> * Après avoir créé le cache, vous ne pouvez plus passer d’une clé gérée par Microsoft à une clé gérée par le client.
> * Une fois le cache créé, vous devez l’autoriser à accéder au coffre de clés. Cliquez sur le bouton **Activer le chiffrement** dans la page **Vue d’ensemble** du cache pour activer le chiffrement. Effectuez cette étape dans les 90 minutes qui suivent la création du cache.
> * Les caches de disque sont créés après cette autorisation. Cela signifie que le temps de création initial du cache est court, mais que le cache ne sera pas prêt à être utilisé pendant dix minutes ou plus après que vous en aurez autorisé l’accès.

Pour obtenir une explication complète du processus de chiffrement avec clé gérée par le client, lisez [Utiliser des clés de chiffrement gérées par le client pour Azure HPC Cache](customer-keys.md).

![capture d’écran de la page des clés de chiffrement avec l’option « gérée par le client » sélectionnée et les champs du coffre de clés affichés](media/create-encryption.png)

Sélectionnez **Gérée par le client** pour choisir le chiffrement avec clé gérée par le client. Les champs de spécification du coffre de clés s’affichent. Sélectionnez le coffre Azure Key Vault à utiliser, puis sélectionnez la clé et la version à utiliser pour ce cache. La clé doit être une clé RSA 2048 bits. Vous pouvez créer un coffre de clés, une clé ou une version de clé à partir de cette page.

Après avoir créé le cache, vous devez l’autoriser à utiliser le service de coffre de clés. Pour plus d’informations, consultez [Autoriser le chiffrement Azure Key Vault à partir du cache](customer-keys.md#3-authorize-azure-key-vault-encryption-from-the-cache).

## <a name="add-resource-tags-optional"></a>Ajouter des balises de ressources (facultatif)

La page **Balises** vous permet d’ajouter des [balises de ressources](../azure-resource-manager/management/tag-resources.md) à votre instance Azure HPC Cache.

## <a name="finish-creating-the-cache"></a>Terminer la création du cache

Après avoir configuré le nouveau cache, cliquez sur l’onglet **Vérifier + créer**. Le portail valide vos choix et vous permet de les passer en revue. Si tout est correct, cliquez sur **Créer**.

La création du cache prend environ 10 minutes. Vous pouvez suivre la progression dans le panneau de notifications du portail Azure.

![capture d’écran des pages de création de cache « déploiement en cours » et « notifications » sur le portail](media/hpc-cache-deploy-status.png)

À l’issue de l’étape de création, une notification s’affiche avec un lien vers la nouvelle instance Azure HPC Cache, et le cache apparaît dans la liste **Ressources** de votre abonnement.

![capture d’écran de l’instance de cache Azure HPC Cache dans le portail Azure](media/hpc-cache-new-overview.png)

> [!NOTE]
> Si votre cache utilise des clés de chiffrement gérées par le client, le cache peut apparaître dans la liste des ressources avant que l’état du déploiement ne devienne Terminé. Dès que l’état du cache est **En attente de la clé**, vous pouvez [l’autoriser](customer-keys.md#3-authorize-azure-key-vault-encryption-from-the-cache) à utiliser le coffre de clés.

## <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

## <a name="create-the-cache-with-azure-cli"></a>Créer le cache avec Azure CLI

[Configurez Azure CLI pour Azure HPC Cache](./az-cli-prerequisites.md).

> [!NOTE]
> Azure CLI ne prend actuellement pas en charge la création d’un cache avec des clés de chiffrement gérées par le client. Utilisez le portail Azure.

Utilisez la commande [az hpc-cache create](/cli/azure/ext/hpc-cache/hpc-cache#ext-hpc-cache-az-hpc-cache-create) pour créer une nouvelle instance Azure HPC Cache.

Fournissez ces valeurs :

* Nom du groupe de ressources en cache
* Nom du cache
* Région Azure
* Sous-réseau de cache, au format suivant :

  ``--subnet "/subscriptions/<subscription_id>/resourceGroups/<cache_resource_group>/providers/Microsoft.Network/virtualNetworks/<virtual_network_name>/sub
nets/<cache_subnet_name>"``

  Le sous-réseau de cache a besoin d’au moins 64 adresses IP (/24), et il ne peut pas héberger d’autres ressources.

* La capacité du cache. Deux valeurs définissent le débit maximal de votre instance Azure HPC Cache :

  * Taille du cache (en Go)
  * La référence des machines virtuelles utilisées dans l’infrastructure du cache

  [az hpc-cache skus list](/cli/azure/ext/hpc-cache/hpc-cache/skus) affiche les références SKU disponibles et les options de taille de cache valides pour chacune d’elles. Les options de taille de cache vont de 3 à 48 To, mais seules certaines valeurs sont prises en charge.

  Ce graphique montre les combinaisons de taille de cache et de référence SKU valides au moment de la préparation de ce document (juillet 2020).

  | Taille du cache | Standard_2G | Standard_4G | Standard_8G |
  |------------|-------------|-------------|-------------|
  | 3072 Go    | oui         | Non          | Non          |
  | 6144 Go    | oui         | Oui         | Non          |
  | 12288 Go   | oui         | oui         | oui         |
  | 24576 Go   | Non          | oui         | oui         |
  | <49152 Go   | Non          | Non          | oui         |

  Pour obtenir des informations importantes sur la tarification, le débit et la taille de votre cache en fonction de votre flux de travail, consultez la section **Définir la capacité du cache** sous l’onglet Instructions du portail.

Exemple de création de cache :

```azurecli
az hpc-cache create --resource-group doc-demo-rg --name my-cache-0619 \
    --location "eastus" --cache-size-gb "3072" \
    --subnet "/subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.Network/virtualNetworks/vnet-doc0619/subnets/default" \
    --sku-name "Standard_2G"
```

La création du cache prend plusieurs minutes. En cas de réussite, la commande create retourne une sortie semblable à celle-ci :

```azurecli
{
  "cacheSizeGb": 3072,
  "health": {
    "state": "Healthy",
    "statusDescription": "The cache is in Running state"
  },
  "id": "/subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.StorageCache/caches/my-cache-0619",
  "location": "eastus",
  "mountAddresses": [
    "10.3.0.17",
    "10.3.0.18",
    "10.3.0.19"
  ],
  "name": "my-cache-0619",
  "provisioningState": "Succeeded",
  "resourceGroup": "doc-demo-rg",
  "sku": {
    "name": "Standard_2G"
  },
  "subnet": "/subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.Network/virtualNetworks/vnet-doc0619/subnets/default",
  "tags": null,
  "type": "Microsoft.StorageCache/caches",
  "upgradeStatus": {
    "currentFirmwareVersion": "5.3.42",
    "firmwareUpdateDeadline": "0001-01-01T00:00:00+00:00",
    "firmwareUpdateStatus": "unavailable",
    "lastFirmwareUpdate": "2020-04-01T15:19:54.068299+00:00",
    "pendingFirmwareVersion": null
  }
}
```

Le message inclut des informations utiles, notamment les éléments suivants :

* Adresses de montage client : utilisez ces adresses IP lorsque vous êtes prêt à connecter les clients au cache. Pour en savoir plus, consultez [Monter le cache Azure HPC Cache](hpc-cache-mount.md).
* État de la mise à niveau : lorsqu’une mise à jour logicielle est publiée, ce message est modifié. Vous pouvez [mettre à niveau du logiciel de cache](hpc-cache-manage.md#upgrade-cache-software) manuellement à un moment approprié, ou la mise à niveau sera appliquée automatiquement après plusieurs jours.

## <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

> [!CAUTION]
> Le module PowerShell Az.HPCCache est en préversion publique. Cette préversion est fournie sans contrat de niveau de service. Il n’est pas recommandé pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="requirements"></a>Spécifications

Si vous choisissez d’utiliser PowerShell localement, cet article vous demande d’installer le module Az PowerShell et de vous connecter à votre compte Azure avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Pour en savoir plus sur l’installation du module Az PowerShell, consultez [Installer Azure PowerShell](/powershell/azure/install-az-ps). Si vous choisissez d’utiliser Cloud Shell, consultez [Vue d’ensemble d’Azure Cloud Shell](../cloud-shell/overview.md) pour plus d’informations.

> [!IMPORTANT]
> Tant que le module PowerShell **Az.HPCCache** est en préversion, vous devez l’installer séparément à l’aide de l’applet de commande `Install-Module`. Une fois que ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module PowerShell Az et sera disponible en mode natif dans Azure Cloud Shell.

```azurepowershell-interactive
Install-Module -Name Az.HPCCache
```

## <a name="create-the-cache-with-azure-powershell"></a>Créer le cache avec Azure PowerShell

> [!NOTE]
> Azure PowerShell ne prend actuellement pas en charge la création d’un cache avec des clés de chiffrement gérées par le client. Utilisez le portail Azure.

Utilisez l’applet de commande [New-AzHpcCache](/powershell/module/az.hpccache/new-azhpccache) pour créer un cache HPC Azure Cache.

Remplacez les valeurs suivantes :

* Nom du groupe de ressources en cache
* Nom du cache
* Région Azure
* Sous-réseau de cache, au format suivant :

  `-SubnetUri "/subscriptions/<subscription_id>/resourceGroups/<cache_resource_group>/providers/Microsoft.Network/virtualNetworks/<virtual_network_name>/sub
nets/<cache_subnet_name>"`

  Le sous-réseau de cache a besoin d’au moins 64 adresses IP (/24), et il ne peut pas héberger d’autres ressources.

* La capacité du cache. Deux valeurs définissent le débit maximal de votre instance Azure HPC Cache :

  * Taille du cache (en Go)
  * La référence des machines virtuelles utilisées dans l’infrastructure du cache

  [Get-AzHpcCacheSku](/powershell/module/az.hpccache/get-azhpccachesku) affiche les références SKU disponibles et les options de taille de cache valides pour chacune d’elles. Les options de taille de cache vont de 3 à 48 To, mais seules certaines valeurs sont prises en charge.

  Ce graphique montre les combinaisons de taille de cache et de référence SKU valides au moment de la préparation de ce document (juillet 2020).

  | Taille du cache | Standard_2G | Standard_4G | Standard_8G |
  |------------|-------------|-------------|-------------|
  | 3072 Go    | oui         | Non          | Non          |
  | 6144 Go    | Oui         | Oui         | non          |
  | 12 288 Go   | Oui         | Oui         | Oui         |
  | 24 576 Go   | non          | Oui         | Oui         |
  | 49 152 Go   | non          | Non          | oui         |

  Pour obtenir des informations importantes sur la tarification, le débit et la taille de votre cache en fonction de votre flux de travail, consultez la section **Définir la capacité du cache** sous l’onglet Instructions du portail.

Exemple de création de cache :

```azurepowershell-interactive
$cacheParams = @{
  ResourceGroupName = 'doc-demo-rg'
  CacheName = 'my-cache-0619'
  Location = 'eastus'
  cacheSize = '3072'
  SubnetUri = "/subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.Network/virtualNetworks/vnet-doc0619/subnets/default"
  Sku = 'Standard_2G'
}
New-AzHpcCache @cacheParams
```

La création du cache prend plusieurs minutes. En cas de réussite, la commande create retourne la sortie suivante :

```Output
cacheSizeGb       : 3072
health            : @{state=Healthy; statusDescription=The cache is in Running state}
id                : /subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.StorageCache/caches/my-cache-0619
location          : eastus
mountAddresses    : {10.3.0.17, 10.3.0.18, 10.3.0.19}
name              : my-cache-0619
provisioningState : Succeeded
resourceGroup     : doc-demo-rg
sku               : @{name=Standard_2G}
subnet            : /subscriptions/<subscription-ID>/resourceGroups/doc-demo-rg/providers/Microsoft.Network/virtualNetworks/vnet-doc0619/subnets/default
tags              :
type              : Microsoft.StorageCache/caches
upgradeStatus     : @{currentFirmwareVersion=5.3.42; firmwareUpdateDeadline=1/1/0001 12:00:00 AM; firmwareUpdateStatus=unavailable; lastFirmwareUpdate=4/1/2020 10:19:54 AM; pendingFirmwareVersion=}
```

Le message inclut des informations utiles, notamment les éléments suivants :

* Adresses de montage client : utilisez ces adresses IP lorsque vous êtes prêt à connecter les clients au cache. Pour en savoir plus, consultez [Monter le cache Azure HPC Cache](hpc-cache-mount.md).
* État de la mise à niveau : lorsqu’une mise à jour logicielle est publiée, ce message est modifié. Vous pouvez [mettre à niveau le logiciel de cache](hpc-cache-manage.md#upgrade-cache-software) manuellement à un moment approprié, ou la mise à niveau sera appliquée automatiquement après plusieurs jours.

---

## <a name="next-steps"></a>Étapes suivantes

Une fois votre cache apparaît dans la liste **Ressources**, vous pouvez passer à l’étape suivante.

* [Définissez des cibles de stockage](hpc-cache-add-storage.md) pour permettre à votre cache d’accéder à vos sources de données.
* Si vous utilisez des clés de chiffrement gérées par le client, vous devez [autoriser le chiffrement Azure Key Vault](customer-keys.md#3-authorize-azure-key-vault-encryption-from-the-cache) à partir de la page de présentation du cache pour terminer la configuration de ce dernier. Vous devez effectuer cette étape avant de pouvoir ajouter un stockage. Pour plus d’informations, consultez [Utiliser des clés de chiffrement gérées par le client](customer-keys.md).