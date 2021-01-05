---
title: Vue d’ensemble du TAP de réseau virtuel Azure | Microsoft Docs
description: Découvrez plus en détail le TAP de réseau virtuel. Le TAP de réseau virtuel vous donne une copie complète du trafic réseau de machine virtuelle pouvant être diffusé en continu vers un collecteur de paquets.
services: virtual-network
documentationcenter: na
author: karthikananth
manager: ganesr
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/14/2019
ms.author: kaanan
ms.openlocfilehash: c468f289c9dbfe949890c5462dd7c9be3e548d5a
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938758"
---
# <a name="virtual-network-tap"></a>TAP de réseau virtuel
> [!IMPORTANT]
> La préversion du TAP de réseau virtuel est actuellement en attente dans toutes les régions Azure. Vous pouvez nous envoyer un e-mail à l’adresse <azurevnettap@microsoft.com> avec votre ID d’abonnement et nous vous informerons des futures mises à jour concernant la préversion. En attendant, vous pouvez utiliser des solutions NVA ou basées sur agent qui fournissent des fonctionnalités de visibilité du TAP/réseau via nos [solutions de partenaire Packet Broker](#virtual-network-tap-partner-solutions) disponibles dans les [offres de la Place de marché Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/category/networking?page=1&subcategories=appliances%3Ball&search=Network%20Traffic&filters=partners).

Le TAP (point d’accès terminal) de réseau virtuel Azure vous permet de diffuser en continu votre trafic réseau de machine virtuelle vers un collecteur de paquets réseau ou un outil analytique. Le collecteur ou l’outil analytique est fourni par une [appliance virtuelle réseau](https://azure.microsoft.com/solutions/network-appliances/) partenaire. Pour obtenir la liste des solutions de partenaires qui fonctionnent avec un TAP de réseau virtuel, consultez les [solutions de partenaires](#virtual-network-tap-partner-solutions).
L’image suivante illustre le fonctionnement d’un TAP de réseau virtuel. Vous pouvez ajouter une configuration TAP sur une [interface réseau](virtual-network-network-interface.md) attachée à une machine virtuelle déployée dans votre réseau virtuel. La destination est une adresse IP de réseau virtuel dans le même réseau virtuel que l’interface réseau supervisée ou un réseau [virtuel appairé](virtual-network-peering-overview.md). La solution du collecteur pour le TAP de réseau virtuel peut être déployée derrière un équilibreur de charge interne Azure pour une haute disponibilité.
![Fonctionnement d’un TAP de réseau virtuel](./media/virtual-network-tap/architecture.png)

## <a name="prerequisites"></a>Conditions préalables requises

Avant de créer un TAP de réseau virtuel, vous devez avoir reçu un e-mail de confirmation indiquant que vous êtes inscrit à la préversion, et disposer d'une ou plusieurs machines virtuelles créées à l’aide d’un modèle de déploiement [Azure Resource Manager](../azure-resource-manager/management/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) et d'une solution partenaire pour agréger le trafic TAP dans la même région Azure. Si vous n’avez pas de solution de partenaire dans votre réseau virtuel, consultez les [solutions de partenaires](#virtual-network-tap-partner-solutions) pour en déployer une. Vous pouvez utiliser la même ressource TAP de réseau virtuel pour agréger le trafic de plusieurs interfaces réseau dans le même abonnement ou des abonnements différents. Si les interfaces réseau supervisées sont dans des abonnements différents, les abonnements doivent être associés au même locataire Azure Active Directory. Par ailleurs, les interfaces réseau supervisées et le point de terminaison de destination pour l’agrégation du trafic TAP peuvent être dans des réseaux virtuels appairés dans la même région. Si vous utilisez ce modèle de déploiement, vérifiez que [le peering de réseaux virtuels](virtual-network-peering-overview.md) est activé avant de configurer le TAP de réseau virtuel.

## <a name="permissions"></a>Autorisations

Les comptes que vous utilisez pour appliquer une configuration TAP sur des interfaces réseau doivent être attribués au rôle [contributeur de réseaux](../role-based-access-control/built-in-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-contributor) ou à un [rôle personnalisé](../role-based-access-control/custom-roles.md?toc=%2fazure%2fvirtual-network%2ftoc.json) avec les autorisations appropriées listées dans le tableau suivant :

| Action | Nom |
|---|---|
| Microsoft.Network/virtualNetworkTaps/* | Nécessaire pour créer, mettre à jour, lire et supprimer une ressource TAP de réseau virtuel |
| Microsoft.Network/networkInterfaces/read | Nécessaire pour lire la ressource d’interface réseau sur laquelle configurer le TAP |
| Microsoft.Network/tapConfigurations/* | Nécessaire pour créer, mettre à jour, lire et supprimer la configuration TAP sur une interface réseau |


## <a name="virtual-network-tap-partner-solutions"></a>Solutions de partenaires pour le TAP de réseau virtuel

### <a name="network-packet-brokers"></a>Répartiteurs de paquets réseau

- [Gigamon GigaSECURE](https://blog.gigamon.com/2018/09/13/why-microsofts-new-vtap-service-works-even-better-with-gigasecure-for-azure)
- [Ixia CloudLens](https://www.ixiacom.com/cloudlens/cloudlens-azure)
- [Nubeva Prisms](https://www.nubeva.com/azurevtap)
- [Big Switch Big Monitoring Fabric](https://www.bigswitch.com/products/big-monitoring-fabric/public-cloud/microsoft-azure)

### <a name="security-analytics-networkapplication-performance-management"></a>Analytique de la sécurité, gestion des performances des applications/du réseau

- [Awake Security](https://awakesecurity.com/technology-partners/microsoft-azure/)
- [Cisco Stealthwatch Cloud](https://blogs.cisco.com/security/cisco-stealthwatch-cloud-and-microsoft-azure-reliable-cloud-infrastructure-meets-comprehensive-cloud-security)
- [Darktrace](https://www.darktrace.com/en/azure/)
- [ExtraHop Reveal(x)](https://www.extrahop.com/partners/tech-partners/microsoft/)
- [Fidelis Cybersecurity](https://www.fidelissecurity.com/technology-partners/microsoft-azure )
- [Flowmon](https://www.flowmon.com/blog/azure-vtap)
- [NetFort LANGuardian](https://www.netfort.com/languardian/solutions/visibility-in-azure-network-tap/)
- [NetScout vSTREAM]( https://www.netscout.com/marketplace-azure)
- [Riverbed SteelCentral AppResponse]( https://www.riverbed.com/products/steelcentral/steelcentral-appresponse-11.html)
- [RSA NetWitness® Platform](https://www.rsa.com/content/dam/en/solution-brief/rsa-netwitness-platform-overview-for-federal-agencies.pdf)
- [Vectra Cognito](https://vectra.ai/microsoftazure)



## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer un TAP de réseau virtuel](tutorial-tap-virtual-network-cli.md).
