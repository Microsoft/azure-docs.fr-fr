---
title: Effectuer une migration à partir d’Azure Access Control Service | Microsoft Docs
description: Découvrez les options permettant de déplacer des applications et des services à partir d’Azure Access Control Service (ACS).
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.custom: aaddev
ms.topic: how-to
ms.workload: identity
ms.date: 10/03/2018
ms.author: ryanwi
ms.reviewer: jlu, annaba, hirsin
ROBOTS: NOINDEX
ms.openlocfilehash: eda648a4d00a0ab4a51c66510060ce16421972ff
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020008"
---
# <a name="how-to-migrate-from-the-azure-access-control-service"></a>Procédure : Effectuer une migration à partir d’Azure Access Control Service

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Microsoft Azure Access Control Service (ACS), un service d’Azure Active Directory (Azure AD), sera mis hors-service le 7 novembre 2018. Les applications et services qui utilisent actuellement Access Control Service doivent être entièrement migrés vers un autre mécanisme d’authentification d’ici-là. Cet article fournit des recommandations aux clients actuels qui prévoient de ne plus utiliser Access Control Service. Si vous n’utilisez pas Access Control Service, aucune action n’est requise de votre part.

## <a name="overview"></a>Vue d’ensemble

Access Control Service est un service d’authentification de cloud qui permet d’authentifier les utilisateurs et de les autoriser à accéder à vos applications et services web. Il permet également de sortir de votre code, de nombreuses fonctionnalités d’authentification et d’autorisation. Access Control Service est principalement utilisé par les développeurs et architectes de clients Microsoft .NET, d’applications web ASP.NET et des services web WCF (Windows Communication Foundation).

Access Control Service s’utilise dans trois cas de figure principaux :

- Authentification auprès de certains services cloud Microsoft, comme Azure Service Bus et Dynamics CRM. Les applications clientes obtiennent des jetons auprès d’Access Control Service, pour s’authentifier auprès de ces services et effectuer diverses actions.
- Ajout d’authentification dans les applications web personnalisées et prédéfinies (comme SharePoint). Grâce à l’authentification passive d’Access Control Service, les applications web prennent en charge l’authentification avec un compte Microsoft (anciennement Live ID) et avec des comptes Google, Facebook, Yahoo, Azure AD et AD FS (Active Directory Federation Service).
- Sécurisation des services web personnalisés avec des jetons émis par Access Control Service. Grâce à l’authentification active, les services web veillent à n’autoriser l’accès qu’aux clients connus qui se sont authentifiés auprès d’Access Control Service.

Ces utilisations et leurs stratégies de migration recommandées sont décrites dans les sections suivantes.

> [!WARNING]
> Dans la plupart des cas, des modifications importantes du code sont nécessaires pour migrer les applications et services existants vers les nouvelles technologies. Nous vous recommandons de commencer à planifier et à exécuter votre migration dès maintenant, pour éviter tout risque d’arrêt ou d’interruption.

Les composants d’Access Control Service sont les suivants :

- Un service STS (Secure Token Service), qui reçoit les demandes d’authentification et émet des jetons de sécurité en retour.
- Le portail Azure Classic, qui permet de créer, supprimer, activer et désactiver des espaces de noms d’Access Control Service.
- Un portail de gestion d’Access Control Service, qui permet de personnaliser et configurer les espaces de noms d’Access Control Service.
- Un service de gestion, qui peut servir à automatiser les fonctions des portails.
- Un moteur de règles de transformation de jetons, qui permet de définir une logique complexe pour manipuler le format de sortie des jetons émis par Access Control Service.

Pour utiliser ces composants, vous devez créer un ou plusieurs espaces de noms Access Control Service. Un *espace de noms* est une instance d’Access Control Service avec laquelle communiquent vos applications et services. Un espace de noms est isolé de tous les autres clients d’Access Control Service. Les autres clients d’Access Control Service utilisent leurs propres espaces de noms. Un espace de noms dans Access Control Service possède une URL dédiée semblable à celle-ci :

```HTTP
https://<mynamespace>.accesscontrol.windows.net
```

Toutes les communications avec le service STS et les opérations de gestion s’effectuent au niveau de cette URL. Vous utilisez différents chemins d’accès pour différents usages. Pour déterminer si vos applications ou services utilisent Access Control Service, surveillez le trafic vers https://&lt;espace_de_noms&gt;.accesscontrol.windows.net. Tout trafic vers cette URL est géré par Access Control Service. Il doit être supprimé. 

Le trafic vers l’URL `https://accounts.accesscontrol.windows.net` fait figure d’exception. Déjà géré par un autre service, il **n’est pas** affecté par la mise hors service d’Access Control. 

Pour plus d’informations sur Access Control Service, consultez la page [Access Control Service 2.0 (archivée)](/previous-versions/azure/azure-services/hh147631(v=azure.100)).

## <a name="find-out-which-of-your-apps-will-be-impacted"></a>Découvrir les applications qui vont être affectées

Exécutez la procédure ci-après pour connaître celles de vos applications qui seront affectées par la mise hors service d’ACS.

### <a name="download-and-install-acs-powershell"></a>Télécharger et installer ACS PowerShell

1. Accédez à PowerShell Gallery et téléchargez [Acs.Namespaces](https://www.powershellgallery.com/packages/Acs.Namespaces/1.0.2).
2. Installez le module en exécutant la commande suivante :

    ```powershell
    Install-Module -Name Acs.Namespaces
    ```

3. Obtenez la liste de toutes les commandes possibles en exécutant la commande suivante :

    ```powershell
    Get-Command -Module Acs.Namespaces
    ```

    Pour obtenir de l’aide sur une commande spécifique, exécutez la commande suivante :

    ```
     Get-Help [Command-Name] -Full
    ```
    
    où `[Command-Name]` est le nom de la commande ACS.

### <a name="list-your-acs-namespaces"></a>Obtenir la liste de vos espaces de noms ACS

1. Connectez-vous à ACS à l’aide de l’applet de commande **Connect-AcsAccount**.
  
    L’exécution de commandes peut nécessiter que vous exécutiez `Set-ExecutionPolicy -ExecutionPolicy Bypass` au préalable et que vous soyez l’administrateur de ces abonnements.

2. Obtenez la liste de vos abonnements Azure disponibles à l’aide de l’applet de commande **Get-AcsSubscription**.
3. Obtenez la liste de vos espaces de noms ACS en utilisant l’applet de commande **Get-AcsNamespace**.

### <a name="check-which-applications-will-be-impacted"></a>Vérifier les applications qui vont être affectées

1. Utilisez l’espace de noms obtenu à l’étape précédente pour accéder à l’adresse `https://<namespace>.accesscontrol.windows.net`.

    Par exemple, si l’un des espaces de noms est contoso-test, accédez à l’adresse `https://contoso-test.accesscontrol.windows.net`.

2. Sous **Relations d’approbation**, sélectionnez **Applications par partie de confiance** pour afficher la liste des applications qui vont être affectées par la mise hors service d’ACS.
3. Répétez les étapes 1 et 2 pour tous les autres espaces de noms ACS dont vous disposez.

## <a name="retirement-schedule"></a>Planification de la mise hors service

Depuis novembre 2017, tous les composants d’Access Control Service sont entièrement pris en charge et opérationnels. Seule restriction, vous [ne peut pas créer d’autres espaces de noms Access Control Service via le portail Azure Classic](https://azure.microsoft.com/blog/acs-access-control-service-namespace-creation-restriction/).

Voici le planning de la mise hors service des composants d’Access Control Service :

- **Novembre 2017** :  l’expérience d’administration d’Azure AD dans le Portail Azure Classic a été [mise hors service](https://blogs.technet.microsoft.com/enterprisemobility/2017/09/18/marching-into-the-future-of-the-azure-ad-admin-experience-retiring-the-azure-classic-portal/). La gestion des espaces de noms pour Access Control sera alors disponible à une nouvelle URL dédiée : `https://manage.windowsazure.com?restoreClassic=true`. Utilisez cette URL pour afficher vos espaces de noms, activer et désactiver des espaces de noms et pour en supprimer si vous le souhaitez.
- **2 avril 2018** : le Portail Azure Classic est entièrement mis hors service, ce qui signifie que la gestion des espaces de noms d’Access Control n’est plus disponible à aucune URL. Dès lors, vous ne pourrez pas désactiver, activer, supprimer ou énumérer vos espaces de noms Access Control Service. Toutefois, le portail de gestion d’Access Control sera entièrement fonctionnel et accessible à l’adresse `https://\<namespace\>.accesscontrol.windows.net`. Tous les autres composants d’Access Control Service continueront à fonctionner normalement.
- **7 novembre 2018** : tous les composants d’Access Control sont définitivement mis hors service. Cela inclut le portail de gestion Access Control Service, le service de gestion, le service STS et le moteur de règles de transformation des jetons. À ce moment-là, toutes les demandes envoyées à Access Control Service (sur \<namespace\>.accesscontrol.windows.net) échoueront. Vous devrez avoir migré l’ensemble des applications et services vers d’autres technologies bien avant cette date.

> [!NOTE]
> Une stratégie désactive les espaces de noms qui n’ont pas demandé de jeton pendant une période donnée. Depuis début septembre 2018, cette période est actuellement de 14 jours d’inactivité, mais sera réduite à 7 jours d’inactivité dans les semaines à venir. Si vous disposez d’espaces de noms Access Control actuellement désactivés, vous pouvez [télécharger et installer ACS PowerShell](#download-and-install-acs-powershell) pour réactiver les espaces de noms.

## <a name="migration-strategies"></a>Stratégies de migration

Les sections suivantes fournissent des recommandations générales pour migrer Access Control Service vers d’autres technologies Microsoft.

### <a name="clients-of-microsoft-cloud-services"></a>Clients de services cloud Microsoft

Chaque service cloud Microsoft qui accepte les jetons émis par Access Control Service prend maintenant en charge au moins une autre forme d’authentification. Le mécanisme d’authentification approprié varie selon chaque service. Nous vous recommandons de consulter la documentation de chaque service pour obtenir des conseils officiels. Pour plus de commodité, chaque ensemble de documentation est mentionné ici :

| Service | Assistance |
| ------- | -------- |
| Azure Service Bus | [Migrer du service Access Control Service d’Azure Active Directory vers le service de signature d’accès partagé](../../service-bus-messaging/service-bus-migrate-acs-sas.md) |
| Azure Service Bus Relay | [Migrer du service Access Control Service d’Azure Active Directory vers le service de signature d’accès partagé](../../azure-relay/relay-migrate-acs-sas.md) |
| Azure Managed Cache | [Migrer vers le Cache Azure pour Redis](../../azure-cache-for-redis/cache-faq.md) |
| Azure DataMarket | [Migrer vers les API Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) |
| BizTalk Services | [Migrer vers la fonctionnalité Logic Apps d’Azure App Service](https://azure.microsoft.com/services/cognitive-services/) |
| Azure Media Services | [Migrer vers l’authentification Azure AD](https://azure.microsoft.com/blog/azure-media-service-aad-auth-and-acs-deprecation/) |
| Sauvegarde Azure | [Questions sur le service de sauvegarde Azure](../../backup/backup-azure-file-folder-backup-faq.md) |

<!-- Dynamics CRM: Migrate to new SDK, Dynamics team handling privately -->
<!-- Azure RemoteApp deprecated in favor of Citrix: https://www.zdnet.com/article/microsoft-to-drop-azure-remoteapp-in-favor-of-citrix-remoting-technologies/ -->
<!-- Exchange push notifications are moving, customers don't need to move -->
<!-- Retail federation services are moving, customers don't need to move -->
<!-- Azure StorSimple: TODO -->
<!-- Azure SiteRecovery: TODO -->

### <a name="sharepoint-customers"></a>Clients SharePoint

Les clients SharePoint 2013, 2016 et SharePoint Online utilisent depuis longtemps ACS à des fins d’authentification dans des scénarios cloud, locaux et hybrides. Certaines fonctionnalités et certains cas d’usage de SharePoint seront affectés par la mise hors service d’ACS, d’autres non. Le tableau ci-dessous résume les conseils de migration pour les fonctionnalités de SharePoint les plus utilisées parmi celles qui exploitent ACS :

| Fonctionnalité | Assistance |
| ------- | -------- |
| Authentification des utilisateurs à partir d’Azure AD | Avant, Azure AD ne prenait pas en charge les jetons SAML 1.1 requis par SharePoint pour l’authentification, et ACS était utilisé comme un intermédiaire assurant la compatibilité de SharePoint avec les formats de jetons Azure AD. Désormais, vous pouvez [connecter SharePoint directement à Azure AD à l’aide de l’application SharePoint sur site Galerie d’applications Azure AD](../saas-apps/sharepoint-on-premises-tutorial.md). |
| [Authentification des applications et authentification de serveur à serveur dans SharePoint en local](/SharePoint/security-for-sharepoint-server/authentication-overview) | Non affecté par la mise hors service d’ACS ; aucune modification n’est nécessaire. | 
| [Autorisation peu fiable pour les compléments SharePoint (fournisseur et SharePoint hébergés)](/sharepoint/dev/sp-add-ins/three-authorization-systems-for-sharepoint-add-ins) | Non affecté par la mise hors service d’ACS ; aucune modification n’est nécessaire. |
| [Recherche hybride cloud SharePoint](/archive/blogs/spses/cloud-hybrid-search-service-application) | Non affecté par la mise hors service d’ACS ; aucune modification n’est nécessaire. |

### <a name="web-applications-that-use-passive-authentication"></a>Applications web utilisant l’authentification passive

Pour les applications web qui utilisent Access Control Service pour authentifier les utilisateurs, Access Control Service fournit les fonctionnalités suivantes aux architectes et développeurs d’applications web :

- Intégration étroite avec Windows Identity Foundation (WIF).
- Fédération avec les comptes Google, Facebook, Yahoo, Azure Active Directory et AD FS, ainsi qu’avec les comptes Microsoft.
- Prise en charge des protocoles d’authentification suivants : OAuth 2.0 Draft 13, WS-Trust et Web Services Federation (WS-Federation).
- Prise en charge des formats de jeton suivants : JSON Web Token (JWT), SAML 1.1, SAML 2.0 et Simple Web Token (SWT).
- Une expérience de découverte de domaine d’accueil, intégrée à WIF, permettant aux utilisateurs de choisir le type de compte à utiliser pour se connecter. Hébergée par l’application web, cette expérience est entièrement personnalisable.
- Transformation des jetons permettant une personnalisation enrichie des revendications reçues par l’application web en provenance d’Access Control Service, notamment :
    - Transfert de revendications provenant des fournisseurs d’identité.
    - Ajout de revendications personnalisées supplémentaires.
    - Logique if-then simple pour émettre des revendications sous certaines conditions.

Malheureusement, il n’existe aucun service qui fournisse toutes ces fonctionnalités. Vous devez évaluer les fonctionnalités d’Access Control Service dont vous avez besoin, puis choisir entre utiliser [Azure Active Directory](https://azure.microsoft.com/develop/identity/signin/), [Azure Active Directory B2C](https://azure.microsoft.com/services/active-directory-b2c/) (Azure AD B2C) ou un autre service d’authentification cloud.

#### <a name="migrate-to-azure-active-directory"></a>Migration vers Azure Active Directory

L’une des solutions envisageables consiste à intégrer vos applications et services directement dans Azure Active Directory. Azure Active Directory est le fournisseur d’identité basé sur le cloud pour les comptes professionnels ou scolaires Microsoft. Azure Active Directory est le fournisseur d’identité pour Microsoft 365 et Azure, entre autres. Il fournit des fonctions d’authentification fédérée semblables à Access Control Service, mais ne prend pas en charge toutes les fonctionnalités d’Access Control Service. 

Le principal exemple concerne la fédération avec des fournisseurs d’identité sociale, tels que Facebook, Google et Yahoo. Si vos utilisateurs se connectent avec ces types d’informations d’identification, Azure Active Directory n’est pas pour vous. 

De plus, Azure Active Directory ne prend pas en charge les mêmes protocoles d’authentification qu’Access Control Service. Par exemple, bien qu’Access Control Service et Azure Active Directory prennent en charge OAuth, il existe des différences subtiles entre chaque implémentation. Plusieurs implémentations vous obligent à modifier le code dans le cadre d’une migration.

Cependant, Azure Active Directory offre plusieurs avantages potentiels aux clients d’Access Control Service. Il prend nativement en charge les comptes professionnels et scolaires Microsoft hébergés dans le cloud, qui sont couramment utilisés par les clients d’Access Control Service. 

Un abonné Azure Active Directory peut également être fédéré dans une ou plusieurs instances Active Directory locale via AD FS. Ainsi, votre application peut authentifier les utilisateurs basés sur le cloud et ceux qui sont hébergés sur site. Il prend également en charge le protocole WS-Federation, ce qui permet d’intégrer relativement facilement une application web à l’aide de WIF.

Le tableau suivant compare les fonctionnalités d’Access Control Service qui sont pertinentes pour les applications web à celles qui sont disponibles dans Azure Active Directory. 

De manière générale, *Azure Active Directory est probablement le meilleur choix pour votre migration si vous permettez aux utilisateurs de se connecter uniquement à l’aide de leur compte professionnel ou scolaire Microsoft*.

| Fonctionnalité | Prise en charge d’Access Control Service | Prise en charge d’Azure Active Directory |
| ---------- | ----------- | ---------------- |
| **Types de comptes** | | |
| Comptes professionnels et scolaires Microsoft | Prise en charge | Prise en charge |
| Comptes Windows Server Active Directory et AD FS |- Pris en charge par le biais de la fédération avec un abonné Azure Active Directory <br />- Pris en charge par le biais de la fédération directe avec AD FS | Pris en charge uniquement par le biais de la fédération avec un locataire Azure AD | 
| Comptes d’autres systèmes de gestion d’identité d’entreprise |- Possible par le biais de la fédération avec un abonné Azure Active Directory <br />- Pris en charge par le biais de la fédération directe | Possible par le biais de la fédération avec un locataire Azure AD |
| Comptes Microsoft destinés à une utilisation personnelle | Prise en charge | Pris en charge par le biais du protocole OAuth v2.0 d’Azure Active Directory, mais pas sur d’autres protocoles | 
| Comptes Facebook, Google, Yahoo | Prise en charge | Aucune prise en charge |
| **Protocoles et compatibilité des SDK** | | |
| WIF | Prise en charge | Pris en charge, mais les instructions disponibles sont limitées |
| Un certificat de fournisseur d'identité WS-Federation | Prise en charge | Prise en charge |
| OAuth 2.0 | Pris en charge pour Draft 13 | Pris en charge pour RFC 6749, la spécification la plus récente |
| WS-Trust | Prise en charge | Non pris en charge |
| **Formats de jeton** | | |
| JWT | Pris en charge dans la version bêta | Prise en charge |
| SAML 1.1 | Prise en charge | PRÉVERSION |
| SAML 2.0 | Prise en charge | Prise en charge |
| SWT | Prise en charge | Non pris en charge |
| **Personnalisations** | | |
| Interface utilisateur de sélection de compte/découverte de domaine d’accueil personnalisable | Code téléchargeable pouvant être incorporé dans des applications | Non pris en charge |
| Chargement de certificats de signature de jeton personnalisés | Prise en charge | Prise en charge |
| Personnalisation des revendications dans les jetons |- Transfert direct des revendications d’entrée émis par des fournisseurs d’identité<br />- Obtention de jeton d’accès auprès du fournisseur d’identité en tant que revendication<br />- Émission de revendications de sortie basées sur les valeurs des revendications d’entrée<br />- Émission de revendications de sortie avec des valeurs constantes |- Impossibilité de transférer directement des revendications émises par des fournisseurs d’identité fédérés<br />- Impossibilité d’obtenir un jeton d’accès auprès du fournisseur d’identité sous la forme d’une revendication<br />- Impossibilité d’émettre des revendications de sortie basées sur les valeurs des revendications d’entrée<br />- Possibilité d’émettre des revendications de sortie avec des valeurs constantes<br />- Possibilité d’émettre des revendications de sortie basées sur des propriétés d’utilisateurs synchronisées avec Azure Active Directory |
| **Automation** | | |
| Automatisation des tâches de gestion et de configuration | Prise en charge par le service de gestion Access Control Service | Pris en charge à l’aide de l’API Microsoft Graph |

Si vous décidez qu’Azure Active Directory est la meilleure voie de migration pour vos applications et services, sachez qu’il existe deux manières d’intégrer votre application dans Azure Active Directory.

Pour utiliser WS-Federation ou WIF en vue d’une intégration dans Azure Active Directory, nous vous recommandons l’approche décrite dans [Comment configurer l’authentification unique fédérée pour une application non issue de la galerie](../manage-apps/configure-saml-single-sign-on.md). Cet article fait référence à la configuration d’Azure Active Directory pour l’authentification unique basée sur SAML, mais il concerne également la configuration de WS-Federation. Cette approche nécessite une licence Azure Active Directory Premium. Cette approche offre deux avantages :

- Vous bénéficiez de toute la flexibilité offerte par la personnalisation des jetons Azure AD. Vous pouvez personnaliser les revendications émises par Azure Active Directory pour les faire correspondre aux revendications émises par Access Control Service. Cela inclut notamment la revendication d’identificateur de nom ou d’ID utilisateur. Pour continuer de recevoir des identificateurs cohérents pour vos utilisateurs même après avoir changé de technologie, vérifiez que les ID d’utilisateur émis par Azure Active Directory correspondent à ceux émis par Access Control Service.
- Vous pouvez configurer un certificat de signature de jeton propre à votre application, dont vous contrôlez la durée de vie.

> [!NOTE]
> Cette approche nécessite une licence Azure Active Directory Premium. Si vous êtes un client d’Access Control Service et que vous avez besoin d’une licence Premium afin de configurer l’authentification unique pour une application, contactez-nous. Nous serons heureux de vous fournir des licences de développeur.

Une autre solution consiste à suivre [cet exemple de code](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation), qui fournit des instructions légèrement pour configurer WS-Federation. Cet exemple de code n’utilise pas WIF mais plutôt l’intergiciel (middleware) OWIN ASP.NET 4.5. Toutefois, les instructions d’inscription d’application sont valides pour les applications qui utilisent WIF, et elles ne nécessitent pas de licence Azure Active Directory Premium. 

Si vous choisissez cette approche, vous devez comprendre la [substitution de clé de signature dans Azure Active Directory](../develop/active-directory-signing-key-rollover.md). Cette approche utilise la clé de signature globale Azure AD pour émettre des jetons. Par défaut, WIF n’actualise pas automatiquement les clés de signature. Quand Azure AD permute ses clés de signature globale, votre implémentation de WIF doit être prête à accepter les changements. Pour plus d’informations, consultez l’article [Substitution de la clé de signature dans Azure Active Directory](/previous-versions/azure/dn641920(v=azure.100)).

Si vous pouvez effectuer une intégration dans Azure Active Directory par le biais du protocole OpenID Connect ou OAuth, nous vous recommandons de le faire. Vous trouverez une documentation complète et des instructions détaillées expliquant comment intégrer Azure Active Directory dans votre application web, dans notre [Guide du développeur Azure Active Directory](../develop/index.yml).

#### <a name="migrate-to-azure-active-directory-b2c"></a>Migration vers Azure Active Directory B2C

L’autre chemin de migration à prendre en compte est Azure AD B2C. Azure Active Directory B2C est un service d’authentification cloud qui, un peu comme Access Control Service, permet aux développeurs d’externaliser leur logique de gestion d’identité et d’authentification à un service cloud. Ce service payant (avec des niveaux gratuits et Premium) est conçu pour les applications exposées aux consommateurs et pouvant compter plusieurs millions d’utilisateurs actifs.

Comme Access Control Service, l’une des fonctionnalités les plus intéressantes d’Azure Active Directory B2C est sa prise en charge de nombreux types de comptes. Avec Azure Active Directory B2C, vous pouvez connecter des utilisateurs avec leur compte Microsoft ou des comptes Facebook, Google, LinkedIn, GitHub, Yahoo, etc. Azure Active Directory B2C prend également en charge les « comptes locaux », ou les noms d’utilisateurs et mots de passe que les utilisateurs créent spécialement pour votre application. Azure Active Directory B2C fournit également une extensibilité étendue qui vous permet de personnaliser vos flux d’authentification. 

Toutefois, Azure Active Directory B2C ne prend pas en charge l’éventail de protocoles d’authentification et de formats de jeton dont les clients d’Access Control Service peuvent avoir besoin. Vous ne pouvez pas non plus l’utiliser pour obtenir des jetons et des informations supplémentaires sur l’utilisateur auprès du fournisseur d’identité, de Microsoft ou autre.

Le tableau suivant compare les fonctionnalités d’Access Control Service qui sont pertinentes pour les applications web, à celles qui sont disponibles dans Azure Active Directory B2C. De manière générale, *Azure AD B2C est probablement le bon choix pour votre migration si votre application est exposée aux consommateurs ou si elle prend en charge de nombreux types de comptes.*

| Fonctionnalité | Prise en charge d’Access Control Service | Prise en charge d’Azure Active Directory B2C |
| ---------- | ----------- | ---------------- |
| **Types de comptes** | | |
| Comptes professionnels et scolaires Microsoft | Prise en charge | Pris en charge par le biais des stratégies personnalisées  |
| Comptes Windows Server Active Directory et AD FS | Pris en charge par le biais de la fédération directe avec AD FS | Pris en charge par le biais de la fédération SAML à l’aide de stratégies personnalisées |
| Comptes d’autres systèmes de gestion d’identité d’entreprise | Pris en charge par le biais de la fédération directe avec WS-Federation | Pris en charge par le biais de la fédération SAML à l’aide de stratégies personnalisées |
| Comptes Microsoft destinés à une utilisation personnelle | Prise en charge | Prise en charge | 
| Comptes Facebook, Google, Yahoo | Prise en charge | Facebook et Google pris en charge en mode natif, Yahoo pris en charge par le biais de la fédération OpenID Connect à l’aide de stratégies personnalisées |
| **Protocoles et compatibilité des SDK** | | |
| Windows Identity Foundation (WIF) | Prise en charge | Non pris en charge |
| Un certificat de fournisseur d'identité WS-Federation | Prise en charge | Non pris en charge |
| OAuth 2.0 | Pris en charge pour Draft 13 | Pris en charge pour RFC 6749, la spécification la plus récente |
| WS-Trust | Prise en charge | Non pris en charge |
| **Formats de jeton** | | |
| JWT | Pris en charge dans la version bêta | Prise en charge |
| SAML 1.1 | Prise en charge | Non pris en charge |
| SAML 2.0 | Prise en charge | Non pris en charge |
| SWT | Prise en charge | Non pris en charge |
| **Personnalisations** | | |
| Interface utilisateur de sélection de compte/découverte de domaine d’accueil personnalisable | Code téléchargeable pouvant être incorporé dans des applications | Interface utilisateur entièrement personnalisable par le biais de feuilles de style CSS personnalisées |
| Chargement de certificats de signature de jeton personnalisés | Prise en charge | Clés de signature (et non certificats) personnalisées prises en charge par le biais de stratégies personnalisées |
| Personnalisation des revendications dans les jetons |- Transfert direct des revendications d’entrée émis par des fournisseurs d’identité<br />- Obtention de jeton d’accès auprès du fournisseur d’identité en tant que revendication<br />- Émission de revendications de sortie basées sur les valeurs des revendications d’entrée<br />- Émission de revendications de sortie avec des valeurs constantes |- Possibilité de transmettre des revendications à partir des fournisseurs d’identité ; stratégies personnalisées requises pour certaines revendications<br />- Impossibilité d’obtenir un jeton d’accès auprès du fournisseur d’identité sous la forme d’une revendication<br />- Possibilité d’émettre des revendications de sortie basées sur les valeurs des revendications d’entrée par le biais de stratégies personnalisées<br />- Possibilité d’émettre des revendications avec des valeurs constantes par le biais de stratégies personnalisées |
| **Automation** | | |
| Automatisation des tâches de gestion et de configuration | Prise en charge par le service de gestion Access Control Service |- Création d’utilisateurs autorisés à l’aide de l’API Microsoft Graph<br />- Impossibilité de créer des abonnés, des applications ou des stratégies B2C par programmation |

Si vous décidez qu’Azure Active Directory B2C représente la meilleure solution de migration pour vos applications et services, vous devez commencer par les ressources suivantes :

- [Documentation d’Azure Active Directory B2C](../../active-directory-b2c/overview.md)
- [Stratégies personnalisées Azure Active Directory B2C](../../active-directory-b2c/custom-policy-overview.md)
- [Tarification d’Azure Active Directory B2C](https://azure.microsoft.com/pricing/details/active-directory-b2c/)

#### <a name="migrate-to-ping-identity-or-auth0"></a>Migrer vers Ping Identity ou Auth0

Parfois, il se peut qu’Azure Active Directory et Azure Active Directory B2C ne suffisent pas pour remplacer Access Control Service dans vos applications web sans apporter de modifications importantes au code. En voici quelques exemples courants :

- Applications Web qui utilisent WIF ou WS-Federation pour se connecter à l’aide de fournisseurs d’identité sociaux tels que Google ou Facebook.
- Applications Web qui effectuent une fédération directe dans un fournisseur d’identité d’entreprise via le protocole WS-Federation.
- Applications Web qui requièrent le jeton d’accès émis par un fournisseur d’identité social (comme Google ou Facebook) sous la forme d’une revendication dans les jetons émis par Access Control Service.
- Applications Web avec des règles complexes de transformation des jetons, qu’Azure Active Directory ou Azure Active Directory B2C ne peut pas reproduire.
- Applications Web à architecture mutualisée qui utilisent Access Control Service pour gérer la fédération de manière centralisée chez de nombreux fournisseurs d’identité.

Dans ce cas, il peut être préférable de migrer votre application web vers un autre service d’authentification cloud. Nous vous recommandons d’explorer les options suivantes. Chacune des options suivantes offre des fonctionnalité similaires à Access Control Service :

![Cette image affiche le logo Auth0](./media/active-directory-acs-migration/rsz-auth0.png) 

[Auth0](https://auth0.com/acs) est un service d’identité cloud souple qui a formulé des [conseils généraux pour la migration des clients d’Access Control Service](https://auth0.com/acs) et prend en charge presque toutes les fonctionnalités d’Access Control Service.

![Cette image affiche le logo Ping Identity](./media/active-directory-acs-migration/rsz-ping.png)

[Ping Identity](https://www.pingidentity.com) propose deux solutions similaires à Access Control Service. PingOne est un service d’identité cloud qui prend en charge la plupart des fonctionnalités d’Access Control Service, et PingFederate est un produit d’identité locale similaire qui offre davantage de souplesse. Pour plus d’informations sur l’utilisation de ces produits, consultez les [Conseils concernant la mise hors service d’Access Control Service de Ping](https://www.pingidentity.com/en/company/blog/posts/2017/migrating-from-microsoft-acs-to-ping-identity.html).

Notre objectif en utilisant Ping Identity et Auth0 est de vous assurer que tous les clients d’Access Control Service ont un chemin de migration pour leurs applications et services, qui réduise le volume de travail requis pour abandonner Access Control Service.

<!--

## SharePoint 2010, 2013, 2016

TODO: Azure AD only, use Azure AD SAML 1.1 tokens, when we bring it back online.
Other IDPs: use Auth0? https://auth0.com/docs/integrations/sharepoint.

-->

### <a name="web-services-that-use-active-authentication"></a>Services Web qui utilisent l’authentification active

Pour les services web sécurisés avec des jetons émis par Access Control Service, Access Control Service offre les fonctionnalités suivantes :

- Possibilité d’inscrire une ou plusieurs *identités de service* dans votre espace de noms Access Control Service. Les identités de service peuvent servir à demander des jetons.
- Prise en charge des protocoles OAuth WRAP et OAuth 2.0 Draft 13 pour demander des jetons, avec les types d’informations d’identification suivants :
    - Un mot de passe simple créé pour l’identité du service
    - Un jeton SWT signé à l’aide d’une clé symétrique ou d’un certificat X509
    - Un jeton SAML émis par un fournisseur d’identité approuvé (en général, une instance AD FS)
- Prise en charge des formats de jeton suivants : JWT, SAML 1.1, SAML 2.0 et SWT.
- Règles de transformation de jetons simples.

Les identités de service dans Access Control Service servent généralement à implémenter une authentification de serveur à serveur. 

#### <a name="migrate-to-azure-active-directory"></a>Migration vers Azure Active Directory

Notre recommandation pour ce type de flux d’authentification consiste à migrer vers [Azure Active Directory](https://azure.microsoft.com/develop/identity/signin/). Azure Active Directory est le fournisseur d’identité basé sur le cloud pour les comptes professionnels ou scolaires Microsoft. Azure Active Directory est le fournisseur d’identité pour Microsoft 365 et Azure, entre autres. 

Mais vous pouvez aussi l’utiliser pour l’authentification de serveur à serveur, grâce à l’implémentation de l’octroi d’informations d’identification de client OAuth dans Azure Active Directory. Le tableau suivant compare les fonctionnalités d’Access Control Service dans l’authentification de serveur à serveur à celles qui sont disponibles dans Azure Active Directory.

| Fonctionnalité | Prise en charge d’Access Control Service | Prise en charge d’Azure Active Directory |
| ---------- | ----------- | ---------------- |
| Inscription de service web | Créer une partie de confiance dans le portail de gestion Access Control Service | Créer une application Web Azure Active Directory dans le portail Azure |
| Inscription de client | Créer une identité de service dans le portail de gestion d’Access Control Service | Créer une autre application Azure Active Directory dans le portail Azure |
| Protocole utilisé |- Protocole OAuth WRAP<br />- Octroi des informations d’identification de client OAuth 2.0 Draft 13 | Octroi des informations d’identification du client OAuth 2.0 |
| Méthodes d’authentification du client |- Mot de passe simple<br />- Jeton SWT signé<br />- Jeton SAML émis par un fournisseur d’identité fédéré |- Mot de passe simple<br />- Jeton JWT signé |
| Formats de jetons |- JWT<br />- SAML 1.1<br />- SAML 2.0<br />- SWT<br /> | JWT uniquement |
| Transformation de jeton |- Ajout de revendications personnalisées<br />- Logique if-then simple d’émission de revendications | Ajouter des revendications personnalisées | 
| Automatisation des tâches de gestion et de configuration | Prise en charge par le service de gestion Access Control Service | Pris en charge à l’aide de l’API Microsoft Graph |

Pour obtenir des conseils sur l’implémentation des scénarios de serveur à serveur, consultez les ressources suivantes :

- Section « service à service » du [Guide du développeur Azure Active Directory](../develop/index.yml)
- [Exemple de code de démon utilisant des informations d’identification de client avec mots de passe simples](https://github.com/Azure-Samples/active-directory-dotnet-daemon)
- [Exemple de code de démon utilisant des informations d’identification de client avec certificats](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential)

#### <a name="migrate-to-ping-identity-or-auth0"></a>Migrer vers Ping Identity ou Auth0

Parfois, il se peut que les informations d’identification de client Azure Active Directory et l’implémentation d’octroi OAuth ne suffisent pas à remplacer Access Control Service dans votre architecture sans modification importante du code. En voici quelques exemples courants :

- Authentification de serveur à serveur à l’aide de formats de jeton autres que JWT.
- Authentification de serveur à serveur à l’aide d’un jeton d’entrée fourni par un fournisseur d’identité externe.
- Authentification de serveur à serveur avec des règles de transformation de jetons qu’Azure Active Directory ne peut pas reproduire.

Dans ces cas, vous pouvez envisager de migrer votre application web vers un autre service d’authentification cloud. Nous vous recommandons d’explorer les options suivantes. Chacune des options suivantes offre des fonctionnalité similaires à Access Control Service :

![Cette image affiche le logo Auth0](./media/active-directory-acs-migration/rsz-auth0.png)

[Auth0](https://auth0.com/acs) est un service d’identité cloud souple qui a formulé des [conseils généraux pour la migration des clients d’Access Control Service](https://auth0.com/acs) et prend en charge presque toutes les fonctionnalités d’Access Control Service.

![Cette image illustre le logo Ping Identity](./media/active-directory-acs-migration/rsz-ping.png)
[Ping Identity](https://www.pingidentity.com) offre deux solutions similaires à ACS. PingOne est un service d’identité cloud qui prend en charge la plupart des fonctionnalités d’Access Control Service, et PingFederate est un produit d’identité locale similaire qui offre davantage de souplesse. Pour plus d’informations sur l’utilisation de ces produits, consultez les [Conseils concernant la mise hors service d’Access Control Service de Ping](https://www.pingidentity.com/en/company/blog/posts/2017/migrating-from-microsoft-acs-to-ping-identity.html).

Notre objectif en utilisant Ping Identity et Auth0 est de vous assurer que tous les clients d’Access Control Service ont un chemin de migration pour leurs applications et services, qui réduise le volume de travail requis pour abandonner Access Control Service.

#### <a name="passthrough-authentication"></a>Authentification directe

Pour l’authentification directe avec transformation arbitraire de jeton, il n’existe aucune technologie Microsoft équivalente à ACS. Si vos clients en ont besoin, Auth0 peut offrir la solution d’approximation la plus proche.

## <a name="questions-concerns-and-feedback"></a>Questions, problèmes et commentaires

Nous avons conscience que de nombreux clients d’Access Control Service ne trouveront pas un chemin de migration en lisant cet article. Vous aurez certainement besoin d’aide ou de conseils pour déterminer le plan adapté. Si vous souhaitez discuter de vos scénarios de migration et nous poser des questions, laissez un commentaire sur cette page.