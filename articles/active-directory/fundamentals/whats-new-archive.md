---
title: Archive des nouveautés d’Azure Active Directory | Microsoft Docs
description: Les notes de publication des nouveautés de la section Vue d’ensemble de ce jeu de contenus contient 6 mois d’activité. Au bout de 6 mois, les éléments sont supprimés de l’article principal et placés dans cet article d’archive.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: conceptual
ms.date: 11/30/2020
ms.author: ajburnle
ms.reviewer: dhanyahk
ms.custom: it-pro, seo-update-azuread-jan, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: a8e9841c71a6f5bc2e12887f41c62b43a17203a6
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107582"
---
# <a name="archive-for-whats-new-in-azure-active-directory"></a>Archive des nouveautés d’Azure Active Directory

L’article principal sur les [notes de publication relatives aux nouveautés d’Azure Active Directory](whats-new.md) contient les mises à jour des six derniers mois, tandis que cet article contient toutes les informations plus anciennes.

Les notes de publication relatives aux nouveautés d’Azure Active Directory fournissent des informations sur les éléments suivants :

- Versions les plus récentes
- Problèmes connus
- Résolution des bogues
- Fonctionnalités dépréciées
- Modifications planifiées

---
## <a name="may-2020"></a>Mai 2020

### <a name="retirement-of-properties-in-signins-riskyusers-and-riskdetections-apis"></a>Mise hors service des propriétés dans les API signIns, riskyUsers et riskDetections

**Type :** Modification planifiée  
**Catégorie de service :** Identity Protection  
**Fonctionnalité de produit :** Protection et sécurité des identités

Actuellement, les types énumérés sont utilisés pour représenter la propriété riskType à la fois dans l’API riskDetections et dans riskyUserHistoryItem (en préversion). Les types énumérés sont également utilisés pour la propriété riskEventTypes dans l’API signIns. À l’avenir, nous représenterons ces propriétés sous forme de chaînes. 

Les clients doivent effectuer la transition vers la propriété riskEventType dans la version bêta des API riskDetections et riskyUserHistoryItem, et pour la propriété riskEventTypes_v2 dans la version bêta de l’API signIns d’ici au 9 septembre 2020. À cette date, nous mettrons hors service les propriétés riskType et riskEventTypes actuelles. Pour plus d’informations, reportez-vous à [Modifications apportées aux propriétés d’événements à risque et aux API de protection d’identité sur Microsoft Graph](https://developer.microsoft.com/graph/blogs/changes-to-risk-event-properties-and-identity-protection-apis-on-microsoft-graph/).

--- 

### <a name="deprecation-of-riskeventtypes-property-in-signins-v10-api-on-microsoft-graph"></a>Dépréciation de la propriété riskEventTypes dans l’API signIns v1.0 sur Microsoft Graph

**Type :** Modification planifiée  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Protection et sécurité des identités

Les types énumérés basculeront vers les types chaîne s’ils représentent des propriétés d’événement à risque dans Microsoft Graph en septembre 2020. En plus de l’impact sur les API en préversion, cette modification affectera également l’API signIns en production.

Nous avons introduit une nouvelle propriété riskEventsTypes_v2 (chaîne) pour l’API signIns v1.0. Nous mettrons hors service la propriété riskEventTypes (enum) actuelle le 11 juin 2022, conformément à notre stratégie de dépréciation de Microsoft Graph. Les clients doivent passer à la propriété riskEventTypes_v2 dans l’API signIns v1.0 avant le 11 juin 2022. Pour plus d’informations, consultez [Dépréciation de la propriété riskEventTypes dans l’API signIns v1.0 sur Microsoft Graph](https://developer.microsoft.com/graph/blogs/deprecation-of-riskeventtypes-property-in-signins-v1-0-api-on-microsoft-graph//).

--- 

### <a name="upcoming-changes-to-mfa-email-notifications"></a>Modifications à venir concernant les notifications par e-mail MFA

**Type :** Modification planifiée  
**Catégorie de service :** MFA  
**Fonctionnalité de produit :** Protection et sécurité des identités
 

Nous apportons les modifications suivantes aux notifications par e-mail pour la MFA cloud :

Des notifications par e-mail vont être envoyées à partir de l’adresse suivante : azure-noreply@microsoft.com et msonlineservicesteam@microsoftonline.com. Nous mettons à jour le contenu des e-mails d’alerte de fraude afin de mieux indiquer les étapes à suivre pour débloquer les utilisations.

---

### <a name="new-self-service-sign-up-for-users-in-federated-domains-who-cant-access-microsoft-teams-because-they-arent-synced-to-azure-active-directory"></a>Nouvelle inscription en libre-service pour les utilisateurs des domaines fédérés qui ne peuvent pas accéder à Microsoft teams parce qu’ils ne sont pas synchronisés à Azure Active Directory.

**Type :** Modification planifiée  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur
 

Actuellement, les utilisateurs qui se trouvent sur des domaines fédérés dans Azure AD, mais qui ne sont pas synchronisés avec le locataire, ne peuvent pas accéder à Teams. À partir de la fin juin, cette nouvelle possibilité leur permettra de le faire par l’extension de la fonctionnalité d’inscription vérifiée par e-mail. Les utilisateurs qui peuvent se connecter à un fournisseur d’identité fédéré, mais qui ne possèdent pas encore d’objet utilisateur dans Azure ID, seront en mesure d’obtenir cet objet créé automatiquement, et d’être authentifié pour Teams. Leur objet utilisateur sera désigné en tant que « inscription en libre-service ». Il s’agit d’une extension de la capacité existante à effectuer une inscription automatique vérifiée par e-mail, à laquelle les utilisateurs des domaines gérés peuvent procéder et avec laquelle ils peuvent être contrôlés à l’aide du même indicateur. Cette modification s’effectuera lors du déploiement se déroulant dans les deux prochains mois. Retrouvez les mises à jour de la documentation [ici](../enterprise-users/directory-self-service-signup.md).
 
---

### <a name="upcoming-fix-the-oidc-discovery-document-for-the-azure-government-cloud-is-being-updated-to-reference-the-correct-graph-endpoints"></a>Correctif à venir : Le document de découverte OIDC pour le cloud Azure Government est mis à jour pour faire référence aux points de terminaison Graph appropriés.

**Type :** Modification planifiée  
**Catégorie de service :** Clouds souverains  
**Fonctionnalité de produit :** Authentification utilisateur
 
À partir de juin, le document de découverte OIDC, [Microsoft Identity Platform and OpenID Connect Protocol](../develop/v2-protocols-oidc.md) (Plateforme d’identités Microsoft et protocole OpenID Connect), relatif au point de terminaison [Cloud Azure Government](../develop/authentication-national-cloud.md) (login.microsoftonline.us), va commencer à retourner le point de terminaison [Graph du cloud national](/graph/deployments) approprié (https://graph.microsoft.us ou https://dod-graph.microsoft.us), en fonction du locataire fourni.  Il présente actuellement le champ « msgraph_host » incorrect du point de terminaison Graph (graph.microsoft.com).  

Ce correctif de bogue sera progressivement déployé sur une période de 2 mois environ.  

---

### <a name="azure-government-users-will-no-longer-be-able-to-sign-in-on-loginmicrosoftonlinecom"></a>Les utilisateurs d’Azure Government ne seront plus en mesure de se connecter à login.microsoftonline.com

**Type :** Planifier la modification  
**Catégorie de service :** Clouds souverains  
**Fonctionnalité de produit :** Authentification utilisateur
 
Le 1er juin 2018, l’adresse de l’autorité officielle Azure Active Directory (Azure AD) pour Azure Government a changé de https://login-us.microsoftonline.com en https://login.microsoftonline.us. Si vous possédez une application au sein d’un locataire Azure Government, vous devez mettre à jour votre application pour qu’elle connecte les utilisateurs sur le point de terminaison .us.

À partir du 5 mai, Azure AD commencera à appliquer le changement de point de terminaison en empêchant les utilisateurs d’Azure Government de se connecter à des applications hébergées dans des locataires Azure Government à l’aide du point de terminaison public (microsoftonline.com). Les applications impactées commenceront à afficher une erreur AADSTS900439-USGClientNotSupportedOnPublicEndpoint. 

Un déploiement progressif de cette modification va avoir lieu, dont la mise en application est prévue de s’achever sur l’ensemble des applications en juin 2020. Pour plus d’informations, consultez le [billet de blog Azure Government](https://devblogs.microsoft.com/azuregov/azure-government-aad-authority-endpoint-update/).

---

### <a name="saml-single-logout-request-now-sends-nameid-in-the-correct-format"></a>La requête SAML Single logout envoie désormais NameID au format correct

**Type :** Résolution  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur
 
Lorsqu’un utilisateur clique sur la déconnexion (par exemple, dans le portail MyApps), Azure AD envoie un message SAML Single Logout à chaque application qui est active dans la session utilisateur et pour laquelle une URL de déconnexion est configurée. Ces messages contiennent un NameID dans un format persistant.

Si le jeton de connexion SAML d’origine a utilisé un format différent pour NameID (par exemple, e-mail/UPN), l’application SAML ne peut pas mettre en corrélation le NameID du message de déconnexion avec une session existante (puisque les NameID utilisés dans les deux messages sont différents), ce qui a provoqué le rejet du message de déconnexion par l’application SAML, et le maintien de l’utilisateur connecté. Ce correctif met le message de déconnexion en adéquation avec le NameID configuré pour l’application.

---

### <a name="hybrid-identity-administrator-role-is-now-available-with-cloud-provisioning"></a>Le rôle d’administrateur d’identité hybride est désormais disponible avec le provisionnement cloud

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement cloud Azure AD  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Les administrateurs informatiques peuvent commencer à utiliser le nouveau rôle d’« administrateur hybride » en tant que rôle le moins privilégié pour configurer le provisionnement cloud Azure ADConnect. Avec ce nouveau rôle, vous n’avez plus besoin d’utiliser le rôle d’administrateur général pour installer et configurer le provisionnement cloud. [Plus d’informations](../roles/delegate-by-task.md#connect)
 
---

### <a name="new-federated-apps-available-in-azure-ad-application-gallery---may-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Mai 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
En mai 2020, nous avons ajouté les 36 applications suivantes à notre galerie d’applications avec prise en charge de la fédération :

[Moula](https://moula.com.au/pay/merchants), [Surveypal](https://www.surveypal.com/app), [Kbot365](https://www.konverso.ai/virtual-assistant-digital-workplace/), [TackleBox](http://www.tacklebox.app/), [Powell Teams](https://powell-software.com/en/powell-teams-en/), [Talentsoft Assistant](https://msteams.talent-soft.com/), [ASC Recording Insights](https://teams.asc-recording.app/product), [GO1](https://www.go1.com/), [B-Engaged](https://b-engaged.se/), [Competella Contact Center Workgroup](http://www.competella.com/), [Asite](http://www.asite.com/), [ImageSoft Identity](https://identity.imagesoftinc.com/), [My IBISWorld](https://identity.imagesoftinc.com/), [insuite](../saas-apps/insuite-tutorial.md), [Change Process Management](../saas-apps/change-process-management-tutorial.md), [Cyara CX Assurance Platform](../saas-apps/cyara-cx-assurance-platform-tutorial.md), [Smart Global Governance](../saas-apps/smart-global-governance-tutorial.md), [Prezi](../saas-apps/prezi-tutorial.md), [Mapbox](../saas-apps/mapbox-tutorial.md), [Datava Enterprise Service Platform](../saas-apps/datava-enterprise-service-platform-tutorial.md), [Whimsical](../saas-apps/whimsical-tutorial.md), [Trelica](../saas-apps/trelica-tutorial.md), [EasySSO for Confluence](../saas-apps/easysso-for-confluence-tutorial.md), [EasySSO for BitBucket](../saas-apps/easysso-for-bitbucket-tutorial.md), [EasySSO for Bamboo](../saas-apps/easysso-for-bamboo-tutorial.md), [Torii](../saas-apps/torii-tutorial.md), [Axiad Cloud](../saas-apps/axiad-cloud-tutorial.md), [Humanage](../saas-apps/humanage-tutorial.md), [ColorTokens ZTNA](../saas-apps/colortokens-ztna-tutorial.md), [CCH Tagetik](../saas-apps/cch-tagetik-tutorial.md), [ShareVault](../saas-apps/sharevault-tutorial.md), [Vyond](../saas-apps/vyond-tutorial.md), [TextExpander](../saas-apps/textexpander-tutorial.md), [Anyone Home CRM](../saas-apps/anyone-home-crm-tutorial.md), [askSpoke](../saas-apps/askspoke-tutorial.md), [ice Contact Center](../saas-apps/ice-contact-center-tutorial.md)

La documentation de toutes ces applications est disponible ici https://aka.ms/AppsTutorial.

Pour lister votre application dans la galerie d’applications Azure AD, veuillez lire les informations détaillées ici https://aka.ms/AzureADAppRequest.

---

### <a name="report-only-mode-for-conditional-access-is-now-generally-available"></a>Disponibilité générale du mode rapport seul pour l’accès conditionnel

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Accès conditionnel  
**Fonctionnalité de produit :** Protection et sécurité des identités

Le [mode rapport seul pour l’accès conditionnel Azure AD](../conditional-access/concept-conditional-access-report-only.md) vous permet d’évaluer le résultat d’une stratégie sans appliquer de contrôles d’accès. Vous pouvez tester les stratégies de rapport seul au sein de votre organisation afin de bien comprendre leur impact avant de les activer, ce qui rend leur déploiement à la fois plus sûr et plus facile. Ces derniers mois, nous avons constaté une adoption forte du mode rapport seul ; plus de 26 millions d’utilisateurs opèrent déjà dans le cadre d’une stratégie de rapport seul. Faisant suite à l’annonce d’aujourd’hui, les nouvelles stratégies d’accès conditionnel Azure AD seront créées par défaut en mode rapport seul. Cela signifie que vous pouvez surveiller l’impact de vos stratégies à partir du moment de leur création. Et ceux d’entre vous qui utilisent les API MS Graph peuvent également [gérer les stratégies de rapport seul par programmation](/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta). 

---

### <a name="self-service-sign-up-for-guest-users"></a>Inscription en libre-service pour les utilisateurs invités

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 
Avec les identités externes dans Azure AD, vous pouvez autoriser des personnes extérieures à votre organisation à accéder à vos applications et ressources tout en leur permettant de se connecter à l’aide de l’identité de leur choix. Lorsque vous partagez une application avec des utilisateurs externes, vous ne savez pas toujours à l’avance qui aura besoin d’y accéder. Avec l’[inscription en libre-service](../external-identities/self-service-sign-up-overview.md), vous pouvez autoriser les utilisateurs invités à s’inscrire et à obtenir un compte invité pour vos applications métier. Le flux d’inscription peut être créé et personnalisé pour prendre en charge les identités Azure AD et celles des réseaux sociaux. Vous pouvez également collecter des informations supplémentaires sur l’utilisateur lors de l’inscription.

---

 ### <a name="conditional-access-insights-and-reporting-workbook-is-generally-available"></a>Disponibilité générale du classeur Insights et rapports sur l’accès conditionnel

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Accès conditionnel  
**Fonctionnalité de produit :** Protection et sécurité des identités

Le [classeur Insights et rapports](../conditional-access/howto-conditional-access-insights-reporting.md) donne aux administrateurs une vue récapitulative de l’accès conditionnel Azure AD dans leur locataire. Avec la possibilité de sélectionner une stratégie individuelle, les administrateurs peuvent mieux comprendre ce que fait chaque stratégie et surveiller les modifications en temps réel. Le classeur diffuse les données stockées dans Azure Monitor, que vous pouvez configurer en quelques minutes en [suivant ces instructions](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). Pour améliorer la visibilité du tableau de bord, nous avons déplacé celui-ci vers le nouvel onglet Insights et rapports dans le menu de l’accès conditionnel Azure AD.

---

### <a name="policy-details-blade-for-conditional-access-is-in-public-preview"></a>Le panneau Détails de la stratégie pour l’accès conditionnel est en préversion publique

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Accès conditionnel  
**Fonctionnalité de produit :** Protection et sécurité des identités

Le nouveau [panneau Détails de la stratégie](../conditional-access/troubleshoot-conditional-access.md) affiche les affectations, conditions et contrôles qui ont été respectés lors de l’évaluation de la stratégie d’accès conditionnel. Vous pouvez accéder au panneau en sélectionnant une ligne sous les onglets Accès conditionnel ou Rapport seul des Détails de connexion.

---

### <a name="new-query-capabilities-for-directory-objects-in-microsoft-graph-are-in-public-preview"></a>De nouvelles fonctionnalités de requête pour les objets d’annuaire dans Microsoft Graph sont en préversion publique

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** MS Graph **Fonctionnalité de produit :** Expérience de développement

De nouvelles fonctionnalités sont introduites pour les API des objets d’annuaire de Microsoft Graph, ce qui permet d’effectuer des opérations de comptage, de recherche, de filtrage et de tri. Les développeurs pourront ainsi interroger rapidement nos objets d’annuaire sans recourir à des solutions de contournement, telles que les filtrage et tri en mémoire. Apprenez-en davantage dans ce [billet de blog](https://aka.ms/CountFilterMSGraphAAD).

Nous sommes actuellement en préversion publique et aimerions avoir votre avis sur ce point. Merci de nous transmettre vos commentaires en répondant à ce [court sondage](https://aka.ms/MsGraphAADSurveyDocs).

---

### <a name="configure-saml-based-single-sign-on-using-microsoft-graph-api-beta"></a>Configurer l’authentification unique SAML à l’aide de l’API Microsoft Graph (Bêta)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO
 
La prise en charge de la création et de la configuration d’une application depuis la galerie Azure AD, à l’aide des API Microsoft Graph dans la version Bêta, est désormais disponible. Si vous avez besoin de configurer l’authentification unique SAML pour plusieurs instances d’une application, gagnez du temps en utilisant les API Microsoft Graph pour [automatiser la configuration de l’authentification unique SAML](/graph/application-saml-sso-configure-api).
 
---

### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---may-2020"></a>Nouveaux connecteurs de provisionnement dans la galerie d’applications Azure AD - Mai 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Intégration tierce
 
Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

* [8x8](../saas-apps/8x8-provisioning-tutorial.md)
* [Juno Journey](../saas-apps/juno-journey-provisioning-tutorial.md)
* [MediusFlow](../saas-apps/mediusflow-provisioning-tutorial.md)
* [New Relic by Organization](../saas-apps/new-relic-by-organization-provisioning-tutorial.md)
* [Oracle Cloud Infrastructure Console](../saas-apps/oracle-cloud-infratstructure-console-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="saml-token-encryption-is-generally-available"></a>Disponibilité générale du chiffrement de jeton SAML

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO
 
Le [chiffrement de jeton SAML](../manage-apps/howto-saml-token-encryption.md) permet aux applications d’être configurées pour recevoir des assertions SAML chiffrées. La fonctionnalité est désormais en disponibilité générale dans tous les clouds.
 
---

### <a name="group-name-claims-in-application-tokens-is-generally-available"></a>Disponibilité générale des revendications de nom de groupe dans les jetons d’application

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO
 
Les revendications de groupe émises dans un jeton peuvent maintenant être limitées aux seuls groupes attribués à l’application.  Cela est particulièrement important lorsque les utilisateurs sont membres d’un grand nombre de groupes, et qu’il existe un risque de dépassement des limites de taille des jetons. Avec cette nouvelle capacité en place, la possibilité d’[ajouter des noms de groupe à des jetons](../hybrid/how-to-connect-fed-group-claims.md) est mise en disponibilité générale.
 
---

### <a name="workday-writeback-now-supports-setting-work-phone-number-attributes"></a>Workday Writeback prend désormais en charge la définition d’attributs de numéro de téléphone professionnel

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Nous avons amélioré l’application de provisionnement Workday Writeback pour qu’elle prenne dorénavant en charge la réécriture des attributs de numéros de téléphone mobile et fixe professionnels. Outre l’e-mail et le nom d’utilisateur, vous pouvez à présent configurer l’application de provisionnement Workday Writeback pour transmettre des valeurs de numéro de téléphone d’Azure AD à Workday. Pour plus d’informations sur la configuration de la réécriture des numéros de téléphone, reportez-vous au tutoriel de l’application [Workday Writeback](../saas-apps/workday-writeback-tutorial.md). 

---

### <a name="publisher-verification-preview"></a>Vérification de l’éditeur (préversion)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Autres  
**Fonctionnalité de produit :** Expérience de développement
 
La vérification de l'éditeur (préversion) permet aux administrateurs et aux utilisateurs finaux de s'assurer de l'authenticité des développeurs d'applications qui s'intègrent à la Plateforme d'identités Microsoft. Pour plus d’informations, reportez-vous à [Vérification de l’éditeur (préversion)](../develop/publisher-verification-overview.md).
 
---

### <a name="authorization-code-flow-for-single-page-apps"></a>Flux du code d’autorisation pour les applications à page unique

**Type :** Fonctionnalité modifiée **Catégorie de service :** Authentification **Fonctionnalité de produit :** Expérience de développement

Du fait [des restrictions sur les cookies tiers dans les navigateurs modernes, telles que ITP de Safari](../develop/reference-third-party-cookies-spas.md), les application à page unique (SPA) doivent utiliser le flux du code d’autorisation, plutôt que le flux implicite, pour maintenir l’authentification unique. MSAL.js v 2.x prend désormais en charge le flux du code d’autorisation. Des mises à jour correspondantes étant présentes sur le portail Azure, vous pouvez actualiser votre SPA pour qu’il soit de type « spa » et utilise le flux du code d’authentification. Pour obtenir des instructions, reportez-vous à [Démarrage rapide : Connecter des utilisateurs et obtenir un jeton d’accès dans une application à page unique JavaScript à l’aide du flux de code d’authentification](../develop/quickstart-v2-javascript-auth-code.md).

---

### <a name="improved-filtering-for-devices-is-in-public-preview"></a>Le filtrage amélioré des appareils est en préversion publique

**Type :** Fonctionnalité modifiée   
**Catégorie de service :** Gestion des appareils **Fonctionnalité de produit :** Gestion du cycle de vie des appareils
 
Auparavant, les seuls filtres que vous pouviez utiliser étaient « Enabled » (Activé) et « Activity date » (Date d’activité). À présent, vous pouvez [filtrer votre liste d’appareils sur plus de propriétés](../devices/device-management-azure-portal.md#device-list-filtering-preview), y compris le type de système d’exploitation, le type de jointure, la conformité, etc. Ces ajouts doivent simplifier la localisation d’un appareil particulier.

---

### <a name="the-new-app-registrations-experience-for-azure-ad-b2c-is-now-generally-available"></a>La nouvelle expérience d’inscriptions d’applications pour Azure AD B2C est désormais en disponibilité générale

**Type :** Fonctionnalité modifiée   
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
La nouvelle expérience d’inscriptions d’applications pour Azure AD B2C est désormais en disponibilité générale. 

Auparavant, vous deviez gérer vos applications B2C destinées aux consommateurs séparément du reste de vos applications, en utilisant l’expérience « Applications » héritée. Cela signifiait différentes expériences de création d’applications à différents emplacements dans Azure.

La nouvelle expérience affiche toutes les inscriptions d’applications Azure AD B2C et les inscriptions d’applications Azure AD dans un même emplacement, et permet de les gérer de manière cohérente. Que vous deviez gérer une application accessible aux clients, ou une application qui accède à Microsoft Graph pour manager par programmation des ressources Azure AD B2C, vous n’avez besoin d’apprendre qu’une seule façon de faire les choses.

Vous pouvez afficher la nouvelle expérience en accédant au service Azure AD B2C et en sélectionnant le panneau d’inscriptions des applications. L’expérience est également accessible à partir du service Azure Active Directory.

L’expérience d’inscriptions d’applications Azure AD B2C repose sur l’[expérience d’inscription d’applications générale](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/) pour les locataires Azure AD, mais elle est personnalisée pour Azure AD B2C. L’expérience « Applications » héritée sera dépréciée à l’avenir.

Pour plus d’informations, consultez [La nouvelle expérience d’inscription d’applications pour Azure AD B2C](../../active-directory-b2c/app-registrations-training-guide.md).

---
## <a name="april-2020"></a>Avril 2020

### <a name="combined-security-info-registration-experience-is-now-generally-available"></a>L’expérience d’inscription combinées des informations de sécurité est désormais généralement disponible

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Authentifications (connexions)

**Fonctionnalité de produit :** Protection et sécurité des identités

L’expérience d’inscription combinée pour l’authentification multifacteur (MFR, Multi-Factor Authentication) et la réinitialisation de mot de passe en libre-service (SSPR, Self-Service Password Reset) est désormais généralement disponible. Cette nouvelle expérience d’inscription permet aux utilisateurs de s’inscrire pour la MFA et la SSPR en un seul processus pas à pas. Lorsque vous déployez la nouvelle expérience pour votre organisation, les utilisateurs peuvent s’inscrire en moins de temps et avec moins de tracas. Consultez le billet de blog [ici](https://bit.ly/3etiRyQ).

---

### <a name="continuous-access-evaluation"></a>Évaluation continue de l’accès

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Authentifications (connexions)

**Fonctionnalité de produit :** Protection et sécurité des identités

L’évaluation continue de l’accès est une nouvelle fonctionnalité de sécurité qui permet une mise en œuvre en quasi-temps réel de stratégies sur des parties se fiant à l’utilisation des jetons d’accès Azure AD lorsque certains événements tels que la suppression d’un compte d’utilisateur se produisent dans Azure AD. Nous déployons cette fonctionnalité en priorité pour les clients Teams et Outlook. Pour plus d’informations, consultez notre [blog](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/moving-towards-real-time-policy-and-security-enforcement/ba-p/1276933) et la [documentation](../conditional-access/concept-continuous-access-evaluation.md).

---

### <a name="sms-sign-in-firstline-workers-can-sign-in-to-azure-ad-backed-applications-with-their-phone-number-and-no-password"></a>Connexion SMS : Les employés de terrain peuvent se connecter à des applications adossées à Azure AD sans introduire de mot de passe, mais simplement leur numéro de téléphone

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Authentifications (connexions)

**Fonctionnalité de produit :** Authentification utilisateur

Office lance une série d’applications métier avant tout mobiles qui s’adressent à des organisations non traditionnelles et à des employés de grandes organisations pour qui l’e-mail n’est pas la méthode de communication principale. Ces applications ciblent les employés de terrain, les travailleurs sans bureau, les agents de terrain ou les effectifs de vente au détail qui n’ont pas forcément d’adresse e-mail professionnelle, ou d’accès à un ordinateur ou à d’autres ressources informatiques. Ce projet permet à ces employés de se connecter à des applications d’entreprise en entrant un numéro de téléphone et un code. Pour en savoir, consultez notre [documentation administrateur](../authentication/howto-authentication-sms-signin.md) et notre [documentation utilisateur final](../user-help/sms-sign-in-explainer.md).

---

### <a name="invite-internal-users-to-use-b2b-collaboration"></a>Inviter les utilisateurs internes à adopter B2B Collaboration

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** B2B

**Fonctionnalité de produit :**

Nous développons la fonctionnalité d’invitation B2B permettant d’inviter des comptes internes existants à utiliser désormais des informations d’identification pour la collaboration B2B. Pour ce faire, vous pouvez transmettre l’objet utilisateur à l’API Invite en plus de paramètres standard tels que l’adresse e-mail invitée. L’ID d’objet, l’UPN, l’appartenance à un groupe, l’affectation d’applications et autres informations des utilisateurs restent intacts, mais ceux-ci utilisent désormais la solution B2B pour s’authentifier avec les informations d’identification de leur locataire de base plutôt qu’avec les informations d’identification internes qu’ils utilisaient avant l’invitation. Pour plus de détails, consultez la [documentation](../external-identities/invite-internal-users.md).

---

### <a name="report-only-mode-for-conditional-access-is-now-generally-available"></a>Disponibilité générale du mode rapport seul pour l’accès conditionnel

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Accès conditionnel

**Fonctionnalité de produit :** Protection et sécurité des identités

Le [mode rapport seul pour l’accès conditionnel Azure AD](../conditional-access/concept-conditional-access-report-only.md) vous permet d’évaluer le résultat d’une stratégie sans appliquer de contrôles d’accès. Vous pouvez tester les stratégies de rapport seul au sein de votre organisation afin de bien comprendre leur impact avant de les activer, ce qui rend leur déploiement à la fois plus sûr et plus facile. Ces derniers mois, nous avons constaté une adoption forte du mode rapport seul, avec plus de 26 millions d’utilisateurs opérant déjà dans le cadre d’une stratégie de rapport seul. À la suite de cette annonce, les nouvelles stratégies d’accès conditionnel Azure AD seront créées par défaut en mode rapport seul. Cela signifie que vous pouvez surveiller l’impact de vos stratégies à partir du moment de leur création. Et ceux d’entre vous qui utilisent les API MS Graph peuvent également [gérer les stratégies de rapport seul par programmation](/graph/api/resources/conditionalaccesspolicy?view=graph-rest-beta). 

---

### <a name="conditional-access-insights-and-reporting-workbook-is-generally-available"></a>Disponibilité générale du classeur Insights et rapports de l’accès conditionnel

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Accès conditionnel

**Fonctionnalité de produit :** Protection et sécurité des identités

Le [classeur Insights et rapports](../conditional-access/howto-conditional-access-insights-reporting.md) de l’accès conditionnel donne aux administrateurs une vue récapitulative de l’accès conditionnel Azure AD dans leur locataire. Avec la possibilité de sélectionner une stratégie individuelle, les administrateurs peuvent mieux comprendre ce que fait chaque stratégie et surveiller les modifications en temps réel. Le classeur diffuse les données stockées dans Azure Monitor, que vous pouvez configurer en quelques minutes en [suivant ces instructions](../reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md). Pour améliorer la visibilité du tableau de bord, nous avons déplacé celui-ci vers le nouvel onglet Insights et rapports dans le menu de l’accès conditionnel Azure AD.

---

### <a name="policy-details-blade-for-conditional-access-is-in-public-preview"></a>Le panneau Détails de la stratégie pour l’accès conditionnel est en préversion publique

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Accès conditionnel

**Fonctionnalité de produit :** Protection et sécurité des identités

Le nouveau [panneau Détails de la stratégie](../conditional-access/troubleshoot-conditional-access.md) affiche les affectations, conditions et contrôles qui ont été respectés lors de l’évaluation de la stratégie d’accès conditionnel. Vous pouvez accéder au panneau en sélectionnant une ligne sous les onglets **Accès conditionnel** ou **Rapport seul** des Détails de connexion.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---april-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Avril 2020

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Applications d’entreprise

**Fonctionnalité de produit :** Intégration tierce

En avril 2020, nous avons ajouté à la galerie d’applications les 31 nouvelles applications suivantes qui prennent en charge la fédération des identités : 

[SincroPool Apps](https://www.sincropool.com/), [SmartDB](https://hibiki.dreamarts.co.jp/smartdb/trial/), [Float](../saas-apps/float-tutorial.md), [LMS365](https://lms.365.systems/), [IWT Procurement Suite](../saas-apps/iwt-procurement-suite-tutorial.md), [Lunni](https://lunni.fi/), [EasySSO for Jira](../saas-apps/easysso-for-jira-tutorial.md), [Virtual Training Academy](https://vta.c3p.ca/app/en/openid?authenticate_with=microsoft), [Meraki Dashboard](../saas-apps/meraki-dashboard-tutorial.md), [Microsoft 365 Mover](https://app.mover.io/login), [Speaker Engage](https://speakerengage.com/login.php), [Honestly](../saas-apps/honestly-tutorial.md), [Ally](../saas-apps/ally-tutorial.md), [DutyFlow](https://app.dutyflow.nl/), [AlertMedia](../saas-apps/alertmedia-tutorial.md), [gr8 People](../saas-apps/gr8-people-tutorial.md), [Pendo](../saas-apps/pendo-tutorial.md), [HighGround](../saas-apps/highground-tutorial.md), [Harmony](../saas-apps/harmony-tutorial.md), [Timetabling Solutions](../saas-apps/timetabling-solutions-tutorial.md), [SynchroNet CLICK](../saas-apps/synchronet-click-tutorial.md), [empower](https://www.made-in-office.com/en/), [Fortes Change Cloud](../saas-apps/fortes-change-cloud-tutorial.md), [Litmus](../saas-apps/litmus-tutorial.md), [GroupTalk](https://recorder.grouptalk.com/), [Frontify](../saas-apps/frontify-tutorial.md), [MongoDB Cloud](../saas-apps/mongodb-cloud-tutorial.md), [TickitLMS Learn](../saas-apps/tickitlms-learn-tutorial.md), [COCO](https://hexaware.com/partnerships-and-alliances/digital-transformation-using-microsoft-azure/), [Nitro Productivity Suite](../saas-apps/nitro-productivity-suite-tutorial.md), [Trend Micro Web Security(TMWS)](https://review.docs.microsoft.com/azure/active-directory/saas-apps/trend-micro-tutorial)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="microsoft-graph-delta-query-support-for-oauth2permissiongrant-available-for-public-preview"></a>Disponibilité de la prise en charge de requête delta Microsoft Graph pour oAuth2PermissionGrant en préversion publique

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** MS Graph

**Fonctionnalité de produit :** Expérience de développement

La requête delta pour oAuth2PermissionGrant est disponible en préversion publique. Vous pouvez maintenant suivre des modifications sans devoir continuellement interroger Microsoft Graph. [En savoir plus.](/graph/api/oAuth2PermissionGrant-delta?tabs=http&view=graph-rest-beta)

---

### <a name="microsoft-graph-delta-query-support-for-organizational-contact-generally-available"></a>Disponibilité générale de la prise en charge de requête delta Microsoft Graph pour les contacts professionnels

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** MS Graph

**Fonctionnalité de produit :** Expérience de développement

La requête delta pour les contacts professionnels est en disponibilité générale. Vous pouvez à présent suivre les modifications dans les applications de production sans devoir interroger continuellement Microsoft Graph. Remplacez tout code interrogeant continuellement les données orgContact par une requête delta afin d’améliorer considérablement les performances. [En savoir plus.](/graph/api/orgcontact-delta?tabs=http)

---

### <a name="microsoft-graph-delta-query-support-for-application-generally-available"></a>Disponibilité générale de la prise en charge de requête delta Microsoft Graph pour les applications

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** MS Graph

**Fonctionnalité de produit :** Expérience de développement

La requête delta pour les applications est généralement disponible. Vous pouvez maintenant suivre les modifications dans les applications de production sans devoir interroger continuellement Microsoft Graph. Remplacez tout code interrogeant continuellement les données d’application par une requête delta afin d’améliorer considérablement les performances. [En savoir plus.](/graph/api/application-delta)

---

### <a name="microsoft-graph-delta-query-support-for-administrative-units-available-for-public-preview"></a>Disponibilité de la prise en charge de requête delta Microsoft Graph pour les unités administratives en préversion publique

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** MS Graph

**Fonctionnalité de produit :** La requête delta de l’expérience développeur pour les unités administratives est disponible en préversion publique. Vous pouvez maintenant suivre des modifications sans devoir continuellement interroger Microsoft Graph. [En savoir plus.](/graph/api/administrativeunit-delta?tabs=http&view=graph-rest-beta)

---

### <a name="manage-authentication-phone-numbers-and-more-in-new-microsoft-graph-beta-apis"></a>Gestion des numéros de téléphone d’authentification et bien plus encore dans les nouvelles API bêta de Microsoft Graph

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** MS Graph

**Fonctionnalité de produit :** Expérience de développement

Ces API constituent un outil essentiel pour la gestion des méthodes d’authentification de vos utilisateurs. Vous pouvez désormais préinscrire et gérer par programme les authentificateurs utilisés pour l’authentification multifacteur et la réinitialisation de mot de passe en libre-service. Il s’agit de l’une des fonctionnalités les plus demandées dans les espaces Azure AD MFA, SSPR et Microsoft Graph. Les nouvelles API que nous avons publiées dans cette vague vous donnent la possibilité d’effectuer les opérations suivantes :

- Lire, ajouter, mettre à jour et supprimer les téléphones d’authentification d’un utilisateur
- Réinitialiser le mot de passe d’un utilisateur
- Activer et désactiver la connexion par SMS

Pour plus d’informations, consultez [Présentation de l’API Méthodes d’authentification Azure AD](/graph/api/resources/authenticationmethods-overview?view=graph-rest-beta).

---

### <a name="administrative-units-public-preview"></a>Préversion publique des unités administratives

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Rôles Azure AD

**Fonctionnalité de produit :** Contrôle d’accès

Les unités administratives vous permettent d’accorder des autorisations d’administrateur limitées à un service, une région ou un autre segment de votre organisation que vous définissez. Vous pouvez utiliser des unités administratives pour déléguer des autorisations à des administrateurs régionaux ou définir une stratégie à un niveau précis. Par exemple, un administrateur de comptes d’utilisateur peut mettre à jour des informations de profil, réinitialiser des mots de passe et attribuer des licences aux utilisateurs uniquement dans son unité administrative.

Les unités administratives permettent à un administrateur central d’effectuer les opérations suivantes :

- Créer une unité administrative pour la gestion décentralisée des ressources
- Attribuer un rôle disposant d’autorisations administratives uniquement sur les utilisateurs Azure AD d’une unité administrative
- Remplir les unités administratives d’utilisateurs et de groupes en fonction des besoins

Pour plus d’informations, consultez [Gestion des unités administratives dans Azure Active Directory (préversion)](../roles/administrative-units.md)

---

### <a name="printer-administrator-and-printer-technician-built-in-roles"></a>Rôles intégrés Administrateur d’imprimantes et Technicien en charge des imprimantes

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Rôles Azure AD

**Fonctionnalité de produit :** Contrôle d’accès

**Administrateur d’imprimantes** : Les utilisateurs ayant ce rôle peuvent inscrire des imprimantes et gérer tous les aspects liés aux configurations de celles-ci dans la solution Impression universelle de Microsoft, dont les paramètres du connecteur d’impression universelle. Ils peuvent consentir à toutes les demandes d’autorisation d’impression déléguée. Les administrateurs d’imprimantes ont également accès aux rapports d’impression. 

**Technicien en charge des imprimantes** : Les utilisateurs ayant ce rôle peuvent inscrire des imprimantes et gérer leur statut dans la solution Impression universelle de Microsoft. Ils peuvent également lire toutes les informations du connecteur. L’une des tâches clés qu’un technicien en charge des imprimantes ne peut pas accomplir est la définition d’autorisations utilisateur sur les imprimantes et le partage d’imprimantes. [En savoir plus.](../roles/permissions-reference.md#printer-administrator)

---

### <a name="hybrid-identity-admin-built-in-role"></a>Rôle intégré d’administrateur d’identité hybride

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Rôles Azure AD

**Fonctionnalité de produit :** Contrôle d’accès

Les utilisateurs de ce rôle peuvent activer, configurer et gérer les services et paramètres liés à l’activation de l’identité hybride dans Azure AD. Ce rôle permet de configurer Azure AD sur l’une des trois méthodes d’authentification prises en charge (synchronisation de hachage du mot de passe, authentification directe ou fédération -AD FS ou fournisseur de fédération tiers-), et de déployer une infrastructure locale associée pour les activer. L’infrastructure locale comprend des agents d’approvisionnement et d’authentification directe. Ce rôle donne la possibilité d’activer l’authentification unique transparente (S-SSO) pour permettre une authentification transparente sur des appareils autres que Windows 10 ou des ordinateurs autres que Windows Server 2016. En outre, ce rôle donne la possibilité de voir les journaux de connexion et d’accéder à des informations en lien avec l’intégrité et l’analytique à des fins de surveillance et de dépannage. [En savoir plus.](../roles/permissions-reference.md#hybrid-identity-administrator)

---

### <a name="network-administrator-built-in-role"></a>Rôle intégré d’administrateur réseau

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** Rôles Azure AD

**Fonctionnalité de produit :** Contrôle d’accès

Les utilisateurs titulaires de ce rôle peuvent passer en revue, depuis l’emplacement où ils se trouvent, les recommandations de Microsoft relatives à l’architecture du périmètre réseau, basées sur la télémétrie du réseau. Les performances réseau pour Microsoft 365 s’appuient sur une architecture de périmètre de réseau client entreprise soigneuse, qui est généralement propre à la localisation de l’utilisateur. Ce rôle permet de modifier les emplacements utilisateur découverts et la configuration des paramètres réseau pour ces emplacements afin de faciliter les mesures de télémétrie et les recommandations de conception améliorées. [En savoir plus.](../roles/permissions-reference.md#network-administrator)

---

### <a name="bulk-activity-and-downloads-in-the-azure-ad-admin-portal-experience"></a>Activité en bloc et téléchargements dans l’expérience du portail d’administration Azure AD

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** User Management

**Fonctionnalité de produit :** Répertoire

Vous pouvez désormais conduire des activités en bloc sur des utilisateurs et des groupes dans Azure AD en téléchargeant un fichier CSV via le portail d’administration d’Azure AD. Vous pouvez créer, supprimer et inviter des utilisateurs. Vous pouvez ajouter et supprimer des membres dans un groupe.

Vous pouvez également télécharger des listes de ressources Azure AD à partir du portail d’administration d’Azure AD. Vous pouvez télécharger les listes des utilisateurs et des groupes figurant dans l’annuaire, ainsi que la liste des membres d’un groupe particulier.

Pour plus d’informations, consultez les articles suivants :

- [Créer des utilisateurs](../enterprise-users/users-bulk-add.md) ou [Inviter des utilisateurs](../external-identities/tutorial-bulk-invite.md)
- [Supprimer des utilisateurs](../enterprise-users/users-bulk-delete.md) ou [Restaurer des utilisateurs supprimés](../enterprise-users/users-bulk-restore.md)
- [Télécharger la liste des utilisateurs](../enterprise-users/users-bulk-download.md) ou [Télécharger la liste des groupes](../enterprise-users/groups-bulk-download.md)
- [Ajouter (importer) des membres](../enterprise-users/groups-bulk-import-members.md), [Supprimer des membres](../enterprise-users/groups-bulk-remove-members.md) ou [Télécharger la liste des membres](../enterprise-users/groups-bulk-download-members.md) d’un groupe

---

### <a name="my-staff-delegated-user-management"></a>Gestion des utilisateurs délégués de Mon personnel

**Type :** Nouvelle fonctionnalité

**Catégorie de service :** User Management

**Fonctionnalité de produit :**

Le portail Mon personnel permet aux responsables sur le terrain, tels que des responsables de magasin, de s’assurer que les membres de leur personnel sont en mesure d’accéder à leurs comptes Azure AD. Au lieu de s’appuyer sur un support technique central, les organisations peuvent déléguer à un responsable sur le terrain des tâches courantes telles que la réinitialisation de mot de passe ou la modification de numéros de téléphone. Grâce au portail Mon personnel, un utilisateur qui ne parvient plus à accéder à son compte peut récupérer l’accès en quelques clics, sans intervention du support technique ou du personnel informatique. Pour plus d’informations, consultez [Gestion des utilisateurs avec Mon personnel (préversion)](../roles/my-staff-configure.md) et [Déléguer la gestion des utilisateurs avec Mon personnel (préversion)](../user-help/my-staff-team-manager.md).

---

### <a name="an-upgraded-end-user-experience-in-access-reviews"></a>Expérience d’utilisateur final mise à niveau dans les révisions d’accès

**Type :** Fonctionnalité modifiée

**Catégorie de service :** Révisions d’accès

**Fonctionnalité de produit :** Gouvernance des identités

Nous avons mis à jour l’expérience du réviseur pour les révisions d’accès Azure AD dans le portail Mes applications. Depuis la fin du mois d’avril, les réviseurs connectés à l’interface réviseur des révisions d’accès Azure AD voient s’afficher une bannière qui leur permet d’essayer l’expérience mise à jour dans le portail Mon accès. Notez que l’expérience des révisions d’accès mise à jour offre les mêmes fonctionnalités que l’expérience antérieure, mais avec une interface utilisateur améliorée s’ajoutant aux nouvelles fonctionnalités pour permettre aux utilisateurs d’être productifs. [Vous pouvez en savoir plus sur l’expérience mise à jour ici](../governance/perform-access-review.md). Cette préversion publique sera disponible jusqu’à la fin du mois de juillet 2020. À partir de cette date, les réviseurs qui n’auront pas adopté l’expérience en préversion seront automatiquement dirigés vers le portail Mon accès pour effectuer des révisions d’accès. Si vous souhaitez que vos réviseurs basculent définitivement vers l’expérience en préversion dans le portail Mon accès dès maintenant, [veuillez introduire une demande ici](https://forms.microsoft.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR5dv-S62099HtxdeKIcgO-NUOFJaRDFDWUpHRk8zQ1BWVU1MMTcyQ1FFUi4u).

---

### <a name="workday-inbound-user-provisioning-and-writeback-apps-now-support-the-latest-versions-of-workday-web-services-api"></a>Les applications d’approvisionnement des utilisateurs entrants et d’écriture différée de Workday prennent désormais en charge les dernières versions de l’API Workday Web Services (WWS).

**Type :** Fonctionnalité modifiée

**Catégorie de service :** Provisionnement d’applications

**Fonctionnalité de produit :** 

Sur la base des commentaires des clients, nous avons mis à jour les applications d’approvisionnement des utilisateurs entrants et d’écriture différée de Workday dans la galerie d’applications d’entreprise pour prendre en charge les dernières versions de l’API Workday Web Services (WWS). Suite à cette modification, les clients peuvent spécifier la version de l’API WWS qu’ils souhaitent utiliser dans la chaîne de connexion. Les clients peuvent ainsi récupérer davantage d’attributs de RH disponibles dans les versions de Workday. L’application d’écriture différée de Workday utilise désormais le service web recommandé Change_Work_Contact_Info de Workday pour surmonter les limitations de Maintain_Contact_Info.

Si aucune version n’est spécifiée dans la chaîne de connexion, par défaut, les applications d’approvisionnement entrant de Workday continueront d’utiliser WWS v 21.1. Pour basculer vers les dernières API Workday pour l’approvisionnement d’utilisateurs entrants, les clients doivent mettre à jour la chaîne de connexion comme indiqué [dans le didacticiel](../saas-apps/workday-inbound-tutorial.md#which-workday-apis-does-the-solution-use-to-query-and-update-workday-worker-profiles), et mettre à jour les XPATH utilisés pour les attributs Workday, comme décrit dans le [Guide de référence sur les attributs Workday](../app-provisioning/workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30). 

Pour utiliser la nouvelle API pour l’écriture différée, aucune modification n’est requise dans l’application d’approvisionnement d’écriture différée de Workday. Du côté de Workday, assurez-vous que le compte d’utilisateur de système d’intégration (ISU) de Workday dispose des autorisations nécessaires pour appeler le processus métier Change_Work_Contact comme indiqué dans la section [Configurer des autorisations de stratégie de sécurité de processus métier](../saas-apps/workday-inbound-tutorial.md#configuring-business-process-security-policy-permissions) du didacticiel. 

Nous avons mis à jour notre [didacticiel](../saas-apps/workday-inbound-tutorial.md) pour refléter la prise en charge de la nouvelle version de l’API.

---

### <a name="users-with-default-access-role-are-now-in-scope-for-provisioning"></a>Utilisateurs titulaires du rôle d’accès par défaut désormais dans l’étendue de l’approvisionnement

**Type :** Fonctionnalité modifiée

**Catégorie de service :** Provisionnement d’applications

**Fonctionnalité de produit :** Gestion du cycle de vie des identités

Historiquement, les utilisateurs pourvus du rôle d’accès par défaut étaient hors de portée du provisionnement. Certains clients souhaitaient que les utilisateurs disposant de ce rôle figurent dans l’étendue du provisionnement. Depuis le 16 avril 2020, toutes les nouvelles configurations d’approvisionnement permettent d’approvisionner les utilisateurs titulaires du rôle d’accès par défaut. Progressivement, nous allons modifier le comportement des configurations d’approvisionnement existantes pour prendre en charge l’approvisionnement d’utilisateurs avec ce rôle. [En savoir plus.](../app-provisioning/application-provisioning-config-problem-no-users-provisioned.md)

---

### <a name="updated-provisioning-ui"></a>IU d’approvisionnement mise à jour

**Type :** Fonctionnalité modifiée

**Catégorie de service :** Provisionnement d’applications

**Fonctionnalité de produit :** Gestion du cycle de vie des identités

Nous avons actualisé notre expérience d’approvisionnement pour créer une vue de gestion plus ciblée. Lorsque vous accédez au panneau d’approvisionnement d’une application d’entreprise déjà configurée, vous pouvez facilement surveiller la progression de l’approvisionnement et gérer des actions telles que le démarrage, l’arrêt et le redémarrage de l’approvisionnement. [En savoir plus.](../app-provisioning/configure-automatic-user-provisioning-portal.md)

---

### <a name="dynamic-group-rule-validation-is-now-available-for-public-preview"></a>Validation de règle de groupe dynamique désormais disponible en préversion publique

**Type :** Fonctionnalité modifiée

**Catégorie de service :** Gestion des groupes

**Fonctionnalité de produit :** Collaboration

Azure Active Directory (Azure AD) offre désormais la possibilité de valider les règles de groupe dynamique. Sous l’onglet **Valider les règles**, vous pouvez valider votre règle dynamique par rapport à des exemples de membres du groupe pour vérifier que la règle fonctionne comme prévu. Lors de la création ou de la mise à jour des règles de groupe dynamiques, les administrateurs veulent savoir si un utilisateur ou un appareil sera membre du groupe. Cela permet d’évaluer si un utilisateur ou un appareil répond aux critères de la règle et facilite la résolution des problèmes quand l’appartenance n’est pas attendue.

Pour plus d’informations, consultez [Valider une règle d’appartenance à un groupe dynamique (préversion)](../enterprise-users/groups-dynamic-rule-validation.md).

---

### <a name="identity-secure-score---security-defaults-and-mfa-improvement-action-updates"></a>Score d’identité sécurisée – Mises à jour des actions d’amélioration des paramètres de sécurité par défaut et de l’authentification multifacteur

**Type :** Fonctionnalité modifiée

**Catégorie de service :** N/A

**Fonctionnalité de produit :** Protection et sécurité des identités

**Prise en charge des paramètres de sécurité par défaut pour les actions d’amélioration d’Azure AD :** Le service Degré de sécurisation Microsoft mettra à jour les actions d’amélioration pour prendre en charge les [paramètres de sécurité par défaut dans Azure AD](./concept-fundamentals-security-defaults.md), facilitant la protection de votre organisation à l’aide de paramètres de sécurité préconfigurés pour contrer des attaques courantes. Cela aura une incidence sur les actions d’amélioration suivantes :

- Vérifier que tous les utilisateurs peuvent utiliser une authentification multifacteur pour un accès sécurisé
- Exiger l'authentification multifacteur pour les rôles administratifs
- Activer une stratégie pour bloquer l’authentification héritée
 
**Mises à jour de l’action d’amélioration de l’authentification multifacteur :** Afin de refléter la nécessité pour les entreprises de garantir une sécurité optimale tout en appliquant des stratégies adaptées à leur activité, le service Degré de sécurisation Microsoft a supprimé trois actions d’amélioration centrées sur l’authentification multifacteur, et en a ajouté deux.

Actions d’amélioration supprimées :

- Inscrire tous les utilisateurs pour l’authentification multifacteur
- Exiger MFA pour tous les utilisateurs
- Exiger l'authentification multifacteur pour les rôles privilégiés Azure AD

Actions d’amélioration ajoutées :

- Vérifier que tous les utilisateurs peuvent utiliser une authentification multifacteur pour un accès sécurisé
- Exiger l'authentification multifacteur pour les rôles administratifs

Ces nouvelles actions d’amélioration nécessitent l’inscription de vos utilisateurs ou administrateurs pour l’authentification multifacteur dans votre annuaire, ainsi que la mise en place d’un ensemble approprié de stratégies adaptées aux besoins de votre organisation. L'objectif principal est de disposer d'une certaine flexibilité tout en permettant à l'ensemble de vos utilisateurs et administrateurs de s'authentifier à l'aide de plusieurs facteurs ou d'invites de vérification d'identité basée sur les risques. Cela peut prendre la forme de l’instauration de plusieurs stratégies qui appliquent des décisions étendues ou de la définition de paramètres de sécurité par défaut (à compter du 16 mars) qui permettent à Microsoft de décider quand exiger l’authentification multifacteur des utilisateurs. [En savoir plus sur les nouveautés du service Degré de sécurisation Microsoft](/microsoft-365/security/mtp/microsoft-secure-score#whats-new).

---

## <a name="march-2020"></a>Mars 2020

### <a name="unmanaged-azure-active-directory-accounts-in-b2b-update-for-march--2021"></a>Comptes Azure Active Directory non gérés dans la mise à jour B2B pour mars 2021

**Type :** Modification planifiée  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 
**À partir du 31 mars 2021**, Microsoft ne prendra plus en charge l’échange d’invitations en créant des locataires et des comptes Azure Active Directory (Azure AD) non managés pour les scénarios de collaboration B2B. Dans cette optique, nous vous encourageons à choisir l’[authentification au moyen d’un code secret à usage unique envoyé par e-mail](../external-identities/one-time-passcode.md).

---

### <a name="users-with-the-default-access-role-will-be-in-scope-for-provisioning"></a>Les utilisateurs dotés du rôle d’accès par défaut seront concernés par le provisionnement.

**Type :** Modification planifiée  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Historiquement, les utilisateurs pourvus du rôle d’accès par défaut étaient hors de portée du provisionnement. Certains clients souhaitaient que les utilisateurs disposant de ce rôle figurent dans l’étendue du provisionnement. Nous travaillons au déploiement d’une modification, pour que toute nouvelle configuration de provisionnement autorise les utilisateurs dotés du rôle d’accès par défaut à être provisionnés. Progressivement, nous allons modifier le comportement des configurations de provisionnement existantes pour prendre en charge le provisionnement des utilisateurs avec ce rôle. Aucune action du client n’est nécessaire. Nous allons poster une mise à jour dans notre [documentation](../app-provisioning/application-provisioning-config-problem-no-users-provisioned.md) dès que cette modification sera en place.

---

### <a name="azure-ad-b2b-collaboration-will-be-available-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet-tenants"></a>La solution Azure AD B2B Collaboration sera disponible dans Microsoft Azure, gérée par les locataires 21Vianet (Azure Chine 21Vianet).

**Type :** Modification planifiée  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 
Les fonctionnalités d’Azure AD B2B Collaboration seront rendues disponibles dans Microsoft Azure, gérées par les locataires 21Vianet (Azure Chine 21Vianet), ce qui permettra aux utilisateurs d’un locataire Azure Chine 21Vianet de collaborer de façon transparente avec les utilisateurs d’autres locataires Azure Chine 21Vianet. [En savoir plus sur Azure AD B2B Collaboration](/azure/active-directory/b2b/).

---
 
### <a name="azure-ad-b2b-collaboration-invitation-email-redesign"></a>Redéfinition de l’e-mail d’invitation à Azure AD B2B Collaboration

**Type :** Modification planifiée  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 
Les [e-mails](../external-identities/invitation-email-elements.md) qui sont envoyés par le service d’invitation d’Azure AD B2B Collaboration pour convier les utilisateurs à rejoindre l’annuaire seront remaniés afin de rendre plus claires les informations d’invitation et les étapes de l’utilisateur à suivre.

---

### <a name="homerealmdiscovery-policy-changes-will-appear-in-the-audit-logs"></a>Les modifications apportées à la stratégie HomeRealmDiscovery s’afficheront dans les journaux d’audit

**Type :** Résolution  
**Catégorie de service :** Audit  
**Fonctionnalité de produit :** Monitoring et création de rapports
 
Nous avons résolu un bogue dans lequel les modifications apportées à la stratégie [HomeRealmDiscovery](../manage-apps/configure-authentication-for-federated-users-portal.md) n’étaient pas incluses dans les journaux d’audit. Vous pouvez désormais voir quand et comment la stratégie a été modifiée, et par qui. 

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---march-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Mars 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
En mars 2020, nous avons ajouté à la galerie d’applications les 51 applications suivantes qui prennent en charge la fédération : 

[Cisco AnyConnect](../saas-apps/cisco-anyconnect.md), [Zoho One China](../saas-apps/zoho-one-china-tutorial.md), [PlusPlus](https://test.plusplus.app/auth/login/azuread-outlook/), [o](../saas-apps/profitco-saml-app-tutorial.md), [iPoint Service Provider](../saas-apps/ipoint-service-provider-tutorial.md), [contexxt.ai SPHERE](https://contexxt-sphere.com/login), [Wisdom By Invictus](../saas-apps/wisdom-by-invictus-tutorial.md), [Flare Digital Signage](https://spark-dev.pixelnebula.com/login), [s](../saas-apps/logzio-cloud-observability-for-engineers-tutorial.md), [SpectrumU](../saas-apps/spectrumu-tutorial.md), [BizzContact](https://bizzcontact.app/), [Elqano SSO](../saas-apps/elqano-sso-tutorial.md), [MarketSignShare](http://www.signshare.com/), [CrossKnowledge Learning Suite](../saas-apps/crossknowledge-learning-suite-tutorial.md), [Netvision Compas](../saas-apps/netvision-compas-tutorial.md), [FCM HUB](../saas-apps/fcm-hub-tutorial.md), [RIB A/S Byggeweb Mobile](https://apps.apple.com/us/app/docia/id529058757), [GoLinks](../saas-apps/golinks-tutorial.md), [Datadog](../saas-apps/datadog-tutorial.md), [Zscaler B2B User Portal](../saas-apps/zscaler-b2b-user-portal-tutorial.md), [LIFT](../saas-apps/lift-tutorial.md), [Planview Enterprise One](../saas-apps/planview-enterprise-one-tutorial.md), [WatchTeams](https://www.devfinition.com/), [Aster](https://demo.asterapp.io/login), [Skills Workflow](../saas-apps/skills-workflow-tutorial.md), [Node Insight](https://admin.nodeinsight.com/AADLogin.aspx), [IP Platform](../saas-apps/ip-platform-tutorial.md), [InVision](../saas-apps/invision-tutorial.md), [Pipedrive](../saas-apps/pipedrive-tutorial.md), [Showcase Workshop](https://app.showcaseworkshop.com/), [Greenlight Integration Platform](../saas-apps/greenlight-integration-platform-tutorial.md), [Greenlight Compliant Access Management](../saas-apps/greenlight-compliant-access-management-tutorial.md), [Grok Learning](../saas-apps/grok-learning-tutorial.md), [Miradore Online](https://login.online.miradore.com/), [Khoros Care](../saas-apps/khoros-care-tutorial.md), [AskYourTeam](../saas-apps/askyourteam-tutorial.md), [TruNarrative](../saas-apps/trunarrative-tutorial.md), [Smartwaiver](https://www.smartwaiver.com/m/user/sw_login.php?wms_login), [Bizagi Studio for Digital Process Automation](../saas-apps/bizagi-studio-for-digital-process-automation-tutorial.md), [insuiteX](https://www.insuite.jp/), [sybo](https://www.systexsoftware.com.tw/), [Britive](../saas-apps/britive-tutorial.md), [WhosOffice](../saas-apps/whosoffice-tutorial.md), [E-days](../saas-apps/e-days-tutorial.md), [Kollective SDN](https://portal.kollective.app/login), [Witivio](https://app.witivio.com/), [Playvox](https://my.playvox.com/login), [Korn Ferry 360](../saas-apps/korn-ferry-360-tutorial.md), [Campus Café](../saas-apps/campus-cafe-tutorial.md), [Catchpoint](../saas-apps/catchpoint-tutorial.md), [Code42](../saas-apps/code42-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="azure-ad-b2b-collaboration-available-in-azure-government-tenants"></a>Azure AD B2B Collaboration disponible dans les locataires Azure Government

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** B2B/B2C
 
Les fonctionnalités d’Azure AD B2B Collaboration sont désormais disponibles entre certains locataires Azure Government.  Pour déterminer si votre locataire est en mesure d’utiliser ces fonctionnalités, suivez les instructions fournies dans [Comment savoir si B2B Collaboration est disponible dans mon locataire Azure US Government ?](../external-identities/current-limitations.md#how-can-i-tell-if-b2b-collaboration-is-available-in-my-azure-us-government-tenant).

---

### <a name="azure-monitor-integration-for-azure-logs-is-now-available-in-azure-government"></a>L’intégration d’Azure Monitor pour les journaux Azure est à présent disponible dans Azure Government

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Monitoring et création de rapports
 
L’intégration d’Azure Monitor aux journaux Azure AD est à présent disponible dans Azure Government. Vous pouvez router les journaux Azure AD (journaux d’audit et de connexion) sur un compte de stockage, Event Hub et Log Analytics. Consultez la [documentation détaillée](../reports-monitoring/concept-activity-logs-azure-monitor.md) ainsi que les [plans de déploiement pour la création de rapports et la supervision](../reports-monitoring/plan-monitoring-and-reporting.md) des scénarios Azure AD.

---

### <a name="identity-protection-refresh-in-azure-government"></a>Actualisation d’Identity Protection dans Azure Government

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Identity Protection  
**Fonctionnalité de produit :** Protection et sécurité des identités

Nous sommes ravis d’annoncer que nous venons de déployer l’expérience actualisée d’[Azure AD Identity Protection](../identity-protection/overview-identity-protection.md)  dans le portail [Microsoft Azure Government](https://portal.azure.us/). Pour plus d’informations, consultez notre [billet sur le blog d’annonces](https://techcommunity.microsoft.com/t5/public-sector-blog/identity-protection-refresh-in-microsoft-azure-government/ba-p/1223667).

---

### <a name="disaster-recovery-download-and-store-your-provisioning-configuration"></a>Reprise d’activité après sinistre : Téléchargement et stockage de votre configuration de provisionnement

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités
 
Le service de provisionnement Azure AD fournit un ensemble complet de fonctionnalités de configuration. Les clients doivent être en mesure d’enregistrer leur configuration afin de pouvoir s’y référer ultérieurement, ou de revenir à une version correcte connue. Nous avons ajouté la possibilité de télécharger votre configuration de provisionnement sous la forme d’un fichier JSON que vous chargez ensuite lorsque vous en avez besoin. [Plus d’informations](../app-provisioning/export-import-provisioning-configuration.md)

---
 
### <a name="sspr-self-service-password-reset-now-requires-two-gates-for-admins-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet"></a>SSPR (réinitialisation de mot de passe en libre-service) exige désormais deux portes pour les administrateurs dans Microsoft Azure géré par 21Vianet (Azure Chine 21Vianet) 

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Réinitialisation du mot de passe en libre-service  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Auparavant, dans Microsoft Azure géré par 21Vianet (Azure Chine 21Vianet), les administrateurs qui utilisaient la réinitialisation de mot de passe en libre-service (SSPR) pour réinitialiser leurs mots de passe n’avaient besoin que d’une seule « porte » (défi) pour prouver leur identité. En général, dans les clouds publics et les autres clouds nationaux, les administrateurs doivent utiliser deux portes pour prouver leur identité lors de l’utilisation de SSPR. Pour autant, comme nous ne prenions en charge ni les appels téléphoniques ni les SMS dans Azure Chine 21Vianet, nous avions autorisé la réinitialisation de mot de passe à une porte par les administrateurs.

Nous sommes en cours de création d’une parité des fonctionnalités SSPR entre Azure Chine 21Vianet et le cloud public. Désormais, les administrateurs devront utiliser deux portes lors de l’utilisation de SSPR. Les SMS, appels téléphoniques ainsi que les codes et notifications de l’application Authenticator seront pris en charge. [Plus d’informations](../authentication/concept-sspr-policy.md#administrator-reset-policy-differences)

---

### <a name="password-length-is-limited-to-256-characters"></a>La longueur du mot de passe est limitée à 256 caractères

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur
 
Pour garantir la fiabilité du service Azure AD, la longueur des mots de passe utilisateur est désormais limitée à 256 caractères. Les utilisateurs dotés de mots de passe plus longs seront invités à modifier leur mot de passe lors d’une prochaine connexion, soit en contactant leur administrateur, soit en utilisant la fonctionnalité de réinitialisation de mot de passe en libre-service.

Cette modification a été activée le 13 mars 2020, à 10 H 00 PST (18:00 UTC), et l’erreur est AADSTS 50052, InvalidPasswordExceedsMaxLength. Pour plus d’informations, consultez l’[avis de changement cassant](../develop/reference-breaking-changes.md#user-passwords-will-be-restricted-to-256-characters).

---

### <a name="azure-ad-sign-in-logs-are-now-available-for-all-free-tenants-through-the-azure-portal"></a>Les journaux de connexion Azure AD sont désormais disponibles pour tous les locataires gratuits via le portail Azure

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Signalement  
**Fonctionnalité de produit :** Monitoring et création de rapports
 
À partir de maintenant, les clients qui disposent de locataires gratuits peuvent accéder aux [journaux de connexion Azure AD depuis le portail Azure](../reports-monitoring/concept-sign-ins.md), sur une durée de 7 jours. Auparavant, les journaux de connexion étaient uniquement disponibles pour les clients détenteurs de licences Azure Active Directory Premium. Avec cette modification, tous les locataires peuvent désormais accéder à ces journaux par l’intermédiaire du portail.

> [!NOTE]
> Les clients ont toujours besoin d’une licence Premium (Azure Active Directory Premium P1 ou P2) pour accéder aux journaux de connexion par le biais de l’API Microsoft Graph et d’Azure Monitor.

---

### <a name="deprecation-of-directory-wide-groups-option-from-groups-general-settings-on-azure-portal"></a>Dépréciation de l’option Groupes à l’échelle de l’annuaire dans les Paramètres généraux des groupes sur le portail Azure

**Type :** Déprécié  
**Catégorie de service :** Gestion des groupes  
**Fonctionnalité de produit :** Collaboration

Pour fournir aux clients un moyen plus souple de créer des groupes au niveau de l’annuaire, tout en répondant mieux à leurs besoins, nous avons remplacé l’option **Groupes à l’échelle de l’annuaire** dans **Groupes** > **Paramètres généraux** du portail Azure par un lien vers la [documentation du groupe dynamique](../enterprise-users/groups-dynamic-membership.md). Nous avons amélioré notre documentation pour inclure des instructions supplémentaires permettant aux administrateurs de créer des groupes de tous les utilisateurs qui incluent ou excluent des utilisateurs invités.

---

## <a name="february-2020"></a>Février 2020

### <a name="upcoming-changes-to-custom-controls"></a>Modifications à venir des contrôles personnalisés

**Type :** Modification planifiée  
**Catégorie de service :** MFA  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Nous envisageons de remplacer la préversion actuelle des contrôles personnalisés par une approche qui permet aux fonctionnalités d’authentification fournies par le partenaire de fonctionner de manière transparente avec les expériences de l’utilisateur final et de l’administrateur Azure Active Directory. Aujourd’hui, les solutions MFA partenaires font face aux limitations suivantes : elles fonctionnent uniquement après l’entrée d’un mot de passe. Elles ne servent pas d’authentification MFA pour l’authentification avancée dans d’autres scénarios clés, et ne s’intègrent pas aux fonctions de gestion des informations d’identification de l’utilisateur final ou d’administration. La nouvelle implémentation autorisera les facteurs d’authentification fournis par les partenaires à fonctionner avec les facteurs intégrés pour les principaux scénarios, notamment l’inscription, l’utilisation, les revendications MFA, l’authentification avancée, la création de rapports et la journalisation. 

Les contrôles personnalisés continueront d’être pris en charge dans la préversion en même temps que la nouvelle conception, jusqu’à ce que la disponibilité générale soit atteinte. À ce stade, nous donnerons à nos clients le temps de migrer vers la nouvelle conception. En raison des limitations de l’approche actuelle, nous n’intégrerons pas de nouveaux fournisseurs tant que la nouvelle conception ne sera pas disponible. Travaillant en étroite collaboration avec les clients et les fournisseurs, nous communiquerons la chronologie des opérations à mesure que nous approcherons de la finalisation. [Plus d’informations](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/upcoming-changes-to-custom-controls/ba-p/1144696#)

---

### <a name="identity-secure-score---mfa-improvement-action-updates"></a>Score d'identité sécurisée - Mises à jour des actions d'amélioration de l'authentification multifacteur (MFA)

**Type :** Modification planifiée  
**Catégorie de service :** MFA  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Afin de refléter la nécessité pour les entreprises de garantir une sécurité optimale tout en appliquant des stratégies adaptées à leur activité, le service Degré de sécurisation Microsoft supprime trois actions d'amélioration centrées sur l'authentification multifacteur (MFA) et en ajoute deux.

Les actions d'amélioration suivantes vont être supprimées :

- Inscrire tous les utilisateurs à l'authentification multifacteur
- Exiger MFA pour tous les utilisateurs
- Exiger l'authentification multifacteur pour les rôles privilégiés Azure AD

Les actions d'amélioration suivantes vont être ajoutées :

- Vérifier que tous les utilisateurs peuvent utiliser l'authentification multifacteur pour garantir un accès sécurisé
- Exiger l'authentification multifacteur pour les rôles administratifs

Ces nouvelles actions d'amélioration nécessiteront l'inscription de vos utilisateurs ou administrateurs à l'authentification multifacteur dans votre répertoire, ainsi que l'établissement d'un ensemble de stratégies adaptées aux besoins de votre organisation. L'objectif principal est de disposer d'une certaine flexibilité tout en permettant à l'ensemble de vos utilisateurs et administrateurs de s'authentifier à l'aide de plusieurs facteurs ou d'invites de vérification d'identité basée sur les risques. Cela peut prendre la forme d'une configuration par défaut de la sécurité qui permet à Microsoft de décider quand imposer l'authentification multifacteur aux utilisateurs, ou de l'établissement de différentes stratégies qui appliquent des décisions à large portée. Dans le cadre de ces mises à jour des actions d'amélioration, les stratégies de protection de base ne seront plus incluses dans les calculs de scoring. [En savoir plus sur les nouveautés du service Degré de sécurisation Microsoft](/microsoft-365/security/mtp/microsoft-secure-score-whats-coming).

---

### <a name="azure-ad-domain-services-sku-selection"></a>Sélection des références SKU Azure AD Domain Services

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Services de domaine Azure AD  
**Fonctionnalité de produit :** Services de domaine Azure AD
 
Certains clients d’Azure AD Domain Services souhaitaient bénéficier d’une flexibilité accrue en matière de sélection des niveaux de performances de leurs instances. Le 1er février 2020, nous sommes passés d'un modèle dynamique (dans lequel Azure AD détermine le niveau de performance et le niveau tarifaire en fonction du nombre d'objets) à un modèle d'auto-sélection. Les clients peuvent désormais choisir un niveau de performance adapté à leur environnement. Cette modification nous permet également de proposer de nouveaux scénarios, comme les forêts de ressources, et des fonctionnalités Premium, comme les sauvegardes quotidiennes. Le nombre d’objets est désormais illimité pour toutes les références SKU, mais nous continuerons à proposer des suggestions de nombre d’objets pour chaque niveau.

**Aucune action immédiate du client n'est nécessaire.** Pour les clients existants, le niveau dynamique qui était utilisé le 1er février 2020 détermine le nouveau niveau par défaut. Ce changement n'a aucun impact sur les prix ou sur les performances. À l'avenir, les clients Azure AD DS devront évaluer les exigences de performance en fonction de l'évolution de la taille de leur répertoire et des caractéristiques de leur charge de travail. Le basculement entre les niveaux de service s'effectuera toujours sans temps d'arrêt, et nous cesserons de déplacer automatiquement les clients vers de nouveaux niveaux en fonction de la croissance de leur répertoire. En outre, il n'y aura aucune augmentation de prix, et la nouvelle tarification sera conforme à notre modèle de facturation actuel. Pour plus d'informations, consultez la [documentation relative aux références SKU Azure AD DS](../../active-directory-domain-services/administration-concepts.md#azure-ad-ds-skus) et la [page consacrée à la tarification d'Azure AD Domain Services](https://azure.microsoft.com/pricing/details/active-directory-ds/).

---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery---february-2020"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App - Février 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
En février 2020, nous avons ajouté à la galerie d’applications les 31 nouvelles applications suivantes qui prennent en charge la fédération : 

[IamIP Patent Platform](../saas-apps/iamip-patent-platform-tutorial.md), [Experience Cloud](../saas-apps/experience-cloud-tutorial.md), [NS1 SSO For Azure](../saas-apps/ns1-sso-azure-tutorial.md), [Barracuda Email Security Service](https://ess.barracudanetworks.com/sso/azure), [ABa Reporting](https://myaba.co.uk/client-access/signin/auth/msad), [In Case of Crisis - Online Portal](../saas-apps/in-case-of-crisis-online-portal-tutorial.md), [BIC Cloud Design](../saas-apps/bic-cloud-design-tutorial.md), [Beekeeper Azure AD Data Connector](../saas-apps/beekeeper-azure-ad-data-connector-tutorial.md), [Korn Ferry Assessments](https://www.kornferry.com/solutions/kf-digital/kf-assess), [Verkada Command](../saas-apps/verkada-command-tutorial.md), [Splashtop](../saas-apps/splashtop-tutorial.md), [Syxsense](../saas-apps/syxsense-tutorial.md), [EAB Navigate](../saas-apps/eab-navigate-tutorial.md), [New Relic (Limited Release)](../saas-apps/new-relic-limited-release-tutorial.md), [Thulium](https://admin.thulium.com/login/instance), [Ticket Manager](../saas-apps/ticketmanager-tutorial.md), [Template Chooser for Teams](https://links.officeatwork.com/templatechooser-download-teams), [Beesy](https://www.beesy.me/index.php/site/login), [Health Support System](../saas-apps/health-support-system-tutorial.md), [MURAL](https://app.mural.co/signup), [Hive](../saas-apps/hive-tutorial.md), [LavaDo](https://appsource.microsoft.com/product/web-apps/lavaloon.lavado_standard?tab=Overview), [Wakelet](https://wakelet.com/login), [Firmex VDR](../saas-apps/firmex-vdr-tutorial.md), [ThingLink for Teachers and Schools](https://www.thinglink.com/), [Coda](../saas-apps/coda-tutorial.md), [NearpodApp](https://nearpod.com/signup/?oc=Microsoft&utm_campaign=Microsoft&utm_medium=site&utm_source=product), [WEDO](../saas-apps/wedo-tutorial.md), [InvitePeople](https://invitepeople.com/login), [Reprints Desk - Article Galaxy](../saas-apps/reprints-desk-article-galaxy-tutorial.md), [TeamViewer](../saas-apps/teamviewer-tutorial.md)

 
Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---
 
### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---february-2020"></a>Nouveaux connecteurs d'approvisionnement dans la Galerie d'applications Azure AD - Février 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Mixpanel](../saas-apps/mixpanel-provisioning-tutorial.md)
- [TeamViewer](../saas-apps/teamviewer-provisioning-tutorial.md)
- [Azure Databricks](/azure/databricks/administration-guide/users-groups/scim/aad)
- [PureCloud by Genesys](../saas-apps/purecloud-by-genesys-provisioning-tutorial.md)
- [Zapier](../saas-apps/zapier-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---
 
### <a name="azure-ad-support-for-fido2-security-keys-in-hybrid-environments"></a>Prise en charge d'Azure AD pour les clés de sécurité FIDO2 dans les environnements hybrides

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur
 
Nous avons le plaisir de vous annoncer le lancement de la préversion publique de la prise en charge d'Azure AD pour les clés de sécurité FIDO2 dans les environnements hybrides. Les utilisateurs peuvent désormais utiliser des clés de sécurité FIDO2 pour se connecter à leurs appareils Windows 10 joints à Azure AD Hybride et se connecter de manière transparente à leurs ressources locales et cloud. La prise en charge des environnements hybrides était la fonctionnalité la plus demandée par nos clients sans mot de passe depuis le lancement de la préversion publique de la prise en charge de FIDO2 dans les appareils joints à Azure AD. L'authentification sans mot de passe à l'aide de technologies avancées telles que la biométrie et le chiffrement par clé publique/privée offre commodité et facilité d'utilisation tout en étant sécurisée. Grâce à cette préversion publique, vous pouvez désormais avoir recours à un mode d'authentification moderne, comme les clés de sécurité FIDO2, pour accéder aux ressources Active Directory traditionnelles. Pour plus d'informations, accédez à [Authentification unique auprès de ressources locales](../authentication/howto-authentication-passwordless-security-key-on-premises.md). 

Commencez par visiter [Activer les clés de sécurité FIDO2 pour votre locataire](../authentication/howto-authentication-passwordless-security-key.md) pour obtenir des instructions pas à pas. 

---
 
### <a name="the-new-my-account-experience-is-now-generally-available"></a>La nouvelle expérience Mon compte est désormais généralement disponible

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Mon profil/compte  
**Fonctionnalité de produit :** Expériences d’utilisateur final
 
Mon compte, le guichet unique pour tous les besoins de gestion des comptes d'utilisateurs finaux, est désormais généralement disponible. Les utilisateurs finaux peuvent accéder à ce nouveau site via une URL, ou à partir de l'en-tête de la nouvelle expérience Mes applications. Pour en savoir plus sur toutes les fonctionnalités en libre-service offertes par la nouvelle expérience, consultez la [Présentation du portail Mon compte](../user-help/my-account-portal-overview.md).

---
 
### <a name="my-account-site-url-updating-to-myaccountmicrosoftcom"></a>Remplacement de l'URL du site Mon compte par myaccount.microsoft.com

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Mon profil/compte  
**Fonctionnalité de produit :** Expériences d’utilisateur final
 
Le mois prochain, l'URL de la nouvelle expérience d'utilisateur final Mon compte sera remplacée par `https://myaccount.microsoft.com`. Pour en savoir plus sur cette expérience et sur les différentes fonctionnalités en libre-service qu'elle offre aux utilisateurs finaux, consultez [Aide du portail Mon compte](../user-help/my-account-portal-overview.md).

---

## <a name="january-2020"></a>Janvier 2020
 
### <a name="the-new-my-apps-portal-is-now-generally-available"></a>Le nouveau portail Mes applications est désormais généralement disponible

**Type :** Modification planifiée  
**Catégorie de service :** My Apps  
**Fonctionnalité de produit :** Expériences d’utilisateur final
 
Mettez à niveau votre organisation vers le nouveau portail Mes applications qui est désormais généralement disponible. Pour plus d’informations sur le nouveau portail et les regroupements, voir [Créer des collections sur le portail Mes applications](../manage-apps/access-panel-collections.md).

---
 
### <a name="workspaces-in-azure-ad-have-been-renamed-to-collections"></a>Les espaces de travail dans Azure AD sont désormais appelés collections

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** My Apps   
**Fonctionnalité de produit :** Expériences d’utilisateur final
 
Les espaces de travail, les filtres que les administrateurs peuvent configurer pour organiser les applications de leurs utilisateurs, sont désormais appelés collections. Pour plus d’informations sur la façon de les configurer, voir [Créer des collections sur le portail Mes applications](../manage-apps/access-panel-collections.md).

---
 
### <a name="azure-ad-b2c-phone-sign-up-and-sign-in-using-custom-policy-public-preview"></a>Inscription et connexion par téléphone à Azure AD B2C à l’aide d’une stratégie personnalisée (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C
 
Avec l’inscription et la connexion par téléphone, les développeurs et les entreprises peuvent autoriser leurs clients à s’inscrire et à se connecter à l’aide d’un mot de passe à usage unique envoyé au numéro de téléphone de l’utilisateur via SMS. Cette fonctionnalité permet également au client de modifier son numéro de téléphone s’il perd l’accès à son téléphone. La puissance des stratégies personnalisées et de l’inscription et de la connexion par téléphone permet aux développeurs et aux entreprises de mettre leur marque en avant en personnalisant des pages. Découvrez comment [Configurer l’inscription et la connexion par téléphone avec des stratégies personnalisées dans Azure AD B2C](../../active-directory-b2c/phone-authentication.md).
 
---
 
### <a name="new-provisioning-connectors-in-the-azure-ad-application-gallery---january-2020"></a>Nouveaux connecteurs d’approvisionnement dans la Galerie d’applications Azure AD – Janvier 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Promapp]( ../saas-apps/promapp-provisioning-tutorial.md)
- [Zscaler Private Access](../saas-apps/zscaler-private-access-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---
 
### <a name="new-federated-apps-available-in-azure-ad-app-gallery---january-2020"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD – Janvier 2020

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce
 
En janvier 2020, nous avons ajouté à la galerie d’applications les 33 applications suivantes qui prennent en charge la fédération : 

[JOSA](../saas-apps/josa-tutorial.md), [Fastly Edge Cloud](../saas-apps/fastly-edge-cloud-tutorial.md), [Terraform Enterprise](../saas-apps/terraform-enterprise-tutorial.md), [Spintr SSO](../saas-apps/spintr-sso-tutorial.md), [Abibot Netlogistik](https://azuremarketplace.microsoft.com/marketplace/apps/aad.abibotnetlogistik), [SkyKick](https://login.skykick.com/login?state=g6Fo2SBTd3M5Q0xBT0JMd3luS2JUTGlYN3pYTE1remJQZnR1c6N0aWTZIDhCSkwzYVQxX2ZMZjNUaWxNUHhCSXg2OHJzbllTcmYto2NpZNkgM0h6czk3ZlF6aFNJV1VNVWQzMmpHeFFDbDRIMkx5VEc&client=3Hzs97fQzhSIWUMUd32jGxQCl4H2LyTG&protocol=oauth2&audience=https://papi.skykick.com&response_type=code&redirect_uri=https://portal.skykick.com/callback&scope=openid%20profile%20offline_access), [Upshotly](../saas-apps/upshotly-tutorial.md), [LeaveBot](https://leavebot.io/#home), [DataCamp](../saas-apps/datacamp-tutorial.md), [TripActions](../saas-apps/tripactions-tutorial.md), [SmartWork](https://www.intumit.com/english/SmartWork.html), [Dotcom-Monitor](../saas-apps/dotcom-monitor-tutorial.md), [SSOGEN - Azure AD SSO Gateway pour Oracle E-Business Suite - EBS, PeopleSoft et JDE](../saas-apps/ssogen-tutorial.md), [Hosted MyCirqa SSO](../saas-apps/hosted-mycirqa-sso-tutorial.md), [Yuhu Property Management Platform](../saas-apps/yuhu-property-management-platform-tutorial.md), [LumApps](https://sites.lumapps.com/login), [Upwork Enterprise](../saas-apps/upwork-enterprise-tutorial.md), [Talentsoft](../saas-apps/talentsoft-tutorial.md), [SmartDB for Microsoft Teams](http://teams.smartdb.jp/login/), [PressPage](../saas-apps/presspage-tutorial.md), [ContractSafe Saml2 SSO](../saas-apps/contractsafe-saml2-sso-tutorial.md), [Maxient Conduct Manager Software](../saas-apps/maxient-conduct-manager-software-tutorial.md), [Helpshift](../saas-apps/helpshift-tutorial.md), [PortalTalk 365](https://www.portaltalk.com/), [CoreView](https://portal.coreview.com/), [Squelch Cloud Office365 Connector](https://laxmi.squelch.io/login), [PingFlow Authentication](https://app-staging.pingview.io/), [ PrinterLogic SaaS](../saas-apps/printerlogic-saas-tutorial.md), [Taskize Connect](../saas-apps/taskize-connect-tutorial.md), [Sandwai](https://app.sandwai.com/), [EZRentOut](../saas-apps/ezrentout-tutorial.md), [AssetSonar](../saas-apps/assetsonar-tutorial.md), [Akari Virtual Assistant](https://akari.io/akari-virtual-assistant/)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="two-new-identity-protection-detections"></a>Deux nouvelles détections de protection des identités

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Identity Protection  
**Fonctionnalité de produit :** Protection et sécurité des identités
 
Nous avons ajouté deux nouveaux types de détections liés à l’authentification à la protection des données : Règles suspectes de manipulation de boîte de réception et Voyage impossible. Ces détections hors connexion sont découvertes par Microsoft Cloud App Security (MCAS) et influencent l’utilisateur et le risque de connexion dans Identity Protection. Pour plus d’informations sur ces détections, voir nos [types de risques de connexion](../identity-protection/concept-identity-protection-risks.md#sign-in-risk).
 
---
 
### <a name="breaking-change-uri-fragments-will-not-be-carried-through-the-login-redirect"></a>Changement cassant : Des fragments d’URI ne sont pas véhiculés via la redirection de la connexion

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur
 
Depuis le 8 février 2020, quand une demande est envoyée à login.microsoftonline.com pour connecter un utilisateur, le service ajoute un fragment vide à la demande.  Cela évite une classe d’attaques de redirection en veillant à ce que le navigateur efface tout fragment existant dans la demande. Aucune application ne doit dépendre de ce comportement. Pour plus d’informations, voir [Changements cassants](../develop/reference-breaking-changes.md#february-2020) dans la documentation de la plateforme Microsoft Identity.

---

## <a name="december-2019"></a>Décembre 2019

### <a name="integrate-sap-successfactors-provisioning-into-azure-ad-and-on-premises-ad-public-preview"></a>Intégrer l’approvisionnement de SAP SuccessFactors dans Azure AD et dans un AD local (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Provisionnement d’applications  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités

Vous pouvez désormais intégrer SAP SuccessFactors en tant que source d’identité faisant autorité dans Azure AD. Cette intégration vous permet d’automatiser le cycle de vie des identités de bout en bout, y compris l’utilisation d’événements basés sur les RH, tels que des embauches ou des résiliations, pour contrôler l’approvisionnement de comptes Azure AD.

Pour plus d’informations sur la configuration de l’approvisionnement entrant de SAP SuccessFactors pour Azure AD, voir le didacticiel [Configurer l’approvisionnement automatique de SAP SuccessFactors](../saas-apps/sap-successfactors-inbound-provisioning-tutorial.md).

---

### <a name="support-for-customized-emails-in-azure-ad-b2c-public-preview"></a>Prise en charge des e-mails personnalisés dans Azure AD B2C (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2C - Gestion des identités consommateurs  
**Fonctionnalité de produit :** B2B/B2C

Vous pouvez désormais utiliser Azure AD B2C pour créer des e-mails personnalisés lorsque vos utilisateurs s’inscrivent pour utiliser vos applications. En utilisant DisplayControls (actuellement en préversion) et un fournisseur de courrier tiers (tel que [SendGrid](https://sendgrid.com/), [SparkPost](https://sparkpost.com/) ou une API REST personnalisée), vous pouvez utiliser vos propres modèle d’e-mail, adresse **D** et texte d’objet, ainsi que des paramètres de localisation du support et de mot de passe à usage unique personnalisé.

Pour plus d’informations, voir [Vérification d’e-mail personnalisée dans Azure Active Directory B2C](../../active-directory-b2c/custom-email-sendgrid.md).

---

### <a name="replacement-of-baseline-policies-with-security-defaults"></a>Remplacement des stratégies de base par des paramètres de sécurité par défaut

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Autres  
**Fonctionnalité de produit :** Protection et sécurité des identités

Dans le cadre d’un modèle sécurisé par défaut pour l’authentification, nous supprimons de tous les locataires les stratégies de protection de base existantes. Cette suppression devrait être terminée fin février. Ces stratégies de protection de base sont remplacées par des paramètres de sécurité par défaut. Si vous avez utilisé des stratégies de protection de base, vous devez planifier un passage à la nouvelle stratégie des paramètres de sécurité par défaut ou à l’accès conditionnel. Si vous n’avez pas utilisé ces stratégies, vous n’avez aucune action à effectuer.

Pour plus d’informations sur les nouveaux paramètres de sécurité par défaut, voir [Présentation des paramètres de sécurité par défaut](./concept-fundamentals-security-defaults.md). Pour plus d’informations sur les stratégies d’accès conditionnel, voir [Stratégies d’accès conditionnel courantes](../conditional-access/concept-conditional-access-policy-common.md).

---

## <a name="november-2019"></a>Novembre 2019

### <a name="support-for-the-samesite-attribute-and-chrome-80"></a>Prise en charge de l’attribut SameSite et de Chrome 80

**Type :** Modification planifiée  
**Catégorie de service :** Authentifications (connexions)  
**Fonctionnalité de produit :** Authentification utilisateur

Dans le cadre d’un modèle sécurisé par défaut pour les cookies, le navigateur Chrome 80 change la façon dont il traite les cookies sans l’attribut `SameSite`. Tout cookie qui ne spécifie pas l’attribut `SameSite` sera traité comme s’il était défini sur `SameSite=Lax`, ce qui entraînera un blocage par Chrome de certains scénarios de partage de cookies entre domaines dont votre application peut dépendre. Pour conserver l’ancien comportement de Chrome, vous pouvez utiliser l’attribut `SameSite=None` et ajouter un attribut `Secure` supplémentaire de sorte que des cookies intersites ne soient accessibles que via des connexions HTTPs. Chrome est programmé pour accomplir ce changement au plus tard le 4 février 2020.

Nous recommandons à tous nos développeurs de tester leurs applications en suivant ces conseils :

- Définissez la valeur par défaut du paramètre **Utiliser un cookie sécurisé** sur **Oui**.

- Définissez la valeur par défaut de l’attribut **SameSite** sur **Aucun**.

- Ajoutez un attribut `SameSite` supplémentaire de **Sécurisé**.

Pour plus d’informations, voir [Modifications de cookie SameSite à venir dans ASP.NET et ASP.NET Core](https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/) et [Perturbation potentielle des sites web client et des produits et services Microsoft dans la version 79 et les versions ultérieures de Chrome](https://support.microsoft.com/help/4522904/potential-disruption-to-microsoft-services-in-chrome-beta-version-79).

---

### <a name="new-hotfix-for-microsoft-identity-manager-mim-2016-service-pack-2-sp2"></a>Nouveau correctif logiciel pour Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2)

**Type :** Résolution  
**Catégorie de service :** Microsoft Identity Manager  
**Fonctionnalité de produit :** Gestion du cycle de vie des identités

Un correctif logiciel cumulatif (build 4.6.34.0) est disponible pour Microsoft Identity Manager (MIM) 2016 Service Pack 2 (SP2). Ce correctif cumulatif résout des problèmes et ajoute des améliorations décrites dans la section « Problèmes résolus et améliorations ajoutées dans cette mise à jour ».

Pour plus d’informations et pour télécharger le correctif logiciel, voir [Disponibilité d’un correctif cumulatif Microsoft Identity Manager 2016 Service Pack 2 (Build 4.6.34.0)](https://support.microsoft.com/help/4512924/microsoft-identity-manager-2016-service-pack-2-build-4-6-34-0-update-r).

---

### <a name="new-ad-fs-app-activity-report-to-help-migrate-apps-to-azure-ad-public-preview"></a>Nouveau rapport activité d’application AD FS pour faciliter la migration d’applications vers Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO

Utilisez le nouveau rapport d’activité d’application des services de fédération Active Directory (AD FS) dans le Portail Azure pour identifier vos applications pouvant être migrées vers Azure AD. Il évalue la compatibilité de toutes les applications AD FS avec Azure AD, recherche tout problème et fournit des instructions sur la préparation d’applications individuelles pour la migration.

Pour plus d’informations, voir [Utiliser le rapport d’activité des applications AD FS (préversion) pour migrer des applications vers Azure AD](../manage-apps/migrate-adfs-application-activity.md).

---

### <a name="new-workflow-for-users-to-request-administrator-consent-public-preview"></a>Nouveau workflow permettant aux utilisateurs de demander le consentement de l’administrateur (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Contrôle d’accès

Le nouveau workflow de consentement de l’administrateur offre aux administrateurs une méthode pour accorder l’accès aux applications qui requièrent l’approbation de l’administrateur. Si un utilisateur tente d’accéder à une application mais n’est pas en mesure de donner son consentement, il peut désormais envoyer une demande d’approbation de l’administrateur. La demande est envoyée par courrier électronique et placée dans une file d’attente accessible à partir du portail Azure à tous les administrateurs désignés comme réviseurs. Une fois qu’un réviseur a effectué une action sur une demande en attente, les utilisateurs à l’origine de la demande sont avertis de l’action.

Pour plus d’informations, voir [Configurer le workflow du consentement de l’administrateur (préversion)](../manage-apps/configure-admin-consent-workflow.md).

---

### <a name="new-azure-ad-app-registrations-token-configuration-experience-for-managing-optional-claims-public-preview"></a>Nouvelle expérience de configuration de jeton d’inscription Azure AD App pour la gestion de revendications facultatives (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Autres  
**Fonctionnalité de produit :** Expérience de développement

Le nouveau panneau de **configuration de jeton d’inscription Azure AD App** sur le portail Azure présente désormais aux développeurs d’applications une liste dynamique de revendications facultatives pour leurs applications. Cette nouvelle expérience permet de simplifier les migrations d’applications Azure AD et de minimiser les erreurs de configuration de revendications facultatives.

Pour plus d’informations, voir [Fournir des revendications facultatives à votre application Azure AD](../develop/active-directory-optional-claims.md).

---

### <a name="new-two-stage-approval-workflow-in-azure-ad-entitlement-management-public-preview"></a>Nouveau workflow d’approbation en deux étapes dans la gestion des droits d’utilisation Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Autres  
**Fonctionnalité de produit :** Gestion des droits d’utilisation

Nous avons introduit un nouveau workflow d’approbation en deux étapes qui vous permet de demander à deux approbateurs d’approuver la demande de package d’accès d’un utilisateur. Par exemple, vous pouvez le définir afin que le gestionnaire de l’utilisateur demandeur doive préalablement approuver, et vous pouvez également exiger qu’un propriétaire de ressource approuve. Si l’un des approbateurs n’approuve pas, l’accès n’est pas accordé.

Pour plus d’informations, voir [Modifier les paramètres de demande et d’approbation d’un package d’accès dans la gestion des droits d’utilisation Azure AD](../governance/entitlement-management-access-package-request-policy.md).

---

### <a name="updates-to-the-my-apps-page-along-with-new-workspaces-public-preview"></a>Mises à jour de la page Mes applications avec les nouveaux espaces de travail (préversion publique)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** My Apps  
**Fonctionnalité de produit :** Intégration tierce

Vous pouvez désormais personnaliser la manière dont les utilisateurs de votre organisation accèdent à l’expérience Mes applications actualisée et l’affichent. Cette nouvelle expérience comprend également la nouvelle fonctionnalité d’espaces de travail, qui permet aux utilisateurs de trouver et d’organiser plus facilement des applications.

Pour plus d’informations sur la nouvelle expérience Mes applications et la création d’espaces de travail, voir [Créer des espaces de travail sur le portail Mes applications](../manage-apps/access-panel-collections.md).

---

### <a name="google-social-id-support-for-azure-ad-b2b-collaboration-general-availability"></a>Prise en charge d’ID social Google pour la collaboration dans Azure AD B2B (disponibilité générale)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** B2B  
**Fonctionnalité de produit :** Authentification utilisateur

La nouvelle prise en charge de l’utilisation d’ID sociaux Google (comptes Gmail) dans Azure AD permet de simplifier la collaboration pour vos utilisateurs et partenaires. Vos partenaires n’ont plus besoin de créer et gérer un nouveau compte spécifique de Microsoft. Microsoft Teams prend désormais entièrement en charge les utilisateurs Google sur tous les clients, ainsi que sur les points de terminaison d’authentification communs et associés à des locataires.

Pour plus d’informations, voir [Ajouter Google comme fournisseur d’identité pour les utilisateurs invités B2B](../external-identities/google-federation.md).

---

### <a name="microsoft-edge-mobile-support-for-conditional-access-and-single-sign-on-general-availability"></a>Prise en charge mobile de Microsoft Edge pour l’accès conditionnel et l’authentification unique (disponibilité générale)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Accès conditionnel  
**Fonctionnalité de produit :** Protection et sécurité des identités

Azure AD pour Microsoft Edge sur iOS et Android prend désormais en charge l’authentification unique et l’accès conditionnel Azure AD :

- **Authentification unique (SSO) Microsoft Edge :** L’authentification unique est désormais disponible sur les clients natifs (tels que Microsoft Outlook et Microsoft Edge) pour toutes les applications connectées à Azure AD.

- **Accès conditionnel Microsoft Edge :** En vertu des stratégies d’accès conditionnel basées sur l’application, vos utilisateurs doivent utiliser des navigateurs protégés par Microsoft Intune, tels que Microsoft Edge.

Pour plus d’informations sur l’accès conditionnel et l’authentification unique avec Microsoft Edge, voir le billet de blog [Prise en charge mobile de Microsoft Edge pour l’accès conditionnel et l’authentification unique désormais généralement disponible](https://techcommunity.microsoft.com/t5/Intune-Customer-Success/Microsoft-Edge-Mobile-Support-for-Conditional-Access-and-Single/ba-p/988179). Pour plus d’informations sur la configuration de vos applications clientes à l’aide d’un [accès conditionnel basé sur l’application](../conditional-access/app-based-conditional-access.md) ou d’un [accès conditionnel basé sur l’appareil](../conditional-access/require-managed-devices.md), voir [Gérer l’accès web à l’aide d’un navigateur protégé par une stratégie Microsoft Intune](/intune/apps/app-configuration-managed-browser).

---

### <a name="azure-ad-entitlement-management-general-availability"></a>Gestion des droits d’utilisation Azure AD (disponibilité générale)

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Autres  
**Fonctionnalité de produit :** Gestion des droits d’utilisation

La gestion des droits d’utilisation Azure AD est une nouvelle fonctionnalité de gouvernance des identités qui aide les organisations à gérer le cycle de vie des identités et des accès à grande échelle. Cette nouvelle fonctionnalité automatise les workflows de demande d’accès, les affectations d’accès, les révisions et les expirations pour des groupes, des applications et des sites SharePoint Online.

Avec la gestion des droits d’utilisation Azure AD, vous pouvez gérer plus efficacement l’accès des employés et des utilisateurs extérieurs à votre organisation qui doivent accéder à ces ressources.

Pour plus d’informations, voir [Qu’est-ce que la gestion des droits d’utilisation Azure AD ?](../governance/entitlement-management-overview.md#license-requirements)

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatiser l’approvisionnement de compte d’utilisateur pour ces applications SaaS nouvellement prises en charge

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce  

Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

[Service SAP Cloud Platform Identity Authentication](../saas-apps/sap-hana-cloud-platform-identity-authentication-tutorial.md), [RingCentral](../saas-apps/ringcentral-provisioning-tutorial.md), [SpaceIQ](../saas-apps/spaceiq-provisioning-tutorial.md), [Miro](../saas-apps/miro-provisioning-tutorial.md), [Cloudgate](../saas-apps/soloinsight-cloudgate-sso-provisioning-tutorial.md), [Infor CloudSuite](../saas-apps/infor-cloudsuite-provisioning-tutorial.md), [OfficeSpace Software](../saas-apps/officespace-software-provisioning-tutorial.md), [Priority Matrix](../saas-apps/priority-matrix-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---november-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD – Novembre 2019

**Type :** Nouvelle fonctionnalité  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** Intégration tierce

En novembre 2019, nous avons ajouté à la galerie d’applications les 21 nouvelles applications ci-après qui prennent en charge la fédération :

[Airtable](../saas-apps/airtable-tutorial.md), [Hootsuite](../saas-apps/hootsuite-tutorial.md), [Blue Access for Members (BAM)](../saas-apps/blue-access-for-members-tutorial.md), [Bitly](../saas-apps/bitly-tutorial.md), [Riva](../saas-apps/riva-tutorial.md), [ResLife Portal](https://app.reslifecloud.com/hub5_signin/microsoft_azuread/?g=44BBB1F90915236A97502FF4BE2952CB&c=5&uid=0&ht=2&ref=), [NegometrixPortal Single Sign On (SSO)](../saas-apps/negometrixportal-tutorial.md), [TeamsChamp](https://login.microsoftonline.com/551f45da-b68e-4498-a7f5-a6e1efaeb41c/adminconsent?client_id=ca9bbfa4-1316-4c0f-a9ee-1248ac27f8ab&redirect_uri=https://admin.teamschamp.com/api/adminconsent&state=6883c143-cb59-42ee-a53a-bdb5faabf279), [Motus](../saas-apps/motus-tutorial.md), [MyAryaka](../saas-apps/myaryaka-tutorial.md), [BlueMail](https://loginself1.bluemail.me/), [Beedle](https://teams-web.beedle.co/#/), [Visma](../saas-apps/visma-tutorial.md), [OneDesk](../saas-apps/onedesk-tutorial.md), [Foko Retail](../saas-apps/foko-retail-tutorial.md), [Qmarkets Idea & Innovation Management](../saas-apps/qmarkets-idea-innovation-management-tutorial.md), [Netskope User Authentication](../saas-apps/netskope-user-authentication-tutorial.md), [uniFLOW Online](../saas-apps/uniflow-online-tutorial.md), [Claromentis](../saas-apps/claromentis-tutorial.md), [Jisc Student Voter Registration](../saas-apps/jisc-student-voter-registration-tutorial.md), [e4enable](https://portal.e4enable.com/)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-and-improved-azure-ad-application-gallery"></a>Galerie d’applications Azure AD nouvelles et améliorées

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO

Nous avons mis à jour la galerie d’applications Azure AD pour faciliter la recherche d’applications pré-intégrées qui prennent en charge l’approvisionnement, OpenID Connect et SAML sur votre client Azure Active Directory.

Pour plus d’informations, vois [Ajouter une application à votre locataire Azure Active Directory](../manage-apps/add-application-portal.md).

---

### <a name="increased-app-role-definition-length-limit-from-120-to-240-characters"></a>Limite de longueur de définition de rôle d’application augmentée de 120 à 240 caractères

**Type :** Fonctionnalité modifiée  
**Catégorie de service :** Applications d’entreprise  
**Fonctionnalité de produit :** SSO

Nous avons entendu des clients dire que la longueur limitée à 120 caractères de la valeur de définition de rôle d’application dans certains services et applications était insuffisante. En réponse à cela, nous avons porté la longueur maximale de la valeur de définition de rôle à 240 caractères.

Pour plus d’informations sur l’utilisation des définitions de rôle spécifiques de l’application, voir [Ajouter des rôles d’application dans votre application et les recevoir dans le jeton](../develop/howto-add-app-roles-in-azure-ad-apps.md).

---

## <a name="october-2019"></a>2 octobre 2019

### <a name="deprecation-of-the-identityriskevent-api-for-azure-ad-identity-protection-risk-detections"></a>Dépréciation de l’API identityRiskEvent pour la détection des risques par Azure AD Identity Protection

**Type :** Modification planifiée **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Suite aux commentaires des développeurs, les abonnés à Azure AD Premium P2 peuvent désormais effectuer des requêtes complexes sur les données de détection des risques d’Azure AD Identity Protection à l’aide de la nouvelle API riskDetection pour Microsoft Graph. La version bêta de l’API [identityRiskEvent](/graph/api/resources/identityriskevent?view=graph-rest-beta) existante cessera de renvoyer des données vers le **10 janvier 2020**. Si votre organisation utilise l’API identityRiskEvent, vous devez passer à la nouvelle API riskDetection.

Pour plus d’informations sur la nouvelle API riskDetection, consultez la [Documentation de référence sur l’API de détection des risques](/graph/api/resources/riskdetection).

---

### <a name="application-proxy-support-for-the-samesite-attribute-and-chrome-80"></a>Prise en charge du proxy d'application pour l'attribut SameSite et Chrome 80

**Type :** Modification planifiée **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

À quelques semaines de la publication du navigateur Chrome 80, nous envisageons de mettre à jour la manière dont les cookies du proxy d’application traitent l’attribut **SameSite**. Avec Chrome 80, tous les cookies qui ne spécifieront pas l’attribut **SameSite** seront traités comme s'ils avaient été définis sur `SameSite=Lax`.

Pour éviter tout impact négatif lié à ce changement, nous mettons à jour l’accès au proxy d’application et les cookies de session en procédant comme suit :

- Définition de la valeur par défaut du paramètre **Utiliser un cookie sécurisé** sur **Oui**.

- Définition de la valeur par défaut de l’attribut **SameSite** sur **Aucun**.

    >[!NOTE]
    > Les cookies d’accès au proxy d’application ont toujours été transmis exclusivement via des canaux sécurisés. Ces modifications s’appliquent uniquement aux cookies de session.

Pour plus d’informations sur les paramètres de cookies du proxy d'application, consultez [Paramètres de cookies pour l’accès aux applications locales dans Azure Active Directory](../manage-apps/application-proxy-configure-cookie-settings.md).

---

### <a name="app-registrations-legacy-and-app-management-in-the-application-registration-portal-appsdevmicrosoftcom-is-no-longer-available"></a>Les inscriptions d’applications (héritées) et la gestion des applications ne sont plus disponibles dans le portail d’inscription des applications (apps.dev.microsoft.com).

**Type :** Modification planifiée **Catégorie de service :** Non applicable **Fonctionnalité produit :** Expérience de développement

Les utilisateurs disposant de comptes Azure AD ne peuvent plus inscrire ni gérer les applications à l’aide du portail Inscriptions d’applications (apps.dev.microsoft.com). Ils ne peuvent plus non plus inscrire ni gérer des applications dans l’expérience Inscriptions des applications (héritées) du portail Azure.

Pour en savoir plus sur la nouvelle expérience en matière d'inscriptions d’applications, consultez le [Guide de formation sur les inscriptions d’applications dans le portail Azure](../develop/quickstart-register-app.md).

---

### <a name="users-are-no-longer-required-to-re-register-during-migration-from-per-user-mfa-to-conditional-access-based-mfa"></a>Les utilisateurs ne sont plus tenus de se réinscrire lors de la migration de l’authentification multifacteur par utilisateur à l’authentification multifacteur basée sur l’accès conditionnel.

**Type :** Corrigé **Catégorie de service :** MFA **Fonctionnalité produit :** Protection et sécurité des identités

Nous avons résolu un problème connu qui obligeait les utilisateurs à se réinscrire s’ils étaient désactivés de l’authentification multifacteur par utilisateur, puis activés pour l’authentification multifacteur basée sur l’accès conditionnel.

Pour contraindre les utilisateurs à se réinscrire, vous pouvez sélectionner l’option **Réinscription à l'authentification multifacteur requise** à partir des méthodes d’authentification de l’utilisateur dans le portail Azure AD. Pour plus d’informations sur la migration des utilisateurs de l’authentification multifacteur (MFA) par utilisateur vers l’authentification MFA basée sur l’accès conditionnel, consultez [Convertir des utilisateurs de l’authentification multifacteur par utilisateur à l’authentification multifacteur basée sur l’accès conditionnel](../authentication/howto-mfa-getstarted.md#convert-users-from-per-user-mfa-to-conditional-access-based-mfa).

---

### <a name="new-capabilities-to-transform-and-send-claims-in-your-saml-token"></a>Nouvelles fonctionnalités de transformation et d’envoi de revendications dans votre jeton SAML

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Nous avons ajouté des fonctionnalités supplémentaires pour vous aider à personnaliser et à envoyer des revendications dans votre jeton SAML. Ces fonctionnalités sont les suivantes :

- Fonctions de transformation des revendications supplémentaires pour vous aider à modifier la valeur que vous envoyez dans la revendication.

- Possibilité d’appliquer plusieurs transformations à une même revendication.

- Possibilité de spécifier la source de la revendication en fonction du type d’utilisateur et du groupe auquel appartient l’utilisateur.

Pour plus d’informations sur ces nouvelles fonctionnalités, notamment sur leur utilisation, consultez [Personnaliser des revendications émises dans le jeton SAML pour les applications d’entreprise](../develop/active-directory-saml-claims-customization.md).

---

### <a name="new-my-sign-ins-page-for-end-users-in-azure-ad"></a>Nouvelle page Mes connexions pour les utilisateurs finaux dans Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Monitoring et création de rapports

Nous avons ajouté une nouvelle page **Mes connexions**https://mysignins.microsoft.com) pour permettre aux utilisateurs de votre organisation d’afficher leur historique de connexion récent afin d'y détecter toute activité inhabituelle. Cette nouvelle page permet à vos utilisateurs de voir :

- Si quelqu’un tente de deviner leur mot de passe.

- Si un attaquant s’est connecté à leur compte et depuis quel emplacement.

- Les applications auxquelles l’attaquant a tenté d’accéder.

Pour plus d’informations, consultez le billet de blog [Les utilisateurs peuvent désormais consulter leur historique afin d'y détecter toute activité inhabituelle](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Users-can-now-check-their-sign-in-history-for-unusual-activity/ba-p/916066).

---

### <a name="migration-of-azure-ad-domain-services-azure-ad-ds-from-classic-to-azure-resource-manager-virtual-networks"></a>Migration d'Azure AD Domain Services (Azure AD DS) des réseaux virtuels classiques aux réseaux virtuels Azure Resource Manager

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD Domain Services **Fonctionnalité produit :** Services de domaine Azure AD

Nous sommes porteurs d'une bonne nouvelle pour nos clients bloqués sur des réseaux virtuels classiques. Vous pouvez désormais effectuer une migration unique à partir d’un réseau virtuel classique vers un réseau virtuel existant Resource Manager. Après avoir migré vers le réseau virtuel Resource Manager, vous pourrez tirer parti de fonctionnalités supplémentaires et mises à niveau, telles que les stratégies de mot de passe affinées, les notifications par e-mail et les journaux d’audit.

Pour plus d'informations, consultez [Préversion - Migrer Azure Active Directory Domain Services depuis le modèle de réseau virtuel classique vers Resource Manager](../../active-directory-domain-services/migrate-from-classic-vnet.md).

---

### <a name="updates-to-the-azure-ad-b2c-page-contract-layout"></a>Mise à jour de la disposition de contrat de page Azure AD B2C

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Nous avons apporté de nouvelles modifications à la version 1.2.0 du contrat de page Azure AD B2C. Dans cette version mise à jour, vous pouvez maintenant contrôler l’ordre de chargement des éléments, ce qui permet aussi de mettre fin au scintillement qui survient lorsque la feuille de style (CSS) est chargée.

Pour obtenir la liste complète des modifications apportées au contrat de page, consultez le [Journal des modifications de version](../../active-directory-b2c/page-layout.md#other-pages-providerselection-claimsconsent-unifiedssd).

---

### <a name="update-to-the-my-apps-page-along-with-new-workspaces-public-preview"></a>Mise à jour de la page Mes applications avec de nouveaux espaces de travail (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Mes applications **Fonctionnalité produit :** Contrôle d’accès

Vous pouvez désormais personnaliser la manière dont les utilisateurs de votre organisation affichent et accèdent à la nouvelle expérience Mes applications, notamment l’utilisation de la nouvelle fonctionnalité Espaces de travail pour faciliter la recherche d’applications. La nouvelle fonctionnalité Espaces de travail fait office de filtre pour les applications auxquelles les utilisateurs de votre organisation ont déjà accès.

Pour plus d’informations sur le déploiement de la nouvelle expérience Mes applications et la création d’espaces de travail, voir [Créer des espaces de travail sur le portail Mes applications (préversion)](../manage-apps/access-panel-collections.md).

---

### <a name="support-for-the-monthly-active-user-based-billing-model-general-availability"></a>Prise en charge du modèle de facturation basée sur les utilisateurs actifs mensuels (disponibilité générale)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Azure AD B2C prend désormais en charge la facturation basée sur les utilisateurs actifs mensuels. Cette facturation repose sur le nombre d’utilisateurs uniques avec activité d’authentification pendant un mois civil. Les clients existants peuvent à tout moment opter pour cette nouvelle méthode de facturation.

Depuis le 1er novembre 2019, tous les nouveaux clients sont automatiquement facturés à l’aide de cette méthode. Cette méthode de facturation offre aux clients des avantages en termes de coût et de planification.

Pour plus d’informations, consultez [Mise à niveau vers le modèle de facturation basée sur les utilisateurs actifs mensuels](../../active-directory-b2c/billing.md#switch-to-mau-billing-pre-november-2019-azure-ad-b2c-tenants).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---october-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD – Octobre 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En octobre 2019, nous avons ajouté à notre galerie d’applications les 35 nouvelles applications ci-dessous, qui prennent en charge la fédération :

[In Case of Crisis – Mobile](../saas-apps/in-case-of-crisis-mobile-tutorial.md), [Juno Journey](../saas-apps/juno-journey-tutorial.md), [ExponentHR](../saas-apps/exponenthr-tutorial.md), [Tact](https://www.tact.ai/products/tact-assistant), [OpusCapita Cash Management](https://appsource.microsoft.com/product/web-apps/opuscapitagroupoy-1036255.opuscapita-cm), [Salestim](https://www.salestim.com/), [Learnster](../saas-apps/learnster-tutorial.md), [Dynatrace](../saas-apps/dynatrace-tutorial.md), [HunchBuzz](https://login.hunchbuzz.com/integrations/azure/process), [Freshworks](../saas-apps/freshworks-tutorial.md), [eCornell](../saas-apps/ecornell-tutorial.md), [ShipHazmat](../saas-apps/shiphazmat-tutorial.md), [Netskope Cloud Security](../saas-apps/netskope-cloud-security-tutorial.md), [Contentful](../saas-apps/contentful-tutorial.md), [Bindtuning](https://bindtuning.com/login), [HireVue Coordinate – Europe](https://www.hirevue.com/), [HireVue Coordinate - USOnly](https://www.hirevue.com/), [HireVue Coordinate - US](https://www.hirevue.com/), [WittyParrot Knowledge Box](https://wittyapi.wittyparrot.com/wittyparrot/api/provision/trail/signup), [Cloudmore](../saas-apps/cloudmore-tutorial.md), [Visit.org](../saas-apps/visitorg-tutorial.md), [Cambium Xirrus EasyPass Portal](https://login.xirrus.com/azure-signup), [Paylocity](../saas-apps/paylocity-tutorial.md), [Mail Luck!](../saas-apps/mail-luck-tutorial.md), [Teamie](https://theteamie.com/), [Velocity for Teams](https://velocity.peakup.org/teams/login), [SIGNL4](https://account.signl4.com/manage), [EAB Navigate IMPL](../saas-apps/eab-navigate-impl-tutorial.md), [ScreenMeet](https://console.screenmeet.com/), [Omega Point](https://pi.ompnt.com/), [Speaking Email for Intune (iPhone)](https://speaking.email/FAQ/98/email-access-via-microsoft-intune), [Speaking Email for Office 365 Direct (iPhone/Android)](https://speaking.email/FAQ/126/email-access-via-microsoft-office-365-direct), [ExactCare SSO](../saas-apps/exactcare-sso-tutorial.md), [iHealthHome Care Navigation System](https://ihealthnav.com/account/signin), [Qubie](https://qubie.azurewebsites.net/static/adminTab/authorize.html)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="consolidated-security-menu-item-in-the-azure-ad-portal"></a>Élément de menu de sécurité consolidé dans le portail Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Vous pouvez désormais accéder à toutes les fonctionnalités de sécurité Azure AD disponibles à partir du nouvel élément de menu **Sécurité** et de la barre **Rechercher** du portail Azure. En outre, la nouvelle page d’accueil **Sécurité**, appelée **Sécurité - Prise en main**, propose des liens vers notre documentation publique, des conseils de sécurité, ainsi que des guides de déploiement.

Le nouveau menu **Sécurité** comprend ce qui suit :

- Accès conditionnel
- Identity Protection
- Security Center
- Identity Secure Score
- Méthodes d’authentification
- MFA
- Rapports sur les risques - Utilisateurs à risque, connexions risquées, détections de risques
- Et bien plus...

Pour plus d’informations, consultez [Sécurité - Prise en main](https://portal.azure.com/#blade/Microsoft_AAD_IAM/SecurityMenuBlade/GettingStarted).

---

### <a name="office-365-groups-expiration-policy-enhanced-with-autorenewal"></a>Stratégie d’expiration des groupes Office 365 améliorée avec renouvellement automatique

**Type :** Fonctionnalité modifiée **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Gestion du cycle de vie des identités

La stratégie d’expiration des groupes Office 365 a été améliorée pour renouveler automatiquement les groupes utilisés de manière active par leurs membres. Les groupes sont renouvelés sur la base de l'activité des utilisateurs dans toutes les applications Office 365, y compris Outlook, SharePoint et Teams.

Cette amélioration permet de réduire les notifications d’expiration de votre groupe et de veiller à ce que les groupes actifs continuent d'être disponibles. Si vous disposez déjà d’une stratégie d’expiration active pour vos groupes Office 365, vous n’avez rien à faire pour activer cette nouvelle fonctionnalité.

Pour plus d'informations, consultez [Configurer la stratégie d’expiration pour les groupes Office 365](../enterprise-users/groups-lifecycle.md).

---

### <a name="updated-azure-ad-domain-services-azure-ad-ds-creation-experience"></a>Expérience de création Azure AD Domain Services (Azure AD DS) mise à jour

**Type :** Fonctionnalité modifiée **Catégorie de service :** Azure AD Domain Services **Fonctionnalité produit :** Services de domaine Azure AD

Nous avons mis à jour Azure AD Domain Services (Azure AD DS) pour y inclure une nouvelle expérience de création améliorée, qui vous permet de créer un domaine géré en seulement trois clics. En outre, vous pouvez désormais charger et déployer Azure AD DS à partir d’un modèle.

Pour plus d’informations, consultez [Didacticiel : Créer et configurer une instance Azure Active Directory Domain Services](../../active-directory-domain-services/tutorial-create-instance.md).

---

## <a name="september-2019"></a>Septembre 2019

### <a name="plan-for-change-deprecation-of-the-power-bi-content-packs"></a>Planifier la modification : Dépréciation des packs de contenu Power BI

**Type :** Modification planifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

À partir du 1er octobre 2019, Power BI commencera à déprécier tous les packs de contenu, y compris le pack de contenu Azure AD Power BI. En guise d’alternative à ce pack de contenu, vous pouvez utiliser des classeurs Azure AD pour obtenir des informations sur les services liés à Azure AD. D’autres classeurs seront bientôt accessibles, y compris des classeurs pour les stratégies d’accès conditionnel en mode rapport uniquement, des insights basés sur le consentement de l’application, et plus.

Pour plus d’informations sur les classeurs, découvrez [comment utiliser des classeurs Azure Monitor pour créer des rapports Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md). Pour plus d’informations sur la dépréciation des packs de contenu, consultez le billet de blog [Annonce de la disponibilité générale des applications de modèle Power BI](https://powerbi.microsoft.com/blog/announcing-power-bi-template-apps-general-availability/).

---

### <a name="my-profile-is-renaming-and-integrating-with-the-microsoft-office-account-page"></a>Mon profil change de nom et s’intègre à la page du compte Microsoft Office

**Type :** Modification planifiée **Catégorie de service :** Mon profil/compte **Fonctionnalité produit :** Collaboration

À compter d’octobre, l’expérience de la page Mon profil deviendra Mon compte. Dans le cadre de cette modification, **Mon compte** s’affichera partout où il est actuellement indiqué **Mon profil**. Outre le changement de nom et quelques améliorations en matière de conception, la mise à jour de l’expérience offrira une intégration supplémentaire à la page du compte Microsoft Office. En particulier, vous serez en mesure d’accéder aux installations et aux abonnements Office à partir de la page **Vue d’ensemble du compte** ainsi qu’à des préférences de contact Office à partir de la page **Confidentialité**.

Pour plus d’informations sur l’expérience de la page Mon profil (préversion), consultez [Vue d’ensemble du portail Mon profil (préversion)](../user-help/my-account-portal-overview.md).

---

### <a name="bulk-manage-groups-and-members-using-csv-files-in-the-azure-ad-portal-public-preview"></a>Gérer en bloc les groupes et les membres à l’aide de fichiers CSV dans le portail Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Nous sommes heureux d’annoncer la disponibilité de la préversion publique des expériences de gestion en bloc des groupes dans le portail Azure AD. Vous pouvez maintenant utiliser un fichier CSV et le portail Azure AD pour gérer les groupes et les listes de membres, notamment :

- Ajout ou suppression de membres dans un groupe.

- Téléchargement de la liste des groupes à partir de l’annuaire.

- Téléchargement de la liste des membres d’un groupe spécifique.

Pour plus d’informations, consultez [Ajouter des membres en bloc](../enterprise-users/groups-bulk-import-members.md), [Supprimer en bloc des membres](../enterprise-users/groups-bulk-remove-members.md), [Télécharger une liste de membres en bloc](../enterprise-users/groups-bulk-download-members.md) et [Télécharger une liste de groupes en bloc](../enterprise-users/groups-bulk-download.md).

---

### <a name="dynamic-consent-is-now-supported-through-a-new-admin-consent-endpoint"></a>Le consentement dynamique est désormais pris en charge par un nouveau point de terminaison de consentement administrateur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Nous avons créé un point de terminaison de consentement administrateur pour prendre en charge le consentement dynamique, ce qui est utile pour les applications qui souhaitent utiliser le modèle de consentement dynamique sur la plateforme d’identités Microsoft.

Pour plus d’informations sur l’utilisation de ce nouveau point de terminaison, consultez [Utilisation du point de terminaison de consentement administrateur](../develop/v2-admin-consent.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---september-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD – Septembre 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En septembre 2019, nous avons ajouté à notre galerie d’applications les 29 nouvelles applications ci-dessous, qui prennent en charge la fédération :

[ScheduleLook](https://schedulelook.bbsonlineservices.net/), [MS Azure SSO Access for Ethidex Compliance Office&trade; - Single  sign-on](../saas-apps/ms-azure-sso-access-for-ethidex-compliance-office-tutorial.md), [iServer Portal](../saas-apps/iserver-portal-tutorial.md), [SKYSITE](../saas-apps/skysite-tutorial.md), [Concur Travel and Expense](../saas-apps/concur-travel-and-expense-tutorial.md), [WorkBoard](../saas-apps/workboard-tutorial.md), `https://apps.yeeflow.com/`, [ARC Facilities](../saas-apps/arc-facilities-tutorial.md), [Luware Stratus Team](https://stratus.emea.luware.cloud/login), [Wide Ideas](https://wideideas.online/wideideas/), [Prisma Cloud](../saas-apps/prisma-cloud-tutorial.md), [JDLT Client Hub](https://clients.jdlt.co.uk/login), [RENRAKU](../saas-apps/renraku-tutorial.md), [SealPath Secure Browser](https://protection.sealpath.com/SealPathInterceptorWopiSaas/Open/InstallSealPathEditorOneDrive), [Prisma Cloud](../saas-apps/prisma-cloud-tutorial.md), `https://app.penneo.com/`, `https://app.testhtm.com/settings/email-integration`, [Cintoo Cloud](https://aec.cintoo.com/login), [Whitesource](../saas-apps/whitesource-tutorial.md), [Hosted Heritage Online SSO](../saas-apps/hosted-heritage-online-sso-tutorial.md), [IDC](../saas-apps/idc-tutorial.md), [CakeHR](../saas-apps/cakehr-tutorial.md), [BIS](../saas-apps/bis-tutorial.md), [Coo Kai Team Build](https://ms-contacts.coo-kai.jp/), [Sonarqube](../saas-apps/sonarqube-tutorial.md), [Adobe Identity Management](../saas-apps/tutorial-list.md), [Discovery Benefits SSO](../saas-apps/discovery-benefits-sso-tutorial.md), [Amelio](https://app.amelio.co/), `https://itask.yipinapp.com/`

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-azure-ad-global-reader-role"></a>Nouveau rôle Lecteur général Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Rôles Azure AD **Fonctionnalité produit :** Contrôle d’accès

À compter du 24 septembre 2019, nous allons commencer à déployer un nouveau rôle Azure Active Directory (AD) nommé Lecteur général. Ce déploiement commencera avec les clients de cloud international et de production pour finir à l’échelle mondiale en octobre.

Le rôle Lecteur général est l’équivalent en lecture seule de l’administrateur général. Les utilisateurs dans ce rôle peuvent lire les paramètres et les informations d’administration entre les services Microsoft 365, mais ne peuvent pas entreprendre d’actions de gestion. Nous avons créé le rôle Lecteur général pour réduire le nombre d’administrateurs généraux de votre organisation. Étant donné que les comptes d’administrateur général sont puissants et vulnérables aux attaques, nous vous recommandons d’avoir moins de cinq administrateurs généraux. Nous vous recommandons d’utiliser le rôle Lecteur général pour la planification, les audits ou les examens. Nous vous recommandons également d’utiliser le rôle Lecteur général associé à d’autres rôles d’administrateur limités, comme Administrateur Exchange, pour vous aider à effectuer des tâches sans avoir besoin du rôle Administrateur général.

Le rôle Lecteur général fonctionne avec le nouveau Centre d’administration Microsoft 365, le Centre d’administration Exchange, le Centre d’administration Teams, le Centre de sécurité, le Centre de conformité, le Centre d’administration Azure AD et le Centre d’administration de gestion des appareils.

>[!NOTE]
> Au début de la préversion publique, le rôle Lecteur général ne fonctionnera pas avec : SharePoint, Privileged Access Management, Customer Lockbox, les étiquettes de confidentialité, Teams Lifecycle, Teams Reporting & Call Analytics, Teams IP Phone Device Management et Teams App Catalog.

Pour plus d’informations, consultez [Autorisations des rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).

---

### <a name="access-an-on-premises-report-server-from-your-power-bi-mobile-app-using-azure-active-directory-application-proxy"></a>Accéder à un serveur de rapports local à partir de votre application mobile Power BI à l’aide du proxy d’application Azure Active Directory

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Une nouvelle intégration entre l’application mobile Power BI et le proxy d’application Azure Active Directory vous permet de vous connecter en toute sécurité à l’application mobile Power BI et de voir les rapports de votre organisation hébergés sur Power BI Report Server en local.

Pour plus d’informations sur l’application mobile Power BI, notamment l’emplacement de téléchargement de l’application, consultez le [site Power BI](https://powerbi.microsoft.com/mobile/). Pour plus d’informations sur la configuration de l’application mobile Power BI avec le proxy d’application Azure Active Directory, consultez [Activer l’accès distant à Power BI Mobile avec le proxy d’application Azure AD](../manage-apps/application-proxy-integrate-with-power-bi.md).

---

### <a name="new-version-of-the-azureadpreview-powershell-module-is-available"></a>Une nouvelle version du module PowerShell AzureADPreview est disponible

**Type :** Fonctionnalité modifiée **Catégorie de service :** Autre **Fonctionnalité produit :** Répertoire

De nouvelles applets de commande ont été ajoutées au module AzureADPreview pour vous aider à définir et assigner des rôles personnalisés dans Azure AD, avec notamment ce qui suit :

- `Add-AzureADMSFeatureRolloutPolicyDirectoryObject`
- `Get-AzureADMSFeatureRolloutPolicy`
- `New-AzureADMSFeatureRolloutPolicy`
- `Remove-AzureADMSFeatureRolloutPolicy`
- `Remove-AzureADMSFeatureRolloutPolicyDirectoryObject`
- `Set-AzureADMSFeatureRolloutPolicy`

---

### <a name="new-version-of-azure-ad-connect"></a>Nouvelle version d’Azure AD Connect

**Type :** Fonctionnalité modifiée **Catégorie de service :** Autre **Fonctionnalité produit :** Répertoire

Nous avons publié une version mise à jour d’Azure AD Connect pour les clients de mise à niveau automatique. Cette nouvelle version comprend plusieurs nouvelles fonctionnalités, améliorations et correctifs de bogues. Pour plus d’informations sur cette nouvelle version, consultez [Azure AD Connect : historique de publication des versions](../hybrid/reference-connect-version-history.md#14250).

---

### <a name="azure-multi-factor-authentication-mfa-server-version-802-is-now-available"></a>Le serveur Azure Multi-Factor Authentication (MFA) version 8.0.2 est désormais disponible

**Type :** Corrigé **Catégorie de service :** MFA **Fonctionnalité produit :** Protection et sécurité des identités

Si vous êtes un client existant qui a activé le serveur MFA avant le 1er juillet 2019, vous pouvez maintenant télécharger la dernière version du serveur MFA (version 8.0.2). Dans cette nouvelle version, nous avons :

- résolu un problème de sorte qu’un e-mail est envoyé à l’utilisateur quand la synchronisation Azure AD fait passer l’état d’un utilisateur de désactivé à activé.

- résolu un problème de sorte que les clients peuvent effectuer correctement la mise à niveau tout en continuant d’utiliser la fonctionnalité Balises.

- ajouté l’indicatif de pays du Kosovo (+383).

- ajouté l’enregistrement d’audit de contournement à usage unique au journal MultiFactorAuthSvc.log.

- amélioré les performances du kit SDK du service web.

- corrigé d’autres bogues mineurs.

À partir du 1er juillet 2019, Microsoft a cessé d’offrir un serveur MFA pour les nouveaux déploiements. Les nouveaux clients qui exigent une authentification multifacteur doivent utiliser le service informatique Azure AD Multi-Factor Authentication. Pour plus d’informations, consultez [Planifier un déploiement informatique d’Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md).

---

## <a name="august-2019"></a>Août 2019

### <a name="enhanced-search-filtering-and-sorting-for-groups-is-available-in-the-azure-ad-portal-public-preview"></a>La recherche, le filtrage et le tri améliorés pour les groupes sont disponibles sur le portail Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Nous sommes heureux d'annoncer la disponibilité de la préversion publique des expériences améliorées liées aux groupes dans le portail Azure AD. Ces améliorations vous aident à mieux gérer les groupes et les listes de membres, en fournissant les éléments suivants :

- Fonctionnalités avancées de recherche, notamment la recherche de sous-chaînes dans des listes de groupes.
- Options avancées de filtrage et de tri sur des listes de membres et de propriétaires.
- Nouvelles fonctionnalités de recherche pour les listes de membres et de propriétaires.
- Comptage de groupes plus précis pour les grands groupes.

Pour plus d’informations, voir [Gérer des groupes dans le portail Azure](./active-directory-groups-members-azure-portal.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context).

---

### <a name="new-custom-roles-are-available-for-app-registration-management-public-preview"></a>De nouveaux rôles personnalisés sont disponibles pour la gestion de l’inscription des applications (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Rôles Azure AD **Fonctionnalité produit :** Contrôle d’accès

Les rôles personnalisés (disponibles avec un abonnement Azure AD P1 ou P2) peuvent désormais vous fournir un accès affiné, en vous permettant de créer des définitions de rôles avec des autorisations spécifiques, puis d’attribuer ces rôles à des ressources spécifiques. Actuellement, vous créez des rôles personnalisés en utilisant des autorisations pour gérer les inscriptions d’applications, puis en affectant le rôle à une application spécifique. Pour plus d’informations sur les rôles personnalisés, consultez [Rôles d’administrateur personnalisés dans Azure Active Directory (préversion)](../roles/custom-overview.md).

Si vous avez besoin de la prise en charge d’autorisations ou de ressources supplémentaires et que vous ne les voyez pas actuellement, vous pouvez envoyer des commentaires à notre [site de retours relatifs à Azure](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032) et nous ajouterons votre demande à notre feuille de route de mise à jour.

---

### <a name="new-provisioning-logs-can-help-you-monitor-and-troubleshoot-your-app-provisioning-deployment-public-preview"></a>Les nouveaux journaux d’approvisionnement peuvent vous aider à surveiller et à résoudre les problèmes de votre déploiement d’approvisionnement d’application (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Gestion du cycle de vie des identités

De nouveaux journaux de provisionnement sont disponibles pour vous aider à surveiller et à résoudre les problèmes de déploiement de l’approvisionnement des utilisateurs et des groupes. Ces nouveaux fichiers journaux incluent des informations sur les éléments suivants :

- Quels groupes ont été créés avec succès dans [ServiceNow](../saas-apps/servicenow-provisioning-tutorial.md)
- Quels rôles ont été importés à partir [d’Amazon Web Services (AWS)](../saas-apps/amazon-web-service-tutorial.md#configure-and-test-azure-ad-sso-for-amazon-web-services-aws) ?
- Les employés qui n’ont pas été importés à partir de [Workday](../saas-apps/workday-inbound-tutorial.md)

Pour en savoir plus, voir [Approvisionnement des rapports dans le portail Azure Active Directory (préversion)](../reports-monitoring/concept-provisioning-logs.md).

---

### <a name="new-security-reports-for-all-azure-ad-administrators-general-availability"></a>Nouveaux rapports de sécurité pour tous les administrateurs Azure AD (disponibilité générale)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Par défaut, tous les administrateurs Azure AD seront bientôt en mesure d’accéder à des rapports de sécurité modernes au sein d’Azure AD. Jusqu’à la fin du mois de septembre, vous pourrez utiliser la bannière en haut des rapports de sécurité modernes pour revenir aux anciens rapports.

Les rapports de sécurité modernes fournissent des fonctionnalités supplémentaires par rapport aux versions antérieures, notamment ce qui suit :

- Filtrage et tri avancés
- Actions en bloc, telles que le rejet de risque utilisateur
- Confirmation d’entités compromises ou sûres
- État du risque, à savoir : À risque, Ignoré, Corrigé ou Compromission confirmée.
- Nouvelles détections liées aux risques (disponibles pour les abonnés Azure AD Premium)

Pour plus d’informations, consultez [Utilisateurs à risque](../identity-protection/howto-identity-protection-investigate-risk.md#risky-users), [Connexions à risque](../identity-protection/howto-identity-protection-investigate-risk.md#risky-sign-ins) et [Détections de risques](../identity-protection/howto-identity-protection-investigate-risk.md#risk-detections).

---

### <a name="user-assigned-managed-identity-is-available-for-virtual-machines-and-virtual-machine-scale-sets-general-availability"></a>L’identité gérée affectée par l’utilisateur est disponible pour les machines virtuelles et les groupes de machines virtuelles identiques (disponibilité générale)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Identités managées pour les ressources Azure **Fonctionnalité produit :** Expérience de développement

Les identités gérées affectées par l’utilisateur sont désormais généralement disponibles pour les machines virtuelles et les groupes de machines virtuelles identiques. Dans le cadre de cet ajout, Azure peut créer une identité dans le locataire Azure AD approuvé par l’abonnement utilisé et peut être affecté à une ou plusieurs instances de service Azure. Pour en savoir plus sur les identités managées affectées par l’utilisateur, consultez la section [Que sont les identités managées pour les ressources Azure ?](../managed-identities-azure-resources/overview.md).

---

### <a name="users-can-reset-their-passwords-using-a-mobile-app-or-hardware-token-general-availability"></a>Les utilisateurs peuvent réinitialiser leur mot de passe à l’aide d’une application mobile ou d’un jeton matériel (disponibilité générale)

**Type :** Fonctionnalité modifiée **Catégorie de service :** Réinitialisation du mot de passe libre-service **Fonctionnalité produit :** Authentification utilisateur

Les utilisateurs qui ont inscrit une application mobile auprès de votre organisation peuvent désormais réinitialiser leur propre mot de passe en approuvant une notification de l’application Microsoft Authenticator ou en entrant un code à partir de leur application mobile ou d’un jeton matériel.

Pour plus d’informations, découvrez le [fonctionnement de ce processus : Réinitialisation de mot de passe en libre-service Azure AD](../authentication/concept-sspr-howitworks.md). Pour plus d’informations sur l’expérience utilisateur, consultez [Vue d’ensemble de la réinitialisation d’un mot de passe professionnel ou scolaire](../user-help/active-directory-passwords-reset-register.md).

---

### <a name="adalnet-ignores-the-msalnet-shared-cache-for-on-behalf-of-scenarios"></a>ADAL.NET ignore le cache partagé MSAL.NET pour les scénarios On-Behalf-Of

**Type :** Corrigé **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Depuis la version 5.0.0-preview de la bibliothèque d’authentification Azure AD (ADAL.NET), les développeurs d’applications doivent [sérialiser un cache par compte pour les applications web et les API web](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization#custom-token-cache-serialization-in-web-applications--web-api). À défaut, certains scénarios utilisant le [flux On-Behalf-Of](../develop/scenario-web-api-call-api-app-configuration.md?tabs=java) pour Java avec certains cas d’usage spécifiques de `UserAssertion` peuvent entraîner une élévation de privilège. Pour éviter cette vulnérabilité, ADAL.NET ignore désormais la bibliothèque d’authentification Microsoft pour le cache partagé dotnet (MSAL.NET) dans des scénarios On-Behalf-Of.

Pour plus d’informations sur ce problème, consultez le document qui traite de la [vulnérabilité liée à l’élévation de privilège de la bibliothèque d’authentification Active Directory](https://portal.msrc.microsoft.com/security-guidance/advisory/CVE-2019-1258).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---august-2019"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App - Août 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En août 2019, nous avons ajouté à notre galerie d’applications les 26 applications suivantes qui prennent en charge la fédération :

[Civic Platform](../saas-apps/civic-platform-tutorial.md), [Amazon Business](../saas-apps/amazon-business-tutorial.md), [ProNovos Ops Manager](../saas-apps/pronovos-ops-manager-tutorial.md), [Cognidox](../saas-apps/cognidox-tutorial.md), [Viareport’s Inativ Portal (Europe)](../saas-apps/viareports-inativ-portal-europe-tutorial.md), [Azure Databricks](https://azure.microsoft.com/services/databricks), [Robin](../saas-apps/robin-tutorial.md), [Academy Attendance](../saas-apps/academy-attendance-tutorial.md), [Priority Matrix](https://sync.appfluence.com/pmwebng/), [Cousto MySpace](https://cousto.platformers.be/account/login), [Uploadcare](https://uploadcare.com/accounts/signup/), [Carbonite Endpoint Backup](../saas-apps/carbonite-endpoint-backup-tutorial.md), [CPQSync by Cincom](../saas-apps/cpqsync-by-cincom-tutorial.md), [Chargebee](../saas-apps/chargebee-tutorial.md), [deliver.media&trade; Portal](https://portal.deliver.media), [Frontline Education](../saas-apps/frontline-education-tutorial.md), [F5](https://www.f5.com/products/security/access-policy-manager), [stashcat AD connect](https://www.stashcat.com), [Blink](../saas-apps/blink-tutorial.md), [Vocoli](../saas-apps/vocoli-tutorial.md), [ProNovos Analytics](../saas-apps/pronovos-analytics-tutorial.md), [Sigstr](../saas-apps/sigstr-tutorial.md), [Darwinbox](../saas-apps/darwinbox-tutorial.md), [Watch by Colors](../saas-apps/watch-by-colors-tutorial.md), [Harness](../saas-apps/harness-tutorial.md), [EAB Navigate Strategic Care](../saas-apps/eab-navigate-strategic-care-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-versions-of-the-azuread-powershell-and-azureadpreview-powershell-modules-are-available"></a>De nouvelles versions des modules PowerShell AzureAD et AzureADPreview sont disponibles

**Type :** Fonctionnalité modifiée **Catégorie de service :** Autre **Fonctionnalité produit :** Répertoire

De nouvelles mises à jour des modules PowerShell AzureAD et AzureADPreview sont disponibles :

- Un nouveau paramètre `-Filter` a été ajouté au paramètre `Get-AzureADDirectoryRole` dans le module AzureAD. Ce paramètre vous permet de filtrer sur les rôles d’annuaire retournés par l’applet de commande.
- De nouvelles applets de commande ont été ajoutées au module AzureADPreview pour vous aider à définir et assigner des rôles personnalisés dans Azure AD, avec notamment ce qui suit :

    - `Get-AzureADMSRoleAssignment`
    - `Get-AzureADMSRoleDefinition`
    - `New-AzureADMSRoleAssignment`
    - `New-AzureADMSRoleDefinition`
    - `Remove-AzureADMSRoleAssignment`
    - `Remove-AzureADMSRoleDefinition`
    - `Set-AzureADMSRoleDefinition`

---

### <a name="improvements-to-the-ui-of-the-dynamic-group-rule-builder-in-the-azure-portal"></a>Améliorations apportées à l’interface utilisateur du générateur de règles de groupe dynamique dans le portail Azure

**Type :** Fonctionnalité modifiée **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Nous avons apporté des améliorations à l’interface utilisateur du générateur de règles de groupe dynamique, disponible dans le portail Azure, pour vous aider à configurer plus facilement une nouvelle règle ou à modifier les règles existantes. Cette amélioration de la conception vous permet de créer des règles avec cinq expressions au lieu d’une seule. Nous avons également mis à jour la liste des propriétés d’appareil pour supprimer les propriétés d’appareil déconseillées.

Pour plus d’informations, consultez gérer les [règles d’appartenance dynamique](../enterprise-users/groups-dynamic-membership.md).

---

### <a name="new-microsoft-graph-app-permission-available-for-use-with-access-reviews"></a>Nouvelle autorisation d’application Microsoft Graph disponible pour une utilisation avec des révisions d’accès

**Type :** Fonctionnalité modifiée **Catégorie de service :** Révisions d’accès **Fonctionnalité produit :** Gouvernance des identités

Nous avons introduit une nouvelle autorisation d’application Microsoft Graph, `AccessReview.ReadWrite.Membership`, qui permet aux applications de créer et de récupérer automatiquement les révisions d’accès pour les appartenances aux groupes et les affectations d’applications. Cette autorisation peut être utilisée par vos tâches planifiées ou dans le cadre de votre automation, sans nécessiter de contexte d’utilisateur connecté.

Pour plus d’informations, consultez [Exemple de création de révisions d’accès Azure AD avec les autorisations d’application Microsoft Graph sur le blog PowerShell](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-how-to-create-Azure-AD-access-reviews-using-Microsoft/m-p/807241).

---

### <a name="azure-ad-activity-logs-are-now-available-for-government-cloud-instances-in-azure-monitor"></a>Les journaux d’activité Azure AD sont désormais disponibles pour les instances Government Cloud dans Azure Monitor

**Type :** Fonctionnalité modifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous avons le plaisir de vous annoncer que les journaux d’activité Azure AD sont désormais disponibles pour les instances Government Cloud dans Azure Monitor. Vous pouvez désormais envoyer des journaux Azure AD à votre compte de stockage ou à un Event Hub pour les intégrer à vos outils SIEM, tels que [Sumologic](../reports-monitoring/howto-integrate-activity-logs-with-sumologic.md), [Splunk](../reports-monitoring/howto-integrate-activity-logs-with-splunk.md) et [ArcSight](../reports-monitoring/howto-integrate-activity-logs-with-arcsight.md).

Pour plus d’informations sur la configuration d’Azure Monitor, consultez [Journaux d’activité Azure Active Directory dans Azure Monitor](../reports-monitoring/concept-activity-logs-azure-monitor.md#cost-considerations).

---

### <a name="update-your-users-to-the-new-enhanced-security-info-experience"></a>Mettez à jour vos utilisateurs vers la nouvelle expérience d’informations de sécurité améliorée

**Type :** Fonctionnalité modifiée **Catégorie de service :**  Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Le 25 septembre 2019, nous allons désactiver l’ancienne expérience d’informations de sécurité non améliorée pour l’inscription et la gestion des informations de sécurité utilisateur. Seule la [nouvelle version améliorée](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Cool-enhancements-to-the-Azure-AD-combined-MFA-and-password/ba-p/354271) sera disponible. Cela signifie que vos utilisateurs ne seront plus en mesure d’utiliser l’ancienne expérience.

Pour en savoir plus sur l’expérience d’informations de sécurité renforcée, consultez notre [documentation administrateur](../authentication/concept-registration-mfa-sspr-combined.md) et notre [documentation utilisateur](../user-help/security-info-setup-signin.md).

#### <a name="to-turn-on-this-new-experience-you-must"></a>Pour activer cette nouvelle expérience, vous devez :

1. Connectez-vous au portail Azure en tant qu’Administrateur général ou Administrateur d’utilisateurs.

2. Accédez à **Azure Active Directory > Paramètres utilisateur > Gérer les paramètres pour les fonctionnalités en préversion du volet d’accès**.

3. Dans la zone **Les utilisateurs peuvent utiliser les fonctionnalités en préversion pour inscrire et gérer les informations de sécurité - amélioré**, choisissez **Sélectionné**, puis un groupe d’utilisateurs ou **Tout** pour activer cette fonctionnalité pour tous les utilisateurs du locataire.

4. Sous la zone **Users can use preview features for registering and managing security **info**,**(les utilisateurs peuvent utiliser les fonctionnalités en préversion pour inscrire et gérer les informations de sécurité), sélectionnez **None** (Aucun).

5. Enregistrez vos paramètres.

    Une fois vos paramètres enregistrés, vous n’avez plus accès à l’ancienne expérience d’informations de sécurité.

>[!Important]
>Si vous n’effectuez pas ces étapes avant le 25 septembre 2019, votre client Azure Active Directory passera automatiquement à l’expérience améliorée. En cas de questions, veuillez nous contacter à l’adresse registrationpreview@microsoft.com.

---

### <a name="authentication-requests-using-post-logins-will-be-more-strictly-validated"></a>Les demandes d’authentification utilisant des connexions POST seront plus rigoureusement validées

**Type :** Fonctionnalité modifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Standards

À partir du 2 septembre 2019, les demandes d’authentification à l’aide de la méthode POST seront plus rigoureusement validées par rapport aux normes HTTP. Plus précisément, les espaces et les guillemets doubles (") ne seront plus supprimés des valeurs du formulaire de demande. Ces modifications ne devraient pas bloquer les clients existants et permettront de s’assurer que les demandes envoyées à Azure AD sont gérées de manière fiable à chaque fois.

Pour plus d’informations, consultez les [avis relatifs aux changements cassants d’Azure AD](../develop/reference-breaking-changes.md#post-form-semantics-will-be-enforced-more-strictly---spaces-and-quotes-will-be-ignored).

---

## <a name="july-2019"></a>Juillet 2019

### <a name="plan-for-change-application-proxy-service-update-to-support-only-tls-12"></a>Planifier la modification : Mise à jour du service proxy d’application pour prendre en charge uniquement le protocole TLS 1.2

**Type :** Modification planifiée **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Pour vous aider à bénéficier de notre chiffrement le plus puissant, nous allons commencer à restreindre l’accès au service proxy d’application aux protocoles TLS 1.2. Cette limitation sera initialement déployée vers les clients qui utilisent déjà les protocoles TLS 1.2. Elle sera donc sans incidence sur vous. L’abandon complet des protocoles TLS 1.0 et TLS 1.1 sera terminé le 31 août 2019. Les clients qui utilisent toujours TLS 1.0 et TLS 1.1 recevront un préavis pour pouvoir se préparer à ce changement.

Pour maintenir la connexion au service proxy d’application tout au long de cette modification, nous vous recommandons de vous assurer que vos combinaisons client-serveur et navigateur-serveur sont à jour pour utiliser TLS 1.2. Nous vous recommandons également de veiller à inclure tous les systèmes client utilisés par vos employés pour accéder aux applications publiées via la service proxy d’application.

Pour plus d’informations, consultez [Ajouter une application locale pour un accès à distance via le service Proxy d’application d’Azure Active Directory](../manage-apps/application-proxy-add-on-premises-application.md).

---

### <a name="plan-for-change-design-updates-are-coming-for-the-application-gallery"></a>Planifier la modification : Mises à jour de conception prochainement disponibles pour la Galerie d’applications

**Type :** Modification planifiée **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

De nouvelles modifications d’interface utilisateur vont être apportées à la conception de la zone **Ajouter à partir de la galerie** du panneau **Ajouter une application**. Ces modifications vous aideront à trouver plus facilement vos applications qui prennent en charge l’approvisionnement automatique, OpenID Connect, Security Assertion Markup Language (SAML) et l’authentification unique (SSO) par mot de passe.

---

### <a name="plan-for-change-removal-of-the-mfa-server-ip-address-from-the-office-365-ip-address"></a>Planifier la modification : Suppression de l’adresse IP de serveur MFA de l’adresse IP Office 365

**Type :** Modification planifiée **Catégorie de service :** MFA **Fonctionnalité produit :** Protection et sécurité des identités

Nous supprimons l’adresse IP de serveur MFA du [service web d’adresse IP et d’URL Office 365](/office365/enterprise/office-365-ip-web-service). Si vous utilisez actuellement ces pages pour mettre à jour les paramètres de votre pare-feu, vous devez également veiller à inclure la liste des adresses IP évoquée dans la section **Configuration requise du serveur Azure Multi-Factor Authentication** de l’article [Prise en main du serveur Azure Multi-Factor Authentication](../authentication/howto-mfaserver-deploy.md#azure-multi-factor-authentication-server-firewall-requirements).

---

### <a name="app-only-tokens-now-require-the-client-app-to-exist-in-the-resource-tenant"></a>Les jetons réservés aux applications exigent à présent que l’application cliente existe dans le locataire de ressource

**Type :** Corrigé **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Le 26 juillet 2019, nous avons modifié la manière dont nous fournissons les jetons d’application uniquement par l’[octroi des informations d’identification du client](../azuread-dev/v1-oauth2-client-creds-grant-flow.md). Auparavant, les applications pouvaient recevoir des jetons pour appeler d’autres applications, que l’application client se trouve ou non dans le locataire. Nous avons mis à jour ce comportement afin que les ressources à locataire unique, parfois appelées API web, ne puissent être appelées que par des applications clientes existant dans le locataire de ressource.

Si votre application ne se trouve pas dans le locataire de ressource, vous recevez le message d’erreur suivant : `The service principal named <app_name> was not found in the tenant named <tenant_name>. This can happen if the application has not been installed by the administrator of the tenant.`. Pour résoudre ce problème, vous devez créer le principal du service de l’application cliente dans le locataire, à l’aide du [point de terminaison de consentement de l’administrateur](../develop/v2-permissions-and-consent.md#using-the-admin-consent-endpoint) ou via [PowerShell](../develop/howto-authenticate-service-principal-powershell.md), ce qui garantit que votre locataire a octroyé la permission d’application pour opérer dans le locataire.

Pour plus d’informations, voir [Quelles sont les nouveautés en matière d’authentification ?](../develop/reference-breaking-changes.md#app-only-tokens-for-single-tenant-applications-are-only-issued-if-the-client-app-exists-in-the-resource-tenant).

> [!NOTE]
> Un consentement existant entre le client et l’API n’est toujours pas requis. Les applications doivent toujours effectuer leurs propres vérifications d’autorisation.

---

### <a name="new-passwordless-sign-in-to-azure-ad-using-fido2-security-keys"></a>Nouvelle connexion sans mot de passe à Azure AD à l’aide de clés de sécurité FIDO2

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Les clients Azure AD peuvent désormais définir des stratégies pour gérer les clés de sécurité FIDO2 pour les utilisateurs et groupes de leur organisation. Les utilisateurs finaux peuvent également enregistrer eux-mêmes leurs clés de sécurité, les utiliser pour se connecter à leurs comptes Microsoft sur des sites web à l’aide d’appareils compatibles FIDO, ainsi que pour se connecter à leurs appareils Windows 10 associés à Azure AD.

Pour plus de détails, voir la section [Activer l’authentification sans mot de passe pour Azure AD (préversion)](../authentication/concept-authentication-passwordless.md) qui traite des informations relatives à l’administrateur, et la section [Configurer des informations de sécurité pour utiliser une clé de sécurité (préversion)](../user-help/security-info-setup-security-key.md) qui traite des informations relatives à l’utilisateur final.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---july-2019"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App – Juillet 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En juillet 2019, nous avons ajouté à notre galerie d’applications les 18 nouvelles applications suivantes qui prennent en charge la fédération :

[Ungerboeck Software](../saas-apps/ungerboeck-software-tutorial.md), [Bright Pattern Omnichannel Contact Center](../saas-apps/bright-pattern-omnichannel-contact-center-tutorial.md), [Clever Nelly](../saas-apps/clever-nelly-tutorial.md), [AcquireIO](../saas-apps/acquireio-tutorial.md), [Looop](https://www.looop.co/schedule-a-demo/), [productboard](../saas-apps/productboard-tutorial.md), [MS Azure SSO Access for Ethidex Compliance Office&trade;](../saas-apps/ms-azure-sso-access-for-ethidex-compliance-office-tutorial.md), [Hype](../saas-apps/hype-tutorial.md), [Abstract](../saas-apps/abstract-tutorial.md), [Ascentis](../saas-apps/ascentis-tutorial.md), [Flipsnack](https://www.flipsnack.com/accounts/sign-in-sso.html), [Wandera](../saas-apps/wandera-tutorial.md), [TwineSocial](https://twinesocial.com/), [Kallidus](../saas-apps/kallidus-tutorial.md), [HyperAnna](../saas-apps/hyperanna-tutorial.md), [PharmID WasteWitness](https://pharmid.com/), [i2B Connect](https://www.i2b-online.com/sign-up-to-use-i2b-connect-here-sso-access/), [JFrog Artifactory](../saas-apps/jfrog-artifactory-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatiser l’approvisionnement de compte d’utilisateur pour ces applications SaaS nouvellement prises en charge

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Monitoring et création de rapports

Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Dialpad](../saas-apps/dialpad-provisioning-tutorial.md)

- [Federated Directory](../saas-apps/federated-directory-provisioning-tutorial.md)

- [Figma](../saas-apps/figma-provisioning-tutorial.md)

- [Leapsome](../saas-apps/leapsome-provisioning-tutorial.md)

- [Peakon](../saas-apps/peakon-provisioning-tutorial.md)

- [Smartsheet](../saas-apps/smartsheet-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, consultez [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md)

---

### <a name="new-azure-ad-domain-services-service-tag-for-network-security-group"></a>Nouvelle balise de service Azure AD Domain Services pour groupe de sécurité réseau

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD Domain Services **Fonctionnalité produit :** Services de domaine Azure AD

Si vous en avez assez de gérer de longues listes d’adresses et de plages d’adresses IP, vous pouvez utiliser la nouvelle étiquette de service réseau **AzureActiveDirectoryDomainServices** dans votre groupe de sécurité réseau Azure pour sécuriser le trafic entrant vers votre sous-réseau de réseau virtuel Azure AD Domain Services.

Pour plus d’informations sur cette nouvelle étiquette de service, voir [Groupes de sécurité réseau pour Azure AD Domain Services](../../active-directory-domain-services/network-considerations.md#network-security-groups-and-required-ports).

---

### <a name="new-security-audits-for-azure-ad-domain-services-public-preview"></a>Nouveaux audits de sécurité pour Azure AD Domain Services (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD Domain Services **Fonctionnalité produit :** Services de domaine Azure AD

Nous sommes heureux d’annoncer le mise en production de l’audit de sécurité du service de domaine Azure AD en préversion publique. L’audit de sécurité vous aide à recueillir des informations essentielles sur vos services d’authentification en diffusant en continu les événements d’audit de sécurité vers des ressources ciblées, dont Stockage Azure, des espaces de travail Log Analytics et Azure Event Analytics, en utilisant le portail du service de domaine Azure AD.

Pour plus d’informations, voir [Activer les audits de sécurité pour Azure Active Directory Domain Services (préversion)](../../active-directory-domain-services/security-audit-events.md).

---

### <a name="new-authentication-methods-usage--insights-public-preview"></a>Utilisation et enseignements des nouvelles méthodes d’authentification (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Réinitialisation du mot de passe libre-service **Fonctionnalité produit :** Monitoring et création de rapports

Les nouveaux rapports sur l’utilisation et les enseignements des méthodes d’authentification peuvent vous aider à comprendre comment des fonctionnalités telles qu’Azure AD Multi-Factor Authentication et la réinitialisation de mot de passe en libre-service sont inscrites et utilisées dans votre organisation, en indiquant notamment le nombre d’utilisateurs inscrits à chaque fonctionnalité, la fréquence de réinitialisation du mot de passe en libre-service et la méthode utilisée pour la réinitialisation.

Pour plus d’informations, voir [Utilisation et enseignements des méthodes d’authentification (préversion)](../authentication/howto-authentication-methods-usage-insights.md).

---

### <a name="new-security-reports-are-available-for-all-azure-ad-administrators-public-preview"></a>Nouveaux rapports de sécurité disponibles pour tous les administrateurs Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Tous les administrateurs Azure AD peuvent désormais sélectionner la bannière en haut des rapports de sécurité existants, tels que le rapport **Utilisateurs avec indicateur de risque**, pour commencer à utiliser la nouvelle expérience de sécurité, comme indiqué dans les rapports **Utilisateurs à risque** et **Connexions à risque**. Au fil du temps, tous les rapports de sécurité seront migrés des anciennes versions vers les nouvelles versions, les nouveaux rapports offrant les fonctionnalités supplémentaires suivantes :

- Filtrage et tri avancés

- Actions en bloc, telles que le rejet de risque utilisateur

- Confirmation d’entités compromises ou sûres

- État du risque, à savoir : À risque, Ignoré, Corrigé ou Compromission confirmée.

Pour plus d’informations, consultez [rapport Utilisateurs à risque](../identity-protection/howto-identity-protection-investigate-risk.md#risky-users) et [rapport Connexions à risque](../identity-protection/howto-identity-protection-investigate-risk.md#risky-sign-ins).

---

### <a name="new-security-audits-for-azure-ad-domain-services-public-preview"></a>Nouveaux audits de sécurité pour Azure AD Domain Services (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD Domain Services **Fonctionnalité produit :** Services de domaine Azure AD

Nous sommes heureux d’annoncer le mise en production de l’audit de sécurité du service de domaine Azure AD en préversion publique. L’audit de sécurité vous aide à recueillir des informations essentielles sur vos services d’authentification en diffusant en continu les événements d’audit de sécurité vers des ressources ciblées, dont Stockage Azure, des espaces de travail Log Analytics et Azure Event Analytics, en utilisant le portail du service de domaine Azure AD.

Pour plus d’informations, voir [Activer les audits de sécurité pour Azure Active Directory Domain Services (préversion)](../../active-directory-domain-services/security-audit-events.md).

---

### <a name="new-b2b-direct-federation-using-samlws-fed-public-preview"></a>Nouvelle fédération directe B2B utilisant SAML/WS-FED (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2B **Fonctionnalité produit :** B2B/B2C

Une fédération directe vous aide à travailler avec des partenaires dont la solution d’identité gérée par l’informatique n’est pas Azure AD, en utilisant des systèmes d’identité qui prennent en charge les normes SAML ou WS-FED. Après que vous avez défini une relation de fédération directe avec un partenaire, tout nouvel utilisateur de ce domaine que vous invitez peut collaborer avec vous à l’aide de son compte organisationnel existant, ce qui contribue à rendre l’expérience utilisateur de vos invités plus transparente.

Pour plus d’informations, voir [Fédération directe avec AD FS et des fournisseurs tiers pour les utilisateurs invités (préversion)](../external-identities/direct-federation.md).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatiser l’approvisionnement de compte d’utilisateur pour ces applications SaaS nouvellement prises en charge

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Monitoring et création de rapports

Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Dialpad](../saas-apps/dialpad-provisioning-tutorial.md)

- [Federated Directory](../saas-apps/federated-directory-provisioning-tutorial.md)

- [Figma](../saas-apps/figma-provisioning-tutorial.md)

- [Leapsome](../saas-apps/leapsome-provisioning-tutorial.md)

- [Peakon](../saas-apps/peakon-provisioning-tutorial.md)

- [Smartsheet](../saas-apps/smartsheet-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, voir [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="new-check-for-duplicate-group-names-in-the-azure-ad-portal"></a>Nouvelle vérification des noms de groupes en double dans le portail Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

À présent, lorsque vous créez ou mettez à jour un nom de groupe à partir du portail Azure AD, nous vérifions que vous ne dupliquez pas un nom de groupe existant dans votre ressource. Si nous déterminons que le nom est déjà utilisé pour un autre groupe, vous êtes invité à modifier votre nom.

Pour plus d’informations, voir [Gérer des groupes dans le portail Azure AD](./active-directory-groups-create-azure-portal.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context).

---

### <a name="azure-ad-now-supports-static-query-parameters-in-reply-redirect-uris"></a>Azure AD prend désormais en charge les paramètres de requête statique dans les URI de réponse (redirection)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Les applications Azure AD peuvent désormais inscrire et utiliser des URI de réponse (redirection) avec des paramètres de requête statique (par exemple, `https://contoso.com/oauth2?idp=microsoft`) pour les demandes OAuth 2.0. Le paramètre de requête statique est soumis à la correspondance des chaînes pour les URI de réponse, comme n’importe quelle autre partie de l’URI de réponse. Si aucune chaîne inscrite ne correspond à l’URI de redirection décodée par URL, la demande est rejetée. Si l’URI de réponse est trouvé, la chaîne entière est utilisée pour rediriger l’utilisateur, y compris le paramètre de requête statique.

Les URI de réponse dynamique restent interdits parce qu’ils constituent un risque pour la sécurité, et ne peuvent pas être utilisé pour conserver des informations d’état dans une demande d’authentification. Pour cela, utilisez le paramètre `state`.

Actuellement, les écrans d’inscription d’application du portail Azure bloquent toujours les paramètres de requête. Toutefois, vous pouvez modifier manuellement le manifeste de l’application pour ajouter et tester des paramètres de requête de test dans votre application. Pour plus d’informations, voir [Quelles sont les nouveautés en matière d’authentification ?](../develop/reference-breaking-changes.md#redirect-uris-can-now-contain-query-string-parameters).

---

### <a name="activity-logs-ms-graph-apis-for-azure-ad-are-now-available-through-powershell-cmdlets"></a>Journaux d’activité (API MS Graph) pour Azure AD désormais disponibles via des cmdlets PowerShell

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous sommes heureux d’annoncer que des journaux d’activité Azure AD (rapports d’audit et de connexion) sont désormais disponibles via le module Azure AD PowerShell. Auparavant, vous pouviez créer vos propres scripts à l’aide de points de terminaison de l’API MS Graph, et nous avons désormais étendu cette fonctionnalité aux cmdlets PowerShell.

Pour plus d’informations sur l’utilisation de ces cmdlets, voir [Cmdlets Azure AD PowerShell pour la création de rapports.](../reports-monitoring/reference-powershell-reporting.md)

---

### <a name="updated-filter-controls-for-audit-and-sign-in-logs-in-azure-ad"></a>Contrôles de filtre mis à jour pour les journaux d’audit et de connexion dans Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous avons mis à jour les rapports de journal d’audit et de connexion afin que vous puissiez appliquer différents filtres sans devoir les ajouter en tant que colonnes sur les écrans de rapport. En outre, vous pouvez maintenant décider du nombre de filtres que vous souhaitez afficher à l’écran. Ces mises à jour interagissent pour faciliter la lecture des rapports et adapter leur étendue à vos besoins.

Pour plus d’informations sur ces mises à jour, voir [Filtrer les journaux d’audit](../reports-monitoring/concept-audit-logs.md#filtering-audit-logs) et [Filtrer les activités de connexion](../reports-monitoring/concept-sign-ins.md#filter-sign-in-activities).

---

## <a name="june-2019"></a>Juin 2019

### <a name="new-riskdetections-api-for-microsoft-graph-public-preview"></a>Nouvelle API riskDetections pour Microsoft Graph (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Nous sommes heureux d’annoncer que la nouvelle API riskDetections pour Microsoft Graph est désormais disponible en préversion publique. Vous pouvez utiliser cette nouvelle API pour afficher une liste des détections de risque au niveau des utilisateurs et des connexions dans le cadre du dispositif de protection des identités de votre organisation. Vous pouvez également utiliser cette API pour interroger plus efficacement vos détections de risque, avec notamment des détails sur le type de détection, son état, son niveau et bien plus encore.

Pour plus d’informations, voir [Documentation de référence sur l’API de détection des risques](/graph/api/resources/riskdetection?view=graph-rest-beta).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---june-2019"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App - Juin 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En juin 2019, nous avons ajouté à notre galerie d’applications les 22 applications suivantes qui prennent en charge la fédération :

[Azure AD SAML Toolkit](../saas-apps/saml-toolkit-tutorial.md), [Otsuka Shokai (大塚商会)](../saas-apps/otsuka-shokai-tutorial.md), [ANAQUA](../saas-apps/anaqua-tutorial.md), [Azure VPN Client](https://portal.azure.com/), [ExpenseIn](../saas-apps/expensein-tutorial.md), [Helper Helper](../saas-apps/helper-helper-tutorial.md), [Costpoint](../saas-apps/costpoint-tutorial.md), [GlobalOne](../saas-apps/globalone-tutorial.md), [Mercedes-Benz In-Car Office](https://me.secure.mercedes-benz.com/), [Skore](https://app.justskore.it/), [Oracle Cloud Infrastructure Console](../saas-apps/oracle-cloud-tutorial.md), [CyberArk SAML Authentication](../saas-apps/cyberark-saml-authentication-tutorial.md), [Scrible Edu](https://www.scrible.com/sign-in/#/create-account), [PandaDoc](../saas-apps/pandadoc-tutorial.md), [Perceptyx](https://apexdata.azurewebsites.net/docs.microsoft.com/azure/active-directory/saas-apps/perceptyx-tutorial), [Proptimise OS](https://proptimise.co.uk/software/), [Vtiger CRM (SAML)](../saas-apps/vtiger-crm-saml-tutorial.md), Oracle Access Manager for Oracle Retail Merchandising, Oracle Access Manager for Oracle E-Business Suite, Oracle IDCS for E-Business Suite, Oracle IDCS for PeopleSoft, Oracle IDCS for JD Edwards

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="automate-user-account-provisioning-for-these-newly-supported-saas-apps"></a>Automatiser l’approvisionnement de compte d’utilisateur pour ces applications SaaS nouvellement prises en charge

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Monitoring et création de rapports

Vous pouvez désormais automatiser la création, la mise à jour et la suppression de comptes d’utilisateur pour ces applications nouvellement intégrées :

- [Zoom](../saas-apps/zoom-provisioning-tutorial.md)

- [Envoy](../saas-apps/envoy-provisioning-tutorial.md)

- [Proxyclick](../saas-apps/proxyclick-provisioning-tutorial.md)

- [4me](../saas-apps/4me-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, consultez [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md)

---

### <a name="view-the-real-time-progress-of-the-azure-ad-provisioning-service"></a>Afficher la progression en temps réel du service d’approvisionnement Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Gestion du cycle de vie des identités

Nous avons mis à jour le mécanisme d’approvisionnement d’Azure AD et y avons inclus une nouvelle barre de progression indiquant l’avancement du processus d’approvisionnement des utilisateurs. Cette mise à jour fournit désormais également des informations sur le nombre d’utilisateurs approvisionnés au cours du cycle actuel et à ce jour.

Pour plus d’informations, consultez [Vérifier l’état de l’approvisionnement d’utilisateurs](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md).

---

### <a name="company-branding-now-appears-on-sign-out-and-error-screens"></a>La marque de société s’affiche désormais sur les écrans de déconnexion et d’erreur

**Type :** Fonctionnalité modifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Nous avons mis à jour Azure AD afin que la marque de votre société s’affiche sur les écrans de déconnexion et d’erreur, ainsi que sur la page de connexion. Vous n’avez rien à faire pour activer cette fonctionnalité. Azure AD utilise simplement les ressources que vous avez déjà configurées dans la section **Marque de société** du Portail Azure.

Pour plus d’informations sur la configuration de la marque de votre société, consultez [Personnaliser les pages Azure Active Directory de votre organisation](./customize-branding.md).

---

### <a name="azure-multi-factor-authentication-mfa-server-is-no-longer-available-for-new-deployments"></a>Le serveur d’authentification multifacteur (MFA) Azure n’est plus disponible pour les nouveaux déploiements

**Type :** Déprécié **Catégorie de service :** MFA **Fonctionnalité produit :** Protection et sécurité des identités

À compter du 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui veulent une authentification multifacteur dans leur organisation doivent utiliser le service informatique Azure AD Multi-Factor Authentication. Les clients ayant activé le serveur MFA avant le 1er juillet ne verront aucune modification. Vous serez toujours en mesure de télécharger la version la plus récente, d’obtenir les mises à jour ultérieures et de générer des informations d’identification d’activation.

Pour plus d’informations, consultez [Prise en main du serveur Azure Multi-Factor Authentication](../authentication/howto-mfaserver-deploy.md). Pour plus d’informations sur le service informatique Azure AD Multi-Factor Authentication, consultez [Planifier un déploiement informatique d’Azure AD Multi-Factor Authentication](../authentication/howto-mfa-getstarted.md).

---

## <a name="may-2019"></a>Mai 2019

### <a name="service-change-future-support-for-only-tls-12-protocols-on-the-application-proxy-service"></a>Modification du service : Future prise en charge uniquement pour les protocoles TLS 1.2 sur le service Proxy d’application

**Type :** Modification planifiée **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Pour fournir un chiffrement de qualité à nos clients, nous autorisons l’accès uniquement aux protocoles TLS 1.2 sur le service Proxy d’application. Cette modification est progressivement déployée chez les clients utilisant déjà exclusivement des protocoles TLS 1.2, vous ne devriez donc voir aucun changement.

L’abandon des protocoles TLS 1.0 et TLS 1.1 se produira sur le 31 août 2019, mais nous vous avertirons de nouveau afin que vous ayez le temps de vous y préparer. Pour vous préparer à cette modification, vérifiez que vos combinaisons client-serveur et navigateur-serveur, y compris les clients dont vos utilisateurs e servent pour accéder aux applications publiées via le Proxy d’application, sont mises à jour pour utiliser le protocole TLS 1.2 pour maintenir la connexion au service Proxy d’application. Pour plus d’informations, consultez [Ajouter une application locale pour un accès à distance via le service Proxy d’application d’Azure Active Directory](../manage-apps/application-proxy-add-on-premises-application.md#prerequisites).

---

### <a name="use-the-usage-and-insights-report-to-view-your-app-related-sign-in-data"></a>Utiliser le rapport d’utilisation et d’insights pour afficher les données de connexion liées aux applications

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Monitoring et création de rapports

Vous pouvez maintenant utiliser le rapport d’utilisation et d’insights, situé dans la section **Applications d’entreprise** du Portail Azure, pour obtenir une vue orientée application de vos données de connexion, notamment des informations sur :

- Les applications principalement utilisées dans votre organisation

- Les applications présentant le plus grand nombre d’échecs de connexion

- Les erreurs de connexion principales pour chaque application

Pour plus d’informations sur cette fonctionnalité, consultez [Rapport d’utilisation et d’insights dans le Portail Azure Active Directory](../reports-monitoring/concept-usage-insights-report.md).

---

### <a name="automate-your-user-provisioning-to-cloud-apps-using-azure-ad"></a>Automatiser l’approvisionnement de vos utilisateurs dans les applications cloud avec Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Monitoring et création de rapports

Suivez ces nouveaux didacticiels pour utiliser le service d’approvisionnement Azure AD pour automatiser la création, la suppression et la mise à jour des comptes utilisateur pour les applications cloud suivantes :

- [Comeet](../saas-apps/comeet-recruiting-software-provisioning-tutorial.md)

- [DynamicSignal](../saas-apps/dynamic-signal-provisioning-tutorial.md)

- [KeeperSecurity](../saas-apps/keeper-password-manager-digitalvault-provisioning-tutorial.md)

Vous pouvez également suivre ce nouveau [didacticiel Dropbox](../saas-apps/dropboxforbusiness-provisioning-tutorial.md), qui fournit des informations sur l’approvisionnement d’objets de groupe.

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, consultez [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="identity-secure-score-is-now-available-in-azure-ad-general-availability"></a>Identity Secure Score est désormais disponible dans Azure AD (disponibilité générale)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Non applicable **Fonctionnalité produit :** Protection et sécurité des identités

Vous pouvez désormais surveiller et améliorer votre stratégie de sécurité relative aux identités avec la fonctionnalité Identity Secure Score d’Azure AD. Cette fonctionnalité utilise un tableau de bord unique qui offre les avantages suivants :

- Évaluer votre méthode de sécurité relative aux identités de façon objective, avec un score compris entre 1 et 223

- Planifier les améliorations à apporter à la sécurité des identités

- Évaluer la réussite de vos améliorations en matière de sécurité

Pour plus d’informations sur la fonctionnalité Identity Security Score, consultez [Qu’est-ce que le degré de sécurisation Identity Secure Score dans Azure Active Directory ?](./identity-secure-score.md).

---

### <a name="new-app-registrations-experience-is-now-available-general-availability"></a>Nouvelle expérience d’inscription des applications désormais disponible (disponibilité générale)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Expérience de développement

La nouvelle expérience de la fonction [Inscriptions d’applications](https://aka.ms/appregistrations) est désormais en disponibilité générale. Cette nouvelle expérience inclut toutes les fonctionnalités clés du Portail Azure et du portail d’inscription des applications que vous connaissez déjà, et les améliore :

- **Meilleure gestion des applications.** Au lieu de voir vos applications sur des portails différents, vous pouvez maintenant voir toutes vos applications au même emplacement.

- **Inscription simplifiée des applications.** Avec une meilleure expérience de navigation et une sélection des autorisations repensée, il est désormais plus facile d’inscrire et de gérer vos applications.

- **Informations plus détaillées.** Vous trouverez des informations encore plus détaillées sur votre application, notamment des guides de démarrage rapide et bien plus encore.

Pour plus d’informations, consultez [Plateforme d’identités Microsoft](../develop/index.yml) et le billet de blog annonçant qu’une [nouvelle expérience d’inscription des applications est désormais disponible](https://developer.microsoft.com/identity/blogs/new-app-registrations-experience-is-now-generally-available/). (annonce de blog).

---

### <a name="new-capabilities-available-in-the-risky-users-api-for-identity-protection"></a>Nouvelles fonctionnalités disponibles dans l’API Risky Users pour la protection des identités

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Nous sommes heureux d’annoncer que vous pouvez maintenant utiliser l’API Risky Users pour récupérer l’historique des risques liés aux utilisateurs, ignorer les utilisateurs à risque et confirmer l’état de compromission des utilisateurs. Cette modification vous permet de mettre à jour plus efficacement l’état des risques liés à vos utilisateurs et de comprendre l’historique des risques associé.

Pour plus d’informations, consultez la [documentation de référence sur l’API Risky Users](/graph/api/resources/riskyuser?view=graph-rest-beta).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---may-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Mai 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En mai 2019, nous avons ajouté à notre galerie d’applications les 21 applications suivantes qui prennent en charge la fédération :

[Freedcamp](../saas-apps/freedcamp-tutorial.md), [Real Links](../saas-apps/real-links-tutorial.md), [Kianda](https://app.kianda.com/sso/OpenID/AzureAD/), [Simple Sign](../saas-apps/simple-sign-tutorial.md), [Braze](../saas-apps/braze-tutorial.md), [Displayr](../saas-apps/displayr-tutorial.md), [Templafy](../saas-apps/templafy-tutorial.md), [Marketo Sales Engage](https://toutapp.com/login), [ACLP](../saas-apps/aclp-tutorial.md), [OutSystems](../saas-apps/outsystems-tutorial.md), [Meta4 Global HR](../saas-apps/meta4-global-hr-tutorial.md), [Quantum Workplace](../saas-apps/quantum-workplace-tutorial.md), [Cobalt](../saas-apps/cobalt-tutorial.md), [webMethods API Cloud](../saas-apps/webmethods-integration-cloud-tutorial.md), [RedFlag](https://pocketstop.com/redflag/), [Whatfix](../saas-apps/whatfix-tutorial.md), [Control](../saas-apps/control-tutorial.md), [JOBHUB](../saas-apps/jobhub-tutorial.md), [NEOGOV](../saas-apps/neogov-tutorial.md), [Foodee](../saas-apps/foodee-tutorial.md), [MyVR](../saas-apps/myvr-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="improved-groups-creation-and-management-experiences-in-the-azure-ad-portal"></a>Expériences de création et de gestion des groupes améliorée dans le Portail Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Nous avons apporté des améliorations concernant la gestion des groupes dans le Portail Azure AD. Ces améliorations permettent aux administrateurs de mieux gérer les listes de groupes, les listes de membres et de fournir des options de création supplémentaires.

Les améliorations incluent :

- Filtrage de base par type d’appartenance et par type de groupe.

- Ajout de nouvelles colonnes, telles que de la source et l’adresse de messagerie.

- Possibilité de sélectionner plusieurs listes de groupes, membres et propriétaires pour faciliter la suppression.

- Possibilité de choisir une adresse de messagerie et d’ajouter des propriétaires lors de la création de groupe.

Pour plus d’informations, consultez [Créer un groupe de base et ajouter des membres avec Azure Active Directory](./active-directory-groups-create-azure-portal.md).

---

### <a name="configure-a-naming-policy-for-office-365-groups-in-azure-ad-portal-general-availability"></a>Configurer une stratégie d’affectation de noms pour les groupes Office 365 dans le Portail Azure AD (disponibilité générale)

**Type :** Fonctionnalité modifiée **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Les administrateurs peuvent désormais configurer une stratégie d’affectation de noms pour les groupes Office 365, depuis le Portail Azure AD. Cette modification permet d’appliquer des conventions d’affectation de noms cohérentes aux groupes Office 365 créés ou modifiés par les utilisateurs de votre organisation.

Vous pouvez appliquer une stratégie de nommage aux groupes Office 365 de deux manières différentes :

- Définir des préfixes ou des suffixes, qui sont automatiquement ajoutés à un nom de groupe.

- Charger un jeu personnalisé de mots bloqués pour votre organisation, qui ne sont pas autorisés dans les noms de groupe (par exemple « PDG, paie, RH »).

Pour plus d’informations, consultez [Appliquer une stratégie de nommage pour les groupes Office 365 dans Azure Active Directory](../enterprise-users/groups-naming-policy.md).

---

### <a name="microsoft-graph-api-endpoints-are-now-available-for-azure-ad-activity-logs-general-availability"></a>Les points de terminaison d’API Microsoft Graph sont désormais disponibles pour les journaux d’activité Azure AD (disponibilité générale)

**Type :** Fonctionnalité modifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous sommes heureux d’annoncer la disponibilité générale de la prise en charge des points de terminaison d’API Microsoft Graph pour les journaux d’activité Azure AD. Avec cette version, vous pouvez maintenant utiliser la version 1.0 des API des journaux d’audit et des journaux de connexion Azure AD.

Pour plus d’informations, consultez la [vue d’ensemble de l’API des journaux d’audit Azure AD](/graph/api/resources/azure-ad-auditlog-overview).

---

### <a name="administrators-can-now-use-conditional-access-for-the-combined-registration-process-public-preview"></a>Les administrateurs peuvent désormais utiliser l’accès conditionnel pour le processus d’inscription combinée (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Les administrateurs peuvent désormais créer des stratégies d’accès conditionnel pour une utilisation dans la page d’inscription combinée. Cela inclut l’application de stratégies pour autoriser l’inscription si :

- Les utilisateurs se trouvent sur un réseau approuvé.

- Les utilisateurs ont un niveau faible de risque à la connexion.

- Les utilisateurs utilisent un appareil géré.

- Les utilisateurs acceptent les conditions d’utilisation de l’organisation.

Pour plus d’informations sur l’accès conditionnel et la réinitialisation de mot de passe, consultez le billet de blog portant sur [l’accès conditionnel pour l’expérience d’inscription combinée Azure AD avec authentification multifacteur et réinitialisation du mot de passe](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Conditional-access-for-the-Azure-AD-combined-MFA-and-password/ba-p/566348). Pour plus d’informations sur les stratégies d’accès conditionnel pour le processus d’inscription combinée, consultez [Stratégies d’accès conditionnel pour l’inscription combinée](../authentication/howto-registration-mfa-sspr-combined.md#conditional-access-policies-for-combined-registration). Pour plus d’informations sur les conditions d’utilisation d’Azure AD, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md).

---

## <a name="april-2019"></a>Avril 2019

### <a name="new-azure-ad-threat-intelligence-detection-is-now-available-as-part-of-azure-ad-identity-protection"></a>Une nouvelle fonction de détection des informations sur les menaces Azure AD est désormais disponible dans Azure AD Identity Protection

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD Identity Protection **Fonctionnalité produit :** Protection et sécurité des identités

Une fonction de détection des informations sur les menaces Azure AD est désormais intégrée à la fonctionnalité mise à jour Azure AD Identity Protection. Cette nouvelle fonction permet de détecter une activité utilisateur inhabituelle pour un utilisateur ou une activité spécifique qui correspond à des modèles d’attaque connus selon les sources d’information sur les menaces internes et externes de Microsoft.

Pour plus d’informations sur la version actualisée d’Azure AD Identity Protection, consultez le billet de blog concernant les [quatre principales améliorations apportées à Azure AD Identity Protection désormais disponibles en préversion publique](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Four-major-Azure-AD-Identity-Protection-enhancements-are-now-in/ba-p/326935) et l’article [Présentation d’Azure Active Directory Identity Protection (actualisée)](../identity-protection/overview-identity-protection.md). (article). Pour plus d’informations sur la détection d’informations sur les menaces Azure AD, consultez l’article [Détections de risques Azure Active Directory Identity Protection](../identity-protection/concept-identity-protection-risks.md).

---

### <a name="azure-ad-entitlement-management-is-now-available-public-preview"></a>La gestion des droits d’utilisation Azure AD est désormais disponible (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gouvernance des identités **Fonctionnalité produit :** Gouvernance des identités

La gestion des droits d’utilisation Azure AD, désormais en préversion publique, permet aux clients de déléguer la gestion des packages d’accès. La façon dont les employés et les partenaires commerciaux peuvent demander un accès, qui doit l’approuver, et la durée de validité de ce dernier sont déterminés par ce biais. Les packages d’accès permettent de gérer l’appartenance aux groupes Azure AD et Office 365, les affectations de rôles dans les applications d’entreprise et les affectations de rôles pour les sites SharePoint Online. Pour en savoir plus sur la gestion des droits d’utilisation, consultez la [vue d’ensemble de la gestion des droits d’utilisation Azure AD](../governance/entitlement-management-overview.md). Pour en savoir plus sur l’éventail des fonctionnalités d’Azure AD Identity Governance, y compris la fonction Privileged Identity Management, les révisions d’accès et les conditions d’utilisation, consultez [Qu’est-ce qu’Azure AD Identity Governance ?](../governance/identity-governance-overview.md)

---

### <a name="configure-a-naming-policy-for-office-365-groups-in-azure-ad-portal-public-preview"></a>Configurer une stratégie d’affectation de noms pour les groupes Office 365 dans le Portail Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Les administrateurs peuvent désormais configurer une stratégie d’affectation de noms pour les groupes Office 365, depuis le Portail Azure AD. Cette modification permet d’appliquer des conventions d’affectation de noms cohérentes aux groupes Office 365 créés ou modifiés par les utilisateurs de votre organisation.

Vous pouvez appliquer une stratégie de nommage aux groupes Office 365 de deux manières différentes :

- Définir des préfixes ou des suffixes, qui sont automatiquement ajoutés à un nom de groupe.

- Charger un jeu personnalisé de mots bloqués pour votre organisation, qui ne sont pas autorisés dans les noms de groupe (par exemple « PDG, paie, RH »).

Pour plus d’informations, consultez [Appliquer une stratégie de nommage pour les groupes Office 365 dans Azure Active Directory](../enterprise-users/groups-naming-policy.md).

---

### <a name="azure-ad-activity-logs-are-now-available-in-azure-monitor-general-availability"></a>Les journaux d’activité Azure AD sont désormais disponibles dans Azure Monitor (disponibilité générale)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous avons tenu compte de vos retours concernant la visualisation des journaux d’activité Azure AD, et nous introduisons donc une nouvelle fonctionnalité Insights dans Log Analytics. Cette fonctionnalité vous permet d’obtenir des informations sur vos ressources Azure AD à l’aide de nos modèles interactifs, appelés classeurs. Ces classeurs préintégrés peuvent fournir des détails sur les applications ou les utilisateurs :

- **Connexions.** Fournit des détails sur les applications et les utilisateurs, y compris l’emplacement de connexion, le système d’exploitation ou le client et la version de navigateur en cours d’utilisation, ainsi que le nombre de connexions ayant réussi ou échoué.

- **Authentification héritée et accès conditionnel.** Fournit des détails sur les applications et les utilisateurs faisant appel à une authentification héritée, y compris l’authentification multifacteur déclenchée par les stratégies d’accès conditionnel, les applications utilisant des stratégies d’accès conditionnel, et ainsi de suite.

- **Analyse des échecs de connexion.** Vous aide à déterminer si vos erreurs de connexion sont causées par une action de l’utilisateur, des problèmes de stratégie ou votre infrastructure.

- **Rapports personnalisés.** Vous pouvez créer ou modifier des classeurs existants afin de personnaliser la fonctionnalité Insights pour votre organisation.

Pour plus d’informations, découvrez [comment utiliser des classeurs Azure Monitor pour créer des rapports Azure Active Directory](../reports-monitoring/howto-use-azure-monitor-workbooks.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---april-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Avril 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En avril 2019, nous avons ajouté à la galerie d’applications les 21 applications suivantes qui prennent en charge la fédération :

[SAP Fiori](../saas-apps/sap-fiori-tutorial.md), [HRworks Single Sign-On](../saas-apps/hrworks-single-sign-on-tutorial.md), [Percolate](../saas-apps/percolate-tutorial.md), [MobiControl](../saas-apps/mobicontrol-tutorial.md), [Citrix NetScaler](../saas-apps/citrix-netscaler-tutorial.md), [Shibumi](../saas-apps/shibumi-tutorial.md), [Benchling](../saas-apps/benchling-tutorial.md), [MileIQ](https://mileiq.onelink.me/991934284/7e980085), [PageDNA](../saas-apps/pagedna-tutorial.md), [EduBrite LMS](../saas-apps/edubrite-lms-tutorial.md), [RStudio Connect](../saas-apps/rstudio-connect-tutorial.md), [AMMS](../saas-apps/amms-tutorial.md), [Mitel Connect](../saas-apps/mitel-connect-tutorial.md), [Alibaba Cloud (Role-based SSO)](../saas-apps/alibaba-cloud-service-role-based-sso-tutorial.md), [Certent Equity Management](../saas-apps/certent-equity-management-tutorial.md), [Sectigo Certificate Manager](../saas-apps/sectigo-certificate-manager-tutorial.md), [GreenOrbit](../saas-apps/greenorbit-tutorial.md), [Workgrid](../saas-apps/workgrid-tutorial.md), [monday.com](../saas-apps/mondaycom-tutorial.md), [SurveyMonkey Enterprise](../saas-apps/surveymonkey-enterprise-tutorial.md), [Indiggo](https://indiggolead.com/)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-access-reviews-frequency-option-and-multiple-role-selection"></a>Nouvelle option de fréquence des révisions d’accès et sélection de plusieurs rôles

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Révisions d’accès **Fonctionnalité produit :** Gouvernance des identités

Les nouvelles mises à jour des révisions d’accès Azure AD vous permettent d’effectuer les actions suivantes :

- Modifier la fréquence de vos révisions d’accès et la définir sur une fréquence **semestrielle**, en plus des options existantes de fréquence hebdomadaire, mensuelle, trimestrielle et annuelle.

- Sélectionnez plusieurs rôles de ressource Azure et Azure AD lors de la création d’une révision d’accès unique. Dans ce cas, tous les rôles sont configurés avec les mêmes paramètres et tous les réviseurs sont informés en même temps.

Pour plus d’informations sur la création d’une révision d’accès, consultez [Créer une révision d’accès de groupes ou d’applications dans Azure AD](../governance/create-access-review.md).

---

### <a name="azure-ad-connect-email-alert-systems-are-transitioning-sending-new-email-sender-information-for-some-customers"></a>Les systèmes d’alerte par e-mail d’Azure AD Connect sont en transition, affichant un nouvel expéditeur pour certains clients

**Type :** Fonctionnalité modifiée **Catégorie de service :** AD Sync **Fonctionnalité produit :** Plateforme

Azure AD Connect est en passe de modifier ses systèmes d’alerte par e-mail, et certains clients peuvent potentiellement voir s’afficher un nouvel expéditeur. Pour résoudre ce problème, vous devez ajouter `azure-noreply@microsoft.com` à liste verte de votre organisation ou vous ne pourrez pas continuer à recevoir des alertes importantes d’Office 365, d’Azure ou de vos services de synchronisation.

---

### <a name="upn-suffix-changes-are-now-successful-between-federated-domains-in-azure-ad-connect"></a>Les modifications de suffixe UPN fonctionnent à présent entre domaines fédérés dans Azure AD Connect

**Type :** Corrigé **Catégorie de service :** AD Sync **Fonctionnalité produit :** Plateforme

Vous pouvez maintenant modifier avec succès le suffixe UPN d’un utilisateur d’un domaine fédéré vers un autre domaine dans Azure AD Connect. Avec ce correctif, vous ne devriez plus rencontrer le message d’erreur FederatedDomainChangeError pendant le cycle de synchronisation ni recevoir de notification électronique indiquant « Impossible de mettre à jour cet objet dans Azure Active Directory, car l’attribut [FederatedUser.UserPrincipalName] n’est pas valide. Mettez à jour la valeur dans vos services d’annuaire locaux ».

Pour plus d’informations, consultez [Résolution des erreurs lors de la synchronisation](../hybrid/tshoot-connect-sync-errors.md#federateddomainchangeerror).

---

### <a name="increased-security-using-the-app-protection-based-conditional-access-policy-in-azure-ad-public-preview"></a>Sécurité accrue avec la stratégie d’accès conditionnel basée sur la protection des applications dans Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

L’accès conditionnel basé sur la protection des applications est désormais disponible avec la stratégie **Exiger une protection des applications**. Cette nouvelle stratégie contribue à augmenter la sécurité de votre organisation en évitant les situations suivantes :

- Que les utilisateurs accèdent aux applications sans licence Microsoft Intune.

- Que les utilisateurs ne soient pas en mesure d’obtenir une stratégie de protection des applications Microsoft Intune.

- Que les utilisateurs accèdent à des applications sans stratégie de protection des applications Microsoft Intune configurée.

Pour plus d’informations, consultez [Comment exiger une stratégie de protection des applications pour l’accès conditionnel aux applications cloud](../conditional-access/app-protection-based-conditional-access.md).

---

### <a name="new-support-for-azure-ad-single-sign-on-and-conditional-access-in-microsoft-edge-public-preview"></a>Nouvelle prise en charge de l’authentification unique Azure AD et de l’accès conditionnel à Microsoft Edge (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Nous avons amélioré notre support Azure AD pour Microsoft Edge, notamment en fournissant une nouvelle prise en charge de l’accès conditionnel et de l’authentification unique Azure AD. Si vous avez déjà utilisé Microsoft Intune Managed Browser, vous pouvez désormais utiliser Microsoft Edge à la place.

Pour plus d’informations sur la configuration et la gestion de vos appareils et applications à l’aide de l’accès conditionnel, consultez [Demander des appareils managés pour l’accès conditionnel à des applications cloud](../conditional-access/require-managed-devices.md) et [Demander des applications clientes approuvées pour l’accès conditionnel à des applications cloud](../conditional-access/app-based-conditional-access.md). Pour plus d’informations sur la gestion de l’accès avec Microsoft Edge à l’aide de Microsoft Intune, consultez [Gérer l’accès à Internet à l’aide d’un navigateur Microsoft Intune protégé par une stratégie](/intune/app-configuration-managed-browser).

---

## <a name="march-2019"></a>Mars 2019

### <a name="identity-experience-framework-and-custom-policy-support-in-azure-active-directory-b2c-is-now-available-ga"></a>Identity Experience Framework et la prise en charge de stratégie personnalisée dans Azure Active Directory B2C sont désormais disponibles (GA)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Vous pouvez désormais créer des stratégies personnalisées dans Azure AD B2C, notamment les tâches suivantes, qui sont prises en charge à l’échelle et dans le cadre de notre Accord de niveau de service (SLA) Azure :

- Créer et charger des parcours utilisateur d’authentification personnalisés à l’aide de stratégies personnalisées.

- Décrire des parcours utilisateur étape par étape comme des échanges entre des fournisseurs de revendications.

- Définir le branchement conditionnel dans des parcours utilisateur.

- Transformer et mapper les revendications pour une utilisation dans les décisions et communications en temps réel.

- Utiliser des services compatibles avec l’API REST dans vos parcours utilisateur d’authentification personnalisés. Par exemple, avec les fournisseurs e-mail, les CRM et les systèmes d’autorisation propriétaires.

- Se fédérer avec des fournisseurs d’identité conformes au protocole OpenIDConnect. Par exemple, avec Azure AD multilocataire, les fournisseurs de comptes de réseaux sociaux ou les fournisseurs de vérification en deux étapes.

Pour plus d’informations sur la création de stratégies personnalisées, consultez [Notes pour les développeurs sur les stratégies personnalisées dans Azure Active Directory B2C](../../active-directory-b2c/custom-policy-developer-notes.md) et lisez le [billet de blog d’Alex Simon, notamment les études de cas](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-custom-policies-to-build-your-own-identity-journeys/ba-p/382791).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---march-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Mars 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En mars 2019, nous avons ajouté à notre galerie d’applications les 14 applications suivantes qui prennent en charge la fédération :

[ISEC7 Mobile Exchange Delegate](https://www.isec7.com/english/), [MediusFlow](https://office365.cloudapp.mediusflow.com/), [ePlatform](../saas-apps/eplatform-tutorial.md), [Fulcrum](../saas-apps/fulcrum-tutorial.md), [ExcelityGlobal](../saas-apps/excelityglobal-tutorial.md), [Explanation-Based Auditing System](../saas-apps/explanation-based-auditing-system-tutorial.md), [Lean](../saas-apps/lean-tutorial.md), [Powerschool Performance Matters](../saas-apps/powerschool-performance-matters-tutorial.md), [Cinode](https://cinode.com/), [Iris Intranet](../saas-apps/iris-intranet-tutorial.md), [Empactis](../saas-apps/empactis-tutorial.md), [SmartDraw](../saas-apps/smartdraw-tutorial.md), [Confirmit Horizons](../saas-apps/confirmit-horizons-tutorial.md), [TAS](../saas-apps/tas-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-zscaler-and-atlassian-provisioning-connectors-in-the-azure-ad-gallery---march-2019"></a>Nouveaux connecteurs d’approvisionnement Zscaler et Atlassian dans la galerie d’applications Azure AD - Mars 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Intégration tierce

Automatisez la création, la mise à jour et la suppression des comptes utilisateur pour les applications suivantes :

[Zscaler](../saas-apps/zscaler-provisioning-tutorial.md), [Zscaler Beta](../saas-apps/zscaler-beta-provisioning-tutorial.md), [Zscaler One](../saas-apps/zscaler-one-provisioning-tutorial.md), [Zscaler Two](../saas-apps/zscaler-two-provisioning-tutorial.md), [Zscaler Three](../saas-apps/zscaler-three-provisioning-tutorial.md), [Zscaler ZSCloud](../saas-apps/zscaler-zscloud-provisioning-tutorial.md), [Atlassian Cloud](../saas-apps/atlassian-cloud-provisioning-tutorial.md)

Pour découvrir comment sécuriser plus efficacement votre organisation à l’aide de l’approvisionnement automatique de comptes utilisateur, consultez [Automatisation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure AD](../app-provisioning/user-provisioning.md).

---

### <a name="restore-and-manage-your-deleted-office-365-groups-in-the-azure-ad-portal"></a>Restaurer et gérer vos groupes Office 365 supprimés dans le Portail Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Vous pouvez désormais afficher et gérer vos groupes Office 365 supprimés à partir du Portail Azure AD. Cette modification vous permet de voir quels groupes sont disponibles pour la restauration, ainsi que de supprimer définitivement tous les groupes qui ne sont pas nécessaires à votre organisation.

Pour plus d’informations, consultez [Restaurer les groupes expirés ou supprimés](../enterprise-users/groups-restore-deleted.md#view-and-manage-the-deleted-microsoft-365-groups-that-are-available-to-restore).

---

### <a name="single-sign-on-is-now-available-for-azure-ad-saml-secured-on-premises-apps-through-application-proxy-public-preview"></a>L’authentification unique est désormais disponible pour les applications locales sécurisées par SAML Azure AD via le proxy d’application (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Vous pouvez désormais fournir une expérience d’authentification unique (SSO) pour les applications locales authentifiées par SAML, ainsi que l’accès à distance à ces applications via le proxy d’application. Pour plus d’informations sur la configuration de l’authentification unique SAML avec vos applications locales, consultez [Authentification unique SAML pour les applications locales avec le proxy d’application (préversion)](../manage-apps/application-proxy-configure-single-sign-on-on-premises-apps.md).

---

### <a name="client-apps-in-request-loops-will-be-interrupted-to-improve-reliability-and-user-experience"></a>Les applications clientes dans des boucles de demande seront interrompues pour améliorer la fiabilité et l’expérience utilisateur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Les applications clientes peuvent émettre de manière incorrecte des centaines de demandes de connexion identiques sur une courte période. Ces demandes, qu’elles soient réussies ou non, contribuent à une mauvaise expérience utilisateur et à des charges de travail accrues pour l’IDP, augmentant la latence pour tous les utilisateurs et réduisant la disponibilité de l’IDP.

Cette mise à jour envoie une erreur `invalid_grant` : `AADSTS50196: The server terminated an operation because it encountered a loop while processing a request` aux applications clientes qui émettent des demandes dupliquées plusieurs fois pendant une courte période, au-delà de l’étendue de fonctionnement normal. Les applications clientes qui rencontrent ce problème doivent afficher une invite interactive demandant à l’utilisateur de se reconnecter. Pour plus d’informations sur cette modification et sur la façon de résoudre votre application si elle rencontre cette erreur, consultez [Nouveautés pour l’authentification](../develop/reference-breaking-changes.md#looping-clients-will-be-interrupted).

---

### <a name="new-audit-logs-user-experience-now-available"></a>Nouvelle expérience utilisateur Journaux d’audit désormais disponible

**Type :** Fonctionnalité modifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous avons créé une nouvelle page **Journaux d’audit** Azure AD pour aider à améliorer la lisibilité et la manière dont vous recherchez vos informations. Pour afficher la nouvelle page **Journaux d’audit**, sélectionnez **Journaux d’audit** dans la section **Activité** d’Azure AD.

![Nouvelle page Journaux d’audit, avec exemple d’informations](media/whats-new/audit-logs-page.png)

Pour plus d’informations sur la nouvelle page **Journaux d’audit**, consultez [Rapports d’activité d’audit dans le Portail Azure Active Directory](../reports-monitoring/concept-audit-logs.md#audit-logs).

---

### <a name="new-warnings-and-guidance-to-help-prevent-accidental-administrator-lockout-from-misconfigured-conditional-access-policies"></a>Nouveaux avertissements et conseils pour empêcher le verrouillage accidentel de l’administrateur à partir de stratégies d’accès conditionnel mal configurées

**Type :** Fonctionnalité modifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Pour éviter que les administrateurs ne bloquent leur accès à leurs propres locataires par le biais de stratégies d’accès conditionnel mal configurées, nous avons créé de nouveaux avertissements et des instructions mises à jour dans le Portail Azure. Pour plus d’informations sur les nouvelles instructions, consultez [Que sont les dépendances de service dans l’accès conditionnel Azure Active Directory](../conditional-access/service-dependencies.md).

---

### <a name="improved-end-user-terms-of-use-experiences-on-mobile-devices"></a>Expériences des conditions d’utilisation de l’utilisateur final améliorées sur les appareils mobiles

**Type :** Fonctionnalité modifiée **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Gouvernance

Nous avons mis à jour nos expériences de conditions d’utilisation existantes pour aider à améliorer la manière dont vous révisez et acceptez les conditions d’utilisation sur un appareil mobile. Vous pouvez désormais effectuer un zoom avant et arrière, revenir en arrière, télécharger les informations et sélectionner des liens hypertexte. Pour plus d’informations sur les conditions d’utilisation mises à jour, consultez [Fonctionnalité Conditions d’utilisation Azure Active Directory](../conditional-access/terms-of-use.md#what-terms-of-use-looks-like-for-users).

---

### <a name="new-azure-ad-activity-logs-download-experience-available"></a>Nouvelle expérience de téléchargement des journaux d’activité Azure AD disponible

**Type :** Fonctionnalité modifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Vous pouvez désormais télécharger de grandes quantités de journaux d’activité directement à partir du portail Azure. Cette mise à jour vous permet de :

- Télécharger jusqu’à 250 000 lignes.

- Recevoir une notification une fois le téléchargement effectué.

- Personnaliser votre nom de fichier.

- Déterminer votre format de sortie, JSON ou CSV.

Pour plus d’informations sur cette fonctionnalité, voir [Démarrage rapide : Télécharger un rapport d’audit à l’aide du Portail Azure](../reports-monitoring/quickstart-download-audit-report.md).

---

### <a name="breaking-change-updates-to-condition-evaluation-by-exchange-activesync-eas"></a>Changement cassant : mises à jour de l’évaluation de la condition par Exchange ActiveSync (EAS)

**Type :** Modification planifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Contrôle d’accès

Nous mettons à jour la manière dont Exchange ActiveSync (EAS) évalue les conditions suivantes :

- Emplacement de l’utilisateur, en fonction du pays, de la région ou de l’adresse IP

- Risque à la connexion

- Plateforme d’appareil.

Si vous avez déjà utilisé ces conditions dans vos stratégies d’accès conditionnel, sachez que le comportement de la condition peut changer. Par exemple, si vous avez précédemment utilisé la condition de l’emplacement de l’utilisateur dans une stratégie, la stratégie pourrait maintenant être ignorée en fonction de l’emplacement de votre utilisateur.

---

## <a name="february-2019"></a>Février 2019

### <a name="configurable-azure-ad-saml-token-encryption-public-preview"></a>Chiffrement de jeton configurable SAML Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Vous pouvez maintenant configurer toute application SAML prise en charge pour qu’elle reçoive des jetons SAML chiffrés. Lorsqu’il est configuré et utilisé avec une application, Azure AD chiffre les assertions SAML émises à l’aide d'une clé publique obtenue à partir d’un certificat stocké dans Azure AD.

Pour plus d’informations sur la configuration de votre chiffrement de jetons SAML, consultez [Configurer le chiffrement de jetons SAML Azure AD](../manage-apps/howto-saml-token-encryption.md).

---

### <a name="create-an-access-review-for-groups-or-apps-using-azure-ad-access-reviews"></a>Créer une révision d’accès pour les groupes ou les applications à l’aide des Révisions d’accès Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Révisions d’accès **Fonctionnalité produit :** Gouvernance

Vous pouvez maintenant inclure plusieurs groupes ou applications dans une seule révision d’accès Azure AD pour l’appartenance au groupe ou l’affectation d’application. Les révisions d’accès avec plusieurs groupes ou applications sont définies à l’aide des mêmes paramètres et tous les réviseurs inclus sont avertis en même temps.

Pour plus d’informations sur la création d’une révision d’accès à l’aide des Révisions d’accès Azure AD, consultez [Créer une révision d’accès de groupes ou d’applications dans les Révisions d’accès Azure AD](../governance/create-access-review.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---february-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Février 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En février 2019, nous avons ajouté à notre galerie d’applications les 27 applications suivantes qui prennent en charge la fédération :

[Euromonitor Passport](../saas-apps/euromonitor-passport-tutorial.md), [MindTickle](../saas-apps/mindtickle-tutorial.md), [FAT FINGER](https://seeforgetest-exxon.azurewebsites.net/Account/create?Length=7), [AirStack](../saas-apps/airstack-tutorial.md), [Oracle Fusion ERP](../saas-apps/oracle-fusion-erp-tutorial.md), [IDrive](../saas-apps/idrive-tutorial.md), [Skyward Qmlativ](../saas-apps/skyward-qmlativ-tutorial.md), [Brightidea](../saas-apps/brightidea-tutorial.md), [AlertOps](../saas-apps/alertops-tutorial.md), [Soloinsight-CloudGate SSO](../saas-apps/soloinsight-cloudgate-sso-tutorial.md), Permission Click, [Brandfolder](../saas-apps/brandfolder-tutorial.md), [StoregateSmartFile](../saas-apps/smartfile-tutorial.md), [Pexip](../saas-apps/pexip-tutorial.md), [Stormboard](../saas-apps/stormboard-tutorial.md), [Seismic](../saas-apps/seismic-tutorial.md), [Share A Dream](https://www.shareadream.org/how-it-works), [Bugsnag](../saas-apps/bugsnag-tutorial.md), [webMethods Integration Cloud](../saas-apps/webmethods-integration-cloud-tutorial.md), [Knowledge Anywhere LMS](../saas-apps/knowledge-anywhere-lms-tutorial.md), [OU Campus](../saas-apps/ou-campus-tutorial.md), [Periscope Data](../saas-apps/periscope-data-tutorial.md), [Netop Portal](../saas-apps/netop-portal-tutorial.md), [smartvid.io](../saas-apps/smartvid.io-tutorial.md), [PureCloud by Genesys](../saas-apps/purecloud-by-genesys-tutorial.md), [ClickUp Productivity Platform](../saas-apps/clickup-productivity-platform-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="enhanced-combined-mfasspr-registration"></a>Inscription MFA/SSPR combinée améliorée

**Type :** Fonctionnalité modifiée **Catégorie de service :** Réinitialisation du mot de passe libre-service **Fonctionnalité produit :** Authentification utilisateur

En réponse aux commentaires des clients, nous avons amélioré l’expérience en préversion d’inscription MFA/SSPR combinée, pour aider vos utilisateurs à inscrire plus rapidement leurs informations de sécurité pour MFA et SSPR.

**Pour activer l’expérience améliorée pour vos utilisateurs dès aujourd’hui, procédez comme suit :**

1. En tant qu’administrateur général ou administrateur des utilisateurs, connectez-vous au Portail Azure et accédez à **Azure Active Directory > Paramètres utilisateur > Gérer les paramètres d’accès aux fonctionnalités en préversion du panneau**.

2. Dans l’option **Utilisateurs pouvant utiliser les fonctionnalités en préversion pour inscrire et gérer les informations de sécurité – actualiser**, choisissez d’activer les fonctionnalités pour un **Groupe sélectionné d’utilisateurs** ou pour **Tous les utilisateurs**.

Au cours des prochaines semaines, nous supprimerons la possibilité d’activer l’ancienne expérience en préversion d’inscription MFA/SSPR combinée pour les locataires sur lesquels elle n’est pas encore activée.

**Pour voir si le contrôle sera supprimé pour votre locataire, procédez comme suit :**

1. En tant qu’administrateur général ou administrateur des utilisateurs, connectez-vous au Portail Azure et accédez à **Azure Active Directory > Paramètres utilisateur > Gérer les paramètres d’accès aux fonctionnalités en préversion du panneau**.

2. Si l’option **Utilisateurs pouvant utiliser les fonctionnalités en préversion pour inscrire et gérer les informations de sécurité** est définie sur **Aucun**, l’option sera supprimée de votre locataire.

Que vous ayez précédemment activé l’ancienne expérience en préversion d’inscription MFA/SSPR combinée pour les utilisateurs ou non, l’ancienne expérience sera désactivée à une date ultérieure. Pour cette raison, nous vous recommandons vivement d’opter pour la nouvelle expérience améliorée dès que possible.

Pour plus d’informations sur l’expérience d’inscription améliorée, consultez [Améliorations utiles de l’expérience d’inscription de réinitialisation du mot de passe et MFA combinée Azure AD](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Cool-enhancements-to-the-Azure-AD-combined-MFA-and-password/ba-p/354271).

---

### <a name="updated-policy-management-experience-for-user-flows"></a>Expérience de gestion de stratégie mise à jour pour les flux utilisateur

**Type :** Fonctionnalité modifiée **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Nous avons mis à jour le processus de création et de gestion de stratégie pour les flux utilisateur (anciennement, les stratégies intégrées). Cette nouvelle expérience est désormais la valeur par défaut pour tous vos locataires Azure AD.

Vous pouvez fournir vos commentaires et suggestions en utilisant les icônes smiley content ou smiley mécontent dans la zone **Envoyez-nous vos commentaires** en haut de l’écran du portail.

Pour plus d’informations sur la nouvelle expérience de gestion de stratégie, consultez le blog [Azure AD B2C now has JavaScript customization and many more new features](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595) (Azure AD B2C inclut désormais la personnalisation JavaScript et beaucoup d’autres fonctionnalités nouvelles).

---

### <a name="choose-specific-page-element-versions-provided-by-azure-ad-b2c"></a>Choisir des versions d’éléments de page spécifiques fournies par Azure AD B2C

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Vous pouvez maintenant choisir une version spécifique des éléments de page fournis par Azure AD B2C. En sélectionnant une version spécifique, vous pouvez tester vos mises à jour avant qu’elles s’affichent sur une page et vous pouvez obtenir un comportement prévisible. En outre, vous pouvez maintenant vous inscrire pour appliquer des versions de page spécifiques afin d’autoriser les personnalisations JavaScript. Pour activer cette fonctionnalité, accédez à la page **Propriétés** dans vos flux utilisateur.

Pour plus d’informations sur la sélection de versions spécifiques des éléments de page, consultez le blog [Azure AD B2C now has JavaScript customization and many more new features](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595) (Azure AD B2C inclut désormais la personnalisation JavaScript et beaucoup d’autres fonctionnalités nouvelles).

---

### <a name="configurable-end-user-password-requirements-for-b2c-ga"></a>Exigences de mot de passe de l’utilisateur configurable pour B2C (DG)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Vous pouvez maintenant configurer la complexité du mot de passe de votre organisation pour les utilisateurs finaux, au lieu d’utiliser votre stratégie de mot de passe Azure AD native. À partir du panneau **Propriétés** de vos flux utilisateur (anciennement, les stratégies intégrées), vous pouvez choisir une complexité du mot de passe entre **Simple** ou **Forte**, ou vous pouvez créer un ensemble **Personnalisé** d’exigences.

Pour plus d’informations sur la configuration de la complexité du mot de passe, consultez [Configurer les exigences de complexité des mots de passe dans Azure Active Directory B2C](../../active-directory-b2c/password-complexity.md).

---

### <a name="new-default-templates-for-custom-branded-authentication-experiences"></a>Nouveaux modèles par défaut pour les expériences d’authentification de marque personnalisée

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Vous pouvez désormais utiliser nos nouveaux modèles par défaut, situés sur le panneau **Dispositions de pages** de vos flux utilisateur (anciennement, les stratégies intégrées), pour créer une expérience d’authentification de marque personnalisée pour vos utilisateurs.

Pour plus d’informations sur l’utilisation des modèles, consultez [Azure Active Directory B2C inclut désormais la personnalisation JavaScript et beaucoup d’autres fonctionnalités nouvelles](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-B2C-now-has-JavaScript-customization-and-many-more-new/ba-p/353595).

---

## <a name="january-2019"></a>Janvier 2019

### <a name="active-directory-b2b-collaboration-using-one-time-passcode-authentication-public-preview"></a>Collaboration Active Directory B2B à l'aide de l'authentification par code secret à usage unique (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2B **Fonctionnalité produit :** B2B/B2C

Nous avons introduit l'authentification par code secret à usage unique (OTP) pour les utilisateurs invités B2B qui ne peuvent pas être authentifiés par d'autres moyens, comme Azure AD, un compte Microsoft (MSA) ou la fédération Google. Avec cette nouvelle méthode d'authentification, les utilisateurs invités n'ont pas besoin de créer de nouveau compte Microsoft. Lors de l'échange d'une invitation ou de l'accès à une ressource partagée, l'utilisateur invité peut demander qu'un code temporaire soit envoyé à une adresse e-mail. À l'aide de ce code temporaire, l'utilisateur invité peut continuer à se connecter.

Pour plus d'informations, consultez [Authentification à l'aide d'un code secret à usage unique envoyé par e-mail (préversion)](../external-identities/one-time-passcode.md) et le blog [Azure AD permet à tous les utilisateurs de partager et de collaborer en toute simplicité sans compte](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-AD-makes-sharing-and-collaboration-seamless-for-any-user/ba-p/325949).

### <a name="new-azure-ad-application-proxy-cookie-settings"></a>Nouveaux paramètres de cookies pour le proxy d'application Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Nous avons introduit trois nouveaux paramètres de cookies, disponibles pour vos applications publiées via le proxy d’application :

- **Utiliser un cookie HTTPOnly.** Définit l'indicateur **HTTPOnly** sur l'accès du proxy d'application et les cookies de session. Cette option offre des avantages supplémentaires en termes de sécurité, et permet notamment d'éviter de copier ou de modifier les cookies via des scripts côté client. Nous vous recommandons d’activer cet indicateur (sélectionnez **Oui**) pour bénéficier de ces avantages supplémentaires.

- **Utiliser un cookie sécurisé.** Définit l'indicateur **Sécurisé** sur l'accès du proxy d'application et les cookies de session. Cette option offre des avantages supplémentaires en termes de sécurité, et veille à ce que les cookies soient uniquement transmis via des canaux sécurisés TLS, tels que HTTPS. Nous vous recommandons d’activer cet indicateur (sélectionnez **Oui**) pour bénéficier de ces avantages supplémentaires.

- **Utiliser un cookie persistant.** Empêche l'expiration des cookies d'accès lorsque le navigateur web est fermé. La durée de vie de ces cookies correspond à la durée de vie du jeton d'accès. Cela étant, les cookies sont réinitialisés une fois le délai d'expiration atteint ou si l’utilisateur supprime manuellement le cookie. Nous vous recommandons de conserver le paramètre par défaut **Non**, et de n'activer le paramètre que pour les applications plus anciennes qui ne partagent pas de cookies entre processus.

Pour plus d’informations sur les nouveaux cookies, consultez [Paramètres de cookies pour l’accès aux applications locales dans Azure Active Directory](../manage-apps/application-proxy-configure-cookie-settings.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---january-2019"></a>Nouvelles applications fédérées disponibles dans la galerie d'applications Azure AD - Janvier 2019

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En janvier 2019, nous avons ajouté à notre galerie d’applications les 35 applications suivantes qui prennent en charge la fédération :

[Firstbird](../saas-apps/firstbird-tutorial.md), [Folloze](../saas-apps/folloze-tutorial.md), [Talent Palette](../saas-apps/talent-palette-tutorial.md), [Infor CloudSuite](../saas-apps/infor-cloud-suite-tutorial.md), [Cisco Umbrella](../saas-apps/cisco-umbrella-tutorial.md), [Zscaler Internet Access Administrator](../saas-apps/zscaler-internet-access-administrator-tutorial.md), [Expiration Reminder](../saas-apps/expiration-reminder-tutorial.md), [InstaVR Viewer](../saas-apps/instavr-viewer-tutorial.md), [CorpTax](../saas-apps/corptax-tutorial.md), [Verb](https://app.verb.net/login), [OpenLattice](https://openlattice.com/agora), [TheOrgWiki](https://www.theorgwiki.com/signup), [Pavaso Digital Close](../saas-apps/pavaso-digital-close-tutorial.md), [GoodPractice Toolkit](../saas-apps/goodpractice-toolkit-tutorial.md), [Cloud Service PICCO](../saas-apps/cloud-service-picco-tutorial.md), [AuditBoard](../saas-apps/auditboard-tutorial.md), [iProva](../saas-apps/iprova-tutorial.md), [Workable](../saas-apps/workable-tutorial.md), [CallPlease](https://webapp.callplease.com/create-account/create-account.html), [GTNexus SSO System](../saas-apps/gtnexus-sso-module-tutorial.md), [CBRE ServiceInsight](../saas-apps/cbre-serviceinsight-tutorial.md), [Deskradar](../saas-apps/deskradar-tutorial.md), [Coralogixv](../saas-apps/coralogix-tutorial.md), [Signagelive](../saas-apps/signagelive-tutorial.md), [ARES for Enterprise](../saas-apps/ares-for-enterprise-tutorial.md), [K2 for Office 365](https://www.k2.com/O365), [Xledger](https://www.xledger.net/), [iDiD Manager](../saas-apps/idid-manager-tutorial.md), [HighGear](../saas-apps/highgear-tutorial.md), [Visitly](../saas-apps/visitly-tutorial.md), [Korn Ferry ALP](../saas-apps/korn-ferry-alp-tutorial.md), [Acadia](../saas-apps/acadia-tutorial.md), [Adoddle cSaas Platform](../saas-apps/adoddle-csaas-platform-tutorial.md)<!-- , [CaféX Portal (Meetings)](https://docs.microsoft.com/azure/active-directory/saas-apps/cafexportal-meetings-tutorial), [MazeMap Link](https://docs.microsoft.com/azure/active-directory/saas-apps/mazemaplink-tutorial)-->

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-azure-ad-identity-protection-enhancements-public-preview"></a>Nouvelles améliorations apportées à Azure AD Identity Protection (préversion publique)

**Type :** Fonctionnalité modifiée **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Nous sommes heureux d’annoncer l'ajout des améliorations suivantes à l’offre de préversion publique d’Azure AD Identity Protection, notamment :

- Interface utilisateur mise à jour et mieux intégrée

- API supplémentaires

- Évaluation des risques renforcée via Machine Learning

- Alignement des produits pour les utilisateurs et les connexions à risque

Consultez [Qu’est-ce qu’Azure Active Directory Identity Protection (version actualisée) ?](../identity-protection/overview-identity-protection.md) pour plus d'informations et partager vos idées via des invites intégrées au produit.

---

### <a name="new-app-lock-feature-for-the-microsoft-authenticator-app-on-ios-and-android-devices"></a>Nouvelle fonctionnalité Verrou d'application dans l'application Microsoft Authenticator sur les appareils iOS et Android

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Application Microsoft Authenticator **Fonctionnalité produit :** Protection et sécurité des identités

Pour sécuriser les codes secrets à usage unique, les informations et les paramètres d'application, vous pouvez activer la fonctionnalité Verrou d'application dans l'application Microsoft Authenticator. Si vous activez Verrou d’application, vous êtes invité à vous authentifier à l’aide de votre code PIN ou verrou biométrique chaque fois que vous ouvrez l’application Microsoft Authenticator.

Pour plus d’informations, consultez [Forum aux questions sur l’application Microsoft Authenticator](../user-help/user-help-auth-app-faq.md).

---

### <a name="enhanced-azure-ad-privileged-identity-management-pim-export-capabilities"></a>Fonctionnalités d'exportation améliorées dans Azure AD Privileged Identity Management (PIM)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Les administrateurs Privileged Identity Management (PIM) peuvent désormais exporter toutes les affectations de rôles actifs et éligibles pour une ressource spécifique, ce qui comprend les affectations de rôles pour toutes les ressources enfants. Avant, il était difficile pour les administrateurs d’obtenir une liste complète des attributions de rôle d’un abonnement, car ils devaient les exporter pour chaque ressource spécifique.

Pour plus d'informations, consultez [Afficher l’historique d’audit et d’activité pour les rôles de ressources Azure dans PIM](../privileged-identity-management/azure-pim-resource-rbac.md).

---

## <a name="novemberdecember-2018"></a>Novembre/décembre 2018

### <a name="users-removed-from-synchronization-scope-no-longer-switch-to-cloud-only-accounts"></a>Des utilisateurs retirés de l’étendue de synchronisation ne basculent plus vers des comptes uniquement cloud

**Type :** Corrigé **Catégorie de service :** Gestion des utilisateurs **Fonctionnalité produit :** Répertoire

>[!Important]
>Nous avons entendu et nous comprenons votre frustration provoquée par ce correctif. Nous avons donc annulé cette modification jusqu’à ce que nous soyons en mesure de rendre l’implémentation de ce correctif plus facile dans votre organisation.

Nous avons résolu un bogue qui faisait que l’indicateur DirSyncEnabled d’un utilisateur passait erronément à **False** lorsque l’objet Active Directory Domain Services (AD DS) était exclu de l’étendue de synchronisation, puis déplacé vers la Corbeille dans Azure AD lors du cycle de synchronisation suivant. Ce correctif a pour effet que, si l’utilisateur est exclu de l’étendue de la synchronisation, puis restauré à partir de la Corbeille Azure AD, le compte d’utilisateur reste synchronisé à partir de l’AD local, comme prévu, et ne peut pas être géré dans le cloud car sa source d’autorité demeure en tant qu’AD local.

Avant ce correctif, un problème se produisait quand l’indicateur DirSyncEnabled était commuté sur False. Cela donnait la fausse impression que ces comptes avaient été convertis en objets uniquement cloud, et que les comptes pouvaient être gérés dans le nuage. Cependant, les comptes conservaient toujours leur source d’autorité comme locale, et toutes les propriétés synchronisées (attributs fantômes) provenant de l’AD local. Cette situation entraînait plusieurs problèmes dans Azure AD et dans d’autres charges de travail cloud (telles qu’Exchange Online) qui s’attendaient à traiter ces comptes comme synchronisés à partir d’AD, mais se comportaient désormais comme des comptes uniquement cloud.

À ce stade, la seule façon de convertir véritablement un compte synchronisé à partir d’AD vers un compte uniquement cloud consiste à désactiver la synchronisation d’annuaires au niveau du locataire, ce qui déclenche une opération de serveur principal pour transférer la source d’autorité. Ce type de modification de source d’autorité nécessite (sans que ce soit limitatif) un nettoyage de tous les attributs associés locaux (tels que LastDirSyncTime et les attributs fantômes), et l’envoi d’un signal à d’autres charges de travail cloud pour que son objet respectif soit également converti en un compte uniquement cloud.

Ce correctif empêche donc des mises à jour directes de l’attribut ImmutableID d’un utilisateur synchronisé à partir d’Active Directory, qui étaient requises dans certains scénarios par le passé. Par conception, l’ImmutableID d’un objet dans Azure AD, comme son nom l’indique, est censé être immuable. De nouvelles fonctionnalités implémentées dans Azure AD Connect Health et le client de synchronization Azure AD Connect sont disponibles pour de tels scénarios :

- **Mise à jour d’ImmutableID à grande échelle pour de nombreux utilisateurs en suivant une approche par étapes**

  Par exemple, vous devez effectuer une migration inter-forêts d’AD DS un peu longue. Solution : Utilisez Azure AD Connect pour **configurer l’ancre source**, puis, lorsque l’utilisateur migre, copiez les valeurs d’ImmutableID existantes d’Azure AD dans l’attribut ms-DS-Consistency-Guid de l’utilisateur d’AD DS local de la nouvelle forêt. Pour plus d’informations, voir [Utilisation de ms-DS-ConsistencyGuid en tant qu’attribut sourceAnchor](../hybrid/plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor).

- **Mises à jour d’ImmutableID à grande échelle pour de nombreux utilisateurs en une fois**

  Par exemple, lors de l’implémentation d’Azure AD Connect, vous commettez une erreur qui vous amène à devoir modifier l’attribut SourceAnchor. Solution : Désactivez la synchronisation d’annuaires au niveau locataire, et effacez toutes les valeurs d’ImmutableID non valides. Pour plus d’informations, voir [Désactiver la synchronisation d’annuaires pour Office 365](/office365/enterprise/turn-off-directory-synchronization).

- **Rétablissement de correspondance entre l’utilisateur local et un utilisateur existant dans Azure AD** Par exemple, un utilisateur recréé dans AD DS génère un doublon dans le compte Azure AD au lieu qu’une correspondance soit rétablie avec un compte Azure AD existant (objet orphelin). Solution : Utilisez Azure AD Connect Health dans le portail Microsoft Azure pour remapper l’ancre source/l’ImmutableID. Pour plus d’informations, voir [Scénario d’objet orphelin](../hybrid/how-to-connect-health-diagnose-sync-errors.md#orphaned-object-scenario).

### <a name="breaking-change-updates-to-the-audit-and-sign-in-logs-schema-through-azure-monitor"></a>Changement cassant : Mises à jour du schéma des journaux d’activité de connexion et d’audit via Azure Monitor

**Type :** Fonctionnalité modifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous publions actuellement à la fois des flux de journaux d’audit et de connexion via Azure Monitor. Vous pouvez donc en toute transparence intégrer les fichiers journaux avec vos outils SIEM ou Log Analytics. En réponse à vos commentaires et en vue de l’annonce de la disponibilité générale de cette fonctionnalité, nous apportons les modifications suivantes à notre schéma. Ces modifications de schéma et les mises à jour de documentation connexes se produiront d’ici la première semaine de janvier.

#### <a name="new-fields-in-the-audit-schema"></a>Nouveaux champs du schéma d’audit
Nous ajoutons un nouveau champ **Type d’opération**, pour fournir le type d’opération effectuée sur la ressource. Par exemple, **Ajouter**, **Mettre à jour** ou **Supprimer**.

#### <a name="changed-fields-in-the-audit-schema"></a>Champs modifiés du schéma d’audit
Les champs suivants font l’objet de modifications dans le schéma d’audit :

|Nom du champ|Ce qui a changé|Anciennes valeurs|Nouvelles valeurs|
|----------|------------|----------|----------|
|Category|Il s’agissait du champ **Nom de service** qui se nomme à présent **Catégories d’audit**. **Nom du service** a été renommé en **loggedByService**.|<ul><li>Approvisionnement des comptes</li><li>Annuaire principal</li><li>Réinitialisation de mot de passe en libre service</li></ul>|<ul><li>User Management</li><li>Gestion des groupes</li><li>Gestion des applications</li></ul>|
|targetResources|Inclut **TargetResourceType** au niveau supérieur.|&nbsp;|<ul><li>Policy</li><li>Application</li><li>Utilisateur</li><li>Groupe</li></ul>|
|loggedByService|Fournit le nom du service ayant généré le journal d’audit.|Null|<ul><li>Approvisionnement des comptes</li><li>Annuaire principal</li><li>Réinitialisation de mot de passe libre-service</li></ul>|
|Résultats|Fournit le résultat des journaux d’audit. Auparavant, nous présentions une énumération, mais nous montrons maintenant la valeur réelle.|<ul><li>0</li><li>1</li></ul>|<ul><li>Succès</li><li>Échec</li></ul>|

#### <a name="changed-fields-in-the-sign-in-schema"></a>Champs modifiés du schéma de connexion
Les champs suivants font l’objet de modifications dans le schéma de connexion :

|Nom du champ|Ce qui a changé|Anciennes valeurs|Nouvelles valeurs|
|----------|------------|----------|----------|
|appliedConditionalAccessPolicies|Ce champ était nommé **conditionalaccessPolicies**. Il s’intitule à présent **conditionalaccessPolicies**.|Aucun changement|Aucun changement|
|conditionalAccessStatus|Fournit le résultat de l’état de la stratégie d’accès conditionnel au moment de la connexion. Auparavant, nous présentions une énumération, mais nous montrons maintenant la valeur réelle.|<ul><li>0</li><li>1</li><li>2</li><li>3</li></ul>|<ul><li>Succès</li><li>Échec</li><li>Non appliqué</li><li>Désactivé</li></ul>|
|appliedConditionalAccessPolicies: result|Fournit le résultat de l’état individuel de la stratégie d’accès conditionnel au moment de la connexion. Auparavant, nous présentions une énumération, mais nous montrons maintenant la valeur réelle.|<ul><li>0</li><li>1</li><li>2</li><li>3</li></ul>|<ul><li>Succès</li><li>Échec</li><li>Non appliqué</li><li>Désactivé</li></ul>|

Pour plus d’informations sur le schéma, consultez [Interpréter le schéma des journaux d’audit Azure Active Directory dans Azure Monitor (préversion)](../reports-monitoring/reference-azure-monitor-audit-log-schema.md).

---

### <a name="identity-protection-improvements-to-the-supervised-machine-learning-model-and-the-risk-score-engine"></a>Améliorations de la protection des identités apportées dans le modèle Machine Learning supervisé et le moteur d’indice de risque

**Type :** Fonctionnalité modifiée **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Indices de risque

Les améliorations apportées à l’utilisateur Identity Protection et au moteur d’évaluation des risques de connexion peuvent contribuer à améliorer la couverture et l’exactitude du risque utilisateur. Les administrateurs peuvent constater que le niveau de risque utilisateur n’est plus directement lié au niveau de risque de détections spécifiques, et qu’il existe une augmentation du nombre et du niveau des événements de connexion à risque.

Les détections de risque sont à présent évaluées par le modèle Machine Learning supervisé, qui calcule le risque utilisateur à l’aide des fonctionnalités supplémentaires des connexions utilisateur et d’un modèle de détections. Selon ce modèle, l’administrateur peut rechercher les utilisateurs présentant des indices de risque élevés, même si les détections associées à cet utilisateur sont de risque faible ou moyen.

---

### <a name="administrators-can-reset-their-own-password-using-the-microsoft-authenticator-app-public-preview"></a>Les administrateurs peuvent réinitialiser leur mot de passe à l’aide de l’application Microsoft Authenticator (préversion publique)

**Type :** Fonctionnalité modifiée **Catégorie de service :** Réinitialisation du mot de passe libre-service **Fonctionnalité produit :** Authentification utilisateur

Les administrateurs Azure AD peuvent à présent réinitialiser leur mot de passe en utilisant les notifications d’application Microsoft Authenticator ou un code à partir d’un jeton matériel ou d’une application d’authentificateur mobile. Pour réinitialiser leur mot de passe, les administrateurs peuvent à présent utiliser deux des méthodes suivantes :

- Notification de l’application Microsoft Authenticator

- Autre application mobile d’authentificateur/code de jeton matériel

- E-mail

- appel téléphonique

- SMS

Pour plus d’informations sur l’utilisation de l’application Microsoft Authenticator pour réinitialiser les mots de passe, consultez [Réinitialisation de mot de passe en libre-service Azure AD - Application Mobile et SSPR (préversion)](../authentication/concept-sspr-howitworks.md#mobile-app-and-sspr)

---

### <a name="new-azure-ad-cloud-device-administrator-role-public-preview"></a>Nouveau rôle Administrateur d’appareil cloud Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion et inscription des appareils **Fonctionnalité produit :** Contrôle d’accès

Les administrateurs peuvent attribuer des utilisateurs au nouveau rôle Administrateur d’appareil cloud pour effectuer des tâches d’administration des appareils cloud. Les utilisateurs attribués au rôle Administrateur d’appareil cloud peuvent activer, désactiver et supprimer des appareils dans Azure AD et lire des clés BitLocker Windows 10 (le cas échéant) dans le portail Azure.

Pour plus d’informations sur les rôles et les autorisations, consultez [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).

---

### <a name="manage-your-devices-using-the-new-activity-timestamp-in-azure-ad-public-preview"></a>Gérer vos appareils avec le nouveau timestamp d’activité dans Azure AD (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion et inscription des appareils **Fonctionnalité produit :** Gestion du cycle de vie des appareils

Nous sommes conscients qu’au fil du temps, vous devez actualiser et mettre hors service des appareils de votre organisation dans Azure AD pour éviter la présence d’appareils obsolètes dans votre environnement. Pour faciliter ce processus, Azure AD met maintenant à jour vos appareils avec un nouveau timestamp d’activité, pour vous aider à gérer le cycle de vie de vos appareils.

Pour en savoir plus sur la récupération et l’utilisation de ce timestamp, consultez la page [Procédure : Gérer les appareils obsolètes dans Azure AD](../devices/manage-stale-devices.md)

---

### <a name="administrators-can-require-users-to-accept-a-terms-of-use-on-each-device"></a>Les administrateurs peuvent demander aux utilisateurs d’accepter les conditions d’utilisation sur chaque appareil

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Gouvernance

Les administrateurs peuvent maintenant activer l’option **Demander le consentement des utilisateurs sur chaque appareil** pour demander aux utilisateurs d’accepter vos conditions d’utilisation sur chaque appareil à partir duquel ils utilisent votre locataire.

Pour plus d’informations, consultez la section [Conditions d’utilisation par appareil de la page Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md#per-device-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-a-recurring-schedule"></a>Les administrateurs peuvent configurer l’expiration des conditions d’utilisation selon une planification périodique

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Gouvernance


Les administrateurs peuvent maintenant activer l’option **Faire expirer les consentements** pour faire expirer les conditions d’utilisation de tous vos utilisateurs en fonction de votre planification périodique spécifiée. La planification peut être une fois par an, deux fois par an, tous les trimestres ou tous les mois. Une fois que les conditions d’utilisation ont expiré, les utilisateurs doivent les réaccepter.

Pour plus d’informations, consultez la section [Ajouter des conditions d’utilisation de la page Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md#add-terms-of-use).

---

### <a name="administrators-can-configure-a-terms-of-use-to-expire-based-on-each-users-schedule"></a>Les administrateurs peuvent configurer l’expiration des conditions d’utilisation selon la planification de chaque utilisateur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Gouvernance

Les administrateurs peuvent maintenant spécifier la durée après laquelle l’utilisateur doit réaccepter les conditions d’utilisation. Par exemple, les administrateurs peuvent spécifier que les utilisateurs doivent réaccepter les conditions d’utilisation tous les 90 jours.

Pour plus d’informations, consultez la section [Ajouter des conditions d’utilisation de la page Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md#add-terms-of-use).

---

### <a name="new-azure-ad-privileged-identity-management-pim-emails-for-azure-active-directory-roles"></a>Nouveaux e-mails Azure Active Directory Privileged Identity Management (PIM) pour les rôles Azure Active Directory

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Les clients qui utilisent Azure AD Privileged Identity Management (PIM) peuvent à présent recevoir un e-mail de synthèse hebdomadaire, comprenant notamment les informations suivantes pour les sept derniers jours :

- Vue d’ensemble des principales attributions de rôles éligibles et permanentes

- Nombre d’utilisateurs activant des rôles

- Nombre d’utilisateurs attribués à des rôles dans PIM

- Nombre d’utilisateurs attribués à des rôles en dehors de PIM

- Nombre d’utilisateurs « rendus permanents » dans PIM

Pour plus d’informations sur PIM et les notifications par e-mail disponibles, consultez [Notifications par e-mail dans PIM](../privileged-identity-management/pim-email-notifications.md).

---

### <a name="group-based-licensing-is-now-generally-available"></a>Disponibilité générale de la gestion des licences basée sur les groupes

**Type :** Fonctionnalité modifiée **Catégorie de service :** Autre **Fonctionnalité produit :** Répertoire

La gestion des licences basée sur les groupes passe de la préversion publique à la disponibilité générale. Dans le cadre de cette disponibilité générale, nous avons accru la scalabilité de cette fonctionnalité et ajouté la possibilité de retraiter les attributions de la gestion des licences basée sur les groupes pour un seul utilisateur, ainsi que la possibilité d’utiliser la gestion des licences basée sur les groupes avec des licences Office 365 E3/A3.

Pour plus d’informations sur la gestion des licences basée sur les groupes, consultez la page sur la [gestion des licences basée sur les groupes dans Azure Active Directory](./active-directory-licensing-whatis-azure-portal.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---november-2018"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD (novembre 2018)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En novembre 2018, nous avons ajouté dans notre galerie d’applications les 26 nouvelles applications ci-après qui prennent en charge la fédération :

[CoreStack](https://cloud.corestack.io/site/login), [HubSpot](../saas-apps/hubspot-tutorial.md), [GetThere](../saas-apps/getthere-tutorial.md), [Gra-Pe](../saas-apps/grape-tutorial.md), [eHour](https://getehour.com/try-now), [Consent2Go](../saas-apps/consent2go-tutorial.md), [Appinux](../saas-apps/appinux-tutorial.md), [DriveDollar](https://azuremarketplace.microsoft.com/marketplace/apps/savitas.drivedollar-azuread?tab=Overview), [Useall](../saas-apps/useall-tutorial.md), [Infinite Campus](../saas-apps/infinitecampus-tutorial.md), [Alaya](https://alayagood.com), [HeyBuddy](../saas-apps/heybuddy-tutorial.md), [Wrike SAML](../saas-apps/wrike-tutorial.md), [Drift](../saas-apps/drift-tutorial.md), [Zenegy for Business Central 365](https://accounting.zenegy.com/), [Everbridge Member Portal](../saas-apps/everbridge-tutorial.md), [IDEO](https://profile.ideo.com/users/sign_up), [Ivanti Service Manager (ISM)](../saas-apps/ivanti-service-manager-tutorial.md), [Peakon](../saas-apps/peakon-tutorial.md), [Allbound SSO](../saas-apps/allbound-sso-tutorial.md), [Plex Apps - Classic Test](https://test.plexonline.com/signon), [Plex Apps – Classic](https://www.plexonline.com/signon), [Plex Apps - UX Test](https://test.cloud.plex.com/sso), [Plex Apps – UX](https://cloud.plex.com/sso), [Plex Apps – IAM](https://accounts.plex.com/), [CRAFTS - Childcare Records, Attendance, & Financial Tracking System](https://getcrafts.ca/craftsregistration)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

## <a name="october-2018"></a>Octobre 2018

### <a name="azure-ad-logs-now-work-with-azure-log-analytics-public-preview"></a>Les journaux d’activité Azure AD fonctionnent maintenant avec Azure Log Analytics (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Nous sommes heureux d’annoncer que vous pouvez maintenant transférer vos journaux d’activité Azure AD vers Azure Log Analytics ! Grâce à cette fonctionnalité très demandée, vous pouvez encore mieux accéder aux analyses de votre activité, de vos opérations et de votre sécurité, et elle vous aide à surveiller votre infrastructure. Pour plus d’informations, consultez le blog [Journaux d’activité Azure Active Directory dans Azure Log Analytics désormais disponible](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Azure-Active-Directory-Activity-logs-in-Azure-Log-Analytics-now/ba-p/274843).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---october-2018"></a>Nouvelles applications fédérées disponibles dans la galerie d'applications Azure AD (octobre 2018)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En octobre 2018, nous avons ajouté à notre galerie d’applications les 14 nouvelles applications ci-dessous, qui prennent en charge la fédération :

[My Award Points](../saas-apps/myawardpoints-tutorial.md), [Vibe HCM](../saas-apps/vibehcm-tutorial.md), ambyint, [MyWorkDrive](../saas-apps/myworkdrive-tutorial.md), [BorrowBox](../saas-apps/borrowbox-tutorial.md), Dialpad, [ON24 Virtual Environment](../saas-apps/on24-tutorial.md), [RingCentral](../saas-apps/ringcentral-tutorial.md), [Zscaler Three](../saas-apps/zscaler-three-tutorial.md), [Phraseanet](../saas-apps/phraseanet-tutorial.md), [Appraisd](../saas-apps/appraisd-tutorial.md), [Workspot Control](../saas-apps/workspotcontrol-tutorial.md), [Shuccho Navi](../saas-apps/shucchonavi-tutorial.md), [Glassfrog](../saas-apps/glassfrog-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="azure-ad-domain-services-email-notifications"></a>Notifications par e-mail pour Azure AD Domain Services

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD Domain Services **Fonctionnalité produit :** Services de domaine Azure AD

Azure AD Domain Services fournit sur le portail Azure des alertes sur les erreurs de configuration ou les problèmes avec votre domaine géré. Ces alertes incluent des guides pas à pas pour que vous puissiez essayer de résoudre les problèmes sans avoir à contacter le support.

En octobre, vous serez en mesure de personnaliser les paramètres de notification pour votre domaine managé afin d’en cas de nouvelles alertes, un e-mail soit envoyé à un groupe de personnes désigné, ce qui évite de devoir vérifier en permanence le portail pour détecter les mises à jour.

Pour plus d’informations, consultez [Paramètres de notification dans Azure AD Domain Services](../../active-directory-domain-services/notifications.md).

---

### <a name="azure-ad-portal-supports-using-the-forcedelete-domain-api-to-delete-custom-domains"></a>Le portail Azure AD assure la prise en charge à l’aide de l’API du domaine ForceDelete pour supprimer des domaines personnalisés.

**Type :** Fonctionnalité modifiée **Catégorie de service :** Gestion des annuaires **Fonctionnalité produit :** Répertoire

Nous sommes heureux de vous annoncer que vous pouvez maintenant utiliser l'API de domaine ForceDelete pour supprimer vos noms de domaine personnalisés en renommant de façon asynchrone les références, comme les utilisateurs, les groupes et les applications de votre nom de domaine personnalisé (contoso.com) pour revenir au nom de domaine par défaut initial (contoso.onmicrosoft.com).

Ce changement vous permet de supprimer plus rapidement vos noms de domaine personnalisés si votre organisation n'utilise plus le nom, ou si vous devez utiliser le nom de domaine avec un autre AD Azure.

Pour plus d’informations, consultez [Supprimer un nom de domaine personnalisé](../enterprise-users/domains-manage.md#delete-a-custom-domain-name).

---

## <a name="september-2018"></a>Septembre 2018

### <a name="updated-administrator-role-permissions-for-dynamic-groups"></a>Mise à jour des autorisations des rôles d’administrateur pour les groupes dynamiques

**Type :** Corrigé **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Nous avons résolu un problème afin que des rôles d’administrateur spécifiques soient désormais en mesure de créer et mettre à jour des règles d’appartenance dynamique sans avoir besoin d’être le propriétaire du groupe.

Ces rôles sont les suivants :

- Administrateur général

- Administrateur Intune

- Administrateur d’utilisateurs

Pour plus d’informations, consultez l’article [Créer un groupe dynamique et vérifier l’état](../enterprise-users/groups-create-rule.md).

---

### <a name="simplified-single-sign-on-sso-configuration-settings-for-some-third-party-apps"></a>Simplification des paramètres de configuration d’authentification unique (SSO) pour certaines applications tierces

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Nous sommes conscients que la configuration de l’authentification unique (SSO) pour les applications SaaS (software as a service) peut se révéler complexe en raison du caractère unique de chaque configuration d’application. Nous avons conçu une expérience de configuration simplifiée qui renseigne automatiquement les paramètres de configuration SSO pour les applications SaaS tierces suivantes :

- Zendesk

- ArcGis Online

- Jamf Pro

Pour commencer à utiliser cette expérience en un clic, accédez au **Portail Azure** > page **Configuration de l’authentification unique** de l’application. Pour plus d’informations, consultez l’article [Intégration des applications SaaS avec Azure Active Directory](../saas-apps/tutorial-list.md).

---

### <a name="azure-active-directory---where-is-your-data-located-page"></a>Page Azure Active Directory - Where is your data located? (Azure Active Directory - Où se trouvent vos données ?)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Autre **Fonctionnalité produit :** GoLocal

Sur la page **Azure Active Directory - Where is your data located?** (Azure Active Directory - Où se trouvent vos données ?), sélectionnez la région de votre entreprise pour connaître le centre de données Azure qui héberge vos données Azure AD au repos pour tous les services Azure AD. Vous pouvez filtrer les informations par service Azure AD pour la région de votre entreprise.

Pour accéder à cette fonctionnalité et pour obtenir plus d’informations, consultez l’article [Azure Active Directory - Where is your data located?](https://aka.ms/AADDataMap) (Azure Active Directory - Où se trouvent vos données ?).

---

### <a name="new-deployment-plan-available-for-the-my-apps-access-panel"></a>Nouveau plan de déploiement disponible pour le panneau d’accès MyApps

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Mes applications **Fonctionnalité produit :** SSO

Consultez le nouveau plan de déploiement disponible pour le panneau d’accès MyApps (https://aka.ms/deploymentplans).
Le panneau d’accès MyApps permet aux utilisateurs de rechercher l’ensemble de leurs applications et d’y accéder à partir d’un emplacement unique. Ce portail offre également aux utilisateurs différentes fonctionnalités en libre-service, telles que des fonctions de demande d’accès aux applications et aux groupes ou de gestion de l’accès à ces ressources pour le compte d’autres utilisateurs.

Pour plus d’informations, consultez l’article [Qu’est-ce que le portail MyApps ?](../user-help/my-apps-portal-end-user-access.md).

---

### <a name="new-troubleshooting-and-support-tab-on-the-sign-ins-logs-page-of-the-azure-portal"></a>Nouvel onglet Résolution des problèmes et support sur la page des journaux d’activité de connexions du Portail Azure

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Le nouvel onglet **Résolution des problèmes et support** de la page **Connexions** du Portail Azure est destinée à aider les administrateurs et les ingénieurs du support à résoudre les problèmes liés aux connexions Azure AD. Ce nouvel onglet fournit le code d’erreur, le message d’erreur et des recommandations de correction (le cas échéant) pour faciliter la résolution du problème. Si vous ne parvenez pas à résoudre le problème, nous vous indiquons également une nouvelle façon de créer un ticket de support à l’aide de l’expérience **Copier dans le Presse-papiers**, qui remplit les champs **ID de demande** et **Date (UTC)** pour le fichier journal dans votre ticket de support.

![Journaux d’activité de connexion présentant le nouvel onglet](media/whats-new/troubleshooting-and-support.png)

---

### <a name="enhanced-support-for-custom-extension-properties-used-to-create-dynamic-membership-rules"></a>Amélioration de la prise en charge des propriétés d’extension personnalisées utilisées pour créer des règles d’appartenance dynamique

**Type :** Fonctionnalité modifiée **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Grâce à cette mise à jour, vous pouvez désormais cliquer sur le lien **Get custom extension properties** (Obtenir les propriétés d’extension personnalisées) du générateur de règles de groupe d’utilisateurs dynamiques, entrer votre ID d’application unique et recevoir la liste complète des propriétés d’extension personnalisées à utiliser lors de la création d’une règle d’appartenance dynamique pour les utilisateurs. Vous pouvez également actualiser cette liste afin d’obtenir les nouvelles propriétés d’extension personnalisées pour cette application.

Pour plus d’informations sur l’utilisation de propriétés d’extension personnalisées pour les règles d’appartenance dynamiques, consultez la section [Attributs d’extension et propriétés d’extension personnalisée](../enterprise-users/groups-dynamic-membership.md#extension-properties-and-custom-extension-properties).

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nouvelles applications clientes approuvées pour l’accès conditionnel basé sur les applications Azure AD

**Type :** Modification planifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Les applications suivantes se trouvent sur la liste des [applications clientes approuvées](../conditional-access/concept-conditional-access-conditions.md#client-apps) :

- Microsoft To-Do

- Microsoft Stream

Pour plus d'informations, consultez les pages suivantes :

- [Accès conditionnel basé sur les applications Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="new-support-for-self-service-password-reset-from-the-windows-7881-lock-screen"></a>Nouvelle prise en charge de la réinitialisation de mot de passe en libre-service à partir de l’écran de verrouillage Windows 7/8/8.1

**Type :** Nouvelle fonctionnalité **Catégorie de service :** SSPR **Fonctionnalité produit :** Authentification utilisateur

Une fois que vous avez configuré cette nouvelle fonctionnalité, vos utilisateurs voient s’afficher un lien leur permettant de réinitialiser leur mot de passe à partir de l’écran **Verrouiller** d’un appareil exécutant Windows 7, Windows 8 ou Windows 8.1. En cliquant sur ce lien, l’utilisateur est orienté vers le même flux de réinitialisation de mot de passe que par le biais du navigateur web.

Pour plus d’informations, consultez l’article [How to enable password reset from Windows 7, 8, and 8.1](../authentication/howto-sspr-windows.md) (Activation de la réinitialisation de mot de passe à partir de Windows 7, 8 et 8.1).

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>Notification de modification : Les codes d’autorisation ne pourront plus être réutilisés

**Type :** Modification planifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

À compter du 15 novembre 2018, Azure AD n’acceptera plus les codes d’authentification utilisés pour les applications. Cette modification de sécurité a pour but d’assurer la conformité d’Azure AD avec la spécification OAuth et s’appliquera aux points de terminaison v1 et v2.

Si votre application réutilise des codes d’autorisation pour obtenir des jetons pour différentes ressources, nous vous recommandons d’utiliser le code pour obtenir un jeton d’actualisation, puis d’utiliser ce jeton d’actualisation pour acquérir des jetons supplémentaires pour d’autres ressources. Les codes d’autorisation ne peuvent être utilisés qu’une seule fois, tandis que les jetons d’actualisation peuvent être utilisés plusieurs fois sur différentes ressources. Toute application qui tente de réutiliser un code d’authentification pendant le flux de code OAuth obtient une erreur invalid_grant.

Pour découvrir les modifications liées à ce protocole et à d’autres protocoles, consultez la [liste complète des nouveautés en matière d’authentification](../develop/reference-breaking-changes.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---september-2018"></a>Nouvelles applications fédérées disponibles dans la galerie d’applications Azure AD - Septembre 2018

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En septembre 2018, nous avons ajouté à notre galerie d’applications les 16 nouvelles applications ci-après qui prennent en charge la fédération :

[Uberflip](../saas-apps/uberflip-tutorial.md), [Comeet Recruiting Software](../saas-apps/comeetrecruitingsoftware-tutorial.md), [Workteam](../saas-apps/workteam-tutorial.md), [ArcGIS Enterprise](../saas-apps/arcgisenterprise-tutorial.md), [Nuclino](../saas-apps/nuclino-tutorial.md), [JDA Cloud](../saas-apps/jdacloud-tutorial.md), [Snowflake](../saas-apps/snowflake-tutorial.md), NavigoCloud, [Figma](../saas-apps/figma-tutorial.md), join.me, [ZephyrSSO](../saas-apps/zephyrsso-tutorial.md), [Silverback](../saas-apps/silverback-tutorial.md), Riverbed Xirrus EasyPass, [Rackspace SSO](../saas-apps/rackspacesso-tutorial.md), Enlyft SSO for Azure, SurveyMonkey, [Convene](../saas-apps/convene-tutorial.md), [dmarcian](../saas-apps/dmarcian-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="support-for-additional-claims-transformations-methods"></a>Prise en charge de méthodes de transformation de revendication supplémentaires

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Nous avons introduit de nouvelles méthodes de transformation de revendication, ToLower() et ToUpper(), qui sont applicables aux jetons SAML à partir de la page **Configuration de l’authentification unique** basée sur SAML.

Pour plus d’informations, consultez l’article [Procédure : personnaliser des revendications émises dans le jeton SAML pour les applications d’entreprise dans Azure AD](../develop/active-directory-saml-claims-customization.md).

---

### <a name="updated-saml-based-app-configuration-ui-preview"></a>Mise à jour de l’interface utilisateur de configuration d’applications basées sur SAML (préversion)

**Type :** Fonctionnalité modifiée **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

L’interface utilisateur de configuration d’applications basées sur SAML que nous avons mise à jour vous offre :

- une expérience améliorée de configuration pas à pas de vos applications SAML ;

- un surcroît de visibilité sur les éléments manquants ou incorrects de votre configuration ;

- la possibilité d’ajouter plusieurs adresses e-mail pour la notification d’expiration du certificat ;

- de nouvelles méthodes de transformation de revendication, ToLower() et ToUpper(), et bien plus encore ;

- un moyen de charger votre propre certificat de signature de jetons pour vos applications d’entreprise ;

- la possibilité de définir le format NameID pour les applications SAML et de définir la valeur NameID sous forme d’Extensions d’annuaire.

Pour activer cet affichage mis à jour, cliquez sur le lien **Essayer la nouvelle expérience** en haut de la page **Authentification unique**. Pour plus d’informations, consultez [Didacticiel : Configurez l’authentification unique SAML pour une application avec Azure Active Directory](../manage-apps/view-applications-portal.md).

---

## <a name="august-2018"></a>Août 2018

### <a name="changes-to-azure-active-directory-ip-address-ranges"></a>Modifications apportées aux plages d’adresses IP Azure Active Directory

**Type :** Modification planifiée **Catégorie de service :** Autre **Fonctionnalité produit :** Plateforme

Nous sommes en train d’introduire dans Azure AD des plages d’adresses IP plus vastes. Si vous avez configuré des plages d’adresses IP Azure AD pour vos pare-feu, routeurs ou groupes de sécurité réseau, vous allez donc devoir les mettre à jour. Nous procédons à cette mise à jour afin que vous n’ayez pas à modifier à nouveau les configurations de plage IP de votre pare-feu, routeur ou groupe de sécurité réseau lorsque Azure AD ajoute de nouveaux points de terminaison.

Le trafic réseau va migrer vers ces nouvelles plages au cours des deux prochains mois. Pour ne pas interrompre le service, vous devez ajouter ces nouvelles valeurs à vos adresses IP avant le 10 septembre 2018 :

- 20.190.128.0/18

- 40.126.0.0/18

Nous vous recommandons vivement de ne pas supprimer les anciennes plages d’adresses IP tant que votre trafic réseau n’a pas été entièrement migré vers les nouvelles plages. Pour obtenir des informations sur la migration et savoir quand vous pouvez supprimer les anciennes plages, consultez [URL et plages d’adresses IP Office 365](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2).

---

### <a name="change-notice-authorization-codes-will-no-longer-be-available-for-reuse"></a>Notification de modification : Les codes d’autorisation ne pourront plus être réutilisés

**Type :** Modification planifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

À compter du 15 novembre 2018, Azure AD n’acceptera plus les codes d’authentification utilisés pour les applications. Cette modification de sécurité a pour but d’assurer la conformité d’Azure AD avec la spécification OAuth et s’appliquera aux points de terminaison v1 et v2.

Si votre application réutilise des codes d’autorisation pour obtenir des jetons pour différentes ressources, nous vous recommandons d’utiliser le code pour obtenir un jeton d’actualisation, puis d’utiliser ce jeton d’actualisation pour acquérir des jetons supplémentaires pour d’autres ressources. Les codes d’autorisation ne peuvent être utilisés qu’une seule fois, tandis que les jetons d’actualisation peuvent être utilisés plusieurs fois sur différentes ressources. Toute application qui tente de réutiliser un code d’authentification pendant le flux de code OAuth obtient une erreur invalid_grant.

Pour découvrir les modifications liées à ce protocole et à d’autres protocoles, consultez la [liste complète des nouveautés en matière d’authentification](../develop/reference-breaking-changes.md).

---

### <a name="converged-security-info-management-for-self-service-password-sspr-and-multi-factor-authentication-mfa"></a>Informations de sécurité convergées pour le mot de passe en libre-service (SSPR) et l’authentification multifacteur (MFA)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** SSPR **Fonctionnalité produit :** Authentification utilisateur

Cette nouvelle fonctionnalité permet de gérer les informations de sécurité (numéro de téléphone, application mobile, etc.) pour SSPR et MFA de manière centralisée, alors qu’elles étaient auparavant dispersées à deux emplacements différents.

Cette centralisation fonctionne également pour les personnes utilisant soit SSPR, soit MFA. En outre, si votre entreprise n’impose pas l’inscription MFA ou SSPR, les utilisateurs peuvent toujours inscrire les méthodes d’informations de sécurité MFA ou SSPR autorisées par leur entreprise à partir du portail My Apps.

Il s’agit d’une inscription à une version préliminaire publique. Les administrateurs peuvent activer la nouvelle expérience (le cas échéant) pour un groupe sélectionné ou tous les utilisateurs d’un client. Pour plus d’informations sur cette centralisation, consultez le [blog Converged experience](https://cloudblogs.microsoft.com/enterprisemobility/2018/08/06/mfa-and-sspr-updates-now-in-public-preview/).

---

### <a name="new-http-only-cookies-setting-in-azure-ad-application-proxy-apps"></a>Nouveau paramètre Cookies HTTP uniquement dans les applications de proxy d’application Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Il existe un nouveau paramètre nommé **Cookies HTTP uniquement** dans vos applications de proxy d’application. Ce paramètre assure une sécurité supplémentaire en incluant l’indicateur HTTPOnly dans l’en-tête de réponse HTTP des cookies de session et d’accès au proxy d’application, en bloquant l’accès au cookie depuis un script côté client et en empêchant des actions telles que la copie ou la modification du cookie. Bien que cet indicateur n’ait jamais été utilisé auparavant, vos cookies ont toujours été chiffrés et transmis au moyen d’une connexion TLS pour vous protéger contre les modifications incorrectes.

Ce paramètre n’est pas compatible avec les applications utilisant les contrôles ActiveX, telles que Bureau à distance. Le cas échéant, nous vous recommandons de désactiver ce paramètre.

Pour plus d’informations sur le paramètre Cookies HTTP uniquement, voir [Publier des applications avec le proxy d’application Azure AD](../manage-apps/application-proxy-add-on-premises-application.md).

---

### <a name="privileged-identity-management-pim-for-azure-resources-supports-management-group-resource-types"></a>Privileged Identity Management (PIM) pour les ressources Azure prend en charge les types de ressources Groupe d’administration

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Les paramètres d’activation et d’attribution juste-à-temps peuvent désormais être appliqués aux types de ressources Groupe d’administration, comme vous le faites déjà pour les abonnements, les groupes de ressources et les ressources (telles que les machines virtuelles, App Services et bien plus encore). En outre, toute personne disposant d’un rôle donnant un accès administrateur pour un Groupe d’administration peut découvrir et gérer cette ressource dans PIM.

Pour plus d’informations sur PIM et les ressources Azure, voir [Découvrir et gérer les ressources Azure à l’aide de Privileged Identity Management](../privileged-identity-management/pim-resource-roles-discover-resources.md).

---

### <a name="application-access-preview-provides-faster-access-to-the-azure-ad-portal"></a>L’accès à l’application (version préliminaire) assure un accès plus rapide au portail Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Aujourd'hui, lors de l’activation d’un rôle avec PIM, l’application des autorisations peut prendre jusqu'à 10 minutes. Si vous choisissez d’utiliser l’accès Application, qui est actuellement en version préliminaire publique, les administrateurs peuvent accéder au portail Azure AD dès l’envoi de la demande d’activation.

Actuellement, l’accès Application prend uniquement en charge l’utilisation du portail Azure AD et les ressources Azure. Pour plus d’informations sur PIM et l’accès Application, voir [Qu’est-ce qu’Azure AD Privileged Identity Management ?](../privileged-identity-management/pim-configure.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---august-2018"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App - Août 2018

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En août 2018, nous avons ajouté à notre galerie d’applications les 16 applications suivantes qui prennent en charge la fédération :

[Hornbill](../saas-apps/hornbill-tutorial.md), [Bridgeline Unbound](../saas-apps/bridgelineunbound-tutorial.md), [Sauce Labs - Mobile and Web Testing](../saas-apps/saucelabs-mobileandwebtesting-tutorial.md), [Meta Networks Connector](../saas-apps/metanetworksconnector-tutorial.md), [Way We Do](../saas-apps/waywedo-tutorial.md), [Spotinst](../saas-apps/spotinst-tutorial.md), [ProMaster (par Inlogik)](../saas-apps/promaster-tutorial.md), SchoolBooking, [4me](../saas-apps/4me-tutorial.md), [Dossier](../saas-apps/dossier-tutorial.md), [N2F - Expense reports](../saas-apps/n2f-expensereports-tutorial.md), [Comm100 Live Chat](../saas-apps/comm100livechat-tutorial.md), [SafeConnect](../saas-apps/safeconnect-tutorial.md), [ZenQMS](../saas-apps/zenqms-tutorial.md), [eLuminate](../saas-apps/eluminate-tutorial.md), [Dovetale](../saas-apps/dovetale-tutorial.md).

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="native-tableau-support-is-now-available-in-azure-ad-application-proxy"></a>La prise en charge native de Tableau est désormais disponible dans le proxy d’application Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Grâce à notre mise à jour d’OpenID Connect vers le protocole OAuth 2.0 Code Grant pour notre protocole d’authentification préalable, vous n’avez plus de configuration supplémentaire à effectuer pour utiliser Tableau avec le proxy d’application. Ce changement de protocole aide également le proxy d’application à mieux prendre en charge les applications plus récentes en utilisant uniquement les redirections HTTP, qui sont couramment prises en charge dans les balises HTML et JavaScript.

---

### <a name="new-support-to-add-google-as-an-identity-provider-for-b2b-guest-users-in-azure-active-directory-preview"></a>Nouvelle prise en charge pour ajouter Google comme fournisseur d’identité pour les utilisateurs invités B2B dans Azure Active Directory (version préliminaire)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2B **Fonctionnalité produit :** B2B/B2C

En configurant la fédération avec Google dans votre organisation, vous pouvez laisser les utilisateurs Gmail invités se connecter à vos applications et ressources partagées à l’aide de leur compte Google, sans qu’ils aient besoin de créer un compte Microsoft (compte de service administré ou MSA) personnel ou un compte Azure AD.

Il s’agit d’une inscription à une version préliminaire publique. Pour plus d’informations sur la fédération Google, voir [Ajouter Google comme fournisseur d’identité pour les utilisateurs invités B2B](../external-identities/google-federation.md).

---

## <a name="july-2018"></a>Juillet 2018

### <a name="improvements-to-azure-active-directory-email-notifications"></a>Améliorations apportées aux notifications par e-mail Azure Active Directory

**Type :** Fonctionnalité modifiée **Catégorie de service :** Autre **Fonctionnalité produit :** Gestion du cycle de vie des identités

Azure Active Directory (Azure AD) a une nouvelle conception visuelle, ainsi qu’un nouvel expéditeur, avec une nouvelle adresse e-mail et un nouveau nom d’affichage, lorsqu’ils proviendront des services suivants :

- Révisions d’accès Azure AD
- Azure AD Connect Health
- Azure AD Identity Protection
- Azure AD Privileged Identity Management
- Notifications de l’expiration prochaine des certificats des applications d’entreprise
- Notifications du service de provisionnement des applications d’entreprise

Les notifications par e-mail proviendront de l’adresse e-mail et du nom d’affichage suivants :

- Adresse e-mail : azure-noreply@microsoft.com
- Nom complet : Microsoft Azure

Pour voir un exemple de nouvelle notification et obtenir plus d’informations, consultez [Notifications par e-mail dans Azure AD Privileged Identity Management](../privileged-identity-management/pim-email-notifications.md).

---

### <a name="azure-ad-activity-logs-are-now-available-through-azure-monitor"></a>Les journaux d’activité Azure AD sont désormais disponibles via Azure Monitor

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Les journaux d’activité Azure AD sont désormais disponibles en version préliminaire publique pour Azure Monitor (service de supervision à l’échelle de la plateforme d’Azure). Azure Monitor vous offre une rétention à long terme et une intégration transparente, en plus de ces améliorations :

- Rétention à long terme par le routage de vos fichiers journaux sur votre propre compte de stockage Azure.

- Intégration transparente de SIEM, sans avoir à écrire ou gérer des scripts personnalisés.

- Intégration transparente avec vos propres solutions personnalisées, outils d’analytique ou solutions de gestion des incidents.

Pour plus d’informations sur ces nouvelles fonctionnalités, consultez notre blog [Les journaux d’activité d’Azure AD dans Azure Monitor diagnostics est désormais disponible en version préliminaire publique](https://cloudblogs.microsoft.com/enterprisemobility/2018/07/26/azure-ad-activity-logs-in-azure-monitor-diagnostics-now-in-public-preview/) et notre documentation, [Journaux d’activité d’Azure Active Directory dans Azure Monitor (préversion)](../reports-monitoring/concept-activity-logs-azure-monitor.md).

---

### <a name="conditional-access-information-added-to-the-azure-ad-sign-ins-report"></a>Informations d’accès conditionnel ajoutées au rapport de connexion Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Protection et sécurité des identités

Cette mise à jour vous permet de voir quelles stratégies sont évaluées lorsqu’un utilisateur se connecte, ainsi que le résultat de la stratégie. En outre, le rapport inclut désormais le type d’application client utilisé par l’utilisateur, pour que vous puissiez identifier le trafic de protocole hérité. Les entrées de rapport peuvent également désormais rechercher un ID de corrélation, qui se trouve dans le message d’erreur visible par l’utilisateur et peuvent être utilisées pour identifier et résoudre les problèmes de la requête de connexion correspondante.

---

### <a name="view-legacy-authentications-through-sign-ins-activity-logs"></a>Afficher les authentifications héritées via les journaux d’activité de connexions

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Avec le nouveau champ **Application cliente** des journaux d’activité de connexions, les clients peuvent désormais voir les utilisateurs qui utilisent des authentifications héritées. Les clients peuvent accéder à ces informations à l’aide de l’API Microsoft Graph Connexions ou via les journaux d’activité de connexions dans le portail Azure AD, et ainsi utiliser la commande **Application cliente** pour filtrer les authentifications héritées. Pour plus de détails, consultez la documentation.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---july-2018"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App - Juillet 2018

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En juillet 2018, nous avons ajouté à notre galerie d’applications les 16 applications suivantes qui prennent en charge la fédération :

[Hub de l’innovation](../saas-apps/innovationhub-tutorial.md), [Leapsome](../saas-apps/leapsome-tutorial.md), [Authentification unique pour certains administrateurs](../saas-apps/certainadminsso-tutorial.md), mise en lots PSUC, [iPass SmartConnect](../saas-apps/ipasssmartconnect-tutorial.md), [Screencast-O-Matic](../saas-apps/screencast-tutorial.md), Classe unifiée de PowerSchool, [intégration Eli](../saas-apps/elionboarding-tutorial.md), [prise en charge à distance Bomgar](../saas-apps/bomgarremotesupport-tutorial.md), [Nimblex](../saas-apps/nimblex-tutorial.md), [Imagineer WebVision](../saas-apps/imagineerwebvision-tutorial.md), [Insight4GRC](../saas-apps/insight4grc-tutorial.md), [Connecteur SecureW2 JoinNow](../saas-apps/securejoinnow-tutorial.md), [Kanbanize](../saas-apps/kanbanize-tutorial.md), [SmartLPA](../saas-apps/smartlpa-tutorial.md), [Base de compétences](../saas-apps/skillsbase-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-user-provisioning-saas-app-integrations---july-2018"></a>Nouvelles intégrations d’applications SaaS pour l’attribution d’utilisateurs - juillet 2018

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Intégration tierce

Azure AD vous permet d’automatiser la création, la maintenance et la suppression d’identités utilisateur dans des applications SaaS comme Dropbox, Salesforce, ServiceNow, etc. En juillet 2018, nous avons ajouté une prise en charge de l’attribution des utilisateurs pour les applications suivantes dans la galerie d’applications Azure AD :

- [Cisco WebEx](../saas-apps/cisco-webex-provisioning-tutorial.md)

- [Bonusly](../saas-apps/bonusly-provisioning-tutorial.md)

Pour obtenir la liste de toutes les applications qui prennent en charge l’attribution d’utilisateurs dans la galerie Azure AD, consultez [Intégration d’application SaaS avec Azure Active Directory](../saas-apps/tutorial-list.md).

---

### <a name="connect-health-for-sync---an-easier-way-to-fix-orphaned-and-duplicate-attribute-sync-errors"></a>Connect Health pour la synchronisation - un moyen plus simple pour corriger les erreurs de synchronisation d’attribut en double et orphelins

**Type :** Nouvelle fonctionnalité **Catégorie de service :** AD Connect **Fonctionnalité produit :** Monitoring et création de rapports

Azure AD Connect Health introduit la mise à jour du libre-service pour vous aider à mettre en surbrillance et corriger les erreurs de synchronisation. Cette fonctionnalité résout les erreurs de synchronisation d’attribut en double et corrige les objets qui sont orphelins à partir d’Azure AD. Ce diagnostic offre les avantages suivants :

- Permet de réduire les erreurs de synchronisation d’attribut en double, en fournissant des corrections spécifiques

- Applique une correction pour les scénarios Azure AD dédiés, résolution des erreurs en une seule étape

- Aucune mise à niveau ou configuration n’est nécessaire pour démarrer et utiliser cette fonctionnalité

Pour plus d’informations, consultez [Diagnostiquer et corriger les erreurs de synchronisation d’attribut en double](../hybrid/how-to-connect-health-diagnose-sync-errors.md)

---

### <a name="visual-updates-to-the-azure-ad-and-msa-sign-in-experiences"></a>Mises à jour de Visual à Azure AD et expériences de connexion MSA

**Type :** Fonctionnalité modifiée **Catégorie de service :** Azure AD **Fonctionnalité produit :** Authentification utilisateur

Nous mettons à jour l’interface utilisateur pour l’expérience de connexion des services en ligne de Microsoft, tels qu’Office 365 et Azure. Cette modification rend les écrans plus simples et moins encombrés. Pour plus d’informations sur cette modification, consultez le blog [Améliorations à venir à l’expérience de connexion Azure AD](https://cloudblogs.microsoft.com/enterprisemobility/2018/04/04/upcoming-improvements-to-the-azure-ad-sign-in-experience/).

---

### <a name="new-release-of-azure-ad-connect---july-2018"></a>Nouvelle version d’Azure AD Connect - juillet 2018

**Type :** Fonctionnalité modifiée **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Gestion du cycle de vie des identités

La dernière version d’Azure AD Connect comprend :

- Correctifs de bogues et mises à jour de la prise en charge

- Disponibilité générale de l’intégration Ping Federate

- Mises à jour de la dernière SQL 2012 du client

Pour plus d’informations sur cette mise à jour, consultez [Azure AD Connect : Historique de publication des versions](../hybrid/reference-connect-version-history.md)

---

### <a name="updates-to-the-terms-of-use-end-user-ui"></a>Mises à jour de l’interface utilisateur des conditions d’utilisation

**Type :** Fonctionnalité modifiée **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Gouvernance

Nous mettons à jour la chaîne de l’acceptation de l’interface utilisateur des conditions d’utilisation.

**Texte actif** Vous devez accepter les conditions d'utilisation pour accéder aux ressources [tenantName].<br>**Nouveau texte.** Vous devez lire les conditions d'utilisation pour accéder à la ressource [tenantName].

**Texte actuel :** Choosing to accept means that you agree to all of the above terms of use. (Accepter signifie que vous acceptez les conditions d’utilisation ci-dessus.)<br>**Nouveau texte :** Please click Accept to confirm that you have read and understood the terms of use. (Veuillez cliquer sur Accepter pour confirmer que vous avez lu et compris les conditions d’utilisation.)

---

### <a name="pass-through-authentication-supports-legacy-protocols-and-applications"></a>L’authentification directe prend en charge les applications et les protocoles hérités

**Type :** Fonctionnalité modifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

L’authentification directe prend en charge les applications et les protocoles hérités. Les limitations suivantes sont maintenant entièrement prises en charge :

- L’utilisateur se connecte aux applications clientes Office héritées, Office 2010 et Office 2013 sans exiger une authentification moderne.

- Accédez aux informations de disponibilité et de partage de calendrier dans des environnements hybrides Exchange sur Office 2010 uniquement.

- L’utilisateur se connecte à des applications clientes Skype Entreprise, sans exiger une authentification moderne.

- L'utilisateur se connecte à PowerShell version 1.0.

- Le programme d’inscription des appareils Apple (Apple DEP) à l’aide de l’Assistant de configuration iOS.

---

### <a name="converged-security-info-management-for-self-service-password-reset-and-multi-factor-authentication"></a>Informations de sécurité convergées pour la réinitialisation de mot de passe en libre-service et l’authentification multifacteur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** SSPR **Fonctionnalité produit :** Authentification utilisateur

Cette nouvelle fonctionnalité permet aux utilisateurs de gérer leurs informations de sécurité (par exemple, numéro de téléphone, adresse de messagerie, application mobile, etc) pour la réinitialisation de mot de passe libre-service (SSPR) et l’authentification multifacteur (MFA) dans une seule expérience. Les utilisateurs ne devront plus inscrire les mêmes informations de sécurité pour SSPR et MFA dans deux expériences différentes. Cette nouvelle expérience s’applique également aux utilisateurs qui dispose de SSPR ou MFA.

Si une organisation ne respecte pas l’inscription MFA ou SSPR, les utilisateurs peuvent inscrire leurs informations de sécurité via le portail **Mes applications**. À partir de là, les utilisateurs peuvent inscrire des méthodes activées pour MFA ou SSPR.

Il s’agit d’une inscription à une version préliminaire publique. Les administrateurs peuvent activer la nouvelle expérience (le cas échéant) pour un groupe sélectionné d’utilisateurs ou de tous les utilisateurs d’un client.

---

### <a name="use-the-microsoft-authenticator-app-to-verify-your-identity-when-you-reset-your-password"></a>Utilisez l’application Microsoft Authenticator pour vérifier votre identité lorsque vous réinitialisez votre mot de passe

**Type :** Fonctionnalité modifiée **Catégorie de service :** SSPR **Fonctionnalité produit :** Authentification utilisateur

Cette fonctionnalité permet aux non administrateurs de vérifier leur identité lors de la réinitialisation d’un mot de passe à l’aide d’une notification ou d’un code à partir de Microsoft Authenticator (ou toute autre application authentificatrice). Une fois que les administrateurs ont activé cette méthode de réinitialisation de mot de passe en libre-service, les utilisateurs qui ont inscrit une application mobile par le biais d’aka.ms/mfasetup ou d’aka.ms/setupsecurityinfo peuvent utiliser leur application mobile comme méthode de vérification lorsqu’ils réinitialisent leur mot de passe.

Les notifications d’application mobile peuvent uniquement être activées dans le cadre d’une stratégie qui nécessite deux méthodes pour réinitialiser votre mot de passe.

---

## <a name="june-2018"></a>Juin 2018

### <a name="change-notice-security-fix-to-the-delegated-authorization-flow-for-apps-using-azure-ad-activity-logs-api"></a>Notification de modification : Correctif de sécurité du flux d’autorisation déléguée pour les applications qui utilisent les API de journaux d’activité Azure AD

**Type :** Modification planifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

En raison du renforcement de la sécurité, nous avons dû modifier les autorisations pour les applications qui utilisent un flux d’autorisation déléguée pour accéder aux [API de journal d’activité Azure AD](../reports-monitoring/concept-reporting-api.md). Cette modification prendra effet d’ici **le 26 juin 2018**.

Si l’une de vos applications utilise des API de journal d’activité Azure AD, procédez comme suit pour vous assurer que l’application ne s’arrête pas après l’application de la modification.

**Pour mettre à jour les autorisations de votre application**

1. Connectez-vous au portail Azure, sélectionnez **Azure Active Directory**, puis **Inscriptions des applications**.
2. Sélectionnez l’application qui utilise l’API Journal d’activité Azure AD, sélectionnez **Paramètres**, **Autorisations requises**, puis l’API **Microsoft Azure Active Directory**.
3. Dans la zone **Autorisations déléguées** du panneau **Activer l’accès**, cochez la case à côté **Lire les données d'annuaire**, puis sélectionnez **Enregistrer**.
4. Sélectionnez **Accorder des autorisations**, puis **Oui**.

    >[!Note]
    >Vous devez être un administrateur général pour accorder des autorisations à l’application.

Pour plus d’informations, consultez la section [Accorder des autorisations](../reports-monitoring/howto-configure-prerequisites-for-reporting-api.md#grant-permissions) des prérequis pour accéder à l’article concernant API de création de rapports Azure AD.

---

### <a name="configure-tls-settings-to-connect-to-azure-ad-services-for-pci-dss-compliance"></a>Configurer les paramètres TLS de connexion aux services Azure AD pour la conformité PCI DSS

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Non applicable **Fonctionnalité produit :** Plateforme

Le protocole TLS (Transport Layer Security) garantit la confidentialité et l’intégrité des données entre deux applications qui communiquent. Il s’agit du protocole de sécurité le plus largement utilisé aujourd’hui.

Le [Conseil des normes de sécurité PCI](https://www.pcisecuritystandards.org/) a déterminé que les premières versions des protocoles TLS et SSL (Secure Sockets Layer) doivent être désactivées en faveur de nouveaux protocoles d’application plus sécurisés, dont l’utilisation est obligatoire depuis le **30 juin 2018**. Cela signifie que si vous vous connectez à des services Azure AD et que vous avez besoin de la conformité PCI DSS, vous devez désactiver TLS 1.0. Plusieurs versions de TLS sont disponibles, toutefois, TLS 1.2 est la version la plus récente disponible pour Azure Active Directory. Nous vous recommandons de passer directement à TLS 1.2, à la fois pour les combinaisons client/serveur et navigateur/serveur.

Les navigateurs obsolètes risquent de ne pas prendre en charge les versions les plus récentes de TLS, telles que TLS 1.2. Pour savoir quelles versions de TLS sont prises en charge par votre navigateur, accédez au site [Qualys SSL Labs](https://www.ssllabs.com/), puis cliquez sur **Test your browser** (Testez votre navigateur). Nous vous recommandons de mettre à jour votre navigateur web vers la dernière version disponible et d’activer uniquement le protocole TLS 1.2.

**Pour activer le protocole TLS 1.2 (pour chaque navigateur)**

- **Microsoft Edge et Internet Explorer (les deux sont configurés à l’aide d’Internet Explorer)**

    1. Ouvrez Internet Explorer, sélectionnez **Outils** > **Options Internet** > **Avancé**.
    2. Dans la zone **Sécurité**, sélectionnez **Utiliser TLS 1.2**, puis sélectionnez **OK**.
    3. Fermez toutes les fenêtres du navigateur, puis redémarrez Internet Explorer.

- **Google Chrome**

    1. Ouvrez Google Chrome, tapez *chrome://settings/* dans la barre d’adresse, puis appuyez sur **Entrée**.
    2. Développez les options de la section **Avancé**, accédez à la zone **Système**, puis sélectionnez **Ouvrir les paramètres proxy**.
    3. Dans la boîte de dialogue **Propriétés Internet**, sélectionnez l’onglet **Avancé**, accédez à la zone **Sécurité**, sélectionnez **Utiliser TLS 1.2**, puis sélectionnez **OK**.
    4. Fermez toutes les fenêtres du navigateur, puis redémarrez Google Chrome.

- **Mozilla Firefox**

    1. Ouvrez Firefox, tapez *about:config* dans la barre d’adresses, puis appuyez sur **Entrée**.
    2. Lancez une recherche sur *TLS*, puis sélectionnez l’entrée **security.tls.version.max**.
    3. Définissez la valeur sur **3** pour forcer le navigateur à utiliser la version TLS 1.2, puis sélectionnez **OK**.

        >[!NOTE]
        >Firefox version 60.0 prend en charge TLS 1.3, vous pouvez donc également définir la valeur de security.tls.version.max sur **4**.

    4. Fermez toutes les fenêtres du navigateur, puis redémarrez Mozilla Firefox.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---june-2018"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App - Juin 2018

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En juin 2018, nous avons ajouté à notre galerie d’applications les 15 applications suivantes qui prennent en charge la fédération :

[Skytap](../saas-apps/skytap-tutorial.md), [Settling music](../saas-apps/settlingmusic-tutorial.md), [SAML 1.1 Token enabled LOB App](../saas-apps/saml-tutorial.md), [Supermood](../saas-apps/supermood-tutorial.md), [Autotask](../saas-apps/autotaskendpointbackup-tutorial.md), [Endpoint Backup](../saas-apps/autotaskendpointbackup-tutorial.md), [Skyhigh Networks](../saas-apps/skyhighnetworks-tutorial.md), Smartway2, [TonicDM](../saas-apps/tonicdm-tutorial.md), [Moconavi](../saas-apps/moconavi-tutorial.md), [Zoho One](../saas-apps/zohoone-tutorial.md), [SharePoint on-premises](../saas-apps/sharepoint-on-premises-tutorial.md), [ForeSee CX Suite](../saas-apps/foreseecxsuite-tutorial.md), [Vidyard](../saas-apps/vidyard-tutorial.md), [ChronicX](../saas-apps/chronicx-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md). Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="azure-ad-password-protection-is-available-in-public-preview"></a>Protection des mots de passe Azure AD disponible en préversion publique

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Authentification utilisateur

Utilisez la protection des mots de passe Azure AD pour éliminer les mots de passe faciles à deviner de votre environnement. L’élimination de ces mots de passe permet de réduire le risque de compromission due à une attaque par pulvérisation de mots de passe.

Voici, plus précisément, ce que permet la protection des mots de passe Azure AD :

- Protège les comptes de votre organisation, à la fois dans Azure AD et dans Windows Server Active Directory (AD)
- Empêche vos utilisateurs d’utiliser les 500 mots de passe les plus couramment utilisés, ainsi que plus d’un million de variations de ces mots de passe
- Permet de gérer la protection des mots de passe Azure AD à partir d’un même emplacement du portail Azure AD, à la fois pour Azure AD et pour les instances locales de Windows Server Active Directory.

Pour plus d’informations sur la protection des mots de passe Azure AD, consultez [Éliminer les mots de passe incorrects de votre organisation](../authentication/concept-password-ban-bad.md).

---

### <a name="new-all-guests-conditional-access-policy-template-created-during-terms-of-use-creation"></a>Nouveau modèle de stratégie d’accès conditionnel « tous les invités » créé en même temps que les conditions d’utilisation

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Gouvernance

Lors de la création de vos conditions d’utilisation, un modèle de stratégie d’accès conditionnel est créé pour « tous les invités » et « toutes les applications ». Ce nouveau modèle de stratégie applique les conditions d’utilisation qui viennent d’être créées, ce qui simplifie les processus de création et de mise en œuvre pour les invités.

Pour plus d’informations, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md).

---

### <a name="new-custom-conditional-access-policy-template-created-during-terms-of-use-creation"></a>Nouveau modèle de stratégie d’accès conditionnel « personnalisé » créé en même temps que les conditions d’utilisation

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Gouvernance

Lors de la création de vos conditions d’utilisation, un modèle de stratégie d’accès conditionnel « personnalisé » est également créé. Ce nouveau modèle de stratégie vous permet de créer les conditions d’utilisation, puis de passer immédiatement au panneau de création de la stratégie d’accès conditionnel, sans devoir accéder manuellement dans le portail.

Pour plus d’informations, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md).

---

### <a name="new-and-comprehensive-guidance-about-deploying-azure-ad-multi-factor-authentication"></a>Nouvelles instructions plus complètes sur le déploiement d’Azure AD Multi-Factor Authentication

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Autre **Fonctionnalité produit :** Protection et sécurité des identités

Nous avons publié de nouvelles instructions étape par étape sur le déploiement d’Azure AD Multi-Factor Authentication (MFA) dans une organisation.

Pour consulter le guide de déploiement MFA, accédez au dépôt GitHub [Identity Deployment Guides](./active-directory-deployment-plans.md) (Guides de déploiement des identités). Pour envoyer vos commentaires sur les guides de déploiement, utilisez le [formulaire de commentaires Plan de déploiement](https://aka.ms/deploymentplanfeedback). Si vous avez des questions sur les guides de déploiement, contactez-nous ici : [IDGitDeploy](mailto:idgitdeploy@microsoft.com).

---

### <a name="azure-ad-delegated-app-management-roles-are-in-public-preview"></a>Rôles délégués de gestion des applications Azure AD en préversion publique

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Contrôle d’accès

Les administrateurs peuvent désormais déléguer des tâches de gestion des applications sans avoir à affecter le rôle Administrateur général. Les nouveaux rôles et nouvelles fonctionnalités sont les suivants :

- **Nouveaux rôles d’administrateur Azure AD standard :**

    - **Administrateur d’application.** Permet de gérer tous les aspects des applications, y compris l’inscription, les paramètres d’authentification unique, les affectations et les licences d’applications, les paramètres de proxy d’application et le consentement (sauf pour les ressources Azure AD).

    - **Administrateur d’applications cloud.** Accorde tous les privilèges de l’administrateur d’applications, sauf pour le proxy d’application, car il ne fournit pas d’accès local.

    - **Développeur d’applications.** Permet d’inscrire des applications, même si l’option **Les utilisateurs peuvent inscrire des applications** est désactivée.

- **Propriété (configurée pour chaque inscription d’application et chaque application d’entreprise, comme pour la propriété de groupe) :**

    - **Propriétaire d’inscription d’application.** Permet de gérer tous les aspects de l’inscription d’une application avec propriétaire, y compris le manifeste d’application et l’ajout de propriétaires supplémentaires.

    - **Propriétaire d’application d’entreprise.** Permet de gérer de nombreux aspects relatifs aux applications d’entreprise avec propriétaire, y compris les paramètres d’authentification unique, les affectations d’applications et le consentement (sauf pour les ressources Azure AD).

Pour plus d’informations sur la préversion publique, consultez le blog [Azure AD delegated application management roles are in public preview!](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/13/hallelujah-azure-ad-delegated-application-management-roles-are-in-public-preview/) . Pour plus d’informations sur les rôles et les autorisations, consultez [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).

---

## <a name="may-2018"></a>Mai 2018

### <a name="expressroute-support-changes"></a>Modifications de la prise en charge d’ExpressRoute

**Type :** Modification planifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Plateforme

Les offres de SaaS, comme Azure Active Directory (Azure AD) sont conçues pour offrir de meilleures performances en accédant directement à Internet, sans passer par ExpressRoute ni par un autre tunnel VPN privé. Pour cette raison, à compter du **1er août 2018**, nous arrêtons la prise en charge d’ExpressRoute pour les services Azure AD qui utilisent le peering public Azure et les communautés Azure qui utilisent le peering Microsoft. Pour tous les services concernés par cette modification, vous remarquerez le passage progressif du trafic Azure AD d’ExpressRoute à Internet.

Même si nous modifions la prise en charge, nous savons qu’il existe toujours des situations où vous devrez peut-être utiliser un ensemble dédié de circuits pour le trafic d’authentification. Pour cette raison, Azure AD continue à prendre en charge les restrictions de plage d’adresses IP par abonné avec ExpressRoute et les services qui utilisent déjà le peering Microsoft avec la communauté « Autres services en ligne Office 365 ». Si vos services sont concernés, mais que vous avez besoin d’ExpressRoute, vous devez procéder comme suit :

- **Si vous utilisez le peering public Azure.** Passez au peering Microsoft et inscrivez-vous à la communauté **Autres services en ligne Office 365 (12076:5100)** . Pour plus d’informations sur le passage du peering public Azure au peering Microsoft, consultez l’article [Déplacer un peering public vers le peering Microsoft](../../expressroute/how-to-move-peering.md).

- **Si vous utilisez le peering Microsoft.** Inscrivez-vous à la communauté **Autres services en ligne Office 365 (12076:5100)** . Pour plus d’informations sur la configuration de routage requise, consultez la section [Prise en charge des communautés BGP](../../expressroute/expressroute-routing.md#bgp) de l’article Configuration requise pour le routage ExpressRoute.

Si vous devez continuer à utiliser des circuits dédiés, discutez avec l’équipe des comptes Microsoft au sujet de la façon d’obtenir l’autorisation d’utiliser la communauté **Autres services en ligne Office 365 (12076:5100)** . Le comité de révision géré par MS Office vérifiera si vous avez vraiment besoin de ces circuits et s’assurera que vous comprenez bien les implications techniques découlant de leur conservation. Les abonnements non autorisés qui essaient de créer des filtres de routage pour Office 365 reçoivent un message d’erreur.

---

### <a name="microsoft-graph-apis-for-administrative-scenarios-for-tou"></a>API Microsoft Graph pour les scénarios d’administration des conditions d’utilisation

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Expérience de développement

Nous avons ajouté l’API Microsoft Graph pour vous permettre de gérer les conditions d’utilisation d’Azure AD. Libre à vous ainsi de créer, mettre à jour et supprimer l’objet des conditions d’utilisation.

---

### <a name="add-azure-ad-multi-tenant-endpoint-as-an-identity-provider-in-azure-ad-b2c"></a>Ajouter un point de terminaison mutualisé Azure AD en tant que fournisseur d’identité dans Azure AD B2C

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

En vous aidant de stratégies personnalisées, vous pouvez maintenant ajouter le point de terminaison commun Azure AD en tant que fournisseur d’identité dans Azure AD B2C. Cela vous permet d’avoir un seul point d’entrée pour tous les utilisateurs Azure AD se connectant à vos applications. Pour plus d’informations, consultez [Azure Active Directory B2C : Autoriser la connexion d’utilisateurs à un fournisseur d’identité Azure AD mutualisé à l’aide de stratégies personnalisées](../../active-directory-b2c/identity-provider-azure-ad-multi-tenant.md).

---

### <a name="use-internal-urls-to-access-apps-from-anywhere-with-our-my-apps-sign-in-extension-and-the-azure-ad-application-proxy"></a>Utiliser des URL internes pour accéder aux applications où que vous vous trouviez avec l’extension de connexion Mes applications et le proxy d’application Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Mes applications **Fonctionnalité produit :** SSO

Les utilisateurs peuvent maintenant accéder aux applications via des URL internes, y compris lorsqu’ils se trouvent en dehors du réseau de l’entreprise. Il leur suffit pour cela d’utiliser l’extension de connexion sécurisée Mes applications pour Azure AD. Cela fonctionne avec toutes les d’application que vous avez publiées à l’aide du proxy d’application Azure AD, dans tous les navigateurs sur lesquels est installée une extension Panneau d’accès. Dès qu’un utilisateur se connecte à l’extension, la fonctionnalité de redirection d’URL est automatiquement activée. Vous pouvez télécharger cette extension pour [Microsoft Edge](https://go.microsoft.com/fwlink/?linkid=845176), [Chrome](https://go.microsoft.com/fwlink/?linkid=866367) et [Firefox](https://go.microsoft.com/fwlink/?linkid=866366).

---

### <a name="azure-active-directory---data-in-europe-for-europe-customers"></a>Azure Active Directory : données en Europe pour les clients européens

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Autre **Fonctionnalité produit :** GoLocal

Les clients européens veulent que leurs données restent en Europe. Ils ne veulent pas qu’elles soient répliquées en dehors des centres de données européens afin de respecter les lois européennes en vigueur et celles régissant leur vie privée. Cet [article](./active-directory-data-storage-eu.md) fournit des détails spécifiques sur les informations d’identité stockées en Europe ainsi que des informations complémentaires sur les données stockées en dehors des centres de données européens.

---

### <a name="new-user-provisioning-saas-app-integrations---may-2018"></a>Nouvelles intégrations d’applications SaaS pour l’attribution d’utilisateurs : mai 2018

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Intégration tierce

Azure AD vous permet d’automatiser la création, la maintenance et la suppression d’identités utilisateur dans des applications SaaS comme Dropbox, Salesforce, ServiceNow, etc. En mai 2018, nous avons ajouté une prise en charge de l’attribution des utilisateurs pour les applications suivantes dans la galerie d’applications Azure AD :

- [BlueJeans](../saas-apps/bluejeans-provisioning-tutorial.md)

- [Cornerstone OnDemand](../saas-apps/cornerstone-ondemand-provisioning-tutorial.md)

- [Zendesk](../saas-apps/zendesk-provisioning-tutorial.md)

Pour obtenir la liste de toutes les applications qui prennent en charge l’attribution d’utilisateurs dans la galerie Azure AD, consultez [https://aka.ms/appstutorial](../saas-apps/tutorial-list.md).

---

### <a name="azure-ad-access-reviews-of-groups-and-app-access-now-provides-recurring-reviews"></a>La fonction de révisions d’accès Azure AD des groupes et applications fournit maintenant des révisions récurrentes.

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Révisions d’accès **Fonctionnalité produit :** Gouvernance

La fonctionnalité de révision des accès de groupes et d’applications fait désormais partie de l’offre Azure AD Premium P2.  Les administrateurs peuvent configurer les révisions d’accès des appartenances à un groupe et des attributions d’applications de façon à ce qu’elles se répètent automatiquement à intervalles réguliers, par exemple tous les mois ou tous les trimestres.

---

### <a name="azure-ad-activity-logs-sign-ins-and-audit-are-now-available-through-ms-graph"></a>Des journaux d’activité Azure AD (connexions et audit) sont à présent disponibles via Microsoft Graph.

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Des journaux d’activité Azure AD (connexions et audit) sont à présent disponibles via l’API Microsoft Graph. Nous avons exposé deux points de terminaison via l’API Microsoft Graph pour accéder à ces journaux d’activité. Pour démarrer, consultez nos [documents](../reports-monitoring/concept-reporting-api.md) concernant l’accès par programmation aux API de création de rapports Azure AD.

---

### <a name="improvements-to-the-b2b-redemption-experience-and-leave-an-org"></a>Améliorations apportées à l’expérience d’échange B2B et à celle permettant de quitter une organisation

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2B **Fonctionnalité produit :** B2B/B2C

**Échange Juste-à-temps :** À partir du moment où vous partagez une ressource avec un utilisateur invité au moyen d’une API B2B, vous n’avez pas besoin de lui envoyer d’invitation par e-mail. Dans la plupart des cas, l’utilisateur invité peut accéder à la ressource. Il lui suffit de se laisser guider dans cette procédure d’échange. Qu’importe qu’il reçoive ou non l’e-mail. Vous n’avez plus besoin de demander à vos utilisateurs invités s’ils ont bien cliqué sur le lien d’échange que vous leur avez envoyé. Aussi, à partir du moment où SPO utilise le gestionnaire d’invitation, les pièces jointes dans le cloud peuvent avoir la même URL réglementaire pour tous les utilisateurs (internes et externes), quel que soit l’état de l’échange.

**Expérience d’échange moderne :** Fini les pages d’accueil d’échange avec fractionnement de l’écran. Les utilisateurs peuvent bénéficier d’une expérience moderne et accepter la déclaration de confidentialité de l’organisation à l’origine de l’invitation, comme ils le font déjà pour les applications tierces.

**Les utilisateurs invités peuvent quitter l’organisation :** Lorsque la relation entre les utilisateurs et l’organisation est terminée, ils peuvent quitter celle-ci d’eux-mêmes. Il n’ont plus besoin d’appeler l’administrateur de l’organisation qui les a invités pour demander leur suppression et n’ont pas non plus besoin d’ouvrir un ticket de support.

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery---may-2018"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App - Mai 2018

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En mai 2018, nous avons ajouté à notre galerie d’applications les 18 applications suivantes qui prennent en charge la fédération :

[AwardSpring](../saas-apps/awardspring-tutorial.md), Infogix Data3Sixty Govern, [Yodeck](../saas-apps/infogix-tutorial.md), [Jamf Pro](../saas-apps/jamfprosamlconnector-tutorial.md), [KnowledgeOwl](../saas-apps/knowledgeowl-tutorial.md), [Envi MMIS](../saas-apps/envimmis-tutorial.md), [LaunchDarkly](../saas-apps/launchdarkly-tutorial.md), [Adobe Captivate Prime](../saas-apps/adobecaptivateprime-tutorial.md), [Montage Online](../saas-apps/montageonline-tutorial.md), [まなびポケット](../saas-apps/manabipocket-tutorial.md), OpenReel, [Arc Publishing - SSO](../saas-apps/arc-tutorial.md), [PlanGrid](../saas-apps/plangrid-tutorial.md), [iWellnessNow](../saas-apps/iwellnessnow-tutorial.md), [Proxyclick](../saas-apps/proxyclick-tutorial.md), [Riskware](../saas-apps/riskware-tutorial.md), [Flock](../saas-apps/flock-tutorial.md), [Reviewsnap](../saas-apps/reviewsnap-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md).

Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="new-step-by-step-deployment-guides-for-azure-active-directory"></a>Nouveaux guides de déploiement étape par étape pour Azure Active Directory

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Autre **Fonctionnalité produit :** Répertoire

Nouveau guide étape par étape concernant le déploiement d’Azure Active Directory (Azure AD), notamment la réinitialisation de mot de passe libre-service, l’authentification unique (SSO), l’accès conditionnel, les proxies d’application, l’attribution d’utilisateurs, ainsi que les services de fédération Active Directory (AD FS) pour l’authentification directe et pour la synchronisation du hachage de mot de passe.

Pour consulter les guides de déploiement, accédez au dépôt GitHub [Identity Deployment Guides](./active-directory-deployment-plans.md) (Guides de déploiement des identités). Pour envoyer vos commentaires sur les guides de déploiement, utilisez le [formulaire de commentaires Plan de déploiement](https://aka.ms/deploymentplanfeedback). Si vous avez des questions sur les guides de déploiement, contactez-nous ici : [IDGitDeploy](mailto:idgitdeploy@microsoft.com).

---

### <a name="enterprise-applications-search---load-more-apps"></a>Recherche d’applications d’entreprise : charger d’autres applications

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Vous ne parvenez pas à localiser vos applications ou les principaux de service ? Nous vous permettons désormais de charger d’autres applications dans la liste de vos applications d’entreprise. Par défaut, nous affichons 20 applications. Vous pouvez désormais cliquer sur **Load more** (Charger plus d’applications) pour afficher d’autres applications.

---

### <a name="the-may-release-of-aadconnect-contains-a-public-preview-of-the-integration-with-pingfederate-important-security-updates-many-bug-fixes-and-new-great-new-troubleshooting-tools"></a>La version d’AADConnect du mois de mai contient une préversion publique de l’intégration à PingFederate, d’importantes mises à jour de sécurité, de nombreux correctifs de bogues et de nouveaux outils de résolution des problèmes.

**Type :** Fonctionnalité modifiée **Catégorie de service :** AD Connect **Fonctionnalité produit :** Gestion du cycle de vie des identités

La version d’AADConnect du mois de mai contient une préversion publique de l’intégration à PingFederate, d’importantes mises à jour de sécurité, de nombreux correctifs de bogues et de nouveaux outils de résolution des problèmes. Vous trouverez les notes de publication [ici](../hybrid/reference-connect-version-history.md).

---

### <a name="azure-ad-access-reviews-auto-apply"></a>Révisions d’accès Azure AD : application automatique

**Type :** Fonctionnalité modifiée **Catégorie de service :** Révisions d’accès **Fonctionnalité produit :** Gouvernance

La fonctionnalité de révision d’accès des groupes et applications fait maintenant partie de l’offre Azure AD Premium P2. Un administrateur peut demander à appliquer automatiquement les modifications du réviseur à ce groupe ou à cette application dès que la révision d’accès est terminée. L’administrateur peut également spécifier ce qu’il advient de l’accès continu de l’utilisateur si les réviseurs n’ont pas répondu, ou encore s’ils ont supprimé l’accès, conservé l’accès ou suivi les suggestions du système.

---

### <a name="id-tokens-can-no-longer-be-returned-using-the-query-response_mode-for-new-apps"></a>Les jetons d’ID ne peuvent plus être retournés à l’aide de la requête response_mode pour les nouvelles applications.

**Type :** Fonctionnalité modifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Les applications créées le 25 avril 2018 et après ne pourront plus demander un **id_token** à l’aide de la **requête** response_mode.  Cette opération met en ligne Azure AD avec les spécifications OIDC et vous aide à limiter la surface d’attaque des applications.  Rien n’empêche les applications créées avant le 25 avril 2018 d’utiliser la **requête** response_mode avec un paramètre response_type **id_token**.  Quand vous demandez un id_token à partir d’Azure AD, le message d’erreur **AADSTS70007: ‘query’ is not a supported value of ‘response_mode’ when requesting a token** s’affiche.

Les requêtes response_mode **fragment** et **form_post** continuent de fonctionner. Au moment de créer des objets d’application (par exemple, pour l’utilisation du proxy d’application), vérifiez que l’une de ces requêtes response_mode est utilisée avant de créer l’application.

---

## <a name="april-2018"></a>Avril 2018

### <a name="azure-ad-b2c-access-token-are-ga"></a>Les jetons d’accès Azure AD B2C sont en disponibilité générale

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Vous pouvez maintenant accéder aux API web sécurisées par Azure AD B2C à l’aide de jetons d’accès. La fonctionnalité passe de la préversion publique à la disponibilité générale. L’interface utilisateur permettant de configurer les applications Azure AD B2C et les API web a été améliorée, et d’autres améliorations mineures ont également été apportées.

Pour plus d’informations, consultez [Azure AD B2C : Demande de jetons d'accès](../../active-directory-b2c/access-tokens.md).

---

### <a name="test-single-sign-on-configuration-for-saml-based-applications"></a>Test de la configuration de l’authentification unique pour les applications SAML

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Lors de la configuration d’applications SSO SAML, vous pouvez tester l’intégration dans la page de configuration. Si vous rencontrez une erreur pendant la connexion, vous pouvez indiquer cette erreur dans l’expérience de test et recevoir d’Azure AD les étapes permettant de résoudre le problème.

Pour plus d'informations, consultez les pages suivantes :

- [Configuration de l'authentification unique pour les applications ne faisant pas partie de la galerie d'applications Azure Active Directory.](../manage-apps/view-applications-portal.md)
- [Débogage d’une authentification unique basée sur SAML aux applications dans Azure Active Directory](../manage-apps/debug-saml-sso-issues.md)

---

### <a name="azure-ad-terms-of-use-now-has-per-user-reporting"></a>La fonctionnalité Conditions d’utilisation Azure AD propose à présent des rapports par utilisateur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Conformité

Les administrateurs peuvent à présent sélectionner des conditions d’utilisation données et afficher tous les utilisateurs qui ont accepté les conditions d’utilisation, ainsi que la date et l’heure de leur consentement.

Pour plus d’informations, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md).

---

### <a name="azure-ad-connect-health-risky-ip-for-ad-fs-extranet-lockout-protection"></a>Azure AD Connect Health : Adresse IP à risque pour la protection par verrouillage Extranet AD FS

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Autre **Fonctionnalité produit :** Monitoring et création de rapports

Connect Health prend maintenant en charge la capacité de détecter des adresses IP qui dépassent un seuil d’échecs de connexion U/P (utilisateur/mot de passe) sur une base horaire ou quotidienne. Voici les fonctionnalités fournies par cette fonctionnalité :

- Rapport complet indiquant l’adresse IP et le nombre d’échecs de connexion généré sur une base horaire/quotidienne avec un seuil personnalisable.
- Alertes par e-mail indiquant quand une adresse IP spécifique a dépassé le seuil d’échecs de connexion U/P sur une base horaire/quotidienne.
- Option de téléchargement pour effectuer une analyse détaillée des données

Pour plus d’informations, consultez [Rapports sur les adresses IP à risque](../hybrid/how-to-connect-health-adfs.md).

---

### <a name="easy-app-config-with-metadata-file-or-url"></a>Configuration d’applications facilitée avec l’URL ou le fichier de métadonnées

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Dans la page Applications d’entreprise, les administrateurs peuvent charger un fichier de métadonnées SAML pour configurer l’authentification basée sur SAML pour les applications Azure AD, que celles-ci figurent ou non dans la galerie.

En outre, vous pouvez utiliser l’URL des métadonnées de fédération d’application Azure AD pour configurer l’authentification unique avec l’application ciblée.

Pour plus d’informations, consultez la page [Configuration de l’authentification unique pour les applications ne faisant pas partie de la galerie d’applications Azure Active Directory](../manage-apps/view-applications-portal.md).

---

### <a name="azure-ad-terms-of-use-now-generally-available"></a>Fonctionnalité Conditions d’utilisation Azure AD maintenant à la disposition générale

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Conformité


La fonctionnalité Conditions d’utilisation Azure AD est passée de la préversion publique à la disponibilité générale.

Pour plus d’informations, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory](../conditional-access/terms-of-use.md).

---

### <a name="allow-or-block-invitations-to-b2b-users-from-specific-organizations"></a>Autoriser ou bloquer des invitations aux utilisateurs B2B à partir d’organisations spécifiques

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2B **Fonctionnalité produit :** B2B/B2C


Vous pouvez maintenant spécifier les organisations partenaires avec lesquelles vous voulez partager et collaborer dans Azure AD B2B Collaboration. Pour ce faire, vous pouvez choisir de créer une liste de domaines autorisés ou refusés spécifiques. Lorsqu’un domaine est bloqué à l’aide de ces fonctionnalités, les employés ne peuvent plus envoyer d’invitations aux personnes de ce domaine.

Cela vous permet de contrôler l’accès à vos ressources, tout en permettant une expérience fluide pour les utilisateurs approuvés.

Cette fonctionnalité B2B Collaboration est disponible pour tous les clients Azure Active Directory. Elle peut servir conjointement avec les fonctionnalités Azure AD Premium, telles que la protection des identités et l’accès conditionnel pour un contrôle plus précis qui permet de savoir quand et de quelle façon les utilisateurs professionnels externes se connectent et obtiennent un accès.

Pour plus d’informations, consultez [Autoriser ou bloquer des invitations aux utilisateurs B2B à partir d’organisations spécifiques](../external-identities/allow-deny-list.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En avril 2018, nous avons ajouté à notre galerie d’applications les 13 applications suivantes qui prennent en charge la fédération :

Criterion HCM, [FiscalNote](../saas-apps/fiscalnote-tutorial.md), [Secret Server (On-Premises)](../saas-apps/secretserver-on-premises-tutorial.md), [Dynamic Signal](../saas-apps/dynamicsignal-tutorial.md), [mindWireless](../saas-apps/mindwireless-tutorial.md), [OrgChart Now](../saas-apps/orgchartnow-tutorial.md), [Ziflow](../saas-apps/ziflow-tutorial.md), [AppNeta Performance Monitor](../saas-apps/appneta-tutorial.md), [Elium](../saas-apps/elium-tutorial.md), [Fluxx Labs](../saas-apps/fluxxlabs-tutorial.md), [Cisco Cloud](../saas-apps/ciscocloud-tutorial.md), Shelf, [SafetyNet](../saas-apps/safetynet-tutorial.md)

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md).

Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-applications-public-preview"></a>Octroyer aux utilisateurs B2B dans Azure AD l’accès à vos applications locales (préversion publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2B **Fonctionnalité produit :** B2B/B2C

Si votre organisation utilise des fonctionnalités d’Azure Active Directory (Azure AD) B2B Collaboration pour inviter des utilisateurs d’organisations partenaires à votre instance Azure AD, vous pouvez désormais fournir à ces utilisateurs B2B un accès aux applications locales. Ces applications locales peuvent utiliser l’authentification basée SAML ou l’Authentification Windows intégrée (IWA) avec la délégation contrainte Kerberos (KCD).

Pour plus d’informations, consultez [Accorder aux utilisateurs B2B dans Azure AD l’accès à vos applications locales](../external-identities/hybrid-cloud-to-on-premises.md).

---

### <a name="get-sso-integration-tutorials-from-the-azure-marketplace"></a>Accéder à des tutoriels sur l’intégration de l’authentification unique dans la Place de marché Azure

**Type :** Fonctionnalité modifiée **Catégorie de service :** Autre **Fonctionnalité produit :** Intégration tierce

Si une application qui est listée dans la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/azure-active-directory-apps?page=1) prend en charge l’authentification unique SAML, en cliquant sur **Obtenir maintenant**, vous accédez au tutoriel sur l’intégration qui est associé à cette application.

---

### <a name="faster-performance-of-azure-ad-automatic-user-provisioning-to-saas-applications"></a>Amélioration des performances de l’attribution automatique d’utilisateurs Azure AD auprès des applications SaaS

**Type :** Fonctionnalité modifiée **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Intégration tierce

Auparavant, les clients qui utilisaient les connecteurs d’attribution d’utilisateurs Azure Active Directory pour les applications SaaS (Salesforce, ServiceNow et Box par exemple) pouvaient rencontrer une baisse de performances si leurs locataires Azure AD contenaient plus de 100 000 utilisateurs et groupes combinés, et s’ils utilisaient les attributions de groupes et d’utilisateurs pour déterminer les utilisateurs à attribuer.

Le 2 avril 2018, de très importantes améliorations des performances ont été déployées sur le service d’attribution Azure AD. Celles-ci réduisent considérablement le temps nécessaire pour effectuer les synchronisations initiales entre Azure Active Directory et les applications SaaS cibles.

Par conséquent, de nombreux clients pour lesquels les synchronisations initiales aux applications nécessitaient plusieurs jours ou ne se terminaient jamais effectuent maintenant cette opération en quelques minutes ou heures.

Pour plus d’informations, consultez [Que se passe-t-il pendant l’approvisionnement ?](../..//active-directory/app-provisioning/how-provisioning-works.md).

---

### <a name="self-service-password-reset-from-windows-10-lock-screen-for-hybrid-azure-ad-joined-machines"></a>Réinitialisation de mot de passe en libre-service à partir de l’écran de verrouillage Windows 10 pour les ordinateurs joints à une version hybride d’Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Réinitialisation du mot de passe libre-service **Fonctionnalité produit :** Authentification utilisateur

Nous avons mis à jour la fonctionnalité Windows 10 SSPR pour prendre en charge les ordinateurs qui sont joints à des environnements Azure AD hybrides. Cette fonctionnalité disponible dans Windows 10 RS4 permet aux utilisateurs de réinitialiser leur mot de passe à partir de l’écran de verrouillage d’un ordinateur Windows 10. Les utilisateurs qui sont activés et inscrits pour la réinitialisation de mot de passe en libre-service peuvent utiliser cette fonctionnalité.

Pour plus d’informations, consultez [Réinitialisation de mot de passe Azure AD depuis l’écran de connexion](../authentication/howto-sspr-windows.md).

---

## <a name="march-2018"></a>Mars 2018

### <a name="certificate-expire-notification"></a>Notification d’expiration de certificat

**Type :** Corrigé **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Azure AD envoie une notification quand le certificat d’une application de la galerie ou hors galerie est sur le point d’expirer.

Certains utilisateurs n’ont pas reçu les notifications concernant les applications d’entreprise configurées pour l’authentification unique SAML. Ce problème a été résolu. Azure AD envoie une notification concernant les certificats expirant dans 7, 30 et 60 jours. Vous pouvez visualiser cet événement dans les journaux d’audit.

Pour plus d'informations, consultez les pages suivantes :

- [Gérer des certificats pour l’authentification unique fédérée sur Azure Active Directory](../manage-apps/manage-certificates-for-federated-single-sign-on.md)
- [Rapports d’activité d’audit dans le portail Azure Active Directory](../reports-monitoring/concept-audit-logs.md)

---

### <a name="twitter-and-github-identity-providers-in-azure-ad-b2c"></a>Fournisseurs d’identité Twitter et GitHub dans Azure AD B2C

**Type :** Nouvelle fonctionnalité **Catégorie de service :** B2C - Gestion des identités **Fonctionnalité produit :** B2B/B2C

Vous pouvez maintenant ajouter Twitter ou GitHub en tant que fournisseur d’identité dans Azure AD B2C. Twitter passe de la version préliminaire publique à la disponibilité générale. GitHub sort en version préliminaire publique.

Pour plus d’informations, consultez la page [Qu’est-ce qu’Azure AD B2B Collaboration ?](../external-identities/what-is-b2b.md)

---

### <a name="restrict-browser-access-using-intune-managed-browser-with-azure-ad-application-based-conditional-access-for-ios-and-android"></a>Restreindre l’accès du navigateur à l’aide d’Intune Managed Browser avec accès conditionnel par application Azure AD pour iOS et Android

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

**Désormais en préversion publique**

**Authentification unique sur Intune Managed Browser :** Vos employés peuvent utiliser l’authentification unique sur des clients natifs (tels que Microsoft Outlook) et Intune Managed Browser pour toutes les applications Azure AD connectés.

**Prise en charge de l’accès conditionnel dans Intune Managed Browser :** Vous pouvez désormais obliger vos employés à utiliser Intune Managed Browser grâce à des stratégies d’accès conditionnel basé sur les applications.

Pour en savoir plus sur ce sujet, consultez notre [billet de blog](https://cloudblogs.microsoft.com/enterprisemobility/2018/03/15/the-intune-managed-browser-now-supports-azure-ad-sso-and-conditional-access/).

Pour plus d'informations, consultez les pages suivantes :

- [Configurer l’accès conditionnel basé sur les applications](../conditional-access/app-based-conditional-access.md)

- [Gérer des stratégies Managed Browser](/mem/intune/apps/manage-microsoft-edge)

---

### <a name="app-proxy-cmdlets-in-powershell-ga-module"></a>Cmdlets de proxy d'application dans la version en disponibilité générale du module PowerShell

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Contrôle d’accès

Les cmdlets de proxy d'application sont désormais pris en charge dans la version en disponibilité générale du module PowerShell. Cela vous oblige à maintenir vos modules PowerShell à jour : si vos dernières mises à jour remontent à plus d'un an, certaines cmdlets peuvent cesser de fonctionner.

Pour plus d'informations, consultez la page [Azure AD](/powershell/module/Azuread/).

---

### <a name="office-365-native-clients-are-supported-by-seamless-sso-using-a-non-interactive-protocol"></a>Les clients Office 365 natifs sont pris en charge par l’authentification unique transparente avec un protocole non interactif

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Les utilisateurs utilisant des clients Office 365 natifs (version 16.0.8730.xxxx et versions ultérieures) profitent d’une expérience de connexion silencieuse avec l’authentification unique transparente. Cette prise en charge est rendue possible par l’ajout un protocole non interactif (WS-Trust) à Azure AD.

Pour plus d’informations, consultez la page [Fonctionnement des connexions sur un client natif avec l’authentification unique transparente](../hybrid/how-to-connect-sso-how-it-works.md#how-does-sign-in-on-a-native-client-with-seamless-sso-work).

---

### <a name="users-get-a-silent-sign-on-experience-with-seamless-sso-if-an-application-sends-sign-in-requests-to-azure-ads-tenant-endpoints"></a>Expérience de connexion silencieuse avec l’authentification unique fluide lorsqu’une application envoie des demandes de connexion aux points de terminaison des locataires d’Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Les utilisateurs obtiennent une expérience de connexion silencieuse avec l’authentification unique transparente si une application (par exemple, `https://contoso.sharepoint.com`) envoie des demandes de connexion aux points de terminaison de locataire d’Azure AD, c’est-à-dire `https://login.microsoftonline.com/contoso.com/<..>` ou `https://login.microsoftonline.com/<tenant_ID>/<..>` au lieu du point de terminaison habituel d’Azure AD (`https://login.microsoftonline.com/common/<...>`).

Pour plus d’informations, consultez la page [Authentification unique transparente Azure Active Directory](../hybrid/how-to-connect-sso.md).

---

### <a name="need-to-add-only-one-azure-ad-url-instead-of-two-urls-previously-to-users-intranet-zone-settings-to-roll-out-seamless-sso"></a>Besoin d’ajouter seulement une URL Azure AD au lieu de deux aux paramètres de zone intranet des utilisateurs pour déployer l’authentification unique transparente

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Pour déployer l’authentification unique transparente chez vos utilisateurs, vous ne devez ajouter d’une URL Azure AD à leurs paramètres de zone intranet via une stratégie de groupe dans Active Directory : `https://autologon.microsoftazuread-sso.com`. Auparavant, les clients devaient ajouter deux URL.

Pour plus d’informations, consultez la page [Authentification unique transparente Azure Active Directory](../hybrid/how-to-connect-sso.md).

---

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En mars 2018, nous avons ajouté à notre galerie d’applications les 15 applications suivantes qui prennent en charge la fédération :

[Boxcryptor](../saas-apps/boxcryptor-tutorial.md), [CylancePROTECT](../saas-apps/cylanceprotect-tutorial.md), Wrike, [SignalFx](../saas-apps/signalfx-tutorial.md), Assistant by FirstAgenda, [YardiOne](../saas-apps/yardione-tutorial.md), Vtiger CRM, inwink, [Amplitude](../saas-apps/amplitude-tutorial.md), [Spacio](../saas-apps/spacio-tutorial.md), [ContractWorks](../saas-apps/contractworks-tutorial.md), [Bersin](../saas-apps/bersin-tutorial.md), [Mercell](../saas-apps/mercell-tutorial.md), [Trisotech Digital Enterprise Server](../saas-apps/trisotechdigitalenterpriseserver-tutorial.md), [Qumu Cloud](../saas-apps/qumucloud-tutorial.md).

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md).

Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="pim-for-azure-resources-is-generally-available"></a>La fonction PIM pour les ressources Azure est en disponibilité générale

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Si vous utilisez Azure AD Privileged Identity Management pour les rôles d’annuaire, vous pouvez maintenant utiliser les fonctions d’accès et d’affectation temporaires PIM pour les rôles de ressources Azure comme les abonnements, les groupes de ressources, les machines virtuelles et les autres ressources prises en charge par Azure Resource Manager. Appliquez l’authentification multifacteur lors de l’activation juste-à-temps de rôles et planifiez les activations en accord avec les fenêtres de changement approuvées. En outre, cette version contient des améliorations non disponibles dans la version préliminaire publique, notamment une nouvelle interface utilisateur, des workflows d’approbation et la possibilité d’étendre les rôles arrivant à expiration et de renouveler les rôles expirés.

Pour plus d’informations, consultez [PIM pour les ressources Azure (préversion)](../privileged-identity-management/azure-pim-resource-rbac.md).

---

### <a name="adding-optional-claims-to-your-apps-tokens-public-preview"></a>Ajout de revendications facultatives à vos jetons d’applications (version préliminaire publique)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Votre application Azure AD peut maintenant demander des revendications personnalisées ou facultatives dans les jetons JWT ou SAML.  Il s’agit de revendications relatives à l’utilisateur ou au locataire qui ne sont pas incluses par défaut dans le jeton en raison d’une contrainte de taille ou d’applicabilité.  Cette fonction est disponible en version préliminaire publique pour les applications Azure AD sur les points de terminaison v1.0 et v2.0.  Consultez la documentation pour plus d’informations sur les revendications pouvant être ajoutées et pour savoir comment modifier le manifeste de votre application pour les demander.

Pour plus d’informations, consultez la page [Revendications facultatives dans Azure AD](../develop/active-directory-optional-claims.md).

---

### <a name="azure-ad-supports-pkce-for-more-secure-oauth-flows"></a>Azure AD prend en charge PKCE pour des flux OAuth plus sécurisés

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Les documents Azure AD ont été mis à jour et indiquent désormais la prise en charge de PKCE, qui permet de sécuriser davantage la communication pendant le flux d’octroi de code d’autorisation OAuth 2.0.  Les méthodes S256 et plaintext code_challenge sont prises en charge sur les points de terminaison v1.0 et v2.0.

Pour plus d’informations, consultez [Demander un code d’autorisation](../develop/v2-oauth2-auth-code-flow.md#request-an-authorization-code).

---

### <a name="support-for-provisioning-all-user-attribute-values-available-in-the-workday-get_workers-api"></a>Prise en charge de l’approvisionnement de toutes les valeurs d’attribut utilisateur disponibles dans l’API Workday Get_Workers

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Provisionnement d’application **Fonctionnalité produit :** Intégration tierce

La version préliminaire publique de l’approvisionnement entrant de Workday vers Active Directory et Azure AD prend désormais en charge la possibilité d’extraire et d’approvisionner toutes les valeurs d’attribut disponibles dans l’API Workday Get_Workers. Cela permet la prise en charge de centaines d’attributs standard et personnalisés supplémentaires, en plus de ceux fournis avec la version initiale du connecteur d’approvisionnement entrant Workday.

Pour plus d'informations, consultez les pages suivantes : [Personnaliser la liste des attributs d’utilisateurs Workday](../saas-apps/workday-inbound-tutorial.md#customizing-the-list-of-workday-user-attributes)

---

### <a name="changing-group-membership-from-dynamic-to-static-and-vice-versa"></a>Changement de l’appartenance dynamique à un groupe en appartenance statique et vice versa

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des groupes **Fonctionnalité produit :** Collaboration

Il est possible de modifier la façon dont l’appartenance est gérée dans un groupe. Cela est utile lorsque vous souhaitez conserver le même nom et le même ID de groupe dans le système, afin que toutes les références au groupe existantes soient toujours valides ; la création d’un groupe nécessiterait la mise à jour de ces références.
Nous avons mis à jour le Centre d’administration Azure AD pour la prise en charge de cette fonctionnalité. Maintenant, les clients peuvent convertir l’appartenance dynamique à des groupes existants en appartenance affectée et vice versa. Les cmdlets PowerShell existants sont également toujours disponibles.

Pour plus d’informations, consultez [Règles d’appartenance de groupe dynamique dans Azure Active Directory](../enterprise-users/groups-dynamic-membership.md)

---

### <a name="improved-sign-out-behavior-with-seamless-sso"></a>Comportement de déconnexion amélioré avec l’authentification unique transparente

**Type :** Fonctionnalité modifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

Auparavant, même si les utilisateurs se déconnectaient explicitement d’une application sécurisée par Azure AD, ils étaient automatiquement reconnectés avec l’authentification unique transparente s’ils tentaient à nouveau d’accéder à une application Azure AD au sein de leur réseau d’entreprise à partir d’un appareil joint à un domaine. Avec cette modification, la déconnexion est prise en charge.  Cela permet aux utilisateurs de conserver ou modifier le compte Azure AD auquel se connecter, au lieu d’être connecté automatiquement à l’aide de l’authentification unique transparente.

Pour plus d’informations, consultez [Authentification unique transparente Azure Active Directory](../hybrid/how-to-connect-sso.md)

---

### <a name="application-proxy-connector-version-154020-released"></a>Connecteur de proxy d’application version 1.5.402.0

**Type :** Fonctionnalité modifiée **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Protection et sécurité des identités

Cette version de connecteur est progressivement déployée tout au long du mois de novembre. Cette nouvelle version de connecteur inclut les modifications suivantes :

- Le connecteur définit désormais des cookies au niveau du domaine et non plus au niveau du sous-domaine. Cela garantit une meilleure expérience d’authentification unique et évite les invites d’authentification redondantes.
- Prise en charge des demandes de codage mémorisé en bloc
- Surveillance améliorée de l’intégrité du connecteur
- Correction de plusieurs bogues et stabilité améliorée

Pour plus d’informations, consultez [Présentation des connecteurs de proxy d’application Azure AD](../manage-apps/application-proxy-connectors.md).

---

## <a name="february-2018"></a>Février 2018

### <a name="improved-navigation-for-managing-users-and-groups"></a>Navigation améliorée pour la gestion des utilisateurs et groupes

**Type :** Modification planifiée **Catégorie de service :** Gestion des annuaires **Fonctionnalité produit :** Répertoire

L’expérience de navigation pour la gestion des utilisateurs et groupes a été simplifiée. Vous pouvez maintenant passer directement de la vue d’ensemble de l’annuaire à la liste de tous les utilisateurs, et accéder plus facilement à la liste des utilisateurs supprimés. Vous pouvez également accéder directement de la vue d’ensemble de l’annuaire à la liste de tous les groupes, et accéder plus facilement aux paramètres de gestion de groupe. Également depuis la vue d’ensemble de l’annuaire, vous pouvez rechercher un utilisateur, un groupe, une application d’entreprise ou une inscription d’application.

---

### <a name="availability-of-sign-ins-and-audit-reports-in-microsoft-azure-operated-by-21vianet-azure-china-21vianet"></a>Disponibilité des rapports d’audit et des rapports sur les connexions dans Microsoft Azure gérée par 21Vianet (Azure China 21Vianet)

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure Stack **Fonctionnalité produit :** Monitoring et création de rapports

Les rapports du journal d’activité Azure AD sont maintenant disponibles dans Microsoft Azure géré par des instances 21Vianet (Azure China 21Vianet). Les journaux d’activité suivants sont inclus :

- **Journaux d’activité des connexions** : incluent tous les journaux d’activité des connexions associés à votre locataire.

- **Journaux d’audit de mot de passe libre-service** : comprennent tous les journaux d’audit SSPR.

- **Journaux d’audit de gestion des annuaires** : incluent tous les journaux d’audit liés à la gestion des annuaires, comme la gestion des utilisateurs, la gestion des applications, etc.

Ces journaux d’activité vous permettent d’obtenir des insights sur le fonctionnement de votre environnement. Les données fournies vous permettent de :

- Déterminer la façon dont les applications et les services sont utilisés par vos utilisateurs.

- Résoudre les problèmes empêchant les utilisateurs d’effectuer leur travail.

Pour plus d’informations sur l’utilisation de ces rapports, consultez [Création de rapports Azure Active Directory](../reports-monitoring/overview-reports.md).

---

### <a name="use-report-reader-role-non-admin-role-to-view-azure-ad-activity-reports"></a>Utilisation du rôle « Lecteur de rapport » (rôle non-administrateur) pour afficher les rapports d’activité Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Monitoring et création de rapports

Suite aux commentaires clients sur la possibilité d’autoriser les rôles non administrateurs à accéder aux journaux d’activité Azure AD, nous permettons désormais aux utilisateurs dotés du rôle « Lecteur de rapport » d’accéder aux journaux d’activité des audits et des connexions sur le portail Azure, ainsi que d’utiliser l’API Microsoft Graph.

Pour plus d’informations sur l’utilisation de ces rapports, consultez [Création de rapports Azure Active Directory](../reports-monitoring/overview-reports.md).

---

### <a name="employeeid-claim-available-as-user-attribute-and-user-identifier"></a>Revendication EmployeeID disponible en tant qu’attribut utilisateur et identificateur d’utilisateur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** SSO

Vous pouvez configurer **EmployeeID** en tant qu’identificateur d’utilisateur et attribut utilisateur pour les utilisateurs membres et les invités B2B dans les applications d’authentification SAML à partir de l’interface utilisateur de l’application d’entreprise.

Pour plus d’informations, consultez [Personnalisation des revendications émises dans le jeton SAML pour les applications d’entreprise dans Azure Active Directory](../develop/active-directory-saml-claims-customization.md).

---

### <a name="simplified-application-management-using-wildcards-in-azure-ad-application-proxy"></a>Gestion simplifiée des applications à l’aide de caractères génériques dans Proxy d’application Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Authentification utilisateur

Pour simplifier le déploiement d’application et réduire votre charge administrative, nous prenons désormais en charge la possibilité de publier des applications à l’aide de caractères génériques. Pour publier une application générique, vous pouvez suivre le flux de publication d’application standard, mais utiliser un caractère générique dans les URL internes et externes.

Pour plus d’informations, consultez [Applications génériques dans le proxy d’application Azure Active Directory](../manage-apps/application-proxy-wildcard.md).

---

### <a name="new-cmdlets-to-support-configuration-of-application-proxy"></a>Nouvelles applets de commande pour prendre en charge la configuration du proxy d’application

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Plateforme

La dernière version du module AzureAD PowerShell Preview contient de nouvelles applets de commande qui permettent aux clients de configurer des applications de proxy d’application à l’aide de PowerShell.

Ces nouvelles applets de commande sont :

- Get-AzureADApplicationProxyApplication
- Get-AzureADApplicationProxyApplicationConnectorGroup
- Get-AzureADApplicationProxyConnector
- Get-AzureADApplicationProxyConnectorGroup
- Get-AzureADApplicationProxyConnectorGroupMembers
- Get-AzureADApplicationProxyConnectorMemberOf
- New-AzureADApplicationProxyApplication
- New-AzureADApplicationProxyConnectorGroup
- Remove-AzureADApplicationProxyApplication
- Remove-AzureADApplicationProxyApplicationConnectorGroup
- Remove-AzureADApplicationProxyConnectorGroup
- Set-AzureADApplicationProxyApplication
- Set-AzureADApplicationProxyApplicationConnectorGroup
- Set-AzureADApplicationProxyApplicationCustomDomainCertificate
- Set-AzureADApplicationProxyApplicationSingleSignOn
- Set-AzureADApplicationProxyConnector
- Set-AzureADApplicationProxyConnectorGroup

---

### <a name="new-cmdlets-to-support-configuration-of-groups"></a>Nouvelles applets de commande pour prendre en charge la configuration des groupes

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Proxy d’application **Fonctionnalité produit :** Plateforme

La dernière version du module AzureAD PowerShell contient des applets de commande qui permettent de gérer les groupes dans Azure AD. Ces applets de commande étaient auparavant disponibles dans le module AzureADPreview et sont maintenant ajoutées au module AzureAD.

Les applets de commande Group désormais mises à la disposition générale sont :

- Get-AzureADMSGroup
- New-AzureADMSGroup
- Remove-AzureADMSGroup
- Set-AzureADMSGroup
- Get-AzureADMSGroupLifecyclePolicy
- New-AzureADMSGroupLifecyclePolicy
- Remove-AzureADMSGroupLifecyclePolicy
- Add-AzureADMSLifecyclePolicyGroup
- Remove-AzureADMSLifecyclePolicyGroup
- Reset-AzureADMSLifeCycleGroup
- Get-AzureADMSLifecyclePolicyGroup

---

### <a name="a-new-release-of-azure-ad-connect-is-available"></a>Nouvelle version d’Azure AD Connect disponible

**Type :** Nouvelle fonctionnalité **Catégorie de service :** AD Sync **Fonctionnalité produit :** Plateforme

Azure AD Connect est l’outil préféré pour synchroniser des données entre Azure AD et des sources de données locales, notamment Windows Server Active Directory et LDAP.

>[!Important]
>Cette version introduit des modifications de schéma et de règles de synchronisation. Le service de synchronisation Azure AD Connect déclenche des étapes d’importation et de synchronisation complètes après une mise à niveau. Pour plus d’informations sur la façon de modifier ce comportement, consultez [Comment différer la synchronisation complète après la mise à niveau](../hybrid/how-to-upgrade-previous-version.md#how-to-defer-full-synchronization-after-upgrade).

Cette version comprend les mises à jour et modifications suivantes :

**Problèmes résolus**

- Résolution du problème de fenêtre de synchronisation sur les tâches en arrière-plan pour la page Filtrage de partitions lors du passage à la page suivante

- Correction d’un bogue qui entraînait une violation d’accès lors de l’action personnalisée ConfigDB.

- Correction d’un bogue de récupération suite à un délai de connexion SQL.

- Correction d’un bogue qui entraînait l’échec d’une vérification des prérequis pour les certificats avec caractères génériques SAN.

- Correction d’un bogue qui entraîne un plantage de miiserver.exe lors d’une exportation du connecteur Azure AD.

- Correction d’un bogue concernant la journalisation d’une tentative d’utilisation d’un mot de passe incorrect sur le contrôleur de domaine qui entraînait le changement de la configuration par l’Assistant Azure AD Connect

**Améliorations et nouvelles fonctionnalités**

- Télémétrie applicative : l’administrateur peut activer/désactiver cette catégorie de données.

- Données d’intégrité Azure AD  : l’administrateur doit accéder au portail de contrôle d’intégrité pour contrôler ses paramètres d’intégrité. Une fois que la stratégie du service a été modifiée, les agents la lisent et la mettent en œuvre.

- Ajout d’actions de configuration d’écriture différée sur les appareils et d’une barre de progression pour l’initialisation de la page.

- Amélioration des diagnostics généraux avec un rapport HTML et une collecte de données complète dans un rapport au format ZIP/HTML.

- Amélioration de la fiabilité des mises à niveau automatiques et ajout d’une télémétrie supplémentaire pour garantir la possibilité de détermination de l’intégrité du serveur.

- Restriction des autorisations disponibles pour les comptes privilégiés sur le compte du Connecteur AD. Dans le cas des nouvelles installations, l’Assistant restreint les autorisations dont disposent les comptes privilégiés sur le compte MSOL après la création de ce dernier. Les modifications affectent les installations rapides et les installations personnalisées avec création automatique de compte.

- Modification du programme d’installation afin de ne pas exiger des privilèges d’administrateur système pour une nouvelle installation d’AADConnect.

- Nouvel utilitaire destiné à résoudre les problèmes de synchronisation pour un objet spécifique. Cet utilitaire vérifie les éléments suivants :

    - détection d’une incompatibilité de nom d’utilisateur principal (UPN) entre l’objet utilisateur synchronisé et le compte d’utilisateur dans le locataire Azure AD ;

    - vérification si l’objet a été exclu de la synchronisation en raison d’un filtrage de domaine ;

    - vérification si l’objet a été exclu de la synchronisation en raison d’un filtrage d’unité d’organisation.

- Nouvel utilitaire destiné à synchroniser le hachage de mot de passe actuel stocké dans le service Active Directory local pour un compte d’utilisateur spécifique. Cet utilitaire ne nécessite aucune modification de mot de passe.

---

### <a name="applications-supporting-intune-app-protection-policies-added-for-use-with-azure-ad-application-based-conditional-access"></a>Ajout d’applications prenant en charge les stratégies Intune App Protection à des fins d’utilisation avec l’accès conditionnel en fonction de l’application Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Nous avons ajouté d’autres applications qui prennent en charge l’accès conditionnel en fonction de l’application. Maintenant, vous pouvez accéder à Office 365 et à d’autres applications cloud liées à Azure AD à l’aide de ces applications clientes approuvées.

Les applications suivantes seront ajoutées d’ici la fin du mois de février :

- Microsoft Power BI

- Microsoft Launcher

- Microsoft Invoicing

Pour plus d'informations, consultez les pages suivantes :

- [Spécification d’application cliente approuvée](../conditional-access/concept-conditional-access-conditions.md#client-apps)
- [Accès conditionnel basé sur les applications Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="terms-of-use-update-to-mobile-experience"></a>Mise à jour des conditions d’utilisation pour l’expérience mobile

**Type :** Fonctionnalité modifiée **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Conformité

Lorsque les conditions d’utilisation s’affichent, vous pouvez maintenant cliquer sur **Des problèmes d’affichage ? Cliquez ici**. Ce lien ouvre les conditions d’utilisation en mode natif sur votre appareil. Quelle que soit la taille de police dans le document ou la taille d’écran de l’appareil, vous pouvez effectuer un zoom avant et lire le document selon vos besoins.

---

## <a name="january-2018"></a>Janvier 2018

### <a name="new-federated-apps-available-in-azure-ad-app-gallery"></a>Nouvelles applications fédérées disponibles dans la galerie Azure AD App

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En janvier 2018, les applications suivantes prenant en charge la fédération ont été ajoutées à la galerie d’applications :

[IBM OpenPages](../saas-apps/ibmopenpages-tutorial.md), [OneTrust Privacy Management Software](../saas-apps/onetrust-tutorial.md), [Dealpath](../saas-apps/dealpath-tutorial.md), [IriusRisk Federated Directory et [Fidelity NetBenefits](../saas-apps/fidelitynetbenefits-tutorial.md).

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md).

Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="sign-in-with-additional-risk-detected"></a>Connexion avec un risque supplémentaire détecté

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

L’insight que vous obtenez pour une détection des risques est liée à votre abonnement Azure AD. Avec l’édition Azure AD Premium P2, vous obtenez des informations très détaillées sur toutes les détections sous-jacentes.

Avec l’édition Azure AD Premium P1, les détections qui ne sont pas couvertes par la licence s’affichent comme des détections des risques Connexion avec un risque supplémentaire détecté.

Pour plus d’informations, consultez [Détections des risques dans Azure Active Directory](../identity-protection/overview-identity-protection.md).

---

### <a name="hide-office-365-applications-from-end-users-access-panels"></a>Masquer des applications Office 365 dans les panneaux d’accès de l’utilisateur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Mes applications **Fonctionnalité produit :** SSO

Désormais, vous pouvez mieux gérer comment les applications Office 365 apparaissent dans les panneaux d’accès de vos utilisateurs, grâce à un nouveau paramètre utilisateur. Cette option permet de réduire le nombre d’applications dans les panneaux d’accès d’un utilisateur, si vous préférez n’afficher que les applications Office dans le portail Office. Ce paramètre se trouve dans **Paramètres utilisateur** et s’intitule **Les utilisateurs peuvent voir uniquement les applications Office 365 dans le portail Office 365**.

Pour plus d’informations, consultez [Masquer une application de l’expérience utilisateur dans Azure Active Directory](../manage-apps/hide-application-from-user-portal.md).

---

### <a name="seamless-sign-into-apps-enabled-for-password-sso-directly-from-apps-url"></a>Se connecter de manière transparente à des applications autorisant l’authentification unique par mot de passe directement depuis l’URL de l’application

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Mes applications **Fonctionnalité produit :** SSO

L’extension de navigateur Mes applications est désormais disponible via un outil pratique qui vous propose une authentification unique à Mes applications sous la forme d’un raccourci dans votre navigateur. Après l’installation, l’utilisateur voit une icône en forme de gaufre dans son navigateur, qui lui assure un accès rapide aux applications. Les utilisateurs peuvent désormais profiter des fonctionnalités suivantes :

- Possibilité de se connecter directement aux applications à authentification unique par mot de passe dans la page de connexion de l’application
- Lancer une application à l’aide de la fonctionnalité de recherche rapide
- Raccourcis vers les applications récemment utilisées à partir de l’extension
- L’extension est disponible pour Microsoft Edge, Chrome et Firefox.

Pour plus d’informations, consultez [Extension de connexion sécurisée à Mes applications](../user-help/my-apps-portal-end-user-access.md#download-and-install-the-my-apps-secure-sign-in-extension).

---

### <a name="azure-ad-administration-experience-in-azure-classic-portal-has-been-retired"></a>Mise hors service de l’expérience d’administration Azure AD dans le portail Azure Classic

**Type :** Déprécié **Catégorie de service :** Azure AD **Fonctionnalité produit :** Répertoire

À compter du 8 janvier 2018, l’expérience d’administration d’Azure AD dans le portail Azure Classic a été retirée. Un retrait qui a coïncidé avec celui du portail Azure Classic. À l’avenir, vous devrez utiliser le [Centre d’administration d’Azure AD](https://aad.portal.azure.com) pour toute l’administration d’Azure AD dans le portail.

---

### <a name="the-phonefactor-web-portal-has-been-retired"></a>Le portail web PhoneFactor a été mis hors service

**Type :** Déprécié **Catégorie de service :** Azure AD **Fonctionnalité produit :** Répertoire

À compter du 8 janvier 2018, le portail web PhoneFactor a été retiré. Ce portail permettait d’administrer le serveur MFA, mais ces fonctions ont été transférées au portail Azure à l’adresse portal.azure.com.

La configuration de MFA se trouve dans : **Azure Active Directory \> serveur MFA**

---

### <a name="deprecate-azure-ad-reports"></a>Déprécier les rapports Azure AD

**Type :** Déprécié **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Gestion du cycle de vie des identités


Avec la disponibilité générale de la nouvelle console d’administration d’Azure Active Directory et les nouvelles API désormais disponibles pour les rapports d’activité et de sécurité, les API de rapport sous le point de terminaison « /rapports » ont été retirées le 31 décembre 2017.

**Qu’est-ce qui est disponible ?**

Dans le cadre de la transition vers la nouvelle console d’administration, nous avons créé 2 API pour récupérer les journaux d’activité d’Azure AD. Le nouvel ensemble d’API fournit une fonctionnalité enrichie de filtrage et de tri en plus de fournir des activités d’audit et de connexion plus avancées. Les données précédemment disponibles via les rapports de sécurité sont désormais accessibles via l’API de détection des risques Identity Protection dans Microsoft Graph.

Pour plus d'informations, consultez les pages suivantes :

- [Prise en main de l’API de création de rapports Azure Active Directory](../reports-monitoring/concept-reporting-api.md)

- [Prise en main d’Azure Active Directory Identity Protection et de Microsoft Graph](../identity-protection/howto-identity-protection-graph-api.md)

---

## <a name="december-2017"></a>Décembre 2017

### <a name="terms-of-use-in-the-access-panel"></a>Conditions d’utilisation dans le Panneau d’accès

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Conformité

Vous pouvez maintenant accéder au Panneau d’accès et afficher les conditions d’utilisation que vous avez précédemment acceptées.

Procédez comme suit :

1. Accédez au [portail MyApps](https://myapps.microsoft.com) et connectez-vous.

2. Dans l’angle supérieur droit, sélectionnez votre nom puis **Profil** dans la liste.

3. Dans votre **profil**, cliquez sur **Vérifier les conditions d’utilisation**.

4. Vérifiez les conditions d’utilisation que vous avez acceptées.

Pour plus d’informations, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory (préversion)](../conditional-access/terms-of-use.md).

---

### <a name="new-azure-ad-sign-in-experience"></a>Nouvelle expérience de connexion Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD **Fonctionnalité produit :** Authentification utilisateur

Les interfaces utilisateur d’Azure AD et du système d’identité de compte Microsoft ont été repensées pour gagner en cohérence. De plus, la page de connexion à Azure AD collecte d’abord le nom d’utilisateur, puis les informations d’identification dans un second écran.

Pour plus d’informations, consultez [The new Azure AD Signin Experience is now in Public Preview (La nouvelle expérience de connexion Azure AD est désormais en préversion publique)](https://cloudblogs.microsoft.com/enterprisemobility/2017/08/02/the-new-azure-ad-signin-experience-is-now-in-public-preview/).

---

### <a name="fewer-sign-in-prompts-a-new-keep-me-signed-in-experience-for-azure-ad-sign-in"></a>Invites de connexion moins nombreuses : La nouvelle expérience « Maintenir la connexion » pour Azure AD est en préversion

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD **Fonctionnalité produit :** Authentification utilisateur

Nous avons remplacé la case à cocher **Maintenir la connexion** dans la page de connexion à Azure AD par une nouvelle invite qui s’affiche une fois l’utilisateur authentifié.

Si vous répondez **Oui** à cette invite, le service vous donne un jeton d’actualisation permanent. Ce comportement est le même que quand vous activiez la case à cocher **Maintenir la connexion** dans l’ancienne expérience. Pour les locataires fédérés, cette invite s’affiche une fois l’utilisateur authentifié auprès du service fédéré.

Pour plus d’informations, consultez [Invites de connexion moins nombreuses : La nouvelle expérience « Maintenir la connexion » pour Azure AD est en préversion](https://cloudblogs.microsoft.com/enterprisemobility/2017/09/19/fewer-login-prompts-the-new-keep-me-signed-in-experience-for-azure-ad-is-in-preview/).

---

### <a name="add-configuration-to-require-the-terms-of-use-to-be-expanded-prior-to-accepting"></a>Ajouter la configuration pour exiger le développement des conditions d’utilisation avant leur acceptation

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Conformité

Une option destinée aux administrateurs oblige les utilisateurs à développer les conditions d’utilisation avant de les accepter.

Sélectionnez **Activé** ou **Désactivé** selon que vous souhaitiez obliger ou non les utilisateurs à développer les conditions d’utilisation. Le paramètre **Activé** oblige les utilisateurs à afficher les conditions d’utilisation avant de les accepter.

Pour plus d’informations, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory (préversion)](../conditional-access/terms-of-use.md).

---

### <a name="scoped-activation-for-eligible-role-assignments"></a>Activation délimitée des attributions de rôles éligibles

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Vous pouvez utiliser l’activation délimitée pour activer les attributions de rôles de ressources Azure éligibles avec moins d’autonomie que les valeurs d’attribution par défaut d’origine. Par exemple, supposons que vous soyez affecté comme propriétaire d’un abonnement dans votre locataire. Avec l’activation délimitée, vous pouvez activer le groupe Propriétaire pour au maximum cinq ressources contenues dans l’abonnement (groupes de ressources et machines virtuelles). La délimitation de l’activation contribue à réduire le risque d’exécution de modifications indésirables sur les ressources Azure critiques.

Pour plus d’informations, consultez [Qu’est-ce qu’Azure AD Privileged Identity Management ?](../privileged-identity-management/pim-configure.md).

---

### <a name="new-federated-apps-in-the-azure-ad-app-gallery"></a>Nouvelles applications fédérées dans la galerie Applications Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Applications d’entreprise **Fonctionnalité produit :** Intégration tierce

En décembre 2017, nous avons ajouté à notre galerie d’applications les applications suivantes qui prennent en charge la fédération :

[Accredible](../saas-apps/accredible-tutorial.md), Adobe Experience Manager, [EFI Digital StoreFront](../saas-apps/efidigitalstorefront-tutorial.md), [Communifire](../saas-apps/communifire-tutorial.md) CybSafe, [FactSet](../saas-apps/factset-tutorial.md), [IMAGE WORKS](../saas-apps/imageworks-tutorial.md), [MOBI](../saas-apps/mobi-tutorial.md), [MobileIron Azure AD integration](../saas-apps/mobileiron-tutorial.md), [Reflektive](../saas-apps/reflektive-tutorial.md), [SAML SSO for Bamboo by resolution GmbH](../saas-apps/bamboo-tutorial.md), [SAML SSO for Bitbucket by resolution GmbH](../saas-apps/bitbucket-tutorial.md), [Vodeclic](../saas-apps/vodeclic-tutorial.md), WebHR, Zenegy Azure AD Integration.

Pour plus d’informations sur les applications, consultez [Intégration des applications SaaS à Azure Active Directory](../saas-apps/tutorial-list.md).

Pour plus d’informations sur le référencement de votre application dans la galerie Azure AD App, consultez [Lister votre application dans la galerie d’applications Azure Active Directory](../develop/v2-howto-app-gallery-listing.md).

---

### <a name="approval-workflows-for-azure-ad-directory-roles"></a>Flux de travail d’approbation pour les rôles d’annuaire Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Les flux de travail d’approbation pour les rôles d’annuaire Azure AD sont généralement disponibles.

Avec les flux de travail d’approbation, les administrateurs de rôles privilégiés peuvent exiger que les membres des rôles éligibles demandent l’activation du rôle avant de pouvoir l’utiliser. Plusieurs utilisateurs et groupes peuvent se voir déléguer des responsabilités d’approbation. Les membres de rôle exigibles peuvent recevoir des notifications lorsque l’approbation est terminée et que leur rôle est actif.

---

### <a name="pass-through-authentication-skype-for-business-support"></a>Authentification directe : Prise en charge de Skype Entreprise

**Type :** Fonctionnalité modifiée **Catégorie de service :** Authentifications (connexions) **Fonctionnalité produit :** Authentification utilisateur

L’authentification directe autorise désormais les connexions des utilisateurs aux applications clientes Skype Entreprise qui prennent en charge l’authentification moderne, notamment les topologies connectée et hybride.

Pour plus d’informations, consultez [Topologies de Skype Entreprise prises en charge avec l’authentification moderne](/skypeforbusiness/plan-your-deployment/modern-authentication/topologies-supported).

---

### <a name="updates-to-azure-ad-privileged-identity-management-for-azure-rbac-preview"></a>Mises à jour d’Azure Active AD Privileged Identity Management pour Azure RBAC (préversion)

**Type :** Fonctionnalité modifiée **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Avec l’actualisation de la préversion publique d’Azure AD Privileged Identity Management (PIM) pour le Contrôle d’accès en fonction du rôle Azure, vous pouvez désormais :

* Utiliser JEA (Just Enough Administration).
* Exiger une approbation pour activer les rôles de ressource.
* Planifier l’activation future d’un rôle qui nécessite l’approbation des rôles Azure AD et RBAC Azure.

Pour plus d’informations, consultez [PIM pour les ressources Azure (préversion)](../privileged-identity-management/azure-pim-resource-rbac.md).

---

## <a name="november-2017"></a>Novembre 2017

### <a name="access-control-service-retirement"></a>Mise hors service d’Access Control Service

**Type :** Modification planifiée **Catégorie de service :** Access Control Service **Fonctionnalité produit :** Access Control service

Microsoft Azure Active Directory Access Control (également appelé Access Control Service) sera mis hors service fin 2018. D’autres informations, notamment un planning détaillé et des instructions générales de migration, seront fournies dans les prochaines semaines. Sur cette page, vous pouvez laisser des commentaires ainsi que des questions sur Access Control Service. Un membre de l’équipe y répondra.

---

### <a name="restrict-browser-access-to-the-intune-managed-browser"></a>Restreindre l’accès du navigateur à Intune Managed Browser

**Type :** Modification planifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Vous pourrez restreindre l’accès du navigateur à Office 365 et à d’autres applications cloud connectées à Azure AD en utilisant Intune Managed Browser comme application approuvée.

Vous pouvez désormais configurer la condition suivante pour conditionner l’accès en fonction de l’application :

**Applications clientes :** Browser

**Quel est l’impact de la modification ?**

Aujourd’hui, l’utilisation de cette condition bloque l’accès. Quand la préversion sera disponible, l’accès nécessitera l’utilisation de l’application Managed Browser.

Vous trouverez des détails supplémentaires sur cette fonctionnalité et d’autres dans les notes de publication et les blogs à venir.

Pour plus d’informations, consultez [Accès conditionnel dans Azure Active Directory](../conditional-access/overview.md).

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nouvelles applications clientes approuvées pour l’accès conditionnel basé sur les applications Azure AD

**Type :** Modification planifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Les applications suivantes se trouvent sur la liste des [applications clientes approuvées](../conditional-access/concept-conditional-access-conditions.md#client-apps) :

- [Microsoft Kaizala](https://www.microsoft.com/garage/profiles/kaizala/)
- Microsoft StaffHub

Pour plus d'informations, consultez les pages suivantes :

- [Spécification d’application cliente approuvée](../conditional-access/concept-conditional-access-conditions.md#client-apps)
- [Accès conditionnel basé sur les applications Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="terms-of-use-support-for-multiple-languages"></a>Prise en charge des conditions d’utilisation en plusieurs langues

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Conformité

Les administrateurs peuvent maintenant créer des conditions d’utilisation qui contiennent plusieurs documents PDF. Vous pouvez baliser ces documents PDF avec une langue correspondante. Le fichier PDF qui s’affiche correspond à la langue que l’utilisateur a spécifiée dans ses préférences. S’il n’existe aucune correspondance, la langue par défaut est affichée.

---

### <a name="real-time-password-writeback-client-status"></a>État en temps réel du client de réécriture du mot de passe

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Réinitialisation du mot de passe libre-service **Fonctionnalité produit :** Authentification utilisateur

Vous pouvez maintenant consulter l’état de votre client de réécriture du mot de passe local. Cette option est disponible dans la section **Intégration locale** de la page [Réinitialisation du mot de passe](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/PasswordReset).

En cas de problème avec la connexion à votre client de réécriture local, un message d’erreur s’affiche avec :

- Des informations sur la raison pour laquelle vous ne pouvez pas vous connecter à votre client de réécriture local.
- Un lien vers la documentation qui vous aidera à résoudre le problème

Pour plus d’informations, consultez [Intégration locale](../authentication/concept-sspr-howitworks.md#on-premises-integration).

---

### <a name="azure-ad-app-based-conditional-access"></a>Accès conditionnel basé sur les applications Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Azure AD **Fonctionnalité produit :** Protection et sécurité des identités

Vous pouvez désormais restreindre l’accès à Office 365 et à d’autres applications cloud connectées à Azure AD, aux [applications clientes approuvées](../conditional-access/concept-conditional-access-conditions.md#client-apps) qui prennent en charge les stratégies de protection des applications Intune par l’[accès conditionnel basé sur les applications Azure AD](../conditional-access/app-based-conditional-access.md). Des stratégies de protection des applications Intune sont utilisées pour configurer et protéger les données d’entreprise sur ces applications clientes.

En combinant des stratégies d’accès conditionnel [basées sur les applications](../conditional-access/app-based-conditional-access.md) et [basées sur les appareils](../conditional-access/require-managed-devices.md), vous bénéficiez de la flexibilité nécessaire pour protéger les données sur les appareils d’entreprise et personnels.

Les conditions et contrôles suivants sont désormais disponibles pour une utilisation avec l’accès conditionnel basé sur les applications :

**Condition de plateforme prise en charge**

- iOS
- Android

**Condition d’applications clientes**

- Applications mobiles et clients de bureau

**Contrôle d’accès**

- Demander une application cliente approuvée

Pour plus d’informations, consultez [Accès conditionnel basé sur les applications dans Azure Active Directory](../conditional-access/app-based-conditional-access.md).

---

### <a name="manage-azure-ad-devices-in-the-azure-portal"></a>Gérer les appareils Azure AD dans le portail Azure

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion et inscription des appareils **Fonctionnalité produit :** Protection et sécurité des identités

À partir de maintenant, tous vos appareils connectés à Azure AD et les activités liées à ces appareils sont regroupés dans un même emplacement. Il existe une nouvelle expérience d’administration pour gérer toutes les identités d’appareils et les paramètres dans le portail Azure. Dans cette version, vous pouvez :

- Afficher tous vos appareils qui sont disponibles pour l’accès conditionnel dans Azure AD.
- Afficher les propriétés, notamment celles de vos appareils hybrides joints à Azure AD.
- Rechercher des clés BitLocker pour vos appareils joints à Azure AD, gérer votre appareil avec Intune, et bien plus encore.
- Gérer les paramètres associés aux appareils Azure AD.

Pour plus d’informations, consultez [Gestion des appareils avec le portail Azure](../devices/device-management-azure-portal.md).

---

### <a name="support-for-macos-as-a-device-platform-for-azure-ad-conditional-access"></a>Prise en charge de macOS comme plateforme d’appareils pour l’accès conditionnel Azure AD

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Vous pouvez maintenant inclure (ou exclure) macOS comme condition de plateforme d’appareils dans votre stratégie d’accès conditionnel Azure AD. Avec l’ajout de macOS aux plateformes d’appareils prises en charge, vous pouvez :

- **Inscrire et gérer des appareils Mac OS avec Intune.** Comme pour d’autres plateformes telles qu’iOS et Android, une application de portail d’entreprise est disponible pour macOS afin d’unifier les inscriptions. Cette nouvelle application de portail d’entreprise pour macOS permet d’inscrire un appareil avec Intune auprès d’Azure AD.
- **Assurez-vous que les appareils macOS respectent les stratégies de conformité de votre entreprise, définies dans Intune.** Dans Intune sur le portail Azure, vous pouvez maintenant définir des stratégies de conformité pour les appareils macOS.
- **Restreindre l’accès aux applications dans Azure AD aux seuls appareils macOS conformes.** La création d’une stratégie d’accès conditionnel propose macOS comme plateforme d’appareils. Désormais, vous pouvez créer des stratégies d’accès conditionnel propres à macOS pour l’ensemble des applications cibles dans Azure.

Pour plus d'informations, consultez les pages suivantes :

- [Créer une stratégie de conformité d’appareil pour les appareils macOS avec Intune](/mem/intune/protect/compliance-policy-create-mac-os)
- [Accès conditionnel dans Azure AD](../conditional-access/overview.md)

---

### <a name="network-policy-server-extension-for-azure-ad-multi-factor-authentication"></a>Extension Network Policy Server pour Azure AD Multi-Factor Authentication

**Type :** Nouvelle fonctionnalité **Catégorie de service :**  Authentification multifacteur **Fonctionnalité produit :** Authentification utilisateur

L’extension NPS (Network Policy Server) pour Azure AD Multi-Factor Authentication ajoute des capacités informatiques Multi-Factor Authentication à votre infrastructure d’authentification en utilisant vos serveurs. Avec cette extension NPS, vous pouvez ajouter des vérifications par appel téléphonique, SMS ou application téléphonique à votre flux d’authentification. Et ce, sans avoir à installer, configurer et gérer de nouveaux serveurs.

Cette extension a été créée pour les organisations qui souhaitent protéger des connexions VPN sans déployer le serveur Microsoft Azure Multi-Factor Authentication. L’extension NPS joue le rôle d’adaptateur entre RADIUS et le service informatique Azure AD Multi-Factor Authentication pour fournir un second facteur d’authentification aux utilisateurs fédérés ou synchronisés.

Pour plus d’informations, consultez [Intégrer votre infrastructure Network Policy Server à Azure AD Multi-Factor Authentication](../authentication/howto-mfa-nps-extension.md).

---

### <a name="restore-or-permanently-remove-deleted-users"></a>Restaurer ou supprimer définitivement les utilisateurs supprimés

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Gestion des utilisateurs **Fonctionnalité produit :** Répertoire

Dans le centre d’administration Azure AD, vous pouvez désormais :

- Restaurer un utilisateur supprimé.
- Supprimer définitivement un utilisateur.

**Pour essayer :**

1. Dans le centre d’administration Azure AD, sélectionnez [Tous les utilisateurs](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/UserManagementMenuBlade/All) dans la section **Gérer**.

2. Dans la liste **Afficher**, sélectionnez **Utilisateurs supprimés récemment**.

3. Sélectionnez un ou plusieurs utilisateurs supprimés récemment, puis restaurez-les ou supprimez-les définitivement.

---

### <a name="new-approved-client-apps-for-azure-ad-app-based-conditional-access"></a>Nouvelles applications clientes approuvées pour l’accès conditionnel basé sur les applications Azure AD

**Type :** Fonctionnalité modifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Les applications suivantes ont été ajoutées à la liste des [applications clientes approuvées](../conditional-access/concept-conditional-access-conditions.md#client-apps) :

- Planificateur Microsoft
- Azure Information Protection

Pour plus d'informations, consultez les pages suivantes :

- [Spécification d’application cliente approuvée](../conditional-access/concept-conditional-access-conditions.md#client-apps)
- [Accès conditionnel basé sur les applications Azure AD](../conditional-access/app-based-conditional-access.md)

---

### <a name="use-or-between-controls-in-a-conditional-access-policy"></a>Utiliser l’opérateur « OU » entre des contrôles dans une stratégie d’accès conditionnel

**Type :** Fonctionnalité modifiée **Catégorie de service :** Accès conditionnel **Fonctionnalité produit :** Protection et sécurité des identités

Vous pouvez désormais utiliser l’opérateur « OU » (exiger l’un des contrôles sélectionnés) dans des stratégies de contrôle d’accès conditionnel. Cette fonctionnalité vous permet de créer des stratégies avec un opérateur « OU» entre des contrôles d’accès. Par exemple, vous pouvez utiliser cette fonctionnalité pour créer une stratégie qui oblige l’utilisateur à se connecter avec l’authentification multifacteur OU à utiliser un appareil conforme.

Pour plus d’informations, consultez [Contrôles dans l’accès conditionnel Azure Active Directory](../conditional-access/controls.md).

---

### <a name="aggregation-of-real-time-risk-detections"></a>Agrégation de détections des risques en temps réel

**Type :** Fonctionnalité modifiée **Catégorie de service :** Protection de l’identité **Fonctionnalité produit :** Protection et sécurité des identités

Dans Azure AD Identity Protection, toutes les détections des risques en temps réel qui proviennent de la même adresse IP un jour donné sont maintenant agrégées par type de détection des risques. Cette modification permet de limiter le nombre de détections des risques affichées, sans modifier la sécurité utilisateur.

La détection en temps réel sous-jacente s’exécute à chaque connexion de l’utilisateur. Si vous avez configuré une stratégie de sécurité des risques de connexion pour l’authentification multifacteur ou pour bloquer l’accès, elle se déclenche à chaque connexion présentant un risque.

---

## <a name="october-2017"></a>Octobre 2017

### <a name="deprecate-azure-ad-reports"></a>Déprécier les rapports Azure AD

**Type :** Modification planifiée **Catégorie de service :** Création de rapports **Fonctionnalité produit :** Gestion du cycle de vie des identités

Le portail Azure vous offre :

- Une nouvelle console d’administration d’Azure AD.
- De nouvelles API pour les rapports sur l’activité et la sécurité.

En raison de ces nouvelles fonctionnalités, les API de rapport sous le point de terminaison /reports seront supprimées le 10 décembre 2017.

---

### <a name="automatic-sign-in-field-detection"></a>Détection automatique du champ de connexion

**Type :** Corrigé **Catégorie de service :** Mes applications **Fonctionnalité produit :** Authentification unique

Azure Active Directory prend en charge la détection automatique des champs de connexion pour les applications qui affichent un champ de nom d’utilisateur et un champ de mot de passe HTML. Ces étapes sont documentées sous [Comment capturer automatiquement les champs de connexion d’une application](../manage-apps/troubleshoot-password-based-sso.md#manually-capture-sign-in-fields-for-an-app). Vous pouvez rechercher cette fonctionnalité en ajoutant une application *ne provenant pas de la galerie* sur la page **Applications d’entreprise** du [portail Azure](https://aad.portal.azure.com). En outre dans cette nouvelle application, vous pouvez configurer le mode **Authentification unique** sur **Authentification unique avec mot de passe**, entrer une URL web, puis enregistrer la page.

En raison d’un problème de service, cette fonctionnalité a été temporairement désactivée. Le problème a été résolu et la détection automatique des champs de connexion est à nouveau disponible.

---

### <a name="new-multi-factor-authentication-features"></a>Nouvelles fonctionnalités d’authentification multifacteur

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Authentification multifacteur **Fonctionnalité produit :** Protection et sécurité des identités

La fonctionnalité MFA (Multi-Factor Authentication) joue un rôle essentiel dans la protection de votre organisation. Pour rendre les informations d’identification plus adaptables et l’expérience plus transparente, les fonctionnalités suivantes ont été ajoutées :

- Les résultats de la demande d’accès multifacteur sont intégrés directement dans le rapport de connexion à Azure AD, qui inclut l’accès par programme aux résultats de MFA.
- La configuration de MFA est intégrée plus étroitement dans l’expérience de configuration d’Azure AD dans le portail Azure.

Avec cette préversion publique, la gestion et la création de rapports MFA font partie intégrante de l’expérience de configuration de base d’Azure AD. Désormais, vous pouvez gérer la fonctionnalité de portail de gestion MFA dans l’expérience Azure AD.

Pour en savoir plus, consultez [Référence pour la génération de rapports d’authentification multifacteur dans le portail Azure](../authentication/howto-mfa-reporting.md).

---

### <a name="terms-of-use"></a>Conditions d’utilisation

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Conditions d’utilisation **Fonctionnalité produit :** Conformité

Vous pouvez utiliser des conditions d’utilisation d’Azure AD pour afficher des informations comme des exclusions pertinentes aux obligations juridiques ou de conformité pour les utilisateurs.

Vous pouvez utiliser la fonctionnalité Conditions d’utilisation d’Azure AD dans les scénarios suivants :

- Conditions d’utilisation générales pour tous les utilisateurs de votre organisation
- Conditions d’utilisation spécifiques en fonction des attributs d’un utilisateur (par exemple, médecins et infirmières, ou employés nationaux et internationaux) créées par des groupes dynamiques
- Conditions d’utilisation spécifiques pour l’accès aux applications à fort impact commercial, comme Salesforce

Pour plus d’informations, consultez [Fonctionnalité Conditions d’utilisation d’Azure Active Directory (préversion)](../conditional-access/terms-of-use.md).

---

### <a name="enhancements-to-privileged-identity-management"></a>Améliorations apportées à Privileged Identity Management

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Privileged Identity Management **Fonctionnalité produit :** Privileged Identity Management

Avec Azure Active Directory Privileged Identity Management, vous pouvez gérer, contrôler et surveiller l’accès aux ressources Azure (préversion) au sein de votre organisation par les éléments suivants :

- Abonnements
- Groupes de ressources
- Machines virtuelles

Toutes les ressources dans le portail Azure tirant parti de la fonctionnalité de contrôle d’accès en fonction du rôle (RBAC) d’Azure peuvent utiliser les capacités de gestion de la sécurité et du cycle de vie offertes par Azure AD Privileged Identity Management.

Pour plus d’informations, consultez [PIM pour les ressources Azure (préversion)](../privileged-identity-management/azure-pim-resource-rbac.md).

---

### <a name="access-reviews"></a>Révisions d’accès

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Révisions d’accès **Fonctionnalité produit :** Conformité

Les organisations peuvent utiliser les révisions d’accès (préversion) pour gérer efficacement les appartenances à des groupes et les accès aux applications d’entreprise :

- Vous pouvez le recertifier l’accès utilisateur invité à l’aide des révisions d’accès de leur accès aux applications et appartenances à des groupes. Les réviseurs peuvent décider rapidement si des utilisateurs invités doivent bénéficier d’un accès ininterrompu en fonction des informations fournies par les révisions d’accès.
- Vous pouvez recertifier l’accès d’employé à des applications et l’appartenance à des groupes à l’aide de révisions d’accès.

Vous pouvez collecter les contrôles de révision d’accès dans des programmes importants pour votre organisation afin de suivre les révisions de conformité ou les applications sensibles aux risques.

Pour plus d’informations, consultez [Révisions d’accès Azure AD](../governance/access-reviews-overview.md).

---

### <a name="hide-third-party-applications-from-my-apps-and-the-office-365-app-launcher"></a>Masquer des applications tierces dans Mes applications et le lanceur d’applications d’Office 365

**Type :** Nouvelle fonctionnalité **Catégorie de service :** Mes applications **Fonctionnalité produit :** Authentification unique

Désormais, vous pouvez mieux gérer les applications qui s’affichent sur les portails de vos utilisateurs grâce à une nouvelle propriété permettant de **masquer une application**. Vous pouvez masquer des applications pour plus de clarté en cas d’affichage de mosaïques d’applications pour les services principaux, de vignettes en double et de lanceurs d’applications des utilisateurs. La commande se trouve dans la section **Properties (Propriétés)** de l’application tierce et s’appelle **Visible to user? (Visible par l’utilisateur ?)** . Vous pouvez également masquer une application par programme l’aide de PowerShell.

Pour plus d’informations, consultez [Masquer une application de l’expérience utilisateur dans Azure Active Directory](../manage-apps/hide-application-from-user-portal.md).


**Qu’est-ce qui est disponible ?**

 Dans le cadre de la transition vers la nouvelle console d’administration, deux API permettant de récupérer les journaux d’activité d’Azure AD sont disponibles. Le nouvel ensemble d’API fournit une fonctionnalité enrichie de filtrage et de tri en plus de fournir des activités d’audit et de connexion plus avancées. Les données auparavant disponibles dans les rapports de sécurité sont désormais accessibles via l’API Identity Protection Risk Detections dans Microsoft Graph.


## <a name="september-2017"></a>Septembre 2017

### <a name="hotfix-for-identity-manager"></a>Correctif logiciel pour Identity Manager

**Type :** Fonctionnalité modifiée **Catégorie de service :** Identity Manager **Fonctionnalité produit :** Gestion du cycle de vie des identités

Un package cumulatif (build 4.4.1642.0) est disponible depuis le 25 septembre 2017 pour Identity Manager 2016 Service Pack 1. Ce package cumulatif :

- Résout les problèmes et ajoute des améliorations.
- Est une mise à jour cumulative qui remplace toutes les mises à jour d’Identity Manager 2016 Service Pack 1 jusqu’à la build 4.4.1459.0 d’Identity Manager 2016.
- Requiert que vous disposiez d’Identity Manager 2016 build 4.4.1302.0.

Pour plus d’informations, consultez [Package de correctifs cumulatifs (build 4.4.1642.0) disponible pour le Service Pack 1 de Microsoft Identity Manager 2016](https://support.microsoft.com/help/4021562).

---
