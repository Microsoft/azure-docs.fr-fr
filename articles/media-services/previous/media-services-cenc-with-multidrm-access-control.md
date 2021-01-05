---
title: Conception d’un système de protection du contenu avec contrôle d’accès à l’aide d’Azure Media Services | Microsoft Docs
description: En savoir plus sur l’octroi d’une licence pour le kit de portage Smooth Streaming client Microsoft.
services: media-services
documentationcenter: ''
author: willzhan
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: willzhan
ms.reviewer: kilroyh;yanmf;juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: b98b66d8f0350c32e89d62d776ee1288d9271712
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96010909"
---
# <a name="design-of-a-content-protection-system-with-access-control-using-azure-media-services"></a>Conception d’un système de protection du contenu avec contrôle d’accès à l’aide d’Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Vue d’ensemble

Il est assez complexe de concevoir et développer un sous-système de gestion des droits numériques (DRM) pour une solution OTT (Over-The-Top) ou de diffusion en continu en ligne. Les opérateurs/fournisseurs de vidéo en ligne ont l’habitude d’externaliser cette tâche à des fournisseurs de services DRM spécialisés. L’objectif de ce document est de présenter une conception et une implémentation de référence d’un sous-système DRM de bout en bout dans une solution OTT ou de diffusion en continu en ligne.

Ce document s’adresse aux ingénieurs qui travaillent sur des sous-systèmes DRM de solutions OTT ou de solutions de diffusion en continu en ligne/multi-écrans, ou à tout lecteur intéressé par les sous-systèmes de gestion des droits numériques. Les lecteurs doivent connaître au moins une des technologies DRM du marché, telles que PlayReady, Widevine, FairPlay ou Adobe Access.

Dans cette discussion axée sur la DRM, nous allons également évoquer le chiffrement commun (CENC) avec un système multi-DRM. Le marché de la diffusion en continu en ligne et de l’OTT est marqué par une nouvelle tendance qui consiste à utiliser le CENC avec un système multi-DRM natif sur différentes plateformes clientes. Cette tendance témoigne d’un bouleversement par rapport à l’époque où l’on utilisait un seul système DRM et son SDK client pour différentes plateformes clientes. Lorsque vous utilisez CENC avec un système multi-DRM natif, PlayReady et Widevine sont chiffrés conformément à la spécification [Common Encryption (ISO/IEC 23001-7 CENC)](https://www.iso.org/iso/home/store/catalogue_ics/catalogue_detail_ics.htm?csnumber=65271/).

L’utilisation de CENC avec un système multi-DRM offre les avantages suivants :

* réduction des frais de chiffrement dans la mesure où un seul processus de chiffrement est utilisé pour cibler différentes plateformes avec ses systèmes DRM natifs ;
* réduction du coût de gestion des éléments multimédias chiffrés puisqu’une seule copie de ces éléments est nécessaire ;
* élimination des coûts de licence client DRM, car le client DRM natif est généralement proposé gratuitement sur sa plateforme d’origine.

Microsoft joue le rôle de promoteur actif de DASH et de CENC, tout comme d’autres acteurs majeurs du secteur. Azure Media Services assure la prise en charge de DASH et de CENC. Pour accéder aux récentes annonces, consultez les blogs suivants :

*  [Annonce des services de distribution de licence Google Widevine dans Azure Media Services](https://azure.microsoft.com/blog/announcing-general-availability-of-google-widevine-license-services/)
* [Azure Media Services adds Google Widevine packaging for delivering a multi-DRM stream](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/) (Azure Media Services ajoute l’empaquetage Google Widevine pour la diffusion de flux multi-DRM)  

### <a name="goals-of-the-article"></a>Objectifs de l’article

Les objectifs de cet article sont les suivants :

* fourniture d’une conception de référence d’un sous-système de gestion des droits numériques qui utilise CENC avec un système multi-DRM ;
* fourniture d’une implémentation de référence sur une plateforme Azure/Media Services ;
* commentaires de certaines rubriques relatives à la conception et l’implémentation.

Dans cet article, la désignation « système multi-DRM » couvre les produits suivants :

* Microsoft PlayReady
* Google Widevine
* Apple FairPlay 

Le tableau suivant présente succinctement la plateforme/application native et les navigateurs pris en charge par chaque DRM.

| **Plateforme cliente** | **Prise en charge DRM native** | **Navigateur/application** | **Formats de diffusion en continu** |
| --- | --- | --- | --- |
| **Téléviseurs intelligents, récepteurs d’opérateur, récepteurs OTT** |PlayReady principalement, et/ou Widevine, et/ou autres |Linux, Opera, WebKit, autre |Divers formats |
| **Appareils Windows 10 (PC Windows, tablettes Windows, Windows Phone, Xbox)** |PlayReady |Microsoft Edge/IE11/EME<br/><br/><br/>Plateforme Windows universelle |DASH (pour HLS, PlayReady n’est pas pris en charge)<br/><br/>DASH, Smooth Streaming (pour HLS, PlayReady n’est pas pris en charge) |
| **Appareils Android (téléphone, tablette, TV)** |Widevine |Chrome/EME |DASH, HLS |
| **iOS (iPhone, iPad), clients OS X et Apple TV** |FairPlay |Safari 8+/EME |HLS |

Compte tenu de l’état actuel du déploiement de chaque DRM, un service met généralement en œuvre deux ou trois DRM pour s’assurer que vous gérez tous les types de points de terminaison de façon optimale.

Il existe un compromis entre la complexité de la logique du service et la complexité côté client pour atteindre un certain niveau d'expérience utilisateur sur les différents clients.

Pour effectuer votre sélection, prenez en compte les points suivants :

* PlayReady est implémenté en mode natif dans tous les appareils Windows, sur certains appareils Android, et est disponible via les kits de développement logiciel (SDK) sur pratiquement n'importe quelle plate-forme.
* Widevine est implémenté en mode natif dans chaque appareil Android, dans Chrome, et dans certains autres appareils.
* FairPlay est disponible uniquement sur iOS et les clients Mac OS ou via iTunes.

Il existe deux options pour un multi-DRM classique :

* PlayReady et Widevine
* PlayReady, Widevine et FairPlay

## <a name="a-reference-design"></a>Une conception de référence
Cette section présente une conception de référence indépendante des technologies utilisées pour sa mise en œuvre.

Un sous-système DRM peut contenir les éléments suivants :

* Gestion des clés
* Packaging DRM
* Remise de licence DRM
* Vérification des droits
* Authentification/autorisation
* Lecteur
* Origine/réseau de diffusion de contenu (CDN)

Le diagramme suivant illustre l’interaction entre les composants d’un sous-système DRM :

![Sous-système de gestion des droits numériques avec CENC](./media/media-services-cenc-with-multidrm-access-control/media-services-generic-drm-subsystem-with-cenc.png)

La conception comprend trois couches de base :

* Une couche back-office (en noir) qui n’est pas présentée à l’extérieur.
* Une couche DMZ (en bleu foncé) qui contient tous les points de terminaison exposés au public.
* Une couche Internet public (en bleu clair) qui contient les réseaux de diffusion en continu et des lecteurs dont le trafic transite par l’Internet public.

Vous devez disposer d’un outil de gestion de contenu pour le contrôle de la protection DRM, que vous utilisiez un chiffrement statique ou dynamique. Données d’entrée pour le chiffrement DRM :

* Contenu vidéo MBR
* Clé de contenu
* URL d’acquisition de licence

Pendant la lecture, le flux de haut niveau suit le déroulement suivant :

* L’utilisateur est authentifié.
* Un jeton d’autorisation est créé pour l’utilisateur.
* Le contenu protégé par DRM (manifeste) est téléchargé sur le lecteur.
* Le lecteur soumet une demande d’acquisition de licence aux serveurs de licence avec un ID de clé et un jeton d’autorisation.

La section suivante décrit la conception de la gestion des clés.

| **ContentKey-to-asset** | **Scénario** |
| --- | --- |
| 1 à 1 |Le cas le plus simple. Il offre le meilleur contrôle. Mais cette configuration génère en général le coût de licence le plus élevé. Chaque élément multimédia protégé nécessite au moins une demande de licence. |
| 1 à plusieurs |Vous pouvez utiliser la même clé de contenu pour plusieurs éléments multimédias. Par exemple, pour tous les éléments multimédias d’un groupe logique, comme un genre ou le sous-ensemble d’un genre (ou séquence génétique), vous pouvez utiliser une clé de contenu unique. |
| Plusieurs à 1 |Plusieurs clés de contenu sont nécessaires pour chaque élément multimédia. <br/><br/>Par exemple, si vous devez appliquer la protection dynamique CENC avec système multi-DRM à MPEG-DASH et le chiffrement AES-128 pour TLS, vous avez besoin de deux clés de contenu distinctes. Chaque clé de contenu doit avoir son propre ContentKeyType. (Pour la clé de contenu utilisée pour la protection CENC dynamique, utilisez ContentKeyType.CommonEncryption. Pour la clé de contenu utilisée pour le chiffrement AES-128 dynamique, utilisez ContentKeyType.EnvelopeEncryption.)<br/><br/>Autre exemple : pour la protection CENC d’un contenu DASH, en théorie, vous pouvez utiliser une clé de contenu pour protéger les flux vidéo et une autre clé de contenu pour protéger les flux audio. |
| Plusieurs à plusieurs |Combinaison des deux scénarios précédents. Un jeu de clés de contenu est utilisé pour chacun des éléments multimédia du même groupe. |

L’utilisation des licences persistantes et non persistantes est un autre facteur important à prendre en compte.

Pourquoi ces considérations sont-elles importantes ?

Si vous utilisez un cloud public pour la remise de licences, les licences persistantes et non persistantes ont un impact direct sur le coût de la remise de licences. Prenons deux cas de conception différents :

* Abonnement mensuel : utilisation d’une licence persistante et d’un mappage 1 à plusieurs entre la clé de contenu et l’élément multimédia. Par exemple, pour tous les films pour enfant, nous utilisons une clé de contenu unique pour le chiffrement. Dans ce cas :

    Nombre total de licences requises pour tous les films/appareils pour enfants = 1

* Abonnement mensuel : utilisation d’une licence non persistante et du mappage 1 à 1 entre la clé de contenu et l’élément multimédia. Dans ce cas :

    Nombre total de licences requises pour tous les films/appareils pour enfants = [nombre de films regardés] x [nombre de séances]

Ces deux conceptions différentes entraînent des modèles de demande de licence bien distincts. Les deux modèles différents débouchent sur des modèles de demande de licence différents si le service de remise de licence est fourni par un cloud public (par exemple, Media Services).

## <a name="map-design-to-technology-for-implementation"></a>Correspondance entre conception et technologie pour la mise en œuvre
La conception générique est ensuite associée à des technologies de la plateforme Azure/Media Services, en spécifiant la technologie à utiliser pour chaque bloc de construction.

Le tableau suivant indique cette correspondance.

| **Bloc de construction** | **Technology** |
| --- | --- |
| **Lecteur** |[Azure Media Player](https://azure.microsoft.com/services/media-services/media-player/) |
| **Fournisseur d’identité (IDP)** |Azure Active Directory (Azure AD) |
| **Service d’émission de jeton de sécurité (STS)** |Azure AD |
| **Flux de travail de protection DRM** |Protection dynamique Media Services |
| **Remise de licence DRM** |* Distribution de licence Media Services (PlayReady, Widevine, FairPlay) <br/>* Serveur de licences Axinom <br/>* Serveur de licences PlayReady personnalisé |
| **Origine** |Point de terminaison de diffusion en continu Media Services |
| **Gestion des clés** |Inutile pour l’implémentation de référence |
| **Gestion de contenu** |Une application de console C# |

En d’autres termes, IDP et STS sont utilisés avec Azure AD. [L’API Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) est utilisée pour le lecteur. Media Services et Media Player prennent en charge DASH et CENC avec système multi-DRM.

Le schéma suivant présente la structure global et le flux de travail avec la correspondance technologique ci-dessus :

![CENC sur Media Services](./media/media-services-cenc-with-multidrm-access-control/media-services-cenc-subsystem-on-AMS-platform.png)

Pour configurer le chiffrement dynamique CENC, l’outil de gestion de contenu utilise les données d’entrée suivantes :

* Contenu ouvert
* Clé de contenu issue de la génération/gestion de clés
* URL d’acquisition de licence
* Liste d’informations fournie par Azure AD

L’outil de gestion de contenu produit le résultat suivant :

* ContentKeyAuthorizationPolicy, qui contient la spécification indiquant la manière dont la distribution de licence vérifie un jeton JWT (JSON Web Token) et les spécifications de licence DRM.
* AssetDeliveryPolicy, qui contient des spécifications sur le format de diffusion en continu, la protection DRM et les URL d’acquisition de licence.

Voici le flux obtenu pendant l’exécution :

* Au moment de l’authentification de l’utilisateur, un jeton JWT est généré.
* L’une des revendications contenues dans le jeton JWT est une revendication de type groupes qui contient l’ID d’objet de groupe EntitledUserGroup. Cette revendication est utilisée pour effectuer la vérification des droits.
* Le lecteur charge le manifeste client d’un contenu protégé par CENC et identifie les éléments suivants :
   * ID de la clé.
   * Le contenu est protégé par CENC.
   * URL d’acquisition de licence.
* Le lecteur effectue une demande d’acquisition de licence en fonction du navigateur/système DRM pris en charge. Dans la demande d’acquisition de licence, l’ID de clé et le jeton JWT sont eux aussi envoyés. Le service de distribution de licence vérifie le jeton JWT et les revendications qu’il contient avant la délivrance de la licence demandée.

## <a name="implementation"></a>Implémentation
### <a name="implementation-procedures"></a>Procédures de mise en œuvre
La mise en œuvre comprend les étapes suivantes :

1. Préparation des éléments multimédias de test. Codez/regroupez une vidéo de test dans un MP4 fragmenté à plusieurs vitesses dans Media Services. Cet élément multimédia *n’est pas* protégé par DRM. La protection DRM est assurée ultérieurement par le biais d’une protection dynamique.

2. Création d’un ID de clé et d’une clé de contenu (éventuellement à partir d’une amorce de clé). Dans ce cas, le système de gestion de clés n’est pas nécessaire car un seul ID de clé et une seule clé de contenu sont nécessaires pour deux éléments multimédias de test.

3. Utilisation de l’API Media Services pour configurer des services de distribution de licence multi-DRM pour l’élément multimédia de test. Si vous utilisez des serveurs de licences personnalisés fournis par votre entreprise ou par des fournisseurs de votre entreprise à la place des services de licence contenus dans Media Services, vous pouvez ignorer cette étape. Vous pouvez spécifier des URL d’acquisition de licence à l’étape de configuration de la distribution de licence. L’API Media Services sert à spécifier certaines configurations détaillées, comme la restriction de la stratégie d’autorisation et les modèles de réponse de licence pour différents services de licence DRM. À ce stade, le portail Azure ne fournit pas l’interface utilisateur nécessaire pour cette configuration. Pour obtenir des informations de niveau API et un exemple de code, consultez [Utilisation du chiffrement commun dynamique PlayReady et/ou Widevine](media-services-protect-with-playready-widevine.md).

4. Utilisation de l’API Media Services pour configurer une stratégie de distribution des éléments multimédias pour l’élément multimédia de test. Pour obtenir des informations de niveau API et un exemple de code, consultez [Utilisation du chiffrement commun dynamique PlayReady et/ou Widevine](media-services-protect-with-playready-widevine.md).

5. Création et configuration d’un locataire Azure AD dans Azure.

6. Création de quelques comptes d’utilisateurs et groupes dans votre locataire Azure AD. Créez au moins un groupe de « Entitled User » et ajoutez un utilisateur à ce groupe. Les utilisateurs de ce groupe sont soumis à la vérification des droits dans l’acquisition de licences. Les utilisateurs qui ne sont pas membres de ce groupe ne peuvent pas réussir le contrôle d’authentification et ne peuvent pas acquérir de licence. L’appartenance à un groupe « Entitled User » est une revendication de groupes obligatoire contenue dans le jeton JWT délivré par Azure AD. Vous spécifiez cette exigence de revendication à l’étape de configuration des services de remise de licences multi-DRM.

7. Création d’une application MVC ASP.NET pour héberger votre lecteur vidéo. Cette application ASP.NET sera protégée par une authentification utilisateurs par rapport au locataire Azure AD. Des revendications adéquates sont incluses dans les jetons d’accès obtenus après authentification de l’utilisateur. Nous vous recommandons d’utiliser l’API OpenID Connect pour cette étape. Installez les packages NuGet suivants :

   * Install-Package Microsoft.Azure.ActiveDirectory.GraphClient
   * Install-Package Microsoft.Owin.Security.OpenIdConnect
   * Install-Package Microsoft.Owin.Security.Cookies
   * Install-Package Microsoft.Owin.Host.SystemWeb
   * Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory

8. Création d’un lecteur à l’aide de [l’API Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/). Utilisez [l’API ProtectionInfo d’Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) pour spécifier la technologie DRM à utiliser sur d’autres plateformes DRM.

9. Le tableau suivant illustre la matrice de test.

    | **DRM** | **Browser** | **Résultat pour un utilisateur autorisé** | **Résultat pour un utilisateur non autorisé** |
    | --- | --- | --- | --- |
    | **PlayReady** |Microsoft Edge ou Internet Explorer 11 sur Windows 10 |Réussite |Échec |
    | **Widevine** |Chrome, Firefox, Opera |Réussite |Échec |
    | **FairPlay** |Safari sur macOS      |Réussite |Échec |
    | **AES-128** |La plupart des navigateurs modernes  |Réussite |Échec |

Pour plus d’informations sur la configuration d’Azure AD pour une application de lecteur MVC ASP.NET, consultez la page [Integrate an Azure Media Services OWIN MVC-based app with Azure Active Directory and restrict content key delivery based on JWT claims](http://gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/) (Intégration d’une application Azure Media Services basée sur OWIN MVC avec Azure Active Directory et remise de clés de contenu basée sur les revendications JWT).

Pour plus d’informations, consultez la page [JWT token authentication in Azure Media Services and dynamic encryption](http://gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/) (Authentification des jetons JWT dans Azure Media Services et chiffrement dynamique).  

Pour plus d’informations sur Azure AD :

* Vous pouvez trouver des informations pour les développeurs dans le [Guide du développeur Azure Active Directory](../../active-directory/azuread-dev/v1-overview.md).
* Vous pouvez trouver des informations pour l’administrateur dans la rubrique [Administration de votre annuaire Azure AD](../../active-directory/fundamentals/active-directory-whatis.md).

### <a name="some-issues-in-implementation"></a>Problèmes de mise en œuvre
Utilisez les informations de dépannage suivantes pour résoudre vos éventuels problèmes d’implémentation.

* L’URL de l’émetteur doit se terminer par « / ». L’audience doit être l’ID client de l’application de lecteur. Ajoutez également « / » à la fin de l’URL de l’émetteur.

    ```xml
    <add key="ida:audience" value="[Application Client ID GUID]" />
    <add key="ida:issuer" value="https://sts.windows.net/[AAD Tenant ID]/" />
    ```

    Dans le [Décodeur JWT](http://jwt.calebb.net/), **aud** et **iss** apparaissent comme suit dans le jeton JWT :

    ![JWT](./media/media-services-cenc-with-multidrm-access-control/media-services-1st-gotcha.png)

* Ajoutez des autorisations à l’application dans Azure AD sur l’onglet **Configuration** de l’application. Des autorisations sont demandées pour chaque application, aussi bien pour les versions locales que pour les versions déployées.

    ![Autorisations](./media/media-services-cenc-with-multidrm-access-control/media-services-perms-to-other-apps.png)

* Utilisez le bon émetteur lorsque vous configurez la protection CENC dynamique.

    ```xml
    <add key="ida:issuer" value="https://sts.windows.net/[AAD Tenant ID]/"/>
    ```

    Ce paramètre ne fonctionne pas :

    ```xml
    <add key="ida:issuer" value="https://willzhanad.onmicrosoft.com/" />
    ```

    Le GUID est l’ID du locataire Azure AD. Vous trouverez le GUID dans le menu contextuel **Points de terminaison** du portail Azure.

* Octroi des autorisations de revendications de membre de groupe. Assurez-vous que le fichier de manifeste de l’application Azure AD comprend bien les paramètres suivants : 

    « groupMembershipClaims » : « All » (la valeur par défaut est Null)

* Définissez le TokenType approprié lorsque vous créez des conditions de restriction.

    ```csharp
    objTokenRestrictionTemplate.TokenType = TokenType.JWT;
    ```

    Puisque vous ajoutez la prise en charge de JWT (Azure AD) en plus des SWT (ACS), la valeur par défaut de TokenType est TokenType.JWT. Si vous utilisez SWT/ACS, vous devez définir le jeton sur TokenType.SWT.

## <a name="additional-topics-for-implementation"></a>Aspects supplémentaires pour l’implémentation
Cette section aborde certains aspects de conception et d’implémentation supplémentaires.

### <a name="http-or-https"></a>HTTP ou HTTPS ?
L’application de lecteur MVC ASP.NET doit prendre en charge les éléments suivants :

* Authentification des utilisateurs via Azure AD, sous HTTPS.
* Échange de jeton JWT entre le client et Azure AD, sous HTTPS.
* Acquisition de licence DRM par le client, qui doit être sous HTTPS si la distribution de licences est assurée par Azure Media Services. La suite de produits PlayReady n’impose pas le format HTTPS pour la distribution de licences. Si votre serveur de licences PlayReady se trouve en dehors de Media Services, vous pouvez utiliser HTTP ou HTTPS.

Puisque l’application de lecteur ASP.NET utilise le protocole HTTPS en tant que meilleure pratique, Media Player se trouve sur une page sous HTTPS. Le protocole HTTP est cependant privilégié pour la diffusion en continu, ce qui signifie que vous devez tenir compte du problème du contenu mixte.

* Le navigateur ne permet pas un contenu mixte. Mais vous pouvez utiliser des plug-ins tels que Silverlight et OSMF pour Smooth et DASH. Le contenu mixte est un problème de sécurité, en raison de la menace que constitue la possibilité d’injecter un logiciel JavaScript malveillant, qui risque de mettre en danger les données du client. Les navigateurs bloquent cette fonctionnalité par défaut. Le seul moyen de contourner ce problème est d’autoriser tous les domaines (HTTPS ou HTTP) côté serveur (origine). Il ne s’agit probablement pas d’une idée judicieuse.
* Évitez le contenu mixte. L’application de lecteur et Media Player doivent utiliser HTTP ou HTTPS. Si vous lisez du contenu mixte, silverlightSS tech vous oblige à supprimer un avertissement de contenu mixte. flashSS tech traite le contenu mixte sans aucun avertissement.
* Si votre point de terminaison de diffusion a été créé avant août 2014, il ne prend pas en charge HTTPS. Dans ce cas, créez et utilisez un nouveau point de terminaison pour le protocole HTTPS.

Dans l’implémentation de référence pour des contenus protégés par DRM, l’application et la diffusion en continu s’exécutent l’une comme l’autre sous HTTPS. Pour les contenus ouverts, le lecteur n’a pas besoin d’authentification ou de licence. Vous pouvez utiliser soit HTTP, soit HTTPS.

### <a name="azure-active-directory-signing-key-rollover"></a>Substitution de la clé de signature Azure Active Directory
La substitution de la clé de signature est un point important à prendre en compte pour votre implémentation. Si vous l’ignorez, le système finira par s’arrêter complètement dans un délai maximum de six semaines.

Azure AD utilise des normes reconnues pour établir une relation de confiance entre lui-même et les applications qui utilisent Azure AD. Azure AD utilise plus particulièrement une clé de signature se composant d’une paire clé publique-clé privée. Lorsqu’Azure AD crée un jeton de sécurité contenant des informations sur l’utilisateur, ce jeton est signé par Azure AD à l’aide d’une clé privée avant d’être renvoyé à l’application. Pour vérifier que le jeton est valide et qu’il a bien été émis par Azure AD, l’application doit valider la signature du jeton. L’application utilise la clé publique exposée par Azure AD, laquelle est contenue dans le document des métadonnées de fédération du locataire. Cette clé publique, de même que la clé de signature dont elle est dérivée, est la même que celle utilisée pour tous les locataires dans Azure AD.

Pour plus d’informations sur la substitution de la clé Azure AD, consultez l’article [Substitution de la clé de signature dans Azure Active Directory](../../active-directory/develop/active-directory-signing-key-rollover.md).

Dans la [paire de clés publique-privée](https://login.microsoftonline.com/common/discovery/keys/) :

* La clé privée est utilisée par Azure AD pour créer un jeton JWT.
* La clé publique est utilisée par une application telle que les services de distribution de licences DRM dans Media Services pour vérifier le jeton JWT.

Pour des raisons de sécurité, Azure AD change régulièrement le certificat (toutes les six semaines). La substitution de clé peut survenir à tout moment en cas de violation de la sécurité. Par conséquent, les services de distribution de licences dans Media Services doivent mettre à jour la clé publique utilisée à chaque fois qu’Azure AD change la paire de clés. Dans le cas contraire, l’authentification par jeton dans Media Services échoue et aucune licence n’est émise.

Pour configurer ce service, vous devez définir TokenRestrictionTemplate.OpenIdConnectDiscoveryDocument au moment de configurer les services de distribution de licences DRM.

Voici comment fonctionne le jeton JWT :

* Azure AD émet le jeton JWT avec la clé privée actuelle pour un utilisateur authentifié.
* Lorsqu’un lecteur voit un CENC avec un contenu protégé par un système multi-DRM, il commence par localiser le jeton JWT émis par Azure AD.
* Le lecteur envoie une demande d’acquisition de licence avec le jeton JWT aux services de distribution de licences dans Media Services.
* Les services de distribution de licences dans Media Services utilisent la clé publique actuelle/valide depuis Azure AD pour vérifier le jeton JWT avant d’émettre des licences.

Les services de distribution de licences DRM vérifient systématiquement la clé publique actuelle/valide à partir d’Azure AD. La clé publique présentée par Azure AD est la clé utilisée pour vérifier un jeton JWT émis par Azure AD.

Que se passe-t-il si la substitution de la clé a lieu après qu’Azure AD a généré un jeton JWT, mais avant que le jeton JSON soit envoyé par les lecteurs aux services de distribution de licences DRM dans Media Services pour vérification ?

Une clé pouvant être substituée à tout moment, il y a toujours plusieurs clés publiques valides disponibles dans le document de métadonnées de la fédération. La distribution de licences Media Services peut utiliser n’importe quelle clé spécifiée dans le document. Puisqu’une clé peut être rapidement changée, une autre peut être utilisée en remplacement et ainsi de suite.

### <a name="where-is-the-access-token"></a>Où se trouve le jeton d’accès ?
Si vous regardez comment une application web appelle une application API sous [Identité d’application avec octroi d’informations d’identification client OAuth 2.0](../../active-directory/azuread-dev/web-api.md), vous obtenez le flux d’authentification suivant :

* Un utilisateur se connecte à Azure AD dans l’application web. Pour plus d’informations, voir la rubrique [Navigateur web vers application web](../../active-directory/azuread-dev/web-app.md).
* Le point de terminaison d’autorisation Azure AD redirige l’agent utilisateur vers l’application cliente avec un code d’autorisation. L’agent utilisateur renvoie le code d’autorisation à l’URI de redirection de l’application cliente.
* L’application web doit obtenir un jeton d’accès pour pouvoir s’authentifier auprès de l’API web et extraire la ressource souhaitée. Elle envoie une demande au point de terminaison du jeton Azure AD et fournit les informations d’identification, l’ID client et l’URI ID d’application de l’API web. Elle présente le code d’autorisation pour prouver que l’utilisateur a donné son consentement.
* Azure AD authentifie l’application et renvoie un jeton d’accès JWT, qui est utilisé pour appeler l’API web.
* Sur HTTPS, l’application web utilise le jeton d’accès JWT renvoyé pour ajouter la chaîne JWT avec la mention « Porteur » dans l’en-tête « Autorisation » de la demande adressée à l’API web. L’API web valide ensuite le jeton JWT. Si la validation réussit, elle renvoie la ressource souhaitée.

Dans ce flux d’identité de l’application, l’API web suppose que l’application web a authentifié l’utilisateur. C’est pour cette raison que ce modèle est appelé « sous-système approuvé ». Le [schéma de flux d’autorisation](../../active-directory/azuread-dev/v1-protocols-oauth-code.md) explique comment fonctionne le flux relatif au code d’autorisation.

L’acquisition de licence avec restriction de jeton suit le même modèle de sous-système approuvé. Le service de distribution de licences dans Media Services est une ressource API web, ou bien la « ressource backend » à laquelle une application web doit accéder. Où se trouve le jeton d’accès ?

Le jeton d’accès est obtenu à partir d’Azure AD. Une fois l’utilisateur authentifié, un code d’autorisation est renvoyé. Il est ensuite utilisé avec l’ID client et la clé d’application en échange d’un jeton d’accès. Le jeton d’accès sert à accéder à une application de « pointeur » qui pointe vers ou représente le service de distribution de licences Media Services.

Pour inscrire et configurer l’application de pointeur dans Azure AD, suivez les étapes ci-dessous :

1. Dans le locataire Azure AD :

   * Ajoutez une application (ressource) avec l’URL de connexion https://[nom_ressource].azurewebsites.net/. 
   * Ajoutez un ID d’application avec l’URL https://[nom_locataire_aad].onmicrosoft.com/[nom_ressource].

2. Ajoutez une nouvelle clé à l’application de ressource.

3. Mettez à jour le fichier manifeste de l’application afin que la propriété groupMembershipClaims se voie attribuer la valeur « groupMembershipClaims » : « All ».

4. Dans l’application Azure AD qui pointe vers l’application web du lecteur, dans la section **Autorisations pour d’autres applications**, ajoutez l’application de ressource ajoutée à l’étape 1. Sous **Autorisations déléguées**, sélectionnez **Accès [nom_ressource]** . Cette option donne à l’application web l’autorisation de créer des jetons d’accès pour accéder à l’application de ressource. Procédez ainsi à la fois pour la version locale et pour la version déployée de l’application web si vous effectuez un développement avec Visual Studio et une application web Azure.

Le jeton JWT émis par Azure AD est le jeton d’accès utilisé pour accéder à la ressource de pointeur.

### <a name="what-about-live-streaming"></a>Qu’en est-il de la diffusion en continu ?
La discussion précédente était essentiellement concentrée sur les éléments multimédias à la demande. Qu’en est-il de la diffusion en continu ?

Vous pouvez utiliser exactement la même conception et la même implémentation pour protéger un service de diffusion en continu dans Media Services, en traitant l’élément multimédia associé à un programme comme un élément multimédia VOD.

Pour diffuser en continu dans Media Services, vous devez plus spécifiquement créer un canal, puis créer un programme sous ce canal. Pour créer le programme, vous devez créer un élément contenant l’archive en direct du programme. Pour pouvoir protéger du contenu en direct avec CENC et un système multi-DRM, appliquez la même procédure d’installation/de traitement à l’élément multimédia, comme s’il s’agissait d’un élément VOD, avant de démarrer le programme.

### <a name="what-about-license-servers-outside-media-services"></a>Qu’en est-il des serveurs de licences hors Media Services ?
Souvent, les clients investissent dans une batterie de serveurs qu’ils hébergent dans leur propre centre de données ou chez des fournisseurs de services DRM. Avec la protection de contenu Media Services, vous pouvez fonctionner en mode hybride. Le contenu peut être hébergé et protégé dynamiquement dans Media Services, alors que les licences DRM sont délivrées par des serveurs situés en dehors de Media Services. Dans ce cas, envisagez les modifications suivantes :

* Le service STS doit émettre des jetons acceptables et pouvant être vérifiés par la batterie de serveurs de licence. Par exemple, les serveurs de licences Widevine fournis par Axinom exigent un jeton JWT spécifique contenant un message d’octroi de droit. Par conséquent, vous devez disposer d’un STS pour émettre un jeton JWT. Pour plus d’informations sur ce type d’implémentation, consultez le [Centre de documentation Azure](https://azure.microsoft.com/documentation/) et l’article [Utilisation d’Axinom pour fournir des licences Widevine à Azure Media Services](media-services-axinom-integration.md).
* Vous n’avez plus besoin de configurer le service de distribution de licences (ContentKeyAuthorizationPolicy) dans Media Services. Vous devez fournir les URL d’acquisition de la licence (pour PlayReady, Widevine et FairPlay) au moment où vous configurez AssetDeliveryPolicy pour paramétrer CENC avec un système multi-DRM.

### <a name="what-if-i-want-to-use-a-custom-sts"></a>Que se passe-t-il si je souhaite utiliser un STS personnalisé ?
Un client peut choisir d’utiliser un STS personnalisé pour fournir des jetons JWT. En voici plusieurs raisons :

* Le fournisseur d’identité utilisé par le client ne prend pas en charge STS. Dans ce cas, un service STS personnalisé peut être une solution.
* Le client peut avoir besoin de contrôler de manière plus souple ou plus stricte l’intégration de STS avec le système de facturation des abonnés du client. Par exemple, un opérateur MVPD peut proposer plusieurs offres d’abonné OTT, par exemple des offres premium, de base et sports. L’opérateur peut chercher à associer les revendications d’un jeton avec l’offre d’un abonné afin que seul le contenu d’une offre spécifique soit mis à disposition. Dans ce cas, un STS personnalisé offre la souplesse et la maîtrise nécessaires.

Lorsque vous utilisez un STS personnalisé, vous devez effectuer deux modifications :

* Lorsque vous configurez le service de distribution de licences pour un élément multimédia, vous devez spécifier la clé de sécurité utilisée par le STS personnalisé, et non la clé actuelle d’Azure AD. (Voir détails ci-dessous.) 
* Une fois le jeton JTW généré, une clé de sécurité est spécifiée à la place de la clé privée du certificat X509 courant dans Azure AD.

Il existe deux types de clés de sécurité :

* Clé symétrique : la même clé est utilisée pour générer et vérifier un jeton JWT.
* Clé asymétrique : une paire de clés publique-privée dans un certificat X509 est utilisée avec une clé privée pour chiffrer/générer un jeton JWT et avec la clé publique pour vérifier le jeton.

> [!NOTE]
> Si vous utilisez .NET Framework/C# en tant que plateforme de développement, le certificat X509 utilisé pour la clé de sécurité asymétrique doit avoir une longueur d’au moins 2 048 bits. Il s’agit d’une exigence de la classe System.IdentityModel.Tokens.X509AsymmetricSecurityKey dans .NET Framework. Dans le cas contraire, l’exception suivante est générée :
> 
> IDX10630 : la longueur de la signature « System.IdentityModel.Tokens.X509AsymmetricSecurityKey » ne peut pas être inférieure à « 2048 » bits.

## <a name="the-completed-system-and-test"></a>Le système et le test terminé
Cette section décrit quelques scénarios dans le système achevé de bout en bout afin que vous puissiez avoir une vision générale du comportement avant d’obtenir un compte de connexion :

* Si vous avez besoin d’un scénario non intégré :

    * Pour les éléments vidéos hébergés dans Media Services qui ne sont pas protégés ou qui sont protégés DRM mais sans authentification par jeton (émission d’une licence à la personne qui en a fait la demande), vous pouvez tester le scénario sans vous connecter. Basculez sur HTTP si votre vidéo est diffusée en continu via HTTP.

* Si vous avez besoin d’un scénario intégré de bout en bout :

    * Pour les éléments vidéos sous protection DRM dynamique dans Media Services qui utilisent une authentification par jeton et un jeton JWT généré par Azure AD, vous devez vous connecter.

Pour l’application web de lecteur et sa connexion, consultez [ce site web](https://openidconnectweb.azurewebsites.net/).

### <a name="user-sign-in"></a>Connexion de l’utilisateur
Pour tester le système DRM intégré de bout en bout, vous devez disposer d’un compte créé ou ajouté.

Quel compte ?

Même si Azure avait initialement autorisé l’accès aux utilisateurs de compte Microsoft uniquement, il autorise désormais l’accès aux utilisateurs des deux systèmes. Toutes les propriétés Azure approuvent Azure AD pour l’authentification et Azure AD authentifie les utilisateurs professionnels. Une relation de fédération a été établie dans laquelle Azure AD approuve le système d’identité consommateur du compte Microsoft pour authentifier les utilisateurs consommateurs. Par conséquent, Azure AD est en mesure d'authentifier les comptes invités de Microsoft, ainsi que les comptes Azure AD natifs.

Dans la mesure où Azure AD approuve le domaine de compte Microsoft, vous pouvez ajouter tous les comptes dans les domaines suivants du locataire Azure AD personnalisé et utiliser le compte pour vous connecter :

| **Nom de domaine** | **Domaine** |
| --- | --- |
| **Domaine client Azure AD personnalisé** |somename.onmicrosoft.com |
| **Domaine d’entreprise** |microsoft.com |
| **Domaine de compte Microsoft** |outlook.com, live.com, hotmail.com |

Vous pouvez contacter l’un des auteurs pour qu’un compte soit créé ou ajouté pour vous.

Les captures d’écran ci-dessous présentent les différentes pages de connexion utilisées par les différents comptes de domaine :

**Compte de domaine client Azure AD personnalisé** : page de connexion personnalisée du domaine client Azure AD.

![Capture d’écran montrant la page de connexion personnalisée du domaine client Azure AD personnalisé.](./media/media-services-cenc-with-multidrm-access-control/media-services-ad-tenant-domain1.png)

**Compte de domaine Microsoft avec carte à puce** : page de connexion personnalisée par l’informatique d’entreprise Microsoft avec authentification à deux facteurs.

![Capture d’écran montrant la page de connexion personnalisée par l’informatique d’entreprise Microsoft avec authentification à deux facteurs.](./media/media-services-cenc-with-multidrm-access-control/media-services-ad-tenant-domain2.png)

**Compte Microsoft** : page de connexion du compte Microsoft pour les consommateurs.

![compte de domaine client Azure personnalisé](./media/media-services-cenc-with-multidrm-access-control/media-services-ad-tenant-domain3.png)

### <a name="use-encrypted-media-extensions-for-playready"></a>Utilisation d’Encrypted Media Extensions pour PlayReady
Sur un navigateur moderne prenant en charge Encrypted Media Extensions (EME) pour PlayReady tel qu’Internet Explorer 11 sous Windows 8.1 et version ultérieure et le navigateur Microsoft Edge sous Windows 10, PlayReady sera la DRM sous-jacente d’EME.

![Utilisation d’EME pour PlayReady](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-playready1.png)

La zone de lecteur foncée est due au fait que la protection PlayReady empêche d’effectuer une capture d’écran de la vidéo protégée.

La capture d’écran suivante montre les plug-ins de lecteur et les composants Microsoft Security Essentials (MSE)/EME pris en charge :

![Plug-ins de lecteur pour PlayReady](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-playready2.png)

EME dans Microsoft Edge et Internet Explorer 11 sur Windows 10 permet d’appeler [PlayReady SL3000](https://www.microsoft.com/playready/features/EnhancedContentProtection.aspx/) sur les appareils Windows 10 compatibles. PlayReady SL3000 déverrouille le flux de contenu premium améliorées (4K, HDR) et les nouveaux modèles de distribution de contenu (pour le contenu amélioré).

Pour vous concentrer uniquement sur les appareils Windows, PlayReady est le seul système DRM dans le matériel disponible sur les appareils Windows (PlayReady SL3000). Un service de diffusion en continu peut utiliser PlayReady via EME ou via une application UWP (Universal Windows Platform), et offrir ainsi une meilleure qualité vidéo à l’aide de PlayReady SL3000 par rapport à un autre DRM. En règle générale, l’utilisation de Chrome ou de Firefox permet de diffuser du contenu jusqu’à 2K, tandis que Microsoft Edge/Internet Explorer 11 ou une application UWP prend en charge du contenu jusqu’à 4K sur le même appareil. La quantité dépend des paramètres de service et de l’implémentation.

#### <a name="use-eme-for-widevine"></a>Utilisation d’EME pour Widevine
Sur un navigateur moderne doté de la prise en charge d’EME/Widevine, telle que Chrome 41 + sous Windows 10, Windows 8.1, Mac OSX Yosemite et Chrome sur Android 4.4.4, la gestion des droits numériques derrière EME est assurée par Google Widevine.

![Utilisation d’EME pour Widevine](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-widevine1.png)

Widevine ne vous empêche pas d’effectuer une capture d’écran de la vidéo protégée.

![Plug-ins de lecteur pour Widevine](./media/media-services-cenc-with-multidrm-access-control/media-services-eme-for-widevine2.png)

### <a name="unentitled-users"></a>Utilisateurs non habilités
Si un utilisateur n’est pas un membre du groupe d’utilisateurs habilités, l’utilisateur échoue à la vérification des droits. Le service de licence multi-DRM refuse alors d’émettre la licence requise comme indiqué. La description détaillée est « L’acquisition de licence a échoué », ce qui correspond à la conception.

![Utilisateurs non habilités](./media/media-services-cenc-with-multidrm-access-control/media-services-unentitledusers.png)

### <a name="run-a-custom-security-token-service"></a>Exécution d’un service d’émission de jeton de sécurité (STS) personnalisé
Si vous exécutez un STS personnalisé, le jeton JWT est émis par le STS personnalisé à l’aide d’une clé symétrique ou asymétrique.

La capture d’écran suivante montre un scénario qui utilise une clé symétrique (à l’aide de Chrome) :

![STS personnalisé avec clé symétrique](./media/media-services-cenc-with-multidrm-access-control/media-services-running-sts1.png)

La capture d’écran suivante montre un scénario qui utilise une clé asymétrique via un certificat X509 (à l’aide d’un navigateur Microsoft moderne) :

![STS personnalisé avec clé asymétrique](./media/media-services-cenc-with-multidrm-access-control/media-services-running-sts2.png)

Dans les deux cas ci-dessus, l’authentification utilisateur reste la même. Elle passe par Azure AD. La seule différence est que les jetons JWT sont émis par le STS personnalisé et non par Azure AD. Lorsque vous configurez la protection CENC dynamique, la restriction du service de distribution de licences spécifie le type de jeton JWT, soit avec une clé symétrique soit avec une clé asymétrique.

## <a name="summary"></a>Résumé

Dans ce document, nous avons évoqué l’utilisation de CENC avec un système multi-DRM natif et un contrôle d’accès via une authentification par jeton. Nous avons vu sa conception et son implémentation à l’aide d’Azure, de Media Services et de Media Player.

* Nous avons proposé une conception de référence contenant tous les composants nécessaires dans un sous-système DRM/CENC.
* Nous avons également présenté une implémentation de référence sur Azure, Media Services et Media Player.
* Certains aspects directement impliqués dans la conception et l’implémentation ont également été abordés.

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
 
