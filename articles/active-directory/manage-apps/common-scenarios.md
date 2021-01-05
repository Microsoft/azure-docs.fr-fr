---
title: Scénarios courants de gestion des applications pour Azure Active Directory | Microsoft Docs
description: Centralisation de la gestion des applications avec Azure AD
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: overview
ms.date: 03/02/2019
ms.author: kenwith
ms.reviewer: arvinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2b78562bc3590bdf5b15e4061ba23e1284b07c16
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94835490"
---
# <a name="centralize-application-management-with-azure-ad"></a>Centralisation de la gestion des applications avec Azure AD

Partout dans le monde, la gestion des mots de passe constitue un véritable cauchemar, tant pour le service informatique que pour les employés. C’est pour cette raison que de plus en plus d’entreprises se tournent vers Azure Active Directory, la solution de gestion des identités et des accès de Microsoft pour le cloud et toutes vos autres ressources. Passez d’une application à l’autre sans avoir à saisir un mot de passe pour chacune d’elles. Passez d’Outlook à Workday ou à ADP aussi vite que vous ouvrez ces applications, et en toute sécurité. Collaborez ensuite avec vos partenaires voire d’autres personnes extérieures à votre organisation, sans avoir à contacter le service informatique. Azure AD vous permet également de gérer les risques en sécurisant vos en utilisant l’authentification multifacteur pour vérifier votre identité, en utilisant une méthode Machine Learning adaptative en continu ainsi que la veille de sécurité pour détecter les connexions suspectes et sécuriser l’accès aux applications dont vous avez besoin, où que vous soyez. C’est la solution idéale non seulement pour les utilisateurs, mais également pour le service informatique. Grâce à des vérifications d’accès juste-à-temps et à une suite de gouvernance à l’échelle complète, Azure AD vous aide à rester en conformité et à appliquer des stratégies. Et vous pouvez même automatiser l’approvisionnement des comptes d’utilisateur, pour une gestion des accès encore plus efficace. Découvrez quelques-uns des scénarios courants utilisés par le client pour exploiter les fonctionnalités de gestion des applications Azure Active Directory.

**Scénarios courants**


> [!div class="checklist"]
> * Authentification unique pour toutes vos applications
> * Approvisionnement et déprovisionnement automatisés 
> * Sécurisation de vos applications
> * Gouverner l’accès à vos applications
> * Accès sécurisé hybride

## <a name="scenario-1-set-up-sso-for-all-your-applications"></a>Scénario 1 : Configurer l’authentification unique pour toutes vos applications

Plus aucun mot de passe à gérer. Accédez en toute sécurité à toutes les ressources dont vous avez besoin avec vos informations d’identification d’entreprise. 

|Fonctionnalité  | Description | Recommandation |
|---------|---------|---------|
|SSO|L’authentification unique fédérée basée sur des normes s’appuie des normes industrielles approuvées.|Utilisez toujours [SAML / OIDC](../develop/v2-howto-app-gallery-listing.md) pour activer l’authentification unique si votre application prend en charge cette norme.|
|Mes applications|Offrez aux utilisateurs un hub simple leur permettant de découvrir toutes leurs applications et d’y accéder. Donnez-leur la possibilité d’accroître leur productivité grâce à des fonctionnalités en libre-service, pour demander l’accès à de nouveaux groupes et applications ou gérer l’accès à ces ressources pour le compte d’autres utilisateurs.| Déployez [Mes applications](./access-panel-deployment-plan.md) dans votre organisation une fois que vous avez intégré vos applications à Azure AD pour l’authentification unique.|

## <a name="scenario-2-automate-provisioning-and-deprovisioning"></a>Scénario 2 : Approvisionnement et déprovisionnement automatisés 


La plupart des applications requièrent l’approvisionnement d’un utilisateur dans l’application avant de pouvoir accéder aux ressources nécessaires. L’utilisation de fichiers CSV ou de scripts complexes peut être coûteuse et difficile à gérer. En outre, les clients doivent s’assurer que les comptes sont supprimés lorsqu’un utilisateur ne devrait plus y accéder. Tirez parti des outils ci-dessous pour automatiser l’approvisionnement et le déprovisionnement. 


|Fonctionnalité  |Description|Recommandation |
|---------|---------|---------|
|Approvisionnement SCIM|[SCIM](https://aka.ms/SCIMOverview) est une méthode recommandée par l’industrie pour l’automatisation de l’approvisionnement des utilisateurs. Toute application compatible avec SCIM peut être intégrée à Azure AD. Créez, mettez à jour et supprimez automatiquement des comptes d’utilisateur sans avoir à gérer les fichiers CSV, les scripts personnalisés ou les solutions locales.|Consultez la liste croissante des applications [préintégrées](../saas-apps/tutorial-list.md) dans la Galerie d’applications Azure AD|
|Microsoft Graph|Tirez parti de toute la variété et de l’étendue des données Azure AD pour enrichir votre application avec les données dont elle a besoin.|Tirez parti de [Microsoft Graph](https://developer.microsoft.com/graph/) pour récupérer des données sur l’écosystème Microsoft. |


## <a name="scenario-3-secure-your-applications"></a>Scénario 3 : Sécurisation de vos applications
L’identité est le pilier de votre sécurité. Si une identité est compromise, il est extrêmement difficile d’arrêter l’effet de domino avant qu’il ne soit trop tard. En moyenne, plus de 100 jours s’écoulent avant que les organisations découvrent qu’une identité a été compromise. Utilisez les outils fournis par Azure AD pour améliorer le niveau de sécurité de vos applications. 

|Fonctionnalité  |Description| Recommandation |
|---------|---------| ---------|
|Azure AD MFA|Azure AD Multi-Factor Authentication (MFA) est la solution de vérification en deux étapes de Microsoft. À l’aide des méthodes d’authentification approuvées par l’administrateur, Azure AD MFA contribue à sécuriser l’accès à vos données et à vos applications, tout en répondant à la nécessité de mettre en place un processus d’authentification simple.| [Activez MFA](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/all-your-creds-are-belong-to-us/ba-p/855124) pour vos utilisateurs.  |
|Accès conditionnel|Avec l’accès conditionnel, vous pouvez mettre en œuvre des décisions de contrôle d’accès automatisées pour déterminer qui peut accéder à vos applications cloud, sur la base de conditions.| Passez en revue les [paramètres de sécurité par défaut](../fundamentals/concept-fundamentals-security-defaults.md) et les [stratégies communes](../conditional-access/concept-conditional-access-policy-common.md) que vos clients utilisent. | 
|Identity Protection|Pour protéger vos utilisateurs, Identity Protection s'appuie sur les connaissances acquises par Microsoft au niveau des organisations avec Azure AD, de l'espace grand public avec les comptes Microsoft et des jeux avec Xbox. Microsoft analyse 6 500 milliards de signaux par jour pour identifier les menaces et protéger les clients.|Activez les [stratégies de protection d’identité par défaut](../identity-protection/concept-identity-protection-policies.md) fournies par notre service. | 

## <a name="scenario-4-govern-access-to-your-applications"></a>Scénario 4 : Gouverner l’accès à vos applications
Identity Governance aide les organisations à trouver un équilibre entre productivité (vitesse à laquelle des personnes peuvent accéder aux applications dont elles ont besoin, par exemple pour joindre mon organisation) et sécurité (comment leur accès doit évoluer au fil du temps, par exemple à la suite de modifications de leur statut professionnel). 

|Fonctionnalité  |Description|Recommandation |
|---------|---------| ---------|
|ELM|La gestion des droits d'utilisation Azure AD peut aider les utilisateurs à l’intérieur comme à l’extérieur de votre organisation à gérer plus efficacement l’accès à leurs applications.| Autorisez les non-administrateurs à gérer l’accès à leurs applications grâce à des [packages d’accès](../governance/entitlement-management-access-package-first.md).|
|Révisions d’accès|L’accès de l’utilisateur aux applications peut être passé en revue régulièrement pour vérifier que seules les personnes appropriées continuent de bénéficier d’un accès.| [Révisez l’accès](../governance/access-reviews-overview.md) à vos applications les plus sensibles. |
|Log Analytics|Générez des rapports sur les utilisateurs qui accèdent aux applications et stockez-les dans l’outil SIEM de votre choix pour mettre en corrélation les informations de différentes sources de données au fil du temps.| Activez [Log Analytics](../reports-monitoring/howto-analyze-activity-logs-log-analytics.md) et configurez des alertes pour les événements critiques liés à vos applications. |


## <a name="scenario-5-hybrid-secure-access"></a>Scénario 5 : Accès sécurisé hybride
L’identité peut vous servir de plan de contrôle uniquement si elle peut se connecter à toutes les applications dans le cloud et locales. Tirez parti des outils fournis par Azure AD et ses partenaires pour sécuriser l’accès aux applications basées sur l’authentification héritée.

|Fonctionnalité  |Description|Recommandation |
|---------|---------|---------|
|Application Proxy (Proxy d’application)|Aujourd’hui, les employés veulent être productifs partout, tout le temps, depuis n’importe quel appareil. Ils doivent accéder aux applications SaaS dans le cloud et aux applications d’entreprise locales. Le proxy Azure AD Application offre cet accès sans réseaux privés virtuels (VPN) ni zones démilitarisées (DMZ), qui sont des systèmes coûteux et complexes.|Configurez l’[accès à distance](./application-proxy.md) pour vos applications locales. |
|F5, Akamai, Zscaler|Votre contrôleur de livraison et de mise en réseau vous permet de protéger facilement vos applications héritées, qui sont toujours critiques pour vos processus d’entreprise, mais que vous ne pouviez pas protéger avant avec Azure AD. Il est probable que vous disposiez déjà de tout ce dont vous avez besoin pour commencer à protéger ces applications.| Vous utilisez Akamai, Citrix, F5 ou Zscaler ? Consultez nos [solutions prédéfinies](./secure-hybrid-access.md). | 

## <a name="related-articles"></a>Articles connexes

- [Gestion des applications](./index.yml)
- [Approvisionnement d’applications](../app-provisioning/user-provisioning.md)
- [Accès sécurisé hybride](./secure-hybrid-access.md)
- [Gouvernance des identités](../governance/identity-governance-overview.md)
- [Plateforme d’identité Microsoft](../develop/v2-overview.md)
- [Identité et sécurité](../conditional-access/index.yml)