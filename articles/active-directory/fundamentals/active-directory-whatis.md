---
title: Qu’est-ce qu’Azure Active Directory ? - Azure Active Directory | Microsoft Docs
description: Vue d’ensemble et informations conceptuelles sur Azure Active Directory, notamment la terminologie, les licences disponibles et une liste de fonctionnalités associées avec des liens permettant d’obtenir des informations supplémentaires.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: overview
ms.date: 06/05/2020
ms.author: ajburnle
ms.custom: it-pro, seodec18, seo-update-azuread-jan, contperf-fy20q4
ms.collection: M365-identity-device-management
ms.openlocfilehash: e773be4331be3170a04c8cf5221200589dd13689
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029026"
---
# <a name="what-is-azure-active-directory"></a>Qu’est-ce qu’Azure Active Directory ?

Azure Active Directory (Azure AD) est le service de gestion de l’accès et des identités basé sur le cloud de Microsoft. Il permet à vos employés de se connecter et d’accéder aux ressources suivantes :

- Ressources externes telles que Microsoft 365, le portail Azure et des milliers d’autres applications SaaS.

- Ressources internes telles que les applications situées sur votre réseau d’entreprise et intranet ainsi que les applications cloud développées par votre propre organisation. Pour plus d’informations sur la création d’un locataire pour votre organisation, consultez [Démarrage rapide : Créer un locataire dans Azure Active Directory](active-directory-access-create-new-tenant.md).

Pour connaître la différence entre Azure AD et Active Directory Domain Services, consultez [Comparer Active Directory et Azure Active Directory](active-directory-compare-azure-ad-to-ad.md). Vous pouvez également utiliser les différentes affiches de la [série Microsoft Cloud pour architectes d’entreprise](/microsoft-365/solutions/cloud-architecture-models) afin de mieux comprendre les services d’identité de base dans Azure, Azure AD et Microsoft 365.

## <a name="who-uses-azure-ad"></a>Qui utilise Azure AD ?

Azure AD s’adresse aux utilisateurs suivants :

- **Les administrateurs informatiques.** En tant qu’administrateur informatique, vous pouvez utiliser Azure AD pour contrôler l’accès à vos applications et à leurs ressources en fonction des besoins de votre entreprise. Par exemple, vous pouvez utiliser Azure AD pour exiger que l’accès aux ressources importantes de l’organisation soit soumis à l’authentification multifacteur. Vous pouvez également utiliser Azure AD pour automatiser le provisionnement des utilisateurs entre Windows Server AD et vos applications cloud, notamment Microsoft 365. Azure AD vous offre enfin des outils puissants pour vous aider à protéger automatiquement les identités et informations d’identification des utilisateurs et à répondre à vos exigences en matière de gouvernance. Pour commencer, inscrivez-vous pour [essayer gratuitement Azure Active Directory Premium pendant 30 jours](https://azure.microsoft.com/trial/get-started-active-directory/).

- **Les développeurs d’applications.** En tant que développeur d’applications, vous pouvez utiliser Azure AD dans le cadre d’une approche normalisée pour ajouter l’authentification unique à votre application, laquelle peut alors fonctionner avec les informations d’identification existantes d’un utilisateur. Azure AD fournit également des API qui peuvent vous aider à créer des expériences d’application personnalisées qui utilisent des données organisationnelles existantes. Pour commencer, inscrivez-vous pour [essayer gratuitement Azure Active Directory Premium pendant 30 jours](https://azure.microsoft.com/trial/get-started-active-directory/). Pour plus d’informations, vous pouvez également consulter [Azure Active Directory pour les développeurs](../develop/index.yml).

- **Les abonnées à Microsoft 365, Office 365, Azure ou Dynamics CRM Online.** En tant qu’abonné, vous utilisez déjà Azure AD. Chaque locataire Microsoft 365, Office 365, Azure et Dynamics CRM Online est automatiquement un locataire Azure AD. Vous pouvez immédiatement commencer à gérer l’accès à vos applications cloud intégrées.

## <a name="what-are-the-azure-ad-licenses"></a>Quelles sont les licences Azure AD ?

Les services d’entreprise Microsoft Online comme Microsoft 365 ou Microsoft Azure nécessitent Azure AD pour la connexion et la protection des identités. Si vous vous abonnez à un service en ligne Microsoft pour entreprise, vous disposez automatiquement d’Azure AD avec un accès à toutes les fonctionnalités gratuites.

Pour enrichir votre implémentation Azure AD, vous pouvez ajouter des fonctionnalités payantes en procédant à une mise à niveau vers les licences Azure Active Directory Premium P1 ou Premium P2. Les licences payantes Azure AD s’appuient sur votre annuaire gratuit existant et couvrent le libre-service, la supervision améliorée, les rapports de sécurité et l’accès sécurisé pour vos utilisateurs mobiles.

>[!Note]
>Pour connaître la structure de prix de ces licences, consultez les [tarifs d’Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).
>
>Les éditions Azure Active Directory, Premium P1 et Premium P2 ne sont actuellement pas prises en charge en Chine. Pour plus d’informations sur les tarifs d’Azure AD, contactez le [forum Azure Active Directory](https://azure.microsoft.com/support/community/?product=active-directory).

- **Azure Active Directory Free.** Offre la gestion des utilisateurs et des groupes, la synchronisation d’annuaires locaux, des rapports de base, des changements de mot de passe en libre-service pour les utilisateurs cloud ainsi que l’authentification unique sur Azure, Microsoft 365 et bon nombre d’applications SaaS populaires.

- **Azure Active Directory Premium P1.** En plus des fonctionnalités Free, la licence P1 offre à vos utilisateurs hybrides l’accès aux ressources locales et cloud. Elle prend également en charge des fonctionnalités administratives avancées, notamment les groupes dynamiques, la gestion de groupes libre-service, Microsoft Identity Manager (suite de gestion des identités et des accès locaux) et l’écriture différée dans le cloud (pour faire bénéficier à vos utilisateurs locaux de la réinitialisation de mot de passe libre-service).

- **Azure Active Directory Premium P2.** En plus des fonctionnalités Free et P1, la licence P2 offre [Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md) pour fournir un accès conditionnel en fonction des risques à vos applications et aux données critiques de l’entreprise. Elle propose également [Privileged Identity Management](../privileged-identity-management/pim-getting-started.md) pour faciliter la découverte, la restriction et la supervision des administrateurs et leur accès aux ressources et, au besoin, fournir un accès juste-à-temps.

- **Licences pour les fonctionnalités avec « paiement à l’utilisation ».** Vous pouvez également obtenir des licences pour des fonctionnalités supplémentaires, par exemple Azure Active Directory B2C (entreprise-client). B2C peut vous aider à doter vos applications orientées clients de solutions de gestion des identités et des accès. Pour plus d’informations, consultez la [documentation sur Azure Active Directory B2C](../../active-directory-b2c/index.yml).

Pour plus d’informations sur l’association d’un abonnement Azure à Azure AD, consultez [Associer ou ajouter un abonnement Azure à Azure Active Directory](active-directory-how-subscriptions-associated-directory.md). Pour plus d’informations sur l’attribution de licences à vos utilisateurs, consultez [ Assigner ou supprimer des licences Azure Active Directory](license-users-groups.md).

## <a name="which-features-work-in-azure-ad"></a>Quelles sont les fonctionnalités disponibles dans Azure AD ?

Après avoir choisi votre licence Azure AD, vous avez accès à une partie ou à la totalité des fonctionnalités suivantes pour votre organisation :

|Category|Description|
|-------|-----------|
|Gestion des applications|Gérez vos applications cloud et locales avec le proxy d’application, l’authentification unique, le portail Mes applications (également appelé panneau d’accès) et les applications SaaS (software as a service). Pour plus d’informations, consultez [Guide pratique pour offrir un accès à distance sécurisé aux applications locales](../manage-apps/application-proxy.md) et la [documentation sur la gestion des applications](../manage-apps/index.yml).|
|Authentification|Gérez la réinitialisation de mot de passe libre-service Azure Active Directory, l’authentification multifacteur, la liste de mots de passe interdits et le verrouillage intelligent. Pour plus d’informations, consultez la [documentation sur Azure AD Authentication](../authentication/index.yml).|
|Azure Active Directory pour les développeurs|Créez des applications qui connectent toutes les identités Microsoft et obtiennent des jetons pour appeler Microsoft Graph, d’autres API Microsoft ou des API personnalisées. Pour plus d’informations, consultez [Plateforme d’identités Microsoft (Azure Active Directory pour développeurs)](../develop/index.yml).|
|Entreprise-entreprise (B2B)|Gérez vos utilisateurs invités et partenaires externes tout en conservant le contrôle de vos données d’entreprise. Pour plus d’informations, consultez la [documentation sur Azure Active Directory B2B](../external-identities/index.yml).|
|Entreprise-client (B2C)|Personnalisez et contrôlez la façon dont les utilisateurs s’inscrivent, se connectent et gèrent leurs profils quand ils utilisent vos applications. Pour plus d’informations, consultez la [documentation sur Azure Active Directory B2C](../../active-directory-b2c/index.yml).|
|Accès conditionnel|Gérez l’accès à vos applications cloud. Pour plus d’informations, consultez la [documentation sur l’accès conditionnel dans Azure AD](../conditional-access/index.yml).|
|Gestion des appareils|Gérez la façon dont vos appareils cloud ou locaux accèdent à vos données d’entreprise. Pour plus d’informations, consultez la [documentation sur la gestion des appareils Azure AD](../devices/index.yml).|
|Services de domaine|Joignez des machines virtuelles Azure à un domaine sans contrôleur de domaine. Pour plus d’informations, consultez la [documentation sur Azure AD Domain Services](../../active-directory-domain-services/index.yml).|
|Utilisateurs d’entreprise|Gérez l’attribution des licences, accédez à des applications et configurez des délégués à l’aide de groupes et de rôles d’administrateur. Pour plus d’informations, consultez la [documentation sur la gestion des utilisateurs Azure Active Directory](../enterprise-users/index.yml).|
|Identité hybride|Utilisez Azure Active Directory Connect et Connect Health pour fournir une identité d’utilisateur unique pour l’authentification et l’autorisation auprès de toutes les ressources, indépendamment de l’emplacement (cloud ou local). Pour plus d’informations, consultez la [documentation sur les identités hybrides](../hybrid/index.yml).|
|Gouvernance des identités|Gérez l’identité de votre organisation au moyen de contrôles d’accès pour vos employés, partenaires commerciaux, fournisseurs, services et applications. Vous pouvez également effectuer des révisions d’accès. Pour plus d’informations, consultez la [documentation sur Azure AD Identity Governance](../governance/identity-governance-overview.md) et [Révisions d’accès Azure AD](../governance/access-reviews-overview.md).|
|Identity Protection|Détectez les vulnérabilités potentielles qui affectent les identités de votre organisation, configurez des stratégies pour répondre aux actions suspectes et prenez les mesures appropriées pour les résoudre. Pour plus d’informations, consultez [Azure AD Identity Protection](../identity-protection/index.yml).|
|Identités gérées pour les ressources Azure|Fournit à vos services Azure une identité managée automatiquement dans Azure AD qui peut authentifier n’importe quel service d’authentification pris en charge par Azure AD, notamment Key Vault. Pour plus d’informations, consultez [Identités managées pour les ressources Azure](../managed-identities-azure-resources/overview.md).|
|Privileged Identity Management (PIM)|Gérez, contrôlez et supervisez l’accès au sein de votre organisation. Cette fonctionnalité inclut l’accès aux ressources dans Azure AD et Azure, ainsi qu’à d’autres services Microsoft Online, comme Microsoft 365 ou Intune. Pour plus d’informations, consultez [Qu’est-ce qu’Azure AD Privileged Identity Management ?](../privileged-identity-management/index.yml).|
|Rapports et analyse|Obtenez des insights sur la sécurité et les modèles d’utilisation de votre environnement. Pour plus d’informations, consultez [Rapports et supervision Azure Active Directory](../reports-monitoring/index.yml).|

## <a name="terminology"></a>Terminologie

Pour mieux comprendre Azure AD et sa documentation, nous vous recommandons de passer en revue les termes suivants.

|Terme ou concept|Description|
|---------------|-----------|
|Identité| Une chose qui peut être authentifiée. Une identité peut être un utilisateur avec un nom d’utilisateur et un mot de passe. Les identités incluent également des applications ou autres serveurs qui peuvent nécessiter l’authentification via des clés secrètes ou des certificats.|
|Compte| Une identité qui a des données associées. Vous ne pouvez pas avoir de compte sans identité.|
|Compte Azure AD| Identité créée par le biais d’Azure AD ou d’un autre service cloud Microsoft comme Microsoft 365. Les identités sont stockées dans Azure AD et sont accessibles à tout abonnement à un service cloud de votre organisation. Ce compte est parfois appelé un compte professionnel ou scolaire.|
|Administrateur de comptes|Ce rôle d’administrateur d’abonnement classique est théoriquement le propriétaire du compte de facturation d’un abonnement. Ce rôle a accès au [Centre des comptes Azure](https://account.azure.com/Subscriptions) et vous permet de gérer tous les abonnements d’un compte. Pour plus d’informations, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles d’administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Administrateur de services|Ce rôle d’administrateur d’abonnement classique vous permet de gérer toutes les ressources Azure, notamment l’accès à celles-ci. Ce rôle a un droit d’accès équivalent à celui d’un utilisateur qui se voit attribuer le rôle Propriétaire au niveau de l’étendue de l’abonnement. Pour plus d’informations, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles d’administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Propriétaire|Ce rôle vous permet de gérer toutes les ressources Azure, notamment l’accès à celles-ci. Ce rôle est basé sur un système d’autorisation plus récent appelé « contrôle d’accès en fonction du rôle Azure » (Azure RBAC) qui permet de gérer avec précision l’accès aux ressources Azure. Pour plus d’informations, consultez [Rôles d’administrateur d’abonnement classique, rôles Azure et rôles d’administrateur Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Administrateur général Azure AD|Ce rôle d’administrateur est automatiquement attribué au créateur du locataire Azure AD. Les administrateurs généraux peuvent effectuer toutes les fonctions d’administration pour Azure AD et tout service fédéré sur Azure AD, comme Exchange Online, SharePoint Online et Skype Entreprise Online. Vous pouvez avoir plusieurs administrateurs généraux, mais seuls les administrateurs généraux peuvent attribuer des rôles d’administrateur (notamment d’autres rôles Administrateur général) aux utilisateurs. Notez que ce rôle d’administrateur est appelé Administrateur général dans le portail Azure, mais il a pour nom **Administrateur d’entreprise** dans l’API Microsoft Graph et Azure AD PowerShell. Pour plus d’informations sur les rôles d’administrateur, consultez [Autorisations de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).|
|Abonnement Azure| Permet de payer les services cloud Azure. Vous pouvez avoir plusieurs abonnements, lesquels sont liés à une carte de crédit.|
|Client Azure| Instance dédiée et approuvée d’Azure AD qui est automatiquement créée quand votre organisation souscrit un abonnement à un service cloud Microsoft tel que Microsoft Azure, Microsoft Intune ou Microsoft 365. Un locataire Azure représente une seule organisation.|
|Locataire unique| Un locataire Azure qui accède à d’autres services dans un environnement dédié est considéré comme un locataire unique.|
|Multi-locataire| Les locataires Azure qui accèdent à d’autres services dans un environnement partagé entre plusieurs organisations sont considérés comme multilocataires.|
|Annuaire Azure AD|Chaque locataire Azure a un annuaire Azure AD dédié et approuvé. L’annuaire Azure AD inclut les utilisateurs, groupes et applications du locataire. Il permet d’effectuer les fonctions de gestion des identités et des accès pour les ressources des locataires.|
|Domaine personnalisé|Chaque nouvel annuaire Azure AD est fourni avec un nom de domaine initial au format nomdomaine.onmicrosoft.com. En plus de ce nom initial, vous pouvez ajouter à la liste les noms de domaine de votre organisation, notamment ceux que vous utilisez pour exercer votre activité commerciale et dont vos utilisateurs se servent pour accéder aux ressources de votre organisation. L’ajout de noms de domaine personnalisés vous permet de créer des noms d’utilisateur qui sont familiers à vos utilisateurs, par exemple alain@contoso.com.|
|Compte Microsoft (également appelé MSA)|Comptes personnels qui donnent accès à vos produits et services cloud Microsoft orientés consommateurs, comme Outlook, OneDrive, Xbox LIVE ou Microsoft 365. Votre compte Microsoft est créé et stocké dans le système de comptes d’identité des consommateurs de Microsoft.|

## <a name="next-steps"></a>Étapes suivantes

- [S’inscrire à Azure Active Directory Premium](active-directory-get-started-premium.md)

- [Associer un abonnement Azure à votre annuaire Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)

- [Check-list pour le déploiement des fonctionnalités d’Azure Active Directory Premium P2](active-directory-deployment-checklist-p2.md)
