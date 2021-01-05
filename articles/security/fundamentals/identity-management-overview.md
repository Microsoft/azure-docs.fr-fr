---
title: Fonctionnalités de sécurité Azure liées à la gestion des identités | Microsoft Docs
description: Découvrez les principales fonctionnalités de sécurité Azure qui facilitent la gestion des identités. Consultez des informations sur des sujets tels que l’authentification unique et le proxy inverse.
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 5aa0a7ac-8f18-4ede-92a1-ae0dfe585e28
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/19/2018
ms.author: terrylan
Customer intent: As an IT Pro or decision maker I am trying to learn about identity management capabilities in Azure
ms.openlocfilehash: 2c7d51da361816841ff556c1da696a48d2ce6a06
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843687"
---
# <a name="azure-identity-management-security-overview"></a>Vue d’ensemble de la sécurité et de la gestion des identités Azure

 La gestion des identités est le processus d’authentification et d’autorisation des [principaux de sécurité](/windows/security/identity-protection/access-control/security-principals). Elle implique également le contrôle des informations relatives à ces principaux (identités). Les principaux de sécurité (identités) peuvent inclure des services, des applications, des utilisateurs, des groupes, etc. Les solutions de gestion des identités et des accès de Microsoft aident les services informatiques à protéger l’accès aux applications et aux ressources dans le centre de données de l’entreprise, mais aussi dans le cloud. Cette protection offre des niveaux supplémentaires de validation, notamment l’authentification multifacteur et les stratégies d’accès conditionnel. En surveillant les activités suspectes via les fonctions avancées de reporting, d’audit et d’alertes de sécurité, vous êtes en mesure de limiter les problèmes de sécurité potentiels. [Azure Active Directory Premium](../../active-directory/fundamentals/active-directory-whatis.md) fournit une authentification unique (SSO) à des milliers d’applications cloud Software as a Service (SaaS) et assure un accès aux applications web que vous exécutez en local.
 
Azure Active Directory (Azure AD) vous offre de nombreux avantages en termes de sécurité :

* Création et gestion d’une identité unique pour chaque utilisateur de l’entreprise hybride, tout en maintenant la synchronisation des utilisateurs, des groupes et des appareils 
* Accès par authentification unique (SSO) à vos applications, notamment à des milliers d’applications SaaS pré-intégrées
* Sécurisation de l’accès aux applications en appliquant une authentification multifacteur basée sur des règles pour les applications aussi bien locales que cloud
* Accès à distance sécurisé aux applications web locales via le proxy d’application Azure AD

Cet article vise à fournir une vue d’ensemble des principales fonctionnalités de sécurité Azure liées à la gestion des identités. Il contient également des liens vers des articles fournissant des informations complémentaires sur chaque fonctionnalité.  

Cet article se concentre sur les principales fonctionnalités de gestion d’identité Azure, à savoir :

* Authentification unique
* Proxy inversé
* Azure Multi-Factor Authentication
* Contrôle d’accès en fonction du rôle Azure (Azure RBAC)
* Surveillance de la sécurité, alertes et rapports Machine Learning
* Gestion des identités et des accès des consommateurs
* Enregistrement de l’appareil
* Privileged Identity Management
* Identity Protection
* Gestion des identités hybrides/Azure AD Connect
* Révisions d’accès Azure AD

## <a name="single-sign-on"></a>Authentification unique

Avec SSO, vous pouvez accéder à toutes les applications et à toutes les ressources dont vous avez besoin pour travailler, en vous connectant une seule fois avec un seul compte d’utilisateur. Une fois connecté, vous pouvez accéder à toutes les applications dont vous avez besoin sans devoir vous authentifier à nouveau (par exemple, taper un mot de passe).

De nombreuses entreprises s’appuient sur des applications SaaS, comme Microsoft 365, Box et Salesforce, pour accroître la productivité de l’utilisateur. Historiquement, le personnel informatique devait créer et mettre à jour chaque compte d’utilisateur dans chaque application SaaS et les utilisateurs devaient mémoriser un mot de passe pour chaque application SaaS.

Azure AD étend les environnements Active Directory locaux dans le cloud, ce qui permet aux utilisateurs d’utiliser leur compte professionnel principal, non seulement pour se connecter à leurs appareils liés au domaine et aux ressources de l’entreprise, mais aussi à toutes les applications SaaS et web nécessaires à leurs travaux.

Non seulement les utilisateurs n’ont plus besoin de gérer plusieurs noms d’utilisateur et mots de passe, mais vous pouvez activer ou désactiver automatiquement l’accès aux applications en fonction des groupes de l’organisation et de leur statut d’employé. Azure AD ajoute des contrôles de sécurité et de gouvernance de l’accès qui vous permettent de gérer de manière centralisée l’accès des utilisateurs sur les différentes applications SaaS.

En savoir plus :

* [Présentation de l’authentification unique](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/)
* [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure Active Directory ?](../../active-directory/manage-apps/what-is-single-sign-on.md)
* [Série de démarrages rapides sur la gestion des applications](../../active-directory/manage-apps/view-applications-portal.md)

## <a name="reverse-proxy"></a>Proxy inversé

Le proxy d’application Azure AD vous permet de publier des applications en local, telles que des sites [SharePoint](https://support.office.com/article/What-is-SharePoint-97b915e6-651b-43b2-827d-fb25777f446f?ui=en-US&rs=en-US&ad=US), des [applications web Outlook](/Exchange/clients/outlook-on-the-web/outlook-on-the-web) et des applications [IIS](https://www.iis.net/) à l’intérieur de votre réseau privé et offre un accès sécurisé aux utilisateurs en dehors de votre réseau. Le proxy d’application assure l’accès à distance et l’authentification unique (SSO) pour de nombreux types d’applications web locales, avec les milliers d’applications SaaS prises en charge par Azure AD. Les employés peuvent se connecter à vos applications depuis leur domicile sur leurs propres appareils et s’authentifier par le biais de ce proxy cloud.

En savoir plus :

* [Activation du proxy d’application Azure AD](../../active-directory/manage-apps/application-proxy-add-on-premises-application.md)
* [Publier des applications avec le Proxy d’application Azure AD](../../active-directory/manage-apps/application-proxy-add-on-premises-application.md)
* [Authentification unique avec le proxy d’application](../../active-directory/manage-apps/application-proxy-configure-single-sign-on-with-kcd.md)
* [Utilisation de l’accès conditionnel](../../active-directory/manage-apps/application-proxy-integrate-with-sharepoint-server.md)

## <a name="multi-factor-authentication"></a>Azure Multi-Factor Authentication

Azure AD Multi-Factor Authentication est une méthode d'authentification qui nécessite l'utilisation de plusieurs méthodes de vérification et ajoute une deuxième couche critique de sécurité aux connexions et transactions des utilisateurs. Azure Multi-Factor Authentication contribue à sécuriser l’accès aux données et aux applications, tout en proposant un processus d’authentification simple et conforme à la demande des utilisateurs. Cette méthode fournit une authentification forte par le biais de diverses options de vérification : appels téléphoniques, SMS, notifications par application mobile ou codes de vérification et jetons OAuth tiers.

En savoir plus :

* [Azure Multi-Factor Authentication](https://azure.microsoft.com/documentation/services/multi-factor-authentication/)
* [Présentation d'Azure AD Multi-Factor Authentication](../../active-directory/authentication/concept-mfa-howitworks.md)
* [Comment fonctionne l’authentification multifacteur Azure AD](../../active-directory/authentication/concept-mfa-howitworks.md)

## <a name="azure-rbac"></a>Azure RBAC

Azure RBAC est un système d’autorisation basé sur Azure Resource Manager qui propose une gestion affinée des accès aux ressources dans Azure. Il vous permet de contrôler de façon granulaire le niveau d’accès dont disposent les utilisateurs. Par exemple, vous pouvez décider d’autoriser un utilisateur à gérer uniquement les réseaux virtuels et un autre utilisateur à gérer toutes les ressources d’un groupe de ressources. Azure inclut plusieurs rôles intégrés que vous pouvez utiliser. Voici les quatre rôles fondamentaux intégrés : Les trois premiers s’appliquent à tous les types de ressources.

- [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) : dispose d’un accès total à toutes les ressources, ainsi que le droit de déléguer l’accès à d’autres personnes. 
- [Contributeur](../../role-based-access-control/built-in-roles.md#contributor) : peut créer et gérer tous les types de ressource Azure mais ne peut pas octroyer l’accès à d’autres personnes.
- [Lecteur](../../role-based-access-control/built-in-roles.md#reader) : peut consulter les ressources Azure existantes.
- [Administrateur de l’accès utilisateur](../../role-based-access-control/built-in-roles.md#user-access-administrator) : vous permet de gérer l’accès des utilisateurs aux ressources Azure.

En savoir plus :

* [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (RBAC Azure) ?](../../role-based-access-control/overview.md)
* [Rôles intégrés Azure](../../role-based-access-control/built-in-roles.md)

## <a name="security-monitoring-alerts-and-machine-learning-based-reports"></a>Surveillance de la sécurité, alertes et rapports Machine Learning

Vous pouvez protéger votre entreprise grâce à la surveillance de la sécurité, aux alertes et aux rapports Machine Learning qui identifient les comportements d’accès incohérents. Vous pouvez utiliser les rapports d’accès et d’utilisation Azure AD pour obtenir une visibilité complète sur l’intégrité et la sécurité du répertoire de votre société. Grâce à ces informations, un administrateur de répertoire est capable de déterminer plus précisément les risques de sécurité potentiels et donc de les atténuer au maximum.

Dans le portail Azure, les rapports sont classés dans les catégories suivantes :

* **Rapports d’anomalies** : contiennent des événements de connexion qui peuvent nous sembler anormaux. Notre objectif est de vous faire part de ces activités et de vous permettre de décider si un événement est suspect.
* **Rapports d’application intégrée** : fournissent des insights sur l’utilisation des applications cloud dans votre organisation. Azure AD permet d’intégrer des milliers d'applications du cloud.
* **Rapports d’erreurs** : indiquent les erreurs qui peuvent survenir quand vous provisionnez des comptes sur des applications externes.
* **Rapports spécifiques à l’utilisateur** : affichent les données d’activité relatives aux connexions de l’appareil d’un utilisateur spécifique.
* **Journaux d’activité** : contiennent un enregistrement de tous les événements audités durant les dernières 24 heures, les derniers 7 jours ou les derniers 30 jours, des modifications d’activité de groupes, et des activités d’enregistrement et de réinitialisation de mot de passe.

En savoir plus :

* [Affichage de vos rapports d’accès et d’utilisation](../../active-directory/reports-monitoring/overview-reports.md)
* [Prise en main de la création de rapports Azure Active Directory](../../active-directory/reports-monitoring/overview-reports.md)
* [Guide Azure Active Directory Reporting Guide](../../active-directory/reports-monitoring/overview-reports.md)

## <a name="consumer-identity-and-access-management"></a>Gestion des identités et des accès des consommateurs

Azure AD B2C est un service de gestion de l’identité, global et hautement disponible pour les applications destinées aux consommateurs gérant des centaines de millions d’identités. Le service peut être intégré sur l’ensemble des plateformes web et mobiles. Vos consommateurs peuvent se connecter à toutes vos applications par le biais d’expériences personnalisables en utilisant leurs comptes de réseaux sociaux existants ou en créant des comptes avec de nouvelles informations d’identification.

Auparavant, les développeurs d’applications qui souhaitaient inscrire et connecter les clients à leurs applications auraient écrit leur propre code. Et ils auraient utilisé des systèmes ou des bases de données locaux pour stocker les noms d'utilisateur et les mots de passe. Azure AD B2C offre à votre organisation un meilleur moyen d’intégrer la gestion des identités des consommateurs dans vos applications grâce à une plateforme sécurisée reposant sur des normes et à un ensemble complet de stratégies extensibles.

Lorsque vous utilisez Azure AD B2C, vos consommateurs peuvent s’inscrire auprès de vos applications à l’aide de leurs comptes sociaux existants (Facebook, Google, Amazon, LinkedIn) ou en créant des informations d’identification (adresse de messagerie et mot de passe, ou nom d’utilisateur et mot de passe).

En savoir plus :

* [Qu’est-ce qu’Azure Active Directory B2C ?](https://azure.microsoft.com/services/active-directory-b2c/)
* [Azure Active Directory B2C (préversion) : inscription et connexion de consommateurs à vos applications](../../active-directory-b2c/overview.md)
* [Azure Active Directory B2C (préversion) : types d’applications](../../active-directory-b2c/application-types.md)

## <a name="device-registration"></a>Enregistrement de l’appareil

L’inscription d’appareil Azure AD constitue la base des scénarios d’[accès conditionnel](../../active-directory/devices/device-management-azure-portal.md) basé sur les appareils. Lors de l’inscription d’un appareil, Azure AD Device Registration fournit une identité à l’appareil, qui sera utilisée pour l’authentifier lors de la connexion d’un utilisateur. L’appareil authentifié et ses attributs peuvent alors être utilisés pour appliquer des stratégies d’accès conditionnel pour les applications qui sont hébergées sur le cloud et localement.

Quand ils sont associés à une solution de gestion des périphériques mobiles comme Intune, les attributs de l’appareil dans Azure AD sont mis à jour avec des informations supplémentaires sur l’appareil. Vous pouvez ainsi créer des règles d’accès conditionnel qui imposent que l’accès à partir des appareils réponde à vos critères de sécurité et de conformité.

En savoir plus :

* [Prise en main du service Azure AD Device Registration](../../active-directory/devices/device-management-azure-portal.md)
* [Inscription automatique auprès d’Azure AD d’appareils Windows joints à un domaine](../../active-directory/devices/hybrid-azuread-join-plan.md)
* [Configuration de l’inscription automatique auprès d’Azure AD d’appareils Windows joints à un domaine](../../active-directory/devices/hybrid-azuread-join-plan.md)

## <a name="privileged-identity-management"></a>Privileged Identity Management

Le service Azure AD Privileged Identity Management vous permet de gérer, de contrôler et de surveiller vos identités privilégiées et l’accès aux ressources dans Azure AD et dans d’autres services en ligne Microsoft, tels que Microsoft 365 et Microsoft Intune.

Les utilisateurs doivent parfois effectuer des opérations privilégiées dans des ressources Azure ou Microsoft 365, ou dans d'autres applications SaaS. Cela signifie souvent que les entreprises doivent leur donner un accès privilégié permanent à Azure AD. Un tel accès constitue un risque de sécurité croissant pour les ressources hébergées dans le cloud, car les entreprises ne peuvent pas suffisamment surveiller ce que ces utilisateurs font avec leurs privilèges d'administrateur. En outre, si un compte d’utilisateur disposant d’un accès privilégié est compromis, cette seule faille peut affecter la sécurité globale du cloud de l’organisation. Azure AD Privileged Identity Management contribue à minimiser ce risque.

Grâce à Azure AD Privileged Identity Management, vous pouvez :

* Identifier les utilisateurs qui ont un rôle d’administrateur dans Azure AD.
* Activer à la demande un accès administrateur « juste à temps » (JIT) aux services Microsoft comme Microsoft 365 et Intune.
* Obtenir des rapports sur l'historique des accès administrateur et sur les modifications apportées aux affectations de l'administrateur.
* Recevoir des alertes sur l'accès à un rôle privilégié.

En savoir plus :

* [Qu’est-ce qu’Azure AD Privileged Identity Management ?](../../active-directory/privileged-identity-management/pim-configure.md)
* [Attribuer des rôles d’annuaire Azure AD dans PIM](../../active-directory/privileged-identity-management/pim-how-to-add-role-to-user.md)

## <a name="identity-protection"></a>Identity Protection

Azure AD Identity Protection est un service de sécurité offrant une vue centralisée des détections des risques et des vulnérabilités potentielles qui affectent les identités de votre organisation. Identity Protection tire parti des fonctions de détection d’anomalie Azure AD existantes, disponibles via les rapports d’activités anormales Azure AD. Identity Protection inclut également de nouveaux types de détection des risques capables de détecter les anomalies en temps réel.

En savoir plus :

* [Azure AD Identity Protection](../../active-directory/identity-protection/overview-identity-protection.md)
* [Channel 9: Azure AD and Identity Show: Identity Protection Preview](https://channel9.msdn.com/Series/Azure-AD-Identity/Azure-AD-and-Identity-Show-Identity-Protection-Preview)

## <a name="hybrid-identity-managementazure-ad-connect"></a>Gestion des identités hybrides/Azure AD Connect

Les solutions d'identité de Microsoft regroupent des fonctionnalités, locales et cloud, de création d'une identité d'utilisateur unique pour l'authentification et l'autorisation d'accès à toutes les ressources, indépendamment de l'emplacement. Nous appelons cette identité « identité hybride ». L’outil Microsoft Azure AD Connect a été conçu pour vous permettre d’atteindre et de remplir vos objectifs en matière d’identité hybride. Cela vous permet de fournir une identité commune à vos utilisateurs pour les applications Microsoft 365, Azure et SaaS intégrées à Azure AD. Elle fournit les fonctionnalités suivantes :

* Synchronization
* Intégration AD FS et de fédération
* Authentification directe
* Surveillance de l’intégrité

En savoir plus :

* [Livre blanc sur les identités hybrides](https://download.microsoft.com/download/D/B/A/DBA9E313-B833-48EE-998A-240AA799A8AB/Hybrid_Identity_White_Paper.pdf)
* [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/)
* [Blog de l’équipe Azure AD](https://blogs.technet.microsoft.com/ad/)

## <a name="azure-ad-access-reviews"></a>Révisions d’accès Azure AD

Les révisions d’accès Azure Active Directory (Azure AD) permettent aux organisations de gérer efficacement les appartenances à des groupes, les accès aux applications d’entreprise, et les attributions de rôles dotés de privilèges.

En savoir plus :

* [Révisions d’accès Azure AD](../../active-directory/governance/access-reviews-overview.md)
* [Manage user access with Azure AD access reviews](../../active-directory/governance/access-reviews-overview.md) (Gérer l’accès des utilisateurs avec les révisions d’accès Azure AD)