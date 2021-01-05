---
title: Azure Standard Load Balancer et Virtual Machine Scale Sets
titleSuffix: Azure Standard Load Balancer and Virtual Machine Scale Sets
description: Dans ce parcours d’apprentissage, familiarisez-vous avec Azure Standard Load Balancer et Virtual Machine Scale Sets.
services: load-balancer
documentationcenter: na
author: irenehua
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/17/2020
ms.author: irenehua
ms.openlocfilehash: fdca40d5113f06d185255be2e237cb52b47e9793
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94697439"
---
# <a name="azure-load-balancer-with-azure-virtual-machine-scale-sets"></a>Azure Load Balancer avec des groupes de machines virtuelles identiques

Lorsque vous utilisez des groupes de machines virtuelles identiques et un équilibreur de charge, tenez compte des instructions suivantes :

## <a name="port-forwarding-and-inbound-nat-rules"></a>Réacheminement de port et règles NAT de trafic entrant
  * Une fois le groupe identique créé, le port principal ne peut pas être modifié lorsqu'une règle d'équilibrage de charge est utilisée par une sonde d'intégrité pour l'équilibreur de charge. Pour modifier le port, vous pouvez supprimer la sonde d'intégrité en mettant à jour le groupe identique de machines virtuelles Azure, puis mettre à jour le port et reconfigurer la sonde d'intégrité.
  * Lorsque vous utilisez le groupe de machines virtuelles identiques dans le pool principal de l’équilibreur de charge, les règles NAT de trafic entrant par défaut sont automatiquement créées.
## <a name="inbound-nat-pool"></a>Pool NAT de trafic entrant
  * Chaque groupe de machines virtuelles identiques doit comporter au moins un pool NAT de trafic entrant. 
  * Le pool NAT de trafic entrant est un ensemble de règles NAT de trafic entrant. Un même pool NAT de trafic entrant ne peut pas prendre en charge plusieurs groupes de machines virtuelles identiques.
  * Pour supprimer un pool NAT d’un groupe de machines virtuelles identiques existant, vous devez d’abord supprimer le pool NAT du groupe identique. Vous trouverez ci-dessous un exemple complet utilisant l’interface CLI :
```azurecli-interactive
  az vmss update
     --resource-group MyResourceGroup
     --name MyVMSS
     --remove virtualMachineProfile.networkProfile.networkInterfaceConfigurations[0].ipConfigurations[0].loadBalancerInboundNatPools
  az vmss update-instances
     -–instance-ids *
     --resource-group MyResourceGroup
     --name MyVMSS
  az network lb inbound-nat-pool delete
     --resource-group MyResourceGroup
     -–lb-name MyLoadBalancer
     --name MyNatPool
```
## <a name="load-balancing-rules"></a>Règles d’équilibrage de charge :
  * Lorsque vous utilisez le groupe de machines virtuelles identiques dans le pool principal de l’équilibreur de charge, la règle d’équilibrage de charge par défaut est automatiquement créée.
## <a name="outbound-rules"></a>Règles de trafic sortant
  *  Pour créer une règle de trafic sortant pour un pool principal à laquelle une règle d’équilibrage de charge fait déjà référence, vous devez d’abord définir **« Créer des règles de trafic sortant implicites »** sur **Non** sur le portail lors de la création de la règle d’équilibrage de charge du trafic entrant.

  :::image type="content" source="./media/vm-scale-sets/load-balancer-and-vm-scale-sets.png" alt-text="Création d'une règle d’équilibrage de charge" border="true":::

Les méthodes suivantes peuvent être utilisées pour déployer un groupe identique de machines virtuelles avec un équilibreur de charge Azure existant.

* [Configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant à l’aide du portail Azure](./configure-vm-scale-set-portal.md).
* [Configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant à l’aide d’Azure PowerShell](./configure-vm-scale-set-powershell.md).
* [Configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant à l’aide d’Azure CLI](./configure-vm-scale-set-cli.md).