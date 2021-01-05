---
title: Forum aux questions (FAQ) - Azure Load Balancer
description: Réponses aux questions fréquemment posées sur Azure Load Balancer.
services: load-balancer
author: erichrt
ms.service: load-balancer
ms.topic: article
ms.date: 04/22/2020
ms.author: errobin
ms.openlocfilehash: 2e559d574413b8eb0be2303798e0b16bfffad2cb
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695399"
---
# <a name="load-balancer-frequently-asked-questions"></a>Questions fréquentes sur Load Balancer

## <a name="what-types-of-load-balancer-exist"></a>Quels types d’instances Load Balancer existent ?
Les équilibreurs de charge internes qui équilibrent le trafic au sein d’un réseau virtuel et les équilibreurs de charge externes qui équilibrent le trafic vers et à partir d’un point de terminaison connecté à Internet. Pour plus d’informations, consultez [Types de Load balancer](components.md#frontend-ip-configurations). 

Pour ces deux types, Azure propose une référence SKU de base et une référence SKU standard qui ont des fonctionnalités de suivi d’intégrité, de performances et de sécurité différentes. Ces différences sont expliquées dans notre article [Comparaison des références SKU](skus.md).

 ## <a name="how-can-i-upgrade-from-a-basic-to-a-standard-load-balancer"></a>Comment puis-je effectuer une mise à niveau de Load Balancer Basic vers Standard Load Balancer ?
Pour obtenir un script automatisé et des conseils sur la mise à niveau d’une référence Load Balancer, consultez l’article [Mise à niveau de l’instance de base vers l’instance standard](upgrade-basic-standard.md).

 ## <a name="what-are-the-different-load-balancing-options-in-azure"></a>Quelles sont les différentes options d’équilibrage de charge dans Azure ?
Consultez le [guide de la technologie de l’équilibreur de charge](/azure/architecture/guide/technology-choices/load-balancing-overview) pour connaître les services d’équilibrage de charge disponibles et les utilisations recommandées pour chacun d’entre eux.

## <a name="where-can-i-find-load-balancer-arm-templates"></a>Où puis-je trouver les modèles ARM de Load Balancer ?
Consultez la [liste des modèles de démarrage rapide Azure Load Balancer](/azure/templates/microsoft.network/loadbalancers#quickstart-templates) pour les modèles ARM de déploiements courants.

## <a name="how-are-inbound-nat-rules-different-from-load-balancing-rules"></a>Quelle est la différence entre les règles NAT entrantes et les règles d’équilibrage de charge ?
Les règles NAT sont utilisées pour spécifier une ressource backend vers laquelle acheminer le trafic. Par exemple, la configuration d’un port d’équilibrage de charge spécifique pour envoyer le trafic RDP vers une machine virtuelle spécifique. Les règles d’équilibrage de charge sont utilisées pour spécifier un pool de ressources backend vers lequel acheminer le trafic, en équilibrant la charge sur chaque instance. Par exemple, une règle d’équilibreur de charge peut acheminer les paquets TCP sur le port 80 de l’équilibreur de charge sur un pool de serveurs Web.

## <a name="what-is-ip-1686312916"></a>Qu’est-ce que l’adresse IP 168.63.129.16 ?
Adresse IP virtuelle de l’hôte marquée en tant qu’infrastructure Azure Load Balancer d’où proviennent les sondes d’intégrité Azure. Lors de la configuration d’instances backend, elles doivent autoriser le trafic à partir de cette adresse IP pour répondre correctement aux sondes d’intégrité. Cette règle n’interagit pas avec l’accès à votre frontend Load Balancer. Vous pouvez remplacer cette règle si vous n’utilisez pas Azure Load Balancer. Apprenez-en davantage sur les balises de service [ici](../virtual-network/service-tags-overview.md#available-service-tags).

## <a name="can-i-use-global-vnet-peering-with-basic-load-balancer"></a>Puis-je utiliser le peering de Global VNET avec la version basique de Load Balancer ?
Non. La version basique de Load Balancer ne prend pas en charge le peering de Global VNET. Vous pouvez utiliser la version standard de Load Balancer à la place. Pour une mise à niveau transparente, consultez l’article [Mise à niveau de la version basique à la version standard](upgrade-basic-standard.md).

## <a name="how-can-i-discover-the-public-ip-that-an-azure-vm-uses"></a>Comment puis-je trouver l’adresse IP publique utilisée par une machine virtuelle Azure ?

Il existe de nombreuses manières de déterminer l’adresse IP source publique d’une connexion sortante. OpenDNS fournit un service qui peut vous indiquer l’adresse IP publique votre machine virtuelle.
La commande nslookup vous permet d’envoyer une requête DNS sur le nom myip.opendns.com au programme de résolution OpenDNS. Le service retourne l’adresse IP source qui a été utilisée pour envoyer la requête. Quand vous exécutez la requête suivante à partir de votre machine virtuelle, la réponse est l’adresse IP publique utilisée pour cette machine virtuelle :

 ```nslookup myip.opendns.com resolver1.opendns.com```

## <a name="how-do-connections-to-azure-storage-in-the-same-region-work"></a>Comment fonctionnent les connexions au Stockage Azure dans la même région ?
Il n’est pas nécessaire de disposer d’une connectivité sortante via les scénarios ci-dessus pour vous connecter au Stockage Azure dans la même région que la machine virtuelle. Si vous n’en souhaitez pas, utilisez les groupes de sécurité réseau (NSG), comme expliqué ci-dessus. Pour la connectivité vers le Stockage Azure dans d’autres régions, une connectivité sortante est requise. Lorsque vous vous connectez au Stockage Azure à partir d’une machine virtuelle dans la même région, l’adresse IP source dans les journaux de diagnostic de stockage est une adresse de fournisseur interne, et non l’adresse IP publique de votre machine virtuelle. Si vous souhaitez restreindre l’accès à votre compte de stockage aux machines virtuelles dans un ou plusieurs sous-réseaux du réseau virtuel dans la même région, utilisez des [points de terminaison de service du réseau virtuel](../virtual-network/virtual-network-service-endpoints-overview.md) et non votre adresse IP publique lors de la configuration de votre pare-feu de compte de stockage. Une fois les points de terminaison de service configurés, l’adresse IP privée de votre réseau virtuel apparaît dans vos journaux de diagnostic de stockage, mais pas l’adresse interne du fournisseur.

## <a name="what-are-best-practises-with-respect-to-outbound-connectivity"></a>Quelles sont les meilleures pratiques en matière de connectivité sortante ?
Standard Load Balancer et les adresses IP publiques standard introduisent des fonctionnalités et des comportements différents pour la connectivité sortante. Ils ne sont pas identiques aux références SKU De base. Si vous souhaitez une connectivité sortante lorsque vous travaillez avec des références SKU Standard, vous devez explicitement la définir avec des adresses IP publiques Standard ou l’équilibreur de charge public Standard. Cela inclut la création de la connectivité sortante lors de l’utilisation d’un équilibreur de charge Standard interne. Nous vous recommandons de toujours utiliser des règles de trafic sortant sur un équilibreur de charge public Standard. Cela signifie que lorsqu’un équilibreur de charge interne Standard est utilisé, vous devez prendre des mesures pour créer une connectivité sortante pour les machines virtuelles dans le pool principal si la connectivité sortante est souhaitée. Dans le contexte de la connectivité sortante, une seule machine virtuelle autonome, toutes les machines virtuelles d’un groupe à haute disponibilité et toutes les instances d’un groupe VMSS se comportent comme un groupe. Cela signifie que, si une seule machine virtuelle d’un groupe à haute disponibilité est associée à une référence SKU Standard, toutes les instances de machine virtuelle au sein de ce groupe à haute disponibilité se comportent maintenant selon les mêmes règles que si elles étaient associées avec la référence SKU Standard, même si une instance individuelle n’est pas directement associée. Ce comportement est également observé dans le cas d’une machine virtuelle autonome avec plusieurs cartes d’interface réseau attachées à un équilibreur de charge. Si une carte réseau est ajoutée comme composant autonome, elle aura le même comportement. Lisez attentivement la totalité de ce document pour comprendre les concepts généraux, passer en revue [Standard Load Balancer](./load-balancer-overview.md) pour connaître les différentes entre les références SKU et les [règles de trafic sortant](load-balancer-outbound-connections.md#outboundrules).
L’utilisation de règles de trafic sortant vous permet un contrôle précis de tous les aspects de la connectivité sortante.
 
## <a name="next-steps"></a>Étapes suivantes
Si votre question ne figure pas dans la liste ci-dessus, veuillez envoyer vos commentaires sur cette page avec votre question. Cela créera un problème GitHub pour l’équipe produit afin de s’assurer que toutes les questions des clients reçoivent une réponse.