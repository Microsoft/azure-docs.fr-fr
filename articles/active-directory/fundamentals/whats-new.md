---
title: Nouveautés Notes de publication - Azure Active Directory | Microsoft Docs
description: Découvrez les nouveautés d’Azure Active Directory, notamment les dernières notes de publication, les problèmes connus, les corrections de bogues, les fonctionnalités déconseillées et les modifications à venir.
services: active-directory
author: ajburnle
manager: daveba
featureFlags:
- clicktale
ms.assetid: 06a149f7-4aa1-4fb9-a8ec-ac2633b031fb
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 12/03/2020
ms.author: ajburnle
ms.reviewer: dhanyahk
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 710e31c18338243f9405a071b8fa544fe44044be
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903154"
---
# <a name="whats-new-in-azure-active-directory"></a>Nouveautés d’Azure Active Directory

>Soyez notifié de la disponibilité des mises à jour sur cette page en faisant un copier-coller de cette URL : `https://docs.microsoft.com/api/search/rss?search=%22Release+notes+-+Azure+Active+Directory%22&locale=en-us` dans votre lecteur de flux ![icône RSS](./media/whats-new/feed-icon-16x16.png).

Azure AD bénéficie d’améliorations en continu. Pour vous informer des développements les plus récents, cet article détaille les thèmes suivants :

- Versions les plus récentes
- Problèmes connus
- Résolution des bogues
- Fonctionnalités dépréciées
- Modifications planifiées

Cette page est mise à jour tous les mois. Donc, consultez-la régulièrement. Si vous recherchez des éléments antérieurs aux six derniers mois, vous les trouverez dans l’[Archive des Nouveautés d’Azure Active Directory](whats-new-archive.md).

---
## <a name="november-2020"></a>Novembre 2020

### <a name="azure-active-directory-tls-10-tls-11-and-3des-deprecation"></a>Dépréciation d’Azure Active Directory TLS 1.0, TLS 1.1 et 3DES

**Type :** Modification planifiée  
**Catégorie de service :** Toutes les applications Azure AD  
**Fonctionnalité de produit :** Standards

Azure Active Directory dépréciera les protocoles suivants dans les régions Azure Active Directory partout dans le monde d’ici au 30 juin 2021 :

- TLS 1.0
- TLS 1.1
- Suite de chiffrement 3DES (TLS_RSA_WITH_3DES_EDE_CBC_SHA)

Les environnements affectés sont les suivants :
- Azure Commercial Cloud
- Office 365 GCC et WW

Annonce : toutes les combinaisons client-serveur et navigateur-serveur doivent utiliser TLS 1.2 et des suites de chiffrement modernes afin de maintenir une connexion sécurisée à Azure Active Directory pour les services Azure, Office 365 et Microsoft 365. Cette modification est liée à l’article [Azure Active Directory TLS 1.0 & 1.1, and 3DES Cipher Suite Deprecation in US Gov Cloud](whats-new.md#azure-active-directory-tls-10-tls-11-and-3des-deprecation-in-us-gov-cloud).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---november-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Novembre 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce

En novembre 2020, nous avons ajouté les 52 applications suivantes à notre galerie d’applications avec prise en charge de la fédération :

[Travel & Expense Management](https://app.expenseonce.com/Account/Login), [Tribeloo](../saas-apps/tribeloo-tutorial.md), [Itslearning File Picker](https://pmteam.itslearning.com/), [Crises Control](../saas-apps/crises-control-tutorial.md), [CourtAlert](https://www.courtalert.com/), [StealthMail](https://stealthmail.com/), [Edmentum - Study Island](https://app.studyisland.com/cfw/login/), [Virtual Risk Manager](../saas-apps/virtual-risk-manager-tutorial.md), [TIMU](../saas-apps/timu-tutorial.md), [Looker Analytics Platform](../saas-apps/looker-analytics-platform-tutorial.md), [Talview - Recruit](https://recruit.talview.com/login), Real Time Translator, [Klaxoon](https://access.klaxoon.com/login), [Podbean](../saas-apps/podbean-tutorial.md), [zcal](https://zcal.co/signup), [expensemanager](https://api.expense-manager.com/), [Netsparker Enterprise](../saas-apps/netsparker-enterprise-tutorial.md), [En-trak Tenant Experience Platform](https://portal.en-trak.app/), [Appian](../saas-apps/appian-tutorial.md), [Panorays](../saas-apps/panorays-tutorial.md), [Builterra](https://portal.builterra.com/), [EVA Check-in](https://my.evacheckin.com/organization), [HowNow WebApp SSO](../saas-apps/hownow-webapp-sso-tutorial.md), [Coupa Risk Assess](../saas-apps/coupa-risk-assess-tutorial.md), [Lucid (All Products)](../saas-apps/lucid-tutorial.md), [GoBright](https://portal.brightbooking.eu/), [SailPoint IdentityNow](../saas-apps/sailpoint-identitynow-tutorial.md),[Resource Central](../saas-apps/resource-central-tutorial.md), [UiPathStudioO365App](https://www.uipath.com/product/platform), [Jedox](../saas-apps/jedox-tutorial.md), [Cequence Application Security](../saas-apps/cequence-application-security-tutorial.md), [PerimeterX](../saas-apps/perimeterx-tutorial.md), [TrendMiner](../saas-apps/trendminer-tutorial.md), [Lexion](../saas-apps/lexion-tutorial.md), [WorkWare](../saas-apps/workware-tutorial.md), [ProdPad](../saas-apps/prodpad-tutorial.md), [AWS ClientVPN](../saas-apps/aws-clientvpn-tutorial.md), [AppSec Flow SSO](../saas-apps/appsec-flow-sso-tutorial.md), [Luum](../saas-apps/luum-tutorial.md), [Freight Measure](https://www.gpcsl.com/freight.html), [Terraform Cloud](../saas-apps/terraform-cloud-tutorial.md), [Nature Research](../saas-apps/nature-research-tutorial.md), [Play Digital Signage](https://login.playsignage.com/login), [RemotePC](../saas-apps/remotepc-tutorial.md), [Prolorus](../saas-apps/prolorus-tutorial.md), [Hirebridge ATS](../saas-apps/hirebridge-ats-tutorial.md), [Teamgage](https://www.teamgage.com/Account/ExternalLoginAzure), [Roadmunk](../saas-apps/roadmunk-tutorial.md), [Sunrise Software Relations CRM](https://cloud.relations-crm.com/), [Procaire](../saas-apps/procaire-tutorial.md), [Mentor® by eDriving: Business](https://www.edriving.com/), [Gradle Enterprise](https://gradle.com/)

La documentation de toutes ces applications est disponible ici https://aka.ms/AppsTutorial

Pour référencer votre application dans la galerie d’applications Azure AD, lisez les informations détaillées ici : https://aka.ms/AzureADAppRequest

---

### <a name="public-preview---custom-roles-for-enterprise-apps"></a>Préversion publique : rôles personnalisés pour les applications d’entreprise

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** RBAC  
**Fonctionnalité de produit :** Contrôle d’accès
 
 [Les rôles RBAC personnalisés pour la gestion déléguée des applications d’entreprise](../users-groups-roles/roles-custom-available-permissions.md) sont désormais en préversion publique. Ces nouvelles autorisations s’appuient sur les rôles personnalisés pour la gestion de l’inscription des applications, ce qui permet un contrôle précis de l’accès de vos administrateurs. Au fil du temps, des autorisations supplémentaires pour déléguer la gestion d’Azure AD seront publiées.

Voici quelques scénarios de délégation courants :
- affectation des utilisateurs et groupes autorisés à accéder aux applications d’authentification unique SAML
- création d’applications de la galerie Azure AD
- lecture et mise à jour des configurations SAML de base pour les applications d’authentification unique SAML
- gestion des certificats de signature pour les applications d’authentification unique SAML
- mise à jour des adresses e-mail de notification d’expiration des certificats de connexion pour les applications d’authentification unique SAML
- mise à jour de la signature de jeton SAML et de l’algorithme de connexion pour les applications d’authentification unique SAML
- création, suppression et mise à jour des attributs et revendications utilisateur pour les applications d’authentification unique SAML
- capacité à activer, désactiver et redémarrer des travaux de provisionnement
- mises à jour du mappage des attributs
- capacité à lire les paramètres de provisionnement associés à l’objet
- capacité à lire les paramètres de provisionnement associés au principal de service
- capacité à autoriser l’accès aux applications pour le provisionnement

---

### <a name="azure-ad-application-proxy-natively-supports-single-sign-on-access-to-applications-that-use-headers-for-authentication"></a>Le Proxy d’application Azure AD prend en charge en mode natif l’accès en authentification unique aux applications qui utilisent des en-têtes pour l’authentification

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Proxy d’application  
**Fonctionnalité de produit :** Contrôle d’accès
 
Le Proxy d’application Azure Active Directory (Azure AD) prend en charge en mode natif l’accès en authentification unique aux applications qui utilisent des en-têtes pour l’authentification. Vous pouvez configurer les valeurs d’en-tête que votre application requiert dans Azure AD. Les valeurs d’en-tête seront envoyées à l’application via un Proxy d’application. Pour en savoir plus, consultez [Authentification unique basée sur l’en-tête pour les applications locales avec le Proxy d’application Azure AD](../manage-apps/application-proxy-configure-single-sign-on-with-headers.md)
 
---

### <a name="general-availability---azure-ad-b2c-phone-sign-up-and-sign-in-using-custom-policy"></a>Disponibilité générale : inscription et connexion par téléphone à Azure AD B2C à l’aide d’une stratégie personnalisée

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C

Avec l’inscription et la connexion par téléphone, les développeurs et les entreprises peuvent autoriser leurs clients à s’inscrire et à se connecter à l’aide d’un mot de passe à usage unique envoyé au numéro de téléphone de l’utilisateur via SMS. Cette fonctionnalité permet également au client de modifier son numéro de téléphone s’il perd l’accès à son téléphone. Grâce à la puissance des stratégies personnalisées, ces procédures permettent aux développeurs et aux entreprises de mettre leur marque en avant en personnalisant des pages. Découvrez comment [Configurer l’inscription et la connexion par téléphone avec des stratégies personnalisées dans Azure AD B2C](../../active-directory-b2c/phone-authentication.md).
 
---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---november-2020"></a>Nouveaux connecteurs de provisionnement dans la galerie d’applications Azure AD - Novembre 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Intégration tierce
 
Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Adobe Identity Management](../saas-apps/adobe-identity-management-provisioning-tutorial.md)
- [Blogin](../saas-apps/blogin-provisioning-tutorial.md)
- [Clarizen One](../saas-apps/clarizen-one-provisioning-tutorial.md)
- [Contentful](../saas-apps/contentful-provisioning-tutorial.md)
- [GitHub AE](../saas-apps/github-ae-provisioning-tutorial.md)
- [Playvox](../saas-apps/playvox-provisioning-tutorial.md)
- [PrinterLogic SaaS](../saas-apps/printer-logic-saas-provisioning-tutorial.md)
- [Tic - Tac Mobile](../saas-apps/tic-tac-mobile-provisioning-tutorial.md)
- [Visibly](../saas-apps/visibly-provisioning-tutorial.md)

Pour plus d’informations, consultez [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../manage-apps/user-provisioning.md).
 
---

### <a name="public-preview---email-sign-in-with-proxyaddresses-now-deployable-via-staged-rollout"></a>Préversion publique : connexion par e-mail avec ProxyAddresses maintenant déployable via un déploiement intermédiaire

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur
 
Les administrateurs de clients peuvent désormais utiliser le déploiement intermédiaire pour déployer la connexion par e-mail avec ProxyAddresses sur des groupes Azure AD spécifiques. Cela vous permet de tester la fonctionnalité avant de la déployer sur tout le client via la stratégie de découverte du domaine d’accueil. Les instructions de déploiement de la connexion par e-mail avec ProxyAddresses via le déploiement intermédiaire sont répertoriées dans la [documentation](../authentication/howto-authentication-use-email-signin.md).
 
---

### <a name="limited-preview---sign-in-diagnostic"></a>Préversion limitée : diagnostic de connexion

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Monitoring et création de rapports
 
Avec la préversion initiale du diagnostic de connexion, les administrateurs peuvent maintenant consulter les connexions utilisateur. Les administrateurs peuvent recevoir des informations contextuelles, spécifiques et pertinentes ainsi que des conseils sur ce qui s’est produit lors d’une connexion et sur la manière de résoudre les problèmes. Le diagnostic est disponible à la fois au niveau d’Azure AD et dans les panneaux de diagnostic et de résolution de l’accès conditionnel. Les scénarios de diagnostic couverts dans cette version sont Accès conditionnel, Multi-Factor Authentication et la connexion réussie.
 
---

### <a name="improved-unfamiliar-sign-in-properties"></a>Amélioration des propriétés de connexion inhabituelles

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Identity Protection  
**Fonctionnalité de produit :** Protection et sécurité des identités

  Les détections des propriétés de connexion inhabituelles ont été mises à jour. Les clients peuvent remarquer une augmentation des détections de propriétés de connexion inhabituelles à haut risque. Pour plus d’informations, consultez [Qu’est-ce que le risque ?](../identity-protection/concept-identity-protection-risks.md)
 
---

### <a name="public-preview-refresh-of-cloud-provisioning-agent-now-available-version-112810"></a>Actualisation de la préversion publique de l’agent de provisionnement cloud maintenant disponible (version : 1.1.281.0)

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Provisionnement cloud Azure AD  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
L’agent de provisionnement cloud a été publié préversion publique et est désormais disponible via le portail. Cette version contient plusieurs améliorations, notamment la prise en charge de GMSA pour vos domaines, ce qui offre une meilleure sécurité, des cycles de synchronisation initiaux améliorés et la prise en charge des grands groupes. Pour plus d’informations, consultez l’[historique](../app-provisioning/provisioning-agent-release-version-history.md) des versions. 
 
---

### <a name="bitlocker-recovery-key-api-endpoint-now-under-informationprotection"></a>Le point de terminaison de l’API de clé de récupération BitLocker se trouve maintenant sous /informationProtection

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Gestion de l’accès aux appareils  
**Fonctionnalité de produit :** Gestion du cycle de vie des appareils
 
Auparavant, vous pouviez récupérer les clés BitLocker via le point de terminaison /bitlocker. Nous allons déprécier ce point de terminaison, et les clients devront commencer à consommer l’API qui se trouve désormais sous /informationProtection. 

Consultez [API de récupération BitLocker](https://docs.microsoft.com/graph/api/resources/bitlockerrecoverykey?view=graph-rest-beta) pour obtenir des mises à jour de la documentation qui reflète ces modifications.

---

### <a name="general-availability-of-application-proxy-support-for-remote-desktop-services-html5-web-client"></a>Disponibilité générale de la prise en charge du proxy d’application pour le client web HTML5 des services Bureau à distance

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Proxy d’application  
**Fonctionnalité de produit :** Contrôle d’accès
 
La prise en charge du proxy d'application Azure AD pour le client web des services Bureau à distance (RDS) est désormais en disponibilité générale. Le client web RDS permet aux utilisateurs d'accéder à l'infrastructure Bureau distant via tout navigateur compatible HTLM5 tel que Microsoft Edge, Internet Explorer 11, Google Chrome, etc. Les utilisateurs peuvent interagir avec des applications ou des bureaux distants, comme ils le feraient avec un appareil local depuis n’importe où. 

En utilisant le Proxy d’application Azure AD, vous pouvez augmenter la sécurité de votre déploiement des services Bureau à distance en appliquant la pré-authentification et les stratégies d’accès conditionnel pour tous les types d’applications clientes riches. Pour en savoir plus, consultez [Publier le Bureau à distance avec le Proxy d’application d’Azure AD](../manage-apps/application-proxy-integrate-with-remote-desktop-services.md)
 
---

### <a name="new-enhanced-dynamic-group-service-is-in-public-preview"></a>Le nouveau service Groupe dynamique amélioré est en préversion publique

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Gestion des groupes  
**Fonctionnalité de produit :** Collaboration
 
Le service Groupe dynamique amélioré est maintenant en préversion publique. Les nouveaux clients qui créent des groupes dynamiques dans leurs locataires utiliseront le nouveau service. Le temps nécessaire à la création d’un groupe dynamique sera proportionnel à la taille du groupe créé et non plus à la taille du locataire. Cette mise à jour améliore de manière significative les performances des clients importants lorsque les clients créent des groupes plus petits. 

Le nouveau service vise également à finaliser en quelques minutes l’ajout et la suppression de membres en raison de modifications d’attributs. En outre, les échecs de traitement uniques ne bloquent pas le traitement du locataire. Pour en savoir plus sur la création de groupes dynamiques, consultez notre [documentation](../enterprise-users/groups-create-rule.md).
 
---
## <a name="october-2020"></a>Octobre 2020

### <a name="azure-ad-on-premises-hybrid-agents-impacted-by-azure-tls-certificate-changes"></a>Agents hybrides locaux Azure AD impactés par les changements des certificats Azure TLS

**Type :** Modification planifiée  
**Catégorie de service :** N/A  
**Fonctionnalité de produit :** plateforme

Microsoft met à jour les services Azure pour qu’ils utilisent des certificats TLS issus d’un autre ensemble d’autorités de certification racines. Cette mise à jour est due au fait que les certificats d’autorité de certification actuels ne sont pas conformes à l’une des réglementations de référence du CA/Browser Forum. Ce changement a un impact sur les agents hybrides Azure AD installés localement qui ont des environnements renforcés disposant d’une liste fixe de certificats racines. Ils devront être mis à jour pour approuver les nouveaux émetteurs de certificats.

Ce changement entraîne une interruption du service si vous n’agissez pas tout de suite. Ces agents comprennent les [connecteurs de proxy d’application](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AppProxy) pour l’accès à distance à un site local, les agents d’[authentification directe](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) qui permettent à vos utilisateurs de se connecter aux applications à l’aide des mêmes mots de passe, et les agents de la [préversion du provisionnement cloud](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) qui effectuent la synchronisation d’AD vers Azure AD. 

Si vous disposez d’un environnement avec des règles de pare-feu définies pour autoriser les appels sortants uniquement vers un téléchargement de liste de révocation de certificats spécifique, vous devez autoriser les URL de liste de révocation de certificats et OCSP suivantes. Pour plus d’informations sur le changement d’URL de liste de révocation de certificats et OCSP afin de permettre l’accès, consultez [Changements des certificats Azure TLS](../../security/fundamentals/tls-certificate-changes.md).

---

### <a name="provisioning-events-will-be-removed-from-audit-logs-and-published-solely-to-provisioning-logs"></a>Les événements de provisionnement sont supprimés des journaux d’audit et publiés uniquement dans les journaux de provisionnement

**Type :** Modification planifiée  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Monitoring et création de rapports
 
L’activité effectuée par le [service de provisionnement](../app-provisioning/user-provisioning.md) SCIM est journalisée à la fois dans les journaux d’audit et dans les journaux de provisionnement. Cela comprend les activités comme la création d’un utilisateur dans ServiceNow, la création d’un groupe dans GSuite ou l’importation d’un rôle à partir d’AWS. À l’avenir, ces événements seront publiés uniquement dans les journaux de provisionnement. Ce changement est implémenté pour éviter les événements en double dans les journaux et les coûts supplémentaires liés à l’utilisation de journaux dans Log Analytics par les clients. 

Nous fournirons une mise à jour quand une date de fin sera atteinte. Cette dépréciation n’est pas prévue pour l’année 2020. 

> [!NOTE]
> Cela n’affecte pas les événements figurant dans les journaux d’audit en dehors des événements de synchronisation émis par le service de provisionnement. Les événements comme la création d’une application, d’une stratégie d’accès conditionnel, d’un utilisateur dans l’annuaire, etc., continueront d’être émis dans les journaux d’audit. [Plus d’informations](../reports-monitoring/concept-provisioning-logs.md?context=azure%2factive-directory%2fapp-provisioning%2fcontext%2fapp-provisioning-context)
 

---

### <a name="azure-ad-on-premises-hybrid-agents-impacted-by-azure-transport-layer-security-tls-certificate-changes"></a>Agents hybrides locaux Azure AD impactés par les changements des certificats Azure TLS (Transport Layer Security)

**Type :** Modification planifiée  
**Catégorie de service :** N/A  
**Fonctionnalité de produit :** plateforme
 
Microsoft met à jour les services Azure pour qu’ils utilisent des certificats TLS issus d’un autre ensemble d’autorités de certification racines. Il y aura une mise à jour parce que les certificats d’autorité de certification actuels ne respectent pas l’une des réglementations de référence du CA/Browser Forum. Ce changement a un impact sur les agents hybrides Azure AD installés localement qui ont des environnements renforcés disposant d’une liste fixe de certificats racines. Ces agents devront être mis à jour pour approuver les nouveaux émetteurs de certificats.

Ce changement entraîne une interruption du service si vous n’agissez pas tout de suite. Ces agents sont notamment les suivants : 
- [Connecteurs de proxy d’application](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AppProxy) pour l’accès à distance à des applications locales 
- Agents d’[authentification directe](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) qui permettent à vos utilisateurs de se connecter à des applications à l’aide des mêmes mots de passe
- Agents de la [préversion du provisionnement cloud](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/AzureADConnect) qui effectuent la synchronisation d’AD vers Azure AD. 

Si vous disposez d’un environnement avec des règles de pare-feu définies pour autoriser les appels sortants uniquement vers un téléchargement de liste de révocation de certificats spécifique, vous devez autoriser les URL de liste de révocation de certificats et OCSP. Pour plus d’informations sur le changement d’URL de liste de révocation de certificats et OCSP afin de permettre l’accès, consultez [Changements des certificats Azure TLS](../../security/fundamentals/tls-certificate-changes.md).
 
---

### <a name="azure-active-directory-tls-10-tls-11-and-3des-deprecation-in-us-gov-cloud"></a>Dépréciation d’Azure Active Directory TLS 1.0, TLS 1.1 et 3DES dans le cloud du secteur public des États-Unis

**Type :** Modification planifiée  
**Catégorie de service :** Toutes les applications Azure AD  
**Fonctionnalité de produit :** Standards
 
Azure Active Directory dépréciera les protocoles suivants avant le 31 mars 2021 :
- TLS 1.0
- TLS 1.1
- Suite de chiffrement 3DES (TLS_RSA_WITH_3DES_EDE_CBC_SHA)

Toutes les combinaisons client-serveur et navigateur-serveur doivent utiliser TLS 1.2 et des suites de chiffrement modernes afin de maintenir une connexion sécurisée à Azure Active Directory pour les services Azure, Office 365 et Microsoft 365.

Les environnements affectés sont les suivants :
- Azure US Gov
- [Office 365 GCC High et DoD](/microsoft-365/compliance/tls-1-2-in-office-365-gcc)
 
---

### <a name="assign-applications-to-roles-on-au-and-object-scope"></a>Affecter des applications à des rôles sur l’étendue de l’unité administrative et de l’objet

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** RBAC  
**Fonctionnalité de produit :** Contrôle d’accès
 
Cette fonctionnalité permet d’attribuer une application (SPN) à un rôle administrateur sur l’étendue de l’unité administrative. Pour en savoir plus, reportez-vous à [Attribuer des rôles dont l’étendue est délimitée à une unité administrative](../roles/admin-units-assign-roles.md).

---

### <a name="now-you-can-disable-and-delete-guest-users-when-theyre-denied-access-to-a-resource"></a>Vous pouvez maintenant désactiver et supprimer des utilisateurs invités quand l’accès à une ressource leur est refusé

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Révisions d’accès  
**Fonctionnalité de produit :** Gouvernance des identités
 
Désactiver et supprimer est un contrôle avancé dans les révisions d’accès Azure AD pour permettre aux organisations de mieux gérer les invités externes dans des groupes et des applications. Si des invités sont refusés dans une révision d’accès, **la désactivation et la suppression** les empêchent automatiquement de se connecter pendant 30 jours. Au bout de 30 jours, ils sont complètement retirés du locataire.

Pour plus d’informations sur cette fonctionnalité, consultez [Désactiver et supprimer des identités externes à l’aide des révisions d’accès Azure AD (préversion)](../governance/access-reviews-external-users.md#disable-and-delete-external-identities-with-azure-ad-access-reviews-preview).
 
---

### <a name="access-review-creators-can-add-custom-messages-in-emails-to-reviewers"></a>Les créateurs de révision d’accès peuvent ajouter des messages personnalisés dans les e-mails envoyés aux réviseurs

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Révisions d’accès  
**Fonctionnalité de produit :** Gouvernance des identités
 
Dans les révisions d’accès Azure AD, les administrateurs qui créent des révisions peuvent désormais écrire un message personnalisé aux réviseurs. Les réviseurs voient le message dans l’e-mail qu’ils reçoivent, les invitant à effectuer la révision. Pour en savoir plus sur l’utilisation de cette fonctionnalité, consultez l’étape 14 de la section [Créer une ou plusieurs révisions d’accès](../governance/create-access-review.md#create-one-or-more-access-reviews).

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---october-2020"></a>Nouveaux connecteurs de provisionnement dans la galerie d’applications Azure AD - Octobre 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Intégration tierce
 
Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Apple Business Manager](../saas-apps/apple-business-manager-provision-tutorial.md)
- [Apple School Manager](../saas-apps/apple-school-manager-provision-tutorial.md)
- [Code42](../saas-apps/code42-provisioning-tutorial.md)
- [AlertMedia](../saas-apps/alertmedia-provisioning-tutorial.md)
- [OpenText Directory Services](../saas-apps/open-text-directory-services-provisioning-tutorial.md)
- [Cinode](../saas-apps/cinode-provisioning-tutorial.md)
- [Global Relay Identity Sync](../saas-apps/global-relay-identity-sync-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).
 
---

### <a name="integration-assistant-for-azure-ad-b2c"></a>Assistant d’intégration pour Azure AD B2C

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C
 
L’expérience de l’Assistant d’intégration (préversion) est maintenant disponible pour les inscriptions d’applications Azure AD B2C. Cette expérience vous guide dans la configuration de votre application pour les scénarios courants. Découvrez-en plus sur les [Bonnes pratiques et recommandations sur la plateforme d’identité Microsoft](../develop/identity-platform-integration-checklist.md).
 
---

### <a name="view-role-template-id-in-azure-portal-ui"></a>Afficher l’ID de modèle de rôle dans l’interface utilisateur du portail Azure

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Rôles Azure  
**Fonctionnalité de produit :** Contrôle d’accès
 

Vous pouvez maintenant afficher l’ID de modèle de chaque rôle Azure AD dans le portail Azure. Dans Azure AD, sélectionnez la **description** du rôle sélectionné. 

Il est recommandé aux clients d’utiliser des ID de modèle de rôle dans leur code et leur script PowerShell plutôt que le nom complet. L’utilisation d’ID de modèle de rôle est prise en charge dans les objets [directoryRoles](/graph/api/resources/directoryrole) et [roleDefinition](/graph/api/resources/unifiedroledefinition?view=graph-rest-beta). Pour plus d’informations sur les ID de modèle de rôle, consultez [ID de modèles de rôle](../roles/permissions-reference.md#role-template-ids).

---

### <a name="api-connectors-for-azure-ad-b2c-sign-up-user-flows-is-now-in-public-preview"></a>Les connecteurs d’API pour Azure AD B2C flux d’utilisateurs d’inscription sont maintenant en préversion publique

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C
 

Les connecteurs d’API peuvent désormais être utilisés avec Azure Active Directory B2C. Les connecteurs d’API vous permettent d’utiliser des API web pour personnaliser vos flux d’utilisateurs d’inscription et les intégrer à des systèmes cloud externes. Vous pouvez utiliser des connecteurs d’API pour :

- Effectuer une intégration à des workflows d’approbation personnalisés
- Valider des données d’entrée utilisateur
- Remplacer des attributs utilisateur 
- Exécuter une logique métier personnalisée 

 Pour en savoir plus, consultez la documentation [Utiliser des connecteurs d’API pour personnaliser et étendre les flux d’utilisateurs d’inscription](../../active-directory-b2c/api-connectors-overview.md).

---

### <a name="state-property-for-connected-organizations-in-entitlement-management"></a>Propriété État pour les organisations connectées dans la gestion des droits d’utilisation

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Gestion des annuaires **Fonctionnalité produit :** Gestion des droits d’utilisation
 

 Toutes les organisations connectées auront désormais une propriété supplémentaire appelée « État ». L’état contrôle la manière dont l’organisation connectée sera utilisée dans les stratégies qui font référence à « toutes les organisations connectées configurées ». La valeur est soit « configuré » (ce qui signifie que l’organisation est dans l’étendue des stratégies qui utilisent la clause « all »), soit « proposé » (ce qui signifie que l’organisation n’est pas dans l’étendue).  

Les organisations connectées créées manuellement auront le paramètre par défaut « configuré ». En attendant, celles créées automatiquement (par le biais de stratégies qui autorisent les utilisateurs d’Internet à demander l’accès) sont définies par défaut sur « proposé ».  Toutes les organisations connectées créées avant le 9 septembre 2020 seront définies sur « configuré ». Les administrateurs peuvent mettre à jour cette propriété si nécessaire. [Plus d’informations](../governance/entitlement-management-organization.md#managing-a-connected-organization-programmatically)
 

---

### <a name="azure-active-directory-external-identities-now-has-premium-advanced-security-settings-for-b2c"></a>Les identités externes Azure Active Directory ont désormais des paramètres de sécurité avancés Premium pour B2C

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C
 
L’accès conditionnel basé sur les risques et les fonctionnalités de détection des risques d’Identity Protection sont désormais disponibles dans [Azure AD B2C](../..//active-directory-b2c/conditional-access-identity-protection-overview.md). Grâce à ces fonctionnalités de sécurité avancées, les clients peuvent :
- Tirer parti d’insights intelligents pour évaluer les risques liés aux applications B2C et aux comptes d’utilisateur final. Les détections incluent les déplacements atypiques, les adresses IP anonymes, les adresses IP liées aux programmes malveillants et à Azure AD Threat Intelligence. Des rapports sur le portail et les API sont également disponibles.
- Gérez automatiquement les risques en configurant des stratégies d’authentification adaptative pour les utilisateurs B2C. Les développeurs d’applications et les administrateurs peuvent réduire les risques en temps réel en exigeant l’authentification multifacteur (MFA, Multi-Factor Authentication) ou le blocage de l’accès en fonction du niveau de risque utilisateur détecté, avec des contrôles supplémentaires disponibles suivant la localisation, le groupe et l’application.
- Effectuer une intégration à des flux d’utilisateurs Azure AD B2C et à des stratégies personnalisées. Les conditions peuvent être déclenchées à partir de flux d’utilisateurs intégrés dans Azure AD B2C ou peuvent être incorporées dans des stratégies personnalisées B2C. Comme avec d’autres aspects du flux d’utilisateurs B2C, la messagerie de l’expérience utilisateur final peut être personnalisée. La personnalisation dépend des alternatives de voix, de marque et d’atténuation de l’organisation.
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---october-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Octobre 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
En octobre 2020, nous avons ajouté à notre galerie d’applications les 27 nouvelles applications suivantes qui prennent en charge la fédération :

[Sentry](../saas-apps/sentry-tutorial.md), [Bumblebee - Productivity Superapp](https://app.yellowmessenger.com/user/login), [ABBYY FlexiCapture Cloud](../saas-apps/abbyy-flexicapture-cloud-tutorial.md), [EAComposer](../saas-apps/eacomposer-tutorial.md), [Genesys Cloud Integration pour Azure](https://apps.mypurecloud.com/msteams-integration/), [Portail Zone Technologie](https://portail.zonetechnologie.com/signin), [Beautiful.ai](../saas-apps/beautiful.ai-tutorial.md), [Datawiza Access Broker](https://console.datawiza.com/), [ZOKRI](https://app.zokri.com/), [CheckProof](../saas-apps/checkproof-tutorial.md), [Ecochallenge.org](https://events.ecochallenge.org/users/login), [atSpoke](http://atspoke.com/login), [Appointment Reminder](https://app.appointmentreminder.co.nz/account/login), [Cloud.Market](https://cloud.market/), [TravelPerk](../saas-apps/travelperk-tutorial.md), [Greetly](https://app.greetly.com/), [OrgVitality SSO}(../saas-apps/orgvitality-sso-tutorial.md), [Web Cargo Air](../saas-apps/web-cargo-air-tutorial.md), [Loop Flow CRM](../saas-apps/loop-flow-crm-tutorial.md), [Starmind](../saas-apps/starmind-tutorial.md), [Workstem](https://hrm.workstem.com/login), [Retail Zipline](../saas-apps/retail-zipline-tutorial.md), [Hoxhunt](../saas-apps/hoxhunt-tutorial.md), [MEVISIO](../saas-apps/mevisio-tutorial.md), [Samsara](../saas-apps/samsara-tutorial.md), [Nimbus](../saas-apps/nimbus-tutorial.md), [Pulse Secure virtual Traffic Manager](../saas-apps/pulse-secure-virtual-traffic-manager-tutorial.md)

La documentation de toutes ces applications est disponible ici https://aka.ms/AppsTutorial

Pour référencer votre application dans la galerie d’applications Azure AD, lisez les informations détaillées ici : https://aka.ms/AzureADAppRequest

---

### <a name="provisioning-logs-can-now-be-streamed-to-log-analytics"></a>Les journaux de provisionnement peuvent désormais être envoyés en streaming vers Log Analytics

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Monitoring et création de rapports
 

Publiez vos journaux de provisionnement dans Log Analytics afin d’effectuer les opérations suivantes :
- Stocker les journaux de provisionnement pendant plus de 30 jours
- Définir des alertes et des notifications personnalisées
- Générer des tableaux de bord pour visualiser les journaux
- Exécuter des requêtes complexes pour analyser les journaux 

Pour découvrir comment utiliser la fonctionnalité, consultez [Découvrir comment le provisionnement s’intègre aux journaux Azure Monitor](../app-provisioning/application-provisioning-log-analytics.md).
 
---

### <a name="provisioning-logs-can-now-be-viewed-by-application-owners"></a>Les journaux de provisionnement peuvent désormais être consultés par les propriétaires d’applications

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Monitoring et création de rapports
 
Vous pouvez désormais autoriser les propriétaires d’applications à superviser l’activité effectuée par le service de provisionnement et à résoudre les problèmes sans leur fournir un rôle privilégié ou faire du service informatique un goulot d’étranglement. [Plus d’informations](../reports-monitoring/concept-provisioning-logs.md)
 
---

### <a name="renaming-10-azure-active-directory-roles"></a>Renommage de 10 rôles Azure Active Directory

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Rôles Azure  
**Fonctionnalité de produit :** Contrôle d’accès
 
Certains rôles intégrés Azure Active Directory (AD) ont des noms qui diffèrent de ceux qui apparaissent dans le Centre d’administration Microsoft 365, le portail Azure AD et Microsoft Graph. Cette incohérence peut entraîner des problèmes dans les processus automatisés. Avec cette mise à jour, nous renommons 10 noms de rôles pour les rendre cohérents. Le tableau suivant contient les nouveaux noms de rôles :

![Tableau des nouveaux noms de rôles](media/whats-new/azure-role.png)

---

### <a name="azure-ad-b2c-support-for-auth-code-flow-for-spas-using-msal-js-2x"></a>Prise en charge d’Azure Active Directory B2C pour le flux de code d’authentification pour les applications monopages (SPA) utilisant MSAL.js 2.x

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C
 
MSAL.js version 2.x prend désormais en charge le flux de code d’autorisation pour les applications web monopages (SPA). Azure AD B2C prend désormais en charge l’utilisation du type d’application SPA sur le portail Azure et l’utilisation du flux de code d’autorisation MSAL.js avec PKCE pour les applications monopages. Cela permettra aux applications monopages utilisant Azure AD B2C de maintenir l’authentification unique avec les navigateurs les plus récents et de respecter les dernières recommandations concernant les protocoles d’authentification. Commencez avec le tutoriel [Inscrire une application monopage (SPA) dans Azure Active Directory B2C](../../active-directory-b2c/tutorial-register-spa.md).

---

### <a name="updates-to-remember-multi-factor-authentication-mfa-on-a-trusted-device-setting"></a>Mises à jour pour mémoriser Multi-Factor Authentication (MFA) sur un paramètre d’appareil approuvé

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** MFA  
**Fonctionnalité de produit :** Protection et sécurité des identités
 

Nous avons récemment mis à jour la [mémorisation de Multi-Factor Authentication (MFA)](../authentication/howto-mfa-mfasettings.md#remember-multi-factor-authentication) sur une fonctionnalité d’appareil approuvé pour étendre l’authentification jusqu’à 365 jours. Les licences Azure Active Directory (Azure AD) Premium peuvent également utiliser la [stratégie de fréquence de connexion pour l’accès conditionnel](../conditional-access/howto-conditional-access-session-lifetime.md#user-sign-in-frequency) qui offre plus de souplesse pour les paramètres de réauthentification.

Pour une expérience utilisateur optimale, nous vous recommandons d’utiliser la fréquence de connexion pour l’accès conditionnel afin d’étendre les durées de vie des sessions sur les appareils approuvés, les localisations ou les sessions à faible risque comme alternative à la mémorisation de Multi-Factor Authentication (MFA) sur un paramètre d’appareil approuvé. Pour commencer, consultez nos [dernières instructions sur l’optimisation de l’expérience de réauthentification](../authentication/concepts-azure-multi-factor-authentication-prompts-session-lifetime.md).

---

## <a name="september-2020"></a>Septembre 2020

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---september-2020"></a>Nouveaux connecteurs de provisionnement dans la galerie d’applications Azure AD - Septembre 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Intégration tierce
 
Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Coda](../saas-apps/coda-provisioning-tutorial.md)
- [Cofense Recipient Sync](../saas-apps/cofense-provision-tutorial.md)
- [InVision](../saas-apps/invision-provisioning-tutorial.md)
- [myday](../saas-apps/myday-provision-tutorial.md)
- [SAP Analytics Cloud](../saas-apps/sap-analytics-cloud-provisioning-tutorial.md)
- [Webroot Security Awareness](../saas-apps/webroot-security-awareness-training-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).
 
---
### <a name="cloud-provisioning-public-preview-refresh"></a>Actualisation de la préversion publique de provisionnement cloud

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement cloud Azure AD **Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
L’actualisation de la préversion publique de provisionnement cloud Azure AD Connect propose deux améliorations majeures développées à partir des commentaires des clients : 

- Expérience de mappage d’attributs par le biais du portail Azure

    Avec cette fonctionnalité, les administrateurs informatiques peuvent mapper des attributs d’utilisateur, de groupe ou de contact entre Active Directory et Azure AD à l’aide de différents types de mappage présents aujourd’hui. Le mappage d’attributs est une fonctionnalité utilisée pour standardiser les valeurs des attributs qui circulent d’Active Directory vers Azure Active Directory. Vous pouvez déterminer s’il faut mapper la valeur d’attribut telle quelle d’Active Directory vers Azure AD, ou utiliser des expressions pour transformer les valeurs d’attributs lors du provisionnement des utilisateurs. [En savoir plus](../cloud-provisioning/how-to-attribute-mapping.md)

- Provisionnement à la demande ou expérience de test utilisateur

    Une fois que vous avez paramétré votre configuration, vous souhaiterez peut-être effectuer un test pour voir si la transformation utilisateur fonctionne comme prévu avant de l’appliquer à tous les utilisateurs dans l’étendue. Avec le provisionnement à la demande, les administrateurs informatiques peuvent entrer le nom unique d’un utilisateur Active Directory et voir si la synchronisation fonctionne comme prévu. Le provisionnement à la demande offre un excellent moyen de s’assurer que les mappages d’attributs que vous avez effectués auparavant fonctionnent comme prévu. [En savoir plus](../cloud-provisioning/how-to-on-demand-provision.md)
 
---

### <a name="audited-bitlocker-recovery-in-azure-ad---public-preview"></a>Récupération BitLocker auditée dans Azure AD - Préversion publique

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Gestion de l’accès aux appareils  
**Fonctionnalité de produit :** Gestion du cycle de vie des appareils
 
Lorsque des administrateurs informatiques ou des utilisateurs finaux lisent des clés de récupération BitLocker auxquelles ils ont accès, Azure Active Directory génère désormais un journal d’audit qui capture les utilisateurs ayant accédé à la clé de récupération. Le même audit fournit des détails sur l’appareil auquel la clé BitLocker a été associée.

Les utilisateurs finaux peuvent [accéder à leurs clés de récupération par le biais de Mon compte](../user-help/my-account-portal-devices-page.md#view-a-bitlocker-key). Les administrateurs informatiques peuvent accéder aux clés de récupération par le biais de l’[API de clé de récupération BitLocker en version bêta](/graph/api/resources/bitlockerrecoverykey?view=graph-rest-beta) ou du portail Azure AD. Pour plus d’informations, consultez [Afficher ou copier des clés BitLocker dans le portail Azure AD](../devices/device-management-azure-portal.md#view-or-copy-bitlocker-keys).

---

### <a name="teams-devices-administrator-built-in-role"></a>Rôle intégré Administrateur d’appareils Teams

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** RBAC  
**Fonctionnalité de produit :** Contrôle d’accès
 
Les utilisateurs détenant rôle [Administrateur d’appareils Teams](../roles/permissions-reference.md#teams-devices-administrator) peuvent gérer des [appareils certifiés par Teams](https://www.microsoft.com/microsoft-365/microsoft-teams/across-devices/devices) à partir du Centre d’administration Teams. 

Ce rôle permet à l’utilisateur d’afficher tous les appareils en un seul coup d’œil, avec la possibilité de rechercher et de filtrer les appareils. L’utilisateur peut également vérifier les détails de chaque appareil, notamment le compte de connexion, la marque et le modèle de l’appareil. L’utilisateur peut modifier les paramètres sur l’appareil et mettre à jour les versions des logiciels. Ce rôle n’accorde pas d’autorisations pour vérifier l’activité Teams et la qualité d’appel de l’appareil.
 
---

### <a name="advanced-query-capabilities-for-directory-objects"></a>Fonctionnalités de requête avancées pour les objets d’annuaire

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** MS Graph  
**Fonctionnalité de produit :** Expérience de développement
 
Toutes les nouvelles fonctionnalités de requête introduites pour les objets d’annuaire dans les API Azure AD sont désormais disponibles dans le point de terminaison v1.0 et prêtes pour la production. Les développeurs peuvent compter, rechercher, filtrer et trier les objets d’annuaire et les liens connexes à l’aide des opérateurs OData standard.

Pour plus d’informations, reportez-vous à la documentation [ici](https://aka.ms/BlogPostMezzoGA). Vous pouvez également envoyer vos commentaires à l’aide de cette [brève enquête](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_yN8EPoGo5OpR1hgmCp1XxUMENJRkNQTk5RQkpWTE44NEk2U0RIV0VZRy4u).
 
---

### <a name="public-preview-continuous-access-evaluation-for-tenants-who-configured-conditional-access-policies"></a>Préversion publique : évaluation continue de l’accès pour les locataires qui ont configuré des stratégies d’accès conditionnel

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
L’évaluation continue de l’accès est désormais disponible en préversion publique pour les locataires Azure AD ayant des stratégies d’accès conditionnel. Avec elle, les stratégies et les événements de sécurité critiques sont évalués en temps réel. Cela comprend la désactivation du compte, la réinitialisation du mot de passe et la modification de l’emplacement. Pour plus d’informations, consultez [Évaluation continue de l’accès](../conditional-access/concept-continuous-access-evaluation.md).

---

### <a name="public-preview-ask-users-requesting-an-access-package-additional-questions-to-improve-approval-decisions"></a>Préversion publique : poser aux utilisateurs qui demandent un package d’accès des questions supplémentaires afin d’améliorer les décisions d’approbation

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Gestion de l’accès utilisateur  
**Fonctionnalité de produit :** Gestion des droits d’utilisation
 
Les administrateurs peuvent désormais exiger que les utilisateurs qui demandent un package d’accès répondent à des questions supplémentaires au-delà de la justification commerciale dans le portail Mon Accès de gestion des droits d’utilisation Azure AD. Les réponses des utilisateurs sont ensuite présentées aux approbateurs afin de les aider à prendre une décision d’approbation d’accès plus précise. Pour en savoir plus, consultez [Collecter des informations supplémentaires sur le demandeur pour approbation (préversion)](../governance/entitlement-management-access-package-approval-policy.md#collect-additional-requestor-information-for-approval-preview).
 
---

### <a name="public-preview-enhanced-user-management"></a>Préversion publique : gestion des utilisateurs améliorée

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** User Management  
**Fonctionnalité de produit :** User Management
 

Le portail Azure AD a été mis à jour afin de faciliter la recherche d’utilisateurs dans les pages Tous les utilisateurs et Utilisateurs supprimés. Les modifications apportées à la préversion sont les suivantes : 
- Propriétés utilisateur plus visibles, notamment l’ID d’objet, l’état de synchronisation d’annuaire, le type de création et l’émetteur d’identité.
- La fonctionnalité de recherche autorise désormais la recherche combinée de noms, d’e-mails et d’ID d’objets.
- Filtrage amélioré par type d’utilisateur (membre, invité et aucun), état de la synchronisation d’annuaire, type de création, nom de la société et nom de domaine.
- Nouvelles fonctionnalités de tri sur des propriétés telles que le nom, le nom d’utilisateur principal et la date de suppression.
- Nouveau nombre total d’utilisateurs mis à jour avec les recherches ou les filtres.

Pour plus d’informations, consultez [Améliorations de la gestion des utilisateurs (préversion) dans Azure Active Directory](../enterprise-users/users-search-enhanced.md).

---

### <a name="new-notes-field-for-enterprise-applications"></a>Nouveau champ de notes pour les applications d’entreprise

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Vous pouvez ajouter des notes de texte libres aux applications d’entreprise. Vous pouvez ajouter toute information pertinente qui vous aidera à gérer les applications Sous Applications d’entreprise. Pour plus d’informations, consultez [Démarrage rapide : Configurer les propriétés d’une application dans votre locataire Azure Active Directory (Azure AD)](../manage-apps/add-application-portal-configure.md). 

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---september-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD – Septembre 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce

En septembre 2020, nous avons ajouté les 34 applications suivantes à notre galerie d’applications avec prise en charge de la fédération :

[VMware Horizon - Unified Access Gateway](), [Pulse Secure PCS](../saas-apps/vmware-horizon-unified-access-gateway-tutorial.md), [Inventory360](../saas-apps/pulse-secure-pcs-tutorial.md), [Frontitude](https://services.enteksystems.de/sso/microsoft/signup), [BookWidgets](https://www.bookwidgets.com/sso/office365), [ZVD_Server](https://zaas.zenmutech.com/user/signin), [HashData for Business](https://hashdata.app/login.xhtml), [SecureLogin](https://securelogin.securelogin.nu/sso/azure/login), [CyberSolutions MAILBASEΣ/CMSS](../saas-apps/cybersolutions-mailbase-tutorial.md), [CyberSolutions CYBERMAILΣ](../saas-apps/cybersolutions-cybermail-tutorial.md), [LimbleCMMS](https://auth.limblecmms.com/), [Glint Inc](../saas-apps/glint-inc-tutorial.md), [zeroheight](../saas-apps/zeroheight-tutorial.md), [Gender Fitness](https://app.genderfitness.com/), [Coeo Portal](https://my.coeo.com/), [Grammarly](../saas-apps/grammarly-tutorial.md), [Fivetran](../saas-apps/fivetran-tutorial.md), [Kumolus](../saas-apps/kumolus-tutorial.md), [RSA Archer Suite](../saas-apps/rsa-archer-suite-tutorial.md), [TeamzSkill](../saas-apps/teamzskill-tutorial.md), [raumfürraum](../saas-apps/raumfurraum-tutorial.md), [Saviynt](../saas-apps/saviynt-tutorial.md), [BizMerlinHR](https://marketplace.bizmerlin.net/bmone/signup), [Mobile Locker](../saas-apps/mobile-locker-tutorial.md), [Zengine](../saas-apps/zengine-tutorial.md), [CloudCADI](https://app.cloudcadi.com/login), [Simfoni Analytics](https://simfonianalytics.com/accounts/microsoft/login/), [Priva Identity & Access Management](https://my.priva.com/), [Nitro Pro](https://www.gonitro.com/nps/product-details/downloads), [Eventfinity](../saas-apps/eventfinity-tutorial.md), [Fexa](../saas-apps/fexa-tutorial.md), [Secured Signing Enterprise Portal](https://www.securedsigning.com/aad/Auth/ExternalLogin/AdminPortal), [Secured Signing Enterprise Portal AAD Setup](https://www.securedsigning.com/aad/Auth/ExternalLogin/AdminPortal), [Wistec Online](https://wisteconline.com/auth/oidc), [Oracle PeopleSoft - Protected by F5 BIG-IP APM](../saas-apps/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial.md)

La documentation de toutes ces applications est disponible ici : https://aka.ms/AppsTutorial.

Pour référencer votre application dans la galerie d’applications Azure AD, lisez les informations détaillées ici : https://aka.ms/AzureADAppRequest.

---

### <a name="new-delegation-role-in-azure-ad-entitlement-management-access-package-assignment-manager"></a>Nouveau rôle de délégation dans la gestion des droits d’utilisation Azure AD : Gestionnaire d'attribution de package d'accès

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Gestion de l’accès utilisateur  
**Fonctionnalité de produit :** Gestion des droits d’utilisation
 
Un nouveau rôle Gestionnaire d’attribution de package d’accès a été ajouté dans la gestion des droits d’utilisation Azure AD afin de fournir des autorisations précises pour gérer les attributions. Vous pouvez maintenant déléguer des tâches à un utilisateur détenteur ce rôle, qui peut déléguer la gestion des attributions d’un package d’accès à un dirigeant d’entreprise. Toutefois, un Gestionnaire d’attribution de package d’accès ne peut pas modifier les stratégies de package d’accès ou d’autres propriétés définies par les administrateurs. 

Avec ce nouveau rôle, vous bénéficiez des privilèges minimaux nécessaires pour déléguer la gestion des attributions et conserver le contrôle administratif de toutes les autres configurations de package d’accès. Pour plus d’informations, consultez [Rôles de gestion des droits d’utilisation](../governance/entitlement-management-delegate.md#entitlement-management-roles).
 
---

### <a name="changes-to-privileged-identity-managements-onboarding-flow"></a>Modifications apportées au flux d’intégration de Privileged Identity Management

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Privileged Identity Management  
**Fonctionnalité de produit :** Privileged Identity Management
 
Auparavant, l’intégration à Privileged Identity Management (PIM) nécessitait le consentement de l’utilisateur et un flux d’intégration dans le panneau PIM qui incluait l’inscription à Azure AD MFA. Avec la récente intégration de l’expérience PIM dans le panneau de rôles et administrateurs Azure AD, nous supprimons cette expérience. Tout locataire ayant une licence P2 valide sera intégré automatiquement à PIM.

L’intégration à PIM n’a aucun effet indésirable direct sur votre locataire. Vous pouvez vous attendre aux changements suivants :
- Options d’attribution supplémentaires telles que actif/éligible avec heure de début et heure de fin lorsque vous effectuez une attribution dans PIM ou dans le panneau de rôles et administrateurs Azure AD. 
- Mécanismes d’étendue supplémentaires, tels que des unités administratives et des rôles personnalisés, introduits directement dans l’expérience d’attribution. 
- Si vous êtes administrateur général ou administrateur de rôle privilégié, vous commencerez peut-être à recevoir quelques e-mails supplémentaires tels que le résumé hebdomadaire PIM. 
- Vous constaterez peut-être aussi la présence du principal de service ms-pim dans le journal d’audit relatif à l’attribution de rôle. Ce changement attendu ne doit pas avoir d’incidence sur votre flux de travail normal.

 Pour plus d’informations, consultez [Commencer à utiliser Privileged Identity Management](../privileged-identity-management/pim-getting-started.md).

---

### <a name="azure-ad-entitlement-management-the-select-pane-of-access-package-resources-now-shows-by-default-the-resources-currently-in-the-selected-catalog"></a>Gestion des droits d’utilisation Azure AD : le volet Sélectionner des ressources de package d’accès affiche désormais par défaut les ressources actuellement dans le catalogue sélectionné

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Gestion de l’accès utilisateur  
**Fonctionnalité de produit :** Gestion des droits d’utilisation
 

Dans le flux de création de package d’accès, sous l’onglet Rôles des ressources, le comportement du volet Sélectionner change. Actuellement, le comportement par défaut consiste à afficher toutes les ressources qui sont détenues par l’utilisateur et les ressources ajoutées au catalogue sélectionné. 

Cette expérience sera changée de façon à afficher uniquement les ressources actuellement ajoutées dans le catalogue par défaut, afin que les utilisateurs puissent facilement sélectionner des ressources à partir du catalogue. La mise à jour facilitera la découverte des ressources à ajouter aux packages d’accès, et réduira les risques liés à l’ajout par inadvertance de ressources détenues par l’utilisateur qui ne font pas partie du catalogue. Pour plus d’informations, consultez [Créer un package d’accès dans la gestion des droits d’utilisation Azure AD](../governance/entitlement-management-access-package-create.md#resource-roles).
 
---

## <a name="august-2020"></a>Août 2020 
 
### <a name="updates-to-azure-multi-factor-authentication-server-firewall-requirements"></a>Mises à jour de la Configuration requise du pare-feu du serveur Azure Multi-Factor Authentication

**Type :** Modification planifiée  
**Catégorie de service :** MFA  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
À partir du 1er octobre 2020, la configuration requise du pare-feu du serveur Azure MFA exigera des plages d’adresses IP supplémentaires.

Si des règles de pare-feu pour le trafic sortant sont en vigueur au sein de votre organisation, mettez à jour les règles de façon à ce que vos serveurs MFA puissent communiquer avec toutes les plages d’adresses IP nécessaires. Les plages d’adresses IP sont documentées dans [Configuration requise du pare-feu du serveur Azure Multi-Factor Authentication](../authentication/howto-mfaserver-deploy.md#azure-multi-factor-authentication-server-firewall-requirements).

---

### <a name="upcoming-changes-to-user-experience-in-identity-secure-score"></a>Modifications à venir de l’expérience utilisateur dans le score d’identité sécurisée

**Type :** Modification planifiée  
**Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Nous mettons à jour le portail Score d’identité sécurisée pour l’aligner sur les modifications introduites dans la [nouvelle version](/microsoft-365/security/mtp/microsoft-secure-score-whats-new) du Niveau de sécurité Microsoft. 

La préversion avec les modifications sera disponible début septembre. Les modifications apportées à la préversion sont les suivantes :
- Le « Score d’identité sécurisée » est renommé « Niveau de sécurité de l’identité » pour l’alignement avec Niveau de sécurité Microsoft
- Points normalisés à l’échelle standard et exprimés en pourcentages au lieu de points

Dans cette préversion, les clients peuvent basculer entre l’expérience existante et la nouvelle expérience. Cette préversion publique sera disponible jusqu’à la fin du mois de novembre 2020. Après la préversion, les clients seront automatiquement dirigés vers la nouvelle expérience utilisateur.

---

### <a name="new-restricted-guest-access-permissions-in-azure-ad---public-preview"></a>Nouvelles autorisations d’accès invité restreintes dans Azure AD – Préversion publique

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Contrôle d’accès   
**Fonctionnalité de produit :** User Management

Nous avons mis à jour les autorisations au niveau répertoire pour les utilisateurs invités. Ces autorisations permettent aux administrateurs d’imposer des restrictions et contrôles supplémentaires à l’accès utilisateur invité externe. Les administrateurs peuvent désormais ajouter des restrictions à l’accès d’invités externes aux informations d’appartenance et de profil des utilisateurs et des groupes. Grâce à cette fonctionnalité en préversion publique, les clients peuvent gérer l’accès des utilisateurs externes à grande échelle en obfusquant les appartenances de groupe, notamment en empêchant les utilisateurs invités de voir les appartenances des groupes dans lesquels ils se trouvent.

Pour plus d’informations, consultez [Autorisations d’accès invité restreintes](../enterprise-users/users-restrict-guest-permissions.md) et [Autorisations par défaut des utilisateurs](./users-default-permissions.md).
 
---

### <a name="general-availability-of-delta-queries-for-service-principals"></a>Disponibilité générale des requêtes delta pour les principaux de service

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** MS Graph  
**Fonctionnalité de produit :** Expérience de développement
 
La requête delta Microsoft Graph prend désormais en charge le type de ressource dans la version v1.0 :
- Principal de service

Désormais, les clients peuvent suivre efficacement les modifications apportées à ces ressources et le service offre la meilleure solution pour synchroniser les modifications apportées à ces ressources avec un magasin de données local. Pour savoir comment configurer ces ressources dans une requête, consultez [Utiliser des requêtes delta pour suivre les modifications apportées aux données de Microsoft Graph](/graph/delta-query-overview).
 
---

### <a name="general-availability-of-delta-queries-for-oauth2permissiongrant"></a>Disponibilité générale des requêtes Delta pour oAuth2PermissionGrant

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** MS Graph  
**Fonctionnalité de produit :** Expérience de développement

La requête delta Microsoft Graph prend désormais en charge le type de ressource dans la version v1.0 :
- OAuth2PermissionGrant

Les clients peuvent désormais suivre efficacement les modifications apportées à ces ressources et le service offre la meilleure solution pour synchroniser les modifications apportées à ces ressources avec un magasin de données local. Pour savoir comment configurer ces ressources dans une requête, consultez [Utiliser des requêtes delta pour suivre les modifications apportées aux données de Microsoft Graph](/graph/delta-query-overview).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---august-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD – Août 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce

En août 2020, nous avons ajouté les 25 applications suivantes à notre galerie d’applications avec prise en charge de la fédération :

[Backup365](https://portal.backup365.io/login), [Soapbox](https://app.soapboxhq.com/create?step=auth&provider=azure-ad2-oauth2), [Alma SIS](https://almau.getalma.com/), [Enlyft Dynamics 365 Connector](http://enlyft.com/), [Serraview Space Utilization Software Solutions](../saas-apps/serraview-space-utilization-software-solutions-tutorial.md), [Uniq](https://web.uniq.app/), [Visibly](../saas-apps/visibly-tutorial.md), [Zylo](../saas-apps/zylo-tutorial.md), [Edmentum - Courseware Assessments Exact Path](https://auth.edmentum.com/elf/login), [CyberLAB](https://cyberlab.evolvesecurity.com/#/welcome), [Altamira HRM](../saas-apps/altamira-hrm-tutorial.md), [WireWheel](../saas-apps/wirewheel-tutorial.md), [Zix Compliance and Capture](https://sminstall.zixcorp.com/teams/teams.php?install_request=true&tenant_id=common), [Greenlight Enterprise Business Controls Platform](../saas-apps/greenlight-enterprise-business-controls-platform-tutorial.md), [Genetec Clearance](https://www.clearance.network/), [iSAMS](../saas-apps/isams-tutorial.md), [VeraSMART](../saas-apps/verasmart-tutorial.md), [Amiko](https://amiko.web.rivero.app/), [Twingate](https://auth.twingate.com/signup), [Funnel Leasing](https://nestiolistings.com/sso/oidc/azure/authorize/), [Scalefusion](https://scalefusion.com/users/sign_in/), [Bpanda](https://goto.bpanda.com/login), [Vivun Calendar Connect](https://app.vivun.com/dashboard/calendar/connect), [FortiGate SSL VPN](../saas-apps/fortigate-ssl-vpn-tutorial.md), [Wandera End User](https://www.wandera.com/)

La documentation de toutes ces applications est disponible ici https://aka.ms/AppsTutorial

Pour référencer votre application dans la galerie d’applications Azure AD, lisez les informations détaillées ici : https://aka.ms/AzureADAppRequest

---

### <a name="resource-forests-now-available-for-azure-ad-ds"></a>Les forêts de ressources sont désormais disponibles pour Azure AD DS 

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Services de domaine Azure AD   
**Fonctionnalité de produit :** Services de domaine Azure AD
 
La capacité des forêts de ressources dans Azure AD Domain Services est désormais généralement disponible. Vous pouvez désormais activer l’autorisation sans synchronisation de hachage de mot de passe pour utiliser Azure AD Domain Services, notamment l’autorisation par carte à puce. Pour en savoir plus, consultez [Concepts et fonctionnalités des jeux de réplicas pour Azure Active Directory Domain Services (préversion)](../../active-directory-domain-services/concepts-replica-sets.md).
 
---

### <a name="regional-replica-support-for-azure-ad-ds-managed-domains-now-available"></a>Prise en charge des réplicas régionaux pour les domaines gérés par Azure AD DS désormais disponible

**Type :** Nouvelle fonctionnalité   
**Catégorie de service :** Services de domaine Azure AD  
**Fonctionnalité de produit :** Services de domaine Azure AD
 
Vous pouvez étendre un domaine managé pour avoir plusieurs jeux de réplicas par locataire Azure AD. Les jeux de réplicas peuvent être ajoutés à n’importe quel réseau virtuel appairé dans toute région Azure prenant en charge Azure Active Directory Domain Services. D’autres jeux de réplicas dans des régions Azure différentes assurent la récupération d’urgence géographique pour les applications héritées si une région Azure est mise hors connexion. Pour en savoir plus, consultez [Concepts et fonctionnalités des jeux de réplicas pour Azure Active Directory Domain Services (préversion)](../../active-directory-domain-services/concepts-replica-sets.md).

---

### <a name="general-availability-of-azure-ad-my-sign-ins"></a>Disponibilité générale de Mes connexions Azure AD

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Expériences d’utilisateur final
 
La nouvelle fonctionnalité Mes connexions Azure AD permet aux utilisateurs d’entreprise d’examiner leur historique de connexion pour vérifier l’absence de toute activité inhabituelle. Elle permet en outre aux utilisateurs finaux de signaler s’ils étaient ou non à l’origine d’activités suspectes. Pour en savoir plus sur l’utilisation de cette fonctionnalité, consultez [Afficher et rechercher votre activité de connexion récente à partir de la page Mes connexions](../user-help/my-account-portal-sign-ins-page.md#confirm-unusual-activity).
 
---

### <a name="sap-successfactors-hr-driven-user-provisioning-to-azure-ad-is-now-generally-available"></a>L’approvisionnement piloté par SAP SuccessFactors RH d’utilisateurs vers Azure AD est désormais généralement disponible

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Vous pouvez à présent intégrer SAP SuccessFactors en tant que source d’identité faisant autorité avec Azure AD, et automatiser le cycle de vie de bout en bout des identités à l’aide d’événements de RH tels que de nouvelles embauches et des démissions pour conduire l’approvisionnement et le désapprovisionnement de comptes dans Azure AD. 

Pour en savoir plus sur la configuration de l’approvisionnement entrant de SAP SuccessFactors vers Azure AD, consultez le didacticiel [Configurer l’approvisionnement d’utilisateurs SAP SuccessFactors vers Active Directory](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md).
 
---

### <a name="custom-open-id-connect-ms-graph-api-support-for-azure-ad-b2c"></a>Prise en charge personnalisée de l’API MS Graph Open ID Connect pour Azure AD B2C

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C
 
Auparavant, des fournisseurs d’Open ID Connect personnalisés ne pouvaient être ajoutés ou gérés que via le portail Azure. Les clients Azure AD B2C peuvent désormais ajouter et gérer ces fournisseurs également via la version bêta des API Microsoft Graph. Pour savoir comment configurer cette ressource à l’aide d’API, consultez [type de ressource identityProvider](/graph/api/resources/identityprovider?view=graph-rest-beta).
 
---

### <a name="assign-azure-ad-built-in-roles-to-cloud-groups"></a>Affecter des rôles intégrés Azure AD à des groupes cloud

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Rôles Azure AD  
**Fonctionnalité de produit :** Contrôle d’accès

Vous pouvez désormais attribuer des rôles intégrés Azure AD à des groupes cloud avec cette nouvelle fonctionnalité. Par exemple, vous pouvez attribuer le rôle Administrateur SharePoint au groupe Contoso_SharePoint_Admins. Vous pouvez également utiliser PIM pour faire du groupe un membre éligible du rôle, au lieu d’accorder un accès permanent. Pour découvrir comment configurer cette fonctionnalité, consultez [Utiliser des groupes cloud pour gérer les attributions de rôles dans Azure Active Directory (préversion)](../roles/groups-concept.md).
 
---

### <a name="insights-business-leader-built-in-role-now-available"></a>Rôle intégré Leader d’entreprise Insights désormais disponible

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Rôles Azure AD  
**Fonctionnalité de produit :** Contrôle d’accès
 
Les utilisateurs titulaires du rôle Leader d’entreprise Insights peuvent accéder à un ensemble de tableaux de bord et d’insights via l’[application M365 Insights](https://www.microsoft.com/microsoft-365/partners/workplaceanalytics). Ceci inclut l’accès total à tous les tableaux de bord et aux insights présentés et la fonctionnalité d’exploration de données. Les utilisateurs titulaires de ce rôle n’ont pas accès aux paramètres de configuration du produit. Cette responsabilité est dévolue au rôle Administrateur Insights. Pour en savoir plus, consultez [Autorisations des rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md#insights-business-leader).
 
---

### <a name="insights-administrator-built-in-role-now-available"></a>Rôle intégré Administrateur Insights désormais disponible

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Rôles Azure AD  
**Fonctionnalité de produit :** Contrôle d’accès
 
Les utilisateurs titulaires du rôle Insights Administrator peuvent accéder à l’ensemble des fonctionnalités d’administration de l’[application M365 Insights](https://www.microsoft.com/microsoft-365/partners/workplaceanalytics). Un utilisateur titulaire de rôle peut lire les informations d’annuaire, surveiller l’intégrité du service, soumettre des tickets de support et accéder aux paramètres d’Administrateur Insights. Pour en savoir plus, consultez [Autorisations des rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md#insights-administrator).
 
--- 

### <a name="application-admin-and-cloud-application-admin-can-manage-extension-properties-of-applications"></a>Un Administrateur d’application ou un Administrateur d’application cloud peuvent gérer les propriétés d’extension des applications

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Rôles Azure AD  
**Fonctionnalité de produit :** Contrôle d’accès
 
Auparavant, seul l’Administrateur général pouvait gérer la [propriété d’extension](/graph/api/application-post-extensionproperty?view=graph-rest-beta&tabs=http). Nous allons désormais activer cette fonctionnalité pour l’Administrateur d’application et l’Administrateur d’application cloud.
 
---

### <a name="mim-2016-sp2-hotfix-462630-and-connectors-1113010"></a>Correctif logiciel MIM 2016 SP2 4.6.263.0 et connecteurs 1.1.1301.0

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Microsoft Identity Manager  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités

Un [correctif cumulatif (build 4.6.263.0)](https://support.microsoft.com/help/4576473/hotfix-rollup-package-build-4-6-263-0-is-available-for-microsoft-ident) est disponible pour Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2). Ce package cumulatif contient des mises à jour pour les composants MIM CM, Gestionnaire de synchronisation MIM et PAM. En outre, la build de connecteurs génériques MIM 1.1.1301.0 inclut des mises à jour du connecteur Graph.

---
 
## <a name="july-2020"></a>Juillet 2020

### <a name="as-an-it-admin-i-want-to-target-client-apps-using-conditional-access"></a>En tant qu’administrateur informatique, je souhaite cibler des applications clientes à l’aide de l’accès conditionnel

**Type :** Modification planifiée   
**Catégorie de service :** Accès conditionnel  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Avec la version en disponibilité générale de la condition des applications clientes dans l’accès conditionnel, les nouvelles stratégies s’appliquent désormais par défaut à toutes les applications clientes. Cela inclut les clients d’authentification hérités. Les stratégies existantes resteront inchangées, mais le bouton *ConfigurerOui/Non* sera supprimé des stratégies existantes pour identifier facilement les applications clientes qui sont appliquées par la stratégie. 

Lorsque vous créez une nouvelle stratégie, veillez à exclure les utilisateurs et les comptes de service qui utilisent encore l’authentification héritée. Si ce n’est pas le cas, ils seront bloqués. [Plus d’informations](../conditional-access/concept-conditional-access-conditions.md)
 
---

### <a name="upcoming-scim-compliance-fixes"></a>Prochains correctifs de conformité SCIM

**Type :** Modification planifiée  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Le service de provisionnement Azure AD utilise la norme SCIM pour l’intégration avec les applications. Notre implémentation de la norme SCIM est en cours d’évolution et nous nous attendons à apporter des modifications à notre comportement sur la façon dont nous effectuons les opérations PATCH, ainsi que la définition de la propriété « active » sur une ressource. [Plus d’informations](../app-provisioning/application-provisioning-config-problem-scim-compatibility.md)
 
---

### <a name="group-owner-setting-on-azure-admin-portal-will-be-changed"></a>Le paramètre propriétaire du groupe sur le portail d’administration Azure sera modifié

**Type :** Modification planifiée  
**Catégorie de service :** Gestion des groupes  
**Fonctionnalité de produit :** Collaboration

Les paramètres de propriétaire sur la page des paramètres généraux des groupes peuvent être configurés pour restreindre les privilèges d’attribution de propriétaire à un groupe limité d’utilisateurs dans le portail d’administration Azure et le panneau d’accès. Nous aurons bientôt la possibilité d’affecter des privilèges de propriétaire de groupe non seulement sur ces deux portails d’expérience utilisateur, mais également d’appliquer la stratégie sur le serveur principal pour fournir un comportement cohérent entre les points de terminaison, tels que PowerShell et Microsoft Graph. 

Nous allons commencer à désactiver le paramètre actuel pour les clients qui ne l’utilisent pas et offriront une option d’étendue aux utilisateurs pour le privilège de propriétaire de groupe dans les prochains mois. Pour obtenir des conseils sur la mise à jour des paramètres de groupe, consultez modifier les informations de votre groupe à l’aide d’[Azure Active Directory](./active-directory-groups-settings-azure-portal.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context).

---

### <a name="azure-active-directory-registration-service-is-ending-support-for-tls-10-and-11"></a>Le service d’inscription Azure Active Directory met fin à la prise en charge de TLS 1.0 et 1.1

**Type :** Modification planifiée  
**Catégorie de service :** Gestion et inscription des appareils  
**Fonctionnalité de produit :** plateforme

Transport Layer Security (TLS) 1.2 et les serveurs et clients de mise à jour communiqueront bientôt avec Azure Active Directory Device Registration service. Le support de TLS 1.0 et 1.1 pour la communication avec Azure AD Device Registration service sera retiré :
- Le 31 août 2020, dans tous les clouds souverains (GCC High, DoD, etc.)
- Le 30 octobre 2020, dans tous les clouds commerciaux

[En savoir plus](../devices/reference-device-registration-tls-1-2.md) sur le protocole TLS 1.2 pour Azure AD Registration Service.

---

### <a name="windows-hello-for-business-sign-ins-visible-in-azure-ad-sign-in-logs"></a>Connexions Windows Hello entreprise visibles dans les journaux de connexion Azure AD

**Type :** Résolution  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Monitoring et création de rapports
 
Windows Hello Entreprise permet aux utilisateurs finaux de se connecter à des ordinateurs Windows en un geste (par exemple, un code confidentiel ou une biométrie). Les administrateurs Azure AD peuvent souhaiter différencier les connexions Windows Hello Entreprise des autres connexions Windows dans le cadre de l’authentification sans mot de passe d’une organisation. 

Les administrateurs peuvent maintenant voir si une authentification Windows a utilisé Windows Hello Entreprise en consultant l’onglet Détails de l’authentification pour un événement de connexion Windows dans le volet Connexions Azure AD dans le portail Azure. Les authentifications Windows Hello Entreprise incluront « WindowsHelloForBusiness » dans le champ méthode d’authentification. Pour plus d’informations sur l’interprétation des journaux de connexion, reportez-vous à la [Documentation des journaux de connexion](../reports-monitoring/concept-sign-ins.md).
 
---

### <a name="fixes-to-group-deletion-behavior-and-performance-improvements"></a>Résolutions pour le comportement de suppression des groupes et améliorations des performances

**Type :** Résolution  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Auparavant, lorsqu'un groupe passait de « in-scope » à « out of scope » et qu'un administrateur cliquait sur « restart » avant que la modification ne soit terminée, l'objet du groupe n'était pas supprimé. À présent, l’objet de groupe sera supprimé de l’application cible lorsqu’il sort de l’étendue (désactivé, supprimé, non assigné ou échec du filtre d’étendue). [Plus d’informations](../app-provisioning/how-provisioning-works.md#incremental-cycles)
 
---

### <a name="public-preview-admins-can-now-add-custom-content-in-the-email-to-reviewers-when-creating-an-access-review"></a>Préversion publique : Les administrateurs peuvent désormais ajouter du contenu personnalisé à l’e-mail aux réviseurs lors de la création d’une révision d’accès

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Révisions d’accès  
**Fonctionnalité de produit :** Gouvernance des identités
 
Lorsqu’une nouvelle révision d’accès est créée, le réviseur reçoit un e-mail lui demandant d’effectuer la révision d’accès. La plupart de nos clients ont demandé la possibilité d’ajouter du contenu personnalisé à l’e-mail, comme des informations de contact, ou d’autres contenus de support supplémentaires pour guider le réviseur. 

Désormais disponible en préversion publique, les administrateurs peuvent spécifier du contenu personnalisé dans l’e-mail envoyé aux réviseurs en ajoutant du contenu dans la section « avancé » des révisions d’accès Azure AD. Pour des conseils sur la création d’une révision d’accès des groupes, consultez [Créer une révision d’accès des groupes et applications dans les révisions d’accès Azure AD](../governance/create-access-review.md).
 
---

### <a name="authorization-code-flow-for-single-page-apps-available"></a>Flux du code d’autorisation pour les applications à page unique disponible

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Expérience de développement
 
Du fait des restrictions sur les cookies tiers dans les navigateurs modernes, telles que ITP de Safari, les application à page unique (SPA) doivent utiliser le flux du code d’autorisation, plutôt que le flux implicite, pour maintenir l’authentification unique. MSAL.js v 2.x prend désormais en charge le flux du code d’autorisation. 

Des mises à jour correspondantes étant présentes sur le portail Azure, vous pouvez actualiser votre SPA pour qu’il soit de type « spa » et utilise le flux du code d’authentification. Pour plus de conseils, consultez [Connecter des utilisateurs et obtenir un jeton d’accès dans une application à page unique JavaScript à l’aide du flux de code d’authentification](../develop/quickstart-v2-javascript-auth-code.md).
 
---

### <a name="azure-ad-application-proxy-now-supports-the-remote-desktop-services-web-client"></a>Le Proxy d’application Azure AD prend désormais en charge le Client Web Services Bureau à distance

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Proxy d’application  
**Fonctionnalité de produit :** Contrôle d’accès

Le Proxy d’application Azure AD prend désormais en charge le Client Web Services Bureau à distance (RDS). Le client web RDS permet aux utilisateurs d'accéder à l'infrastructure Bureau distant via tout navigateur compatible HTLM5 tel que Microsoft Edge, Internet Explorer 11, Google Chrome, etc. Les utilisateurs peuvent interagir avec des applications ou des bureaux distants, comme ils le feraient avec un appareil local depuis n’importe où. En utilisant le Proxy d’application Azure AD, vous pouvez augmenter la sécurité de votre déploiement des services Bureau à distance en appliquant la pré-authentification et les stratégies d’accès conditionnel pour tous les types d’applications clientes riches. Pour plus d’informations, consultez [Publier le Bureau à distance avec le Proxy d’application d’Azure AD](../manage-apps/application-proxy-integrate-with-remote-desktop-services.md).
 
---

### <a name="next-generation-azure-ad-b2c-user-flows-in-public-preview"></a>Flux utilisateur Azure AD B2C de nouvelle génération en préversion publique

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C
 
L'expérience de flux d'utilisateurs simplifiée offre la parité des fonctions avec les fonctions d'aperçu et est le foyer de toutes les nouvelles fonctionnalités. Les utilisateurs pourront activer de nouvelles fonctionnalités au sein du même flux d'utilisateurs, ce qui réduira la nécessité de créer plusieurs versions à chaque nouvelle sortie de fonctionnalité. Enfin, la nouvelle expérience utilisateur conviviale simplifie la sélection et la création de flux utilisateur. Essayez-la dès maintenant en [créant flux d’utilisateur](../../active-directory-b2c/tutorial-create-user-flows.md). 

Pour plus d'informations sur les flux d'utilisateurs, voir [Versions des flux d'utilisateurs dans Azure Active Directory B2C](../../active-directory-b2c/user-flow-versions.md).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---july-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Juillet 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
En juillet 2020, nous avons ajouté les 55 applications suivantes à notre galerie d’applications avec prise en charge de la fédération :

[Clap Your Hands](http://www.rmit.com.ar/), [Appreiz](https://microsoftteams.appreiz.com/), [Inextor Vault](https://inexto.com/inexto-suite/inextor), [Beekast](https://my.beekast.com/), [Templafy OpenID Connect](https://app.templafy.com/), [PeterConnects receptionist](https://msteams.peterconnects.com/), [AlohaCloud](https://appfusions.alohacloud.com/auth), [Control Tower](https://bpm.tnxcorp.com/sso/microsoft), [Cocoom](https://start.cocoom.com/), [COINS Construction Cloud](https://sso.coinsconstructioncloud.com/#login/), [Medxnote MT](https://task.teamsmain.medx.im/authorization), [Reflekt](https://reflekt.konsolute.com/login), [Rever](https://app.reverscore.net/access), [MyCompanyArchive](https://login.mycompanyarchive.com/), [GReminders](https://app.greminders.com/o365-oauth), [Titanfile](../saas-apps/titanfile-tutorial.md), [Wootric](../saas-apps/wootric-tutorial.md), [SolarWinds Orion](https://support.solarwinds.com/SuccessCenter/s/orion-platform?language=en_US),  [OpenText Directory Services](../saas-apps/opentext-directory-services-tutorial.md), [Datasite](../saas-apps/datasite-tutorial.md), [BlogIn](../saas-apps/blogin-tutorial.md), [IntSights](../saas-apps/intsights-tutorial.md), [kpifire](../saas-apps/kpifire-tutorial.md), [Textline](../saas-apps/textline-tutorial.md), [Cloud Academy - SSO](../saas-apps/cloud-academy-sso-tutorial.md), [Community Spark](../saas-apps/community-spark-tutorial.md), [Chatwork](../saas-apps/chatwork-tutorial.md), [CloudSign](../saas-apps/cloudsign-tutorial.md), [C3M Cloud Control](../saas-apps/c3m-cloud-control-tutorial.md), [SmartHR](https://smarthr.jp/), [NumlyEngage™](../saas-apps/numlyengage-tutorial.md), [Michigan Data Hub Single Sign-On](../saas-apps/michigan-data-hub-single-sign-on-tutorial.md), [Egress](../saas-apps/egress-tutorial.md), [SendSafely](../saas-apps/sendsafely-tutorial.md), [Eletive](https://app.eletive.com/), [Right-Hand Cybersecurity ADI](https://right-hand.ai/), [Fyde Enterprise Authentication](https://enterprise.fyde.com/), [Verme](../saas-apps/verme-tutorial.md), [Lenses.io](../saas-apps/lensesio-tutorial.md), [Momenta](../saas-apps/momenta-tutorial.md), [Uprise](https://app.uprise.co/sign-in), [Q](https://q.moduleq.com/login), [CloudCords](../saas-apps/cloudcords-tutorial.md), [TellMe Bot](https://tellme365liteweb.azurewebsites.net/), [Inspire](https://app.inspiresoftware.com/), [Maverics Identity Orchestrator SAML Connector](https://www.strata.io/identity-fabric/), [Smartschool (School Management System)](https://smartschoolz.com/login), [Zepto - Intelligent timekeeping](https://user.zepto-ai.com/signin), [Studi.ly](https://studi.ly/), [Trackplan](http://www.trackplanfm.com/), [Skedda](../saas-apps/skedda-tutorial.md), [WhosOnLocation](../saas-apps/whos-on-location-tutorial.md), [Coggle](../saas-apps/coggle-tutorial.md), [Kemp LoadMaster](https://kemptechnologies.com/cloud-load-balancer/), [BrowserStack Single Sign-on](../saas-apps/browserstack-single-sign-on-tutorial.md)

La documentation de toutes ces applications est disponible ici https://aka.ms/AppsTutorial

Pour lister votre application dans la galerie d’applications Azure AD, veuillez lire les informations détaillées ici : https://aka.ms/AzureADAppRequest

---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---july-2020"></a>Nouveaux connecteurs de provisionnement dans la galerie d’applications Azure AD - Juillet 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Intégration tierce

Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour la nouvelle application nouvellement intégrée [LinkedIn Learning](../saas-apps/linkedin-learning-provisioning-tutorial.md).

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="view-role-assignments-across-all-scopes-and-ability-to-download-them-to-a-csv-file"></a>Afficher les attributions de rôle dans toutes les étendues et la possibilité de les télécharger dans un fichier CSV

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Rôles Azure AD  
**Fonctionnalité de produit :** Contrôle d’accès
 
Vous pouvez maintenant afficher les attributions de rôle sur toutes les étendues d’un rôle sous l’onglet « Rôles et administrateurs » dans le portail Azure AD. Vous pouvez également télécharger ces attributions de rôle pour chaque rôle dans un fichier CSV. Pour obtenir des conseils sur l’affichage et l’ajout d’attributions de rôles, consultez [Afficher et affecter des rôles d’administrateur dans Azure Active Directory](../roles/manage-roles-portal.md).
 
---

### <a name="azure-multi-factor-authentication-software-development-azure-mfa-sdk-deprecation"></a>Désapprobation du développement de logiciels Azure Multi-Factor Authentication (kit de développement logiciel (SDK) Azure MFA)

**Type :** Déprécié  
**Catégorie de service :** MFA  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Le développement de logiciels Azure Multi-Factor Authentication (kit de développement logiciel (SDK) Azure MFA) a atteint sa fin de vie le 14 novembre 2018, comme annoncé pour la première fois en novembre 2017. Microsoft va arrêter le service SDK en vigueur le 30 septembre 2020. Les appels passés au kit de développement logiciel (SDK) échouent.

Si votre organisation utilise le kit de développement logiciel (SDK) Azure MFA, vous devez migrer le 30 septembre 2020 :
- Kit de développement logiciel (SDK) Azure MFA pour MIM :  Si vous utilisez le kit de développement logiciel (SDK) avec MIM, vous devez migrer vers le serveur Azure MFA et activer Privileged Access Management (PAM) à la suite de ces [instructions](/microsoft-identity-manager/working-with-mfaserver-for-mim).   
- Kit de développement logiciel (SDK) Azure MFA pour les applications personnalisées : Envisagez d’intégrer votre application dans Azure AD et d’utiliser l’accès conditionnel pour appliquer l’authentification multifacteur. Pour commencer, consultez cette [page](../manage-apps/plan-an-application-integration.md). 

---

## <a name="june-2020"></a>Juin 2020 

### <a name="user-risk-condition-in-conditional-access-policy"></a>Condition de risque d’utilisateur dans la stratégie d’accès conditionnel

**Type :** Modification planifiée  
**Catégorie de service :** Accès conditionnel  
**Fonctionnalité de produit :** Protection et sécurité des identités
 

La prise en charge des risques d’utilisateur dans une stratégie d’accès conditionnel Azure AD vous permet de créer plusieurs stratégies basées sur les risques de l’utilisateur. Différents niveaux de risque d’utilisateur minimum peuvent être nécessaires pour différents utilisateurs et applications. En fonction du risque de l’utilisateur, vous pouvez créer des stratégies pour bloquer l’accès, exiger une authentification multifacteur, la modification sécurisée du mot de passe ou une redirection vers Microsoft Cloud App Security pour appliquer une stratégie de session, par exemple un audit supplémentaire.

La condition de risque d’utilisateur requiert Azure AD Premium P2 car elle utilise Azure Identity Protection, qui est une offre P2. Pour plus d’informations sur l’accès conditionnel, consultez [Documentation relative à l’Accès conditionnel Azure Active Directory](../conditional-access/index.yml).

---

### <a name="saml-sso-now-supports-apps-that-require-spnamequalifier-to-be-set-when-requested"></a>L’authentification unique SAML prend désormais en charge les applications qui requièrent la définition de SPNameQualifier à la demande

**Type :** Résolution  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO
 
Certaines applications SAML requièrent que SPNameQualifier soit retourné à la demande dans le sujet de l’assertion. Azure AD répond maintenant correctement lorsqu’un SPNameQualifier est demandé dans la stratégie NameID de la demande. Cette règle s’applique également à la connexion initiée par le fournisseur de service, et prochainement à la connexion initiée par le fournisseur d'identité.  Pour en savoir plus sur le protocole SAML dans Azure Active Directory, consultez [Protocole SAML d’authentification unique](../develop/single-sign-on-saml-protocol.md).

---

### <a name="azure-ad-b2b-collaboration-supports-inviting-msa-and-google-users-in-azure-government-tenants"></a>Azure AD B2B collaboration prend en charge l’invitation des utilisateurs MSA et Google dans les locataires Azure Government

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 

Les locataires Azure Government qui utilisent les fonctionnalités de collaboration B2B peuvent désormais inviter des utilisateurs disposant d’un compte Microsoft ou Google. Pour déterminer si votre locataire peut utiliser ces fonctionnalités, suivez les instructions fournies dans [Comment savoir si B2B Collaboration est disponible dans mon locataire Azure US Government ?](../external-identities/current-limitations.md#how-can-i-tell-if-b2b-collaboration-is-available-in-my-azure-us-government-tenant)

 
---
 
### <a name="user-object-in-ms-graph-v1-now-includes-externaluserstate-and-externaluserstatechangeddatetime-properties"></a>L’objet utilisateur dans MS Graph v1 comprend maintenant les propriétés externalUserState et externalUserStateChangedDateTime

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 

Les propriétés externalUserState et externalUserStateChangedDateTime peuvent être utilisées pour rechercher les invités B2B qui n’ont pas encore accepté leurs invitations, ainsi que l’automatisation des builds, notamment la suppression d’utilisateurs qui n’ont pas accepté leurs invitations après un certain nombre de jours. Ces propriétés sont désormais disponibles dans MS Graph v1. Pour obtenir des conseils sur l’utilisation de ces propriétés, reportez-vous à [Type de ressource utilisateur](/graph/api/resources/user).
 
---

### <a name="manage-authentication-sessions-in-azure-ad-conditional-access-is-now-generally-available"></a>La gestion des sessions d’authentification dans l’accès conditionnel Azure AD est désormais mise à la disposition générale

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Accès conditionnel  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Les fonctionnalités de gestion des sessions d’authentification vous permettent de configurer la fréquence à laquelle vos utilisateurs doivent fournir des informations d’identification de connexion ainsi que des informations d’identification après la fermeture et la réouverture des navigateurs, pour offrir plus de sécurité et de flexibilité à votre environnement.
 
Par ailleurs, la gestion des sessions d’authentification ne s'appliquait auparavant qu'à l'authentification à un facteur sur les appareils joints à Azure AD, joints à Azure AD Hybride et inscrits auprès d’Azure AD. Désormais, la gestion des sessions d’authentification s’applique également à MFA. Pour plus d’informations, consultez [Configurer la gestion de session d’authentification avec l’accès conditionnel](../conditional-access/howto-conditional-access-session-lifetime.md).

---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---june-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Juin 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
En juin 2020, nous avons ajouté les 29 nouvelles applications suivantes à notre galerie d’applications avec prise en charge de la fédération :

[Shopify Plus](../saas-apps/shopify-plus-tutorial.md), [Ekarda](../saas-apps/ekarda-tutorial.md), [MailGates](../saas-apps/mailgates-tutorial.md), [BullseyeTDP](../saas-apps/bullseyetdp-tutorial.md), [Raketa](../saas-apps/raketa-tutorial.md), [Segment](../saas-apps/segment-tutorial.md), [Ai Auditor](https://www.mindbridge.ai/products/ai-auditor/), [Pobuca Connect](https://app.pobu.ca/), [Proto.io](../saas-apps/proto.io-tutorial.md), [Gatekeeper](https://www.gatekeeperhq.com/), [Hub Planner](../saas-apps/hub-planner-tutorial.md), [Ansira-Partner Go-to-Market Toolbox](https://ansira.com/technology/channel-engagement), [IBM Digital Business Automation on Cloud](../saas-apps/ibm-digital-business-automation-on-cloud-tutorial.md), [Kisi Physical Security](../saas-apps/kisi-physical-security-tutorial.md), [ViewpointOne](https://team.viewpoint.com/), [IntelligenceBank](../saas-apps/intelligencebank-tutorial.md), [pymetrics](../saas-apps/pymetrics-tutorial.md), [Zero](https://www.teamzero.com/), [InStation](https://instation.invillia.com/), [edX for Business SAML 2.0 Integration](../saas-apps/edx-for-business-saml-integration-tutorial.md), [MOOC Office 365](https://mooc.office365-training.com/en/), [SmartKargo](../saas-apps/smartkargo-tutorial.md), [PKIsigning platform](https://platform.pkisigning.nl/), [SiteIntel](../saas-apps/siteintel-tutorial.md), [Field iD](../saas-apps/field-id-tutorial.md), [Curricula SAML](../saas-apps/curricula-saml-tutorial.md), [Perforce Helix Core - Helix Authentication Service](../saas-apps/perforce-helix-core-tutorial.md), [MyCompliance Cloud](https://cloud.metacompliance.com/), [Smallstep SSH](https://smallstep.com/sso-ssh/)  

La documentation de toutes ces applications est disponible ici https://aka.ms/AppsTutorial. Pour lister votre application dans la galerie d’applications Azure AD, veuillez lire les informations détaillées ici : https://aka.ms/AzureADAppRequest.

---

### <a name="api-connectors-for-external-identities-self-service-sign-up-are-now-in-public-preview"></a>Les connecteurs d’API pour l’inscription en libre-service des identités externes sont désormais en préversion publique

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 
Les connecteurs d’API d’identités externes vous permettent de tirer parti des API Web pour intégrer l’inscription en libre-service à des systèmes cloud externes. Cela signifie que vous pouvez désormais appeler des API Web en tant qu’étapes spécifiques dans un flux d’inscription pour déclencher des flux de travail personnalisés basés sur le cloud. Par exemple, vous pouvez utiliser des connecteurs d’API pour :

- Intégrer à des flux de travaux d’approbation personnalisés.
- Effectuer une vérification d’identité
- Valider des données d’entrée utilisateur
- Remplacer des attributs utilisateur
- Exécuter une logique métier personnalisée

Pour plus d’informations sur toutes les expériences possibles avec les connecteurs d’API, consultez [Utiliser des connecteurs d’API pour personnaliser et étendre l’inscription en libre-service](../external-identities/api-connectors-overview.md) ou [Personnaliser l’inscription en libre-service d’identités externes avec des intégrations d’API Web](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/customize-external-identities-self-service-sign-up-with-web-api/ba-p/1257364#.XvNz2fImuQg.linkedin).
 
---

### <a name="provision-on-demand-and-get-users-into-your-apps-in-seconds"></a>Provisionner à la demande et faire accéder les utilisateurs à vos applications en quelques secondes

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Le service de provisionnement Azure AD fonctionne actuellement de façon cyclique. Le service s’exécute toutes les 40 minutes. La [fonctionnalité de provisionnement à la demande](https://aka.ms/provisionondemand) vous permet de choisir un utilisateur et de le configurer en quelques secondes. Cette fonctionnalité vous permet de résoudre rapidement les problèmes de provisionnement, sans avoir à relancer le cycle de provisionnement. 
 
---

### <a name="new-permission-for-using-azure-ad-entitlement-management-in-graph"></a>Nouvelle autorisation pour utiliser la gestion des droits d’utilisation Azure AD dans Graph

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Autres  
**Fonctionnalité de produit :** Gestion des droits d’utilisation
 
Une nouvelle autorisation déléguée EntitlementManagement.Read.All est désormais disponible pour une utilisation avec l’API de gestion des droits dans Microsoft Graph version bêta. Pour en savoir plus sur les API disponibles, consultez [Utilisation de l’API de gestion des droits Azure AD](/graph/api/resources/entitlementmanagement-root?view=graph-rest-beta).

---

### <a name="identity-protection-apis-available-in-v10"></a>API de protection des données disponibles dans la version 1.0

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Identity Protection  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Les API riskyUsers et riskDetections Microsoft Graph sont désormais mises à la disposition générale. Elles sont maintenant disponibles au niveau du point de terminaison v1.0, et nous vous invitons à les utiliser en production. Pour plus d’informations, consultez la [documentation Microsoft Graph](/graph/api/resources/identityprotectionroot).
 
---

### <a name="sensitivity-labels-to-apply-policies-to-microsoft-365-groups-is-now-generally-available"></a>Les étiquettes de sensibilité permettant d’appliquer des stratégies à des groupes Microsoft 365 sont désormais mises à la disposition générale

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Gestion des groupes  
**Fonctionnalité de produit :** Collaboration
 

Vous pouvez maintenant créer des étiquettes de sensibilité et utiliser les paramètres d’étiquette pour appliquer des stratégies à des groupes Microsoft 365, notamment la confidentialité (publique ou privée) et la stratégie d’accès utilisateur externe. Vous pouvez créer une étiquette avec la stratégie de confidentialité privée et une stratégie d’accès utilisateur externe pour ne pas autoriser l’ajout d’utilisateurs invités. Lorsqu’un utilisateur applique cette étiquette à un groupe, le groupe est privé et aucun utilisateur invité n’est autorisé à y être ajouté. 

Les étiquettes de sensibilité sont importantes pour protéger les données critiques de votre entreprise et vous permettre de gérer les groupes à grande échelle, de manière conforme et sécurisée. Pour obtenir des conseils sur l’utilisation des étiquettes de sensibilité, consultez [Attribuer des étiquettes de sensibilité aux groupes Microsoft 365 dans Azure Active Directory (préversion)](../enterprise-users/groups-assign-sensitivity-labels.md).
 
---

### <a name="updates-to-support-for-microsoft-identity-manager-for-azure-ad-premium-customers"></a>Mises à jour de la prise en charge de Microsoft Identity Manager pour les clients Azure AD Premium

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Microsoft Identity Manager  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
La prise en charge Azure est désormais disponible pour les composants d’intégration Azure AD de Microsoft Identity Manager 2016, jusqu’à la fin de la prise en charge étendue de Microsoft Identity Manager 2016. Pour en savoir plus, consultez [Mise à jour du support pour les clients Azure AD Premium utilisant Microsoft Identity Manager](/microsoft-identity-manager/support-update-for-azure-active-directory-premium-customers).

---

### <a name="the-use-of-group-membership-conditions-in-sso-claims-configuration-is-increased"></a>L’utilisation des conditions d’appartenance de groupe dans la configuration des revendications SSO a été augmentée

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO
 
Auparavant, le nombre de groupes disponibles lors d’une modification conditionnelle des revendications en fonction de l’appartenance à un groupe dans une configuration d’application unique était limité à 10. L’utilisation des conditions d’appartenance de groupe dans la configuration des revendications SSO a maintenant été augmentée à un maximum de 50 groupes. Pour plus d’informations sur la configuration des revendications, reportez-vous à [Configuration des revendications SSO des applications d’entreprise](../develop/active-directory-saml-claims-customization.md#emitting-claims-based-on-conditions). 

---

### <a name="enabling-basic-formatting-on-the-sign-in-page-text-component-in-company-branding"></a>Activation de la mise en forme de base sur le composant Texte de la page de connexion dans la marque de société.

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur
 
La fonctionnalité Marque de société sur l’expérience de connexion Azure AD/Microsoft 365 a été mise à jour pour permettre au client d’ajouter des liens hypertexte et une mise en forme simple, y compris une police en gras, le soulignement et l’italique. Pour obtenir des conseils sur l’utilisation de cette fonctionnalité, voir [Personnaliser la page de connexion Azure Active Directory de votre organisation](./customize-branding.md).

---

### <a name="provisioning-performance-improvements"></a>Améliorations apportées aux performances de provisionnement

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Le service de provisionnement a été mis à jour pour réduire le temps nécessaire à l’exécution d’un [cycle incrémentiel](../app-provisioning/how-provisioning-works.md#incremental-cycles). Cela signifie que les utilisateurs et les groupes seront provisionnés dans leurs applications plus rapidement qu’auparavant. Toutes les tâches de provisionnement créées après le 10/06/2020 bénéficient automatiquement des améliorations de performances. Toutes les applications configurées pour un provisionnement avant le 10/06/2020 devront redémarrer une fois après cette date pour tirer parti des améliorations de performances. 

---

### <a name="announcing-the-deprecation-of-adal-and-ms-graph-parity"></a>Annonce de la dépréciation d’ADAL et de la parité MS Graph

**Type :** Déprécié  
**Catégorie de service :** N/A  
**Fonctionnalité de produit :** Gestion du cycle de vie des appareils

Maintenant que Microsoft Authentication Libraries (MSAL) est disponible, nous n’ajouterons plus de nouvelles fonctionnalités à Azure Active Directory Authentication Libraries (ADAL) et arrêterons les correctifs de sécurité le 30 juin 2022. Pour plus d’informations sur la migration vers MSAL, reportez-vous à [Migrer des applications vers la bibliothèque d’authentification Microsoft (MSAL)](../develop/msal-migration.md).

En outre, nous avons terminé la préparation de toutes les fonctionnalités Azure AD Graph disponibles via MS Graph. Ainsi, les API Graph Azure AD recevront uniquement des correctifs de bogues et de sécurité jusqu’au 30 juin 2022. Pour plus d’informations, consultez [Mettre à jour vos applications pour utiliser la bibliothèque d’authentification Microsoft et l’API Microsoft Graph](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/update-your-applications-to-use-microsoft-authentication-library/ba-p/1257363)
 
---
 
