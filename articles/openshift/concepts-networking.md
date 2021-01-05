---
title: Concepts – Schéma de mise en réseau pour Azure Red Hat sur OpenShift 4
description: Schéma et vue d’ensemble de mise en réseau pour Azure Red Hat OpenShift
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: container-service
ms.date: 11/23/2020
ms.openlocfilehash: 9cfe8c7e7d2484649bf458524032365b692c9243
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97093517"
---
# <a name="network-concepts-for-azure-red-hat-openshift-aro"></a>Concepts de réseau pour Azure Red Hat OpenShift (ARO)

Ce guide propose une vue d’ensemble des réseaux dans des clusters Azure Red Hat OpenShift sur OpenShift 4, ainsi qu’un diagramme et une liste des points de terminaison importants. Pour plus d’informations sur les principaux concepts de réseau OpenShift, consultez la [documentation sur les réseaux Azure Red Hat OpenShift 4](https://docs.openshift.com/aro/4/networking/understanding-networking.html).

![Schéma de mise en réseau Azure Red Hat OpenShift 4](./media/concepts-networking/aro4-networking-diagram.png)

Lorsque vous déployez Azure Red Hat sur OpenShift 4, l’ensemble de votre cluster est contenu dans un réseau virtuel. Dans ce réseau virtuel, vos nœuds principaux et les nœuds Worker résident tous dans leur propre sous-réseau. Chaque sous-réseau utilise un équilibreur de charge interne et un équilibreur de charge public.

## <a name="networking-components"></a>Composants de mise en réseau

La liste suivante présente les composants réseau importants d’un cluster Azure Red Hat OpenShift.

* **aro-pls**
    * Il s’agit d’un point de terminaison Azure Private Link utilisé par les ingénieurs de fiabilité des sites Microsoft et Red Hat pour gérer le cluster.
* **aro-internal-lb**
    * Ce point de terminaison équilibre le trafic vers le serveur d’API. Pour cet équilibreur de charge, les nœuds principaux se trouvent dans le pool de back-ends.
* **aro-public-lb**
    * Si l’API est publique, ce point de terminaison route et équilibre le trafic vers le serveur d’API. Ce point de terminaison assigne une adresse IP sortante publique afin que les maîtres puissent accéder à Azure Resource Manager et rapporter l’intégrité du cluster.
* **aro-internal**
    * Ce point de terminaison équilibre le trafic interne du service. Pour cet équilibreur de charge, les nœuds Worker se trouvent dans le pool de back-ends.
    * Cet équilibreur de charge n’est pas créé par défaut. Cet équilibreur de charge est créé une fois que vous avez créé un service de type LoadBalancer avec les annotations correctes. Par exemple : service.beta.kubernetes.io/azure-load-balancer-internal: "true".
* **aro-internal-lb**
    * Ce point de terminaison est utilisé pour tout trafic public. Lorsque vous créez une application et une route, il s’agit du chemin pour le trafic d’entrée.
    * Cet équilibreur de charge couvre également la connectivité Internet sortante à partir de tout pod s’exécutant dans les nœuds Worker via les règles de trafic sortant Azure Load Balancer.
        * Actuellement, les règles de trafic sortant ne sont pas configurables. Elles allouent 1 024 ports TCP à chaque nœud.
        * DisableOutboundSnat n’est pas configuré dans les règles d’équilibreur de charge. Par conséquent, les pods peuvent obtenir comme adresse IP de sortie toute adresse IP publique configurée dans cet équilibreur de charge Azure.
        * En conséquence des deux points précédents, le seul moyen d’ajouter des ports SNAT éphémères consiste à ajouter des services publics de type LoadBalancer à ARO.
* **aro-outbound-pip**
    * Ce point de terminaison sert d’adresse IP publique (PIP) pour les nœuds Worker.
    * Ce point de terminaison permet aux services d’ajouter une adresse IP spécifique provenant d’un cluster Azure Red Hat OpenShift à une liste verte.
* **aro-nsg**
    * Quand vous exposez un service, l’API crée une règle dans ce groupe de sécurité réseau afin que le trafic le traverse et atteigne le plan de contrôle et les nœuds.
    * Par défaut, ce groupe de sécurité réseau autorise tout le trafic sortant. Actuellement, le trafic sortant peut être limité uniquement au plan de contrôle Azure Red Hat OpenShift.
* **aro-controlplane-nsg**
  * Ce point de terminaison autorise uniquement le trafic à entrer via le port 6443 pour les nœuds principaux.
* **Azure Container Registry**
    * Il s’agit d’un registre de conteneurs fourni et utilisé en interne par Microsoft. Ce Registre est en lecture seule et n’est pas destiné à être utilisé par les utilisateurs Azure Red Hat OpenShift.
        * Ce registre fournit des images de plateforme hôte et des composants de cluster. Par exemple, des conteneurs de surveillance ou de journalisation.
        * Les connexions à ce registre se produisent sur le point de terminaison de service (connectivité interne entre les services Azure).
        * Ce registre interne n’est pas disponible en dehors du cluster par défaut.
* **Liaison privée**
    * Autorise la connectivité réseau à partir du plan de gestion dans un cluster pour que les ingénieurs de fiabilité des sites Microsoft et Red Hat facilitent la gestion de votre cluster.

## <a name="networking-policies"></a>Stratégies réseau

* **Entrée** : la stratégie réseau d’entrée est prise en charge dans le cadre d’[OpenShift SDN](https://docs.openshift.com/container-platform/4.5/networking/openshift_sdn/about-openshift-sdn.html). Cette stratégie réseau est activée par défaut et l’application est exécutée par les utilisateurs. Alors que la stratégie réseau d’entrée est conforme à la version V1 de NetworkPolicy, les types Sortie et IPBlock ne sont pas pris en charge.

* **Sortie** : les stratégies réseau de sortie sont prises en charge à l’aide de la fonctionnalité de [pare-feu de sortie](https://docs.openshift.com/container-platform/4.5/networking/openshift_sdn/configuring-egress-firewall.html) d’OpenShift. Il n’y a qu’une seule stratégie de sortie par espace de noms/projet. Les stratégies de sortie ne sont pas prises en charge sur l’espace de noms « par défaut » et sont évaluées dans l’ordre (du premier au dernier).

## <a name="networking-basics-in-openshift"></a>Notions de base des réseaux dans OpenShift

OpenShift [SDN](https://docs.openshift.com/container-platform/4.5/networking/openshift_sdn/about-openshift-sdn.html) (Software Defined Networking) est utilisé pour configurer un réseau de superposition à l’aide d’Open vSwitch [(OVS)](https://www.openvswitch.org/), une implémentation OpenFlow basée sur la spécification CNI (Container Network Interface). SDN prend en charge différents plug-ins, la Stratégie réseau étant celui utilisé dans Azure Red Hat sur OpenShift 4. Toutes les communications réseau sont gérées par SDN, de sorte qu’aucune route supplémentaire n’est nécessaire sur vos réseaux virtuels pour mettre en place une communication de pod à pod.

## <a name="networking--for-azure-red-hat-openshift"></a>Réseau pour Azure Red Hat OpenShift

Les fonctionnalités réseau suivantes sont spécifiques à Azure Red Hat OpenShift :
* Les utilisateurs peuvent créer leur cluster ARO dans un réseau virtuel existant ou créer un réseau virtuel lors de la création de leur cluster ARO.
* Les CIDR des réseaux de service et de pod sont configurables.
* Les nœuds et les maîtres figurent dans des sous-réseaux différents.
* Les sous-réseaux de réseau virtuel des nœuds et maîtres doivent être /27 au minimum.
* Le CIDR de pod doit avoir une taille minimale de /18 (le réseau de pod est constitué d’adresses IP non routables et n’est utilisé qu’à l’intérieur d’OpenShift SDN).
* Chaque nœud se voit allouer un sous-réseau /23 (512 adresses IP) pour ses pods. Cette valeur ne peut pas être modifiée.
* Vous ne pouvez pas attacher un pod à plusieurs réseaux.
* Vous ne pouvez pas configurer une adresse IP statique de sortie. (Ceci est une fonctionnalité OpenShift. Pour obtenir des informations, consultez la documentation sur la [configuration des adresses IP de sortie](https://docs.openshift.com/container-platform/4.5/networking/openshift_sdn/assigning-egress-ips.html)).

## <a name="network-settings"></a>Paramètres réseau

Les paramètres réseau suivants sont disponibles pour les clusters Azure Red Hat OpenShift 4 :

* **Visibilité de l’API** – Définissez la visibilité de l’API lors de l’exécution de la [commande az aro create](tutorial-create-cluster.md#create-the-cluster).
    * « Publique » – Le serveur d’API est accessible par les réseaux externes.
    * « Privée » – Le serveur d’API a affecté une adresse IP privée à partir du sous-réseau des maîtres, accessible uniquement à l’aide des réseaux connectés (réseaux virtuels appairés, autres sous-réseaux du cluster). Une zone DNS privée est créée au nom du client.
* **Visibilité des entrées** – Définissez la visibilité de l’API lors de l’exécution de la [commande az aro create](tutorial-create-cluster.md#create-the-cluster).
    * Les routes « publiques » sont définies par défaut sur l’équilibreur de charge public Standard Load Balancer (vous pouvez changer cela).
    * Les routes « privées » sont définies par défaut sur l’équilibreur de charge interne (vous pouvez modifier cela).

## <a name="network-security-groups"></a>Groupes de sécurité réseau
Les groupes de sécurité réseau sont créés dans le groupe de ressources du nœud, qui est verrouillé pour les utilisateurs. Les groupes de sécurité réseau sont attribués directement aux sous-réseaux et non aux cartes réseau du nœud. Les groupes de sécurité réseau sont non modifiables, et les utilisateurs ne disposent pas d’autorisations pour les changer.

Avec un serveur d’API visible publiquement, vous ne pouvez pas créer des groupes de sécurité réseau et les affecter aux cartes réseau.

## <a name="domain-forwarding"></a>Transfert de domaine
Azure Red Hat OpenShift utilise CoreDNS. Le transfert de domaine peut être configuré. Vous ne pouvez pas apporter votre propre serveur DNS dans vos réseaux virtuels. Pour plus d'informations, consultez la documentation sur l’[utilisation du transfert DNS](https://docs.openshift.com/aro/4/networking/dns-operator.html#nw-dns-forward_dns-operator).

## <a name="whats-new-in-openshift-45"></a>Nouveautés d’OpenShift 4.5

Avec la prise en charge d’OpenShift 4.5, Azure Red Hat OpenShift a introduit quelques changements architecturaux importants. Ces changements s’appliquent uniquement aux clusters nouvellement créés qui exécutent OpenShift 4.5. L’architecture réseau des clusters existants qui ont été mis à niveau vers OpenShift 4.5 n’est pas modifiée par le processus de mise à niveau. Les utilisateurs devront recréer leurs clusters pour utiliser cette nouvelle architecture.

![Diagramme des réseaux Azure Red Hat OpenShift 4.5](./media/concepts-networking/aro-4-5-networking-diagram.png)

Comme indiqué dans le diagramme ci-dessus, vous remarquerez quelques changements :
* Avant, ARO utilisait deux équilibreurs de charge publics : un pour le serveur d’API et un pour le pool de nœuds worker. Avec cette mise à jour de l’architecture, ces éléments ont été regroupés sous un seul et même équilibreur de charge. 
* Pour réduire la complexité, les ressources d’adresses IP sortantes dédiées ont été supprimées.
* Le plan de contrôle ARO partage désormais le même groupe de sécurité réseau que les nœuds worker ARO.

Pour plus d’informations sur OpenShift 4.5, consultez les [notes de publication OpenShift 4.5](https://docs.openshift.com/container-platform/4.5/release_notes/ocp-4-5-release-notes.html).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le trafic sortant et sur ce qu’Azure Red Hat OpenShift prend en charge pour la sortie, consultez la documentation sur les [stratégies de prise en charge](support-policies-v4.md).
