---
title: Déployer Privileged Identity Management (PIM) - Azure AD | Microsoft Docs
description: Décrit comment planifier le déploiement d’Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: conceptual
ms.workload: identity
ms.subservice: pim
ms.date: 08/27/2020
ms.author: curtand
ms.custom: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8c9ba7217dfc167a06a1fea389cfc40a5e1251ca
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92367803"
---
# <a name="deploy-azure-ad-privileged-identity-management-pim"></a>Déployer Azure AD Privileged Identity Management (PIM)

Cet article est un guide pas à pas expliquant comment planifier le déploiement de Privileged Identity Management (PIM) dans votre organisation Azure Active Directory (Azure AD). Vous réaffecterez les utilisateurs ayant des rôles à privilèges élevés à des rôles intégrés ou personnalisés moins puissants dans la mesure du possible et prévoirez des attributions de rôles juste-à-temps pour vos rôles les plus privilégiés. Dans cet article, nous faisons des recommandations tant pour la planification du déploiement que pour son implémentation.

> [!TIP]
> Tout au long de cet article, vous verrez des éléments marqués comme :
>
> :heavy_check_mark: **Microsoft recommande**
>
> Il s’agit de suggestions générales et vous devez uniquement les implémenter si elles s’appliquent aux besoins spécifiques de votre entreprise.

## <a name="licensing-requirements"></a>Exigences en termes de licence

Pour utiliser Privileged Identity Management, votre répertoire doit avoir l’une des licences payantes ou d’essai gratuit suivantes. Pour plus d’informations, consultez [Exigences relatives aux licences pour l’utilisation de Privileged Identity Management](subscription-requirements.md).

- Azure AD Premium P2
- Enterprise Mobility + Security (EMS) E5
- Microsoft 365 Education A5
- Microsoft 365 Entreprise E5

## <a name="how-pim-works"></a>Fonctionnement de PIM

Cette section passe en revue, à des fins de planification, les parties pertinentes du processus de Privileged Identity Management. Pour plus d’informations, consultez [Qu’est-ce qu’Azure AD Privileged Identity Management ?](pim-configure.md)

1. Commencez à utiliser Privileged Identity Management pour que les utilisateurs puissent bénéficier de rôles privilégiés.
1. Quand un utilisateur éligible doit utiliser son rôle privilégié, il active le rôle à l’aide de Privileged Identity Management.
1. L’utilisateur peut être invité à spécifier les paramètres suivants :

    - Utiliser l'authentification multifacteur
    - Demander l’approbation pour l’activation
    - Fournir une raison professionnelle pour l’activation

1. Une fois que l’utilisateur a activé son rôle, il dispose des autorisations de rôle pour une durée définie.
1. Les administrateurs peuvent afficher un historique de toutes les activités de Privileged Identity Management dans le journal d’audit. Ils peuvent également renforcer la sécurité de leurs organisations Azure AD et respecter les exigences en matière de conformité à l’aide de fonctionnalités de Privileged Identity Management telles que les révisions d’accès et les alertes.

## <a name="roles-that-can-be-managed-by-pim"></a>Rôles qui peuvent être gérés par PIM

Les **rôles Azure AD** se trouvent tous dans Azure Active Directory (par exemple, Administrateur général, Administrateur Exchange et Administrateur de la sécurité). Vous trouverez plus d’informations sur les rôles et leurs fonctionnalités dans [Autorisations des rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md). Afin d’obtenir de l’aide pour déterminer les rôles à affecter à vos administrateurs, consultez [Rôles moins privilégiés par tâche](../roles/delegate-by-task.md).

Les **rôles Azure** sont liés à une ressource, un groupe de ressources, un abonnement ou un groupe d’administration Azure. Vous pouvez utiliser PIM pour fournir un accès juste-à-temps aux rôles Azure intégrés comme Propriétaire, Administrateur de l’accès utilisateur et Contributeur, ainsi qu’à des [rôles personnalisés](../../role-based-access-control/custom-roles.md). Pour plus d’informations sur les rôles Azure, consultez [Contrôle d’accès en fonction du rôle Azure](../../role-based-access-control/overview.md).

Pour plus d’informations, consultez [Rôles que vous ne pouvez pas gérer dans Privileged Identity Management](pim-roles.md).

## <a name="deployment-plan"></a>Plan de déploiement

Avant de déployer Privileged Identity Management dans votre organisation, suivez les instructions et comprenez les concepts de cette section pour vous aider à créer un plan adapté aux besoins de votre organisation en matière d’identités privilégiées.

### <a name="identify-your-stakeholders"></a>Identifier les participants

La section suivante vous permet d’identifier toutes les parties prenantes impliquées dans le projet et qui doivent l’approuver, le réviser ou se tenir informés de celui-ci. Elle inclut des tableaux distincts pour le déploiement de PIM pour les rôles Azure AD et les rôles Azure. Ajoutez des participants au tableau suivant en fonction de votre organisation.

- SO = Valider ce projet
- R = Réviser ce projet et fournir une entrée
- I = Se tenir informé de ce projet

#### <a name="stakeholders-privileged-identity-management-for-azure-ad-roles"></a>Participants : Privileged Identity Management pour les rôles Azure AD

| Nom | Role | Action |
| --- | --- | --- |
| Nom et e-mail | **Architecte de l’identité ou administrateur général Azure**<br/>Représentant de l’équipe de gestion des identités responsable de la définition de l’alignement de cette modification sur l’infrastructure principale de gestion des identités dans votre organisation. | SO/R/I |
| Nom et e-mail | **Propriétaire de service / Supérieur hiérarchique**<br/>Représentant des propriétaires du service informatique d’un service ou d’un groupe de services. Il est essentiel à la prise de décisions et au déploiement de Privileged Identity Management pour son équipe. | SO/R/I |
| Nom et e-mail | **Responsable de la sécurité**<br/>Représentant de l’équipe de sécurité qui peut valider que le plan répond aux exigences de votre organisation en matière de sécurité. | SO/R |
| Nom et e-mail | **Responsable du support informatique / Support technique**<br/>Représentant de l’organisation du support informatique qui peut fournir des commentaires sur la capacité de prise en charge de cette modification du point de vue du support technique. | R/I |
| Nom et e-mail des utilisateurs pilotes | **Utilisateurs de rôles privilégiés**<br/>Groupe d’utilisateurs pour lesquels Privileged Identity Management est implémenté. Ils devront savoir comment activer leurs rôles une fois Privileged Identity Management implémenté. | I |

#### <a name="stakeholders-privileged-identity-management-for-azure-roles"></a>Participants : Privileged Identity Management pour les rôles Azure

| Nom | Role | Action |
| --- | --- | --- |
| Nom et e-mail | **Propriétaire de l’abonnement / de la ressource**<br/>Un représentant des propriétaires du service informatique de chaque abonnement ou ressource pour lequel ou laquelle vous voulez déployer Privileged Identity Management | SO/R/I |
| Nom et e-mail | **Responsable de la sécurité**<br/>Représentant de l’équipe de sécurité qui peut valider que le plan répond aux exigences de sécurité de votre organisation. | SO/R |
| Nom et e-mail | **Responsable du support informatique / Support technique**<br/>Représentant de l’organisation du support informatique qui peut fournir des commentaires sur la capacité de prise en charge de cette modification du point de vue du support technique. | R/I |
| Nom et e-mail des utilisateurs pilotes | **Utilisateurs du rôle Azure**<br/>Groupe d’utilisateurs pour lesquels Privileged Identity Management est implémenté. Ils devront savoir comment activer leurs rôles une fois Privileged Identity Management implémenté. | I |

### <a name="start-using-privileged-identity-management"></a>Commencer à utiliser Privileged Identity Management

Dans le cadre du processus de planification, vous devez préparer Privileged Identity Management en suivant notre article [commencer à utiliser Privileged Identity Management](pim-getting-started.md). Privileged Identity Management vous donne accès à certaines fonctionnalités qui sont conçues pour faciliter votre déploiement.

Si votre objectif est de déployer Privileged Identity Management pour les ressources Azure, vous devez consulter notre article [Découvrir des ressources Azure à gérer dans Privileged Identity Management](pim-resource-roles-discover-resources.md). Seuls les propriétaires d’abonnements et de groupes d’administration peuvent placer ces ressources sous la gestion de Privileged Identity Management. Une fois qu’elles sont sous gestion, la fonctionnalité PIM est accessible aux propriétaires à tous les niveaux, notamment celui du groupe d’administration, de l’abonnement, du groupe de ressources et de la ressource. Si vous êtes un administrateur général qui tente de déployer Privileged Identity Management pour vos ressources Azure, vous pouvez [élever l’accès pour gérer tous les abonnements Azure](../../role-based-access-control/elevate-access-global-admin.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json) afin d’obtenir l’accès à toutes les ressources Azure du répertoire pour la découverte. Toutefois, il est conseillé d’obtenir l’approbation de chacun des propriétaires d’abonnements avant de gérer leurs ressources avec Privileged Identity Management.

### <a name="enforce-principle-of-least-privilege"></a>Appliquer le principe des privilèges minimum

Il est important de vous assurer que vous avez appliqué le principe du privilège minimum dans votre organisation, aussi bien pour les rôles Azure AD que pour les rôles Azure.

#### <a name="plan-least-privilege-delegation"></a>Planifier la délégation du privilège minimum

Pour les rôles Azure AD, il est courant pour les organisations d’attribuer le rôle Administrateur général à un certain nombre d’administrateurs quand la plupart d’entre eux n’ont besoin que d’un ou deux rôles d’administrateur spécifiques et moins puissants. Avec un grand nombre d’administrateurs généraux ou d’autres rôles à privilèges élevés, il est difficile de suivre de suffisamment près les attributions de rôles privilégiés.

Suivez ces étapes pour implémenter le principe du privilège minimum pour vos rôles Azure AD.

1. Appréhendez la précision des rôles en lisant et en comprenant les [rôles d’administrateur Azure AD disponibles](../roles/permissions-reference.md#available-roles). Vous et votre équipe devez également faire référence aux [rôles d’administrateur par tâche d’identité dans Azure AD](../roles/delegate-by-task.md), qui explique le rôle moins privilégié pour des tâches spécifiques.

1. Listez les utilisateurs avec des rôles privilégiés dans votre organisation. Vous pouvez utiliser la capacité [Détection et insights (préversion)](pim-security-wizard.md) de Privileged Identity Management pour réduire votre exposition.

    ![Page Découverte et Insights (version préliminaire) pour réduire l’exposition via des rôles privilégiés](./media/pim-deployment-plan/new-preview-page.png)

1. Pour tous les administrateurs généraux dans votre organisation, découvrez pourquoi ils ont besoin du rôle. Ensuite, supprimez-les du rôle Administrateur général et attribuez-leur des rôles intégrés ou des rôles personnalisés avec un moindre privilège dans Azure Active Directory. Pour information, Microsoft ne compte actuellement qu’une dizaine d’administrateurs ayant le rôle Administrateur général. Apprenez-en davantage sur [la façon dont Microsoft utilise Privileged Identity Management](https://www.microsoft.com/itshowcase/Article/Content/887/Using-Azure-AD-Privileged-Identity-Management-for-elevated-access).

1. Pour tous les autres rôles Azure AD, passez en revue la liste des affectations, identifiez les administrateurs qui n’ont plus besoin du rôle et supprimez-les de leurs affectations.

Pour automatiser les deux dernières étapes, vous pouvez utiliser les révisions d’accès dans Privileged Identity Management. En suivant les étapes de [démarrer une révision d’accès des rôles Azure AD dans Privileged Identity Management](pim-how-to-start-security-review.md), vous pouvez configurer une révision d’accès pour chaque rôle Azure AD avec un ou plusieurs membres.

![Volet Créer une révision d’accès pour les rôles Azure AD](./media/pim-deployment-plan/create-access-review.png)

Définissez les réviseurs sur **Membres (auto)** . Tous les utilisateurs du rôle recevront un e-mail leur demandant de confirmer qu’ils ont besoin de l’accès. Par ailleurs, activez l’option **Exiger la raison lors de l’approbation** dans les paramètres avancés afin que les utilisateurs puissent indiquer pourquoi ils ont besoin du rôle. En fonction de ces informations, vous pourrez supprimer des utilisateurs des rôles inutiles ou les déléguer à des rôles d’administrateur plus granulaires.

Les révisions d’accès s’appuient sur les e-mails pour demander aux utilisateurs de revoir leur accès aux rôles. Si vous avez des comptes privilégiés sans e-mail lié, veillez à renseigner le champ de messagerie secondaire sur ces comptes. Pour plus d’informations, consultez [Attribut proxyAddresses dans Azure AD](https://support.microsoft.com/help/3190357/how-the-proxyaddresses-attribute-is-populated-in-azure-ad).

#### <a name="plan-azure-resource-role-delegation"></a>Planifier la délégation du rôle de ressource Azure

Pour les abonnements et les ressources Azure, vous pouvez configurer un processus de révision d’accès similaire pour passer en revue les rôles dans chaque abonnement ou ressource. L’objectif de ce processus consiste à réduire les attributions de rôles Propriétaire et Administrateur de l’accès utilisateur attachées à chaque abonnement ou ressource et à supprimer les attributions inutiles. Toutefois, les organisations délèguent souvent de telles tâches au propriétaire de chaque abonnement ou ressource, car il a une meilleure compréhension des rôles spécifiques (en particulier, des rôles personnalisés).

Si vous avez le rôle Administrateur général et que vous tentez de déployer PIM pour les rôles Azure dans votre organisation, vous pouvez [élever l’accès pour gérer tous les abonnements Azure](../../role-based-access-control/elevate-access-global-admin.md?toc=%2fazure%2factive-directory%2fprivileged-identity-management%2ftoc.json) afin d’accéder à chaque abonnement. Vous pouvez ensuite rechercher le propriétaire de chaque abonnement et travailler avec lui pour supprimer les affectations inutiles et réduire l’affectation de rôle Propriétaire.

Les utilisateurs avec le rôle Propriétaire pour un abonnement Azure peuvent également utiliser des [révisions d’accès pour les ressources Azure](pim-resource-roles-start-access-review.md) afin d’auditer et de supprimer des attributions de rôles inutiles, comme le processus décrit précédemment pour les rôles Azure AD.

### <a name="decide-which-role-assignments-should-be-protected-by-privileged-identity-management"></a>Identifier les attributions de rôle qui doivent être protégées par Privileged Identity Management

Après le nettoyage des attributions de rôle privilégié dans votre organisation, vous devez choisir les rôles à protéger avec Privileged Identity Management.

Si un rôle est protégé par Privileged Identity Management, les utilisateurs éligibles qui lui sont attribués doivent s’élever pour utiliser les privilèges octroyés par le rôle. Le processus d’élévation peut également inclure l’obtention de l’approbation, l’utilisation de l’authentification multifacteur et l’indication d’un motif d’activation. Privileged Identity Management pouvez également suivre les élévations de privilèges par le biais des notifications et des journaux d’événements d’audit Privileged Identity Management et Azure AD.

Le choix des rôles à protéger avec Privileged Identity Management peut être difficile et sera différent pour chaque organisation. Cette section présente nos conseils en matière de meilleures pratiques pour les rôles Azure et Azure AD.

#### <a name="azure-ad-roles"></a>Rôles Azure AD

Il est important de classer par ordre de priorité la protection des rôles Azure AD qui comportent le plus d’autorisations. En fonction des modèles d’utilisation parmi tous les clients Privileged Identity Management, les 10 principaux rôles Azure AD managés par Privileged Identity Management sont les suivants :

1. Administrateur général
1. Administrateur de sécurité
1. Administrateur d’utilisateurs
1. Administrateur Exchange
1. Administrateur SharePoint
1. Administrateur Intune
1. Lecteur de sécurité
1. Administrateur de services fédérés
1. Administrateur de facturation
1. Administrateur Skype Entreprise

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de gérer tous les rôles Administrateur général et Administrateur de la sécurité à l’aide de Privileged Identity Management dans un premier temps, car il s’agit des utilisateurs qui peuvent être les plus nuisibles quand ils sont compromis.

Il est important de prendre en compte quelles données et autorisations sont les plus sensibles pour votre organisation. Par exemple, certaines organisations souhaitent protéger leurs rôles Administrateur Power BI ou Administrateur Teams à l’aide de Privileged Identity Management, car ils peuvent accéder aux données et modifier les workflows de base.

Si des rôles sont attribués à des utilisateurs invités, ils sont vulnérables aux attaques.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de gérer tous les rôles avec des utilisateurs invités à l’aide de Privileged Identity Management pour réduire les risques associés à des comptes d’utilisateurs invités compromis.

Les rôles de lecteur comme Lecteur de répertoire, Lecteur du Centre de messages et Lecteur de sécurité sont parfois considérés comme étant moins importants que d’autres rôles parce qu’ils n’ont pas d’autorisation d’écriture. Cependant, certains de nos clients protègent également ces rôles, car les attaquants ayant accès à ces comptes pourraient être en mesure de lire des données sensibles, telles que des données personnelles. Prenez ce risque en considération quand vous décidez si les rôles de lecteur de votre organisation doivent être gérés à l’aide de Privileged Identity Management.

#### <a name="azure-roles"></a>Rôles Azure

Quand vous choisissez les attributions de rôle qui doivent être managées à l’aide de Privileged Identity Management pour les ressources Azure, vous devez tout d’abord identifier les abonnements/ressources qui sont essentiels pour votre organisation. Des exemples de ces abonnements/ressources sont :

- Ressources qui hébergent les données les plus sensibles
- Ressources desquelles dépendent les applications orientées client de base

Si vous êtes un administrateur général qui éprouve des difficultés à décider quels abonnements et quelles ressources sont les plus importants, nous vous conseillons de contacter les propriétaires d’abonnements de votre organisation pour dresser la liste des ressources gérées par chaque abonnement. Ensuite, collaborez avec les propriétaires d’abonnements pour regrouper les ressources en fonction du niveau de gravité (bas, moyen, élevé) en cas de compromission. Classez par ordre de priorité la gestion des ressources avec Privileged Identity Management en fonction de ce niveau de gravité.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de collaborer avec les propriétaires d’abonnements/de ressources de services critiques pour configurer le workflow Privileged Identity Management pour tous les rôles dans les abonnements/ressources sensibles.

Privileged Identity Management pour les ressources Azure prend en charge les comptes de service associés à un délai. Vous devez traiter les comptes de service exactement comme un compte d’utilisateur normal.

Pour les abonnements/ressources qui ne sont pas aussi critiques, vous n’avez pas besoin de configurer Privileged Identity Management pour tous les rôles. Toutefois, vous devez toujours protéger les rôles Propriétaire et Administrateur de l’accès utilisateur avec Privileged Identity Management.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de gérer les rôles Propriétaire et Administrateur de l’accès utilisateur de l’ensemble des abonnements/ressources à l’aide de Privileged Identity Management.

### <a name="decide-whether-to-use-a-group-to-assign-roles"></a>Décider s’il faut utiliser un groupe pour attribuer des rôles

L’attribution d’un rôle à un groupe plutôt qu’à des utilisateurs individuels est une décision stratégique. Lors de la planification, envisagez d’attribuer un rôle à un groupe pour gérer les attributions de rôles dans les cas suivants :

- De nombreux utilisateurs sont affectés à un rôle
- Vous voulez déléguer l’attribution du rôle

#### <a name="many-users-are-assigned-to-a-role"></a>De nombreux utilisateurs sont affectés à un rôle

Le suivi des personnes qui sont affectées à un rôle et la gestion de leurs affectations en fonction du moment où elles en ont besoin peut prendre du temps lorsqu’ils sont effectués manuellement. Pour affecter un groupe à un rôle, vous devez d’abord [créer un groupe assignable à un rôle](../roles/groups-create-eligible.md) puis rendre le groupe éligible à un rôle. Cette action soumet tous les membres du groupe au même processus d’activation que les utilisateurs individuels qui sont éligibles pour s’élever dans ce rôle. Les membres du groupe activent leurs affectations au groupe individuellement à l’aide de la demande d’activation et du processus d’approbation Privileged Identity Management. Le groupe n’est pas activé, seulement l’appartenance au groupe de l’utilisateur.

#### <a name="you-want-to-delegate-assigning-the-role"></a>Vous voulez déléguer l’attribution du rôle

Un propriétaire de groupe peut gérer l’appartenance à un groupe. Pour les groupes Azure AD assignables à un rôle, seuls l’administrateur de rôle privilégié, l’administrateur général et les propriétaires de groupes peuvent gérer l’appartenance à un groupe. En ajoutant de nouveaux membres au groupe, le membre obtient l’accès aux rôles auxquels le groupe est affecté, que l’affectation soit éligible ou active. Utilisez les propriétaires de groupe pour déléguer la gestion de l’appartenance à un groupe pour un rôle attribué afin de réduire l’étendue des privilèges requis. Pour plus d’informations sur l’affectation d’un propriétaire à un groupe lors de la création du groupe, consultez [Créer un groupe avec attribution de rôle dans Azure AD](../roles/groups-create-eligible.md).

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de placer les groupes Azure AD assignables à un rôle sous la gestion de Privileged Identity Management. Une fois qu’un groupe assignable à un rôle a été placé sous la gestion de PIM, il est appelé groupe d’accès privilégié. Utilisez PIM pour exiger des propriétaires de groupe qu’ils activent leur attribution de rôle Propriétaire avant de pouvoir gérer l’appartenance au groupe. Pour plus d’informations sur le placement de groupes sous la gestion de PIM, consultez [Introduire un groupe d’accès privilégié (préversion) dans Privileged Identity Management](groups-discover-groups.md).

### <a name="decide-which-role-assignments-should-be-permanent-or-eligible"></a>Identifier les affectations de rôles qui doivent être permanentes ou éligibles

Une fois que vous avez dressé la liste des rôles qui seront gérés par Privileged Identity Management, vous devez choisir les utilisateurs qui auront le rôle éligible et ceux qui auront le rôle actif en permanence. Les rôles actifs en permanence sont les rôles normaux attribués via les ressources Azure et Azure Active Directory, tandis que les rôles éligibles peuvent uniquement être attribués dans Privileged Identity Management.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de n’avoir aucune attribution active en permanence pour les rôles Azure AD et les rôles Azure autres que les [deux comptes d’accès d’urgence de secours](../roles/security-emergency-access.md) recommandés, qui doivent avoir le rôle Administrateur général permanent.

Même si nous recommandons de n’avoir aucun administrateur permanent, il est parfois difficile pour les organisations d’atteindre immédiatement cet objectif. Voici des éléments à prendre en compte lors de cette décision :

- Fréquence d’élévation : si l’utilisateur n’a besoin de l’affectation de rôle privilégié qu’une seule fois, l’affectation ne doit pas être permanente. En revanche, si l’utilisateur a besoin du rôle pour ses tâches quotidiennes et si l’utilisation de Privileged Identity Management réduit considérablement sa productivité, le rôle permanent peut être envisagé.
- Cas spécifiques à votre organisation : si la personne qui reçoit le rôle éligible fait partie d’une équipe distante ou est un cadre de haut rang au point que la communication et l’application du processus d’élévation s’avèrent difficiles, le rôle permanent peut être envisagé.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de configurer des révisions d’accès récurrentes pour les utilisateurs avec des affectations de rôles permanentes (le cas échéant). Apprenez-en davantage sur la révision d’accès récurrente dans la dernière section de ce plan de déploiement

### <a name="draft-your-privileged-identity-management-settings"></a>Composer vos paramètres Privileged Identity Management

Avant d’implémenter votre solution Privileged Identity Management, il est conseillé de composer vos paramètres Privileged Identity Management pour chaque rôle privilégié utilisé par votre organisation. Cette section contient des exemples de paramètres Privileged Identity Management pour des rôles particuliers (ils sont indiqués uniquement pour référence et peuvent être différents pour votre organisation). Chacun de ces paramètres est expliqué en détail avec des recommandations de Microsoft après les tableaux.

#### <a name="privileged-identity-management-settings-for-azure-ad-roles"></a>Paramètres Privileged Identity Management pour les rôles Azure AD

| Role | Exiger une authentification multifacteur | Notification | Ticket d’incident | Exiger une approbation | Approbateur | Durée d’activation | Administrateur permanent |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Administrateur général | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | Autres administrateurs généraux | 1 heure | Comptes d’accès d’urgence |
| Administrateur Exchange | :heavy_check_mark: | :heavy_check_mark: | :x: | :x: | None | 2 heures | None |
| Administrateur du support technique | :x: | :x: | :heavy_check_mark: | :x: | None | 8 heures | None |

#### <a name="privileged-identity-management-settings-for-azure-roles"></a>Paramètres Privileged Identity Management pour les rôles Azure

| Role | Exiger une authentification multifacteur | Notification | Exiger une approbation | Approbateur | Durée d’activation | Administrateur actif | Expiration active | Expiration éligible |
| --- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| Propriétaire d’abonnements critiques | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | Autres propriétaires de l’abonnement | 1 heure | None | n/a | 3 mois |
| Administrateur de l’accès utilisateur d’abonnements moins critiques | :heavy_check_mark: | :heavy_check_mark: | :x: | None | 1 heure | None | n/a | 3 mois |
| Contributeur de machine virtuelle | :x: | :heavy_check_mark: | :x: | None | 3 heures | None | n/a | 6 mois |

Le tableau suivant décrit chacun des paramètres.

| Paramètre | Description |
| --- | --- |
| Role | Nom du rôle pour lequel vous définissez les paramètres. |
| Exiger une authentification multifacteur | Indique si l’utilisateur éligible doit effectuer une authentification multifacteur avant d’activer le rôle.<br/><br/> :heavy_check_mark: **Microsoft recommande** d’appliquer l’authentification multifacteur pour tous les rôles d’administrateur, en particulier si les rôles ont des utilisateurs invités. |
| Notification | Si la valeur est true, les rôles Administrateur général, Administrateur de rôle privilégié et Administrateur de la sécurité de l’organisation reçoivent une notification par e-mail quand un utilisateur éligible active le rôle.<br/><br/>**Remarque :** Certaines organisations ne disposent pas d’une adresse e-mail liée à leurs comptes d’administrateurs ; pour obtenir ces notifications par e-mail, vous devez définir une autre adresse e-mail afin que les administrateurs reçoivent ces e-mails. |
| Ticket d’incident | Indique si l’utilisateur éligible doit enregistrer un numéro de ticket d’incident lors de l’activation de son rôle. Ce paramètre permet à une organisation d’identifier chaque activation avec un numéro d’incident interne pour atténuer les activations indésirables.<br/><br/> :heavy_check_mark: **Microsoft recommande** de tirer parti des numéros de ticket d’incident pour lier Privileged Identity Management à votre système interne. Cette méthode peut être utile pour les approbateurs qui ont besoin de contexte pour l’activation. |
| Exiger une approbation | Indique si l’utilisateur éligible doit obtenir une approbation pour activer le rôle.<br/><br/> :heavy_check_mark: **Microsoft recommande** de configurer l’approbation pour les rôles avec le plus d’autorisations. Selon les modèles d’utilisation de tous les clients Privileged Identity Management, Administrateur général, Administrateur d’utilisateurs, Administrateur Exchange, Administrateur de la sécurité et Administrateur de mots de passe sont les rôles les plus courants avec configuration de l’approbation. |
| Approbateur | Si l’approbation est nécessaire pour activer le rôle éligible, liste les personnes qui doivent approuver la demande. Par défaut, Privileged Identity Management définit comme approbateur tous les utilisateurs qui sont des administrateurs de rôles privilégiés, qu’ils soient permanents ou éligibles.<br/><br/>**Remarque :** Si un utilisateur est à la fois éligible pour un rôle Azure AD et un approbateur du rôle, il ne sera pas en mesure de s’approuver.<br/><br/> :heavy_check_mark: **Microsoft recommande** de choisir comme approbateurs les utilisateurs qui sont les plus compétents quant au rôle et ses utilisateurs fréquents plutôt qu’un administrateur général. |
| Durée d’activation | Durée pendant laquelle un utilisateur est activé dans le rôle avant l’expiration. |
| Administrateur permanent | Liste des utilisateurs qui seront un administrateur permanent pour le rôle (activation jamais nécessaire).<br/><br/> :heavy_check_mark: **Microsoft recommande** de n’avoir aucun administrateur permanent pour tous les rôles à l’exception des administrateurs généraux. Vous trouverez plus d’informations à ce sujet dans la section relative aux personnes qui doivent devenir éligibles et à celles qui doivent être actives en permanence de ce plan. |
| Administrateur actif | Pour les ressources Azure, un administrateur actif correspond à la liste des utilisateurs qui n’ont jamais besoin d’activation pour utiliser le rôle. Cette liste n’est pas considérée comme un administrateur permanent comme dans les rôles Azure AD, car vous pouvez définir une heure d’expiration à laquelle l’utilisateur perdra ce rôle. |
| Expiration active | Les attributions de rôles actives pour les rôles Azure expirent après la durée configurée. Vous pouvez choisir entre 15 jours, 1 mois, 3 mois, 6 mois, 1 an, ou un rôle actif en permanence. |
| Expiration éligible | Les attributions de rôles éligibles pour les rôles Azure expirent après cette durée. Vous pouvez choisir entre 15 jours, 1 mois, 3 mois, 6 mois, 1 an, ou un rôle éligible en permanence. |

## <a name="implementation-plan"></a>Plan d’implémentation

La base d’une planification appropriée est la base sur laquelle vous pouvez déployer correctement une application avec Azure Active Directory.  Elle assure une incorporation et une sécurité intelligentes qui simplifient l’intégration tout en réduisant le temps nécessaire à des déploiements réussis.  Cette combinaison garantit que votre application est intégrée en toute simplicité, tout en limitant les temps d’arrêt pour vos utilisateurs finaux.

### <a name="identify-test-users"></a>Identifier les utilisateurs de test

Utilisez cette section pour identifier un groupe ou des groupes d’utilisateurs pour valider l’implémentation. Selon les paramètres que vous avez sélectionnés dans la section de planification, identifiez les utilisateurs que vous souhaitez tester pour chaque rôle.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de définir les propriétaires de services de chaque rôle Azure AD comme utilisateurs de test afin qu’ils se familiarisent avec le processus et deviennent des partisans internes du déploiement.

Dans ce tableau, identifiez les utilisateurs de test qui vérifieront que les paramètres des rôles fonctionnent.

| Nom de rôle | Utilisateurs de test |
| --- | --- |
| &lt;Nom du rôle&gt; | &lt;Utilisateurs pour tester le rôle&gt; |
| &lt;Nom du rôle&gt; | &lt;Utilisateurs pour tester le rôle&gt; |

### <a name="test-implementation"></a>Implémentation de test

Maintenant que vous avez identifié les utilisateurs de test, utilisez cette étape pour configurer Privileged Identity Management pour ceux-ci. Si votre organisation souhaite incorporer le workflow Privileged Identity Management dans votre propre application interne au lieu d’utiliser Privileged Identity Management dans le portail Azure, toutes les opérations dans Privileged Identity Management sont également prises en charge via notre [Graphique API](/graph/api/resources/privilegedidentitymanagement-root).

#### <a name="configure-privileged-identity-management-for-azure-ad-roles"></a>Configurer Privileged Identity Management pour les rôles Azure AD

1. [Configurez les paramètres de rôle Azure AD](pim-how-to-change-default-settings.md) selon ce que vous avez planifié.

1. Accédez à **Rôles Azure AD** , sélectionnez **Rôles** , puis sélectionnez le rôle que vous avez configuré.

1. Pour le groupe d’utilisateurs de test, s’ils sont déjà des administrateurs permanents, vous pouvez les rendre éligibles en les recherchant et en les convertissant du statut permanent au statut éligible en sélectionnant e bouton de sélection (…) de leur ligne. S’ils n’ont pas encore les affectations de rôles, vous pouvez [créer une affectation éligible](pim-how-to-add-role-to-user.md#make-a-user-eligible-for-a-role).

1. Répétez les étapes 1 à 3 pour tous les rôles que vous souhaitez tester.

1. Une fois que vous avez configuré les utilisateurs de test, vous devez leur envoyer le lien pour savoir comment [activer leur rôle Azure AD](pim-how-to-activate-role.md).

#### <a name="configure-privileged-identity-management-for-azure-roles"></a>Configurer Privileged Identity Management pour les rôles Azure

1. [Configurez les paramètres de rôle de ressource Azure](pim-resource-roles-configure-role-settings.md) pour un rôle dans un abonnement ou une ressource que vous souhaitez tester.

1. Accédez à **Ressources Azure** pour cet abonnement et sélectionnez **Rôles** , puis sélectionnez le rôle que vous avez configuré.

1. Pour le groupe d’utilisateurs de test, s’ils sont déjà des administrateurs actifs, vous pouvez les rendre éligibles en les recherchant et en [mettant à jour leur attribution de rôle](pim-resource-roles-assign-roles.md#update-or-remove-an-existing-role-assignment). S’ils n’ont pas encore le rôle, vous pouvez [affecter un nouveau rôle](pim-resource-roles-assign-roles.md#assign-a-role).

1. Répétez les étapes 1 à 3 pour tous les rôles que vous souhaitez tester.

1. Une fois que vous avez configuré les utilisateurs de test, vous devez leur envoyer le lien pour savoir comment [activer leur rôle de ressource Azure](pim-resource-roles-activate-your-roles.md).

Vous devez utiliser cette étape pour vérifier si toute la configuration que vous avez définie pour les rôles fonctionne correctement. Utilisez le tableau suivant pour documenter vos tests. Vous devez également utiliser cette étape pour optimiser la communication avec les utilisateurs affectés.

| Role | Comportement attendu lors de l’activation | Résultats réels |
| --- | --- | --- |
| Administrateur général | (1) Exiger une authentification multifacteur<br/>(2) Exiger une approbation<br/>(3) L’approbateur reçoit une notification et peut approuver<br/>(4) Le rôle expire au bout d’un délai prédéfini |  |
| Propriétaire de l’abonnement *X* | (1) Exiger une authentification multifacteur<br/>(2) L’affectation éligible expire après la période configurée |  |

### <a name="communicate-privileged-identity-management-to-affected-stakeholders"></a>Communiquer Privileged Identity Management aux parties prenantes affectées

Le déploiement de Privileged Identity Management introduit des étapes supplémentaires pour les utilisateurs des rôles privilégiés. Même si Privileged Identity Management réduit considérablement les problèmes de sécurité associés aux identités privilégiées, la modification doit être communiquée efficacement avant le déploiement à l’échelle de l’organisation. En fonction du nombre d’administrateurs impactés, les organisations choisissent souvent de créer un document interne, une vidéo ou un e-mail concernant la modification. Les éléments fréquemment inclus dans ces communications sont notamment :

- Qu’est-ce que PIM
- Quel est l’avantage pour l’organisation
- Qui sera concerné
- Quand est-ce que PIM sera déployé
- Quelles étapes supplémentaires seront nécessaires pour que les utilisateurs activent leur rôle
    - Vous devez envoyer des liens vers notre documentation :
    - [Activer les rôles Azure AD](pim-how-to-activate-role.md)
    - [Activer les rôles Azure](pim-resource-roles-activate-your-roles.md)
- Informations de contact ou lien du support technique pour tout problème associé à PIM

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de prendre du temps avec votre équipe de support/support technique pour lui présenter le workflow Privileged Identity Management (si votre organisation dispose d’une équipe de support informatique interne). Fournissez-lui la documentation appropriée ainsi que vos informations de contact.

### <a name="move-to-production"></a>Passer en production

Une fois que votre test est terminé et réussi, passez Privileged Identity Management en production en répétant toutes les étapes dans les phases de test pour tous les utilisateurs de chaque rôle que vous avez défini dans votre configuration Privileged Identity Management. Par Privileged Identity Management pour les rôles Azure AD, les organisations testent et déploient souvent Privileged Identity Management pour les administrateurs généraux avant de tester et de déployer Privileged Identity Management pour d’autres rôles. Dans le même temps, pour les ressources Azure, les organisations testent et déploient normalement Privileged Identity Management, un abonnement Azure à la fois.

### <a name="if-a-rollback-is-needed"></a>Si une restauration est nécessaire

Si Privileged Identity Management ne fonctionne pas comme vous le souhaitez dans l’environnement de production, les étapes de restauration suivantes peuvent vous aider à revenir à un état correct connu avant la configuration de Privileged Identity Management :

#### <a name="azure-ad-roles"></a>Rôles Azure AD

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Ouvrez **Azure AD Privileged Identity Management** .
1. Sélectionnez **Rôles Azure AD** , puis **Rôles** .
1. Pour chaque rôle que vous avez configuré, sélectionnez le bouton de sélection ( **…** ) pour tous les utilisateurs ayant une affectation éligible.
1. Sélectionnez l’option **Rendre permanent** afin de rendre l’attribution de rôle permanente.

#### <a name="azure-roles"></a>Rôles Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Ouvrez **Azure AD Privileged Identity Management** .
1. Sélectionnez **Ressources Azure** , puis un abonnement ou une ressource que vous souhaitez restaurer.
1. Sélectionnez **Rôles** .
1. Pour chaque rôle que vous avez configuré, sélectionnez le bouton de sélection ( **…** ) pour tous les utilisateurs ayant une affectation éligible.
1. Sélectionnez l’option **Rendre permanent** afin de rendre l’attribution de rôle permanente.

## <a name="next-steps-after-deploying"></a>Étapes suivantes après le déploiement

Le déploiement réussi de Privileged Identity Management en production est une étape importante en termes de sécurisation des identités privilégiées de votre organisation. Avec le déploiement de Privileged Identity Management sont fournies des fonctionnalités Privileged Identity Management supplémentaires que vous devez utiliser pour la sécurité et la conformité.

### <a name="use-privileged-identity-management-alerts-to-safeguard-your-privileged-access"></a>Utiliser des alertes Privileged Identity Management pour protéger votre accès privilégié

Pour plus d’informations sur l’utilisation de la fonctionnalité d’alerte intégrée de Privileged Identity Management pour protéger votre organisation, consultez [Alertes de sécurité](pim-how-to-configure-security-alerts.md#security-alerts). Ces alertes incluent notamment les suivantes : les administrateurs n’utilisent pas les rôles privilégiés, les rôles sont affectés en dehors de Privileged Identity Management, les rôles sont activés trop fréquemment, et bien plus encore. Pour protéger entièrement votre organisation, vous devez parcourir régulièrement votre liste d’alertes et résoudre les problèmes. Vous pouvez afficher et résoudre vos alertes de la façon suivante :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
1. Ouvrez **Azure AD Privileged Identity Management** .
1. Sélectionnez **Rôles Azure AD** , puis **Alertes** .

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de traiter immédiatement toutes les alertes marquées avec le niveau de gravité élevé. Pour les alertes avec les niveaux de gravité moyen et bas, vous devez rester informé et apporter des modifications si vous pensez qu’il existe une menace de sécurité.

Si l’une des alertes spécifiques est inutile ou ne s’applique pas à votre organisation, vous pouvez toujours ignorer l’alerte dans la page des alertes. Vous pouvez toujours rétablir ce rejet ultérieurement dans la page des paramètres Azure AD.

### <a name="set-up-recurring-access-reviews-to-regularly-audit-your-organizations-privileged-identities"></a>Configurer des révisions d’accès récurrentes pour auditer régulièrement les identités privilégiées de votre organisation

Les révisions d’accès représentent le meilleur moyen de demander aux utilisateurs auxquels des rôles privilégiés ont été affectés ou à des réviseurs spécifiques si chaque utilisateur a besoin de l’identité privilégiée. Les révisions d’accès sont très utiles si vous souhaitez réduire la surface d’attaque et garantir la conformité. Pour plus d’informations sur le démarrage d’une révision d’accès, consultez [Révisions d’accès des rôles Azure AD](pim-how-to-start-security-review.md) et [Révisions d’accès des rôles Azure](pim-resource-roles-start-access-review.md). Pour certaines organisations, la réalisation d’une révision d’accès périodique est nécessaire pour garantir la conformité aux lois et réglementations alors que pour d’autres, la révision d’accès représente le meilleur moyen d’appliquer le principe des privilèges minimum au sein de l’organisation.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de configurer des révisions d’accès tous les trimestres pour l’ensemble de vos rôles Azure et Azure AD.

Dans la plupart des cas, le réviseur pour les rôles Azure AD est l’utilisateur lui-même tandis que le réviseur pour les rôles Azure est le propriétaire de l’abonnement dans lequel se trouve le rôle. Toutefois, il arrive souvent que les sociétés aient des comptes privilégiés non liés à l’adresse e-mail d’une personne en particulier. Dans ce cas, personne ne lit ni ne révise l’accès.

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** d’ajouter une adresse e-mail secondaire pour tous les comptes avec des affectations de rôles privilégiés qui ne sont liés à aucune adresse e-mail régulièrement consultée

### <a name="get-the-most-out-of-your-audit-log-to-improve-security-and-compliance"></a>Tirer le meilleur parti de votre journal d’audit pour améliorer la sécurité et la conformité

Le journal d’audit est l’endroit où vous pouvez vous tenir informé et être conforme aux réglementations. Privileged Identity Management stocke actuellement un historique de 30 jours résumant tous les historiques de votre organisation dans son journal d’audit, notamment :

- Activation/désactivation de rôles éligibles
- Activités d’attribution de rôle à l’intérieur et en dehors de Privileged Identity Management
- Modifications dans les paramètres de rôle
- Activités de demande/d’approbation/de refus pour l’activation de rôle avec configuration de l’approbation
- Mise à jour des alertes

Vous pouvez accéder aux journaux d’audit si vous êtes un administrateur général ou un administrateur de rôle privilégié. Pour plus d’informations, consultez [Historique d’audit pour les rôles Azure AD](pim-how-to-use-audit-log.md) et [Historique d’audit pour les rôles Azure](azure-pim-resource-rbac.md).

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** d’avoir au moins un administrateur pour lire tous les événements d’audit une fois par semaine et exporter vos événements d’audit tous les mois.

Si vous souhaitez stocker automatiquement vos événements d’audit sur une plus longue période, le journal d’audit de Privileged Identity Management est synchronisé automatiquement dans les [journaux d’audit Azure AD](../reports-monitoring/concept-audit-logs.md).

> [!TIP]
> :heavy_check_mark: **Microsoft recommande** de configurer la [supervision des journaux Azure](../reports-monitoring/concept-activity-logs-azure-monitor.md) afin d’archiver les événements d’audit dans un compte de stockage Azure pour une sécurité et une conformité accrues.
