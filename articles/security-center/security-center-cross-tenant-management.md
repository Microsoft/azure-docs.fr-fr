---
title: Gestion multilocataire dans Azure Security Center | Microsoft Docs
description: Découvrez comment configurer la gestion multilocataire pour gérer la posture de sécurité de plusieurs locataires dans Security Center à l’aide de la gestion déléguée des ressources Azure.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 7d51291a-4b00-4e68-b872-0808b60e6d9c
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/11/2019
ms.author: memildin
ms.openlocfilehash: d76ecc55c2e156199b2db789721946fd1f36de27
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922912"
---
# <a name="cross-tenant-management-in-security-center"></a>Gestion multilocataire dans Security Center

La gestion multilocataire permet d’afficher et de gérer la posture de sécurité de plusieurs locataires dans Security Center, en tirant parti de la [gestion déléguée des ressources Azure](../lighthouse/concepts/azure-delegated-resource-management.md). Gérez efficacement plusieurs locataires à partir d’une même vue, sans avoir à vous connecter à l’annuaire de chaque locataire.

- Les fournisseurs de services peuvent gérer la posture de sécurité des ressources pour plusieurs clients, à partir de leur propre locataire.

- Les équipes de sécurité qui comprennent plusieurs locataires peuvent afficher et gérer leur posture de sécurité à partir d’un même emplacement.

## <a name="set-up-cross-tenant-management"></a>Configurer la gestion multilocataire

La gestion des ressources déléguées Azure est l’un des principaux composants d’Azure Lighthouse. Configurez la gestion multilocataire en déléguant à votre propre locataire l’accès aux ressources des locataires managés, en utilisant ces instructions de la documentation Azure Lighthouse : [Gestion des ressources déléguées Azure](../lighthouse/concepts/azure-delegated-resource-management.md).


## <a name="how-does-cross-tenant-management-work-in-security-center"></a>Fonctionnement de la gestion multilocataire dans Security Center

Vous pouvez examiner et gérer les abonnements de plusieurs locataires de la même façon que vous géreriez plusieurs abonnements dans un seul locataire.

Dans la barre de menus supérieure, cliquez sur l’icône de filtre, puis sélectionnez les abonnements de l’annuaire de chaque client que vous souhaitez voir.

  ![Filtrer les locataires](./media/security-center-cross-tenant-management/cross-tenant-filter.png)

Les vues et les actions sont plus ou moins identiques. Voici quelques exemples :

- **Gérer les stratégies de sécurité** : Dans une vue, vous pouvez gérer la posture de sécurité de nombreuses ressources à l’aide de [stratégies](tutorial-security-policy.md), appliquer les recommandations de sécurité, ainsi que collecter et gérer les données liées à la sécurité.
- **Améliorer le niveau de sécurité et l’état de conformité** : La visibilité multilocataire vous permet de voir la posture de sécurité globale de tous vos locataires, et ainsi, de déterminer où et comment améliorer le [degré de sécurisation](secure-score-security-controls.md) et la [posture de conformité](security-center-compliance-dashboard.md) pour chacun d’entre eux.
- **Appliquer les recommandations** : Vous pouvez surveiller et appliquer une [recommandation](security-center-recommendations.md) à toutes les ressources des différents locataires en même temps. Ensuite, vous pouvez immédiatement vous attaquer aux vulnérabilités qui présentent le risque le plus élevé parmi tous les locataires.
- **Gérer les alertes** : Détectez les [alertes](security-center-alerts-overview.md) sur les différents locataires. Appliquez les recommandations concernant les ressources non conformes à l’aide des [étapes de correction](security-center-managing-and-responding-alerts.md).

- **Gérer les fonctionnalités avancées de défense cloud et bien plus encore** : Gérez les divers services de protection contre les menaces et de protection, comme l’[accès juste-à-temps (JIT) aux machines virtuelles](security-center-just-in-time.md), le [renforcement du réseau adaptatif](security-center-adaptive-network-hardening.md), les [contrôles d’application adaptatifs](security-center-adaptive-application.md), etc.
 
## <a name="next-steps"></a>Étapes suivantes
Cet article explique le fonctionnement de la gestion multilocataire dans Security Center. Pour plus d’informations sur le Centre de sécurité, consultez les rubriques suivantes :

* [Renforcer votre posture de sécurité avec Azure Security Center](security-center-monitoring.md) – Découvrez comment superviser l’intégrité de vos ressources Azure.
* [FAQ Azure Security Center](faq-general.md) : forum aux questions concernant l’utilisation de ce service.
* [Découvrez Azure Lighthouse dans les scénarios de grandes entreprises](../lighthouse/concepts/enterprise.md) – Découvrez en quoi Azure Lighthouse peut simplifier la gestion inter-locataire au sein d’une grande entreprise qui utilise plusieurs locataires Azure AD.