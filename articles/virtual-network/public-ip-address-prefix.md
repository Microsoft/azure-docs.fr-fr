---
title: Préfixe d’adresse IP publique Azure
description: Découvrez ce qu’est un préfixe d’adresse IP publique Azure et comment il vous permet d’attribuer des adresses IP publiques prévisibles à vos ressources.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/29/2020
ms.author: allensu
ms.openlocfilehash: 53dd6d2dda762b3cbf53f4aaec6cd3692a9656e9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87432575"
---
# <a name="public-ip-address-prefix"></a>Préfixe d’adresse IP publique

Un préfixe d’IP publique est une plage réservée d’adresses IP dans Azure. Azure alloue une plage contiguë d’adresses à votre abonnement en fonction du nombre que vous spécifiez. 

Si vous n’êtes pas familiarisé avec les adresses IP publique, consultez [Adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses).

Les adresses IP publiques sont attribuées à partir d’un pool d’adresses dans chaque région Azure. Vous pouvez [télécharger](https://www.microsoft.com/download/details.aspx?id=56519) la liste des plages qu’Azure utilise pour chaque région. Par exemple, 40.121.0.0/16 est une plage parmi plus de 100 plages qu’Azure utilise dans la région USA Est. La plage inclut les adresses utilisables comprises entre 40.121.0.1 et 40.121.255.254.

Vous créez un préfixe d’adresse IP publique dans une région et un abonnement Azure en spécifiant un nom et le nombre d’adresses que le préfixe doit inclure. 

Les plages d’IP publiques sont assignées avec un préfixe de votre choix. Si vous créez un préfixe de /28, Azure fournit 16 adresses IP de l’une de ses plages.

Vous ne savez pas quelle plage Azure attribue jusqu’à ce que vous créiez la plage, mais les adresses sont contiguës. 

Les préfixes d’IP publiques sont payants. Pour plus d’informations, consultez la [tarification des IP publiques](https://azure.microsoft.com/pricing/details/ip-addresses).

## <a name="why-create-a-public-ip-address-prefix"></a>Pourquoi créer un préfixe d’adresse IP publique ?

Quand vous créez des ressources d’IP publique, Azure attribue une IP publique disponible à partir de l’une des plages utilisées dans cette région. 

Tant qu’Azure n’a pas attribué l’adresse IP, vous ne connaissez pas l’adresse IP exacte. Ce processus peut être problématique lorsque vous créez des règles de pare-feu qui autorisent des adresses IP spécifiques. Pour chaque adresse IP ajoutée, une règle de pare-feu correspondante doit être ajoutée.

Lorsque vous attribuez des adresses à vos ressources à partir d’un préfixe d’IP publique, la mise à jour des règles de pare-feu n’est pas nécessaire. La plage entière est ajoutée à la règle.

## <a name="benefits"></a>Avantages

- Création de ressources d’IP publique à partir d’une plage connue.
- Configuration des règles de pare-feu avec des plages qui incluent des IP publiques que vous avez déjà attribuées et des adresses que vous n’avez pas encore attribuées. Cette configuration évite d’avoir à changer les règles de pare-feu quand vous attribuez des adresses IP à de nouvelles ressources.
- La taille par défaut d’une plage que vous pouvez créer est /28 ou 16 adresses IP.
- Il n’existe pas de limites quant au nombre de plages que vous pouvez créer. Toutefois, il existe des limites sur le nombre maximal d’IP publiques statiques que vous pouvez avoir dans un abonnement Azure. Par conséquent, le nombre de plages que vous créez ne peut pas comprendre plus d’IP publiques statiques que vous ne pouvez en avoir dans votre abonnement. Pour plus d'informations, consultez [Limites Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Les adresses que vous créez à l’aide d’adresses provenant du préfixe peuvent être attribuées à n’importe quelle ressource Azure à laquelle vous pouvez attribuer une adresse IP publique.
- Vous pouvez facilement voir quelles adresses IP de la plage sont allouées et lesquelles ne le sont pas encore.

## <a name="scenarios"></a>Scénarios
Vous pouvez associer les ressources suivantes à une adresse IP publique statique à partir d’un préfixe :

|Ressource|Scénario|Étapes|
|---|---|---|
|Machines virtuelles| L’association d’IP publiques à partir d’un préfixe à vos machines virtuelles dans Azure réduit la surcharge de gestion lors de l’ajout d’adresses IP à une liste verte dans le pare-feu. Vous pouvez ajouter la totalité d’un préfixe avec une seule règle de pare-feu. Quand vous effectuez une mise à l’échelle avec des machines virtuelles dans Azure, vous pouvez associer des adresses IP du même préfixe, ce qui réduit les coûts, fait gagner du temps et limite la surcharge de gestion.| Pour associer des adresses IP d’un préfixe à votre machine virtuelle : </br> 1. [Créez un préfixe.](manage-public-ip-address-prefix.md) </br> 2. [Créez une adresse IP à partir du préfixe.](manage-public-ip-address-prefix.md) </br> 3. [Associez l’adresse IP à l’interface réseau de votre machine virtuelle.](virtual-network-network-interface-addresses.md#add-ip-addresses) </br> Vous pouvez également [associer les adresses IP à un groupe de machines virtuelles identiques](https://azure.microsoft.com/resources/templates/101-vmms-with-public-ip-prefix/).
| Équilibreurs de charge Standard | L’association d’IP publiques d’un préfixe à la configuration d’adresses IP frontale ou à la règle de trafic sortant d’un équilibreur de charge garantit la simplification de votre espace d’adressage IP public Azure. Simplifiez votre scénario en préparant les connexions sortantes de sorte à partir d’une plage d’adresses IP contiguës. | Pour associer des adresses IP d’un préfixe à votre équilibreur de charge : </br> 1. [Créez un préfixe.](manage-public-ip-address-prefix.md) </br> 2. [Créez une adresse IP à partir du préfixe.](manage-public-ip-address-prefix.md) </br> 3. Quand vous créez l’équilibreur de charge, sélectionnez ou mettez à jour l’adresse IP créée à l’étape 2 ci-dessus comme adresse IP frontale de votre équilibreur de charge. |
| Pare-feu Azure | Vous pouvez utiliser une adresse IP publique d’un préfixe pour une NAT de trafic sortant. Tout le trafic virtuel sortant est traduit en IP publique de [pare-feu Azure](../firewall/overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). | Pour associer une adresse IP d’un préfixe à votre pare-feu : </br> 1. [Créez un préfixe.](manage-public-ip-address-prefix.md) </br> 2. [Créez une adresse IP à partir du préfixe.](manage-public-ip-address-prefix.md) </br> 3. Quand vous [déployez le pare-feu Azure](../firewall/tutorial-firewall-deploy-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json#deploy-the-firewall), veillez à sélectionner l’adresse IP précédemment allouée à partir du préfixe.|
| Application Gateway v2 | Vous pouvez utiliser l’adresse IP publique d’un préfixe pour la mise à l’échelle automatique et la passerelle redondante interzone Application Gateway v2. | Pour associer une adresse IP d’un préfixe à votre passerelle : </br> 1. [Créez un préfixe.](manage-public-ip-address-prefix.md) </br> 2. [Créez une adresse IP à partir du préfixe.](manage-public-ip-address-prefix.md) </br> 3. Quand vous [déployez la passerelle applicative](../application-gateway/quick-create-portal.md#create-an-application-gateway), veillez à sélectionner l’adresse IP précédemment allouée à partir du préfixe.|

## <a name="constraints"></a>Contraintes

- Vous ne pouvez pas spécifier les adresses IP pour le préfixe. Azure alloue les adresses IP pour le préfixe, en fonction de la taille que vous spécifiez.
- Vous pouvez créer un préfixe pouvant inclure jusqu’à 16 adresses IP ou un /28 par défaut. Pour plus d’informations, consultez [Demandes d’augmentation de limites réseau](https://docs.microsoft.com/azure/azure-portal/supportability/networking-quota-requests) et [Limites d’Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).
- Vous ne pouvez pas changer la plage une fois que vous avez créé le préfixe.
- Seules les adresses IP publiques créées avec la référence (SKU) standard peuvent être attribuées à partir de la plage du préfixe. Pour en savoir plus sur les références (SKU) d’adresses IP publiques, consultez [Adresse IP publique](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses).
- Les adresses de la plage ne peuvent être attribuées qu’à des ressources Azure Resource Manager. Il n’est pas possible d’attribuer des adresses aux ressources du modèle de déploiement classique.
- Toutes les IP publiques créées à partir du préfixe doivent exister dans la même région et le même abonnement Azure que le préfixe. Les adresses doivent être attribuées à des ressources de la même région et du même abonnement.
- Vous ne pouvez pas supprimer un préfixe si des adresses qu’il contient sont attribuées à des ressources d’adresse P publique associées à une ressource. Dissociez d’abord toutes les ressources d’adresse IP publique qui sont des adresses IP attribuées du préfixe.


## <a name="next-steps"></a>Étapes suivantes

- [Créer](manage-public-ip-address-prefix.md) un préfixe d’adresse IP publique
