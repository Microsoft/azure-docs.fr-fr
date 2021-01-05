---
title: Accès hybride sécurisé à Azure AD avec F5 | Microsoft Docs
description: Intégration de F5 BIG-IP Access Policy Manager et Azure Active Directory pour l’accès hybride sécurisé
services: active-directory
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 11/12/2020
ms.author: gasinh
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0e011417b936ed83b4658e6dad25bf8e8ee88aed
ms.sourcegitcommit: e5f9126c1b04ffe55a2e0eb04b043e2c9e895e48
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96317565"
---
# <a name="f5-big-ip-access-policy-manager-and-azure-active-directory-integration-for-secure-hybrid-access"></a>Intégration de F5 BIG-IP Access Policy Manager et Azure Active Directory pour l’accès hybride sécurisé

La prolifération de la mobilité et l'évolution du paysage des menaces imposent une surveillance accrue de l'accès aux ressources et de la gouvernance, plaçant le concept de [confiance nulle](https://www.microsoft.com/security/blog/2020/04/02/announcing-microsoft-zero-trust-assessment-tool/) au centre de tous les programmes de modernisation.
Chez Microsoft et F5, nous réalisons que cette transformation numérique représente généralement un parcours de plusieurs années pour une entreprise, laissant potentiellement des ressources critiques exposées jusqu'à leur modernisation. Les solutions F5 BIG-IP et Azure Active Directory Secure Hybrid Access (SHA) visent non seulement à améliorer l'accès à distance aux applications sur site, mais aussi à renforcer la posture de sécurité globale de ces services vulnérables.

Dans ce contexte, une étude montre que 60 à 80 % des applications sur site sont héritées par nature, c'est-à-dire qu'elles ne peuvent pas être intégrées directement à Azure Active Directory (AD). Cette même étude a également indiqué qu'une grande partie de ces systèmes fonctionnent sur des versions inférieures de SAP, Oracle, SAGE et d'autres charges de travail bien connues qui fournissent des services essentiels.

SHA comble cette lacune en permettant aux organisations de continuer à utiliser leurs investissements F5 pour proposer des réseaux et d'applications de qualité supérieure. Combiné avec Azure AD, SHA fait le pont entre le paysage hétérogène des applications et le plan de contrôle moderne de l'identité.

Le fait qu'Azure AD pré-authentifie l'accès aux services publiés de BIG-IP présente de nombreux avantages :

- Authentification sans mot de passe via [Windows Hello](https://docs.microsoft.com/windows/security/identity-protection/hello-for-business/hello-overview), [MS Authenticator](https://docs.microsoft.com/azure/active-directory/user-help/user-help-auth-app-download-install), [des clés Fast Identity Online (FIDO)](https://docs.microsoft.com/azure/active-directory/authentication/howto-authentication-passwordless-security-key) et l’[authentification par certificat](https://docs.microsoft.com/azure/active-directory/authentication/active-directory-certificate-based-authentication-get-started)

- [Accès conditionnel](https://docs.microsoft.com/azure/active-directory/conditional-access/overview) préemptif et [Multi-Factor Authentication (MFA)](https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-howitworks)

- [Identity Protection](https://docs.microsoft.com/azure/active-directory/identity-protection/overview-identity-protection#:~:text=Identity%20Protection%20is%20a%20tool%20that%20allows%20organizations,detection%20data%20to%20third-party%20utilities%20for%20further%20analysis) : contrôle adaptatif via le profilage des risques des utilisateurs et des sessions

- [Détection des informations d’identification divulguées](https://docs.microsoft.com/azure/active-directory/identity-protection/concept-identity-protection-risks)

- [Réinitialisation de mot de passe en libre-service (SSPR)](https://docs.microsoft.com/azure/active-directory/authentication/tutorial-enable-sspr)

- [Collaboration avec les partenaires](https://docs.microsoft.com/azure/active-directory/governance/entitlement-management-external-users) : gestion des droits pour l'accès régi des invités

- [Cloud App Security (CASB)](https://docs.microsoft.com/cloud-app-security/what-is-cloud-app-security) : pour la détection et le contrôle complets des applications

- Surveillance des menaces : [Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/) pour l’analyse avancée des menaces

- Le [portail Azure AD](https://azure.microsoft.com/features/azure-portal/) : un plan de contrôle unique pour l’administration de l’identité et de l’accès

## <a name="scenario-description"></a>Description du scénario

En tant que contrôleur de remise d’applications (ADC) et SSL-VPN, un système BIG-IP fournit un accès local et distant à tous les types de services, notamment :

- Applications web modernes et héritées

- Applications non web

- Services d’API web REST et SOAP

Son Local Traffic Manager (LTM) permet la publication sécurisée des services grâce à une fonctionnalité de proxy inverse. En même temps, un Access Policy Manager (APM) sophistiqué complète le système BIG-IP avec un ensemble plus riche de capacités, permettant la fédération et l’authentification unique (SSO).

L’intégration repose sur une approbation de fédération standard entre APM et Azure AD, commune à la plupart des cas d’utilisation SHA qui incluent le [scénario SSL-VPN](f5-aad-password-less-vpn.md). Les ressources Security Assertion Markup Language (SAML), OAuth et Open ID Connect (OIDC) ne font pas non plus exception, car elles peuvent elles aussi être sécurisées pour un accès à distance. Il existe également des scénarios où un système BIG-IP devient un point de blocage pour l'accès de type « confiance nulle » à tous les services, y compris les applications SaaS.

La capacité d'intégration d'un système BIG-IP à Azure AD permet la transition de protocole nécessaire pour sécuriser les services traditionnels ou non intégrés à Azure AD avec des contrôles modernes comme l'[authentification sans mot de passe](https://www.microsoft.com/security/business/identity/passwordless) et l'[accès conditionnel](https://docs.microsoft.com/azure/active-directory/conditional-access/overview). Dans ce scénario, un système BIG-IP continue à remplir son rôle de proxy inverse, tout en transmettant la pré-authentification et l'autorisation à Azure AD, selon chaque service.

Les étapes 1 à 4 du diagramme illustrent l'échange de pré-authentification en amont entre un utilisateur, un système BIG-IP, et Azure AD, dans un flux initié par un fournisseur de services. Les étapes 5 et 6 montrent l'enrichissement ultérieur de la session APM et de l’authentification unique SSO vers les services principaux-end individuels.

![L’image montre l’architecture de haut niveau](./media/f5-aad-integration/integration-flow-diagram.png)

| Étape | Description |
|:------|:-----------|
| 1. | L'utilisateur sélectionne une icône d'application dans le portail, résolvant l'URL vers SAML SP (BIG-IP) |
| 2. | BIG-IP redirige l'utilisateur vers SAML IDP (Azure AD) pour une pré-authentification|
| 3. | Azure AD traite les stratégies d’autorité de certification et les [contrôles de session](https://docs.microsoft.com/azure/active-directory/conditional-access/concept-conditional-access-session) pour l’autorisation|
| 4. | L’utilisateur retourne vers BIG-IP en présentant les revendications SAML émises par Azure AD |
| 5. | BIG-IP demande au service publié toutes les informations de session supplémentaires à inclure dans [SSO](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sso) et [Contrôle d’accès en fonction du rôle (RBAC)](https://docs.microsoft.com/azure/role-based-access-control/overview) |
| 6. | BIG-IP transfère la demande du client au service principal

## <a name="user-experience"></a>Expérience utilisateur

Qu'ils soient employés, affiliés ou consommateurs, la plupart des utilisateurs connaissent déjà l'expérience de connexion à Office 365. L'accès aux services BIG-IP via SHA leur sera donc largement familier.

Les utilisateurs trouvent à présent leurs services publiés BIG-IP consolidés dans les plateformes de lancement [MyApps](https://docs.microsoft.com/azure/active-directory/user-help/my-apps-portal-end-user-access) ou [O365](https://o365pp.blob.core.windows.net/media/Resources/Microsoft%20365%20Business/Launchpad%20Overview_for%20Partners_10292019.pdf), ainsi que des fonctionnalités en libre-service à un ensemble plus complet de services, quel que soit le type d’appareil ou l’emplacement. Ils peuvent même continuer à accéder aux services publiés directement via le portail Webtop propriétaire BIG-IP, s’ils préfèrent. Lors de la déconnexion, SHA s'assure que la session d'un utilisateur est terminée aux deux extrémités, BIG-IP et Azure AD, garantissant ainsi que les services restent entièrement protégés contre tout accès non autorisé.  

Les captures d'écran fournies proviennent du portail de l’application Azure AD auquel les utilisateurs accèdent en toute sécurité pour trouver leurs services publiés BIG-IP et gérer les propriétés de leur compte.  

![La capture d’écran montre la galerie woodgrove myapps](media/f5-aad-integration/woodgrove-app-gallery.png)

![La capture d’écran montre la page en libre-service woodgrove myaccounts](media/f5-aad-integration/woodgrove-myaccount.png)

## <a name="insights-and-analytics"></a>Insights et analytiques

Le rôle d'un système BIG-IP est essentiel pour toute entreprise : les instances BIG-IP déployées doivent donc être surveillées pour garantir que les services publiés sont hautement disponibles, tant au niveau de SHA que sur le plan opérationnel.

Plusieurs options existent pour consigner les événements, soit localement, soit à distance grâce à une solution de gestion des informations et des événements de sécurité (SIEM), permettant le stockage et le traitement off-box des données de télémétrie. Une solution très efficace pour surveiller une activité spécifique à Azure AD et SHA consiste à utiliser [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview) et [Azure Sentinel](https://docs.microsoft.com/azure/sentinel/overview), qui offrent les avantages suivants :

- Présentation détaillée de votre organisation, potentiellement dans plusieurs clouds, et des emplacements locaux, y compris une infrastructure BIG-IP

- Plan de contrôle unique fournissant une vue combinée de tous les signaux, évitant ainsi la dépendance vis-à-vis d’outils complexes et disparates

![L’image montre le flux de surveillance](media/f5-aad-integration/azure-sentinel.png)

## <a name="prerequisites"></a>Prérequis

L’intégration de F5 BIG-IP à Azure AD pour SHA impose les conditions préalables suivantes :

- Une instance F5 BIG-IP s’exécutant sur l’une des plateformes suivantes :

  - Appliance physique

  - Hypervisor Virtual Edition, par exemple Microsoft Hyper-V, VMware ESXi, Linux KVM et Citrix Hypervisor

  - Cloud Virtual Edition, par exemple Azure, VMware, KVM, Community Xen, MS Hyper-V, AWS, OpenStack et Google Cloud

    L’emplacement réel d’une instance BIG-IP peut être local ou n’importe quelle plateforme cloud prise en charge, y compris Azure, à condition qu’elle dispose d’une connectivité Internet, de ressources publiées et de tout autre service requis, par exemple Active Directory.  

- Une licence APM F5 BIG-IP active, via l’une des options suivantes :

   - F5 BIG-IP® Best Bundle (ou)

   - Licence autonome F5 BIG-IP Access Policy Manager™

   - Licence de composant additionnel F5 BIG-IP Access Policy Manager™ (APM) sur une instance existante de BIG-IP F5 BIG-IP® Local Traffic Manager™ (LTM)

   - Une [licence d’essai](https://www.f5.com/trial/big-ip-trial.php) BIG-IP Access Policy Manager™ (APM) de 90 jours

- Gestion des licences Azure AD via l’une des options suivantes :

   - Un [abonnement gratuit](https://docs.microsoft.com/windows/client-management/mdm/register-your-free-azure-active-directory-subscription#:~:text=%20Register%20your%20free%20Azure%20Active%20Directory%20subscription,will%20take%20you%20to%20the%20Azure...%20More%20) Azure AD fournit la configuration minimale requise pour l’implémentation de SHA avec l’authentification sans mot de passe

   - Un [abonnement Premium](https://azure.microsoft.com/pricing/details/active-directory/) fournit tous les avantages supplémentaires décrits dans la préface, notamment [Accès conditionnel](https://docs.microsoft.com/azure/active-directory/conditional-access/overview), [MFA](https://docs.microsoft.com/azure/active-directory/authentication/concept-mfa-howitworks) et [Identity Protection](https://docs.microsoft.com/azure/active-directory/identity-protection/overview-identity-protection)

Aucune expérience ni connaissance préalable de F5 BIG-IP n’est requise pour implémenter SHA, mais nous vous recommandons de vous familiariser avec la terminologie de F5 BIG-IP. La riche [base de connaissances](https://www.f5.com/services/resources/glossary) de F5 est également un excellent endroit pour commencer à acquérir des connaissances sur BIG-IP.

## <a name="deployment-scenarios"></a>Scénarios de déploiement

Les tutoriels suivants fournissent des instructions détaillées sur la mise en œuvre de certains des modèles les plus courants pour BIG-IP et Azure AD SHA :

- [Procédure pas à pas du déploiement de F5 BIG-IP dans Azure](f5-bigip-deployment-guide.md)

- [F5 BIG-IP APM et authentification unique Azure AD dans les applications Kerberos](https://docs.microsoft.com/azure/active-directory/saas-apps/kerbf5-tutorial#configure-f5-single-sign-on-for-kerberos-application)

- [F5 BIG-IP APM et authentification unique Azure AD dans les applications basées sur un en-tête](https://docs.microsoft.com/azure/active-directory/saas-apps/headerf5-tutorial#configure-f5-single-sign-on-for-header-based-application)

- [Sécurisation de F5 BIG-IP SSL-VPN avec Azure AD SHA](f5-aad-password-less-vpn.md)

## <a name="additional-resources"></a>Ressources supplémentaires

- [Fin des mots de passe, adoptez l’authentification sans mot de passe](https://www.microsoft.com/security/business/identity/passwordless)

- [Accès hybride sécurisé Azure Active Directory](https://azure.microsoft.com//services/active-directory/sso/secure-hybrid-access/)

- [Microsoft Zero Trust framework to enable remote work](https://www.microsoft.com/security/blog/2020/04/02/announcing-microsoft-zero-trust-assessment-tool/)

- [Prise en main avec Azure Sentinel](https://azure.microsoft.com/services/azure-sentinel/?&OCID=AID2100131_SEM_XfknpgAAAHoVMTvh:20200922160358:s&msclkid=5e0e022409fc1c94dab85d4e6f4710e3&ef_id=XfknpgAAAHoVMTvh:20200922160358:s&dclid=CJnX6vHU_esCFUq-ZAod1iQF6A)

## <a name="next-steps"></a>Étapes suivantes

Exécutez une preuve de concept SHA en utilisant votre infrastructure BIG-IP existante, ou en déployant une instance d’évaluation. [Le déploiement d’une machine virtuelle BIG-IP Virtual Edition (VE) dans Azure](f5-bigip-deployment-guide.md) prend environ 30 minutes et vous fournit :

- Une plateforme entièrement sécurisée pour modéliser une preuve de concept SHA

- Une instance de pré-production, une plateforme entièrement sécurisée à utiliser pour tester les nouvelles mises à jour système et les correctifs logiciels BIG-IP

En même temps, vous devez identifier une ou deux applications qui peuvent être ciblées pour être publiées via BIG-IP et protégées avec SHA.  

Notre recommandation est de commencer avec une application qui n'est pas encore publiée via BIG-IP afin d'éviter une éventuelle perturbation des services de production. Les lignes directrices mentionnées dans cet article vous aideront à vous familiariser avec la procédure générale de création des différents objets de configuration BIG-IP et de mise en place de SHA. Une fois ces opérations terminées, vous devriez être en mesure de faire de même avec tout autre nouveau service, et disposer également de suffisamment de connaissances pour convertir les services publiés BIG-IP existants dans SHA avec un minimum d'efforts.

Le guide interactif ci-dessous présente la procédure de haut niveau pour la mise en œuvre de SHA et l'expérience de l'utilisateur final.

[![L’image montre une couverture interactive du guide](media/f5-aad-integration/interactive-guide.png)](https://aka.ms/Secure-Hybrid-Access-F5-Interactive-Guide)
