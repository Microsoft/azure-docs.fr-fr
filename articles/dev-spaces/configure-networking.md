---
title: Configurer la mise en réseau pour Azure Dev Spaces dans différentes topologies de réseau
services: azure-dev-spaces
ms.date: 03/17/2020
ms.topic: conceptual
description: Décrit la configuration réseau requise pour l’exécution d’Azure Dev Spaces dans Azure Kubernetes Service
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs, CNI, kubenet, SDN, réseau
ms.openlocfilehash: 09114ab13555cbf9ef42b37c86ffb76a8fe3ab3f
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91970336"
---
# <a name="configure-networking-for-azure-dev-spaces-in-different-network-topologies"></a>Configurer la mise en réseau pour Azure Dev Spaces dans différentes topologies de réseau

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces s’exécute sur les clusters Azure Kubernetes Service (AKS) avec la configuration de mise en réseau par défaut. Si vous souhaitez modifier la configuration de mise en réseau de votre cluster AKS, par exemple, placer le cluster derrière un pare-feu, utiliser des groupes de sécurité réseau ou utiliser des stratégies réseau, vous devez incorporer des considérations supplémentaires pour l’exécution d’Azure Dev Spaces.

![Configuration du réseau virtuel](media/configure-networking/virtual-network-clusters.svg)

## <a name="virtual-network-or-subnet-configurations"></a>Configurations de réseau virtuel ou de sous-réseau

Votre cluster AKS peut avoir une configuration de réseau virtuel ou de sous-réseau différente pour limiter le trafic d’entrée ou de sortie pour votre cluster AKS. Par exemple, votre cluster peut se trouver derrière un pare-feu, tel qu’un Pare-feu Azure, ou vous pouvez utiliser des groupes de sécurité réseau ou des rôles personnalisés pour restreindre le trafic réseau. Vous trouverez un exemple de configuration réseau dans l’[exemple de dépôt Azure Dev Spaces sur GitHub][sample-repo].

Azure Dev Spaces présente certaines exigences en matière de trafic réseau *d’entrée et de sortie*, ainsi que de trafic *d’entrée uniquement*. Si vous utilisez Azure Dev Spaces sur un cluster AKS avec une configuration de réseau virtuel ou de sous-réseau qui limite le trafic pour votre cluster AKS, vous devez suivre les exigences de trafic d’entrée uniquement et de trafic d’entrée et de sortie suivantes pour qu’Azure Dev Spaces fonctionne correctement.

### <a name="ingress-and-egress-network-traffic-requirements"></a>Exigences en matière de trafic réseau d’entrée et de sortie

Azure Dev Spaces a besoin d’un trafic d’entrée et de sortie pour les noms de domaine complets suivants :

| FQDN                       | Port       | Utilisation      |
|----------------------------|------------|----------|
| cloudflare.docker.com      | HTTPS : 443 | Pour tirer (pull) des images Docker pour Azure Dev Spaces |
| gcr.io                     | HTTPS : 443 | Pour tirer (pull) des images Helm pour Azure Dev Spaces |
| storage.googleapis.com     | HTTPS : 443 | Pour tirer (pull) des images Helm pour Azure Dev Spaces |

Mettez à jour la configuration de votre pare-feu ou de votre sécurité pour autoriser le trafic réseau à destination et en provenance de tous les noms de domaine complets ci-dessus et des [services d’infrastructure Azure Dev Spaces][service-tags]. Par exemple, si vous utilisez un pare-feu pour sécuriser votre réseau, les noms de domaine complets ci-dessus doivent être ajoutés à la règle d’application du pare-feu. L’étiquette du service Azure Dev Spaces doit également être [ajoutée au pare-feu][firewall-service-tags]. Ces deux mises à jour du pare-feu sont nécessaires pour autoriser le trafic vers et à partir de ces domaines.

### <a name="ingress-only-network-traffic-requirements"></a>Exigences en matière de trafic réseau d’entrée uniquement

Azure Dev Spaces fournit un routage au niveau de l’espace de noms Kubernetes ainsi qu’un accès public aux services à l’aide de son propre nom de domaine complet. Pour que ces deux fonctionnalités fonctionnent, mettez à jour votre configuration de pare-feu ou réseau pour autoriser l’entrée publique à l’adresse IP externe du contrôleur d’entrée Azure Dev Spaces sur votre cluster. Vous pouvez également créer un [équilibreur de charge interne][aks-internal-lb] et ajouter une règle NAT dans votre pare-feu pour convertir l’adresse IP publique de votre pare-feu en adresse IP de votre équilibreur de charge interne. Vous pouvez également utiliser [traefik][traefik-ingress] ou [NGINX][nginx-ingress] pour créer un contrôleur d’entrée personnalisé.

## <a name="aks-cluster-network-requirements"></a>Configuration réseau requise pour le cluster AKS

AKS vous permet d’utiliser des [stratégies réseau][aks-network-policies] pour contrôler le trafic d’entrée et de sortie entre les pods sur un cluster, ainsi que le trafic de sortie depuis un pod. Azure Dev Spaces présente certaines exigences en matière de trafic réseau *d’entrée et de sortie*, ainsi que de trafic *d’entrée uniquement*. Si vous utilisez Azure Dev Spaces sur un cluster AKS avec des stratégies réseau AKS, vous devez suivre les exigences de trafic d’entrée uniquement et de trafic d’entrée et de sortie suivantes pour qu’Azure Dev Spaces fonctionne correctement.

### <a name="ingress-and-egress-network-traffic-requirements"></a>Exigences en matière de trafic réseau d’entrée et de sortie

Azure Dev Spaces vous permet de communiquer directement avec un pod dans un espace de développement sur votre cluster à des fins de débogage. Pour que cette fonctionnalité fonctionne, ajoutez une stratégie réseau qui autorise la communication d’entrée et de sortie vers les adresses IP de l’infrastructure Azure Dev Spaces, qui [varient selon la région][service-tags].

### <a name="ingress-only-network-traffic-requirements"></a>Exigences en matière de trafic réseau d’entrée uniquement

Azure Dev Spaces assure le routage entre les pods dans les espaces de noms. Par exemple, les espaces de noms où Azure Dev Spaces est activé peuvent avoir une relation parent/enfant, ce qui permet de router le trafic réseau entre les pods dans les espaces de noms parent et enfant. Azure Dev Spaces expose également les points de terminaison de service à l’aide de son propre nom de domaine complet. Pour configurer différentes façons d’exposer des services et leur impact sur le routage au niveau de l’espace de noms, consultez [Utilisation de différentes options de point de terminaison][endpoint-options].

## <a name="using-azure-cni"></a>Utilisation d’Azure CNI

Par défaut, les clusters AKS sont configurés pour utiliser [kubenet][aks-kubenet] pour la mise en réseau, lequel fonctionne avec Azure Dev Spaces. Vous pouvez également configurer votre cluster AKS pour qu’il utilise [Azure Container Networking Interface (CNI)][aks-cni]. Pour utiliser Azure Dev Spaces avec Azure CNI sur votre cluster AKS, autorisez vos espaces d’adressage de réseau virtuel et de sous-réseau jusqu’à 10 adresses IP privées pour les pods déployés par Azure Dev Spaces. Plus d’informations sur les adresses IP privées sont disponibles dans la [documentation AKS Azure CNI][aks-cni-ip-planning].

## <a name="using-api-server-authorized-ip-ranges"></a>Utilisation des plages d’adresses IP autorisées du serveur d’API

Les clusters AKS vous permettent de configurer une sécurité supplémentaire qui limite l’adresse IP pouvant interagir avec vos clusters, par exemple, en utilisant des réseaux virtuels personnalisés ou [en sécurisant l’accès au serveur d’API à l’aide de plages d’adresses IP autorisées][aks-ip-auth-ranges]. Pour utiliser Azure Dev Spaces dans le cadre de l’utilisation de cette sécurité supplémentaire lors de la [création][aks-ip-auth-range-create] de votre cluster, vous devez [autoriser des plages supplémentaires en fonction de votre région][service-tags]. Vous pouvez également [mettre à jour][aks-ip-auth-range-update] un cluster existant pour autoriser ces plages supplémentaires. Vous devez également autoriser l’adresse IP de toutes les machines de développement qui se connectent à votre cluster AKS à des fins de débogage, à se connecter à votre serveur d’API.

## <a name="using-aks-private-clusters"></a>Utilisation des clusters privés AKS

À ce stade, Azure Dev Spaces n’est pas pris en charge avec des [clusters privés AKS][aks-private-clusters].

## <a name="using-different-endpoint-options"></a>Utilisation de différentes options de point de terminaison

Azure Dev Spaces a la possibilité d’exposer des points de terminaison pour vos services s’exécutant sur AKS. Lorsque vous activez Azure Dev Spaces sur votre cluster, vous disposez des options suivantes pour configurer le type de point de terminaison de votre cluster :

* Un point de terminaison *public*, ce qui est le comportement par défaut, déploie un contrôleur d’entrée avec une adresse IP publique. L’adresse IP publique est inscrite sur le DNS du cluster, ce qui permet un accès public à vos services à l’aide d’une URL. Vous pouvez afficher cette URL à l’aide de `azds list-uris`.
* Un point de terminaison *privé* déploie un contrôleur d’entrée avec une adresse IP privée. Avec une adresse IP privée, l’équilibreur de charge pour votre cluster est accessible uniquement à partir du réseau virtuel du cluster. L’adresse IP privée de l’équilibreur de charge est inscrite sur le DNS du cluster afin que les services à l’intérieur du réseau virtuel du cluster soient accessibles à l’aide d’une URL. Vous pouvez afficher cette URL à l’aide de `azds list-uris`.
* Si vous définissez *aucun* pour l’option de point de terminaison, aucun contrôleur d’entrée n’est déployé. Si aucun contrôleur d’entrée n’est déployé, les [fonctionnalités de routage d’Azure Dev Spaces][dev-spaces-routing] ne fonctionneront pas. Si vous le souhaitez, vous pouvez implémenter votre propre solution de contrôleur d’entrée à l’aide de [traefik][traefik-ingress] ou [NGINX][nginx-ingress], ce qui permettra aux fonctionnalités de routage de fonctionner à nouveau.

Pour configurer votre option de point de terminaison, utilisez *-e* ou *--endpoint* lors de l’activation d’Azure Dev Spaces sur votre cluster. Par exemple :

> [!NOTE]
> L’option de point de terminaison nécessite que vous exécutiez Azure CLI version 2.2.0 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

```azurecli
az aks use-dev-spaces -g MyResourceGroup -n MyAKS -e private
```

## <a name="client-requirements"></a>Configuration requise des clients

Azure Dev Spaces utilise des outils côté client, tels que l’extension d’interface CLI Azure Dev Spaces, l’extension Visual Studio Code et l’extension Visual Studio, pour communiquer avec votre cluster AKS à des fins de débogage. Pour utiliser les outils Azure Dev Spaces côté client, autorisez le trafic à partir des machines de développement vers l’[infrastructure Azure Dev Spaces][dev-spaces-allow-infrastructure]. Si vous utilisez des [plages d’adresses IP autorisées du serveur d’API][auth-range-section], vous devez également autoriser l’adresse IP de toutes les machines de développement qui se connectent à votre cluster AKS à des fins de débogage, à se connecter à votre serveur d’API.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage plus sur le fonctionnement d’Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Fonctionnement d’Azure Dev Spaces](how-dev-spaces-works.md)

[aks-cni]: ../aks/configure-azure-cni.md
[aks-cni-ip-planning]: ../aks/configure-azure-cni.md#plan-ip-addressing-for-your-cluster
[aks-kubenet]: ../aks/configure-kubenet.md
[aks-internal-lb]: ../aks/internal-lb.md
[aks-ip-auth-ranges]: ../aks/api-server-authorized-ip-ranges.md
[aks-ip-auth-range-create]: ../aks/api-server-authorized-ip-ranges.md#create-an-aks-cluster-with-api-server-authorized-ip-ranges-enabled
[aks-ip-auth-range-update]: ../aks/api-server-authorized-ip-ranges.md#update-a-clusters-api-server-authorized-ip-ranges
[aks-network-policies]: ../aks/use-network-policies.md
[aks-private-clusters]: ../aks/private-clusters.md
[auth-range-section]: #using-api-server-authorized-ip-ranges
[azure-cli-install]: /cli/azure/install-azure-cli
[dev-spaces-allow-infrastructure]: #virtual-network-or-subnet-configurations
[dev-spaces-routing]: how-dev-spaces-works-routing.md
[endpoint-options]: #using-different-endpoint-options
[firewall-service-tags]: ../firewall/service-tags.md
[traefik-ingress]: how-to/ingress-https-traefik.md
[nginx-ingress]: how-to/ingress-https-nginx.md
[sample-repo]: https://github.com/Azure/dev-spaces/tree/master/advanced%20networking
[service-tags]: ../virtual-network/service-tags-overview.md#available-service-tags