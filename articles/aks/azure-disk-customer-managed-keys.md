---
title: Utiliser une clé gérée par le client pour chiffrer les disques Azure dans Azure Kubernetes Service (AKS)
description: BYOK (Bring Your Own Keys) pour chiffrer les disques de données et de système d’exploitation AKS.
services: container-service
ms.topic: article
ms.date: 09/01/2020
ms.openlocfilehash: c71052f44f6912006a952bae16fd89ec1dc937db
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607889"
---
# <a name="bring-your-own-keys-byok-with-azure-disks-in-azure-kubernetes-service-aks"></a>BYOK (Bring Your Own Keys) avec des disques Azure dans Azure Kubernetes Service (AKS)

Le stockage Azure chiffre toutes les données dans un compte de stockage au repos. Par défaut, les données sont chiffrées avec des clés managées par Microsoft Pour renforcer le contrôle sur les clés de chiffrement, vous pouvez fournir des clés managées par le client à utiliser pour le chiffrement au repos des disques de système d’exploitation et des disques de données pour vos clusters AKS. En savoir plus sur les clés gérées par le client sur [Linux][customer-managed-keys-linux] et [Windows][customer-managed-keys-windows].

## <a name="limitations"></a>Limites
* La prise en charge du chiffrement de disque de données est limitée aux clusters AKS exécutant Kubernetes 1.17 et les versions ultérieures.
* Le chiffrement du système d’exploitation et du disque de données avec des clés gérées par le client ne peut être activé que lors de la création d’un cluster AKS.

## <a name="prerequisites"></a>Prérequis
* Vous devez activer la suppression réversible et la protection contre le vidage pour *Azure Key Vault* lors de l’utilisation de Key Vault pour chiffrer des disques managés.
* Vous devez disposer d’Azure CLI 2.11.1 ou d’un version ultérieure.

## <a name="create-an-azure-key-vault-instance"></a>Créer une instance Azure Key Vault

Utilisez une instance Azure Key Vault pour stocker vos clés.  Vous pouvez éventuellement utiliser le portail Azure pour [configurer des clés gérées par le client avec Azure Key Vault][byok-azure-portal]

Créez un *groupe de ressources*, puis une instance *Key Vault*. Activez ensuite la suppression réversible et la protection contre le vidage.  Veillez à utiliser les mêmes noms de régions et de groupes de ressources pour chaque commande.

```azurecli-interactive
# Optionally retrieve Azure region short names for use on upcoming commands
az account list-locations
```

```azurecli-interactive
# Create new resource group in a supported Azure region
az group create -l myAzureRegionName -n myResourceGroup

# Create an Azure Key Vault resource in a supported Azure region
az keyvault create -n myKeyVaultName -g myResourceGroup -l myAzureRegionName  --enable-purge-protection true --enable-soft-delete true
```

## <a name="create-an-instance-of-a-diskencryptionset"></a>Créer une instance de DiskEncryptionSet

Remplacez *myKeyVaultName* par le nom de votre coffre de clés.  Vous devez également disposer d’une *clé* stockée dans Azure Key Vault pour effectuer les étapes suivantes.  Stockez votre clé existante dans le coffre de clés que vous avez créé au cours des étapes précédentes, ou [générez une nouvelle clé][key-vault-generate] et remplacez *myKeyName* ci-dessous par son nom.
    
```azurecli-interactive
# Retrieve the Key Vault Id and store it in a variable
keyVaultId=$(az keyvault show --name myKeyVaultName --query "[id]" -o tsv)

# Retrieve the Key Vault key URL and store it in a variable
keyVaultKeyUrl=$(az keyvault key show --vault-name myKeyVaultName  --name myKeyName  --query "[key.kid]" -o tsv)

# Create a DiskEncryptionSet
az disk-encryption-set create -n myDiskEncryptionSetName  -l myAzureRegionName  -g myResourceGroup --source-vault $keyVaultId --key-url $keyVaultKeyUrl 
```

## <a name="grant-the-diskencryptionset-access-to-key-vault"></a>Accorder à DiskEncryptionSet l’accès au coffre de clés

Utilisez DiskEncryptionSet et les groupes de ressources que vous avez créés lors des étapes précédentes, puis accordez à la ressource DiskEncryptionSet l’accès au coffre de clés Azure.

```azurecli-interactive
# Retrieve the DiskEncryptionSet value and set a variable
desIdentity=$(az disk-encryption-set show -n myDiskEncryptionSetName  -g myResourceGroup --query "[identity.principalId]" -o tsv)

# Update security policy settings
az keyvault set-policy -n myKeyVaultName -g myResourceGroup --object-id $desIdentity --key-permissions wrapkey unwrapkey get
```

## <a name="create-a-new-aks-cluster-and-encrypt-the-os-disk"></a>Créer un cluster AKS et chiffrer le disque de système d’exploitation

Créez un **groupe de ressources** et un cluster AKS, puis utilisez votre clé pour chiffrer le disque de système d’exploitation. Les clés gérées par le client sont prises en charge uniquement dans les versions de Kubernetes postérieures à la version 1.17. 

> [!IMPORTANT]
> Veillez à créer un nouveau groupe de ressources pour votre cluster AKS

```azurecli-interactive
# Retrieve the DiskEncryptionSet value and set a variable
diskEncryptionSetId=$(az disk-encryption-set show -n mydiskEncryptionSetName -g myResourceGroup --query "[id]" -o tsv)

# Create a resource group for the AKS cluster
az group create -n myResourceGroup -l myAzureRegionName

# Create the AKS cluster
az aks create -n myAKSCluster -g myResourceGroup --node-osdisk-diskencryptionset-id $diskEncryptionSetId --kubernetes-version KUBERNETES_VERSION --generate-ssh-keys
```

Quand de nouveaux pools de nœuds sont ajoutés au cluster créé ci-dessus, la clé gérée par le client fournie pendant l’opération de création est utilisée pour chiffrer le disque de système d’exploitation.

## <a name="encrypt-your-aks-cluster-data-diskoptional"></a>Chiffrez votre disque de données de cluster AKS (facultatif)
La clé de chiffrement du disque du système d’exploitation sera utilisée pour chiffrer le disque de données si la clé n’est pas fournie pour le disque de données à partir de v 1.17.2, et vous pouvez également chiffrer les disques de données AKS avec les autres clés.

> [!IMPORTANT]
> Vérifiez que vous disposez des informations d’identification AKS appropriées. Le principal du service doit disposer d’un accès de contributeur au groupe de ressources où diskencryptionset est déployé. Sinon, vous obtenez une erreur suggérant que le principal du service ne dispose pas d’autorisations.

```azurecli-interactive
# Retrieve your Azure Subscription Id from id property as shown below
az account list
```

```
someuser@Azure:~$ az account list
[
  {
    "cloudName": "AzureCloud",
    "id": "666e66d8-1e43-4136-be25-f25bb5de5893",
    "isDefault": true,
    "name": "MyAzureSubscription",
    "state": "Enabled",
    "tenantId": "3ebbdf90-2069-4529-a1ab-7bdcb24df7cd",
    "user": {
      "cloudShellID": true,
      "name": "someuser@azure.com",
      "type": "user"
    }
  }
]
```

Créez un fichier appelé **byok-azure-disk.yaml** et contenant les informations suivantes.  Remplacez myAzureSubscriptionId, myResourceGroup et myDiskEncrptionSetName par vos valeurs, puis appliquez le fichier YAML.  Veillez à utiliser le groupe de ressources dans lequel votre ressource DiskEncryptionSet est déployée.  Si vous utilisez Azure Cloud Shell, vous pouvez créer ce fichier à l’aide de vi ou de nano comme si vous travailliez sur un système virtuel ou physique :

```
kind: StorageClass
apiVersion: storage.k8s.io/v1  
metadata:
  name: hdd
provisioner: kubernetes.io/azure-disk
parameters:
  skuname: Standard_LRS
  kind: managed
  diskEncryptionSetID: "/subscriptions/{myAzureSubscriptionId}/resourceGroups/{myResourceGroup}/providers/Microsoft.Compute/diskEncryptionSets/{myDiskEncryptionSetName}"
```
Ensuite, exécutez ce déploiement dans votre cluster AKS :
```azurecli-interactive
# Get credentials
az aks get-credentials --name myAksCluster --resource-group myResourceGroup --output table

# Update cluster
kubectl apply -f byok-azure-disk.yaml
```

## <a name="next-steps"></a>Étapes suivantes

Consulter les [bonnes pratiques relatives à la sécurité des clusters AKS][best-practices-security]

<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: ./operator-best-practices-cluster-security.md
[byok-azure-portal]: ../storage/common/customer-managed-keys-configure-key-vault.md
[customer-managed-keys-windows]: ../virtual-machines/disk-encryption.md#customer-managed-keys
[customer-managed-keys-linux]: ../virtual-machines/disk-encryption.md#customer-managed-keys
[key-vault-generate]: ../key-vault/general/manage-with-cli2.md
[supported-regions]: ../virtual-machines/windows/disk-encryption.md#supported-regions
