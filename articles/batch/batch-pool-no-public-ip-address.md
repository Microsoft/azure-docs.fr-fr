---
title: Créer un pool Azure Batch sans adresses IP publiques
description: Découvrez comment créer un pool sans adresses IP publiques
author: pkshultz
ms.topic: how-to
ms.date: 10/08/2020
ms.author: peshultz
ms.custom: references_regions
ms.openlocfilehash: 09a5632f969117e69e68bbe0df2bfbab9a8a102b
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842133"
---
# <a name="create-an-azure-batch-pool-without-public-ip-addresses"></a>Créer un pool Azure Batch sans adresses IP publiques

Lorsque vous créez un pool Azure Batch, vous pouvez provisionner le pool de configuration de machine virtuelle sans adresse IP publique. Cet article explique comment configurer un pool Batch sans adresses IP publiques.

## <a name="why-use-a-pool-without-public-ip-addresses"></a>Pourquoi utiliser un pool sans adresses IP publiques ?

Par défaut, une adresse IP publique est affectée à tous les nœuds de calcul d’un pool de configuration de machine virtuelle Azure Batch. Cette adresse est utilisée par le service Batch pour planifier des tâches et pour communiquer avec les nœuds de calcul, notamment dans le cadre de l’accès sortant à Internet.

Pour restreindre l’accès à ces nœuds et réduire la détectabilité de ces nœuds à partir d’Internet, vous pouvez provisionner le pool sans adresses IP publiques.

> [!IMPORTANT]
> La prise en charge des pools sans adresses IP publiques dans Azure Batch est actuellement en préversion publique pour les régions suivantes : France Centre, Asie Est, USA Centre-Ouest, USA Centre Sud, USA Ouest 2, USA Est, Europe Nord, USA Est 2, USA Centre, Europe Ouest, USA Centre Nord, USA Ouest, Australie Est, Japon Est, Japon Ouest.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

- **Authentification**. Pour utiliser un pool sans adresses IP publiques au sein d’un [réseau virtuel](./batch-virtual-network.md), l’API du client Batch doit utiliser l’authentification Azure Active Directory (AD). La prise en charge d’Azure Batch pour Azure AD est documentée dans [Authentifier les solutions de service Batch avec Active Directory](batch-aad-auth.md). Si vous ne créez pas votre pool au sein d’un réseau virtuel, vous pouvez utiliser l’authentification Azure AD ou l’authentification basée sur les clés.

- **Un réseau virtuel Azure**. Si vous créez votre pool dans un [réseau virtuel](batch-virtual-network.md), vous devez respecter les exigences et appliquer les configurations qui suivent. Pour préparer un réseau virtuel avec un ou plusieurs sous-réseaux à l’avance, vous pouvez utiliser le portail Azure, Azure PowerShell, l’interface de ligne de commande Azure (CLI) ou d’autres méthodes.
  - Le réseau virtuel doit se trouver dans la même région et le même abonnement que le compte Batch utilisé pour créer le pool.
  - Le sous-réseau spécifié pour le pool doit avoir suffisamment d’adresses IP non attribuées pour contenir le nombre de machines virtuelles ciblées pour le pool, autrement dit, la somme des propriétés `targetDedicatedNodes` et `targetLowPriorityNodes` du pool. Si le sous-réseau ne dispose pas de suffisamment d’adresses IP non attribuées, le pool alloue partiellement les nœuds de calcul, et une erreur de redimensionnement se produit.
  - Vous devez désactiver les stratégies relatives aux services de liaison privée et aux réseaux de points de terminaison. Pour cela, vous pouvez utiliser Azure CLI : ```az network vnet subnet update --vnet-name <vnetname> -n <subnetname> --resouce-group <resourcegroup> --disable-private-endpoint-network-policies --disable-private-link-service-network-policies```

> [!IMPORTANT]
> Pour chaque groupe de 100 nœuds dédiés ou basse priorité, Batch alloue un service de liaison privée et un équilibreur de charge. Ces ressources sont limitées par les [quotas de ressources](../azure-resource-manager/management/azure-subscription-service-limits.md) de l’abonnement. Pour les grands pools, vous devrez peut-être [demander une augmentation du quota](batch-quota-limit.md#increase-a-quota) pour une ou plusieurs de ces ressources. En outre, aucun verrou de ressource ne doit être appliqué à une ressource créée par Batch, puisque cela empêche le nettoyage des ressources à la suite d’actions lancées par l’utilisateur, telles que la suppression d’un pool ou le redimensionnement à zéro.

## <a name="current-limitations"></a>Limites actuelles

1. Les pools sans adresses IP publiques doivent utiliser la configuration de machine virtuelle et non la configuration des services cloud.
1. La [configuration des points de terminaison personnalisés](pool-endpoint-configuration.md) pour les nœuds de calcul Batch ne fonctionne pas avec les pools qui n’ont pas d’adresses IP publiques.
1. Étant donné que ce type de pool n’a pas d’adresses IP publiques, vous ne pouvez pas [utiliser vos propres adresses IP publiques](create-pool-public-ip.md).

## <a name="create-a-pool-without-public-ip-addresses-in-the-azure-portal"></a>Créer un pool sans adresses IP publiques dans le portail Azure

1. Accédez à votre compte  Batch dans le portail Azure.
1. Dans la fenêtre **Paramètres** située à gauche, sélectionnez **Pools**.
1. Dans la fenêtre **Pools**, sélectionnez **Ajouter**.
1. Dans la fenêtre **Ajouter un pool**, sélectionnez l’option que vous souhaitez utiliser à partir de la liste déroulante **Type d’image**.
1. Sélectionnez le **serveur de publication, l’offre ou la référence SKU** qui correspond à votre image.
1. Spécifiez les autres paramètres obligatoires, notamment **Taille de nœud**, **Nœuds dédiés cibles** et **Nœuds basse priorité** et éventuellement les paramètres facultatifs.
1. Vous pouvez également sélectionner le réseau virtuel et le sous-réseau que vous souhaitez utiliser. Ce réseau virtuel doit se trouver dans le même groupe de ressources que le pool que vous créez.
1. Dans **Type de provisionnement des adresses IP**, sélectionnez **NoPublicIPAddresses**.

![Capture de l’écran Ajouter un pool avec l’option NoPublicIPAddresses sélectionnée.](./media/batch-pool-no-public-ip-address/create-pool-without-public-ip-address.png)

## <a name="use-the-batch-rest-api-to-create-a-pool-without-public-ip-addresses"></a>Utiliser l’API REST Batch pour créer un pool sans adresses IP publiques

L’exemple ci-dessous montre comment utiliser l’[API REST Azure Batch](/rest/api/batchservice/pool/add) pour créer un pool qui utilise des adresses IP publiques.

### <a name="rest-api-uri"></a>URI de l’API REST

```http
POST {batchURL}/pools?api-version=2020-03-01.11.0
client-request-id: 00000000-0000-0000-0000-000000000000
```

### <a name="request-body"></a>Corps de la demande

```json
"pool": {
     "id": "pool2",
     "vmSize": "standard_a1",
     "virtualMachineConfiguration": {
          "imageReference": {
               "publisher": "Canonical",
               "offer": "UbuntuServer",
               "sku": "16.040-LTS"
          },
     "nodeAgentSKUId": "batch.node.ubuntu 16.04"
     }
     "networkConfiguration": {
          "subnetId": "/subscriptions/<your_subscription_id>/resourceGroups/<your_resource_group>/providers/Microsoft.Network/virtualNetworks/<your_vnet_name>/subnets/<your_subnet_name>",
          "publicIPAddressConfiguration": {
               "provision": "NoPublicIPAddresses"
          }
     },
     "resizeTimeout": "PT15M",
     "targetDedicatedNodes": 5,
     "targetLowPriorityNodes": 0,
     "taskSlotsPerNode": 3,
     "taskSchedulingPolicy": {
          "nodeFillType": "spread"
     },
     "enableAutoScale": false,
     "enableInterNodeCommunication": true,
     "metadata": [
    {
      "name": "myproperty",
      "value": "myvalue"
    }
       ]
}
```

## <a name="outbound-access-to-the-internet"></a>Accès sortant vers Internet

Dans un pool sans adresses IP publiques, vos machines virtuelles ne pourront pas accéder à l’Internet public si vous ne configurez pas votre réseau correctement, par exemple, à l’aide de la [traduction d'adresses réseau (NAT) de réseau virtuel](../virtual-network/nat-overview.md). Notez que la NAT autorise uniquement l’accès sortant à Internet à partir des machines virtuelles du réseau virtuel. Les nœuds de calcul créés par Batch ne sont pas accessibles publiquement, car ils n’ont pas d’adresses IP publiques.

Pour fournir une connectivité sortante, vous pouvez également utiliser un routage défini par l’utilisateur (UDR). Cela vous permet de router le trafic vers un ordinateur proxy disposant d’un accès Internet public.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [création de pools dans un réseau virtuel](batch-virtual-network.md)
- En savoir plus sur l’[utilisation des points de terminaison privés avec des comptes Batch](private-connectivity.md)
