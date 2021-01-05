---
title: Réseau virtuel pour services Azure
titlesuffix: Azure Virtual Network
description: Découvrez comment déployer des services Azure dédiés sur un réseau virtuel et apprenez-en davantage sur les fonctionnalités fournies par ces déploiements.
services: virtual-network
documentationcenter: na
author: mohnader
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/06/2020
ms.author: kumud
ms.openlocfilehash: 7e53127b51116c87dd8b92ea3cd87ebf1892d38b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999686"
---
# <a name="deploy-dedicated-azure-services-into-virtual-networks"></a>Déployer des services Azure dédiés dans des réseaux virtuels

Quand vous déployez des services Azure dédiés sur un [réseau virtuel](virtual-networks-overview.md), vous pouvez communiquer avec les ressources de service de manière privée, par le biais d’adresses IP privées.

![Services déployés sur un réseau virtuel](./media/virtual-network-for-azure-services/deploy-service-into-vnet.png)

Le déploiement de services au sein d’un réseau virtuel fournit les fonctionnalités suivantes :

- Les ressources situées sur le réseau virtuel peuvent communiquer entre elles de manière privée, par le biais d’adresses IP privées. Exemple : le transfert direct des données entre HDInsight et SQL Server sur une machine virtuelle, au sein d’un réseau virtuel.
- Les ressources locales peuvent accéder aux ressources d’un réseau virtuel à l’aide d’adresses IP privées par le biais d’une connexion [VPN site à site (passerelle VPN)](../vpn-gateway/design.md?toc=%2fazure%2fvirtual-network%2ftoc.json#s2smulti) ou [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Les réseaux virtuels peuvent être [appairés](virtual-network-peering-overview.md) pour permettre aux ressources situées sur les réseaux virtuels de communiquer entre elles, par le biais d’adresses IP privées.
- Les instances de service dans un réseau virtuel sont généralement entièrement managées par le service Azure. Cela inclut la surveillance de l’intégrité des ressources et la mise à l’échelle avec une charge.
- Les instances de service sont déployées dans un sous-réseau d’un réseau virtuel. L’accès au réseau entrant ou sortant pour le sous-réseau doit être ouvert par le biais de [groupes de sécurité réseau](security-overview.md#network-security-groups), selon les recommandations fournies par le service.
- Certains services imposent également des restrictions sur le sous-réseau dans lequel ils sont déployés, en limitant l’application des stratégies, des itinéraires ou en combinant des ressources des machines virtuelles et de service au sein du même sous-réseau. Vérifiez sur chaque service les restrictions spécifiques, car elles peuvent changer au fil du temps. Azure NetApp Files, Dedicated HSM, Azure Container Instances, App Service sont des exemples de ces services. 
- Les services peuvent éventuellement nécessiter un [sous-réseau délégué](virtual-network-manage-subnet.md#add-a-subnet) comme identificateur explicit qu’un sous-réseau peut héberger un service particulier. Grâce à la délégation, les services obtiennent des autorisations explicites pour créer pour créer des ressources propres au service dans le sous-réseau délégué.
- Consultez un exemple de réponse de l’API REST sur un [réseau virtuel avec un sous-réseau délégué](https://docs.microsoft.com/rest/api/virtualnetwork/virtualnetworks/get#get-virtual-network-with-a-delegated-subnet). Une liste complète de services qui utilisent le modèle de sous-réseau délégué peut être obtenue via l’API [Available Delegations](https://docs.microsoft.com/rest/api/virtualnetwork/availabledelegations/list) (Délégations disponibles).

### <a name="services-that-can-be-deployed-into-a-virtual-network"></a>Services pouvant être déployés dans un réseau virtuel

|Category|Service| Sous-réseau<sup>1</sup> dédié
|-|-|-|
| Calcul | Machines virtuelles : [Linux](../virtual-machines/linux/infrastructure-networking-guidelines.md?toc=%2fazure%2fvirtual-network%2ftoc.json) ou [Windows](../virtual-machines/windows/infrastructure-networking-guidelines.md?toc=%2fazure%2fvirtual-network%2ftoc.json) <br/>[Groupes de machines virtuelles identiques](../virtual-machine-scale-sets/virtual-machine-scale-sets-mvss-existing-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Service cloud](https://msdn.microsoft.com/library/azure/jj156091) : Réseau virtuel (classique) uniquement<br/> [Azure Batch](../batch/nodes-and-pools.md?toc=%2fazure%2fvirtual-network%2ftoc.json#virtual-network-vnet-and-firewall-configuration)| Non <br/> Non <br/> Non <br/> Non <sup>2</sup>
| Réseau | [Application Gateway - WAF](../application-gateway/application-gateway-ilb-arm.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Pare-feu Azure](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)  <br/> [Azure Bastion](../bastion/bastion-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Appliances virtuelles réseau](/windows-server/networking/sdn/manage/use-network-virtual-appliances-on-a-vn)| Oui <br/> Oui <br/> Oui <br/> Oui <br/> Non
|Données|[RedisCache](../azure-cache-for-redis/cache-how-to-premium-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure SQL Managed Instance](../azure-sql/managed-instance/connectivity-architecture-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)| Oui <br/> Oui <br/> 
|Analytics | [Azure HDInsight](../hdinsight/hdinsight-extend-hadoop-virtual-network.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure Databricks](../azure-databricks/what-is-azure-databricks.md?toc=%2fazure%2fvirtual-network%2ftoc.json) |Non <sup>2</sup> <br/> Non <sup>2</sup> <br/> 
| Identité | [Services de domaine Azure Active Directory](../active-directory-domain-services/active-directory-ds-getting-started-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json) |Non <br/>
| Containers | [Azure Kubernetes Service (AKS)](../aks/concepts-network.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure Container Instances (ACI)](https://www.aka.ms/acivnet)<br/>[Moteur Azure Container Service](https://github.com/Azure/acs-engine) avec le [plug-in](https://github.com/Azure/acs-engine/tree/master/examples/vnet) CNI Réseau virtuel Azure<br/>[Azure Functions](../azure-functions/functions-networking-options.md#virtual-network-integration) |Non <sup>2</sup><br/> Oui <br/><br/> Non <br/> Oui
| Web | [Gestion des API](../api-management/api-management-using-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Web Apps](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Environnement App Service](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure Logic Apps](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>|Oui <br/> Oui <br/> Oui <br/> Oui
| Hébergée | [Module de sécurité matériel (HSM) dédié Azure](../dedicated-hsm/index.yml?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>[Azure NetApp Files](../azure-netapp-files/azure-netapp-files-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json)<br/>|Oui <br/> Oui <br/>
| Azure Spring Cloud | [Déployer dans un réseau virtuel Azure (injection de réseau virtuel)](../spring-cloud/spring-cloud-tutorial-deploy-in-azure-virtual-network.md)<br/>| Oui <br/>
| | |

<sup>1</sup> « Dédié » implique que seules des ressources propres au service peuvent être déployées dans ce sous-réseau et ne peuvent pas être combinées avec des machines virtuelles/groupes de machines virtuelles identiques (VMSS) des clients <br/> 
<sup>2</sup> Il est recommandé, au titre de bonne pratique, d’avoir ces services dans un sous-réseau dédié, mais il ne s’agit pas d’une obligation imposée par le service.
