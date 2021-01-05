---
title: Configurer une préférence de routage pour une adresse IP publique – Portail Azure
description: Découvrez comment créer une adresse IP publique avec une préférence de routage du trafic Internet
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/02/2020
ms.author: mnayak
ms.openlocfilehash: ef1e33a2e43f26dcaf794b2ed81f27d39639b52d
ms.sourcegitcommit: 5b93010b69895f146b5afd637a42f17d780c165b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96533976"
---
# <a name="configure-routing-preference-for-a-public-ip-address-using-the-azure-portal"></a>Configurer une préférence de routage pour une adresse IP publique à l’aide du portail Azure

Cet article montre comment configurer une [préférence de routage](https://docs.microsoft.com/azure/virtual-network/routing-preference-overview) via un réseau de fournisseur de services Internet (option **Internet**) pour une adresse IP publique à l’aide du portail Azure. Une fois l’adresse IP publique créée, vous pouvez l’associer aux ressources Azure suivantes pour les trafics entrant et sortant via Internet :

* Machine virtuelle
* Jeu de mise à l’échelle de machine virtuelle
* Azure Kubernetes Service (AKS)
* Équilibreur de charge accessible via Internet
* Application Gateway
* Pare-feu Azure

Par défaut, la préférence de routage pour l’adresse IP publique est définie sur le réseau Microsoft mondial pour tous les services Azure, et peut être associée à n’importe lequel de ceux-ci.

> [!IMPORTANT]
> La préférence de routage est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) maintenant.

## <a name="create-a-public-ip-address-with-a-routing-preference"></a>Créer une adresse IP publique avec une préférence de routage
1. Connectez-vous au [portail Azure](https://preview.portal.azure.com/).
2. Sélectionnez **Créer une ressource**. 
3. Dans la zone de recherche, tapez *Adresse IP publique*.
3. Dans les résultats de recherche, sélectionnez **Adresse IP publique**. Ensuite, dans la page **Adresse IP publique**, sélectionnez **Créer**.
3. Dans les options de **Préférence de routage**, sélectionnez **Internet**.

      ![Créez une adresse IP publique](./media/routing-preference-portal/pip-new.png)

    > [!NOTE]
    > Les adresses IP publiques sont créées avec une adresse IPv4 et IPv6. Toutefois, actuellement, la préférence de routage ne prend en charge qu’IPV4.

Vous pouvez associer l’adresse IP publique créée ci-dessus à une machine virtuelle [Windows](../virtual-machines/windows/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Linux](../virtual-machines/linux/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Utilisez la section CLI sur la page du tutoriel : [Associez une adresse IP publique à une machine virtuelle](associate-public-ip-address-vm.md#azure-cli) pour associer l’adresse IP publique à votre machine virtuelle. Vous pouvez associer l’adresse IP publique créée ci-dessus avec un [équilibreur de charge Azure](../load-balancer/load-balancer-overview.md) en l’assignant à la configuration **frontale** de l’équilibreur de charge. L’adresse IP publique sert d’adresse IP virtuelle (VIP) à charge équilibrée.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur l’[adresse IP publique avec préférence de routage](routing-preference-overview.md).
- [Configurer la préférence de routage pour une machine virtuelle](tutorial-routing-preference-virtual-machine-portal.md).
- [Configurer une préférence de routage pour une adresse IP publique à l’aide de PowerShell](routing-preference-powershell.md).
- En savoir plus sur les [adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) dans Azure.
- En savoir plus sur tous les [paramètres d’adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address).
