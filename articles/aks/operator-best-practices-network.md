---
title: Meilleures pratiques pour les ressources réseau
titleSuffix: Azure Kubernetes Service
description: Découvrez les meilleures pratiques de l’opérateur pour les ressources de réseau virtuel et la connectivité dans Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 12/10/2018
ms.openlocfilehash: 9ec6423a853aacbc8a03cc5472bf1a95a5623b1f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89482723"
---
# <a name="best-practices-for-network-connectivity-and-security-in-azure-kubernetes-service-aks"></a>Meilleures pratiques pour la connectivité réseau et la sécurité dans Azure Kubernetes Service (AKS)

Lorsque vous créez et gérez des clusters dans Azure Kubernetes Service (AKS), vous assurez une connectivité réseau à vos nœuds et à vos applications. Parmi ces ressources réseau figurent des plages d’adresses IP, des équilibreurs de charge et des contrôleurs d’entrée. Pour maintenir une qualité de service élevée dans les applications, il faut planifier, puis configurer ces ressources.

Cet article porte sur les meilleures pratiques en matière de connectivité réseau et de sécurité pour les opérateurs de clusters. Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * comparer les modes réseau kubenet et Azure CNI dans Azure Kubernetes Service ;
> * prévoir l’adressage IP et la connectivité nécessaires ;
> * distribuer le trafic à l’aide d’équilibreurs de charge, de contrôleurs d’entrée ou de pare-feu d’applications web ;
> * se connecter en toute sécurité aux nœuds de cluster.

## <a name="choose-the-appropriate-network-model"></a>Choisir le modèle réseau adapté

**Meilleures pratiques** : pour une intégration avec des réseaux virtuels existants ou des réseaux locaux, utilisez la mise en réseau Azure CNI dans AKS. Ce modèle réseau offre également une meilleure séparation des ressources et plus de contrôle dans un environnement d’entreprise.

Les réseaux virtuels assurent une connectivité de base pour les nœuds AKS et les clients qui accèdent à vos applications. Il existe deux façons différentes de déployer des clusters AKS dans des réseaux virtuels :

* **Mise en réseau Kubenet** : Azure gère les ressources de réseau virtuel pendant le déploiement du cluster et utilise le plug-in Kubernetes [Kubenet][kubenet].
* **Mise en réseau Azure CNI** : effectue le déploiement dans un réseau virtuel, et utilise le plug-in Kubernetes [Azure Container Networking Interface (CNI)][cni-networking]. Les pods reçoivent des adresses IP individuelles qui peuvent communiquer avec d’autres services réseau ou ressources locales.

L’interface Azure CNI est un protocole indépendant du fournisseur qui permet au runtime du conteneur d’adresser des demandes à un fournisseur de réseau. Elle affecte des adresses IP aux pods et aux nœuds et offre des fonctionnalités de Gestion des adresses IP (IPAM) pour la connexion à des réseaux virtuels Azure existants. Chaque ressource de type nœud ou pod reçoit une adresse IP dans le réseau virtuel Azure, sans qu’aucun routage supplémentaire soit nécessaire pour communiquer avec d’autres ressources ou services.

![Diagramme représentant 2 nœuds avec des ponts les reliant chacun à un réseau virtuel Azure](media/operator-best-practices-network/advanced-networking-diagram.png)

Pour les déploiements de production, kubenet et Azure CNI sont tous deux des options valides.

Un des avantages notables de la mise en réseau Azure CNI pour la production est que le modèle réseau permet de séparer le contrôle et la gestion des ressources. Du point de vue de la sécurité, il est souvent préférable que différentes équipes gèrent et sécurisent ces ressources. Une mise en réseau Azure CNI vous permet de vous connecter directement à des ressources Azure existantes, à des ressources locales ou à d’autres services avec les adresses IP assignées à chacun des pods.

Avec une mise en réseau Azure CNI, la ressource de réseau virtuel se trouve dans un groupe de ressources distinct du cluster AKS. Déléguez des permissions au principal de service AKS pour qu’il puisse accéder à ces ressources et les gérer. Le principal du service utilisé par le cluster AKS doit disposer au moins des autorisations [Contributeur de réseau](../role-based-access-control/built-in-roles.md#network-contributor) sur le sous-réseau de votre réseau virtuel. Si vous souhaitez définir un [rôle personnalisé](../role-based-access-control/custom-roles.md) au lieu d’utiliser le rôle de contributeur de réseau intégré, les autorisations suivantes sont nécessaires :
  * `Microsoft.Network/virtualNetworks/subnets/join/action`
  * `Microsoft.Network/virtualNetworks/subnets/read`

Pour plus d’informations sur la délégation du principal du service AKS, voir [Déléguer l’accès à d’autres ressources Azure][sp-delegation]. Au lieu d’utiliser le principal de service, vous pouvez aussi utiliser l’identité managée affectée par le système pour les autorisations. Pour plus d’informations, consultez [Utiliser des identités managées](use-managed-identity.md).

Dans la mesure où chaque nœud et chaque pod reçoit sa propre adresse IP, planifiez les plages d’adresses des sous-réseaux AKS. Le sous-réseau doit être assez grand pour offrir une adresse IP à chacun des nœuds, pods et ressources réseau déployés. Chaque cluster AKS doit être placé dans son propre sous-réseau. Pour autoriser la connexion à des réseaux locaux ou en peering dans Azure, n’utilisez pas de plages d’adresses IP qui recouvrent des ressources réseau existantes. Des limites par défaut s’appliquent au nombre de pods qu’exécute chaque nœud avec une mise en réseau Kubenet ou Azure CNI. Pour gérer les événements de scale-out et les mises à niveau de cluster, des adresses IP supplémentaires sont également nécessaires dans le sous-réseau attribué. Cet espace d’adressage supplémentaire est particulièrement important si vous utilisez des conteneurs Windows Server, car ces pools de nœuds nécessitent une mise à niveau pour appliquer les derniers correctifs de sécurité. Pour plus d’informations sur les nœuds Windows Server, consultez [Mettre à niveau un pool de nœuds dans AKS][nodepool-upgrade].

Pour calculer l’adresse IP requise, voir [Configurer la mise en réseau Azure CNI dans AKS][advanced-networking].

### <a name="kubenet-networking"></a>Mise en réseau Kubenet

Si la mise en réseau Kubenet n’impose pas de configurer les réseaux virtuels avant le déploiement du cluster, elle présente des inconvénients :

* Les nœuds et les pods sont placés sur différents sous-réseaux IP. Le routage défini par l’utilisateur (UDR) et le transfert IP sont utilisés pour acheminer le trafic entre les nœuds et les pods. Ce routage supplémentaire peut réduire les performances du réseau.
* Les connexions à des réseaux locaux existants et le peering avec d’autres réseaux virtuels Azure peuvent être complexes.

Une mise en réseau Kubenet convient pour de petites charges de travail de développement et de test, dans la mesure où il n’est pas nécessaire de créer le réseau virtuel et les sous-réseaux séparément du cluster AKS. Les sites web simples recevant peu de trafic et le lift-and-shift de charges de travail dans des conteneurs peuvent également tirer avantage de la simplicité des clusters AKS déployés avec la mise en réseau Kubenet. Pour la plupart des déploiements de production, vous devez planifier et utiliser une mise en réseau Azure CNI. Vous pouvez également [configurer vos propres plages d’adresses IP et réseaux virtuels à l’aide de Kubenet][aks-configure-kubenet-networking].

## <a name="distribute-ingress-traffic"></a>Distribuer le trafic d’entrée

**Meilleures pratiques** : pour répartir le trafic HTTP ou HTTPS sur vos applications, utilisez des contrôleurs et des ressources d’entrée. Les contrôleurs d’entrée offrent des fonctionnalités supplémentaires par rapport à un équilibreur de charge Azure standard. Ils peuvent être gérés comme des ressources Kubernetes natives.

Un équilibreur de charge Azure peut distribuer le trafic de client aux applications du cluster AKS, mais sa compréhension du trafic reste limitée. Une ressource d’équilibrage de charge, qui agit au niveau de la couche 4, répartit le trafic en fonction du protocole ou des ports. La plupart des applications web qui utilisent HTTP ou HTTPS doivent de préférence utiliser des contrôleurs et des ressources d’entrée Kubernetes, qui fonctionnent au niveau de la couche 7. L’entrée peut distribuer le trafic en fonction de l’URL de l’application et gérer l’arrêt TLS/SSL. Cette capacité réduit également le nombre d’adresses IP exposées et mappées. Avec un équilibreur de charge, une adresse IP publique doit généralement être affectée et mappée au service dans le cluster AKS pour chaque application. Avec une ressource d’entrée, une seule adresse IP peut répartir le trafic entre plusieurs applications.

![Diagramme montrant le flux de trafic d’entrée dans un cluster AKS](media/operator-best-practices-network/aks-ingress.png)

 Il existe deux composants d’entrée :

 * une *ressource* d’entrée ;
 * un *contrôleur* d’entrée.

La ressource d’entrée est un manifeste YAML de `kind: Ingress` qui définit l’hôte, les certificats et les règles de routage du trafic vers les services qui s’exécutent dans le cluster AKS. L’exemple de manifeste YAML suivant distribuerait le trafic de *myapp.com* à un des deux services, *blogservice* ou *storeservice*. Le client est dirigé vers l’un ou l’autre selon l’URL consultée.

```yaml
kind: Ingress
metadata:
 name: myapp-ingress
   annotations: kubernetes.io/ingress.class: "PublicIngress"
spec:
 tls:
 - hosts:
   - myapp.com
   secretName: myapp-secret
 rules:
   - host: myapp.com
     http:
      paths:
      - path: /blog
        backend:
         serviceName: blogservice
         servicePort: 80
      - path: /store
        backend:
         serviceName: storeservice
         servicePort: 80
```

Un contrôleur d’entrée est un démon qui s’exécute sur un nœud AKS et surveille les demandes entrantes. Le trafic est ensuite distribué selon les règles définies dans la ressource d’entrée. Le contrôleur d’entrée le plus courant est basé sur [NGINX], mais AKS ne limite pas à un contrôleur spécifique : vous pouvez utiliser d’autres contrôleurs comme [Contour][contour], [HAProxy][haproxy] ou [Traefik][traefik].

Les contrôleurs d’entrée doivent être planifiés sur un nœud Linux. Les nœuds Windows Server ne doivent pas exécuter le contrôleur d’entrée. Utilisez un sélecteur de nœud dans votre manifeste YAML ou votre déploiement de graphique Helm pour indiquer que la ressource doit s’exécuter sur un nœud Linux. Pour plus d’informations, consultez [Use node selectors to control where pods are scheduled in AKS (Utiliser des sélecteurs de nœud pour contrôler l’emplacement de planification des pods dans AKS)][concepts-node-selectors].

Il existe de nombreux scénarios pour l’entrée, notamment ceux des guides pratiques suivants :

* [Créer un contrôleur d’entrée dans Azure Kubernetes Service (AKS)][aks-ingress-basic]
* [Create an ingress controller to an internal virtual network in Azure Kubernetes Service (AKS)][aks-ingress-internal] (Créer un contrôleur d’entrée pour un réseau virtuel interne dans Azure Kubernetes Service (AKS))
* [Créer un contrôleur d’entrée qui utilise vos propres certificats TLS][aks-ingress-own-tls]
* Créer un contrôleur d’entrée qui utilise Let’s Encrypt pour générer automatiquement des certificats TLS [avec une adresse IP publique dynamique][aks-ingress-tls] ou [avec une adresse IP publique statique][aks-ingress-static-tls]

## <a name="secure-traffic-with-a-web-application-firewall-waf"></a>Sécuriser le trafic avec un pare-feu d’applications web (WAF)

**Meilleures pratiques** : pour analyser le risque d’attaques dans le trafic, utilisez un pare-feu d’applications web (WAF) comme [Barracuda pour Azure][barracuda-waf] ou Azure Application Gateway. Ces ressources réseau plus avancées peuvent également acheminer le trafic au-delà des connexions HTTP et HTTPS et de l’arrêt TLS de base.

Un contrôleur d’entrée qui distribue le trafic aux services et applications est généralement une ressource Kubernetes du cluster AKS. Le contrôleur s’exécute en tant que démon sur un nœud AKS et consomme des ressources du nœud, comme le processeur, la mémoire et la bande passante réseau. Dans les environnements de grande taille, il est souvent nécessaire de décharger une partie du routage du trafic ou de l’arrêt TLS sur une ressource réseau située en dehors du cluster AKS. L’objectif est également d’analyser le risque d’attaques dans le trafic entrant.

![Un pare-feu d’applications web (WAF) comme Azure Application Gateway peut protéger et distribuer le trafic d’un cluster AKS](media/operator-best-practices-network/web-application-firewall-app-gateway.png)

Un pare-feu d’applications web (WAF) ajoute une couche de sécurité supplémentaire en filtrant le trafic entrant. Le projet OWASP (Open Web Application Security Project) propose un ensemble de règles pour surveiller les attaques de type script intersites ou cookie poisoning. [Azure Application Gateway][app-gateway] (actuellement en préversion dans AKS) est un pare-feu WAF capable d’intégrer ces fonctionnalités de sécurité dans des clusters AKS, avant que le trafic n’atteigne les clusters et les applications. D’autres solutions tierces assurent également ces fonctions, ce qui peut permettre de continuer à utiliser les investissements et l’expertise déjà acquis sur un produit donné.

Les ressources d’équilibrage de charge ou d’entrée continuent de s’exécuter dans le cluster AKS pour affiner davantage la distribution du trafic. App Gateway peut être géré de manière centralisée comme contrôleur d’entrée avec une définition de ressource. Pour commencer, voir [Créer un contrôleur d’entrée Application Gateway][app-gateway-ingress].

## <a name="control-traffic-flow-with-network-policies"></a>Contrôler le flux de trafic avec des stratégies réseau

**Bonnes pratiques** - Utilisez des stratégies réseau pour autoriser ou refuser le trafic vers les pods. Par défaut, tout le trafic est autorisé entre les pods au sein d’un cluster. Pour une sécurité accrue, définissez des règles qui limitent la communication des pods.

L’utilisation de stratégies réseau est une fonctionnalité Kubernetes qui vous permet de contrôler le flux de trafic entre les pods. Vous pouvez choisir d’autoriser ou de refuser un trafic en fonction de paramètres, tels que des étiquettes attribuées, un espace de noms ou un port de trafic. L’utilisation de stratégies réseau offre une méthode native du cloud pour contrôler le flux de trafic. Les pods étant créés de façon dynamique dans un cluster AKS, les stratégies réseau nécessaires peuvent être appliquées automatiquement. N’utilisez pas des groupes de sécurité réseau Azure pour contrôler le trafic de pod à pod, mais plutôt des stratégies réseau.

Pour utiliser une stratégie réseau, la fonctionnalité doit être activée lorsque vous créez un cluster AKS. Vous ne pouvez pas activer une stratégie réseau sur un cluster AKS existant. Prévoyez le temps nécessaire pour vérifier que vous activez la stratégie réseau sur les clusters et que vous pouvez les utiliser selon vos besoins. Une stratégie réseau doit uniquement être utilisée pour les nœuds et pods Linux dans AKS.

Une stratégie réseau est créée en tant que ressource Kubernetes à l’aide d’un manifeste YAML. Les stratégies sont appliquées à des pods définis, puis des règles d’entrée ou de sortie définissent la circulation du trafic. L’exemple suivant applique une stratégie réseau à des pods dotés de l’étiquette *app: backend*. La règle d’entrée autorise ensuite uniquement le trafic provenant des pods dotés de l’étiquette *app: frontend* :

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

Pour commencer à utiliser des stratégies, consultez [Sécuriser le trafic entre les pods avec des stratégies réseau dans Azure Kubernetes Service (AKS)][use-network-policies].

## <a name="securely-connect-to-nodes-through-a-bastion-host"></a>Se connecter en toute sécurité aux nœuds à travers un hôte bastion

**Meilleures pratiques** : n’exposez pas de connectivité à distance sur vos nœuds AKS. Créez un hôte bastion, ou une jumpbox, dans un réseau virtuel de gestion. Utilisez l’hôte bastion pour acheminer le trafic en toute sécurité dans votre cluster AKS aux tâches de gestion à distance.

La plupart des opérations effectuées dans AKS peuvent exploiter des outils de gestion Azure ou le serveur d’API Kubernetes. Les nœuds AKS, non connectés à l’Internet public, ne sont disponibles que sur un réseau privé. Pour vous connecter aux nœuds et effectuer des tâches de maintenance ou résoudre des problèmes, acheminez vos connexions à travers un hôte bastion ou une jumpbox. Cet hôte doit se trouver dans un réseau virtuel de gestion distinct, qui offre un peering sécurisé avec le réseau virtuel du cluster AKS.

![Se connecter à des nœuds AKS à l’aide d’un hôte bastion ou d’une jumpbox](media/operator-best-practices-network/connect-using-bastion-host-simplified.png)

Le réseau de gestion de l’hôte bastion doit lui aussi être sécurisé. Utilisez [Azure ExpressRoute][expressroute] ou une [passerelle VPN][vpn-gateway] pour vous connecter à un réseau local et contrôler l’accès avec des groupes de sécurité réseau.

## <a name="next-steps"></a>Étapes suivantes

Cet article porte sur la sécurité et la connectivité réseau. Pour plus d’informations sur les principes fondamentaux des réseaux dans Kubernetes, voir [Concepts relatifs au réseau des applications dans Azure Kubernetes Service (AKS)][aks-concepts-network].

<!-- LINKS - External -->
[cni-networking]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[kubenet]: https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#kubenet
[app-gateway-ingress]: https://github.com/Azure/application-gateway-kubernetes-ingress
[nginx]: https://www.nginx.com/products/nginx/kubernetes-ingress-controller
[contour]: https://github.com/heptio/contour
[haproxy]: https://www.haproxy.org
[traefik]: https://github.com/containous/traefik
[barracuda-waf]: https://www.barracuda.com/products/webapplicationfirewall/models/5

<!-- INTERNAL LINKS -->
[aks-concepts-network]: concepts-network.md
[sp-delegation]: kubernetes-service-principal.md#delegate-access-to-other-azure-resources
[expressroute]: ../expressroute/expressroute-introduction.md
[vpn-gateway]: ../vpn-gateway/vpn-gateway-about-vpngateways.md
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-ingress-tls]: ingress-tls.md
[aks-ingress-own-tls]: ingress-own-tls.md
[app-gateway]: ../application-gateway/overview.md
[use-network-policies]: use-network-policies.md
[advanced-networking]: configure-azure-cni.md
[aks-configure-kubenet-networking]: configure-kubenet.md
[concepts-node-selectors]: concepts-clusters-workloads.md#node-selectors
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool