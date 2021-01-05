---
title: Vue d’ensemble de Kubernetes avec Azure Arc
services: azure-arc
ms.service: azure-arc
ms.date: 05/19/2020
ms.topic: overview
author: mlearned
ms.author: mlearned
description: Cet article offre une vue d’ensemble de Kubernetes avec Azure Arc.
keywords: Kubernetes, Arc, Azure, conteneurs
ms.custom: references_regions
ms.openlocfilehash: 7e48ebf98f12e79cb154fb50d8e6dbdfaea1cd95
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92371305"
---
# <a name="what-is-azure-arc-enabled-kubernetes-preview"></a>Qu’est-ce que Kubernetes avec Azure Arc (préversion) ?

Vous pouvez attacher et configurer des clusters Kubernetes à l’intérieur ou à l’extérieur d’Azure en utilisant Kubernetes avec Azure Arc (préversion). Quand un cluster Kubernetes est attaché à Azure Arc, il s’affiche dans le portail Azure. Il reçoit alors un ID Azure Resource Manager et une identité managée. Les clusters sont attachés à des abonnements Azure standard, sont situés dans un groupe de ressources et peuvent recevoir des étiquettes comme n’importe quelle autre ressource Azure. 

Pour connecter un cluster Kubernetes à Azure, l’administrateur du cluster doit déployer des agents. Ces agents s’exécutent dans un espace de noms Kubernetes nommé `azure-arc` et représentent des déploiements Kubernetes standard. Les agents sont responsables de la connectivité à Azure, de la collecte des journaux et des métriques Azure Arc, et de la surveillance des demandes de configuration. 

Kubernetes avec Azure Arc prend en charge le protocole SSL standard pour sécuriser les données en transit. Par ailleurs, les données sont chiffrées au repos dans une base de données Azure Cosmos DB pour garantir leur confidentialité.
 
> [!NOTE]
> Kubernetes avec Azure Arc est en préversion. Nous ne la recommandons pas pour les charges de travail de production.

## <a name="supported-kubernetes-distributions"></a>Distributions Kubernetes prises en charge

Kubernetes avec Azure Arc est compatible avec n’importe quel cluster certifié CNCF (Cloud Native Computing Foundation) comme AKS-engine sur Azure, AKS-engine sur Azure Stack Hub, GKE, EKS et le cluster VMware vSphere.

Les fonctionnalités de Kubernetes avec Azure Arc ont été testées par l’équipe Arc sur les distributions suivantes :
* RedHat OpenShift 4.3
* Rancher RKE 1.0.8
* Canonical Charmed Kubernetes 1.18
* Moteur AKS
* Moteur AKS sur Azure Stack Hub
* AKS sur Azure Stack HCI
* Cluster API Provider Azure

## <a name="supported-scenarios"></a>Scénarios pris en charge 

Kubernetes avec Azure Arc prend en charge les scénarios suivants : 

* Connecter un cluster Kubernetes s’exécutant en dehors d’Azure pour l’inventaire, le regroupement et l’étiquetage.

* Déployer des applications et appliquer une configuration à l’aide de la gestion de la configuration basée sur GitOps. 

* Utiliser Azure Monitor pour conteneurs pour voir et superviser vos clusters. 

* Appliquer des stratégies à l’aide d’Azure Policy pour Kubernetes. 

[!INCLUDE [azure-lighthouse-supported-service](../../../includes/azure-lighthouse-supported-service.md)]

## <a name="supported-regions"></a>Régions prises en charge 

Kubernetes avec Azure Arc est actuellement pris en charge dans ces régions : 

* USA Est 
* Europe Ouest

## <a name="frequently-asked-questions"></a>Forum Aux Questions (FAQ)

* Quelle est la différence entre Azure Arc enabled Kubernetes et Azure Kubernetes Service (AKS) ?

    Azure Kubernetes Service (AKS) est l’offre de Kubernetes managé fournie par Azure. AKS simplifie le déploiement d’un cluster Kubernetes managé dans Azure. AKS permet de réduire la complexité et la surcharge opérationnelle de la gestion d’un cluster Kubernetes en déléguant une grande partie de cette responsabilité à Azure. Les maîtres Kubernetes sont gérés par Azure. Vous gérez uniquement les nœuds de l’agent.

    Azure Arc enabled Kubernetes vous permet de connecter des clusters Kubernetes à Azure pour étendre les fonctionnalités de gestion d’Azure telles qu’Azure Monitor et Azure Policy. C’est vous qui effectuez la maintenance du cluster Kubernetes sous-jacent.

* Ai-je besoin de connecter à Azure Arc mes clusters Azure Kubernetes Service qui s’exécutent sur Azure ?

    Non. Toutes les fonctionnalités d’Azure Arc enabled Kubernetes comme Azure Monitor et Azure Policy (Gatekeeper) sont disponibles en mode natif avec AKS, qui a déjà une représentation des ressources dans Azure.
    
* Dois-je connecter à Azure Arc mon cluster AKS exécuté sur Azure Stack HCI ? Qu’en est-il des clusters Kubernetes exécutés sur Azure Stack Hub ou Azure Stack Edge ?

    Oui, car la connexion de ces clusters à Azure Arc présente des avantages. Cela permet d’avoir une représentation des ressources pour ces clusters Kubernetes dans Azure Resource Manager. Grâce à cette représentation des ressources, des fonctionnalités telles que Cluster Configuration, Azure Monitor et Azure Policy (Gatekeeper) peuvent être étendues à ces clusters Kubernetes.

## <a name="next-steps"></a>Étapes suivantes

* [Connecter un cluster](./connect-cluster.md)
