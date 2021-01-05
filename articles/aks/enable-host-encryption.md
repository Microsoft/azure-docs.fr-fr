---
title: Activer le chiffrement basé sur l’hôte sur Azure Kubernetes Service (AKS)
description: Découvrez comment configurer un chiffrement basé sur l’hôte dans un cluster Azure Kubernetes service (AKS)
services: container-service
ms.topic: article
ms.date: 07/10/2020
ms.openlocfilehash: 14ec39272bf2f434aaa57217a90667a62e82901a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183292"
---
# <a name="host-based-encryption-on-azure-kubernetes-service-aks-preview"></a>Activer le chiffrement basé sur l’hôte sur Azure Kubernetes Service (AKS) (préversion)

Avec le chiffrement basé sur l’hôte, les données stockées sur l’hôte de machine virtuelle des machines virtuelles de vos nœuds d’agent AKS sont chiffrées au repos et les flux sont chiffrés dans le service de stockage. Cela signifie que les disques temporaires sont chiffrés au repos avec des clés gérées par la plateforme. Le cache du système d’exploitation et des disques de données est chiffré au repos avec des clés gérées par le client ou par la plateforme, selon le type de chiffrement défini sur ces disques. Par défaut, lorsque vous utilisez AKS, le système d’exploitation et les disques de données sont chiffrés au repos avec des clés gérées par la plateforme, ce qui signifie que les caches de ces disques sont également chiffrés au repos par défaut avec des clés gérées par la plateforme.  Vous pouvez spécifier vos propres clés gérées à l’aide de [Bring your own keys (BYOK) avec des disques Azure dans Azure Kubernetes service](azure-disk-customer-managed-keys.md). Le cache de ces disques sera également chiffré à l’aide de la clé que vous spécifiez au cours de cette étape.


## <a name="before-you-begin"></a>Avant de commencer

Cette fonctionnalité ne peut être définie qu’au moment de la création du cluster ou de la création du pool de nœuds.

> [!NOTE]
> Le chiffrement basé sur l’hôte est disponible dans les [régions Azure][supported-regions] qui prennent en charge le chiffrement côté serveur des disques managés Azure et uniquement avec des [tailles de machine virtuelle spécifiques prises en charge][supported-sizes].

### <a name="prerequisites"></a>Prérequis

- Vérifiez que l’extension CLI `aks-preview` version 0.4.55 ou ultérieure est installée
- Vérifiez que l’indicateur de fonctionnalité `EncryptionAtHost` sous `Microsoft.Compute` est activé.
- Vérifiez que l’indicateur de fonctionnalité `EnableEncryptionAtHostPreview` sous `Microsoft.ContainerService` est activé.

### <a name="register-encryptionathost--preview-features"></a>Enregistrer `EncryptionAtHost` fonctionnalités d’évaluation

Pour créer un cluster AKS qui utilise un chiffrement basé sur l’hôte, vous devez activer les indicateurs de fonctionnalité `EnableEncryptionAtHostPreview` et `EncryptionAtHost` sur votre abonnement.

Inscrivez l’indicateur de fonctionnalité `EncryptionAtHost` à l’aide de la commande [az feature register][az-feature-register], comme indiqué dans l’exemple suivant :

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "EncryptionAtHost"

az feature register --namespace "Microsoft.ContainerService"  --name "EnableEncryptionAtHostPreview"
```

Quelques minutes sont nécessaires pour que l’état s’affiche *Registered* (Inscrit). Vous pouvez vérifier l’état de l’enregistrement à l’aide de la commande [az feature list][az-feature-list] :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.Compute/EncryptionAtHost')].{Name:name,State:properties.state}"

az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableEncryptionAtHostPreview')].{Name:name,State:properties.state}"
```

Lorsque vous êtes prêt, actualisez l'enregistrement du fournisseur de ressources `Microsoft.ContainerService` et `Microsoft.Compute` à l'aide de la commande [az provider register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.Compute

az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="install-aks-preview-cli-extension"></a>Installer l’extension CLI de préversion d’aks

Pour créer un cluster AKS qui héberge le chiffrement basé sur l’hôte, vous avez besoin de la dernière extension CLI *aks-preview*. Installez l’extension Azure CLI *aks-preview* à l’aide de la commande [az extension add][az-extension-add], ou recherchez toutes les mises à jour disponibles à l’aide de la commande [az extension update][az-extension-update] :

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="limitations"></a>Limites

- Ne peut être activé que sur les nouveaux pools de nœuds ou les nouveaux clusters.
- Ne peut être activé que dans les [régions Azure][supported-regions] qui prennent en charge le chiffrement côté serveur des disques managés Azure et uniquement avec des [tailles de machine virtuelle spécifiques prises en charge][supported-sizes].
- Requiert un cluster AKS et un pool de nœuds basés sur Virtual Machine Scale Sets (VMSS) comme *type d’ensemble de machines virtuelles*.

## <a name="use-host-based-encryption-on-new-clusters-preview"></a>Utiliser le chiffrement basé sur l’hôte sur les nouveaux clusters (préversion)

Configurez les nœuds de l’agent de cluster pour utiliser le chiffrement basé sur l’hôte lorsque le cluster est créé. Utilisez l’indicateur `--aks-custom-headers` pour définir l’en-tête `EnableEncryptionAtHost`.

```azurecli-interactive
az aks create --name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --aks-custom-headers EnableEncryptionAtHost=true
```

Si vous souhaitez créer des clusters sans chiffrement basé sur l’hôte, vous pouvez le faire en omettant le paramètre `--aks-custom-headers` personnalisé.

## <a name="use-host-based-encryption-on-existing-clusters-preview"></a>Utiliser le chiffrement basé sur l’hôte sur les clusters existants (préversion)

Vous pouvez activer le chiffrement basé sur l’hôte sur les clusters existants en ajoutant un nouveau pool de nœuds à votre cluster. Configurez un nouveau pool de nœuds pour utiliser le chiffrement basé sur l’hôte à l’aide de l’indicateur `--aks-custom-headers`.

```azurecli
az aks nodepool add --name hostencrypt --cluster-name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --aks-custom-headers EnableEncryptionAtHost=true
```

Si vous souhaitez créer de nouveaux pools de nœuds sans la fonctionnalité de chiffrement basé sur l’hôte, vous pouvez le faire en omettant le paramètre `--aks-custom-headers` personnalisé.

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les [meilleures pratiques pour la sécurité du cluster AKS][best-practices-security] En savoir plus sur le [chiffrement basé sur l’hôte](../virtual-machines/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).


<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: ./operator-best-practices-cluster-security.md
[supported-regions]: ../virtual-machines/disk-encryption.md#supported-regions
[supported-sizes]: ../virtual-machines/disk-encryption.md#supported-vm-sizes
[azure-cli-install]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register