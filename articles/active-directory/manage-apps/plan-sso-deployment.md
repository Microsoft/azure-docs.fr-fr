---
title: Planifier le déploiement de l’authentification unique Azure Active Directory
description: Guide concernant la planification, le déploiement et la gestion de l’authentification unique au sein d’une organisation.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 06/10/2020
ms.author: baselden
ms.reviewer: jeedes
ms.collection: M365-identity-device-management
ms.custom: has-adal-ref
ms.openlocfilehash: 14520c7c448231045b36e025c2c92332f9daac53
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658721"
---
# <a name="plan-a-single-sign-on-deployment"></a>Planifier un déploiement de l’authentification unique

Avec l’authentification unique (SSO), l’utilisateur accède à l’ensemble des applications et des ressources dont il a besoin en ne connectant qu’une seule fois à l’aide d’un même compte d’utilisateur. Avec l’authentification unique, les utilisateurs peuvent accéder à toutes les applications dont ils ont besoin sans avoir à se réauthentifier à chaque fois.

## <a name="benefits-of-sso"></a>Avantages de l’authentification unique

Lorsque les utilisateurs s’authentifient auprès d’applications dans Azure Active Directory (Azure AD), l’authentification unique (SSO) constitue la méthode la plus sécurisée et la plus pratique. 

De nombreuses organisations s’appuient sur des applications SaaS, comme Microsoft 365, Box et Salesforce, pour accroître la productivité des utilisateurs finaux. Avant, le personnel informatique devait créer et mettre à jour chaque compte d’utilisateur dans chacune des applications SaaS, et les utilisateurs devaient mémoriser un mot de passe pour chacune de ces applications.

La Place de marché Azure propose plus de 3 000 applications avec connexion SSO préintégrée, ce qui facilite leur intégration dans les locataires.

## <a name="licensing"></a>Licence

- **Gestion des licences Azure AD** - Pour les applications SaaS pré-intégrées, l’authentification unique est gratuite. Toutefois, le nombre d’objets contenus dans votre annuaire et les fonctionnalités que vous souhaitez déployer peuvent nécessiter des licences supplémentaires. Pour obtenir la liste complète des conditions de licence, consultez la page [Tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).
- **Licences d’application** - Le type de licence que vous choisissez pour vos applications SaaS doit répondre aux besoins de votre entreprise. Contactez le propriétaire de l’application pour déterminer si les utilisateurs affectés à l’application disposent de licences adaptées à leurs rôles dans l’application. Si Azure AD gère l’attribution automatique d’utilisateurs en fonction des rôles, les rôles attribués dans Azure AD doivent s’aligner sur le nombre de licences détenues au sein de l’application. Si le nombre de licences détenues dans l’application est incorrect, cela peut entraîner des erreurs pendant l’attribution ou la mise à jour des utilisateurs.

## <a name="plan-your-sso-team"></a>Organiser l’équipe SSO

- **Impliquez les bonnes personnes** - Lorsque des projets technologiques échouent, cela est souvent dû à une mauvaise entente sur les attentes (impact, résultats ou responsabilités). Pour éviter un tel cas de figure, [veillez à faire appel aux bonnes personnes](../fundamentals/active-directory-deployment-plans.md) et à ce que les parties prenantes comprennent bien leur rôle.
- **Prévoyez une bonne communication** - La communication est essentielle à la réussite de tout nouveau service. Communiquez de manière proactive avec vos utilisateurs sur ce qui va changer, à quel moment les changements seront appliqués et comment ils peuvent obtenir de l’aide en cas de problème. Passez en revue les différentes [façons dont les utilisateurs finaux peuvent accéder à leurs applications compatibles SSO](end-user-experiences.md), et adaptez vos communications à chaque cas de figure. 

## <a name="plan-your-sso-protocol"></a>Planifier l’implémentation du protocole SSO

Une implémentation SSO basée sur les protocoles de fédération améliore la sécurité, la fiabilité et l’expérience utilisateur, et est plus facile à mettre en œuvre. De nombreuses applications sont préintégrées à Azure AD et sont fournies avec un [guide étape par étape](../saas-apps/tutorial-list.md). Ces guides sont disponibles sur la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/). Vous trouverez des informations détaillées sur chaque méthode SSO dans l’article [S’authentifier avec l’authentification unique auprès des applications dans Azure Active Directory](what-is-single-sign-on.md).

Il existe deux principales façons de permettre à vos utilisateurs d’utiliser l’authentification unique pour vos applications :

- **Avec l’authentification unique fédérée**, Azure AD authentifie l’utilisateur dans l’application en utilisant son compte Azure AD. Cette méthode est applicable pour les applications qui prennent en charge les protocoles comme SAML 2.0, WS-Federation ou OpenID Connect. Il s’agit du mode d’authentification unique qui offre le plus de possibilités. Nous recommandons d’utiliser l’authentification SSO fédérée avec Azure AD (si prise en charge par l’application) plutôt que l’authentification unique basée sur les mots de passe ou sur ADFS.

- **Avec l’authentification unique basée sur les mots de passe**, les utilisateurs finaux se connectent à l’application à l’aide d’un nom d’utilisateur et d’un mot de passe la première fois qu’ils y accèdent. Après la première connexion, Azure AD fournit le nom d’utilisateur et le mot de passe à l’application. L’authentification unique par mot de passe permet de sécuriser le stockage et la lecture des mots de passe des applications avec une extension de navigateur web ou une application mobile. Cette option s’appuie sur le processus de connexion existant fourni par l’application, permet aux administrateurs de gérer les mots de passe et ne nécessite pas que l’utilisateur connaisse le mot de passe.

### <a name="considerations-for-federation-based-sso"></a>Points à noter concernant l’authentification unique basée sur la fédération

- **Utilisation d’OpenID Connect et d’OAuth** : si l’application à laquelle vous vous connectez la prend en charge, utilisez la méthode OIDC/OAuth 2.0 afin d’activer l’authentification unique pour cette application. Cette méthode nécessite moins de configuration et offre une expérience utilisateur plus riche. Pour plus d’informations, consultez [OAuth 2.0](../develop/v2-oauth2-auth-code-flow.md), [OpenID Connect 1.0](../develop/v2-protocols-oidc.md) et [Guide du développeur Azure Active Directory](../develop/index.yml).
- **Configuration des point de terminaison pour l’authentification unique SAML** - Si vous utilisez le SAML, vos développeurs auront besoin de certaines informations avant de configurer l’application. Pour plus d’informations, consultez [Configurer l’authentification unique SAML](configure-saml-single-sign-on.md).
- **Gestion des certificats pour l’authentification unique SAML** : lorsque vous activez l’authentification unique fédérée pour votre application, Azure AD crée un certificat qui est par défaut valide pendant trois ans. Vous pouvez personnaliser la date d’expiration de ce certificat si nécessaire. Vérifiez que vous disposez de processus permettant de renouveler les certificats avant leur expiration. Pour plus d’informations, consultez [Gestion des certificats Azure AD](./manage-certificates-for-federated-single-sign-on.md).

### <a name="considerations-for-password-based-sso"></a>Points à noter concernant l’authentification basée sur les mots de passe

L’utilisation d’Azure AD pour l’authentification basée sur les mots de passe nécessite le déploiement d’une extension de navigateur qui permettra de récupérer les informations d’identification de façon sécurisée et de renseigner les formulaires de connexion. Configurez un mécanisme permettant de déployer l’extension à grande échelle avec des [navigateurs pris en charge](../user-help/my-apps-portal-end-user-access.md). Options disponibles :

- [Stratégie de groupe pour Internet Explorer](./access-panel-deployment-plan.md)
- [Configuration Manager pour Internet Explorer](/configmgr/core/clients/deploy/deploy-clients-to-windows-computers)
- [Téléchargement et configuration pilotés par l’utilisateur pour Chrome, Firefox, Microsoft Edge ou Internet Explorer](../user-help/my-apps-portal-end-user-access.md)

Pour plus d’informations, consultez [Configurer l’authentification unique par mot de passe](./configure-password-single-sign-on-non-gallery-applications.md).

#### <a name="capturing-login-forms-metadata-for-applications-that-arent-in-the-gallery"></a>Capture des métadonnées des formulaires de connexion pour les applications qui ne figurent pas dans la galerie

Microsoft prend en charge la capture des métadonnées d’une application web pour le stockage du mot de passe (capture des champs Nom d’utilisateur et Mot de passe). Accédez à l’URL de connexion pendant le processus de configuration de l’application afin de capturer les métadonnées des formulaires. Demandez au propriétaire de l’application l’URL de connexion exacte. Ces informations sont utilisées pendant le processus d’authentification, qui mappe les informations d’identification Azure AD à l’application lors de l’authentification.

Pour plus d’informations, consultez [Qu’est-ce que l’accès aux applications et l’authentification unique avec Azure AD ? – Authentification unique basée sur les mots de passe](./what-is-single-sign-on.md).

#### <a name="indications-that-metadata-in-forms-needs-to-be-recaptured"></a>Indications que les métadonnées des formulaires doivent être recapturées

Si la mise en page HTML des applications est modifiée, il est possible que vous deviez recapturer les métadonnées pour vous aligner sur les modifications. Le plus souvent, les symptômes qui indiquent que la mise en page HTML a été modifiée sont les suivants :

- Les utilisateurs signalent qu’après avoir cliqué sur l’application, ils restent « coincés » sur la page de connexion
- Les utilisateurs signalent que le nom d’utilisateur et le mot de passe ne sont pas renseignés

#### <a name="shared-accounts"></a>Comptes partagés

Du point de vue de la connexion, les applications avec comptes partagés ne sont pas différentes de celles de la galerie qui utilisent l’authentification unique par mot de passe pour les utilisateurs individuels. Toutefois, certaines étapes supplémentaires sont nécessaires lorsque vous planifiez et configurez une application conçue pour utiliser des comptes partagés :

1. Collaborez avec les utilisateurs des applications métier pour documenter les éléments suivants :
   1. Les groupes d’utilisateurs de l’organisation qui utiliseront l’application
   1. Les informations d’identification existantes de l’application qui sont associées à chaque groupe d’utilisateur 
1. Pour chaque combinaison groupe d’utilisateurs-informations d’identification, créez un groupe de sécurité localement ou dans le cloud, selon vos besoins.
1. Réinitialisez les informations d’identification partagées. Une fois que l’application est déployée dans Azure AD, les utilisateurs n’ont plus besoin du mot de passe du compte partagé. Dans la mesure où Azure AD stocke le mot de passe, vous pouvez en créer un très long et très complexe. 
1. Si l’application le prend en charge, configurez la substitution automatique du mot de passe. De cette façon, même l’administrateur qui a effectué la configuration initiale ne connaîtra pas le mot de passe du compte partagé. 

## <a name="plan-your-authentication-method"></a>Planifier la méthode d’authentification

Le choix de la méthode d’authentification est une première décision essentielle dans la configuration d’une solution d’identité hybride Azure AD. L’implémentation de la méthode d’authentification est configurée à l’aide d’Azure AD Connect, qui provisionne également les utilisateurs dans le cloud.

Pour choisir une méthode d’authentification, vous devez prendre en compte l’infrastructure existante, le temps nécessaire à l’implémentation, sa complexité et les coûts associés. Ces facteurs sont différents pour chaque organisation et peuvent varier au fil du temps. Vous devez choisir celle qui est la mieux adaptée à votre cas. Pour plus d’informations, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](../hybrid/choose-ad-authn.md).

## <a name="plan-your-security-and-governance"></a>Planifier la sécurité et la gouvernance 

L’identité est le nouvel élément central de la sécurité et des investissements, car les périmètres réseau sont devenus de plus en plus poreux et de moins en moins efficaces face à l’explosion des appareils BYOD et des applications cloud. 

### <a name="plan-access-reviews"></a>Planifier les révisions d’accès

Les [révisions d’accès](../governance/create-access-review.md) permettent aux organisations de gérer efficacement les appartenances à des groupes, les accès aux applications d’entreprise et les attributions de rôles. Vous devez prévoir de réviser les accès des utilisateurs régulièrement pour vérifier que seules les personnes autorisées continuent de bénéficier d’un accès.

Voici certains éléments clés à planifier lorsque vous organisez des révisions d’accès :

1. La fréquence de révisions d’accès qui correspond à vos besoins métier. Vous pouvez effectuer une révision une fois par semaine, une fois par mois ou une fois par an, et même, à la demande.

1. Créez des groupes qui représentent les réviseurs des rapports d’accès aux applications. Les réviseurs que vous choisissez doivent très bien connaître l’application, ses utilisateurs cibles et ses cas d’utilisation.

1. Une révision d’accès implique le fait de retirer les autorisations d’accès aux utilisateurs qui ne sont plus autorisés. Prévoyez comment gérer les potentielles demandes de support provenant d’utilisateurs non autorisés. Dans Azure AD, un utilisateur supprimé conserve cet état pendant 30 jours, au cours desquels il peut être restauré par un administrateur, si nécessaire. Au bout de 30 jours, cet utilisateur est définitivement supprimé. À l’aide du portail Azure Active Directory, un administrateur général peut, de façon explicite, supprimer définitivement un utilisateur récemment supprimé avant que cette période arrive à son terme.

### <a name="plan-auditing"></a>Planifier l’audit

Azure AD fournit des [rapports contenant des insights techniques et commerciaux](../reports-monitoring/overview-reports.md). 

Vous pouvez utiliser des rapports de sécurité et d’activité. Les rapports de sécurité montrent les utilisateurs et les connexions considérés comme étant à risque. Les rapports d’activité vous permettent de mieux comprendre le comportement des utilisateurs de votre organisation, par le biais d’informations détaillées sur les activités de connexion et de pistes d’audit pour toutes les connexions. Vous pouvez utiliser des rapports pour gérer les risques, augmenter la productivité et superviser la conformité.

| Type de rapport | Révision d’accès | Rapports de sécurité | Rapport de connexion |
|-------------|---------------|------------------|----------------|
| Révision | Autorisations et utilisation des applications | Comptes potentiellement compromis | Qui accède aux applications |
| Actions potentielles | Auditer l’accès, révoquer les autorisations | Révoquer l’accès, forcer la réinitialisation de la sécurité | Révoquer l’accès |

Azure AD conserve la plupart des données d’audit pendant 30 jours et rend les données disponibles via le portail d’administration Azure ou via une API que vous pouvez télécharger dans vos systèmes d’analyse.

### <a name="consider-using-microsoft-cloud-application-security"></a>Utilisation de Microsoft Cloud App Security

Microsoft Cloud App Security (MCAS) est une solution Cloud Access Security Broker (CASB). Elle vous permet de bénéficier d’une visibilité accrue sur vos applications et services cloud, de fonctionnalités d’analytique sophistiquées pour identifier et combattre les cybermenaces, et de nouveaux moyens pour maîtriser le déplacement de vos données.

Le déploiement de MCAS vous permet de faire ce qui suit :

- Utiliser Cloud Discovery pour mapper et identifier votre environnement cloud et les applications cloud utilisées par votre organisation
- Approuver et désapprouver les applications de votre cloud
- Utiliser des connecteurs faciles à déployer qui tirent parti des API de fournisseurs pour améliorer la visibilité et la gouvernance des applications auxquelles vous vous connectez
- Utiliser la protection Contrôle d’application par accès conditionnel pour obtenir une visibilité en temps réel et contrôler l’accès et les activités au sein de vos applications cloud
- Bénéficier d’un contrôle permanent en définissant et en optimisant des stratégies

Le contrôle de session Microsoft Cloud Application Security (MCAS) est disponible pour la plupart des navigateurs, la plupart des plateformes et la plupart des systèmes d’exploitation. Les applications mobiles et les applications de bureau peuvent également être bloquées ou autorisées. Grâce à l’intégration native à Azure AD, les applications qui sont configurées avec SAML et les applications OpenID Connect avec l’authentification unique à Azure AD peuvent être prises en charge (notamment [plusieurs applications proposées](/cloud-app-security/proxy-intro-aad)).

Pour plus d’informations sur MCAS, consultez [Vue d’ensemble de Microsoft Cloud App Security](/cloud-app-security/what-is-cloud-app-security). MCAS est un service par abonnement basé sur l’utilisateur. Pour connaître les informations relatives aux licences, consultez la [feuille de données sur les licences MCAS](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE2NXYO).

### <a name="use-conditional-access"></a>Utiliser l’accès conditionnel

Avec l’accès conditionnel, vous pouvez automatiser les décisions de contrôle d’accès basées sur des critères concernant vos applications cloud.

Des stratégies d’accès conditionnel sont appliquées au terme de l’authentification premier facteur. L’accès conditionnel ne constitue donc pas une première ligne de défense dans des scénarios comme les attaques par déni de service, mais il peut utiliser des signaux émanant de ces événements pour déterminer l’accès. Par exemple, il peut s’appuyer sur le niveau de risque de la connexion, sur l’emplacement de la requête, etc. Pour plus d’informations sur l’accès conditionnel, consultez [Vue d’ensemble](../conditional-access/plan-conditional-access.md) et [Plan de déploiement](../conditional-access/plan-conditional-access.md).

## <a name="azure-sso-technical-requirements"></a>Exigences techniques de l’authentification unique Azure

La section suivante explique en détail comment configurer votre application, notamment les environnements, les points de terminaison, le mappage de revendications, les attributs, les certificats et les protocoles nécessaires. Vous aurez besoin de ces informations pour configurer l’authentification unique dans le [portail Azure AD](https://portal.azure.com/).

### <a name="authentication-mechanism-details"></a>Informations détaillées sur le mécanisme d’authentification

Microsoft fournit un tutoriel pour toutes les applications SaaS pré-intégrées. Vous n’aurez donc pas besoin de ces informations. Si l’application n’est pas disponible dans la Place de marché ou dans la galerie, vous devrez peut-être collecter les éléments de données suivants :

- **Le fournisseur d’identité que l’application utilise actuellement pour l’authentification unique, le cas échéant**, par exemple : AD FS, PingFederate, Okta
- **Les protocoles pris en charge par l’application cible**, par exemple, SAML 2.0, OpenID Connect, OAuth, l’authentification basée sur les formulaires, WS-Fed, WS-Trust
- **Le protocole actuellement configuré avec Azure AD**, par exemple, SAML 2.0 ou 1.1, OpenID Connect, OAuth, basé sur les formulaires, WS-Fed

### <a name="attribute-requirements"></a>Exigences relatives aux attributs

Il existe un ensemble préconfiguré d’attributs et de mappages d’attributs entre les objets utilisateur Azure AD et les objets utilisateur de chaque application SaaS. Certaines applications gèrent d’autres types d’objets, tels que des groupes. Planifiez le mappage des attributs d’utilisateur entre Azure AD et votre application, puis [personnalisez les mappages d’attributs par défaut](../app-provisioning/customize-application-attributes.md) selon les besoins de votre entreprise.

### <a name="certificate-requirements"></a>Configuration requise des certificats

Le certificat de l’application doit être à jour. Si ce n’est pas le cas, les utilisateurs risquent de ne pas pouvoir accéder à l’application. La plupart des certificats d’application SaaS sont valides pendant 36 mois. Cette durée peut être modifiée dans le panneau de l’application. Assurez-vous de documenter l’expiration et de choisir comment gérer le renouvellement des certificats. 

Vous pouvez gérer vos certificats de deux façons. 

- **Substitution automatique des certificats** : Microsoft prend en charge la [substitution des clés de signature dans Azure AD](../develop/active-directory-signing-key-rollover.md). Il s’agit de la méthode que nous recommandons pour gérer les certificats. Toutefois, tous les éditeurs de logiciels ne prennent pas en charge ce scénario.

- **Mise à jour manuelle** : chaque application a son propre certificat, qui expire à une date définie. Avant que le certificat de l’application n’expire, créez un certificat puis envoyez-le à l’éditeur de logiciels. Ces informations peuvent être tirées (pull) à partir des métadonnées de fédération. [En savoir plus sur les métadonnées de fédération](../azuread-dev/azure-ad-federation-metadata.md)

## <a name="implement-sso"></a>Implémenter l’authentification unique

Suivez les étapes ci-dessous pour planifier et déployer votre solution dans votre organisation :

### <a name="user-configuration-for-sso"></a>Configuration des utilisateurs pour l’authentification unique

- **Identifier les utilisateurs de test**

   Contactez le propriétaire de l’application et demandez-lui de créer au moins trois utilisateurs de test au sein de l’application. Vérifiez que les informations que vous allez utiliser comme identificateur principal sont correctement renseignées et qu’elles correspondent à un attribut qui est disponible dans Azure AD. Dans la plupart des cas, cela sera mappé sur « NameID » pour les applications SAML. Pour les jetons JWT, il s’agit du « preferred_username ».
   
   Créez l’utilisateur manuellement en tant qu’utilisateur basé sur le cloud dans Azure AD ou synchronisez l’utilisateur à partir du système local en utilisant le moteur de synchronisation Azure AD Connect. Vérifiez que les informations correspondent aux revendications envoyées à l’application.

- **Configurer l’authentification unique**

   Dans la [liste des applications](../saas-apps/tutorial-list.md), accédez au tutoriel sur l’authentification unique correspondant à votre application, puis suivez les étapes pour configurer votre application SaaS.

   Si vous ne trouvez pas votre application, consultez la [documentation sur les applications personnalisées](./configure-saml-single-sign-on.md). Vous verrez comment ajouter une application qui ne se trouve pas dans la galerie Azure AD.

   Si vous le souhaitez, vous pouvez utiliser les revendications émises dans le jeton SAML pour l’application d’entreprise en vous aidant de la [documentation Microsoft](../develop/active-directory-claims-mapping.md). Vérifiez que cela correspond à ce que vous vous attendez à recevoir dans la réponse SAML pour votre application. Si vous rencontrez des problèmes lors de la configuration, suivez nos instructions sur le [débogage de l’intégration SSO](./debug-saml-sso-issues.md).

### <a name="provide-sso-change-communications-to-end-users"></a>Informer les utilisateurs finaux du changement d’authentification

Créez votre plan de communication. Informez vos utilisateurs finaux lorsqu’un changement est prévu et lorsqu’il a été effectué. Dites-leur ce qu’ils peuvent faire et comment obtenir de l’aide.

### <a name="verify-end-user-scenarios-for-sso"></a>Vérifier différents scénarios utilisateur impliquant l’authentification unique

Vous pouvez utiliser les cas de test suivants pour exécuter des tests sur les appareils personnels et les appareils d’entreprise, afin de vérifier que vos configurations SSO fonctionnent comme prévu. Dans les scénarios ci-dessous, un utilisateur accède à une URL d’application et suit un flux d’authentification lancé par le fournisseur de services.

| Scénario | Résultat attendu du flux d’authentification unique lancé par l’utilisateur |
|----------|---------------------------------------------------|
| Connexion à l’application avec Internet Explorer sur le réseau de l’entreprise. | L’authentification Windows intégrée (IWA) s’effectue sans autre invite. |
| Connexion à l’application à l’aide d’Internet Explorer en dehors du réseau de l’entreprise, avec une nouvelle tentative de connexion. | Invite basée sur les formulaires sur le serveur AD FS. L’utilisateur se connecte et le navigateur l’invite à entrer ses informations d’identification pour l’authentification multifacteur. |
| Connexion à l’application à l’aide d’Internet Explorer en dehors du réseau de l’entreprise, avec une session en cours et aucune précédente authentification multifacteur. | L’utilisateur ne reçoit pas d’invite pour l’authentification premier facteur. L’utilisateur est invité à s’authentifier via l’authentification multifacteur. |
| Connexion à l’application avec Internet Explorer en étant en dehors du réseau de l’entreprise, avec une session en cours et une authentification multifacteur effectuée pour cette session. | L’utilisateur ne reçoit pas d’invite pour l’authentification premier facteur. L’utilisateur ne reçoit pas l’authentification multifacteur. L’utilisateur s’authentifie dans l’application à l’aide de l’authentification unique. |
| Connexion à l’application avec Firefox/Chrome/Safari en étant en dehors du réseau de l’entreprise, avec une session en cours et une authentification multifacteur effectuée pour cette session. | L’utilisateur ne reçoit pas d’invite pour l’authentification premier facteur. L’utilisateur ne reçoit pas l’authentification multifacteur. L’utilisateur s’authentifie dans l’application à l’aide de l’authentification unique. |
| Connexion à l’application à l’aide de Chrome/Firefox/Safari en dehors du réseau de l’entreprise, avec une nouvelle tentative de connexion. | Invite basée sur les formulaires sur le serveur AD FS. L’utilisateur se connecte et le navigateur l’invite à entrer ses informations d’identification pour l’authentification multifacteur. |
| Connexion à l’application avec Chrome/Firefox en étant sur le réseau de l’entreprise, avec une session en cours. | L’utilisateur ne reçoit pas d’invite pour l’authentification premier facteur. L’utilisateur ne reçoit pas l’authentification multifacteur. L’utilisateur s’authentifie dans l’application à l’aide de l’authentification unique. |
| Connexion à l’application avec l’application mobile, avec une nouvelle tentative de connexion. | Invite basée sur les formulaires sur le serveur AD FS. L’utilisateur se connecte et le client ADAL l’invite à entrer ses informations d’identification pour l’authentification multifacteur. |
| L’utilisateur non autorisé tente de se connecter à l’application avec l’URL de connexion. | Invite basée sur les formulaires sur le serveur AD FS. L’utilisateur ne parvient pas à se connecter avec l’authentification premier facteur. |
| L’utilisateur autorisé tente de se connecter, mais entre un mot de passe incorrect. | L’utilisateur accède à l’URL de l’application et reçoit une erreur indiquant que le nom d’utilisateur ou le mot de passe est incorrect. |
| L’utilisateur autorisé clique sur un lien dans un e-mail et est déjà authentifié. | L’utilisateur clique sur l’URL et est connecté à l’application sans invite supplémentaire. |
| L’utilisateur autorisé clique sur un lien dans un e-mail et n’est pas encore authentifié. | L’utilisateur clique sur l’URL et est invité à s’authentifier avec l’authentification premier facteur. |
| L’utilisateur autorisé se connecte à l’application avec l’application mobile (lancée par le fournisseur de services), avec une nouvelle tentative de connexion. | Invite basée sur les formulaires sur le serveur AD FS. L’utilisateur se connecte et le client ADAL l’invite à entrer ses informations d’identification pour l’authentification multifacteur. |
| L’utilisateur non autorisé tente de se connecter à l’application avec l’URL de connexion (lancée par le fournisseur de services). | Invite basée sur les formulaires sur le serveur AD FS. L’utilisateur ne parvient pas à se connecter avec l’authentification premier facteur. |
| L’utilisateur autorisé tente de se connecter, mais entre un mot de passe incorrect.| L’utilisateur accède à l’URL de l’application et reçoit une erreur indiquant que le nom d’utilisateur ou le mot de passe est incorrect. |
| L’utilisateur autorisé se déconnecte puis se reconnecte. | Si une URL de déconnexion est configurée, l’utilisateur est déconnecté de tous les services et est invité à s’authentifier. |
| L’utilisateur autorisé se déconnecte puis se reconnecte. | Si aucune URL de déconnexion n’est configurée, l’utilisateur est automatiquement connecté à l’aide d’un jeton existant provenant de la session de navigateur Azure AD existante. |
| L’utilisateur autorisé clique sur un lien dans un e-mail et est déjà authentifié. | L’utilisateur clique sur l’URL et est connecté à l’application sans invite supplémentaire. |
| L’utilisateur autorisé clique sur un lien dans un e-mail et n’est pas encore authentifié. | L’utilisateur clique sur l’URL et est invité à s’authentifier avec l’authentification premier facteur. |

## <a name="manage-sso"></a>Gérer l’authentification unique

Cette section dresse la liste des exigences et des recommandations concernant la gestion de l’authentification unique.

### <a name="required-administrative-roles"></a>Rôles d’administration nécessaires

Utilisez toujours le rôle ayant le moins de privilèges possible pour accomplir une tâche dans Azure Active Directory. Microsoft recommande de [passer en revue les différents rôles disponibles](../roles/permissions-reference.md) et de choisir celui qui répond le mieux aux besoins de chaque utilisateur de l’application. Certains rôles devront peut-être être appliqués temporairement, puis supprimés une fois le déploiement terminé.

| Utilisateur| Rôles | Rôle Azure AD (si nécessaire) |
|--------|-------|-----------------------------|
| Administrateur du support technique | Support de niveau 1 | None |
| Administrateur d’identité | Configurer et déboguer quand des problèmes impactent Azure AD | Administrateur général |
| Administrateur d’application | Attestation de l’utilisateur dans l’application, configuration des utilisateurs disposant d’autorisations | None |
| Administrateurs de l’infrastructure | Propriétaire de la substitution de certificat | Administrateur général |
| Chef d’entreprise/Partie prenante | Attestation de l’utilisateur dans l’application, configuration des utilisateurs disposant d’autorisations | None |

Nous vous recommandons d’utiliser [Privileged Identity Management](../privileged-identity-management/pim-configure.md) (PIM) pour gérer vos rôles dans le but de fournir aux utilisateurs qui disposent d’autorisations sur l’annuaire des fonctionnalités supplémentaires au niveau des audits, du contrôle et des révisions d’accès.

### <a name="sso-certificate-lifecycle-management"></a>Gestion du cycle de vie des certificats SSO

Il est important d’identifier les bons rôles ainsi que les listes de distribution e-mail chargées de gérer le cycle de vie du certificat de signature entre Azure AD et l’application configurée avec l’authentification unique. Voici quelques rôles clés qu’il est recommandé de mettre en place :

- Un responsable pour mettre à jour les propriétés utilisateur dans l’application
- Un responsable d’astreinte pour le support en cas de bogue
- Une liste de distribution e-mail surveillée de près pour les notifications de modification des certificats

La durée de vie maximale d’un certificat est de trois ans. Nous vous recommandons d’établir un processus pour gérer les modifications de certificat entre Azure AD et votre application. Cela vous permet d’éviter, ou du moins, de réduire les risques d’une panne due à l’expiration du certificat ou de forcer la substitution du certificat.

### <a name="rollback-process"></a>Processus de restauration

Lorsque vous aurez terminé les tests basés sur vos cas de test, vous devrez faire passer votre application en production. Le passage en production signifie que vous allez implémenter vos configurations planifiées et testées dans votre locataire de production, puis les déployer. Toutefois, il est important d’avoir un plan de repli en cas d’échec de votre déploiement. Si la configuration de l’authentification unique échoue pendant le déploiement, vous devez savoir comment réduire les risques de panne et l’impact sur vos utilisateurs.

Ce sont les méthodes d’authentification qui sont disponibles dans votre application qui déterminent la meilleure stratégie à adopter. Veillez à toujours mettre à disposition des propriétaires d’application une documentation détaillée expliquant les étapes exactes à effectuer pour revenir à l’état de configuration de connexion d’origine, au cas où votre déploiement rencontre des problèmes.

- **Si votre application prend en charge plusieurs fournisseurs d’identité**, par exemple LDAP, AD FS et Ping, ne supprimez pas la configuration de l’authentification unique existante pendant le déploiement. Désactivez-la plutôt pendant la migration au cas où vous devriez la rebasculer. 

- **Si votre application ne prend pas en charge plusieurs fournisseurs d’identité** mais permet aux utilisateurs de se connecter à l’aide de l’authentification basée sur les formulaires (nom d’utilisateur/mot de passe), assurez-vous que les utilisateurs pourront revenir à cette méthode en cas d’échec du déploiement de la nouvelle configuration SSO.

### <a name="access-management"></a>Gestion de l’accès

Nous vous recommandons de choisir une approche par étapes lorsque vous gérez l’accès aux ressources. Les approches les plus courantes sont notamment : l’utilisation de groupes locaux qui sont synchronisés via Azure AD Connect, la [création de groupes dynamiques dans Azure AD en fonction des attributs utilisateur](../enterprise-users/groups-dynamic-membership.md) et la [création de groupes en libre-service](../enterprise-users/groups-self-service-management.md) gérés dans Azure AD par un propriétaire de ressources.

### <a name="monitor-security"></a>Superviser la sécurité

Nous vous recommandons de passer en revue les différents aspects de la sécurité des applications SaaS à une fréquence régulière et d’effectuer les actions correctives nécessaires.

### <a name="troubleshooting"></a>Dépannage

Les liens suivants présentent des scénarios de dépannage. Vous pouvez créer un guide contenant ces scénarios et ces résolutions pour l’équipe du support technique.

#### <a name="consent-issues"></a>Problèmes de consentement

- [Erreur de consentement inattendue](./application-sign-in-unexpected-user-consent-prompt.md)

- [Erreur de consentement de l’utilisateur](./application-sign-in-unexpected-user-consent-error.md)

#### <a name="sign-in-issues"></a>Problèmes de connexion

- [Problèmes de connexion à partir du portail personnalisé](./application-sign-in-other-problem-access-panel.md)

- [Problèmes de connexion à partir de Mes applications](./application-sign-in-other-problem-access-panel.md)

- [Erreur sur la page de connexion de l’application](./application-sign-in-problem-application-error.md)

- [Problème de connexion à une application Microsoft](./application-sign-in-problem-first-party-microsoft.md)

#### <a name="sso-issues-for-applications-listed-in-the-azure-application-gallery"></a>Problèmes d’authentification unique pour des applications listées dans la galerie d’applications Azure

- [Problème d’authentification unique par mot de passe pour des applications listées dans la galerie d’applications Azure](./troubleshoot-password-based-sso.md) 

- [Problème d’authentification unique fédérée pour des applications listées dans la galerie d’applications Azure](./application-sign-in-problem-federated-sso-gallery.md)   

#### <a name="sso-issues-for-applications-not-listed-in-the-azure-application-gallery"></a>Problèmes d’authentification unique pour des applications non listées dans la galerie d’applications Azure

- [Problème d’authentification unique par mot de passe pour des applications non listées dans la galerie d’applications Azure](./troubleshoot-password-based-sso.md) 

- [Problème d’authentification unique fédérée pour des applications non listées dans la galerie d’applications Azure](./application-sign-in-problem-federated-sso-gallery.md)

## <a name="next-steps"></a>Étapes suivantes

[Déboguer l’authentification unique SAML](./debug-saml-sso-issues.md)

[Mappage de revendications pour les applications via PowerShell](../develop/active-directory-claims-mapping.md)

[Personnalisation des revendications émises dans un jeton SAML](../develop/active-directory-saml-claims-customization.md)

[Protocole SAML d’authentification unique](../develop/single-sign-on-saml-protocol.md)

[Protocole SAML de déconnexion unique](../develop/single-sign-out-saml-protocol.md)

[Azure AD B2B](../external-identities/what-is-b2b.md) (pour les utilisateurs externes tels que les partenaires et les fournisseurs)

[Accès conditionnel Azure AD](../conditional-access/overview.md)

[Azure Identity Protection](../identity-protection/overview-identity-protection.md)

[Accès par authentification unique](./what-is-single-sign-on.md)

[Tutoriel sur l’authentification unique dans les applications](../saas-apps/tutorial-list.md)

[Téléchargement de livre blanc : Plan de déploiement de l’authentification unique](https://aka.ms/SSODeploymentPlan)