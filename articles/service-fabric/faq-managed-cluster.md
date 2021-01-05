---
title: Questions courantes sur les clusters managés Service Fabric
description: Foire aux questions sur les clusters managés Service Fabric, notamment les capacités, les cas d’usage et les scénarios courants.
ms.topic: troubleshooting
ms.author: pepogors
author: peterpogorski
ms.date: 09/28/2020
ms.custom: references_regions
ms.openlocfilehash: 4dc41d2c13c834657534971041440bb744cfca38
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92319823"
---
# <a name="service-fabric-managed-clusters-frequently-asked-questions"></a>Questions fréquentes (FAQ) sur les clusters Service Fabric managés

Les utilisateurs posent fréquemment des questions sur l’utilisation et les fonctionnalités des clusters managés Service Fabric. Ce document regroupe un grand nombre de ces questions fréquentes et leurs réponses.

## <a name="general"></a>Général

### <a name="what-are-service-fabric-managed-clusters"></a>Que sont les clusters managés Service Fabric ?

Les clusters managés Service Fabric sont une évolution du modèle de ressource de cluster Service Fabric qui est conçu pour faciliter le déploiement et la gestion des clusters. Un cluster managé Service Fabric utilise le modèle d’encapsulation Azure Resource Manager, de sorte qu’un utilisateur n’a besoin de définir et de déployer qu’une seule ressource de cluster par rapport aux nombreuses ressources indépendantes qu’il doit déployer aujourd’hui (groupe de machines virtuelles identiques, équilibreur de charge, IP, etc.).

### <a name="what-regions-are-supported-in-the-preview"></a>Quelles sont les régions prises en charge dans la préversion ?

Les régions prises en charge pour les clusters managés Service Fabric en préversion incluent `centraluseuap`, `eastus2euap`, `eastasia`, `northeurope`, `westcentralus` et `eastus2`.

### <a name="can-i-do-an-in-place-migration-of-my-existing-service-fabric-cluster-to-a-managed-cluster-resource"></a>Puis-je effectuer une migration sur place de mon cluster Service Fabric existant vers une ressource de cluster managé ?

À ce stade, vous devez créer une ressource de cluster Service Fabric pour utiliser le nouveau type de ressource de cluster managé Service Fabric.

### <a name="is-there-an-additional-cost-for-service-fabric-managed-clusters"></a>Les clusters managés Service Fabric engendrent-ils un coût supplémentaire ?

Non, aucun coût supplémentaire n’est associé à un cluster managé Service Fabric au-delà du coût des ressources de calcul, de stockage et de mise en réseau sous-jacentes requises pour le cluster.

### <a name="is-there-a-new-sla-introduced-by-the-service-fabric-managed-cluster-resource"></a>Existe-t-il un nouveau contrat SLA introduit par la ressource de cluster managé Service Fabric ?

Le contrat SLA ne change pas par rapport au modèle actuel de ressources Service Fabric.

### <a name="what-is-the-difference-between-a-basic-and-standard-sku-cluster"></a>Quelle est la différence entre les clusters des niveaux tarifaires De base et Standard ?

Un cluster de niveau tarifaire De base signifie que la plupart des configurations sont fournies par le fournisseur de ressources Service Fabric. Les clusters de niveau tarifaire De base sont destinés à être utilisés pour les tests et les environnements de préproduction. Un cluster de niveau tarifaire Standard permet aux utilisateurs de configurer le cluster en fonction de leurs besoins. Pour plus d’informations, consultez [SKU de cluster managé Service Fabric](./overview-managed-cluster.md#service-fabric-managed-cluster-skus).

## <a name="cluster-deployment-and-management"></a>Déploiement et gestion de clusters

### <a name="i-run-custom-script-extensions-on-my-virtual-machine-scale-set-can-i-continue-to-do-that-with-a-managed-service-fabric-resource"></a>J’exécute des extensions de script personnalisées sur mon groupe de machines virtuelles identiques. Puis-je continuer à le faire avec une ressource Service Fabric managée ?

Oui, vous pouvez toujours spécifier des extensions de machine virtuelle sur un type de nœud. Pour plus d’informations, consultez l’exemple d’extension de type Nœud.

### <a name="i-want-to-have-an-internal-only-load-balancer-is-that-possible"></a>Je souhaite avoir un équilibreur de charge interne uniquement, est-ce possible ?

Il n’est actuellement pas possible d’avoir un équilibreur de charge interne uniquement. Nous vous recommandons de verrouiller les règles de groupe de sécurité réseau pour bloquer tout trafic entrant/sortant indésirable.

### <a name="can-i-autoscale-my-cluster"></a>Puis-je mettre automatiquement mon cluster à l’échelle ? 
La mise à l’échelle automatique n’est pas disponible actuellement dans la préversion.

### <a name="can-i-deploy-my-cluster-across-availability-zones"></a>Puis-je déployer mon cluster sur plusieurs zones de disponibilité ? 
Les clusters dans plusieurs zones de disponibilité ne sont pas disponibles actuellement dans la préversion.

### <a name="can-i-select-between-automatic-and-manual-upgrades-for-my-cluster-runtime"></a>Puis-je choisir entre des mises à niveau automatiques et manuelles pour mon runtime de cluster ? 
Dans la préversion, toutes les mises à niveau du runtime sont effectuées automatiquement.

## <a name="applications"></a>Applications

### <a name="is-there-a-local-development-experience-for-service-fabric-managed-clusters"></a>Existe-t-il une expérience de développement locale pour les clusters managés Service Fabric ?

L’expérience de développement locale reste inchangée par rapport aux clusters Service Fabric existants. Pour plus d’informations sur l’expérience de développement locale, consultez [Créer une application .NET](./service-fabric-quickstart-dotnet.md).

### <a name="can-i-deploy-my-applications-as-an-azure-resource-manager-resource"></a>Puis-je déployer mes applications en tant que ressource Azure Resource Manager ?

Dans la préversion, vous ne pouvez pas déployer des applications en tant que ressource Azure Resource Manager. Les applications doivent être déployées en se connectant directement au cluster par le biais de PowerShell ou de l’interface CLI. Cette fonctionnalité sera ajoutée avant que les clusters managés Service Fabric soient mis en disponibilité générale.