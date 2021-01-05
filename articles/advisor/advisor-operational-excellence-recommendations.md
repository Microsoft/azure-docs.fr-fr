---
title: Améliorer l’excellence opérationnelle avec Advisor
description: Utilisez Azure Advisor pour améliorer et atteindre l’excellence opérationnelle de vos abonnements Azure.
ms.topic: article
ms.date: 10/24/2019
ms.openlocfilehash: 63e88129a7418e82ea13429c33d8735e96616476
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92122617"
---
# <a name="achieve-operational-excellence-by-using-azure-advisor"></a>Atteindre l’excellence opérationnelle à l’aide d’Azure Advisor

Les recommandations d’Azure Advisor en matière d’excellence opérationnelle peuvent vous aider dans les domaines suivants : 
- Efficacité des processus et des workflows.
- Simplicité de gestion des ressources.
- Bonnes pratiques de déploiement. 

Vous trouverez ces recommandations sous l’onglet **Excellence opérationnelle** du tableau de bord Advisor.

## <a name="create-azure-service-health-alerts-to-be-notified-when-azure-problems-affect-you"></a>Créer des alertes Azure Service Health pour être averti des problèmes Azure qui vous concernent

Nous vous recommandons de configurer des alertes Azure Service Health pour recevoir une notification quand des problèmes Azure vous concernent. [Azure Service Health](https://azure.microsoft.com/features/service-health/) est un service gratuit qui offre des conseils et un support personnalisés lorsque vous êtes concerné par un problème de service Azure. Advisor identifie les abonnements qui n’ont pas d’alertes configurées et recommande de les configurer.


## <a name="design-your-storage-accounts-to-prevent-reaching-the-maximum-subscription-limit"></a>Créer vos comptes de stockage de manière à éviter d’atteindre le nombre maximum d’abonnements

Une région Azure peut prendre en charge un maximum de 250 comptes de stockage par abonnement. Si vous atteignez cette limite, vous ne pourrez pas créer d’autres comptes de stockage dans cette combinaison abonnement/région. Advisor vérifie vos abonnements, puis fournit des recommandations pour vous aider à créer moins de comptes de stockage pour une combinaison abonnement/région qui est proche du nombre limite.

## <a name="ensure-you-have-access-to-azure-cloud-experts-when-you-need-it"></a>Assurez-vous d’avoir accès à des experts du cloud Azure chaque fois que vous en avez besoin

Quand vous exécutez une charge de travail critique pour l’entreprise, il est essentiel d’avoir accès au support technique en cas de besoin. Advisor identifie les éventuels abonnements stratégiques dont le plan de support ne prévoit aucun support technique. Il recommande d’effectuer une mise à niveau pour choisir une option qui inclut un support technique.

## <a name="delete-and-re-create-your-pool-to-remove-a-deprecated-internal-component"></a>Supprimer et recréer votre pool pour retirer un composant interne déprécié

Si votre pool utilise un composant interne déprécié, supprimez et recréez le pool pour améliorer la stabilité et les performances.

## <a name="repair-invalid-log-alert-rules"></a>Réparer les règles d’alerte de journal invalides

Azure Advisor détecte les règles d’alerte dont la section des conditions contient des requêtes non valides. Vous pouvez créer des règles d’alerte de journal dans Azure Monitor et les utiliser pour exécuter des requêtes d’analytique à des intervalles spécifiés. Les résultats de la requête déterminent si une alerte doit être déclenchée. Il arrive que des requêtes d’analytique deviennent non valides au fil du temps en raison de modifications effectuées dans les ressources, les tables ou les commandes référencées. Advisor vous recommande alors de corriger la requête dans la règle d’alerte pour éviter qu’elle ne se désactive automatiquement, et garantir ainsi une couverture de supervision complète de vos ressources dans Azure. [Découvrez-en plus sur la résolution des problèmes liés aux règles d’alerte.](../azure-monitor/platform/alerts-troubleshoot-log.md)

## <a name="use-azure-policy-recommendations"></a>Utiliser les recommandations Azure Policy

Azure Policy est un service Azure que vous pouvez utiliser pour créer, affecter et gérer des stratégies. Ces stratégies appliquent des règles et des effets sur vos ressources. Les recommandations Azure Policy suivantes peuvent vous aider à atteindre l’excellence opérationnelle : 

**Gérer les étiquettes.** Cette stratégie ajoute ou remplace la balise spécifiée et la valeur lors de la création ou de la mise à jour d’une ressource. Vous pouvez corriger des ressources existantes en déclenchant une tâche de correction. Cette stratégie ne modifie pas les étiquettes sur les groupes de ressources.

**Appliquer les exigences de conformité géographique.** Cette stratégie vous permet de restreindre les emplacements que votre organisation peut spécifier lors du déploiement de ressources. 

**Spécifier les références SKU de machine virtuelle autorisées pour les déploiements.** Cette stratégie vous permet de spécifier un ensemble de références de machine virtuelle que votre organisation peut déployer.

**Appliquer *Auditer les machines virtuelles qui n’utilisent pas de disques managés* .**

**Activer *Hériter une étiquette du groupe de ressources* .** Cette stratégie ajoute ou remplace la balise spécifiée et la valeur du groupe de ressources parent lors de la création ou de la mise à jour d’une ressource. Vous pouvez corriger des ressources existantes en déclenchant une tâche de correction.

Advisor recommande quelques stratégies Azure individuelles qui aident les clients à atteindre l’excellence opérationnelle en adoptant les meilleures pratiques. Si un client décide d’attribuer une stratégie recommandée, nous supprimons la recommandation. Si le client décide de supprimer la stratégie ultérieurement, Advisor continue de supprimer la recommandation, car nous interprétons sa suppression comme un signal fort de ce qui suit :

1.  Le client a supprimé la stratégie parce que, malgré la recommandation d’Advisor, elle ne s’applique pas à son cas d’usage spécifique. 
2.  Après avoir attribué et supprimé la stratégie, le client la connaît bien et peut l’attribuer ou la supprimer à nouveau si nécessaire, sans avoir besoin d’assistance, si elle devient pertinente pour son cas d’usage. Si le client trouve qu’il est dans son intérêt d’attribuer à nouveau la même stratégie, il peut le faire dans Azure Policy sans que cela nécessite de recommandation dans Advisor. Notez que cette logique s’applique spécifiquement à la recommandation de stratégie dans la catégorie Excellence opérationnelle. Ces règles ne s’appliquent pas aux recommandations de sécurité.  


## <a name="no-validation-environment-enabled"></a>Aucun environnement de validation activé
Azure Advisor détermine que vous n’avez pas d’environnement de validation activé dans l’abonnement actuel. Lors de la création de vos pools d’hôtes, vous avez sélectionné \"Non\" pour \"Environnement de validation\" sous l’onglet Propriétés. Le fait d’avoir au moins un pool d’hôtes avec un environnement de validation activé garantit la continuité des activités via les déploiements du service Windows Virtual Desktop avec une détection précoce des problèmes potentiels. [En savoir plus](../virtual-desktop/create-validation-host-pool.md)

## <a name="ensure-production-non-validation-environment-to-benefit-from-stable-functionality"></a>Garantir un environnement de production (et non de validation) pour bénéficier de fonctionnalités stables
Azure Advisor détecte que l’environnement de validation est activé pour un trop grand nombre de vos pools d’hôtes. Pour que les environnements de validation remplissent au mieux leur fonction, vous devez avoir au moins un, mais jamais plus de la moitié de vos pools d’hôtes dans l’environnement de validation. En ayant un équilibre parfait entre vos pools d’hôtes avec l’environnement de validation activé et ceux pour lesquels il est désactivé, vous pourrez tirer le meilleur parti des avantages des déploiements en plusieurs phases proposés par Windows Virtual Desktop avec certaines mises à jour. Pour résoudre ce problème, ouvrez les propriétés de votre pool d’hôtes et sélectionnez \"Non\" à côté du paramètre \"Environnement de validation\".

## <a name="enable-traffic-analytics-to-view-insights-into-traffic-patterns-across-azure-resources"></a>Activer Traffic Analytics pour obtenir des insights sur les modèles de trafic des ressources Azure
Traffic Analytics est une solution cloud qui offre une visibilité sur l’activité des utilisateurs et des applications dans Azure. Traffic Analytics analyse les journaux de flux du groupe de sécurité réseau Network Watcher pour fournir des insights sur le flux de trafic. Avec Traffic Analytics, vous pouvez afficher les éléments du réseau qui utilisent le plus de bande passante lors des déploiements Azure et non Azure, détecter les ports ouverts, les protocoles et les flux malveillants de votre environnement, et optimiser le déploiement de votre réseau pour de meilleures performances. Vous pouvez traiter les journaux de flux toutes les 10 ou 60 minutes, ce qui vous permet d’analyser plus rapidement votre trafic. Il est recommandé d’activer Traffic Analytics pour vos ressources Azure. 


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les recommandations d’Advisor, consultez les ressources suivantes :
* [Présentation du conseiller](advisor-overview.md)
* [Prise en main](advisor-get-started.md)
* [Recommandations d’Advisor en matière de coûts](advisor-cost-recommendations.md)
* [Recommandations d’Advisor en matière de performances](advisor-performance-recommendations.md)
* [Recommandations d’Advisor en matière de fiabilité](advisor-high-availability-recommendations.md)
* [Recommandations d’Advisor en matière de sécurité](advisor-security-recommendations.md)
* [API REST Advisor](/rest/api/advisor/)
