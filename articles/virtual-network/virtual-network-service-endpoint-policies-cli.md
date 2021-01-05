---
title: Limiter l’exfiltration de données vers le Stockage Azure - Azure CLI
description: Dans cet article, vous allez apprendre à limiter et restreindre l'exfiltration de données de réseau virtuel aux ressources Stockage Azure avec les stratégies de points de terminaison de service de réseau virtuel utilisant l’interface de ligne de commande Azure (CLI).
services: virtual-network
documentationcenter: virtual-network
author: rdhillon
manager: ''
editor: ''
tags: azure-resource-manager
Customer intent: I want only specific Azure Storage account to be allowed access from a virtual network subnet.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure-services
ms.date: 02/03/2020
ms.author: rdhillon
ms.custom: ''
ms.openlocfilehash: a080c3953c48227301052a0bb151c8cba96e515a
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94737334"
---
# <a name="manage-data-exfiltration-to-azure-storage-accounts-with-virtual-network-service-endpoint-policies-using-the-azure-cli"></a>Gérer l’exfiltration de données vers des comptes Stockage Azure avec des stratégies de points de terminaison de service de réseau virtuel à l’aide d’Azure CLI

Les stratégies de points de terminaison de service de réseau virtuel vous permettent d’appliquer un contrôle d’accès sur des comptes Stockage Azure depuis un réseau virtuel sur des points de terminaison de service. Il s’agit d’une stratégie clé pour sécuriser vos charges de travail, gérer les comptes de stockage autorisés et gérer les destinations d’exfiltration de données autorisées.
Dans cet article, vous apprendrez comment :

* Créer un réseau virtuel et ajouter un sous-réseau.
* Activer un point de terminaison de service pour Stockage Azure.
* Créer deux comptes de stockage Azure et autoriser l’accès réseau à ces derniers à partir du sous-réseau précédemment créé.
* Créer une stratégie de point de terminaison de service pour autoriser l’accès à l’un des comptes de stockage uniquement.
* Déployer une machine virtuelle sur le sous-réseau.
* Vérifier l’accès au compte de stockage autorisé à partir du sous-réseau.
* Vérifier que l’accès au compte de stockage non autorisé à partir du sous-réseau est refusé.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

Avant de créer un réseau virtuel, vous devez créer un groupe de ressources pour le réseau virtuel et toutes les autres ressources créées dans cet article. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli-interactive
az group create \
  --name myResourceGroup \
  --location eastus
```

Créez un réseau virtuel avec un sous-réseau en utilisant la commande [az network vnet create](/cli/azure/network/vnet).

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Private \
  --subnet-prefix 10.0.0.0/24
```

## <a name="enable-a-service-endpoint"></a>Activer un point de terminaison de service 

Dans cet exemple, un point de terminaison *Microsoft.Storage* est créé pour le sous-réseau *Private* : 

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.0.0/24 \
  --service-endpoints Microsoft.Storage
```

## <a name="restrict-network-access-for-a-subnet"></a>Restreindre l’accès réseau d’un sous-réseau

Créez un groupe de sécurité réseau avec la commande [az network nsg create](/cli/azure/network/nsg). L’exemple suivant crée un groupe de sécurité réseau nommé *myNsgPrivate*.

```azurecli-interactive
az network nsg create \
  --resource-group myResourceGroup \
  --name myNsgPrivate
```

Pour associer le groupe de sécurité réseau au sous-réseau *Private*, utilisez [az network vnet subnet update](/cli/azure/network/vnet/subnet). L’exemple suivant associe le groupe de sécurité réseau *myNsgPrivate* au sous-réseau *Private* :

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --name Private \
  --resource-group myResourceGroup \
  --network-security-group myNsgPrivate
```

Créez des règles de sécurité avec [az network nsg rule create](/cli/azure/network/nsg/rule). La règle qui suit autorise un accès sortant vers les adresses IP publiques affectées au service Stockage Azure : 

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Allow-Storage-All \
  --access Allow \
  --protocol "*" \
  --direction Outbound \
  --priority 100 \
  --source-address-prefix "VirtualNetwork" \
  --source-port-range "*" \
  --destination-address-prefix "Storage" \
  --destination-port-range "*"
```

Chaque groupe de sécurité réseau contient plusieurs [règles de sécurité par défaut](security-overview.md#default-security-rules). La règle qui suit remplace une règle de sécurité par défaut, qui autorise l’accès de trafic sortant à toutes les adresses IP publiques. L’option `destination-address-prefix "Internet"` refuse l’accès sortant à toutes les adresses IP publiques. La règle précédente remplace cette règle, du fait de sa priorité plus élevée, ce qui permet d’accéder aux adresses IP publiques du Stockage Azure.

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Deny-Internet-All \
  --access Deny \
  --protocol "*" \
  --direction Outbound \
  --priority 110 \
  --source-address-prefix "VirtualNetwork" \
  --source-port-range "*" \
  --destination-address-prefix "Internet" \
  --destination-port-range "*"
```

La règle suivante autorise le trafic SSH entrant vers le sous-réseau à partir de n’importe quel endroit. La règle remplace une règle de sécurité par défaut qui refuse tout le trafic entrant provenant d’Internet. Le SSH est autorisé sur le sous-réseau afin que la connectivité puisse être testée dans une étape ultérieure.

```azurecli-interactive
az network nsg rule create \
  --resource-group myResourceGroup \
  --nsg-name myNsgPrivate \
  --name Allow-SSH-All \
  --access Allow \
  --protocol Tcp \
  --direction Inbound \
  --priority 120 \
  --source-address-prefix "*" \
  --source-port-range "*" \
  --destination-address-prefix "VirtualNetwork" \
  --destination-port-range "22"
```

## <a name="restrict-network-access-to-azure-storage-accounts"></a>Restreindre l’accès réseau aux comptes Stockage Azure

Cette section répertorie les étapes permettant de limiter l’accès réseau pour un compte Stockage Azure à partir du sous-réseau donné dans un réseau virtuel via un point de terminaison de service.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Créez deux comptes Stockage Azure avec la commande [az storage account create](/cli/azure/storage/account).

```azurecli-interactive
storageAcctName1="allowedstorageacc"

az storage account create \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2

storageAcctName2="notallowedstorageacc"

az storage account create \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2
```

Une fois les comptes de stockage créés, récupérez la chaîne de connexion des comptes de stockage dans une variable avec [az storage account show-connection-string](/cli/azure/storage/account). La chaîne de connexion sera utilisée pour créer un partage de fichiers lors d’une prochaine étape.

```azurecli-interactive
saConnectionString1=$(az storage account show-connection-string \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)

saConnectionString2=$(az storage account show-connection-string \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)
```

<a name="account-key"></a>Affichez le contenu de la variable et notez la valeur de **AccountKey** qui est retournée dans la sortie, car vous devrez l’utiliser dans une prochaine étape.

```azurecli-interactive
echo $saConnectionString1

echo $saConnectionString2
```

### <a name="create-a-file-share-in-the-storage-account"></a>Créer un partage de fichiers dans le compte de stockage

Créez un partage de fichiers dans le compte de stockage avec [az storage share create](/cli/azure/storage/share). Dans une étape ultérieure, ce partage de fichiers sera monté pour vérifier qu’il est possible d’y accéder via le réseau.

```azurecli-interactive
az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString1 > /dev/null

az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString2 > /dev/null
```

### <a name="deny-all-network-access-to-the-storage-account"></a>Refuser tout accès réseau au compte de stockage

Par défaut, les comptes de stockage acceptent les connexions réseau provenant des clients de n’importe quel réseau. Pour limiter l’accès aux réseaux sélectionnés, définissez l’action par défaut sur *Refuser* avec [az storage account update](/cli/azure/storage/account). Une fois l’accès réseau refusé, le compte de stockage n’est plus accessible par aucun des réseaux.

```azurecli-interactive
az storage account update \
  --name $storageAcctName1 \
  --resource-group myResourceGroup \
  --default-action Deny

az storage account update \
  --name $storageAcctName2 \
  --resource-group myResourceGroup \
  --default-action Deny
```

### <a name="enable-network-access-from-virtual-network-subnet"></a>Activer l’accès réseau à partir du sous-réseau de réseau virtuel

Autorisez l’accès réseau au compte de stockage à partir du sous-réseau *Private* avec [az storage account network-rule add](/cli/azure/storage/account/network-rule).

```azurecli-interactive
az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName1 \
  --vnet-name myVirtualNetwork \
  --subnet Private

az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName2 \
  --vnet-name myVirtualNetwork \
  --subnet Private
```

## <a name="apply-policy-to-allow-access-to-valid-storage-account"></a>Appliquer la stratégie pour autoriser l’accès au compte de stockage valide

Les stratégies de points de terminaison de service Azure sont uniquement disponibles pour le Stockage Azure. Pour cet exemple de configuration, nous allons donc activer le point de terminaison de service pour *Microsoft.Storage* sur ce sous-réseau.

Les stratégies de points de terminaison de service sont appliquées sur les points de terminaison de service. Nous allons commencer par créer une stratégie de points de terminaison de service. Après quoi, nous allons créer les définitions de stratégie dans le cadre de cette stratégie pour les comptes Stockage Azure à approuver pour ce sous-réseau.

Créer une stratégie de point de terminaison de service

```azurecli-interactive
az network service-endpoint policy create \
  --resource-group myResourceGroup \
  --name mysepolicy \
  --location eastus
```

Enregistrez l’URI de ressource pour le compte de stockage autorisé dans une variable. Avant d’exécuter la commande ci-dessous, remplacez *\<your-subscription-id>* par la valeur réelle de votre ID d’abonnement.

```azurecli-interactive
$serviceResourceId="/subscriptions/<your-subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/allowedstorageacc"
```

Créer et ajouter une définition de stratégie pour autoriser le compte Stockage Azure ci-dessus dans la stratégie de point de terminaison de service

```azurecli-interactive
az network service-endpoint policy-definition create \
  --resource-group myResourceGroup \
  --policy-name mysepolicy \
  --name mypolicydefinition \
  --service "Microsoft.Storage" \
  --service-resources $serviceResourceId
```

Mettre à jour le sous-réseau de réseau virtuel pour l’associer à la stratégie de points de terminaison de service créée à l’étape précédente

```azurecli-interactive
az network vnet subnet update \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --service-endpoints Microsoft.Storage \
  --service-endpoint-policy mysepolicy
```

## <a name="validate-access-restriction-to-azure-storage-accounts"></a>Vérifier la restriction d’accès aux comptes de stockage Azure

### <a name="create-the-virtual-machine"></a>Créer la machine virtuelle

Pour tester l’accès réseau à un compte de stockage, déployez une machine virtuelle sur le sous-réseau.

Créez une machine virtuelle dans le sous-réseau *Private* avec [az vm create](/cli/azure/vm). Si des clés SSH n’existent pas déjà dans un emplacement de clé par défaut, la commande les crée. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPrivate \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Private \
  --generate-ssh-keys
```

La création de la machine virtuelle ne nécessite que quelques minutes. Une fois la machine virtuelle créée, prenez note de la valeur de **publicIpAddress** dans la sortie retournée. Cette adresse sera utilisée pour accéder à la machine virtuelle à partir d’Internet dans une prochaine étape.

### <a name="confirm-access-to-storage-account"></a>Vérifier l’accès au compte de stockage

Ouvrez une session SSH avec la machine virtuelle *myVmPrivate*. Remplacez *\<publicIpAddress>* par l’adresse IP publique de votre machine virtuelle *myVmPrivate*.

```bash 
ssh <publicIpAddress>
```

Créez un dossier comme point de montage :

```bash
sudo mkdir /mnt/MyAzureFileShare1
```

Montez le partage de fichiers Azure dans le répertoire que vous avez créé. Avant d’exécuter la commande ci-dessous, remplacez *\<storage-account-key>* par la valeur *AccountKey* de **$saConnectionString 1**.

```bash
sudo mount --types cifs //allowedstorageacc.file.core.windows.net/my-file-share /mnt/MyAzureFileShare1 --options vers=3.0,username=allowedstorageacc,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

Vous recevez l’invite `user@myVmPrivate:~$`. Le partage de fichiers Azure est monté sur */mnt/MyAzureFileShare*.

### <a name="confirm-access-is-denied-to-storage-account"></a>Vérifier que l’accès au compte de stockage est refusé

À partir de la même machine virtuelle *myVmPrivate*, créez un répertoire pour un point de montage :

```bash
sudo mkdir /mnt/MyAzureFileShare2
```

Essayez de monter le partage de fichiers Azure à partir du compte de stockage *notallowedstorageacc* dans le répertoire que vous avez créé. Cet article suppose que vous ayez déployé la dernière version d’Ubuntu. Si vous utilisez une version antérieure d’Ubuntu, consultez [Montage sur Linux](../storage/files/storage-how-to-use-files-linux.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour en savoir plus sur le montage des partages de fichiers. 

Avant d’exécuter la commande ci-dessous, remplacez *\<storage-account-key>* par la valeur *AccountKey* de **$saConnectionString 2**.

```bash
sudo mount --types cifs //notallowedstorageacc.file.core.windows.net/my-file-share /mnt/MyAzureFileShare2 --options vers=3.0,username=notallowedstorageacc,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

L’accès est refusé et vous recevez une erreur `mount error(13): Permission denied`, car ce compte de stockage ne figure pas dans la liste verte de la stratégie de points de terminaison de service que nous avons appliquée au sous-réseau. 

Fermez la session SSH sur la machine virtuelle *myVmPublic*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, utilisez [az group delete](/cli/azure) pour le supprimer, ainsi que toutes les ressources qu’il contient.

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de cet article, vous avez appliqué une stratégie de point de terminaison de service sur un point de terminaison de service de réseau virtuel Azure au stockage Azure. Vous avez créé des comptes de stockage Azure et vous avez limité l’accès réseau à certains comptes de stockage (et refusé l’accès à d’autres) à partir d’un sous-réseau de réseau virtuel. Pour en savoir plus sur les stratégies de point de terminaison de service, consultez [Vue d’ensemble des stratégies de points de terminaison de service](virtual-network-service-endpoint-policies-overview.md).
