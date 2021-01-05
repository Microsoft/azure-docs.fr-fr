---
title: Qu’est-ce qu’un contrôleur d’entrée Azure Application Gateway ?
description: Cet article présente ce qu’est un contrôleur d’entrée Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: article
ms.date: 06/10/2020
ms.author: caya
ms.openlocfilehash: 26f53a8f93d4d51ec8f8fd91051496a46670f432
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397346"
---
# <a name="what-is-application-gateway-ingress-controller"></a>Qu’est-ce qu’un contrôleur d’entrée Application Gateway ?
Le contrôleur d’entrée Application Gateway (AGIC) est une application Kubernetes, qui permet aux clients d’[Azure Kubernetes Service (AKS)](https://azure.microsoft.com/services/kubernetes-service/) de tirer parti de l’équilibreur de charge L7 [Application Gateway](https://azure.microsoft.com/services/application-gateway/) natif d’Azure pour exposer un logiciel cloud à Internet. AGIC surveille le cluster Kubernetes sur lequel il est hébergé et met à jour en permanence une Application Gateway, afin que les services sélectionnés soient exposés à Internet.

Le contrôleur d’entrée s’exécute dans son propre pod sur l’AKS du client. AGIC surveille les modifications dans un sous-ensemble de ressources Kubernetes. L’état du cluster AKS est converti en configuration spécifique à l’Application Gateway et appliqué à [Azure Resource Manager (ARM)](../azure-resource-manager/management/overview.md).

## <a name="benefits-of-application-gateway-ingress-controller"></a>Avantages du contrôleur d’entrée Application Gateway
AGIC permet d’éliminer la nécessité d’avoir un autre IP public/équilibreur de charge devant le cluster AKS et d’éviter plusieurs tronçons dans votre chemin d’accès aux données avant que les demandes atteignent le cluster AKS. Application Gateway communique directement avec les pods à l’aide de leur IP privé et ne nécessite pas de services NodePort ou KubeProxy. Cela offre également un meilleur niveau de performance à vos déploiements.

Le contrôleur d’entrée est pris en charge exclusivement par les références SKU Standard_v2 et WAF_v2, ce qui vous offre également des avantages en matière de mise à l’échelle automatique. Application Gateway peut réagir en réponse à une augmentation ou une diminution de la charge du trafic et mettre à l’échelle en conséquence, sans consommer de ressources de votre cluster AKS.

L’utilisation d’Application Gateway en plus d’AGIC permet également de protéger votre cluster AKS en fournissant une stratégie TLS et la fonctionnalité de pare-feu d’applications web (WAF).

![Azure Application Gateway + AKS](./media/application-gateway-ingress-controller-overview/architecture.png)

AGIC est configuré par le biais de la [ressource d’entrée](https://kubernetes.io/docs/user-guide/ingress/) Kubernetes, ainsi que du service et des déploiements/pods. Il offre un certain nombre de fonctionnalités, tirant parti de l’équilibreur de charge L7 Application Gateway natif d’Azure. Pour n’en nommer que quelques-unes :
  - Routage d’URL
  - Affinité basée sur les cookies
  - Arrêt TLS
  - TLS de bout en bout
  - Prise en charge des sites web publics, privés et hybrides
  - Pare-feu d’applications web intégré

## <a name="difference-between-helm-deployment-and-aks-add-on"></a>Différence entre le déploiement avec Helm et en tant que module complémentaire AKS
Il y a deux façons de déployer AGIC pour votre cluster AKS : par le biais de Helm ou en tant que module complémentaire AKS. Le principal avantage du déploiement d’AGIC en tant que module complémentaire AKS est que cette méthode est beaucoup plus simple que le déploiement avec Helm. Pour une nouvelle installation, vous pouvez déployer une nouvelle Application Gateway et un nouveau cluster AKS avec AGIC activé en tant que module complémentaire sur une seule ligne dans Azure CLI. De plus, le module complémentaire est un service entièrement managé qui offre d’autres avantages comme les mises à jour automatiques et une meilleure prise en charge. AGIC n’est pas pris en charge par AKS quand il est déployé via Helm, alors qu’il l’est quand il est déployé en tant que module complémentaire AKS. 

Le module complémentaire AGIC est toujours déployé en tant que pod dans le cluster AKS du client. Il y a toutefois quelques différences entre la version d’AGIC déployée avec Helm et celle déployée en tant que module complémentaire. Voici une liste des différences entre les deux versions : 
  - Les valeurs de déploiement avec Helm ne peuvent pas être modifiées sur le module complémentaire AKS :
    - `verbosityLevel` est défini sur 5 par défaut
    - `usePrivateIp` est défini sur false par défaut ; cette valeur peut être remplacée par l’[annotation use-private-ip](ingress-controller-annotations.md#use-private-ip)
    - `shared` n’est pas pris en charge sur le module complémentaire 
    - `reconcilePeriodSeconds` n’est pas pris en charge sur le module complémentaire
    - `armAuth.type` n’est pas pris en charge sur le module complémentaire
  - S’il a été déployé avec Helm, AGIC prend en charge ProhibitedTargets, ce qui signifie qu’AGIC peut configurer Application Gateway spécifiquement pour les clusters AKS sans perturber les autres serveurs back-end existants. Le module complémentaire AGIC n’assure pas cette prise en charge pour le moment. 
  - Étant donné que le module complémentaire AGIC est un service managé, les clients sont automatiquement mis à jour vers la dernière version du module complémentaire AGIC, ce qui n’est pas le cas dans un déploiement d’AGIC via Helm où le client doit mettre à jour AGIC manuellement. 

> [!NOTE]
> La méthode de déploiement d’AGIC en tant que module complémentaire AKS est actuellement disponible en préversion. Nous vous déconseillons d’exécuter des charges de travail de production sur des fonctionnalités encore en préversion. Par conséquent, si vous êtes curieux d’essayer cette méthode, nous vous recommandons de configurer un nouveau cluster réservé aux tests. 

Les tableaux suivants récapitulent les scénarios actuellement pris en charge avec la version d’AGIC déployée par le biais de Helm et celle déployée en tant que module complémentaire AKS. 

### <a name="aks-add-on-agic-single-aks-cluster"></a>AGIC déployé en tant que module complémentaire AKS (cluster AKS unique)
|                  |1 Application Gateway |2 Application Gateway ou plus |
|------------------|---------|--------|
|**1 AGIC**|Oui, cette méthode est prise en charge |Non, la prise en charge est dans notre backlog |
|**2 AGIC ou plus**|Non, 1 seul AGIC pris en charge/cluster |Non, 1 seul AGIC pris en charge/cluster |

### <a name="helm-deployed-agic-single-aks-cluster"></a>AGIC déployé avec Helm (cluster AKS unique)
|                  |1 Application Gateway |2 Application Gateway ou plus |
|------------------|---------|--------|
|**1 AGIC**|Oui, cette méthode est prise en charge |Non, la prise en charge est dans notre backlog |
|**2 AGIC ou plus**|Doit utiliser la fonctionnalité ProhibitedTarget partagée et vérifier les espaces de noms distincts |Oui, cette méthode est prise en charge |

### <a name="helm-deployed-agic-2-aks-clusters"></a>AGIC déployé avec Helm (2 clusters AKS ou plus)
|                  |1 Application Gateway |2 Application Gateway ou plus |
|------------------|---------|--------|
|**1 AGIC**|N/A |N/A |
|**2 AGIC ou plus**|Doit utiliser la fonctionnalité ProhibitedTarget partagée |N/A |

## <a name="next-steps"></a>Étapes suivantes
- [**Déploiement « greenfield » en tant que module complémentaire AKS**](tutorial-ingress-controller-add-on-new.md) : instructions pour installer AGIC en tant que module complémentaire AKS et Application Gateway sur une infrastructure vierge.
- [**Déploiement « brownfield » en tant que module complémentaire AKS**](tutorial-ingress-controller-add-on-existing.md) : installez le module complémentaire AGIC sur un cluster AKS avec une Application Gateway existante.
- [**Déploiement « greenfield » avec Helm**](ingress-controller-install-new.md) : installez AGIC par le biais de Helm, un nouveau cluster AKS et une nouvelle Application Gateway sur une infrastructure vierge.
- [**Déploiement « brownfield » avec Helm**](ingress-controller-install-existing.md) : déployez AGIC par le biais de Helm sur un cluster AKS existant et Application Gateway.