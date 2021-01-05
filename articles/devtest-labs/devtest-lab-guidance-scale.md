---
title: Scale up de votre infrastructure Azure DevTest Labs
description: Cet article fournit des conseils concernant la scalabilité de votre infrastructure Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.reviewer: christianreddington,anthdela,juselph
ms.openlocfilehash: 50bf08678a12a1a0499abd08c52a264d03f4a401
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85478788"
---
# <a name="scale-up-your-azure-devtest-labs-infrastructure"></a>Scale up de votre infrastructure Azure DevTest Labs
Avant d’implémenter DevTest Labs à l’échelle de l’entreprise, plusieurs points de décision clés sont à examiner. La compréhension de ces points de décision à un niveau hiérarchique élevé éclaire les décisions de conception à venir de l’organisation. Toutefois, ces points ne doivent pas retenir une organisation de démarrer une preuve de concept. Les trois domaines les plus importants en matière de planification initiale de scale-up sont :

- Le réseau et la sécurité
- La topologie de l’abonnement
- Les rôles et responsabilités

## <a name="networking-and-security"></a>Le réseau et la sécurité
La mise en réseau et la sécurité sont des pierres angulaires pour toutes les organisations. Alors qu’un déploiement à l’échelle de l’entreprise nécessite une analyse beaucoup plus approfondie, une preuve de concept réussie ne demande que peu de conditions. Portez votre attention sur les thèmes suivants :

- **Abonnement Azure** : pour déployer DevTest Labs, vous devez avoir accès à un abonnement Azure avec les droits appropriés de création des ressources. Il existe plusieurs façons d’accéder aux abonnements Azure, y compris un Accord Entreprise et le Paiement à l’utilisation. Pour plus d’informations sur l’accès à un abonnement Azure, consultez [Licences Azure pour l’entreprise](https://azure.microsoft.com/pricing/enterprise-agreement/).
- **Accès aux ressources locales** : certaines organisations ont besoin de donner un accès aux ressources locales à leurs ressources dans DevTest Labs. Une connexion sécurisée depuis votre environnement local vers Azure est nécessaire. Par conséquent, il est important que vous définissez/configuriez un VPN ou une connexion Express Route avant de commencer. Pour en savoir plus, consultez [Vue d’ensemble des Réseaux virtuels](../virtual-network/virtual-networks-overview.md).
- **Exigences de sécurité supplémentaires** : les autres exigences de sécurité (stratégies de l’ordinateur, accès à des adresses IP publiques, connexion à internet...) sont des scénarios qui nécessiteront peut-être d’être passés en revue avant d’implémenter une preuve de concept. 

## <a name="subscription-topology"></a>La topologie de l’abonnement
La topologie de l’abonnement est un élément de conception critique lors du déploiement de DevTest Labs dans l’entreprise. Il n’est cependant pas nécessaire de consolider toutes les décisions jusqu’à l’exécution d’une preuve de concept. Lorsque vous évaluez le nombre d’abonnements requis pour une implémentation dans l’entreprise, deux extrêmes sont possibles : 

- Un abonnement pour toute l’organisation
- Un abonnement par utilisateur

Nous allons mettre en avant les avantages de chaque approche.

### <a name="one-subscription"></a>Abonnement unique
Souvent, cette approche n’est pas facile à gérer pour une grande entreprise. Toutefois, la limitation du nombre d’abonnements offre les avantages suivants :

- **Prévisibilité** des coûts pour l’entreprise.  La budgétisation est beaucoup plus facile avec un seul abonnement, car toutes les ressources se trouvent dans un seul pool. Cette approche simplifie la prise de décision concernant le moment de mettre en place des mesures de contrôle des coût au cours du cycle de facturation.
- **La gestion** des machines virtuelles, des artefacts, des formules, de la configuration réseau, des autorisations, des stratégies, etc., est plus facile, car les mises à jour ne sont nécessaires que dans un seul abonnement et non pas dans plusieurs.
- **La mise en réseau** demande un effort beaucoup moins important s’agissant d’un abonnement unique pour les entreprises où la connectivité locale est obligatoire. Il est nécessaire de disposer de réseaux virtuels de connexions sur plusieurs abonnements (modèle hub-and-spoke), ce qui implique des abonnements supplémentaires, et donc plus de configuration, de gestion, d’espaces d’adressage IP, etc.
- **La collaboration entre équipes** est plus facile lorsque tout le monde travaille avec le même abonnement : par exemple, il est plus facile de réaffecter une machine virtuelle à un collègue, de partager des ressources avec l’équipe, etc.

### <a name="subscription-per-user"></a>Un abonnement par utilisateur
Un abonnement distinct par utilisateur fournit des opportunités similaires à l’autre extrême. Les avantages d’avoir plusieurs abonnements incluent :

- **Les quotas de mise à l’échelle Azure** ne vont pas entraver l’adoption. Par exemple, à ce jour, Azure autorise 200 comptes de stockage par abonnement. Il existe des quotas opérationnels pour la plupart des services Azure (nombre d’entre eux peuvent être personnalisés, mais certains non). Dans ce modèle d’un abonnement par utilisateur, il est très peu probable que la plupart des quotas soient atteints. Pour plus d’informations sur les quotas Azure actuels, consultez [Abonnement Azure et limites, quotas et contraintes du service](../azure-resource-manager/management/azure-subscription-service-limits.md).
- **Les rétrofacturations** à des développeurs individuels ou des groupes de développeurs sont facilitées, permettant aux organisations d’intégrer les coûts à l’aide de leur modèle actuel.
- **La propriété et les autorisations** relatives aux environnements DevTest Labs sont simples. Vous donnez aux développeurs un accès de niveau abonnement et ils sont entièrement responsables de tout, y compris la configuration du réseau, les stratégies de laboratoire et la gestion des machines virtuelles.

Dans l’entreprise, les contraintes aux extrêmes du spectre peuvent être importantes. Par conséquent, vous devrez peut-être configurer les abonnements de façon à vous trouver au milieu de ces extrêmes. En termes de meilleure pratique, l’objectif d’une organisation doit être d’utiliser un nombre d’abonnements aussi petit que possible tout en n’oubliant pas les fonctions de forçage qui augmentent le nombre total d’abonnements. Pour réitérer, la topologie d’abonnements est critique pour le déploiement en entreprise de DevTest Labs mais ne doit pas différer une preuve de concept. Vous trouverez des détails supplémentaires dans l’article [Gouvernance](devtest-lab-guidance-governance-policy-compliance.md) sur la façon de décider de la granularité d’abonnement et de laboratoire dans l’organisation.

## <a name="roles-and-responsibilities"></a>Les rôles et responsabilités
Une preuve de concept DevTest Labs comporte trois rôles principaux avec des responsabilités définies : propriétaire de l’abonnement, propriétaire DevTest Labs, utilisateur DevTest Labs et éventuellement un contributeur.

- **Propriétaire de l’abonnement** : le propriétaire de l’abonnement dispose des droits d’administrer un abonnement Azure, y compris l’affectation d’utilisateurs, la gestion des stratégies, la création et la gestion de la topologie de réseau, la demande d’augmentations de quota, etc. Pour plus d’informations, consultez [cet article](../role-based-access-control/rbac-and-directory-admin-roles.md).
- **Propriétaire DevTest Labs** : le propriétaire DevTest Labs dispose d’un accès administratif complet au laboratoire. Cette personne est responsable de l’ajout/de la suppression des utilisateurs, des paramètres des coûts de gestion, des paramètres de laboratoire généraux et d’autres tâches basées sur la machine virtuelle/l’artefact. Un propriétaire de laboratoire a également tous les droits d’un utilisateur DevTest Labs.
- **Utilisateur DevTest Labs** : l’utilisateur DevTest Labs peut créer et consommer les machines virtuelles dans le laboratoire. Ces personnes disposent de pouvoirs d’administration minimum sur les machines virtuelles qu’ils créent (démarrer/arrêter/supprimer/configurer leurs machines virtuelles). Les utilisateurs ne peuvent pas gérer les machines virtuelles d’autres utilisateurs.

## <a name="next-steps"></a>Étapes suivantes
Voir l’article suivant de cette série : [Orchestrer l’implémentation d’Azure DevTest Labs](devtest-lab-guidance-orchestrate-implementation.md)