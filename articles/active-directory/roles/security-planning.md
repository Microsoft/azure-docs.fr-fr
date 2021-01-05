---
title: Pratiques d’accès sécurisé pour les administrateurs dans Azure AD | Microsoft Docs
description: Assurez-vous que l’accès des administrateurs et les comptes administrateur de votre organisation sont sécurisés. Pour les architectes de systèmes et les professionnels de l’informatique qui configurent Azure AD, Azure et Microsoft Online Services.
services: active-directory
keywords: ''
author: curtand
manager: daveba
ms.author: curtand
ms.date: 11/05/2020
ms.topic: conceptual
ms.service: active-directory
ms.workload: identity
ms.subservice: roles
ms.custom: it-pro
ms.reviewer: martincoetzer; MarkMorow
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3a7304ffbb7718205ead48e27989d4a169375997
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861984"
---
# <a name="securing-privileged-access-for-hybrid-and-cloud-deployments-in-azure-ad"></a>Sécurisation de l’accès privilégié pour les déploiements hybrides et cloud dans Azure AD

La sécurité des ressources professionnelles dépend de l’intégrité des comptes privilégiés qui administrent vos systèmes informatiques. Des pirates informatiques s’efforcent de dérober des informations d’identification permettant de cibler des comptes administrateur ou d’autres comptes disposant d’un accès privilégié dans le but d’accéder à des données sensibles.

Pour les services cloud, la prévention et la réponse sont de la responsabilité conjointe du fournisseur de services cloud et du client. Pour plus d’informations sur les dernières menaces sur les points de terminaison et le cloud, consultez le [rapport de renseignements de sécurité Microsoft](https://www.microsoft.com/security/operations/security-intelligence-report). Cet article peut vous aider à créer une feuille de route afin de combler les lacunes de vos plans actuels, des conseils sont fournis ici.

> [!NOTE]
> Microsoft s’engage aux plus hauts niveaux de confiance, de transparence, de conformité aux normes et aux réglementations. Découvrez comment l’équipe de réponse aux incidents mondiale de Microsoft permet d’atténuer les effets des attaques contre les services cloud, et comment la sécurité est intégrée dans les produits professionnels et services cloud Microsoft dans le [Centre de gestion de la confidentialité Microsoft - Sécurité](https://www.microsoft.com/trustcenter/security) et les objectifs de conformité Microsoft dans le [Centre de gestion de la confidentialité Microsoft - Conformité](https://www.microsoft.com/trustcenter/compliance).

Traditionnellement, la sécurité d’une organisation était axée sur les points d’entrée et de sortie d’un réseau faisant office de périmètre de sécurité. Toutefois, en raison de l’utilisation d’applications SaaS et d’appareils personnels sur Internet, cette approche a perdu en efficacité. Dans Azure AD, nous remplaçons le périmètre de sécurité réseau par une authentification dans la couche identité de votre organisation, sous le contrôle d’utilisateurs assumant le rôle d’administrateurs privilégiés. L’accès de ceux-ci doit être protégé, que l’environnement soit local, cloud ou hybride.

La sécurisation de l’accès privilégié nécessite d’apporter des modifications aux éléments suivants :

* Processus, pratiques d’administration et gestion des connaissances
* Composants techniques telles que les défenses d’hôte, les protections de compte et la gestion des identités

Sécurisez votre accès privilégié d’une manière gérée et déclarée dans les services Microsoft importants à vos yeux. Si vous disposez de comptes administrateur locaux, suivez les conseils relatifs à l’accès privilégié local et hybride géré dans Active Directory prodigués dans [Sécurisation de l’accès privilégié](/windows-server/identity/securing-privileged-access/securing-privileged-access).

> [!NOTE]
> Les conseils de cet article font principalement référence aux fonctionnalités d’Azure Active Directory incluses dans les plans Azure Active Directory Premium P1 et P2. Azure Active Directory Premium P2 est inclus dans la suite EMS E5 et la suite Microsoft 365 E5. Ces conseils supposent que votre organisation dispose déjà de licences Azure AD Premium P2 pour vos utilisateurs. Si vous ne disposez pas de ces licences, certains conseils peuvent ne pas s’appliquer à votre organisation. Dans cet article, l’expression « administrateur général » est synonyme d’« administrateur d’entreprise » ou d’« administrateur client ».

## <a name="develop-a-roadmap"></a>Créer une feuille de route

Microsoft vous recommande de créer et de suivre une feuille de route pour sécuriser l’accès privilégié contre les cybercriminels. Vous pouvez toujours ajuster votre feuille de route pour prendre en compte vos fonctionnalités existantes et des exigences spécifiques au sein de votre organisation. Chaque étape de la feuille de route doit indiquer le coût et la difficulté pour les adversaires d’attaquer un accès privilégié pour vos ressources locales, cloud et hybrides. Microsoft recommande de suivre les quatre étapes de la feuille de route ci-dessous. Commencez par planifier les implémentations les plus efficaces et les plus rapides. Cet article destiné à vous guider est basé sur les enseignements que Microsoft a tirés de son expérience en matière de cyber-attaques et de traitement de celles-ci. Les chronologies mentionnées dans cette feuille de route sont approximatives.

![Étapes de la feuille de route avec délais](./media/security-planning/roadmap-timeline.png)

* Étape 1 (24-48 heures) : Éléments critiques que nous vous recommandons de traiter immédiatement

* Étape 2 (2-4 semaines) : Atténuer les techniques d’attaque les plus fréquemment utilisées

* Étape 3 (1-3 mois) : Gagner en visibilité et disposer d’un contrôle total sur l’activité administrative

* Étape 4 (six mois et plus) : Continuer à créer des défenses pour renforcer votre plateforme de sécurité

Ce cadre de feuille de route est conçu pour optimiser l’utilisation de technologies Microsoft que vous avez peut-être déjà déployées. Vous pouvez y associer des outils de sécurité d’autres fournisseurs que vous avez déjà déployés ou envisagez de déployer.

## <a name="stage-1-critical-items-to-do-right-now"></a>Étape 1 : Tâches critiques à accomplir sans délai

![Étape 1 Éléments critiques à traiter en premier](./media/security-planning/stage-one.png)

L’étape 1 de la feuille de route cible les tâches critiques rapides et faciles à implémenter. Nous vous recommandons de les traiter immédiatement au cours des premières 24 à 48 heures pour garantir un niveau de base d’accès privilégié sécurisé. Cette étape de la feuille de route d’accès privilégié sécurisé comprend les actions suivantes :

### <a name="general-preparation"></a>Préparation générale

#### <a name="turn-on-azure-ad-privileged-identity-management"></a>Activer Azure AD Privileged Identity Management

Nous vous recommandons d’activer Azure AD Privileged Identity Management (PIM) dans votre environnement de production Azure AD. Après avoir activé PIM, vous recevrez des emails de notification concernant des changements de rôles d’accès privilégié. Ces notifications contiennent des avertissements précoces lorsque des utilisateurs sont ajoutés à des rôles disposant de privilèges élevés.

Azure AD Privileged Identity Management est inclus dans Azure AD Premium P2 ou EMS E5. Pour protéger l’accès aux applications et ressources locales et dans le cloud, inscrivez-vous à l’[essai gratuit de 90 jours d’Enterprise Mobility + Security](https://www.microsoft.com/cloud-platform/enterprise-mobility-security-trial). Azure AD Privileged Identity Management et Azure AD Identity Protection surveillent l’activité de sécurité à l’aide de rapports, d’audit et d’alertes Azure AD.

Après avoir activé Azure AD Privileged Identity Management :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l’aide d’un compte d’administrateur général de votre organisation de production Azure AD.

2. Pour sélectionner l’organisation Azure AD dans laquelle vous souhaitez utiliser Privileged Identity Management, sélectionnez votre nom d’utilisateur dans l’angle supérieur droit du portail Azure.

3. Dans le menu du portail Azure, sélectionnez **Tous les services** et filtrez la liste pour **Azure AD Privileged Identity Management**.

4. Ouvrez Privileged Identity Management à partir de la liste **Tous les services** et épinglez-le à votre tableau de bord.

Assurez-vous que la première personne à utiliser PIM au sein votre organisation est affectée aux rôles **Administrateur de la sécurité** et **Administrateur de rôle privilégié**. Seuls les administrateurs de rôle privilégié peuvent gérer les attributions de rôles d’annuaire Azure AD d’utilisateurs. L’Assistant Sécurité de PIM vous guide tout au long du processus de découverte initiale et d’attribution. Vous pouvez quitter l’Assistant sans apporter de modifications supplémentaires pour l’instant.

#### <a name="identify-and-categorize-accounts-that-are-in-highly-privileged-roles"></a>Identifier et classer les comptes dans des rôles à privilèges élevés

Après avoir activé Azure AD Privileged Identity Management, affichez les utilisateurs assignés aux rôles Azure AD suivants :

* Administrateur général
* Administrateur de rôle privilégié
* Administrateur Exchange
* Administrateur SharePoint

Si vous ne disposez pas d’Azure AD Privileged Identity Management dans votre organisation, vous pouvez utiliser l’[API PowerShell](/powershell/module/azuread/get-azureaddirectoryrolemember). Commencez avec le rôle d’administrateur général, car celui-ci dispose des mêmes autorisations sur tous les services cloud auxquels votre organisation s’est abonnée. Ces autorisations sont acquises, quelle que soit leur provenance : Centre d’administration Microsoft 365, Portail Azure ou module Azure AD pour Microsoft PowerShell.

Supprimez les comptes qui ne sont plus nécessaires dans ces rôles. Ensuite, classez les comptes restants qui sont affectés aux rôles d’administrateur :

* Affectés à des utilisateurs administratifs, mais également utilisés à des fins non administratives (par exemple, courrier personnel)
* Affectés à des utilisateurs administratifs et dédiés à des fins exclusivement administratives
* Partagés entre plusieurs utilisateurs
* Pour les scénarios d’accès d’urgence de secours
* Pour les scripts automatisés
* Pour les utilisateurs externes

#### <a name="define-at-least-two-emergency-access-accounts"></a>Définir au moins deux comptes d’accès d’urgence

Il peut arriver qu’un utilisateur soit accidentellement privé de son rôle. Par exemple, si un fournisseur d’identité local fédéré n’est pas disponible, les utilisateurs ne peuvent pas se connecter ou activer un compte d’administrateur existant. Vous pouvez vous préparer à une privation d’accès accidentelle en stockant au moins deux comptes d’accès d’urgence.

Des comptes d’accès d’urgence permettent de restreindre les accès privilégiés au sein d’une organisation Azure AD. Ces comptes hautement privilégiés ne sont pas attribués à des personnes spécifiques. Les comptes d’accès d’urgence sont limités aux situations où il est impossible d’utiliser des comptes administratifs normaux. Veillez à contrôler les comptes d’accès d’urgence et à limiter leur utilisation strictement au temps nécessaire.

Évaluez les comptes qui sont affectés ou éligibles pour le rôle d’administrateur général. Si vous ne voyez aucun compte uniquement cloud utilisant le domaine \*.onmicrosoft.com (conçu pour l’accès d’urgence), créez-en. Pour plus d’informations, consultez [Managing emergency access administrative accounts in Azure AD](security-emergency-access.md) (Gestion des comptes d’administration de l’accès d’urgence dans Azure AD).

#### <a name="turn-on-multi-factor-authentication-and-register-all-other-highly-privileged-single-user-non-federated-admin-accounts"></a>Activez l’authentification multifacteur et inscrivez tous les autres comptes d’administrateur non fédérés mono-utilisateurs hautement privilégiés.

Exigez Azure AD Multi-Factor Authentication (MFA) lors de la connexion de tous les utilisateurs individuels auxquels sont affectés un ou plusieurs rôles d’administrateur Azure AD : Administrateur général, Administrateur de rôle privilégié, Administrateur de services Exchange et Administrateur de services SharePoint. Suivez le guide pour activer [l’authentification multifacteur (MFA) pour vos comptes Administrateur](../authentication/howto-mfa-userstates.md) et vérifier que tous ces utilisateurs sont inscrits sur [https://aka.ms/mfasetup](https://aka.ms/mfasetup). Vous trouverez plus d’informations aux étapes 2 et 3 du guide [Protéger l’accès aux données et services dans Microsoft 365](https://support.office.com/article/Protect-access-to-data-and-services-in-Office-365-a6ef28a4-2447-4b43-aae2-f5af6d53c68e). 

## <a name="stage-2-mitigate-frequently-used-attacks"></a>Étape 2 : Atténuer les attaques fréquemment utilisées

![Étape 2 Atténuer les attaques fréquemment utilisées](./media/security-planning/stage-two.png)

L’étape 2 de la feuille de route vise à atténuer les techniques d’attaque les plus fréquemment utilisées de vol d’informations d’identification et d’abus et peut être implémentée en 2 à 4 semaines environ. Cette étape de la feuille de route d’accès privilégié sécurisé comprend les actions suivantes.

### <a name="general-preparation"></a>Préparation générale

#### <a name="conduct-an-inventory-of-services-owners-and-admins"></a>Inventorier les services, propriétaires et administrateurs

Avec la prolifération des appareils personnels (BYOD) et du télétravail, ainsi que la croissance de la connectivité sans fil, il est essentiel de surveiller qui se connecte à votre réseau. Un audit de sécurité peut révéler la présence sur votre réseau d’appareils, d’applications et de programmes que votre organisation ne prend pas en charge et qui constituent un risque élevé. Pour plus d’informations, consultez [Présentation de la gestion et surveillance de la sécurité Azure](../../security/fundamentals/management-monitoring-overview.md). Veillez à inclure toutes les tâches suivantes dans votre processus d’inventaire.

* Identifiez les utilisateurs disposant de rôles d’administrateur et les services où ils peuvent gérer.
* Azure AD PIM vous permet d’identifier les utilisateurs au sein votre organisation, qui disposent d’un accès administrateur à Azure AD.
* En plus des rôles définis dans Azure AD, Microsoft 365 inclut un ensemble de rôles d’administrateur que vous pouvez affecter aux utilisateurs de votre organisation. Chaque rôle d’administrateur correspond à des fonctions d’entreprise courantes et fournit aux personnes de votre organisation des autorisations pour effectuer des tâches spécifiques dans le [Centre d’administration Microsoft 365](https://admin.microsoft.com). Utilisez le Centre d’administration Microsoft 365 pour déterminer quels sont les utilisateurs de votre organisation qui disposent d’un accès administrateur à Microsoft 365, y compris via des rôles non gérés dans Azure AD. Pour plus d’informations, consultez [À propos des rôles d’administrateur Microsoft 365](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d) et [Pratiques de sécurité pour Office 365](/office365/servicedescriptions/office-365-platform-service-description/office-365-securitycompliance-center).
* Inventoriez le services dont votre organisation dépend, tels qu’Azure, Intune ou Dynamics 365.
* Assurez-vous que vos comptes utilisés à des fins d’administration présentent les caractéristiques suivantes :

  * disposent d’adresses e-mail professionnelles ;
  * sont inscrits à Azure AD Multi-Factor Authentication ou utilisent une authentification multifacteur localement.
* Demandez aux utilisateurs leur justification d’entreprise d’accès administrateur.
* Supprimez l’accès administrateur pour les personnes et les services n’en ayant pas besoin.

#### <a name="identify-microsoft-accounts-in-administrative-roles-that-need-to-be-switched-to-work-or-school-accounts"></a>Identifier les comptes Microsoft dans des rôles d’administrateur à basculer vers des comptes professionnels ou scolaires

Si vos administrateurs généraux initiaux ont réutilisé leurs informations d’identification de compte Microsoft existantes quand ils ont commencé à utiliser Azure AD, remplacez les comptes Microsoft par des comptes cloud ou synchronisés individuels.

#### <a name="ensure-separate-user-accounts-and-mail-forwarding-for-global-administrator-accounts"></a>Vérifier les comptes d’utilisateur distincts et le transfert de messagerie pour les comptes administrateur général

Les comptes de courrier personnels étant régulièrement hameçonnés par des pirates informatiques, il constituent un risque qui rend les adresses de courrier personnelles inacceptables pour des comptes d’administrateur général. Pour dissocier les risques liés à Internet des privilèges administratifs, créez un compte dédié pour chaque utilisateur disposant de tels privilèges.

* Veillez à créer des comptes distincts que les utilisateurs peuvent utiliser pour accomplir des tâches d’administration générale.
* Assurez-vous que vos administrateurs généraux n’ouvrent pas d’e-mails ou n’exécutent pas de programmes accidentellement avec leurs comptes administrateur.
* Veillez à ce que les e-mails adressés à ces comptes soient transféré vers une boîte aux lettres professionnelle.

#### <a name="ensure-the-passwords-of-administrative-accounts-have-recently-changed"></a>Vérifier que les mots de passe des comptes administrateur ont été récemment modifiés

Assurez-vous que tous les utilisateurs se sont connectés à leur compte administrateur et ont modifié leur mot de passe au moins une fois au cours des 90 derniers jours. Vérifiez également que les mots de passe de tous les comptes partagés ont été modifiés récemment.

#### <a name="turn-on-password-hash-synchronization"></a>Activer la synchronisation de hachage de mot de passe

Azure AD Connect synchronise le hachage du mot de passe d’un utilisateur entre un Active Directory local et une organisation Azure AD basée dans le cloud. Vous pouvez utiliser une synchronisation de hachage du mot de passe en tant que sauvegarde si vous utilisez les services de fédération Active Directory (AD FS). Cette sauvegarde peut être utile si vos serveurs Active Directory ou AD FS locaux sont temporairement indisponibles.

La synchronisation de hachage du mot de passe permet aux utilisateurs de se connecter à un service en utilisant le mot de passe qu’ils utilisent pour se connecter à leur instance Active Directory locale. La synchronisation de hachage du mot de passe permet à Azure AD Identity Protection de détecter des informations d’identification compromises en comparant des hachages de mot de passe avec des mots de passe connus pour être compromis. Pour plus d’informations, consultez [Implémenter la synchronisation de hachage du mot de passe avec la synchronisation Azure AD Connect](../hybrid/how-to-connect-password-hash-synchronization.md).

#### <a name="require-multi-factor-authentication-for-users-in-privileged-roles-and-exposed-users"></a>Imposer une authentification multifacteur aux utilisateurs titulaires de rôles privilégiés et aux utilisateurs exposés

Azure AD recommande d’exiger une authentification multifacteur (MFA) de tous vos utilisateurs. Veillez à prendre en considération les utilisateurs titulaires d’un compte dont la compromission aurait un impact significatif (par exemple, des responsables de finances). L’authentification multifacteur réduit le risque d’attaque auquel expose un mot de passe compromis.

Activer :

* [MFA utilisant des stratégies d’accès conditionnel](../authentication/howto-mfa-getstarted.md) pour tous les utilisateurs de votre organisation.

Si vous utilisez Windows Hello Entreprise, l’exigence d’authentification multifacteur peut être satisfaite grâce à l’interface de connexion de Windows Hello. Pour plus d’informations, consultez [Windows Hello](/windows/uwp/security/microsoft-passport).

#### <a name="configure-identity-protection"></a>Configurer la protection des identités

Azure AD Identity Protection est un outil de surveillance et de signalement basé sur un algorithme, qui détecte des vulnérabilités potentielles des identités de votre organisation. Vous pouvez configurer des réponses automatiques aux activités suspectes détectées et prendre les mesures appropriées pour les résoudre. Pour plus d’informations, consultez [Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md).

#### <a name="obtain-your-microsoft-365-secure-score-if-using-microsoft-365"></a>Obtenir votre Niveau de sécurité Microsoft 365 (si vous utilisez Microsoft 365)

Niveau de sécurité examine vos paramètres et activités pour les services Microsoft 365 que vous utilisez, et les compare à une base de référence établie par Microsoft. Vous obtenez un score reflétant votre degré d’alignement sur les meilleures pratiques de sécurité. Toute personne disposant des autorisations d’administrateur pour un abonnement Microsoft 365 Business Standard ou Enterprise peut accéder au Niveau de sécurité en visitant la page [https://securescore.office.com](https://securescore.office.com/).

#### <a name="review-the-microsoft-365-security-and-compliance-guidance-if-using-microsoft-365"></a>Passer en revue l’aide Microsoft 365 en matière de sécurité et de conformité (si vous utilisez Microsoft 365)

Le [plan de sécurité et de conformité](https://support.office.com/article/Plan-for-security-and-compliance-in-Office-365-dc4f704c-6fcc-4cab-9a02-95a824e4fb57) offre une vue d’ensemble de l’approche qu’un client Office 365 doit adopter pour configurer Office 365 et activer d’autres fonctionnalités d’EMS. Passez ensuite en revue les étapes 3 à 6 de la procédure [Protéger l’accès aux données et services dans Microsoft 365](https://support.office.com/article/Protect-access-to-data-and-services-in-Office-365-a6ef28a4-2447-4b43-aae2-f5af6d53c68e) et le guide de procédure [Superviser la sécurité et la conformité dans Microsoft 365](https://support.office.com/article/Monitor-security-and-compliance-in-Office-365-b62f1722-fd39-44eb-8361-da61d21509b6).

#### <a name="configure-microsoft-365-activity-monitoring-if-using-microsoft-365"></a>Configurer la supervision de l’activité Microsoft 365 (si vous utilisez Microsoft 365)

Supervisez les utilisateurs de Microsoft 365 dans votre organisation afin d’identifier les membres du personnel qui disposent d’un compte d’administrateur mais qui n’ont pas besoin d’accéder à Microsoft 365 car ils ne se connectent pas à ces portails. Pour plus d’informations, consultez [Rapports d’activité dans le Centre d’administration Microsoft 365](https://support.office.com/article/Activity-Reports-in-the-Office-365-admin-center-0d6dfb17-8582-4172-a9a9-aed798150263).

#### <a name="establish-incidentemergency-response-plan-owners"></a>Définir des propriétaires de plan de réponse d’incident/d’urgence

La mise en place d’une capacité de réponse aux incidents efficace nécessite une planification et des ressources considérables. Vous devez continuellement surveiller les attaques informatiques et établir des priorités pour la gestion des incidents. Collectez, analysez et signalez les données d’incident pour nouer des relations et entrer en communication avec d’autres groupes et propriétaires de plans internes. Pour plus d’informations, consultez [Centre de réponse aux problèmes de sécurité Microsoft](https://technet.microsoft.com/security/dn440717).

#### <a name="secure-on-premises-privileged-administrative-accounts-if-not-already-done"></a>Sécuriser les comptes d’administration privilégiés locaux (si ce n’est pas déjà fait)

Si votre organisation Azure Active Directory est synchronisée avec un Active Directory local, suivez les conseils de la [Feuille de route de l’accès privilégié sécurisé](/windows-server/identity/securing-privileged-access/securing-privileged-access) : Cette étape comprend les opérations suivants :

* création de comptes administrateur distincts pour les utilisateurs qui doivent effectuer des tâches administratives locales :
* déploiement de stations de travail disposant d’un accès privilégié pour les administrateurs Active Directory ;
* création de mots de passe d’administrateur local uniques pour les stations de travail et les serveurs.

### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>Étapes supplémentaires pour les organisations gérant l’accès à Azure

#### <a name="complete-an-inventory-of-subscriptions"></a>Inventorier les abonnements

Utilisez le portail Enterprise et le portail Azure pour identifier les abonnements de votre organisation qui hébergent des applications de production.

#### <a name="remove-microsoft-accounts-from-admin-roles"></a>Supprimer les comptes Microsoft des rôles Administrateur

Vous ne devez pas utiliser de comptes Microsoft d’autres programmes, tels que Xbox Live et Outlook, en tant que comptes administrateur pour les abonnements de votre organisation. Supprimez le statut administratif de tous les comptes Microsoft, et remplacez-les par des comptes professionnels ou scolaires Azure Active Directory (par exemple, chris@contoso.com). Pour l’administration, servez-vous sur de comptes authentifiés dans Azure AD et non dans d’autres services.

#### <a name="monitor-azure-activity"></a>Surveiller l’activité Azure

Le Journal d’activité Azure fournit un historique des événements au niveau de l’abonnement dans Azure. Il fournit des informations indiquant qui a créé, mis à jour et supprimé quelles ressources et quand ces événements se sont produits. Pour plus d’informations, consultez [Audit et réception de notifications relatives à des actions importantes dans votre abonnement Azure](../../azure-monitor/platform/alerts-activity-log.md).

### <a name="additional-steps-for-organizations-managing-access-to-other-cloud-apps-via-azure-ad"></a>Étapes supplémentaires pour les organisations gérant l’accès à d’autres applications cloud via Azure AD

#### <a name="configure-conditional-access-policies"></a>Configurer des stratégies d’accès conditionnel

Préparez des stratégies d’accès conditionnel pour les applications locales et hébergées dans le cloud. Si vous disposez d’appareils rattachés à l’espace de travail des utilisateurs, obtenez plus d’informations dans [Configuration d’un accès conditionnel en local à l’aide du service Azure Active Directory Device Registration](../../active-directory-b2c/overview.md).

## <a name="stage-3-take-control-of-admin-activity"></a>Étape 3 : Contrôler l’activité administrative

![Étape 3 Contrôler l’activité administrative](./media/security-planning/stage-three.png)

L’étape 3 s’ajoute aux atténuations de l’étape 2 et doit être implémentée dans un délai approximatif de 1 à 3 mois. Cette étape de la feuille de route d’accès privilégié sécurisé comprend les composants suivants.

### <a name="general-preparation"></a>Préparation générale

#### <a name="complete-an-access-review-of-users-in-administrator-roles"></a>Effectuer une vérification de l’accès des utilisateurs de rôles d’administrateur

Le fait que de plus en plus d’utilisateurs en entreprise bénéficient d’un accès privilégié via des services cloud peut conduire à une plateforme non managée. Aujourd’hui, des utilisateurs peuvent devenir des administrateurs généraux pour Microsoft 365, des administrateurs d’abonnements Azure, ou disposer d’un accès administrateur à des machines virtuelles ou via des applications SaaS.

Votre organisation doit amener tous les employés à gérer les transactions commerciales ordinaires en tant qu’utilisateurs non privilégiés, et ne leur accorder de droits d’administrateur qu’en cas de besoin. Procédez à des révisions d’accès pour identifier et confirmer les utilisateurs éligibles pour activer des privilèges d’administrateur.

Nous vous recommandons :

1. Déterminez quels utilisateurs sont des administrateurs Azure AD, accordez un accès administrateur à la demande juste-à-temps, ainsi que les contrôles de sécurité en fonction du rôle.
2. Affectez un autre rôle aux utilisateurs ne disposant d’aucune justification précise à l’accès administrateur (si aucun rôle n’est éligible, supprimez-les).

#### <a name="continue-rollout-of-stronger-authentication-for-all-users"></a>Poursuivre le lancement d’une authentification renforcée pour tous les utilisateurs

Demandez aux utilisateurs fortement exposés de recourir à une authentification forte et moderne, telle qu’Azure AD MFA ou Windows Hello. Voici quelques exemples d’utilisateurs fortement exposés :

* Cadres supérieurs
* Responsables de haut niveau
* Personnel informatique et de sécurité critique

#### <a name="use-dedicated-workstations-for-administration-for-azure-ad"></a>Utiliser des stations de travail dédiées pour l’administration d’Azure AD

Des attaquants peuvent tenter de cibler des comptes privilégiés et perturber l’intégrité et l’authenticité des données. Ils utilisent souvent du code malveillant qui modifie la logique du programme ou espionne l’administrateur quand celui-ci entre des informations d’identification. Les stations de travail d’accès privilégié (PAW) fournissent un système d’exploitation dédié aux tâches sensibles et protégé contre les attaques Internet et les vecteurs de menaces. La séparation de ces tâches et comptes sensibles des stations de travail et appareils utilisés au quotidien offre une protection renforcée contre les risques suivants :

* attaques par hameçonnage ;
* vulnérabilités du système d’exploitation et des applications ;
* attaques par emprunt d’identité ;
* attaques visant à dérober des informations d’identification, par exemple en enregistrant les frappes, et autres attaques de type « pass-The-Hash » et « pass-the-ticket ».

En déployant des stations de travail dédiées à un accès privilégié, vous pouvez réduire le risque que des administrateurs saisissent leurs informations d’identification dans un environnement de bureau dont la sécurité n’a pas été renforcée. Pour plus d’informations, consultez [Stations de travail d’accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations).

#### <a name="review-national-institute-of-standards-and-technology-recommendations-for-handling-incidents"></a>Passer en revue les recommandations relatives à la gestion des incidents du National Institute of Standards and Technology

Le National Institute of Standards and Technology (NIST) fournit des instructions pour la gestion des incidents, et plus particulièrement pour l’analyse des données relatives aux incidents et détermination de la réponse adéquate à chaque incident. Pour plus d’informations, consultez le document [(NIST) Computer Security Incident Handling Guide (SP 800-61, Revision 2)](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf) (Guide de la gestion des incidents de sécurité informatique du NIST (SP 800-61, Révision 2)).

#### <a name="implement-privileged-identity-management-pim-for-jit-to-additional-administrative-roles"></a>Implémenter Privileged Identity Management (PIM) pour JIT sur des rôles administratifs supplémentaires

Pour Azure Active Directory, utilisez la fonctionnalité [Gestion des identités privilégiées Azure Active Directory](../privileged-identity-management/pim-configure.md). L’activation à durée limitée de rôles privilégiés vous permet de :

* Activer des privilèges administratifs pour effectuer une tâche spécifique
* Appliquer MFA pendant le processus d’activation
* Utiliser des alertes pour informer les administrateurs de modifications hors-bande
* Permettre à des utilisateurs de conserver leur accès privilégié pendant une période préconfigurée
* Autoriser les administrateurs de la sécurité à :

  * Détecter toutes les identités privilégiées
  * Consulter les rapports d’audit
  * Créer des révisions d’accès afin d’identifier tous les utilisateurs éligibles pour l’activation de privilèges d’administrateur

Si vous utilisez déjà Azure AD Privileged Identity Management, ajustez la plage de temps des privilèges à durée limitée si nécessaire (par exemple, les fenêtres de maintenance).

#### <a name="determine-exposure-to-password-based-sign-in-protocols-if-using-exchange-online"></a>Déterminer l’exposition à des protocoles de connexion basés sur un mot de passe (si vous utilisez Exchange Online)

Nous vous recommandons d’identifier tous les utilisateurs détenteurs d’informations d’identification dont la compromission pourrait nuire à l’organisation. Pour ces utilisateurs, instaurez des exigences d’authentification fortes et utilisez un accès conditionnel Azure AD pour les empêcher de se connecter à leur e-mail à l’aide d’un nom d’utilisateur et d’un mot de passe. Vous pouvez [bloquer l’authentification héritée en instaurant un accès conditionnel](../conditional-access/block-legacy-authentication.md), et [bloquer l’authentification de base](/exchange/clients-and-mobile-in-exchange-online/disable-basic-authentication-in-exchange-online) via Exchange Online.

#### <a name="complete-a-roles-review-assessment-for-microsoft-365-roles-if-using-microsoft-365"></a>Effectuer une évaluation de révision des rôles pour les rôles Microsoft 365 (si vous utilisez Microsoft 365)

Évaluez si tous les utilisateurs administrateurs se trouvent dans les rôles appropriés (supprimez et réaffectez en fonction de cette évaluation).

#### <a name="review-the-security-incident-management-approach-used-in-microsoft-365-and-compare-with-your-own-organization"></a>Passer en revue l’approche de gestion des incidents de sécurité utilisée dans Microsoft 365 et comparer avec votre organisation

Vous pouvez télécharger ce rapport à partir de [Gestion des incidents de sécurité dans Microsoft 365](https://www.microsoft.com/download/details.aspx?id=54302).

#### <a name="continue-to-secure-on-premises-privileged-administrative-accounts"></a>Continuer à sécuriser les comptes d’administration privilégiés locaux

Si votre Azure Active Directory est connecté à Active Directory en local, suivez les conseils de la [Feuille de route de l’accès privilégié sécurisé](/windows-server/identity/securing-privileged-access/securing-privileged-access) : étape 2. Dans le cadre de cette étape, vous allez effectuer les opérations suivantes :

* Déployer des stations de travail à accès privilégié pour tous les administrateurs
* Exiger une authentification multifacteur
* Utiliser juste assez d’administration pour la maintenance du contrôleur de domaine, ce qui réduit la surface d’attaque des domaines
* Mettre en place une évaluation avancée des menaces pour la détection des attaques

### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>Étapes supplémentaires pour les organisations gérant l’accès à Azure

#### <a name="establish-integrated-monitoring"></a>Définir une surveillance intégrée

L’[Azure Security Center](../../security-center/security-center-introduction.md) :

* Assure une gestion intégrée de la stratégie et de la surveillance de la sécurité dans vos abonnements Azure.
* Aide à détecter les menaces qui pourraient passer inaperçues.
* Fonctionne avec un vaste éventail de solutions de sécurité.

#### <a name="inventory-your-privileged-accounts-within-hosted-virtual-machines"></a>Inventorier vos comptes privilégiés dans des machines virtuelles hébergées

Le plus souvent, vous n’avez pas besoin d’accorder à des utilisateurs des autorisations d’accès illimitées à l’ensemble de vos abonnements ou ressources Azure. Utilisez des rôles d’administrateur Azure AD pour accorder à vos utilisateurs uniquement l’accès dont ils ont besoin pour accomplir leur travail. Par exemple, vous pouvez utiliser des rôles d’administrateur Azure AD pour permettre à un administrateur de gérer uniquement les machines virtuelles dans un abonnement, et à un autre de gérer uniquement les bases de données SQL au sein du même abonnement. Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure](../../active-directory-b2c/overview.md).

#### <a name="implement-pim-for-azure-ad-administrator-roles"></a>Implémenter PIM pour les rôles d’administrateur Azure AD

Utilisez Privileged Identity Management avec des rôles d’administrateur Azure AD pour gérer, contrôler et surveiller l’accès aux ressources Azure. PIM protège en réduisant le temps d’exposition des privilèges et en augmentant votre visibilité sur leur utilisation grâce à des rapports et alertes. Pour plus d’informations, consultez [Qu’est-ce qu’Azure AD Privileged Identity Management ?](../privileged-identity-management/pim-configure.md).

#### <a name="use-azure-log-integrations-to-send-relevant-azure-logs-to-your-siem-systems"></a>Utiliser des intégrations des journaux d’activité Azure pour envoyer les journaux d’activité Azure pertinents à vos systèmes SIEM

L’intégration des journaux d’activité Azure permet d’intégrer des journaux d’activité bruts de vos ressources Azure dans les systèmes SIEM (Security Information and Event Management) existants de votre organisation. [Azure log Integration](/previous-versions/azure/security/fundamentals/azure-log-integration-overview) collecte des événements Windows à partir de journaux de l’observateur d’événements Windows et de événements Azure à partir des ressources suivantes :

* Journaux d’activité Azure
* Alertes Azure Security Center
* Journaux de ressources Azure

### <a name="additional-steps-for-organizations-managing-access-to-other-cloud-apps-via-azure-ad"></a>Étapes supplémentaires pour les organisations gérant l’accès à d’autres applications cloud via Azure AD

#### <a name="implement-user-provisioning-for-connected-apps"></a>Implémenter l’approvisionnement de l’utilisateur pour des applications connectées

Azure AD vous permet d’automatiser la création et la gestion des identités utilisateurs dans des applications cloud telles que Dropbox, Salesforce et ServiceNow. Pour plus d’informations, consultez [Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

#### <a name="integrate-information-protection"></a>Intégrer la protection des informations

Microsoft Cloud App Security vous permet d’analyser des fichiers et de définir des stratégies basées sur des étiquettes de classification d’Azure Information Protection, offrant ainsi une visibilité et un contrôle accrus de vos données dans le cloud. Analysez et classez des fichiers dans le cloud et apposez des étiquettes de protection des informations Azure. Pour plus d’informations, consultez [Azure Information Protection integration](/cloud-app-security/azip-integration) (Intégration d’Azure Information Protection).

#### <a name="configure-conditional-access"></a>Configurer un accès conditionnel

Configurez un accès conditionnel en fonction du groupe, de l’emplacement et du niveau de confidentialité des [applications SaaS](https://azure.microsoft.com/overview/what-is-saas/) et des applications connectées à Azure AD. 

#### <a name="monitor-activity-in-connected-cloud-apps"></a>Surveiller l’activité dans les applications cloud connectées

Nous vous recommandons d’utiliser [Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security) pour garantir que l’accès utilisateur est protégé également dans les applications connectées. Cette fonctionnalité sécurise l’accès de l’entreprise aux applications cloud et sécurise vos comptes administrateur, ce qui vous permet d’effectuer les opérations suivantes :

* Étendre la visibilité et le contrôle aux applications cloud
* Créer des stratégies d’accès, des activités et le partage de données
* Identifier automatiquement les activités à risque, les comportements anormaux et les menaces
* Empêcher la fuite de données
* Réduire les risques et prévention des menaces et application de stratégies automatisées

L’agent Cloud App Security SIEM s’intègre à Cloud App Security avec votre serveur SIEM pour permettre la supervision centralisée des alertes et des activités de Microsoft 365. Il s’exécute sur votre serveur et transmet les alertes et les activités de Cloud App Security au serveur SIEM. Pour plus d’informations, consultez [SIEM integration](/cloud-app-security/siem) (Intégration de SIEM).

## <a name="stage-4-continue-building-defenses"></a>Étape 4 : Continuer à créer des défenses

![Étape 4 : Adopter une posture de sécurité proactive](./media/security-planning/stage-four.png)

L’Étape 4 de la feuille de route doit être implémentée à partir du sixième mois. Complétez votre feuille de route pour renforcer vos protections des accès privilégiés contre des attaques potentielles connues aujourd’hui. Pour les menaces de sécurité de demain, nous vous recommandons de considérer la sécurité comme un processus continu visant à augmenter les coûts et à réduire le taux de réussite des adversaires ciblant votre environnement.

La sécurisation des accès privilégiés est importante pour offrir des assurances en matière de sécurité pour vos ressources d’entreprise. Elle doit cependant d’inscrire dans un programme de sécurité complet qui offre des assurances de sécurité continues. Ce programme doit inclure des éléments tels que les suivants :

* Policy
* Opérations
* Sécurité des informations
* Serveurs
* Applications
* PC
* Appareils
* Infrastructure cloud

Pour gérer les comptes d’accès privilégiés, nous vous recommandons d’adopter les pratiques suivantes :

* Assurez-vous que les administrateurs accomplissent leurs tâches quotidiennes en tant qu’utilisateurs sans privilèges.
* N’accordez que l’accès privilégié nécessaire, et retirez-le ensuite (juste-à-temps).
* Conservez les journaux d’activité d’audit relatifs aux comptes privilégiés.

Pour plus d’informations sur la création d’une feuille de route de sécurité complète, consultez les [ressources d’architecture informatique cloud de Microsoft](https://almbok.com/office365/microsoft_cloud_it_architecture_resources). Pour demander aux services Microsoft de vous aider à implémenter une partie quelconque de votre feuille de route, contactez votre représentant Microsoft ou consultez [Créer des cyber-défenses critiques pour protéger votre entreprise](https://www.microsoft.com/en-us/microsoftservices/campaigns/cybersecurity-protection.aspx).

Cette dernière étape de la feuille de route d’accès privilégié sécurisé comprend les composants suivants.

### <a name="general-preparation"></a>Préparation générale

#### <a name="review-admin-roles-in-azure-ad"></a>Examiner les rôles d’administrateur dans Azure AD

Déterminez si les rôles d’administrateur Azure AD intégrés actuels sont toujours à jour et assurez-vous que les utilisateurs ne sont assignés qu’à ceux dont ils ont besoin. Azure AD vous permet d’affecter des administrateurs distincts à l’exercice de fonctions différentes. Pour plus d’informations, consultez [Attribution de rôles d’administrateur dans Azure Active Directory](permissions-reference.md).

#### <a name="review-users-who-have-administration-of-azure-ad-joined-devices"></a>Passer en revue les utilisateurs administrateurs d’appareils joints Azure AD

Pour plus d’informations, consultez [Comment configurer des appareils hybrides joints à Azure Active Directory](../devices/hybrid-azuread-join-plan.md).

#### <a name="review-members-of-built-in-microsoft-365-admin-roles"></a>Passer en revue les membres disposant de [rôles d’administrateur Microsoft 365 intégrés](https://support.office.com/article/About-Office-365-admin-roles-da585eea-f576-4f55-a1e0-87090b6aaa9d)
Ignorez cette étape si vous n’utilisez pas Microsoft 365.
‎
#### <a name="validate-incident-response-plan"></a>Valider le plan de réponse aux incidents

Pour améliorer votre plan, Microsoft vous recommande de vérifier régulièrement que votre plan fonctionne comme prévu :

* Parcourir votre feuille de route pour voir ce qui n’a pas fonctionné
* Sur la base de l’analyse post-mortem, révisez les pratiques actuelles ou définissez-en de nouvelles.
* Assurez-vous que votre plan de réponse aux incidents mis à jour et les pratiques sont distribués dans l’ensemble de votre organisation.


### <a name="additional-steps-for-organizations-managing-access-to-azure"></a>Étapes supplémentaires pour les organisations gérant l’accès à Azure 

Déterminez si vous devez [transférer la propriété d’un abonnement Azure à un autre compte](../../cost-management-billing/manage/billing-subscription-transfer.md).
‎

## <a name="break-glass-what-to-do-in-an-emergency"></a>« Secours » : que faire en cas d’urgence

![Comptes pour l’accès de secours d’urgence](./media/security-planning/emergency.jpeg)

1. Fournissez aux gestionnaires et responsables de la sécurité clés des informations sur l’incident.

2. Passez en revue votre manuel d’attaque.

3. Utilisez les nom d’utilisateur et mot de passe de votre compte d’urgence pour vous connecter à Azure AD.

4. Obtenez de l’aide de Microsoft en [ouvrant une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md).

5. Examinez les [rapports de connexion Azure AD](../reports-monitoring/overview-reports.md). Un certain temps peut s’écouler entre la survenance d’un événement et le moment où celui-ci est inclus dans le rapport.

6. Pour les environnements hybrides, si votre infrastructure locale fédérée et votre serveur AD FS ne sont pas disponible, vous pouvez passer temporairement de l’authentification fédérée à l’utilisation de la synchronisation du hachage de mot de passe. Cela permet de revenir de la fédération de domaine à une authentification managée jusqu’à ce que le serveur AD FS soit disponible.

7. Surveillez les courriers électroniques des comptes privilégiés.

8. Veillez à effectuer des sauvegardes des journaux d’activité concernés en vue d’un éventuel examen légal et plus approfondi.

Pour plus d’informations sur la façon dont Microsoft Office 365 gère les incidents de sécurité, consultez [Security Incident Management in Microsoft Office 365](https://aka.ms/Office365SIM) (Gestion des incidents de sécurité dans Microsoft Office 365).

## <a name="faq-answers-for-securing-privileged-access"></a>Questions fréquentes : Réponses aux questions relatives à la sécurisation de l’accès privilégié  

**Q :** Que faire si je n’ai pas encore implémenté de composants d’accès sécurisé ?

**Réponse :** Définissez au minimum deux comptes de secours, affectez MFA à vos comptes administrateur privilégiés et séparez les comptes utilisateurs des comptes administrateur général.

**Q :** Après une violation, quel est le problème à traiter en premier ?

**Réponse :** Veillez à exiger l’authentification la plus stricte des personnes hautement exposées.

**Q :** Que se passe-t-il si nos administrateurs privilégiés ont été désactivés ?

**Réponse :** Créez un compte administrateur général tenu à jour en permanence.

**Q :** Que se passe-t-il s’il ne reste qu’un seul administrateur général et que celui-ci est inaccessible ?

**Réponse :** Utilisez un de vos comptes de secours pour obtenir un accès privilégié immédiat.

**Q :** Comment protéger les administrateurs au sein de mon organisation ?

**Réponse :** Les administrateurs doivent toujours effectuer leurs tâches quotidiennes en tant qu’utilisateurs « non privilégiés » standard.

**Q :** Quelles sont les bonnes pratiques pour créer des comptes administrateur dans Azure AD ?

**Réponse :** Réservez l’accès privilégié pour des tâches administratives spécifiques.

**Q :** Quels outils permettent de réduire l’accès administrateur permanent ?

**Réponse :** Privileged Identity Management (PIM) et les rôles d’administrateur Azure AD.

**Q :** Quelle est la position de Microsoft sur la synchronisation des comptes administrateur avec Azure AD ?

**Réponse :** Des comptes administrateur de niveau 0 sont utilisés uniquement pour les comptes AD locaux. Ces comptes ne sont généralement pas synchronisés avec Azure AD dans le cloud. Les comptes administrateur de niveau 0 incluent les comptes, groupes et autres ressources exerçant un contrôle administratif direct ou indirect sur la forêt, les domaines, les contrôleurs de domaine et les ressources Active Directory locaux.

**Q :** Comment empêcher les administrateurs d’accorder un accès administrateur aléatoire dans le portail ?

**Réponse :** Utilisez des comptes non privilégiés pour tous les utilisateurs et la plupart des administrateurs. Commencez par définir l’empreinte de l’organisation afin de déterminer les comptes administrateur qui doivent être privilégiés. Surveillez également les nouveaux utilisateurs administratifs créés.

## <a name="next-steps"></a>Étapes suivantes

* [Centre de gestion de la confidentialité Microsoft - Sécurité produit](https://www.microsoft.com/trustcenter/security) : fonctionnalités de sécurité des produits et services cloud de Microsoft

* [Centre de gestion de la confidentialité Microsoft - Conformité](https://www.microsoft.com/trustcenter/compliance/complianceofferings) : ensemble complet des offres de conformité de Microsoft pour les services cloud

* [Conseils sur la conduite d’une évaluation des risques](https://www.microsoft.com/trustcenter/guidance/risk-assessment) : gérez les exigences de sécurité et de conformité pour les services de cloud computing Microsoft

### <a name="other-microsoft-online-services"></a>Autres services Microsoft Online Services

* [Microsoft Intune Security](https://www.microsoft.com/trustcenter/security/intune-security) : Intune fournit des fonctionnalités de gestion des appareils mobiles, de gestion des applications mobiles et de gestion des ordinateurs à partir du cloud.

* [Sécurité de Microsoft Dynamics 365](https://www.microsoft.com/trustcenter/security/dynamics365-security) : Dynamics 365 est la solution cloud de Microsoft qui centralise les fonctions de gestion de la relation client (CRM) et de planification des ressources d’entreprise (ERP).
