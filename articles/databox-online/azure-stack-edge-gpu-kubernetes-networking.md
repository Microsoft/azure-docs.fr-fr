---
title: Comprendre la mise en réseau Kubernetes sur un appareil Azure Stack Edge Pro | Microsoft Docs
description: Décrit le fonctionnement de la mise en réseau Kubernetes sur un appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 08/21/2020
ms.author: alkohli
ms.openlocfilehash: 001304ad6eda27db2285aaa9ad8b28929e2a04f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90899323"
---
# <a name="kubernetes-networking-in-your-azure-stack-edge-pro-gpu-device"></a>Mise en réseau Kubernetes dans votre appareil Azure Stack Edge Pro avec GPU

Sur votre appareil Azure Stack Edge Pro, un cluster Kubernetes est créé lorsque vous configurez le rôle de calcul. Une fois le cluster Kubernetes créé, les applications en conteneur peuvent être déployées sur le cluster Kubernetes dans des pods. Il existe des moyens distincts d’utiliser la mise en réseau pour les pods dans votre cluster Kubernetes. 

Cet article décrit la mise en réseau dans un cluster Kubernetes en général et spécifiquement dans le contexte de votre appareil Azure Stack Edge Pro. 

## <a name="networking-requirements"></a>Configuration requise du réseau

Voici un exemple d’application à 2 niveaux standard déployée sur le cluster Kubernetes.

- L’application dispose d’un serveur web frontal et d’une application de base de données dans le serveur principal. 
- Une adresse IP est affectée à chaque pod, mais ces adresses IP peuvent changer au redémarrage et au basculement du pod. 
- Chaque application est constituée de plusieurs pods, et il doit y avoir un équilibrage de la charge du trafic sur tous les réplicas de pods. 

![Configuration réseau requise pour Kubernetes](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-networking-1.png)

Le scénario ci-dessus entraîne les exigences de mise en réseau suivantes :

 - L’accès à l’application externe doit être effectué par un utilisateur d’application en dehors du cluster Kubernetes via un nom ou une adresse IP. 
 - Les applications au sein du cluster Kubernetes, par exemple, les pods frontaux et de back-end doivent être en mesure de communiquer entre eux.

Pour résoudre les deux besoins, nous introduisons un service Kubernetes. 


## <a name="networking-services"></a>Services de mise en réseau

Il existe deux types de services Kubernetes : 

- **Service d’adresse IP de cluster** : fournit un point de terminaison constant pour les pods d’application. Les pods associés à ces services ne sont pas accessibles à partir de l’extérieur du cluster Kubernetes. L’adresse IP utilisée avec ces services provient de l’espace d’adressage dans le réseau privé. 
- **Adresse IP de l’équilibreur de charge** : comme le service d’adresse IP du cluster, mais l’adresse IP associée provient du réseau externe et est accessible depuis l’extérieur du cluster Kubernetes.


<!--## Networking example for an app


![Kubernetes networking example](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-networking-2.png)

Each of these applications pods has a label associated with it. For example, the web server application pods have a label `app = WS` and the service has a label selector which the same as `app = WS`. Whenever a service of type load balancer or cluster IP is created, there is a control loop that runs in the master and publishes an endpoint corresponding to this service. This service uses a combination of labels and label selectors to discover the pods associated with this service. As a pod gets created, the new endpoint for the pod is added to the endpoint mapping. Whenever a pod is deleted, it gets deleted from the endpoint mapping. Using this endpoint controller, the service has a most up-to-date view of the pods that make up this application.

For discovery of applications within the cluster, Kubernetes cluster has an inbuilt DNS server pod. This is a cluster DNS that resolves service names to cluster IP. Anytime a cluster IP service is created, a DNS record is added to the DNS server that maps the name of the service to the cluster internal IP. That is how the applications within the cluster can discover each other. For load balancing, there is also the `kube-proxy`. This runs on every node and captures the traffic that comes in through the cluster IP and then distributes the traffic across the pods. 

When an application or the end user would first use the IP address associated with the service of type load balancer to discover the service. Then it would use the label select `app = WS` to discover the pods associated with the application. The `kube-proxy` component would then distribute the traffic and ensure that it hits one of the web server application pods. If the web server app wanted to talk to the database app, then it would simply use the name of the service and using the name and the DNS server pod, resolve the name to an IP address. Again using labels and label selector, it would discover the pods associated with the database application. The `kube-proxy` would then distribute the traffic across each of the database app nodes.-->


## <a name="kubernetes-networking-on-azure-stack-edge-pro"></a>Mise en réseau Kubernetes sur Azure Stack Edge Pro

Les serveurs DNS Calico, Metallb et Core sont tous les composants installés pour la mise en réseau sur votre appareil Azure Stack Edge Pro. 

- **Calico** affecte une adresse IP à partir d’une plage d’adresses IP privées à chaque pod et configure la mise en réseau pour ces pods afin qu’un pod sur un nœud puisse communiquer avec le pod sur un autre nœud. 
- **Metallb** s’exécute sur un pod dans le cluster et attribue l’adresse IP aux services de type équilibreur de charge. Les adresses IP de l’équilibreur de charge sont choisies dans la plage d’adresses IP de service fournie par le biais de l’interface utilisateur locale. 
- **DNS de base** : ce module complémentaire configure le nom du service de mappage des enregistrements DNS sur l’adresse IP du cluster.

Les adresses IP utilisées pour les nœuds Kubernetes et les services externes sont fournies via la page **Réseau de calcul** de l’interface utilisateur locale de l’appareil.

![Attribution d’adresses IP Kubernetes dans l’interface utilisateur locale](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-ip-assignment-local-ui-1.png)

L’attribution d’adresses IP est destinée aux :

- **Adresses IP des nœuds Kubernetes** : Cette plage d’adresses IP est utilisée pour les nœuds principaux et Worker Kubernetes. Ces adresses IP sont utilisées lorsque les nœuds Kubernetes communiquent entre eux.

- **Adresses IP des services externes Kubernetes** : Cette plage d’adresses IP est utilisée pour les services externes (également appelés services d’équilibreur de charge) qui sont exposés en dehors du cluster Kubernetes.  


## <a name="next-steps"></a>Étapes suivantes

Pour configurer la mise en réseau Kubernetes sur votre appareil, Azure Stack Edge Pro, consultez :

- [Exposez une application sans état en externe sur votre appareil Azure Stack Edge Pro via IoT Edge](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md).

- [Exposez une application sans état en externe sur votre appareil Azure Stack Edge Pro via kuebctl](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md).
