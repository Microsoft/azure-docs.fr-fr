---
title: Créer un pool avec des adresses IP publiques spécifiées
description: Découvrez comment créer un pool Batch qui utilise vos propres adresses IP publiques.
ms.topic: how-to
ms.date: 10/08/2020
ms.openlocfilehash: e822311718847e173763847d503335f71457308b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91849326"
---
# <a name="create-an-azure-batch-pool-with-specified-public-ip-addresses"></a>Créer un pool Azure Batch avec des adresses IP publiques spécifiées

Quand vous créez un pool Azure Batch, vous pouvez configurer le pool dans un sous-réseau d’un [réseau virtuel Azure](batch-virtual-network.md) (VNet) que vous spécifiez. Les machines virtuelles du pool Batch sont accessibles par le biais d’adresses IP publiques créées par Batch. Ces adresses IP publiques peuvent changer au cours de la durée de vie du pool, ce qui signifie que vos paramètres réseau peuvent devenir obsolètes si les adresses IP ne sont pas actualisées.

Vous pouvez créer une liste d’adresses IP publiques statiques à utiliser avec les machines virtuelles de votre pool. Cela vous permet de contrôler la liste des adresses IP publiques et de vous assurer qu’elles ne changeront pas de manière inattendue. Cela peut s’avérer particulièrement utile si vous travaillez avec un service externe, tel qu’une base de données, qui restreint l’accès à certaines adresses IP.

Pour plus d’informations sur la création de pools sans adresses IP publiques, consultez [Créer un pool Azure Batch sans adresses IP publiques](./batch-pool-no-public-ip-address.md).

## <a name="prerequisites"></a>Prérequis

- **Authentification**. Pour utiliser une adresse IP publique, l’API du client Batch doit utiliser l’[authentification Azure Active Directory (AD)](batch-aad-auth.md).

- **Un réseau virtuel Azure**. Vous devez utiliser un [réseau virtuel](batch-virtual-network.md) du même abonnement Azure que celui dans lequel vous créez votre pool et vos adresses IP. Seuls les réseaux virtuels basés sur Azure Resource Manager peuvent être utilisés. Vérifiez que le réseau virtuel satisfait l’ensemble des [conditions générales](batch-virtual-network.md#vnet-requirements).

- **Au moins une adresse IP publique Azure**. Pour créer une ou plusieurs adresses IP publiques, vous pouvez utiliser le [portail Azure](../virtual-network/virtual-network-public-ip-address.md#create-a-public-ip-address), l’[interface de ligne de commande Azure](/cli/azure/network/public-ip#az-network-public-ip-create) ou [Azure PowerShell](/powershell/module/az.network/new-azpublicipaddress). Veillez à respecter la configuration requise indiquée ci-dessous.

> [!NOTE]
> Batch alloue automatiquement des ressources réseau supplémentaires dans le groupe de ressources contenant les adresses IP publiques. Pour chaque groupe de 100 nœuds dédiés, Batch alloue généralement un groupe de sécurité réseau (NSG) et un équilibreur de charge. Ces ressources sont limitées par les quotas de ressources de l’abonnement. Lors de l’utilisation de grands pools, vous devrez peut-être [demander une augmentation du quota](batch-quota-limit.md#increase-a-quota) pour une ou plusieurs de ces ressources.

## <a name="public-ip-address-requirements"></a>Exigences relatives aux adresses IP publiques

Quand vous créez vos adresses IP publiques, gardez à l’esprit les exigences suivantes :

- Les adresses IP publiques doivent se trouver dans la même région et le même abonnement que le compte Batch utilisé pour créer le pool.
- L’**attribution d’adresses IP** doit être définie sur **Statique**.
- **SKU** doit être défini sur **Standard**.
- Un nom DNS doit être spécifié.
- Les adresses IP publiques doivent être utilisées uniquement pour les pools de configuration de machines virtuelles. Aucune autre ressource ne doit utiliser ces adresses IP, sinon le pool peut rencontrer des échecs d’allocation.
- Aucune stratégie de sécurité ni aucun verrou de ressource ne doit limiter l’accès d’un utilisateur à l’adresse IP publique.
- Le nombre d’adresses IP publiques spécifiées pour le pool doit être suffisamment élevée pour satisfaire le nombre de machines virtuelles ciblées pour le pool. Ce nombre doit être au moins égal à la somme des propriétés **targetDedicatedNodes** et **targetLowPriorityNodes** du pool. S’il n’y a pas assez d’adresses IP, le pool alloue partiellement les nœuds de calcul et une erreur de redimensionnement se produit. Batch utilise une adresse IP publique pour chaque groupe de 100 machines virtuelles.
- Veillez à toujours disposer d’une mémoire tampon supplémentaire d’adresses IP publiques. Nous vous recommandons d’ajouter au moins une adresse IP publique supplémentaire, ou environ 10 % du nombre total d’adresses IP publiques que vous ajoutez à un pool, en fonction de la valeur la plus élevée. Cette mémoire tampon supplémentaire aidera Batch lors de son optimisation interne en cas de scale-down, et autorisera un scale-up plus rapide après un échec de scale-down ou de scale-up.
- Une fois le pool créé, vous ne pouvez pas ajouter ni changer la liste des adresses IP publiques utilisées par le pool. Si vous avez besoin de changer la liste, vous devez supprimer le pool, puis le recréer.

## <a name="create-a-batch-pool-with-public-ip-addresses"></a>Créer un pool Batch avec des adresses IP publiques

L’exemple ci-dessous montre comment utiliser l’[API REST du service Azure Batch](/rest/api/batchservice/pool/add) pour créer un pool qui utilise des adresses IP publiques.

### <a name="batch-service-rest-api"></a>API REST du service Batch

URI de l’API REST

```http
POST {batchURL}/pools?api-version=2020-03-01.11.0
client-request-id: 00000000-0000-0000-0000-000000000000
```

Corps de la requête

```json
"pool": {
      "id": "pool2",
      "vmSize": "standard_a1",
      "virtualMachineConfiguration": {
        "imageReference": {
          "publisher": "Canonical",
          "offer": "UbuntuServer",
          "sku": "16.04.0-LTS"
        },
        "nodeAgentSKUId": "batch.node.ubuntu 16.04"
      },
"networkConfiguration": {
          "subnetId": "/subscriptions/<subId>/resourceGroups/<rgId>/providers/Microsoft.Network/virtualNetworks/<vNetId>/subnets/<subnetId>",
          "publicIPAddressConfiguration": {
            "provision": "usermanaged",
            "ipAddressIds": [
              "/subscriptions/<subId>/resourceGroups/<rgId>/providers/Microsoft.Network/publicIPAddresses/<publicIpId>"
          ]
        },

       "resizeTimeout":"PT15M",
      "targetDedicatedNodes":5,
      "targetLowPriorityNodes":0,
      "taskSlotsPerNode":3,
      "taskSchedulingPolicy": {
        "nodeFillType":"spread"
      },
      "enableAutoScale":false,
      "enableInterNodeCommunication":true,
      "metadata": [ {
        "name":"myproperty",
        "value":"myvalue"
      } ]
    }
```

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur le [workflow et les ressources principales du service Batch](batch-service-workflow-features.md), telles que les pools, les nœuds, les travaux et les tâches.
- Apprenez-en davantage sur la [création d’un pool sur un sous-réseau d’un réseau virtuel Azure](batch-virtual-network.md).
- Découvrez comment [créer un pool Azure Batch sans adresses IP publiques](./batch-pool-no-public-ip-address.md).

