---
title: Guide de référence des opérations de gestion des identités et des accès Azure Active Directory
description: Ce guide de référence des opérations décrit les vérifications et actions à entreprendre pour sécuriser les opérations de gestion des identités et des accès.
services: active-directory
author: martincoetzer
manager: daveba
tags: azuread
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: fundamentals
ms.date: 10/31/2019
ms.author: martinco
ms.openlocfilehash: 66bce573be5a31641bdff809b8e9a79b617a703a
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370999"
---
# <a name="azure-active-directory-identity-and-access-management-operations-reference-guide"></a>Guide de référence des opérations de gestion des identités et des accès Azure Active Directory

Cette section du [Guide de référence des opérations Azure AD](active-directory-ops-guide-intro.md) décrit les vérifications et actions à entreprendre pour sécuriser et gérer le cycle de vie des identités et leurs affectations.

> [!NOTE]
> Ces suggestions sont valables à la date de publication mais peuvent évoluer. Les organisations doivent évaluer en permanence leurs pratiques de gestion des identités à mesure que les produits et services Microsoft évoluent.

## <a name="key-operational-processes"></a>Processus opérationnels clés

### <a name="assign-owners-to-key-tasks"></a>Affecter les propriétaires à des tâches clés

La gestion d’Azure Active Directory nécessite l’exécution continue de tâches et de processus opérationnels clés qui ne font peut-être pas partie d’un projet de lancement. Il est toujours important de configurer ces tâches pour maintenir votre environnement. Les tâches clés et leurs propriétaires recommandés sont listés ci-après :

| Tâche | Propriétaire |
| :- | :- |
| Définir le processus de création d’abonnements Azure | Varie selon l’organisation |
| Déterminer qui obtient les licences Enterprise Mobility + Security | Équipe des opérations IAM |
| Décider qui obtient les licences Microsoft 365 | Équipe de productivité |
| Décider qui obtient les autres licences, par exemple Dynamics, Visual Studio Codespaces | Propriétaire de l’application |
| Attribuer des licences | Équipe des opérations IAM |
| Résoudre les problèmes et corriger les erreurs d’affectation de licence | Équipe des opérations IAM |
| Provisionner les identités pour les applications dans Azure AD | Équipe des opérations IAM |

Quand vous passerez votre liste en revue, vous devrez peut-être affecter un propriétaire à des tâches qui en sont dépourvues, ou modifier la propriété des tâches avec propriétaires qui ne sont pas conformes aux suggestions ci-dessus.

#### <a name="assigning-owners-recommended-reading"></a>Affectation aux propriétaires des lectures recommandées

- [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md)
- [Gouvernance dans Azure](../../governance/index.yml)

## <a name="on-premises-identity-synchronization"></a>Synchronisation des identités locales

### <a name="identify-and-resolve-synchronization-issues"></a>Identifier et résoudre les problèmes de synchronisation

Microsoft vous recommande de disposer d’une bonne base de référence et d’une bonne compréhension des problèmes de votre environnement local susceptibles d’entraîner des problèmes de synchronisation par rapport au cloud. Dans la mesure où les outils automatisés tels que [IdFix](/office365/enterprise/prepare-directory-attributes-for-synch-with-idfix) et [Azure AD Connect Health](../hybrid/whatis-azure-ad-connect.md#why-use-azure-ad-connect-health) peuvent générer un grand nombre de faux positifs, nous vous recommandons d’identifier les erreurs de synchronisation non traitées depuis plus de 100 jours pour nettoyer ces objets erronés. Les erreurs de synchronisation non résolues à long terme peuvent générer des incidents de support. La rubrique [Résolution des problèmes liés aux erreurs de synchronisation](../hybrid/tshoot-connect-sync-errors.md) présente une vue d’ensemble des différents types d’erreur de synchronisation, certains des scénarios à l’origine de ces erreurs ainsi que les solutions possibles.

### <a name="azure-ad-connect-sync-configuration"></a>Configuration de la synchronisation Azure AD Connect

Pour activer toutes les expériences utilisateur hybrides, l’état de la sécurité basée sur l’appareil ainsi que l’intégration à Azure AD, vous devez synchroniser les comptes d’utilisateur dont vos employés se servent pour se connecter à leurs postes de travail.

Si vous ne synchronisez pas la forêt à laquelle les utilisateurs se connectent, vous devez changer la synchronisation pour qu’elle provienne de la forêt appropriée.

#### <a name="synchronization-scope-and-object-filtering"></a>Étendue de synchronisation et filtrage d’objets

La suppression des compartiments connus d’objets dont la synchronisation n’est pas nécessaire présente les avantages opérationnels suivants :

- Moins de sources d’erreurs de synchronisation
- Cycles de synchronisation plus rapides
- Réduction des « déchets » en provenance de l’environnement local, par exemple la pollution de la liste d’adresses globale pour les comptes de service locaux qui ne sont pas pertinents dans le cloud

> [!NOTE]
> Si vous constatez que vous importez de nombreux objets qui ne sont pas exportés vers le cloud, vous devez filtrer par UO ou en fonction d’attributs spécifiques.

Voici des exemples d’objets à exclure :

- Comptes de service non utilisés pour les applications cloud
- Groupes qui ne sont pas censés être utilisés dans les scénarios cloud tels que ceux qui permettent d’octroyer un accès aux ressources
- Utilisateurs ou contacts qui sont des identités externes destinées à être représentées avec Azure AD B2B Collaboration
- Comptes d’ordinateur pour lesquels les employés ne sont pas censés accéder aux applications cloud, par exemple, les serveurs

> [!NOTE]
> Si plusieurs comptes sont provisionnés pour une même identité humaine dans le cadre d’une migration, d’une fusion ou d’une acquisition de domaine hérité, vous devez synchroniser uniquement le compte de l’utilisateur au jour le jour, par exemple l’objet dont il se sert pour se connecter à son ordinateur.

Dans l’idéal, vous devez trouver un équilibre entre la réduction du nombre d’objets à synchroniser et la complexité des règles. En règle générale, le [filtrage](../hybrid/how-to-connect-sync-configure-filtering.md) d’UO/de conteneur et un simple mappage d’attribut à l’attribut cloudFiltered constituent une combinaison de filtrage efficace.

> [!IMPORTANT]
> Si vous utilisez le filtrage de groupe en production, vous devez passer à une autre approche de filtrage.

#### <a name="sync-failover-or-disaster-recovery"></a>Basculement de la synchronisation ou reprise d’activité après sinistre

Azure AD Connect joue un rôle clé dans le processus d’approvisionnement. Si le serveur de synchronisation se déconnecte pour une raison quelconque, les changements apportés en local ne peuvent pas être mis à jour dans le cloud et peuvent entraîner des problèmes d’accès pour les utilisateurs. Il est donc important de définir une stratégie de basculement qui permet aux administrateurs de reprendre rapidement la synchronisation après la déconnexion du serveur de synchronisation. Ces stratégies peuvent être classées selon les catégories suivantes :

- **Déployer les serveurs Azure AD Connect en mode de préproduction** - Permet à un administrateur de « promouvoir » le serveur de préproduction à l’aide d’un simple changement de configuration.
- **Utiliser la virtualisation** - Si Azure AD Connect est déployé sur une machine virtuelle, les administrateurs peuvent tirer profit de la pile de virtualisation pour effectuer une migration dynamique ou pour redéployer rapidement la machine virtuelle et reprendre ainsi la synchronisation.

Si votre organisation ne dispose pas d’une stratégie de reprise d’activité après sinistre et de basculement pour Sync, vous ne devez pas hésiter à déployer Azure AD Connect en mode de préproduction. De même, en cas d’incompatibilité entre votre configuration de production et votre configuration de préproduction, vous devez redéfinir la base de référence du mode de préproduction d’Azure AD Connect pour qu’elle corresponde à la configuration de production, notamment les versions et configurations logicielles.

![Capture d’écran de la configuration du mode de préproduction d’Azure AD Connect](./media/active-directory-ops-guide/active-directory-ops-img1.png)

#### <a name="stay-current"></a>Rester à jour

Microsoft met régulièrement à jour Azure AD Connect. Restez à jour pour bénéficier des améliorations de performances, des résolutions de bogues et des nouvelles fonctionnalités apportées par chaque nouvelle version.

Si votre version d’Azure AD Connect a plus de six mois de retard, vous devez effectuer une mise à niveau vers la version la plus récente.

#### <a name="source-anchor"></a>Ancre source

L’utilisation de **ms-DS-consistencyguid** en tant qu’ [ancre source](../hybrid/plan-connect-design-concepts.md) facilite la migration des objets entre les forêts et les domaines, ce qui est courant dans la consolidation/le nettoyage, les fusions, les acquisitions et les scissions de domaines AD.

Si vous utilisez actuellement **ObjectGuid** en tant qu’ancre source, nous vous recommandons d’utiliser **ms-DS-ConsistencyGuid** .

#### <a name="custom-rules"></a>Règles personnalisées

Les règles personnalisées d’Azure AD Connect permettent de contrôler le flux d’attributs entre les objets locaux et les objets cloud. Toutefois, une utilisation intensive ou abusive des règles personnalisées peut présenter les risques suivants :

- Résolution des problèmes de complexité
- Dégradation des performances durant l’exécution d’opérations complexes sur des objets
- Probabilité plus élevée de divergence de configuration entre le serveur de production et le serveur de préproduction
- Surcharge supplémentaire durant la mise à niveau d’Azure AD Connect si des règles personnalisées sont créées avec une priorité supérieure à 100 (utilisées par les règles intégrées)

Si vous utilisez des règles trop complexes, vous devez examiner les raisons de cette complexité et rechercher des possibilités de simplification. De même, si vous avez créé des règles personnalisées avec une valeur de priorité supérieure à 100, vous devez les corriger pour qu’elles ne présentent aucun risque ou pour qu’elles n’entrent pas en conflit avec l’ensemble par défaut.

Voici des exemples d’utilisation abusive de règles personnalisées :

- **Compenser les données erronées dans l’annuaire** - Dans ce cas, il est recommandé de collaborer avec les propriétaires de l’équipe AD, de nettoyer les données de l’annuaire sous forme de tâche de correction, et de changer les processus pour éviter la réintroduction de données incorrectes.
- **Correction ponctuelle d’utilisateurs individuels** - Il est courant de trouver des règles qui sont confrontées à des valeurs hors norme, généralement en raison d’un problème lié à un utilisateur spécifique.
- **« CloudFiltering » excessivement compliqué** - Bien que la réduction du nombre d’objets soit une bonne pratique, il existe un risque de création d’une étendue de synchronisation compliquée basée sur de nombreuses règles de synchronisation. S’il existe une logique complexe pour inclure/exclure des objets au-delà du filtrage d’UO, il est recommandé de traiter cette logique en dehors de la synchronisation et d’étiqueter les objets à l’aide d’un simple attribut « cloudFiltered » pouvant être transmis avec une simple règle de synchronisation.

#### <a name="azure-ad-connect-configuration-documenter"></a>Documentation de configuration Azure AD Connect

L’outil de [documentation de configuration Azure AD Connect](https://github.com/Microsoft/AADConnectConfigDocumenter) vous permet de générer la documentation d’une installation Azure AD Connect pour mieux comprendre la configuration de la synchronisation, renforcer le sentiment de fiabilité et savoir ce qui a changé une fois que vous avez appliqué une nouvelle build ou configuration d’Azure AD Connect, ou que vous avez ajouté ou mis à jour des règles de synchronisation personnalisées. Les fonctionnalités actuelles de l’outil sont les suivantes :

- Documentation sur la configuration complète de la synchronisation Azure AD Connect.
- Documentation sur les changements apportés à la configuration de deux serveurs de synchronisation Azure AD Connect ou sur les changements apportés à une base de référence de configuration donnée.
- Génération d’un script de déploiement PowerShell pour migrer les différences ou personnalisations des règles de synchronisation d’un serveur à un autre.

## <a name="assignment-to-apps-and-resources"></a>Affectation aux applications et ressources

### <a name="group-based-licensing-for-microsoft-cloud-services"></a>Licences basées sur les groupes pour les services de cloud computing Microsoft

Azure Active Directory rationalise la gestion des licences via les [licences basées sur les groupes](./active-directory-licensing-whatis-azure-portal.md) pour les services de cloud computing Microsoft. Ainsi, IAM fournit l’infrastructure de groupe et la gestion déléguée de ces groupes aux équipes appropriées dans les organisations. Il existe plusieurs façons de configurer l’appartenance aux groupes dans Azure AD, notamment :

- **Synchronisée à partir de l’environnement local** - Les groupes peuvent provenir d’annuaires locaux, ce qui convient parfaitement aux organisations qui disposent de processus de gestion de groupes établis pouvant être étendus pour affecter des licences dans Microsoft 365.

- **Basée sur des attributs/dynamique** - Les groupes peuvent être créés dans le cloud en fonction d’une expression basée sur des attributs utilisateur, par exemple Service est égal à « ventes ». Azure AD conserve les membres du groupe en cohérence avec l’expression définie. L’utilisation de ce genre de groupe pour l’affectation de licence permet une affectation de licence basée sur des attributs, ce qui convient parfaitement aux organisations ayant des données de haute qualité dans leur annuaire.

- **Propriété déléguée** - Des groupes peuvent être créés dans le cloud et être désignés en tant que propriétaires. Ainsi, vous pouvez autonomiser les propriétaires d’entreprise, par exemple l’équipe de collaboration ou l’équipe du décisionnel, en leur permettant de définir quelles sont les personnes qui disposent d’un accès.

Si vous utilisez un processus manuel pour affecter des licences et des composants à des utilisateurs, nous vous recommandons d’implémenter les licences basées sur les groupes. Si votre processus actuel ne permet pas de superviser les erreurs de licence ou les éléments affectés par rapport aux éléments disponibles, vous devez définir les améliorations à apporter au processus pour résoudre les erreurs de licence et superviser l’affectation des licences.

Un autre aspect de la gestion des licences concerne la définition des plans de service (composants de la licence) à activer selon les fonctions de tâche dans l’organisation. Si vous octroyez l’accès à des plans de service qui ne sont pas nécessaires, les utilisateurs risquent de voir dans le portail Office des outils auxquels ils n’ont pas été formés ou qu’ils ne doivent pas utiliser. Cela peut augmenter le volume de travail du support technique, entraîner un provisionnement inutile, et mettre en péril la conformité et la gouvernance, par exemple si vous provisionnez OneDrive Entreprise pour des personnes qui ne sont peut-être pas autorisées à partager du contenu.

Utilisez les instructions suivantes afin de définir des plans de service pour les utilisateurs :

- Les administrateurs doivent définir des « bundles » de plans de service à proposer aux utilisateurs en fonction de leur rôle, par exemple pour distinguer le personnel de bureau du personnel d’entretien.
- Créez des groupes par cluster, et affectez la licence avec le plan de service.
- Vous pouvez éventuellement définir un attribut pour contenir les packages des utilisateurs.

> [!IMPORTANT]
> La gestion des licences basées sur les groupes dans Azure AD introduit le concept d’utilisateurs en état d’erreur d’affectation de licence. Si vous remarquez des erreurs de licence, vous devez immédiatement [identifier et résoudre](../enterprise-users/licensing-groups-resolve-problems.md) les problèmes d’affectation de licence.

![Capture d’écran d’un écran d’ordinateur Description générée automatiquement](./media/active-directory-ops-guide/active-directory-ops-img2.png)

#### <a name="lifecycle-management"></a>Gestion du cycle de vie

Si vous utilisez un outil tel que [Microsoft Identity Manager](/microsoft-identity-manager/) ou un système tiers, qui repose sur une infrastructure locale, nous vous recommandons de déplacer l’affectation de l’outil existant, d’implémenter des licences basées sur les groupes et de définir une gestion du cycle de vie basée sur les [groupes](../enterprise-users/licensing-group-advanced.md#use-group-based-licensing-with-dynamic-groups). De même, si votre processus existant ne prend pas en compte les nouveaux employés ou les employés qui quittent l’organisation, vous devez déployer des licences basées sur les groupes en fonction des groupes dynamiques, et définir un cycle de vie d’appartenance aux groupes. Enfin, si les licences basées sur les groupes sont déployées sur des groupes locaux dépourvus de gestion du cycle de vie, utilisez des groupes cloud pour activer les fonctionnalités telles que la propriété déléguée ou l’appartenance dynamique basée sur les attributs.

### <a name="assignment-of-apps-with-all-users-group"></a>Affectation d’applications avec le groupe « Tous les utilisateurs »

Les propriétaires de ressources peuvent penser que le groupe **Tous les utilisateurs** contient uniquement les **Employés d’entreprise** , alors qu’il peut contenir en fait les **Employés d’entreprise** et les **Invités** . Vous devez donc être particulièrement vigilant quand vous utilisez le groupe **Tous les utilisateurs** pour l’affectation d’applications et l’octroi d’un accès aux ressources telles que du contenu ou des applications SharePoint.

> [!IMPORTANT]
> Si le groupe **Tous les utilisateurs** est activé et utilisé pour les stratégies d’accès conditionnel, l’affectation d’applications ou de ressources, veillez à [sécuriser le groupe](../external-identities/use-dynamic-groups.md) si vous ne souhaitez pas qu’il inclue des utilisateurs invités. De plus, vous devez corriger vos affectations de licences en les créant et en les affectant aux groupes qui contiennent uniquement des **Employés d’entreprise** . En revanche, si vous constatez que le groupe **Tous les utilisateurs** est activé, mais qu’il n’est pas utilisé pour octroyer l’accès aux ressources, vérifiez que les recommandations de votre organisation précisent bien de se servir de ce groupe (qui inclut les **Employés de l’entreprise** et les **Invités** ) de manière intentionnelle.

### <a name="automated-user-provisioning-to-apps"></a>Provisionnement automatisé des utilisateurs pour les applications

Le [provisionnement automatisé des utilisateurs](../app-provisioning/user-provisioning.md) pour les applications constitue le meilleur moyen de créer un provisionnement, un déprovisionnement et un cycle de vie cohérents des identités sur plusieurs systèmes.

Si vous provisionnez des applications de manière ad hoc, ou si vous utilisez des éléments tels que des fichiers CSV, JIT ou une solution locale ne prenant pas en charge la gestion du cycle de vie, nous vous recommandons d’[implémenter le provisionnement d’applications](../app-provisioning/user-provisioning.md#how-do-i-set-up-automatic-provisioning-to-an-application) avec Azure AD pour les applications prises en charge, et de définir un modèle cohérent pour les applications qui ne sont pas encore prises en charge par Azure AD.

![Service de provisionnement Azure AD](./media/active-directory-ops-guide/active-directory-ops-img3.png)

### <a name="azure-ad-connect-delta-sync-cycle-baseline"></a>Base de référence du cycle de synchronisation delta d’Azure AD Connect

Il est important de bien évaluer l’ampleur des changements apportés à votre organisation, et de vérifier si la synchronisation ne prend pas trop de temps pour pouvoir prévoir son délai.

La fréquence de la [synchronisation delta par défaut](../hybrid/how-to-connect-sync-feature-scheduler.md) est de 30 minutes. Si la synchronisation delta prend régulièrement plus de 30 minutes, ou s’il existe des écarts importants entre les performances de synchronisation delta de la préproduction et de la production, recherchez et passez en revue les [facteurs qui influent sur les performances d’Azure AD Connect](../hybrid/plan-connect-performance-factors.md).

#### <a name="azure-ad-connect-troubleshooting-recommended-reading"></a>Lecture recommandée pour la résolution des problèmes d’Azure AD Connect

- [Préparation des attributs d’annuaire pour la synchronisation avec Microsoft 365 à l’aide de l’outil IdFix](/office365/enterprise/prepare-directory-attributes-for-synch-with-idfix)
- [Azure AD Connect : Résolution des problèmes liés aux erreurs de synchronisation](../hybrid/tshoot-connect-sync-errors.md)

## <a name="summary"></a>Résumé

Une infrastructure d’identité sécurisée comporte cinq aspects. Cette liste va vous permettre de trouver et de prendre rapidement les mesures nécessaires pour sécuriser et gérer le cycle de vie des identités et de leurs droits dans votre organisation.

- Affectez des propriétaires aux tâches clés.
- Recherchez et résolvez les problèmes de synchronisation.
- Définissez une stratégie de basculement pour la reprise d’activité après sinistre.
- Rationalisez la gestion des licences et l’affectation d’applications.
- Automatisez le provisionnement des utilisateurs pour les applications.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment bien démarrer avec les [vérifications et actions de la gestion de l’authentification](active-directory-ops-guide-auth.md).