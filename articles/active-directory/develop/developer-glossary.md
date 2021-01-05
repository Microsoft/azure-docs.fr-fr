---
title: Glossaire du développeur de la plateforme d’identité Microsoft | Azure
description: Liste de termes liés aux concepts et fonctionnalités de développeur de la plateforme d’identité Microsoft couramment utilisés.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/24/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: jmprieur, saeeda, jesakowi, nacanuma
ms.openlocfilehash: b02329d624eda440230fb99e02e08c841c5580f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90705994"
---
# <a name="microsoft-identity-platform-developer-glossary"></a>Glossaire du développeur de la plateforme d’identité Microsoft

Cet article contient des définitions pour certains des principaux concepts et terminologies de développeur, qui s’avèrent utiles lors de l’apprentissage du développement d’applications à l’aide de la plateforme d’identité Microsoft.

## <a name="access-token"></a>access token

Un type de [jeton de sécurité](#security-token) émis par un [serveur d’autorisation](#authorization-server) et utilisé par une [application cliente](#client-application) pour accéder à un [serveur de ressources protégé](#resource-server). Apparaissant généralement sous la forme d’un jeton [JSON Web Token (JWT)][JWT], le jeton représente l’autorisation accordée au client par le [propriétaire de la ressource](#resource-owner) pour un niveau d’accès demandé. Le jeton contient toutes les [revendications](#claim) applicables sur le sujet, permettant à l’application cliente de l’utiliser en guise d’informations d’identification lors de l’accès à une ressource donnée. Le propriétaire de la ressource n’a ainsi pas non plus besoin d’exposer ses informations d’identification au client.

Les jetons d’accès ne sont valides que pendant une brève période de temps et ne peuvent pas être révoqués. Un serveur d’autorisation peut également émettre un [jeton d’actualisation](#refresh-token) lors de l’émission du jeton d’accès. Les jetons d’actualisation sont généralement fournis uniquement aux applications clientes confidentielles.

Les jetons d’accès sont parfois qualifiés de « utilisateur + Application » ou « d’application uniquement », selon les informations d’identification représentées. Par exemple, lorsqu’une application cliente utilise :

* [L’octroi d’autorisation « code d’autorisation »](#authorization-grant), l’utilisateur final s’authentifie tout d’abord en tant que propriétaire de la ressource, délégant l’autorisation d’accès à la ressource au client. Le client s’authentifie après, lors de l’obtention du jeton d’accès. Le jeton est alors parfois désigné plus spécifiquement sous le nom de jeton « utilisateur + application », car il représente à la fois l’utilisateur qui a autorisé l’application cliente et l’application.
* [L’octroi d’autorisation « informations d’identification du client »](#authorization-grant), le client fournit l’authentification unique, fonctionnant sans authentification/autorisation du propriétaire des ressources. Le jeton est alors parfois désigné sous le nom de jeton « d’application uniquement ».

Consultez [Informations de référence sur les jetons de la plateforme d’identité Microsoft][AAD-Tokens-Claims] pour en savoir plus.

## <a name="application-id-client-id"></a>ID d’application (ID client)

L’identificateur unique Azure AD est émis pour une inscription d’application qui identifie une application spécifique et les configurations associées. Cet ID d’application ([ID client](https://tools.ietf.org/html/rfc6749#page-15)) est utilisé lors de l’exécution de requêtes d’authentification et est fourni aux bibliothèques d’authentification pendant le développement. L’ID d’application (ID client) n’est pas secret.

## <a name="application-manifest"></a>manifeste d’application

Fonctionnalité fournie par le [portail Azure][AZURE-portal], qui produit une représentation JSON de la configuration d’identité de l’application servant de mécanisme de mise à jour des entités [Application][Graph-App-Resource] et [ServicePrincipal][Graph-Sp-Resource] associées. Pour plus d’informations, consultez [(Understanding the Azure Active Directory application manifest) Connaître le manifeste d’application Azure Active Directory][AAD-App-Manifest].

## <a name="application-object"></a>objet application

Lorsque vous inscrivez/mettez à jour une application dans le [portail Azure][AZURE-portal], le portail crée/met à jour un objet application et un [objet principal du service](#service-principal-object) correspondant pour ce locataire. L’objet application *définit* la configuration d’identité de l’application de manière globale (sur tous les clients où elle dispose d’un accès) et fournit un modèle à partir duquel son ou ses objets principal du service correspondants sont *dérivés* à des fins d’utilisation locale lors de l’exécution (sur un client spécifique).

Pour plus d’informations, consultez [Application and Service Principal Objects (Objets application et principaux du service)][AAD-App-SP-Objects].

## <a name="application-registration"></a>inscription d’application

Afin de pouvoir s’intégrer à Azure AD et déléguer à ce service les fonctions de gestion de l’identité et de l’accès, l’application doit être inscrite auprès d’un [client](#tenant)Azure AD. Lorsque vous inscrivez votre application auprès d’Azure AD, vous fournissez une configuration d’identité pour votre application, ce qui permet à cette dernière de s’intégrer à Azure AD et d’utiliser des fonctionnalités telles que :

* Gestion robuste de l’authentification unique à l’aide de la gestion d’Azure AD Identity Management et de l’implémentation du protocole [OpenID Connect][OpenIDConnect]
* Accès réparti des [applications clientes](#client-application) aux [ressources protégées](#resource-server) via le [serveur d’autorisation](#authorization-server) OAuth 2.0 ;
* [Infrastructure de consentement](#consent) pour la gestion de l’accès client aux ressources protégées en fonction de l’autorisation du propriétaire des ressources.

Pour plus d’informations, consultez [Integrating applications with Azure Active Directory (Intégration d’applications dans Azure Active Directory)][AAD-Integrating-Apps].

## <a name="authentication"></a>Authentification

Action de demander à une partie des informations d’identification légitimes, fournissant la base de la création d’un principal de sécurité à des fins de contrôle de l’identité et de l’accès. Pendant un [octroi d’autorisation OAuth2](#authorization-grant) par exemple, la partie s’authentifiant remplit le rôle de [propriétaire des ressources](#resource-owner) ou [d’application cliente](#client-application), selon l’octroi utilisé.

## <a name="authorization"></a>autorisation

Action de donner à un principal de sécurité authentifié le droit de faire quelque chose. Il existe deux cas d’utilisation principaux dans le modèle de programmation Azure AD :

* Pendant un flux [d’octroi d’autorisation OAuth2](#authorization-grant) : lorsque le [propriétaire des ressources](#resource-owner) accorde l’autorisation à [l’application cliente](#client-application), permettant au client d’accéder aux ressources du propriétaire.
* Pendant l’accès aux ressources par le client : de la manière implémentée par le [serveur de ressources](#resource-server), en utilisant les valeurs de [revendication](#claim) présentes dans le [jeton d’accès](#access-token) pour prendre des décisions de contrôle d’accès.

## <a name="authorization-code"></a>code d’autorisation

« Jeton » à courte durée de vie fourni à une [application cliente](#client-application) par le [point de terminaison d’autorisation](#authorization-endpoint) dans le cadre du flux « code d’autorisation », l’un des quatre [octrois d’autorisation](#authorization-grant) OAuth2. Le code est renvoyé à l’application cliente en réponse à l’authentification d’un [propriétaire de ressources](#resource-owner), indiquant que le propriétaire des ressources a délégué l’autorisation d’accéder aux ressources. Dans le cadre de ce flux, le code est échangé plus tard contre un [jeton d’accès](#access-token).

## <a name="authorization-endpoint"></a>point de terminaison d’autorisation

Un des points de terminaison implémentés par le [serveur d’autorisation](#authorization-server), utilisé pour interagir avec le [propriétaire des ressources](#resource-owner) afin de fournir un [octroi d’autorisation](#authorization-grant) pendant un flux d’octroi d’autorisation OAuth2. Selon le flux d’octroi d’autorisation utilisé, l’octroi fourni peut varier et prendre notamment la forme d’un [code d’autorisation](#authorization-code) ou d’un [jeton de sécurité](#security-token).

Pour plus d’informations, consultez les [types d’octroi d’autorisation][OAuth2-AuthZ-Grant-Types] et les [points de terminaison d’autorisation][OAuth2-AuthZ-Endpoint] de la spécification OAuth2, ainsi que la [spécification OpenIDConnect][OpenIDConnect-AuthZ-Endpoint].

## <a name="authorization-grant"></a>octroi d’autorisation

Informations d’identification représentant [l’autorisation](#authorization) accordée à une [application cliente](#client-application) par le [propriétaire des ressources](#resource-owner) d’accéder à ses ressources protégées. Une application cliente peut utiliser un des [quatre types d’octroi définis par l’infrastructure d’autorisation OAuth2][OAuth2-AuthZ-Grant-Types] pour obtenir une autorisation, selon le type ou les exigences du client : octroi d’un code d’autorisation, octroi d’informations d’identification client, octroi implicite et octroi d’informations de mot de passe du propriétaire de la ressource. Les informations d’identification renvoyées au client prennent la forme d’un [jeton d’accès](#access-token) ou d’un [code d’autorisation](#authorization-code) (échangé plus tard contre un jeton d’accès), selon le type de flux d’octroi d’autorisation utilisé.

## <a name="authorization-server"></a>serveur d’autorisation

Comme le définit l’[infrastructure d’autorisation OAuth2][OAuth2-Role-Def], serveur responsable de l’émission de jetons d’accès pour le [client](#client-application) après avoir authentifié le [propriétaire des ressources](#resource-owner) et obtenu son autorisation. Une [application cliente](#client-application) interagit avec le serveur d’autorisation lors de l’exécution via ses points de terminaison [d’autorisation](#authorization-endpoint) et de [jeton](#token-endpoint), conformément aux [octrois d’autorisation](#authorization-grant) définis par l’infrastructure d’autorisation OAuth2.

Dans le cas de l’intégration d’applications de plateforme d’identité Microsoft, cette dernière implémente le rôle de serveur d’autorisation pour les applications Azure AD et les API de service Microsoft, par exemple les [API Graph Microsoft][Microsoft-Graph].

## <a name="claim"></a>revendication

Un [jeton de sécurité](#security-token) contient des revendications qui fournissent des assertions sur une entité (telle qu’une [application cliente](#client-application) ou un [propriétaire de ressources](#resource-owner)) à une autre entité (telle que le [serveur de ressources](#resource-server)). Les revendications sont des paires nom/valeur qui relaient des informations sur le sujet du jeton (par exemple, le principal de sécurité authentifié par le [serveur d’autorisation](#authorization-server)). Les revendications présentes dans un jeton donné dépendent de plusieurs variables, notamment le type de jeton, le type d’informations d’identification utilisées pour authentifier le sujet et la configuration de l’application.

Consultez [Informations de référence sur les jetons de la plateforme d’identité Microsoft][AAD-Tokens-Claims] pour en savoir plus.

## <a name="client-application"></a>d’application cliente

Comme le définit l’[infrastructure d’autorisation OAuth2][OAuth2-Role-Def], application qui effectue des demandes de ressources protégées au nom du [propriétaire des ressources](#resource-owner). Le terme « cliente » n’implique pas de caractéristiques d’implémentation matérielle particulières (par exemple, si l’application s’exécute sur un serveur, un ordinateur de bureau ou d’autres appareils).

Une application cliente demande [l’autorisation](#authorization) à un propriétaire de ressources de participer à un [flux d’autorisation OAuth2](#authorization-grant) et peut accéder aux API/données au nom du propriétaire des ressources. L’infrastructure d’autorisation OAuth2 [définit deux types de clients][OAuth2-Client-Types], « confidentiel » et « public », en fonction de la capacité du client à préserver la confidentialité de ses informations d’identification. Les applications peuvent implémenter un [client web (confidentiel)](#web-client) s’exécutant sur un serveur web, un [client natif (public)](#native-client) installé sur un appareil ou un [client basé sur un agent utilisateur (public)](#user-agent-based-client) s’exécutant dans le navigateur d’un appareil.

## <a name="consent"></a>consentement

Processus par lequel un [propriétaire de ressources](#resource-owner) octroie à une [application cliente](#client-application) l’accès à des ressources protégées avec des [autorisations spécifiques](#permissions), en son nom. Selon les autorisations demandées par le client, un administrateur ou un utilisateur sera invité à donner son consentement pour autoriser l’accès aux données de l’entreprise ou à ses données individuelles respectivement. Notez que dans un scénario d’application [mutualisée](#multi-tenant-application), le [principal du service](#service-principal-object) de l’application est également enregistré dans le client de l’utilisateur donnant son consentement.

Consultez l’[infrastructure de consentement](consent-framework.md) pour plus d’informations.

## <a name="id-token"></a>Jeton d’ID

[Jeton de sécurité][OpenIDConnect-ID-Token] [OpenID Connect](#security-token) fourni par le [point de terminaison d’autorisation](#authorization-server) d’un [serveur d’autorisation](#authorization-endpoint) et contenant des [revendications](#claim) se rapportant à l’authentification d’un [propriétaire de ressources](#resource-owner) utilisateur final. Comme un jeton d’accès, un jeton d’ID est représenté sous forme de jeton [JSON Web Token (JWT)][JWT] signé numériquement. À la différence d’un jeton d’accès cependant, les revendications d’un jeton d’ID ne sont pas utilisés à des fins liées à l’accès aux ressources et plus particulièrement pour le contrôle d’accès.

Consultez [Informations de référence sur les jetons de la plateforme d’identité Microsoft][AAD-Tokens-Claims] pour en savoir plus.

## <a name="microsoft-identity-platform"></a>Plateforme d’identité Microsoft

La plateforme d’identités Microsoft est une évolution de la plateforme de développeur et de service d’identité Azure Active Directory (Azure AD). Elle permet aux développeurs de générer des applications qui connectent toutes les identités Microsoft et obtiennent des jetons pour appeler Microsoft Graph, d’autres APIs Microsoft ou des API que des développeurs ont créées. C’est une plateforme complète qui se compose d’un service d’authentification, de bibliothèques, de fonctionnalités d’inscription et de configuration d’application, d’une documentation de développement exhaustive et d’exemples de code et autres contenus destinés aux développeurs. La plateforme d’identités Microsoft prend en charge les protocoles standard tels qu’OAuth 2.0 et OpenID Connect.

## <a name="multi-tenant-application"></a>application mutualisée

Classe d’applications qui permet aux utilisateurs configurés dans n’importe quel [client](#tenant) Azure AD, y compris ceux autres que celui où le client est enregistré, de se connecter et de donner leur [consentement](#consent). Les applications [clientes natives](#native-client) sont mutualisées par défaut, tandis que les applications [clientes web](#web-client) et [ressources web/API](#resource-server) peuvent choisir entre une architecture à client unique et une architecture mutualisée. Par opposition, une application web inscrite en tant qu’application à client unique permet uniquement des connexions depuis des comptes d’utilisateurs configurés dans le même client que celui dans lequel l’application est inscrite.

Pour plus d’informations, consultez [Comment connecter un utilisateur Azure Active Directory (AD) à l’aide du modèle d’application mutualisée][AAD-Multi-Tenant-Overview].

## <a name="native-client"></a>client natif

Type d’ [application cliente](#client-application) installé en mode natif sur un appareil. Étant donné que tout le code est exécuté sur un appareil, il est considéré comme un client « public » en raison de son incapacité à stocker les informations d’identification de façon privée/confidentielle. Pour plus d’informations, consultez les [types et profils de clients OAuth2][OAuth2-Client-Types].

## <a name="permissions"></a>autorisations

Une [application cliente](#client-application) accède à un [serveur de ressources](#resource-server) en déclarant des demandes d’autorisation. Deux types sont disponibles :

* Les autorisations déléguées, qui spécifient un accès [en fonction de l’étendue](#scopes) en utilisant l’autorisation déléguée donnée par le [propriétaire des ressources](#resource-owner) connecté, sont présentées à la ressource lors de l’exécution sous forme de [revendications « scp »](#claim) dans le [jeton d’accès](#access-token) du client.
* Les autorisations d’application, qui spécifient un accès [en fonction du rôle](#roles) en utilisant les informations d’identification/de l’identité de l’application cliente, sont présentées à la ressource lors de l’exécution sous forme de [revendications « de rôles »](#claim) dans le jeton d’accès du client.

Elles apparaissent également pendant le processus de [consentement](#consent) , donnant à l’administrateur ou au propriétaire des ressources la possibilité d’autoriser/de refuser l’accès client aux ressources de son client.

Vous pouvez configurer des requêtes d’autorisation dans la page **Autorisations de l’API** d’une application dans le [portail Azure][AZURE-portal] en sélectionnant les « Autorisations déléguées » et les « Autorisations d’application » souhaitées (ces dernières nécessitent l’appartenance au rôle Administrateur général). Du fait qu’un [client public](#client-application) ne peut pas conserver de façon sécurisée les informations d’identification, il peut demander uniquement des autorisations déléguées, alors qu’un [client confidentiel](#client-application) peut demander des autorisations déléguées et des autorisations d’application. L’[objet application](#application-object) du client stocke les autorisations déclarées dans sa [propriété requiredResourceAccess][Graph-App-Resource].

## <a name="refresh-token"></a>jeton d'actualisation

Un type de [jeton de sécurité](#security-token) émis par un [serveur d’autorisation](#authorization-server) et utilisé par une [application cliente](#client-application) pour demander un nouveau [jeton d’accès](#access-token) avant l’expiration du jeton d’accès. Généralement sous la forme d’un [JSON Web Token (JWT)][JWT].

Contrairement aux jetons d’accès, les jetons d’actualisation peuvent être révoqués. Si une application cliente tente de demander un nouveau jeton d’accès à l’aide d’un jeton d’actualisation qui a été révoqué, le serveur d’autorisation refuse la demande et l’application cliente n’a plus l’autorisation d’accéder au [serveur de ressources](#resource-server) pour le compte du [propriétaire de la ressource](#resource-owner).

## <a name="resource-owner"></a>propriétaire de la ressource

Comme le définit [l’infrastructure d’autorisation OAuth2][OAuth2-Role-Def], entité capable d’octroyer l’accès à une ressource protégée. Lorsque le propriétaire de ressource est une personne, on le désigne sous le nom d’utilisateur final. Par exemple, lorsqu’une [application cliente](#client-application) souhaite accéder à la boîte aux lettres d’un utilisateur via l’[API Graph Microsoft][Microsoft-Graph], l’autorisation du propriétaire de ressources de la boîte aux lettres est nécessaire.

## <a name="resource-server"></a>serveur de ressources

Comme le définit l’[infrastructure d’autorisation OAuth2][OAuth2-Role-Def], serveur hébergeant des ressources protégées capable d’accepter et de répondre aux demandes de ressources protégées effectuées par les [applications clientes](#client-application) qui présentent un [jeton d’accès](#access-token). Également appelé serveur de ressources protégées ou application de ressources.

Un serveur de ressources expose des API et applique l’accès à ses ressources protégées via des [étendues](#scopes) et des [rôles](#roles), en s’appuyant sur l’infrastructure d’autorisation OAuth 2.0. Citons, par exemple, l’[API Microsoft Graph][Microsoft-Graph] qui fournit un accès aux données du client Azure AD, et les API Microsoft 365 qui fournissent un accès à des données telles que le courrier et le calendrier.

Tout comme une application cliente, la configuration d’identité d’une application de ressources est établie via [l’inscription](#application-registration) dans un client Azure AD, fournissant à la fois l’objet application et l’objet principal du service. Certaines API fournies par Microsoft, telles que l’API Microsoft Graph, proposent des principaux du service préinscrits mis à disposition dans tous les clients lors du provisionnement.

## <a name="roles"></a>roles

Comme les [étendues](#scopes), les rôles offrent au [serveur de ressources](#resource-server) un moyen de régir l’accès à ses ressources protégées. Il en existe deux types : un rôle d’utilisateur implémente le contrôle d’accès en fonction du rôle pour les utilisateurs/groupes qui requièrent un accès à la ressource, tandis qu’un rôle d’application implémente le même contrôle pour les [applications clientes](#client-application) qui requièrent cet accès.

Les rôles sont des chaînes définies par les ressources (par exemple, « Expense approver », « Read-only » ou « Directory.ReadWrite.All »), gérées dans le [portail Azure][AZURE-portal] via le [manifeste d’application](#application-manifest) de la ressource et stockées dans la [propriété appRoles][Graph-Sp-Resource] de cette dernière. Le portail Azure est également utilisé pour affecter des utilisateurs aux rôles « utilisateur » et pour configurer les [autorisations d’application](#permissions) du client, lui permettant d’accéder à un rôle « application ».

Pour une présentation détaillée des rôles d’application exposés par l’API Microsoft Graph, consultez [Graph API Permission Scopes (Étendues des autorisations de l’API Graph)][Graph-Perm-Scopes]. Pour obtenir un exemple d’implémentation pas à pas, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide du portail Azure][AAD-RBAC].

## <a name="scopes"></a>étendues

Comme les [rôles](#roles), les étendues offrent au [serveur de ressources](#resource-server) un moyen de régir l’accès à ses ressources protégées. Les portées sont utilisées pour implémenter un contrôle d’accès [en fonction de la portée][OAuth2-Access-Token-Scopes], pour une [application cliente](#client-application) qui s’est vu attribuer un accès délégué à la ressource par son propriétaire.

Les portées sont des chaînes définies par les ressources (par exemple, « Mail.Read » ou « Directory.ReadWrite.All »), gérées dans le [portail Azure][AZURE-portal] via le [manifeste d’application](#application-manifest) de la ressource et stockées dans la [propriété oauth2Permissions][Graph-Sp-Resource] de cette dernière. Le portail Azure est également utilisé pour configurer les [autorisations déléguées](#permissions) de l’application cliente, qui lui permettent d’accéder à une portée.

Une convention d’affectation de noms recommandée consiste à utiliser le format « ressource.opération.contrainte ». Pour une présentation détaillée des étendues exposées par l’API Microsoft Graph, consultez [Graph API Permission Scopes (Étendues des autorisations de l’API Graph)][Graph-Perm-Scopes]. Pour les étendues exposées par les services Microsoft 365, consultez [Référence des autorisations des API Microsoft 365][O365-Perm-Ref].

## <a name="security-token"></a>jeton de sécurité

Document signé contenant des revendications, tel qu’un jeton OAuth2 ou une assertion SAML 2.0. Pour un [octroi d’autorisation](#authorization-grant) OAuth2, un [jeton d’accès](#access-token) (OAuth2), un [jeton d’actualisation](#refresh-token) et un [jeton d’ID](https://openid.net/specs/openid-connect-core-1_0.html#IDToken) sont des types de jetons de sécurité, qui sont tous deux implémentés sous forme de jetons [JSON Web Token (JWT)][JWT].

## <a name="service-principal-object"></a>objet principal du service

Lorsque vous inscrivez/mettez à jour une application dans le [portail Azure][AZURE-portal], le portail crée/met à jour un [objet application](#application-object) et un objet principal du service correspondant pour ce locataire. L’objet application *définit* la configuration d’identité de l’application de manière globale (sur tous les clients où l’application associée s’est vue octroyer un accès) et constitue le modèle à partir duquel son ou ses objets principal du service correspondants sont *dérivés* à des fins d’utilisation locale lors de l’exécution (sur un client spécifique).

Pour plus d’informations, consultez [Application and Service Principal Objects (Objets application et principaux du service)][AAD-App-SP-Objects].

## <a name="sign-in"></a>connexion

Processus par lequel une [application cliente](#client-application) initialise l’authentification de l’utilisateur final et enregistre l’état associé à des fins d’acquisition d’un [jeton de sécurité](#security-token) et d’adaptation de l’étendue de la session de l’application à cet état. L’état peut inclure des artefacts tels que les informations de profil utilisateur et des informations dérivées des revendications de jeton.

La fonction d’ouverture de session d’une application est généralement utilisée pour mettre en œuvre l’authentification unique (SSO). Elle peut également être précédée d’une fonction « inscription », qui sert de point d’entrée pour qu’un utilisateur final accède à une application (lors de la première connexion). La fonction d’inscription sert à rassembler et à rendre persistant un état supplémentaire propre à l’utilisateur et peut nécessiter le [consentement de l’utilisateur](#consent).

## <a name="sign-out"></a>déconnexion

Processus de désactivation de l’authentification d’un utilisateur final par lequel l’état utilisateur associé à [l’application cliente](#client-application) pendant la [connexion](#sign-in) est détaché.

## <a name="tenant"></a>tenant

Une instance d’un répertoire Azure AD est désignée sous le nom de client Azure AD. Celui-ci fournit plusieurs fonctionnalités, notamment :

* un service de registre pour les applications intégrées
* l’authentification des comptes utilisateurs et des applications enregistrées
* les points de terminaison REST requis pour prendre en charge différents protocoles, notamment OAuth2 et SAML, y compris le [point de terminaison d’autorisation](#authorization-endpoint), le [point de terminaison de jeton](#token-endpoint) et le point de terminaison « commun » utilisé par les [applications mutualisées](#multi-tenant-application).

Les locataires Azure AD sont créés/associés avec des abonnements Azure et Microsoft 365 pendant l’inscription, fournissant des fonctionnalités de gestion des identités et des accès pour l’abonnement. Les administrateurs d’abonnement Azure peuvent également créer d’autres locataires Azure AD via le portail Azure. Pour plus d’informations sur les diverses méthodes permettant d’accéder à un locataire, consultez [Obtention d’un locataire Azure Active Directory][AAD-How-To-Tenant]. Pour plus d’informations sur la relation entre les abonnements et un locataire Azure AD et pour obtenir des instructions sur l’association ou l’ajout d’un abonnement à un locataire Azure AD, consultez [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory][AAD-How-Subscriptions-Assoc].

## <a name="token-endpoint"></a>point de terminaison de jeton

Un des points de terminaison implémentés par le [serveur d’autorisation](#authorization-server) pour prendre en charge les [octrois d’autorisation](#authorization-grant) OAuth2. En fonction de l’octroi, il peut être utilisé pour acquérir un [jeton d’accès](#access-token) (et les jetons « d’actualisation » liés) à un [client](#client-application) ou un [jeton d’ID](#id-token) lorsqu’il est utilisé avec le protocole [OpenID Connect][OpenIDConnect].

## <a name="user-agent-based-client"></a>Client basé sur un agent utilisateur

Type d’[application cliente](#client-application) qui télécharge du code à partir d’un serveur web et s’exécute au sein d’un agent utilisateur (par exemple, un navigateur web), comme une application monopage (SPA). Étant donné que tout le code est exécuté sur un appareil, il est considéré comme un client « public » en raison de son incapacité à stocker les informations d’identification de façon privée/confidentielle. Pour plus d’informations, consultez les [types et profils de clients OAuth2][OAuth2-Client-Types].

## <a name="user-principal"></a>principal de l’utilisateur

De la même manière qu’un objet principal du service est utilisé pour représenter une instance d’application, un objet principal de l’utilisateur est un autre type de principal de sécurité, qui représente un utilisateur. Le [type de ressource d’utilisateur][Graph-User-Resource] Microsoft Graph définit le schéma d’un objet utilisateur, y compris les propriétés propres à l’utilisateur, telles que le nom et le prénom, le nom d’utilisateur principal, l’appartenance à un rôle de répertoire, etc. Cela permet de fournir la configuration d’identité utilisateur utilisée par Azure AD pour établir le principal d’un utilisateur lors de l’exécution. Le principal de l’utilisateur est utilisé pour représenter un utilisateur authentifié lors de l’authentification unique, de l’enregistrement de la délégation du [consentement](#consent), de la prise de décisions de contrôle d’accès, etc.

## <a name="web-client"></a>clientes web

Type d’ [application cliente](#client-application) qui exécute tout le code sur un serveur web et est capable de fonctionner comme un client « confidentiel » en stockant de manière sécurisée ses informations d’identification sur le serveur. Pour plus d’informations, consultez les [types et profils de clients OAuth2][OAuth2-Client-Types].

## <a name="next-steps"></a>Étapes suivantes

Le [Guide du développeur de la plateforme d’identité Microsoft][AAD-Dev-Guide] est la page d’accueil rassemblant toutes les rubriques liées au développement de la plateforme d’identité Microsoft, notamment une présentation de l’[intégration d’applications][AAD-How-To-Integrate] et les principes de base de l’[authentification de la plateforme d’identité Microsoft et des scénarios d’authentification pris en charge][AAD-Auth-Scenarios]. Vous trouverez également des exemples de code et des tutoriels qui vous permettront d'être rapidement opérationnel sur [GitHub](https://github.com/azure-samples?utf8=%E2%9C%93&q=active%20directory&type=&language=).

Utilisez la section des commentaires suivante pour fournir des commentaires et nous aider à affiner et à présenter notre contenu, y compris les demandes de nouvelles définitions ou la mise à jour de définitions existantes !

<!--Image references-->

<!--Reference style links -->
[AAD-App-Manifest]:reference-app-manifest.md
[AAD-App-SP-Objects]:app-objects-and-service-principals.md
[AAD-Auth-Scenarios]:authentication-scenarios.md
[AAD-Dev-Guide]:azure-ad-developers-guide.md
[Graph-Perm-Scopes]: /graph/permissions-reference
[Graph-App-Resource]: /graph/api/resources/application
[Graph-Sp-Resource]: /graph/api/resources/serviceprincipal?view=graph-rest-beta
[Graph-User-Resource]: /graph/api/resources/user
[AAD-How-Subscriptions-Assoc]:../fundamentals/active-directory-how-subscriptions-associated-directory.md
[AAD-How-To-Integrate]: ./active-directory-how-to-integrate.md
[AAD-How-To-Tenant]:quickstart-create-new-tenant.md
[AAD-Integrating-Apps]:quickstart-v1-integrate-apps-with-azure-ad.md
[AAD-Multi-Tenant-Overview]:howto-convert-app-to-be-multi-tenant.md
[AAD-Security-Token-Claims]: ./active-directory-authentication-scenarios/#claims-in-azure-ad-security-tokens
[AAD-Tokens-Claims]:access-tokens.md
[AZURE-portal]: https://portal.azure.com
[AAD-RBAC]: ../../role-based-access-control/role-assignments-portal.md
[JWT]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32
[Microsoft-Graph]: https://developer.microsoft.com/graph
[O365-Perm-Ref]: /graph/permissions-reference
[OAuth2-Access-Token-Scopes]: https://tools.ietf.org/html/rfc6749#section-3.3
[OAuth2-AuthZ-Endpoint]: https://tools.ietf.org/html/rfc6749#section-3.1
[OAuth2-AuthZ-Grant-Types]: https://tools.ietf.org/html/rfc6749#section-1.3
[OAuth2-Client-Types]: https://tools.ietf.org/html/rfc6749#section-2.1
[OAuth2-Role-Def]: https://tools.ietf.org/html/rfc6749#page-6
[OpenIDConnect]: https://openid.net/specs/openid-connect-core-1_0.html
[OpenIDConnect-AuthZ-Endpoint]: https://openid.net/specs/openid-connect-core-1_0.html#AuthorizationEndpoint
[OpenIDConnect-ID-Token]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken