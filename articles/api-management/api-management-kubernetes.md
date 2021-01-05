---
title: Utiliser Gestion des API Azure avec des microservices déployés dans Azure Kubernetes Service | Microsoft Docs
description: Cet article décrit les options de déploiement de Gestion des API avec AKS
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 12/14/2019
ms.author: apimpm
ms.openlocfilehash: f8c6fce5b22d67dd1022fbaac763ea5df3b0930f
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505373"
---
# <a name="use-azure-api-management-with-microservices-deployed-in-azure-kubernetes-service"></a>Utiliser Gestion des API Azure avec des microservices déployés dans Azure Kubernetes Service

Les microservices sont idéaux pour la création d’API. Avec [Azure Kubernetes Service](https://azure.microsoft.com/services/kubernetes-service/) (AKS), vous pouvez rapidement déployer et exploiter une [architecture basée sur des microservices](/azure/architecture/guide/architecture-styles/microservices) dans le cloud. Vous pouvez ensuite tirer parti de [Gestion des API Azure](https://aka.ms/apimrocks) (Gestion des API) pour publier vos microservices en tant qu’API pour consommation interne et externe. Cet article décrit les options de déploiement de Gestion des API avec AKS. Il suppose une connaissance de base de Kubernetes, de Gestion des API et de la mise en réseau Azure. 

## <a name="background"></a>Arrière-plan

Lorsque vous publiez des microservices en tant qu’API pour consommation, il peut être difficile de gérer la communication entre les microservices et les clients qui les consomment. Il existe une multitude de problèmes transversaux tels que l’authentification, l’autorisation, la limitation, la mise en cache, la transformation et la surveillance. Ces problèmes sont valides que les microservices soient exposés ou non à des clients internes ou externes. 

Le modèle de [passerelle API](/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern) résout ces problèmes. Une passerelle API sert de porte d’entrée pour les microservices, dissocie les clients de vos microservices, ajoute une couche de sécurité supplémentaire et réduit la complexité de vos microservices en supprimant la charge liée à la gestion des problèmes communs. 

[Gestion des API Azure](https://aka.ms/apimrocks) est une solution clé en main pour répondre aux besoins de votre passerelle API. Vous pouvez créer rapidement une passerelle cohérente et moderne pour vos microservices et les publier en tant qu’API. En tant que solution de gestion des API au cycle de vie complet, elle offre également des fonctionnalités supplémentaires, notamment un portail de développement en libre-service pour la découverte d’API, la gestion du cycle de vie des API et l’analytique des API.

En cas d’utilisation conjointe, AKS et Gestion des API offrent une plateforme pour le déploiement, la publication, la sécurisation, la surveillance et la gestion de vos API basées sur des microservices. Dans cet article, nous allons examiner quelques options de déploiement AKS conjointement avec Gestion des API. 

## <a name="kubernetes-services-and-apis"></a>Services Kubernetes et API

Dans un cluster Kubernetes, les conteneurs sont déployés dans des [pods](https://kubernetes.io/docs/concepts/workloads/pods/pod/), qui sont éphémères et ont un cycle de vie. Lorsqu’un nœud Worker est supprimé, les pods exécutés sur le nœud sont perdus. Par conséquent, l’adresse IP d’un pod peut changer à tout moment. Vous ne pouvez pas compter sur ce service pour communiquer avec le pod. 

Pour résoudre ce problème, Kubernetes a introduit le concept de [services](https://kubernetes.io/docs/concepts/services-networking/service/). Un service Kubernetes est une couche d’abstraction qui définit un groupe logique de pods et permet l’exposition du trafic externe, l’équilibrage de charge et la découverte de service pour ces pods. 

Lorsque vous êtes prêt à publier vos microservices en tant qu’API via Gestion des API, vous devez réfléchir à la façon de mapper vos services dans Kubernetes aux API dans Gestion des API. Il n’y a pas de règle définie. Cela dépend de la façon dont vous avez conçu et partitionné vos capacités ou domaines métier en microservices au début. Par exemple, si les pods derrière un service sont responsables de toutes les opérations sur une ressource donnée (par exemple, client), le service peut être mappé à une API. Si les opérations sur une ressource sont partitionnées en plusieurs microservices (par exemple, GetOrder, PlaceOrder), plusieurs services peuvent être agrégés logiquement en une seule API dans Gestion des API (voir Fig. 1). 

Les mappages peuvent également évoluer. Étant donné que Gestion des API crée une façade devant les microservices, vous pouvez refactoriser et redimensionner vos microservices au fil du temps. 

![Mapper des services aux API](./media/api-management-aks/service-api-mapping.png)

## <a name="deploy-api-management-in-front-of-aks"></a>Déployer Gestion des API devant AKS

Il existe plusieurs options de déploiement de Gestion des API devant un cluster AKS. 

Même si un cluster AKS est toujours déployé dans un réseau virtuel (VNet), il n’est pas nécessaire de déployer une instance Gestion des API dans un réseau virtuel. Lorsque Gestion des API ne réside pas dans le réseau virtuel du cluster, le cluster AKS doit publier des points de terminaison publics pour que Gestion des API s’y connecte. Dans ce cas, il est nécessaire de sécuriser la connexion entre Gestion des API et AKS. En d’autres termes, vous devez vous assurer que le cluster est uniquement accessible exclusivement via Gestion des API. Examinons les options. 

### <a name="option-1-expose-services-publicly"></a>Option 1 : exposer les services publiquement

Les services d’un cluster AKS peuvent être exposés publiquement à l’aide des [types de service](../aks/concepts-network.md) NodePort, LoadBalancer ou ExternalName. Dans ce cas, les services sont accessibles directement à partir de l’Internet public. Une fois que Gestion des API a été déployée devant le cluster, vous devez vous assurer que tout le trafic entrant passe par Gestion des API en appliquant l’authentification dans les microservices. Par exemple, Gestion des API peut inclure un jeton d’accès dans chaque demande envoyée au cluster. Chaque microservice est chargé de valider le jeton avant de traiter la demande. 


Il peut s’agir de l’option la plus simple pour déployer Gestion des API devant AKS, en particulier si vous disposez déjà d’une logique d’authentification implémentée dans vos microservices. 

![Publier des services directement](./media/api-management-aks/direct.png)

Avantages :
* Configuration facile du côté Gestion des API, car elle n’a pas besoin d’être injectée dans le réseau virtuel du cluster
* Aucune modification du côté AKS si les services sont déjà exposés publiquement et si la logique d’authentification existe déjà dans les microservices

Inconvénients :
* Risque de sécurité potentiel en raison de la visibilité publique des points de terminaison de service
* Aucun point d’entrée unique pour le trafic de cluster entrant
* Complique les microservices avec une logique d’authentification dupliquée

### <a name="option-2-install-an-ingress-controller"></a>Option n°2 : Installer un contrôleur d’entrée

Bien que l’option 1 puisse être plus facile, elle présente des inconvénients notables comme indiqué ci-dessus. Si une instance Gestion des API ne réside pas dans le réseau virtuel du cluster, l’authentification Mutual TLS (mTLS) est un moyen fiable de garantir que le trafic est sécurisé et approuvé dans les deux sens entre une instance Gestion des API et un cluster AKS. 

L’authentification Mutual TLS est [prise en charge en mode natif](./api-management-howto-mutual-certificates.md) par Gestion des API et peut être activée dans Kubernetes en [installant un contrôleur d’entrée](../aks/ingress-own-tls.md) (Fig. 3). Par conséquent, l’authentification est effectuée dans le contrôleur d’entrée, ce qui simplifie les microservices. En outre, vous pouvez ajouter les adresses IP Gestion des API à la liste autorisée par le contrôleur d’entrée pour vous assurer que seule Gestion des API a accès au cluster.  

 
![Publier via un contrôleur d’entrée](./media/api-management-aks/ingress-controller.png)


Avantages :
* Configuration facile du côté Gestion des API, car elle n’a pas besoin d’être injectée dans le réseau virtuel du cluster et mLTS est pris en charge de façon native
* Centralise la protection du trafic de cluster entrant au niveau de la couche de contrôleur d’entrée
* Réduit les risques de sécurité en réduisant au minimum les points de terminaison de cluster visibles publiquement

Inconvénients :
* Augmente la complexité de la configuration du cluster en raison d’un travail supplémentaire d’installation, de configuration et de maintenance du contrôleur d’entrée et de gestion des certificats utilisés pour mTLS
* Risque de sécurité en raison de la visibilité publique du ou des points de terminaison de contrôleur d’entrée


Lors de la publication d’API via la Gestion des API, il est facile et courant de sécuriser l’accès à ces API au moyen de clés d’abonnement. Les développeurs qui utilisent les API publiées doivent inclure une clé d’abonnement valide dans les requêtes HTTP lorsqu’ils effectuent des appels vers ces API. Sinon, les appels sont immédiatement rejetés par la passerelle de Gestion des API. Ils ne sont pas transférés vers les services backend.

Pour obtenir une clé d’abonnement permettant d’accéder aux API, un abonnement est nécessaire. Un abonnement est avant tout un conteneur nommé pour une paire de clés d’abonnement. Les développeurs devant utiliser les API publiées peuvent obtenir des abonnements. Et ils n’ont pas besoin de l’approbation des éditeurs d’API. Les éditeurs d’API peuvent également créer des abonnements directement pour les consommateurs d’API.

### <a name="option-3-deploy-apim-inside-the-cluster-vnet"></a>Option 3 : Déployer APIM dans le réseau virtuel de cluster

Dans certains cas, les clients ayant des contraintes réglementaires ou des exigences de sécurité strictes peuvent trouver les solutions facultatives 1 et 2 non viables en raison des points de terminaison exposés publiquement. Dans d’autres cas, le cluster AKS et les applications qui consomment les microservices peuvent résider dans le même réseau virtuel, de sorte qu’il n’y a aucune raison d’exposer le cluster publiquement, car tout le trafic de l’API reste dans le réseau virtuel. Pour ces scénarios, vous pouvez déployer Gestion des API dans le réseau virtuel de cluster. [Le niveau Premium de Gestion des API](https://aka.ms/apimpricing) prend en charge le déploiement de réseau virtuel. 

Il existe deux modes de [déploiement de Gestion des API dans un réseau virtuel](./api-management-using-with-vnet.md) : externe et interne. 

Si les consommateurs d’API ne résident pas dans le réseau virtuel du cluster, le mode externe (Fig. 4) doit être utilisé. Dans ce mode, la passerelle Gestion des API est injectée dans le réseau virtuel de cluster, mais accessible à partir de l’Internet public via un équilibreur de charge externe. Cela permet de masquer complètement le cluster tout en permettant aux clients externes d’utiliser les microservices. En outre, vous pouvez utiliser des fonctionnalités de mise en réseau Azure, telles que les groupes de sécurité réseau pour limiter le trafic réseau.

![Mode de réseau virtuel externe](./media/api-management-aks/vnet-external.png)

Si tous les consommateurs d’API résident dans le réseau virtuel du cluster, le mode interne (Fig. 5) peut être utilisé. Dans ce mode, la passerelle Gestion des API est injectée dans le réseau virtuel de cluster et accessible uniquement à partir de ce réseau virtuel via un équilibreur de charge interne. Il n’existe aucun moyen d’atteindre la passerelle Gestion des API ou le cluster AKS à partir de l’Internet public. 

![Mode de réseau virtuel interne](./media/api-management-aks/vnet-internal.png)

 Dans les deux cas, le cluster AKS n’est pas visible publiquement. Par rapport à l’option 2, le contrôleur d’entrée n’est peut-être pas nécessaire. Selon votre scénario et votre configuration, l’authentification peut toujours être nécessaire entre Gestion des API et vos microservices. Par exemple, si un maillage de service est adopté, il requiert toujours l’authentification TLS mutuelle. 

Avantages :
* Option la plus sécurisée, car le cluster AKS n’a pas de point de terminaison public
* Simplifie la configuration du cluster, car il n’a aucun point de terminaison public
* Possibilité de masquer Gestion des API et AKS à l’intérieur du réseau virtuel à l’aide du mode interne
* Possibilité de contrôler le trafic réseau à l’aide de fonctionnalités de mise en réseau Azure, telles que les groupes de sécurité réseau

Inconvénients :
* Augmente la complexité du déploiement et de la configuration de Gestion des API pour fonctionner à l’intérieur du réseau virtuel

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus, consultez [Concepts de réseau pour les applications dans AKS](../aks/concepts-network.md).
* Pour en savoir plus, consultez [Utilisation de Gestion des API avec des réseaux virtuels](./api-management-using-with-vnet.md)
