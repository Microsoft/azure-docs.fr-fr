---
title: Publier des applications locale avec le proxy d’application Azure AD
description: Découvrez comment utiliser le proxy d’application pour publier des applications web locales en externe pour les utilisateurs distants. Découvrez l’architecture du proxy d’application, ses connecteurs, ses méthodes d’authentification et ses avantages de sécurité.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 12/31/2020
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 02b08e6fcdd6714886f8afea1c0a461eb6128fe4
ms.sourcegitcommit: 8be279f92d5c07a37adfe766dc40648c673d8aa8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97832639"
---
# <a name="using-azure-ad-application-proxy-to-publish-on-premises-apps-for-remote-users"></a>Utilisation du proxy d’application Azure AD pour publier des applications locales pour les utilisateurs distants

Azure Active Directory (Azure AD) offre de nombreuses fonctionnalités de protection des utilisateurs, des applications et des données dans le cloud et localement. En particulier, la fonctionnalité Proxy d’application Azure AD peut être implémentée par les professionnels de l’informatique pour publier des applications web locales en externe. Les utilisateurs distants qui doivent accéder aux applications internes peuvent alors y accéder de manière sécurisée.

Dans l’environnement de travail moderne, il est crucial de pouvoir sécuriser l’accès aux applications internes pour les utilisateurs qui ne sont pas sur votre réseau. Avec des scénarios comme BYOD (apportez votre propre appareil) et les appareils mobiles, les professionnels de l’informatique doivent répondre à deux objectifs :

* Permettre aux utilisateurs finaux d’être productifs à tout moment et en tout lieu
* Protéger les ressources de l’entreprise à tout moment

De nombreuses entreprises pensent contrôler et protéger leurs ressources tant qu’elles se trouvent dans les limites de leur réseau d’entreprise. En réalité, dans l’environnement de travail numérique d’aujourd’hui, ces limites ont été étendues par les appareils mobiles, et les ressources et services dans le cloud. La gestion de la protection des identités des utilisateurs et des données qu’ils stockent sur leurs appareils et applications est devenue complexe.

Vous utilisez peut-être déjà Azure AD pour gérer les utilisateurs dans le cloud qui doivent accéder aux applications Microsoft 365 et autres applications SaaS, ainsi qu’aux applications web hébergées localement. Si vous avez déjà Azure AD, vous pouvez l’utiliser comme point de contrôle pour autoriser un accès fluide et sécurisé à vos applications locales. Ou bien, vous vous demandez encore peut-être s’il faut migrer vers le cloud. Dans ce cas, vous pouvez commencer votre parcours vers le cloud en implémentant le proxy d’application et en prenant les premières mesures pour créer une base d’identités solide.

La liste ci-dessous, bien que non exhaustive, illustre certaines possibilités que vous offre le proxy d’application quand vous l’implémentez dans un scénario de coexistence hybride :

* Publier des applications web localement en externe de manière simplifiée sans zone DMZ
* Prendre en charge l’authentification unique (SSO) sur les appareils, ressources et applications dans le cloud et localement
* Prendre en charge l’authentification multifacteur pour les applications cloud et locales
* Tirer parti rapidement des fonctionnalités du cloud avec la sécurité du cloud Microsoft
* Centraliser la gestion des comptes d’utilisateur
* Centraliser le contrôle des identités et la sécurité
* Ajouter ou supprimer automatiquement l’accès des utilisateurs aux applications basées sur l’appartenance au groupe

Cet article décrit comment Azure AD et le proxy d’application offrent aux utilisateurs distants une expérience d’authentification unique (SSO). Les utilisateurs se connectent de manière sécurisée aux applications locales sans VPN, ni serveurs à double hébergement et règles de pare-feu. Cet article vous aide à comprendre comment le proxy d’application apporte les fonctionnalités et les avantages de sécurité du cloud à vos applications web locales. Il décrit aussi l’architecture et les topologies possibles.

## <a name="remote-access-in-the-past"></a>L’accès à distance, hier

Avant, votre plan de contrôle pour protéger les ressources internes contre les attaquants tout en facilitant l’accès aux utilisateurs distants consistait à utiliser une zone DMZ ou un réseau de périmètre. Toutefois, les solutions de VPN et de proxy inverse déployées dans la zone DMZ pour que les clients externes puissent accéder aux ressources d’entreprise ne conviennent pas au monde du cloud. Elles présentent généralement les inconvénients suivants :

* Coûts matériels
* Gestion de la sécurité (mise à jour corrective, supervision des ports, etc.)
* Authentification des utilisateurs en périphérie
* Authentification des utilisateurs sur les serveurs web dans le réseau de périmètre
* Gestion de l’accès VPN pour les utilisateurs distants avec la distribution et la configuration des logiciels clients VPN. Gestion des serveurs joints à un domaine dans la zone DMZ, qui peut être vulnérable aux attaques extérieures.

Aujourd’hui, parce que nous évoluons dans un environnement massivement tourné vers le cloud, Azure AD apparaît comme la meilleure option pour contrôler qui et quoi accède à votre réseau. Le proxy d’application Azure AD s’intègre aux technologies cloud et d’authentification modernes, comme les applications SaaS et les fournisseurs d’identité. Cette intégration permet aux utilisateurs d’accéder aux applications où qu’ils soient. Non seulement le proxy d’application convient mieux à l’environnement de travail numérique d’aujourd’hui, mais il est plus sécurisé que les solutions de VPN et de proxy inverse, et plus facile à implémenter. Les utilisateurs distants peuvent accéder aux applications locales de la même manière qu’ils accèdent aux applications Microsoft et aux autres applications SaaS intégrées à Azure AD. Vous n’avez pas besoin de modifier ou de mettre à jour vos applications pour qu’elles fonctionnent avec le Proxy d’application. Par ailleurs, le proxy d’application ne vous oblige pas à ouvrir des connexions entrantes dans votre pare-feu. Avec le proxy d’application, vous avez simplement à le configurer et à l’oublier.

## <a name="the-future-of-remote-access"></a>L’accès à distance, demain

Dans l’environnement de travail numérique d’aujourd’hui, les utilisateurs travaillent n’importe où avec plusieurs appareils et applications. La seule constante est l’identité de l’utilisateur. C’est pourquoi la première étape pour sécuriser un réseau aujourd’hui est d’utiliser les fonctionnalités de [gestion d’identités d’Azure AD](../../security/fundamentals/identity-management-overview.md) comme plan de contrôle de la sécurité. Un modèle qui utilise l’identité comme plan de contrôle est généralement constitué des composants suivants :

* Un fournisseur d’identité pour assurer le suivi des utilisateurs et les informations qui les concernent.
* Un annuaire d’appareils pour maintenir une liste des appareils qui ont accès aux ressources d’entreprise. Cet annuaire comprend les informations de l’appareil (par exemple, type d’appareil, intégrité etc.).
* Un service d’évaluation de stratégie pour déterminer si un utilisateur et un appareil sont conformes à la stratégie définie par les administrateurs de sécurité.
* La possibilité d’accorder ou de refuser l’accès aux ressources de l’organisation.

Le proxy d’application Azure AD vous permet d’assurer le suivi des utilisateurs qui doivent accéder aux applications web publiées localement et dans le cloud. Il fournit un point de gestion central pour ces applications. Bien que cela n’est pas obligatoire, nous vous recommandons d’activer aussi l’accès conditionnel Azure AD. En définissant des conditions sur la façon dont les utilisateurs s’authentifient et obtiennent l’accès, vous contrôlez davantage qui accède à quelles applications.

**Remarque :** Comprenez bien que le proxy d’application est censé remplacer le VPN ou le proxy inverse pour les utilisateurs itinérants (ou distants) qui ont besoin d’accéder aux ressources internes. Il n’est pas destiné aux utilisateurs internes sur le réseau d’entreprise. Les utilisateurs internes qui utilisent inutilement le proxy d’application peuvent ralentir les performances de manière inattendue et indésirable.

![Azure Active Directory et toutes vos applications](media/what-is-application-proxy/azure-ad-and-all-your-apps.png)

### <a name="an-overview-of-how-app-proxy-works"></a>Vue d’ensemble du fonctionnement du proxy d’application

Le proxy d’application est un service Azure AD que vous configurez dans le portail Azure. Il vous permet de publier un point de terminaison d’URL HTTP/HTTPS public externe dans le cloud Azure, qui se connecte à une URL de serveur d’applications interne dans votre organisation. Ces applications web locales peuvent être intégrées à Azure AD pour prendre en charge l’authentification unique. Les utilisateurs finaux peuvent accéder aux applications web locales de la même manière qu’ils accèdent aux applications Microsoft 365 et autres applications SaaS.

Cette fonctionnalité comprend les composants suivants : le service de proxy d’application, qui s’exécute dans le cloud, le connecteur de proxy d’application, qui est un agent léger s’exécutant sur un serveur local, et Azure AD, qui est le fournisseur d’identité. Les trois composants fonctionnent ensemble pour fournir à l’utilisateur une expérience d’authentification unique afin d’accéder aux applications web locales.

Après s’être connectés, les utilisateurs externes peuvent accéder aux applications web locales à l’aide d’une URL connue ou [Mes applications](../user-help/my-apps-portal-end-user-access.md) sur leurs appareils de bureau ou iOS/MAC. Par exemple, le proxy d’application peut fournir l’accès à distance et l’authentification unique aux services Bureau à distance, aux sites SharePoint, à Tableau, à Qlik, à Outlook sur le web et aux applications métier (LOB).

![Architecture du proxy d’application Azure AD](media/what-is-application-proxy/azure-ad-application-proxy-architecture.png)

### <a name="authentication"></a>Authentification

Il existe plusieurs façons de configurer une application pour l’authentification unique et la méthode que vous choisissez dépend de l’authentification utilisée par votre application. Le proxy d’application prend en charge les types d’applications suivants :

* Applications web
* API web que vous voulez exposer aux applications enrichies sur différents appareils
* Applications hébergées derrière une passerelle Bureau à distance
* Applications clientes complètes intégrées à la [bibliothèque d’authentification Microsoft (MSAL)](../develop/v2-overview.md)

Le proxy d’application fonctionne avec les applications qui utilisent le protocole d’authentification natif suivant :

* **[Authentification Windows intégrée (IWA)](application-proxy-configure-single-sign-on-with-kcd.md).** Pour IWA, les connecteurs de proxy d’application utilisent la délégation Kerberos contrainte (KCD) pour authentifier les utilisateurs dans l’application.

Le proxy d’application prend également en charge les protocoles d’authentification suivants avec l’intégration de tiers ou dans des scénarios de configuration spécifiques :

* [**Authentification basée sur l’en-tête**](./application-proxy-configure-single-sign-on-with-headers.md). Cette méthode d’authentification utilise un service d’authentification tiers appelé PingAccess et est utilisée quand l’application se sert d’en-têtes pour l’authentification. Dans ce scénario, l’authentification est gérée par PingAccess.
* [**Authentification basée sur des formulaires ou un mot de passe**](application-proxy-configure-single-sign-on-password-vaulting.md). Avec cette méthode d’authentification, les utilisateurs se connectent à l’application avec un nom d’utilisateur et un mot de passe quand ils y accèdent la première fois. Après la première connexion, Azure AD fournit le nom d’utilisateur et le mot de passe à l’application. Dans ce scénario, l’authentification est gérée par Azure AD.
* [**Authentification SAML**](application-proxy-configure-single-sign-on-on-premises-apps.md). L’authentification unique SAML est prise en charge pour les applications qui utilisent les protocoles SAML 2.0 ou WS-Federation. Grâce à l’authentification unique SAML, Azure AD s’authentifie auprès de l’application en utilisant le compte Azure AD de l’utilisateur.

Pour plus d’informations sur les méthodes prises en charge, consultez [Choix d’une méthode d’authentification unique](sso-options.md#choosing-a-single-sign-on-method).

### <a name="security-benefits"></a>Avantages en matière de sécurité

La solution d’accès à distance offerte par le proxy d’application et Azure AD prend en charge plusieurs avantages de sécurité pour les clients, notamment :

* **Accès authentifié**. Le proxy d’application convient mieux pour publier des applications avec une [préauthentification](application-proxy-security.md#authenticated-access) afin de garantir que seules les connexions authentifiées accèdent à votre réseau. Pour les applications publiées avec une préauthentification, aucun trafic n’est autorisé à passer par le service de proxy d’application vers votre environnement local sans un jeton valide. La préauthentification, par nature, bloque un grand nombre d’attaques ciblées, car seules les identités authentifiées peuvent accéder à l’application back-end.
* **Accès conditionnel**. Vous pouvez appliquer des contrôles de stratégie plus riches avant que les connexions à votre réseau soient établies. Avec l’accès conditionnel, vous pouvez définir des restrictions sur le trafic que vous autorisez à accéder à votre application back-end. Vous créez des stratégies qui limitent les connexions en fonction de l’emplacement, de la force de l’authentification et du profil de risque de l’utilisateur. Avec l’évolution de l’accès conditionnel, d’autres contrôles sont ajoutés pour renforcer la sécurité, par exemple, l’intégration à Microsoft Cloud App Security (MCAS). L’intégration de MCAS vous permet de configurer une application locale pour la [supervision en temps réel](application-proxy-integrate-with-microsoft-cloud-application-security.md) en tirant parti de l’accès conditionnel pour superviser et contrôler les sessions en temps réel en fonction de stratégies d’accès conditionnel.
* **Arrêt du trafic**. Tout le trafic vers l’application back-end est arrêté au niveau du service Proxy d’application dans le cloud tandis que la session est rétablie avec le serveur back-end. Cette stratégie de connexion signifie que vos serveurs back-end ne sont pas exposés au trafic HTTP direct. Ils sont mieux protégés contre les attaques par déni de service ciblées, car votre pare-feu n’est pas attaqué.
* **Tous les accès sont sortants**. Les connecteurs de proxy d’application utilisent seulement des connexions sortantes vers le service de proxy d’application dans le cloud sur les ports 80 et 443. Sans aucune connexion entrante, vous n’avez pas besoin d’ouvrir des ports de pare-feu pour les connexions ou les composants de la zone DMZ. Toutes les connexions sont sortantes et sur un canal sécurisé.
* **Intelligence basée sur l’analytique de sécurité et le machine learning (ML)** . Parce qu’il fait partie d’Azure Active Directory, le proxy d’application peut tirer parti d’[Azure AD Identity Protection](../identity-protection/overview-identity-protection.md) (nécessite des [licences Premium P2](https://azure.microsoft.com/pricing/details/active-directory/)). Azure AD Identity Protection combine l’intelligence de sécurité du machine learning avec des flux de données de [Digital Crimes Unit](https://news.microsoft.com/stories/cybercrime/index.html) de Microsoft et [Microsoft Security Response Center](https://www.microsoft.com/msrc) pour identifier de façon proactive les comptes compromis. Identity Protection offre une protection en temps réel pour les connexions à haut risque. Il prend en considération des facteurs comme les accès à partir d’appareils infectés, de réseaux anonymes, ou à partir d’emplacements atypiques et peu probables pour augmenter le profil de risque d’une session. Ce profil de risque est utilisé pour la protection en temps réel. Nombre de ces rapports et événements sont déjà disponibles via une API pour l’intégration avec vos systèmes SIEM.

* **Accès à distance en tant que service**. Vous n’avez pas besoin de vous occuper de la maintenance et des mises à jour correctives de vos serveurs locaux pour activer l’accès à distance. Le proxy d’application est un service Internet contrôlé par Microsoft et, de ce fait, vous obtenez toujours les derniers correctifs de sécurité et mises à niveau. Les logiciels sans correctifs sont toujours responsables d’un grand nombre d’attaques. Selon le Department of Homeland Security, [85 % des attaques ciblées sont inévitables](https://www.us-cert.gov/ncas/alerts/TA15-119A). Avec ce modèle de service, vous n’avez plus besoin de gérer vos serveurs de périphérie et de vous dépêcher d’appliquer les correctifs en fonction des besoins.

* **Intégration d’Intune**. Avec Intune, le trafic d’entreprise est routé séparément du trafic personnel. Le proxy d’application permet de vérifier que le trafic d’entreprise est authentifié. Les fonctionnalités du [proxy d’application et d’Intune Managed Browser](/intune/app-configuration-managed-browser#how-to-configure-application-proxy-settings-for-protected-browsers) peuvent aussi être utilisées ensemble pour permettre aux utilisateurs distants d’accéder de manière sécurisée aux sites web internes sur des appareils iOS et Android.

### <a name="roadmap-to-the-cloud"></a>Feuille de route vers le cloud

Un autre avantage majeur de l’implémentation du proxy d’application est d’étendre Azure AD à votre environnement local. En fait, l’implémentation du proxy d’application est une étape essentielle de la migration de votre organisation et vos applications vers le cloud. En migrant vers le cloud et en abandonnant l’authentification locale, vous réduisez votre empreinte locale et utilisez les fonctionnalités de gestion d’identités d’Azure AD comme plan de contrôle. Avec peu ou pas de mise à jour pour les applications existantes, vous avez accès à des fonctionnalités cloud comme l’authentification unique, l’authentification multifacteur et la gestion centralisée. L’installation des composants nécessaires pour le proxy d’application est un processus simple pour établir un framework d’accès à distance. En migrant vers le cloud, vous avez accès aux dernières mises à jour et fonctionnalités d’Azure AD, notamment la haute disponibilité et la reprise d’activité.

Pour en savoir plus sur la migration de vos applications vers Azure AD, consultez le livre blanc [Migration de vos applications vers Azure Active Directory](https://aka.ms/migrateapps/whitepaper).

## <a name="architecture"></a>Architecture

Le diagramme suivant montre globalement comment les services d’authentification Azure AD et le proxy d’application fonctionnent ensemble pour fournir une authentification unique dans les applications locales pour les utilisateurs finaux.

![Flux d’authentification du proxy d’application Azure AD](media/what-is-application-proxy/azure-ad-application-proxy-authentication-flow.png)

1. Une fois que l’utilisateur a accédé à l’application par le biais d’un point de terminaison, il est redirigé vers la page de connexion Azure AD. Si vous avez configuré des stratégies d’accès conditionnel, des conditions spécifiques sont évaluées à ce stade pour vérifier que vous respectez les exigences de sécurité de votre organisation.
2. Après la connexion, Azure AD envoie un jeton à l’appareil client de l’utilisateur.
3. Le client envoie le jeton au service Proxy d’application qui récupère le nom d’utilisateur principal (UPN) et le nom de sécurité principal (SPN) du jeton.
4. Le proxy d’application transfère la demande, qui est utilisée par le [connecteur](application-proxy-connectors.md) de proxy d’application.
5. Le connecteur effectue l’authentification supplémentaire nécessaire, le cas échéant, pour le compte de l’utilisateur (*facultatif selon la méthode d’authentification*), demande le point de terminaison interne du serveur d’applications et envoie la demande à l’application locale.
6. La réponse du serveur d’applications est envoyée par le biais du connecteur et du service de proxy d’application.
7. La réponse est envoyée à l’utilisateur par le service de proxy d’application.

|**Composant**|**Description**|
|:-|:-|
|Point de terminaison|Le point de terminaison est une URL ou un [portail d’utilisateurs finaux](end-user-experiences.md). Les utilisateurs peuvent accéder à des applications en dehors de votre réseau au moyen d’une URL externe. Les utilisateurs au sein de votre réseau peuvent accéder à l’application par le biais d’une URL ou d’un portail d’utilisateurs finaux. Lorsque les utilisateurs passent par l’un de ces points de terminaison, ils s’authentifient dans Azure AD, puis sont acheminés via le connecteur vers l’application locale.|
|Azure AD|Azure AD effectue l’authentification en utilisant l’annuaire des locataires stocké dans le cloud.|
|Service Proxy d’application|Ce service Proxy d’application s’exécute dans le cloud dans le cadre d’Azure AD. Il transmet le jeton d’authentification de l’utilisateur pour le connecteur Proxy d’application. Proxy d'application transmet tous les en-têtes accessibles de la requête et définit les en-têtes à l’adresse IP du client conformément à son protocole. Si la demande entrante envoyée au proxy a déjà cet en-tête, l’adresse IP du client est ajoutée à la fin de la liste délimitée par des virgules qui est la valeur de l’en-tête.|
|Connecteur de proxy d’application|Le connecteur est un agent léger qui s’exécute sur un serveur Windows au sein de votre réseau. Le connecteur gère la communication entre le service Application Proxy dans le cloud et l’application locale. Le connecteur n’utilise que des connexions sortantes ; vous n’êtes donc pas obligé d’ouvrir des ports entrants ou de placer quoi que ce soit dans la zone démilitarisée. Les connecteurs sont sans état et extraient les informations dont ils ont besoin sur le cloud. Pour savoir comment les connecteurs équilibrent la charge ou s’authentifient, voir [Présentation des connecteurs de proxy d’application Azure AD](application-proxy-connectors.md).|
|Active Directory (AD)|Active Directory s'exécute localement pour effectuer l’authentification des comptes de domaine. Lorsque l’authentification unique est configurée, le connecteur communique avec AD pour effectuer toute authentification supplémentaire requise.|
|Application locale|Enfin, l’utilisateur est en mesure d’accéder à une application locale.|

Le proxy d’application Azure AD comprend le service de proxy d’application basé sur le cloud et un connecteur local. Le connecteur écoute les demandes du service de proxy d’application et gère les connexions aux applications internes. Notez bien que toutes les communications se font sur TLS et vont toujours du connecteur vers le service de proxy d’application. Autrement dit, les communications sont uniquement sortantes. Le connecteur utilise un certificat client pour l’authentification sur le service de proxy d’application de presque tous les appels. La seule exception à la sécurité de connexion est l’étape initiale d’installation, au cours de laquelle le certificat client est établi. Consultez la section [Sous le capot](application-proxy-security.md#under-the-hood) du proxy d’application pour plus d’informations.

### <a name="application-proxy-connectors"></a>Connecteurs de proxy d’application

Les [connecteurs de proxy d’application](application-proxy-connectors.md) sont des agents légers déployés localement qui facilitent la connexion sortante vers le service de proxy d’application dans le cloud. Les connecteurs doivent être installés sur un serveur Windows ayant accès à l’application back-end. Les utilisateurs se connectent au service cloud de proxy d’application qui route leur trafic vers les applications via les connecteurs, comme illustré ci-dessous.

![Connexions réseau du proxy d’application Azure AD](media/what-is-application-proxy/azure-ad-application-proxy-network-connections.png)

La configuration et l’inscription entre un connecteur et le service de proxy d’application s’effectuent de la manière suivante :

1. L’administrateur informatique ouvre les ports 80 et 443 pour le trafic sortant et autorise l’accès à plusieurs URL servant au connecteur, au service de proxy d’application et à Azure AD.
2. L’administrateur se connecte au portail Azure et exécute un fichier pour installer le connecteur sur un serveur Windows local.
3. Le connecteur commence à « écouter » sur le service de proxy d’application.
4. L’administrateur ajoute l’application locale à Azure AD et configure des paramètres comme l’URL dont les utilisateurs ont besoin pour se connecter à leurs applications.

Pour plus d’informations, consultez [Planifier un déploiement du proxy d’application Azure AD](application-proxy-deployment-plan.md).

Nous vous recommandons de toujours déployer plusieurs connecteurs pour des questions de redondance et de mise à l’échelle. Les connecteurs, conjointement avec le service, gèrent toutes les tâches de haute disponibilité et peuvent être ajoutés ou supprimés dynamiquement. Chaque fois qu’une nouvelle demande arrive, elle est routée vers l’un des connecteurs disponibles. Lorsqu’un connecteur est en cours d’exécution, il reste actif car il se connecte au service. Si un connecteur est temporairement indisponible, il ne répond pas à ce trafic. Les connecteurs inutilisés sont marqués comme inactifs et supprimés au bout de 10 jours d’inactivité.

Les connecteurs interrogent également le serveur pour déterminer si une version plus récente du connecteur est disponible. Vous avez la possibilité de mettre à jour manuellement les connecteurs, mais ils sont mis à jour automatiquement tant que le service de mise à jour du connecteur de proxy d’application est en cours d’exécution. Pour les abonnés avec plusieurs connecteurs, les mises à jour automatiques ciblent un seul connecteur à la fois dans chaque groupe afin d’éviter les temps d’arrêt dans votre environnement.

**Remarque** : Vous pouvez superviser la [page d’historique des versions](application-proxy-release-version-history.md) du proxy d’application pour être averti quand des mises à jour sont publiées en vous abonnant à son flux RSS.

Chaque connecteur de proxy d’application est attribué à un [groupe de connecteurs](application-proxy-connector-groups.md). Les connecteurs d’un même groupe de connecteurs agissent comme une seule unité pour assurer la haute disponibilité et l’équilibrage de charge. Vous pouvez créer des groupes, leur attribuer des connecteurs dans le portail Azure, puis attribuer des connecteurs spécifiques pour servir des applications spécifiques. Nous vous recommandons d’avoir au moins deux connecteurs dans chaque groupe de connecteurs pour assurer la haute disponibilité.

Les groupes de connecteurs sont utiles pour prendre en charge les scénarios suivants :

* Publication d’applications géographiques
* Segmentation/isolation des applications
* Publication d’applications web s’exécutant dans le cloud ou localement

Pour plus d’informations sur l’endroit où installer vos connecteurs et l’optimisation de votre réseau, consultez [Considérations sur la topologie du réseau lors de l’utilisation du proxy d’application Azure Active Directory](application-proxy-network-topology.md).

## <a name="other-use-cases"></a>Autres cas d’usage

À ce stade, nous nous sommes concentrés sur l’utilisation du proxy d’application pour publier des applications locales en externe tout en activant l’authentification unique pour toutes vos applications cloud et locales. Toutefois, il existe d’autres cas d’usage pour le proxy d’application. Ils comprennent :

* **Publier des API REST de manière sécurisée**. Quand vous avez une logique métier ou des API s’exécutant localement ou hébergées sur des machines virtuelles dans le cloud, le proxy d’application fournit un point de terminaison public pour accéder à l’API. L’accès du point de terminaison d’API vous permet de contrôler l’authentification et l’autorisation sans ports entrants. Il fournit une sécurité supplémentaire grâce à des fonctionnalités Azure AD Premium, comme l’authentification multifacteur et l’accès conditionnel basé sur l’appareil, pour tous les appareils de bureau, iOS, MAC et Android qui utilisent Intune. Pour plus d’informations, consultez [Comment permettre aux applications clientes natives d’interagir avec des applications de proxy](application-proxy-configure-native-client-application.md) et [Protéger une API à l’aide d’OAuth 2.0 avec Azure Active Directory et la gestion des API](../../api-management/api-management-howto-protect-backend-with-aad.md).
* **Services Bureau à distance** **(RDS)** . Les déploiements de services Bureau à distance standard nécessitent des connexions entrantes ouvertes. Toutefois, le [déploiement de services Bureau à distance avec le proxy d’application](application-proxy-integrate-with-remote-desktop-services.md) a une connexion sortante permanente à partir du serveur qui exécute le service du connecteur. De cette façon, vous pouvez proposer plus d’applications aux utilisateurs finaux en publiant des applications locales via les services Bureau à distance. Vous pouvez aussi réduire la surface d’attaque du déploiement avec un ensemble limité de contrôles (vérification en deux étapes et accès conditionnel) sur les services Bureau à distance.
* **Publier des applications qui se connectent avec WebSocket**. La prise en charge de [Qlik Sense](application-proxy-qlik.md) est en préversion publique et sera étendue à d’autres applications par la suite.
* **Autorisez les applications clientes natives à interagir avec des applications de proxy**. Vous pouvez utiliser le proxy d’application Azure AD pour publier des applications web, mais aussi pour publier des [applications clientes natives](application-proxy-configure-native-client-application.md) configurées avec la bibliothèque d’authentification Azure AD (ADAL). Les applications clientes natives sont différentes des applications web parce qu’elles sont installées sur un appareil alors que les applications web sont accessibles via un navigateur.

## <a name="conclusion"></a>Conclusion

Notre manière de travailler et les outils que nous utilisons évoluent rapidement. Comme de plus en plus d’employés apportent leurs propres appareils dans l’entreprise et que l’utilisation des applications SaaS (Software as a service) se généralise, la façon dont les entreprises gèrent et sécurisent leurs données doit aussi évoluer. Les entreprises ne fonctionnent plus seulement dans les limites de leurs murs, protégées par des fossés. Les données à présent voyagent bien plus qu’avant entre les environnements locaux et cloud. Cette évolution a permis d’augmenter la productivité des utilisateurs et leur capacité à collaborer, mais elle complique aussi beaucoup la protection des données sensibles.

Si vous utilisez actuellement Azure AD pour gérer les utilisateurs dans un scénario de coexistence hybride ou que vous voulez commencer votre migration vers le cloud, l’implémentation du proxy d’application Azure AD peut vous aider à réduire la taille de votre empreinte locale en fournissant l’accès à distance en tant que service.

Les organisations doivent commencer à utiliser le proxy d’application dès aujourd'hui pour bénéficier des avantages suivants :

* Publier des applications locales en externe sans avoir à gérer un VPN traditionnel ou d’autres solutions de publication web locales et approche de zone DMZ
* Authentification unique sur toutes les applications, qu’il s’agisse d’application Microsoft 365 ou autres applications SaaS ou d’applications locales
* Sécurité à l’échelle du cloud où Azure AD s’appuie sur la télémétrie Microsoft 365 pour empêcher les accès non autorisés
* Intégration d’Intune pour assurer l’authentification du trafic d’entreprise
* Centralisation de la gestion des comptes d’utilisateur
* Mises à jour automatiques pour assurer l’installation des derniers correctifs de sécurité
* De nouvelles fonctionnalités dès leur publication, dont, plus récemment, la prise en charge de l’authentification unique SAML et la gestion plus granulaire des cookies d’application

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur la planification, l’exploitation et la gestion du proxy d’application Azure AD, consultez [Planifier un déploiement du proxy d’application Azure AD](application-proxy-deployment-plan.md).
* Pour planifier une démonstration en direct ou obtenir un essai gratuit de 90 jours pour l’évaluation, consultez [Bien démarrer avec Enterprise Mobility + Security](https://www.microsoft.com/cloud-platform/enterprise-mobility-security-trial).
