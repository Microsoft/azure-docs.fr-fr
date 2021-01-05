---
title: Limiter l’accès réseau aux ressources PaaS - Azure CLI
description: Dans cet article, vous allez apprendre à limiter et restreindre l’accès réseau aux ressources Azure, telles que Stockage Azure et Azure SQL Database, avec les points de terminaison de service de réseau virtuel à l’aide de l’interface de ligne de commande Azure (CLI).
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
Customer intent: I want only resources in a virtual network subnet to access an Azure PaaS resource, such as an Azure Storage account.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure-services
ms.date: 03/14/2018
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1a1eab081a7c1e83a57ef4735c6eb5248d92defc
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94734104"
---
# <a name="restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-the-azure-cli"></a>Restreindre l’accès réseau aux ressources PaaS avec des points de terminaison de service de réseau virtuel en utilisant Azure CLI

Les points de terminaison de service de réseau virtuel permettent de restreindre l’accès réseau à certaines ressources du service Azure en n’autorisant leur accès qu’à partir d’un sous-réseau du réseau virtuel. Vous pouvez également supprimer l’accès Internet aux ressources. Les points de terminaison de service fournissent une connexion directe entre votre réseau virtuel et les services Azure pris en charge, ce qui vous permet d’utiliser l’espace d’adressage privé de votre réseau virtuel pour accéder aux services Azure. Le trafic destiné aux ressources Azure via les points de terminaison de service reste toujours sur le serveur principal de Microsoft Azure. Dans cet article, vous apprendrez comment :

* Créer un réseau virtuel avec un sous-réseau
* Ajouter un sous-réseau et activer un point de terminaison de service
* Créer une ressource Azure et autoriser l’accès réseau à cette ressource uniquement à partir d’un sous-réseau
* Déployer une machine virtuelle sur chaque sous-réseau
* Vérifier l’accès à une ressource à partir d’un sous-réseau
* Vérifier que l’accès à une ressource est refusé à partir d’un sous-réseau et d’Internet

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
  --subnet-name Public \
  --subnet-prefix 10.0.0.0/24
```

## <a name="enable-a-service-endpoint"></a>Activer un point de terminaison de service 

Vous ne pouvez activer des points de terminaison de service que pour les services qui prennent en charge les points de terminaison de service. Si vous souhaitez voir les services d’un emplacement Azure donné pour lesquels les points de terminaison de service ont été activés, utilisez [az network vnet list-endpoint-services](/cli/azure/network/vnet). L’exemple suivant retourne la liste des services de la région *eastus* pour lesquels les points de terminaison de service ont été activés. La liste des services retournés augmente avec chaque nouvelle activation des points de terminaison de service.

```azurecli-interactive
az network vnet list-endpoint-services \
  --location eastus \
  --out table
``` 

Créez un sous-réseau supplémentaire dans le réseau virtuel avec [az network vnet subnet create](/cli/azure/network/vnet/subnet). Dans cet exemple, un point de terminaison *Microsoft.Storage* est créé pour le sous-réseau : 

```azurecli-interactive
az network vnet subnet create \
  --vnet-name myVirtualNetwork \
  --resource-group myResourceGroup \
  --name Private \
  --address-prefix 10.0.1.0/24 \
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

## <a name="restrict-network-access-to-a-resource"></a>Restreindre l’accès réseau à une ressource

Les étapes nécessaires pour restreindre l’accès réseau aux ressources créées par le biais des services Azure avec activation des points de terminaison varient d’un service à l’autre. Pour connaître les étapes à suivre, consultez la documentation relative à chacun des services. La suite de cet article comprend des étapes permettant de restreindre l’accès réseau pour un compte Stockage Azure.

### <a name="create-a-storage-account"></a>Créez un compte de stockage.

Créez un compte de stockage Azure avec la commande [az storage account create](/cli/azure/storage/account). Remplacez `<replace-with-your-unique-storage-account-name>` par un nom qui n’existe dans aucun autre emplacement Azure. Le nom doit comprendre entre 3 et 24 caractères, correspondant à des chiffres et à des lettres en minuscules.

```azurecli-interactive
storageAcctName="<replace-with-your-unique-storage-account-name>"

az storage account create \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --sku Standard_LRS \
  --kind StorageV2
```

Une fois le compte de stockage créé, récupérez la chaîne de connexion du compte de stockage dans une variable avec [az storage account show-connection-string](/cli/azure/storage/account). La chaîne de connexion sera utilisée pour créer un partage de fichiers lors d’une prochaine étape.

```azurecli-interactive
saConnectionString=$(az storage account show-connection-string \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --query 'connectionString' \
  --out tsv)
```

<a name="account-key"></a>Affichez le contenu de la variable et notez la valeur de **AccountKey** qui est retournée dans la sortie, car vous devrez l’utiliser dans une prochaine étape.

```azurecli-interactive
echo $saConnectionString
```

### <a name="create-a-file-share-in-the-storage-account"></a>Créer un partage de fichiers dans le compte de stockage

Créez un partage de fichiers dans le compte de stockage avec [az storage share create](/cli/azure/storage/share). Dans une étape ultérieure, ce partage de fichiers sera monté pour vérifier qu’il est possible d’y accéder via le réseau.

```azurecli-interactive
az storage share create \
  --name my-file-share \
  --quota 2048 \
  --connection-string $saConnectionString > /dev/null
```

### <a name="deny-all-network-access-to-a-storage-account"></a>Refuser tout accès réseau au compte de stockage

Par défaut, les comptes de stockage acceptent les connexions réseau provenant des clients de n’importe quel réseau. Pour limiter l’accès aux réseaux sélectionnés, définissez l’action par défaut sur *Refuser* avec [az storage account update](/cli/azure/storage/account). Une fois l’accès réseau refusé, le compte de stockage n’est plus accessible par aucun des réseaux.

```azurecli-interactive
az storage account update \
  --name $storageAcctName \
  --resource-group myResourceGroup \
  --default-action Deny
```

### <a name="enable-network-access-from-a-subnet"></a>Activer l’accès réseau à partir d’un sous-réseau

Autorisez l’accès réseau au compte de stockage à partir du sous-réseau *Private* avec [az storage account network-rule add](/cli/azure/storage/account/network-rule).

```azurecli-interactive
az storage account network-rule add \
  --resource-group myResourceGroup \
  --account-name $storageAcctName \
  --vnet-name myVirtualNetwork \
  --subnet Private
```
## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Pour tester l’accès réseau à un compte de stockage, déployez une machine virtuelle sur chaque sous-réseau.

### <a name="create-the-first-virtual-machine"></a>Créer la première machine virtuelle

Créez une machine virtuelle dans le sous-réseau *Public* avec [az vm create](/cli/azure/vm). Si des clés SSH n’existent pas déjà dans un emplacement de clé par défaut, la commande les crée. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVmPublic \
  --image UbuntuLTS \
  --vnet-name myVirtualNetwork \
  --subnet Public \
  --generate-ssh-keys
```

La création de la machine virtuelle ne nécessite que quelques minutes. Une fois la machine virtuelle créée, l’interface CLI Azure affiche des informations similaires à celles de l’exemple suivant : 

```azurecli 
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVmPublic",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "13.90.242.231",
  "resourceGroup": "myResourceGroup"
}
```

Prenez note de la valeur de **publicIpAddress** dans la sortie retournée. Cette adresse sera utilisée pour accéder à la machine virtuelle à partir d’Internet dans une prochaine étape.

### <a name="create-the-second-virtual-machine"></a>Créer la deuxième machine virtuelle

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

## <a name="confirm-access-to-storage-account"></a>Vérifier l’accès au compte de stockage

Ouvrez une session SSH avec la machine virtuelle *myVmPrivate*. Remplacez *\<publicIpAddress>* par l’adresse IP publique de votre machine virtuelle *myVmPrivate*.

```bash 
ssh <publicIpAddress>
```

Créez un dossier comme point de montage :

```bash
sudo mkdir /mnt/MyAzureFileShare
```

Montez le partage de fichiers Azure dans le répertoire que vous avez créé. Avant d’exécuter la commande suivante, remplacez `<storage-account-name>` par le nom du compte et `<storage-account-key>` par la clé que vous avez récupérée dans [Créer un compte de stockage](#create-a-storage-account).

```bash
sudo mount --types cifs //<storage-account-name>.file.core.windows.net/my-file-share /mnt/MyAzureFileShare --options vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

Vous recevez l’invite `user@myVmPrivate:~$`. Le partage de fichiers Azure est monté sur */mnt/MyAzureFileShare*.

Vérifiez que la machine virtuelle ne dispose d’aucune connexion sortante à d’autres adresses IP publiques :

```bash
ping bing.com -c 4
```

Vous ne recevez aucune réponse, car le groupe de sécurité réseau associé au sous-réseau *Private* n’autorise pas l’accès sortant aux adresses IP publiques autres que les adresses affectées au service Stockage Azure.

Fermez la session SSH sur la machine virtuelle *myVmPrivate*.

## <a name="confirm-access-is-denied-to-storage-account"></a>Vérifier que l’accès au compte de stockage est refusé

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle *myVmPublic*. Remplacez `<publicIpAddress>` par l’adresse IP publique de votre machine virtuelle *myVmPublic* : 

```bash 
ssh <publicIpAddress>
```

Créez un répertoire comme point de montage :

```bash
sudo mkdir /mnt/MyAzureFileShare
```

Essayez de monter le partage de fichiers Azure dans le répertoire que vous avez créé. Cet article suppose que vous ayez déployé la dernière version d’Ubuntu. Si vous utilisez une version antérieure d’Ubuntu, consultez [Montage sur Linux](../storage/files/storage-how-to-use-files-linux.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour en savoir plus sur le montage des partages de fichiers. Avant d’exécuter la commande suivante, remplacez `<storage-account-name>` par le nom du compte et `<storage-account-key>` par la clé que vous avez récupérée dans [Créer un compte de stockage](#create-a-storage-account) :

```bash
sudo mount --types cifs //storage-account-name>.file.core.windows.net/my-file-share /mnt/MyAzureFileShare --options vers=3.0,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino
```

L’accès est refusé et vous recevez une erreur `mount error(13): Permission denied`, car la machine virtuelle *myVmPublic* est déployée sur le sous-réseau *Public*. Le sous-réseau *Public* ne dispose pas d’un point de terminaison de service activé pour Stockage Azure, et le compte de stockage autorise uniquement l’accès réseau à partir du sous-réseau *Private* et non à partir du sous-réseau *Public*.

Fermez la session SSH sur la machine virtuelle *myVmPublic*.

Sur votre ordinateur, essayez d’afficher les partages de votre compte de stockage avec [az storage share list](/cli/azure/storage/share?view=azure-cli-latest). Remplacez `<account-name>` et `<account-key>` par le nom de compte de stockage et la clé utilisés dans la section [Créer un compte de stockage](#create-a-storage-account) :

```azurecli-interactive
az storage share list \
  --account-name <account-name> \
  --account-key <account-key>
```

L’accès est refusé et vous recevez une erreur indiquant que *la requête n’est pas autorisée à effectuer cette opération*, car votre ordinateur ne fait pas partie du sous-réseau *Private* du réseau virtuel *MyVirtualNetwork*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin d’un groupe de ressources, utilisez [az group delete](/cli/azure) pour le supprimer, ainsi que toutes les ressources qu’il contient.

```azurecli-interactive 
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez activé un point de terminaison de service pour un sous-réseau de réseau virtuel. Vous avez vu que les points de terminaison de service peuvent être activés pour les ressources déployées à l’aide de différents services Azure. Vous avez créé un compte Stockage Azure et un accès réseau à ce compte de stockage, limité aux ressources du sous-réseau du réseau virtuel. Pour en savoir plus sur les points de terminaison de service, consultez [Points de terminaison de service de réseau virtuel](virtual-network-service-endpoints-overview.md) et [Ajouter, modifier ou supprimer un sous-réseau de réseau virtuel](virtual-network-manage-subnet.md).

Si votre compte comporte plusieurs réseaux virtuels, vous pouvez relier deux réseaux virtuels pour que les ressources de chaque réseau virtuel puissent communiquer entre elles. Pour connaître la marche à suivre, consultez [Connecter des réseaux virtuels](tutorial-connect-virtual-networks-cli.md).
