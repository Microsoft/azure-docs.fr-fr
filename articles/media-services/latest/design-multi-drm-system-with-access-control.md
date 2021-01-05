---
title: Système de protection de contenu multi-DRM – Azure Media Services V3
description: Cet article fournit une description détaillée de la conception d’un système de protection du contenu multi-DRM avec Azure Media Services.
services: media-services
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: seodec18
ms.openlocfilehash: 58edf1e0257cf9de8d8f3a3b56f295dcaf1f6cbf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89298196"
---
# <a name="design-of-a-multi-drm-content-protection-system-with-access-control"></a>Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Il est assez complexe de concevoir et développer un sous-système de gestion des droits numériques (DRM) pour une solution OTT (Over-The-Top) ou de diffusion en continu en ligne. Les opérateurs/fournisseurs de vidéo en ligne ont l’habitude d’externaliser cette tâche à des fournisseurs de services DRM spécialisés. L’objectif de ce document est de présenter une conception et une implémentation de référence pour un sous-système DRM de bout en bout dans une solution OTT ou de diffusion en continu en ligne.

Ce document s’adresse aux ingénieurs qui travaillent sur des sous-systèmes DRM de solutions OTT ou de solutions de diffusion en continu en ligne/multi-écrans, ou à tout lecteur intéressé par les sous-systèmes de gestion des droits numériques. Les lecteurs doivent connaître au moins une des technologies DRM du marché, telles que PlayReady, Widevine, FairPlay ou Adobe Access.

Dans cette discussion, le terme « multi-DRM » inclut les trois DRM pris en charge par Azure Media Services : le chiffrement commun (CENC) pour PlayReady et Widevine, FairPlay, ainsi que le chiffrement AES-128 avec clé en clair. Le marché de la diffusion en continu en ligne et de l’OTT est marqué par une nouvelle tendance qui consiste à utiliser un système multi-DRM natif sur différentes plateformes clientes. Cette tendance témoigne d’un bouleversement par rapport à l’époque où l’on utilisait un seul système DRM et son SDK client pour différentes plateformes clientes. Lorsque vous utilisez CENC avec un système multi-DRM natif, PlayReady et Widevine sont chiffrés conformément à la spécification [Common Encryption (ISO/IEC 23001-7 CENC)](https://www.iso.org/iso/home/store/catalogue_ics/catalogue_detail_ics.htm?csnumber=65271/).

Les avantages d’un système multi-DRM natif pour la protection du contenu sont les suivants :

* réduction des frais de chiffrement dans la mesure où un seul processus de chiffrement est utilisé pour cibler différentes plateformes avec ses systèmes DRM natifs ;
* réduction du coût de gestion des éléments multimédias, puisqu’une seule copie de ces éléments est nécessaire à l’emplacement de stockage ;
* élimination des coûts de licence client DRM, car le client DRM natif est généralement proposé gratuitement sur sa plateforme d’origine.

### <a name="goals-of-the-article"></a>Objectifs de l’article

Les objectifs de cet article sont les suivants :

* conception de référence d’un sous-système DRM qui utilise les trois DRM (CENC pour DASH, FairPlay pour HLS et PlayReady pour la diffusion en continu lisse) ;
* implémentation de référence sur une plateforme Azure et Azure Media Services ;
* commentaires de certaines rubriques relatives à la conception et l’implémentation.

Le tableau suivant récapitule la prise en charge des systèmes DRM natifs sur différentes plateformes et la prise en charge d’EME dans différents navigateurs.

| **Plateforme cliente** | **DRM natif** | **EME** |
| --- | --- | --- |
| **Téléviseurs connectés, décodeurs** | PlayReady, Widevine et/ou autres | EME/Navigateur incorporé pour PlayReady et/ou Widevine|
| **Windows 10** | PlayReady | Microsoft Edge/IE11 pour PlayReady|
| **Appareils Android (téléphone, tablette, TV)** |Widevine |Chrome pour Widevine |
| **iOS** | FairPlay | Safari pour FairPlay (à partir d’iOS 11.2) |
| **macOS** | FairPlay | Safari pour FairPlay (à partir de Safari 9+ sur Mac OS X 10.11+ El Capitan)|
| **tvOS** | FairPlay | |

Compte tenu de l’état actuel du déploiement de chaque DRM, un service met généralement en œuvre deux ou trois DRM pour s’assurer que vous gérez tous les types de points de terminaison de façon optimale.

Il existe un compromis entre la complexité de la logique du service et la complexité côté client pour atteindre un certain niveau d'expérience utilisateur sur les différents clients.

Pour effectuer votre sélection, prenez en compte les points suivants :

* PlayReady est implémenté en mode natif dans tous les appareils Windows, sur certains appareils Android, et est disponible via les kits de développement logiciel (SDK) sur pratiquement n'importe quelle plate-forme.
* Widevine est implémenté en mode natif dans chaque appareil Android, dans Chrome, et dans certains autres appareils. Widevine est également pris en charge dans les navigateurs Firefox et Opera sur DASH.
* FairPlay est disponible sur iOS, macOS et tvOS.


## <a name="a-reference-design"></a>Une conception de référence
Cette section présente une conception de référence indépendante des technologies utilisées pour sa mise en œuvre.

Un sous-système DRM peut contenir les éléments suivants :

* Gestion des clés
* Packaging de chiffrement DRM
* Remise de licence DRM
* Vérification des droits/Contrôle d'accès
* Authentification/Autorisation utilisateur
* Application de lecteur
* Origine/réseau de diffusion de contenu (CDN)

Le diagramme suivant illustre l’interaction entre les composants d’un sous-système DRM :

![Sous-système de gestion des droits numériques avec CENC](./media/design-multi-drm-system-with-access-control/media-services-generic-drm-subsystem-with-cenc.png)

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
| **Secure Token Service (Service d’émission de jeton de sécurité - STS)** |Azure AD |
| **Flux de travail de protection DRM** |Protection dynamique Azure Media Services |
| **Remise de licence DRM** |* Distribution de licence Media Services (PlayReady, Widevine, FairPlay) <br/>* Serveur de licences Axinom <br/>* Serveur de licences PlayReady personnalisé |
| **Origine** |Point de terminaison de streaming Azure Media Services |
| **Gestion des clés** |Inutile pour l’implémentation de référence |
| **Gestion de contenu** |Une application de console C# |

En d’autres termes, IDP et STS sont fournis par Azure AD. [L’API Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) est utilisée pour le lecteur. Azure Media Services et le lecteur multimédia Azure prennent en charge CENC sur DASH, FairPlay sur HLS, PlayReady sur la diffusion en continu lisse et le chiffrement AES-128 pour DASH, HLS et la diffusion en continu lisse.

Le schéma suivant présente la structure global et le flux de travail avec la correspondance technologique ci-dessus :

![CENC sur Media Services](./media/design-multi-drm-system-with-access-control/media-services-cenc-subsystem-on-AMS-platform.png)

Pour configurer la protection de contenu DRM, l’outil de gestion de contenu utilise les données d’entrée suivantes :

* Contenu ouvert
* Clé de contenu issue de la gestion de clés
* URL d’acquisition de licence
* Liste d’informations provenant d’Azure AD, telles que l’audience, l’émetteur et les revendications de jeton

L’outil de gestion de contenu produit le résultat suivant :

* ContentKeyPolicy décrit le modèle de licence DRM pour chaque type de DRM utilisé.
* ContentKeyPolicyRestriction décrit le contrôle d’accès avant l’émission d’une licence DRM.
* Streamingpolicy décrit les différentes combinaisons de format de conteneur, mode de chiffrement, protocole de diffusion en continu et DRM, pour la diffusion en continu.
* StreamingLocator décrit la clé de contenu/le vecteur d’initialisation utilisés pour le chiffrement et les URL de diffusion en continu. 

Voici le flux obtenu pendant l’exécution :

* Au moment de l’authentification de l’utilisateur, un jeton JWT est généré.
* L’une des revendications contenues dans le jeton JWT est une revendication de type groupes qui contient l’ID d’objet de groupe EntitledUserGroup. Cette revendication est utilisée pour effectuer la vérification des droits.
* Le lecteur charge le manifeste client d’un contenu protégé par CENC et identifie les éléments suivants :
   * ID de la clé.
   * Le contenu est protégé par DRM.
   * URL d’acquisition de licence.
* Le lecteur effectue une demande d’acquisition de licence en fonction du navigateur/système DRM pris en charge. Dans la demande d’acquisition de licence, l’ID de clé et le jeton JWT sont eux aussi envoyés. Le service de distribution de licence vérifie le jeton JWT et les revendications qu’il contient avant la délivrance de la licence demandée.

## <a name="implementation"></a>Implémentation
### <a name="implementation-procedures"></a>Procédures de mise en œuvre
La mise en œuvre comprend les étapes suivantes :

1. Préparation des éléments multimédias de test. Codez/regroupez une vidéo de test dans un MP4 fragmenté à plusieurs vitesses dans Media Services. Cet élément multimédia *n’est pas* protégé par DRM. La protection DRM est assurée ultérieurement par le biais d’une protection dynamique.

2. Création d’un ID de clé et d’une clé de contenu (éventuellement à partir d’une amorce de clé). Dans ce cas, le système de gestion de clés n’est pas nécessaire car un seul ID de clé et une seule clé de contenu sont nécessaires pour deux éléments multimédias de test.

3. Utilisation de l’API Media Services pour configurer des services de distribution de licence multi-DRM pour l’élément multimédia de test. Si vous utilisez des serveurs de licences personnalisés fournis par votre entreprise ou par des fournisseurs de votre entreprise à la place des services de licence contenus dans Media Services, vous pouvez ignorer cette étape. Vous pouvez spécifier des URL d’acquisition de licence à l’étape de configuration de la distribution de licence. L’API Media Services sert à spécifier certaines configurations détaillées, comme la restriction de la stratégie d’autorisation et les modèles de réponse de licence pour différents services de licence DRM. À ce stade, le portail Azure ne fournit pas l’interface utilisateur nécessaire pour cette configuration. Pour obtenir des informations de niveau API et un exemple de code, consultez [Utilisation du chiffrement commun dynamique PlayReady et/ou Widevine](protect-with-drm.md).

4. Utilisation de l’API Media Services pour configurer une stratégie de distribution des éléments multimédias pour l’élément multimédia de test. Pour obtenir des informations de niveau API et un exemple de code, consultez [Utilisation du chiffrement commun dynamique PlayReady et/ou Widevine](protect-with-drm.md).

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

* Vous pouvez trouver des informations pour les développeurs dans le [Guide du développeur Azure Active Directory](../../active-directory/develop/v2-overview.md).
* Vous pouvez trouver des informations pour l’administrateur dans la rubrique [Administration de votre annuaire Azure AD](../../active-directory/fundamentals/active-directory-whatis.md).

### <a name="some-issues-in-implementation"></a>Problèmes de mise en œuvre

Utilisez les informations de dépannage suivantes pour résoudre vos éventuels problèmes d’implémentation.

* L’URL de l’émetteur doit se terminer par « / ». L’audience doit être l’ID client de l’application de lecteur. Ajoutez également « / » à la fin de l’URL de l’émetteur.

    ```xml
    <add key="ida:audience" value="[Application Client ID GUID]" />
    <add key="ida:issuer" value="https://sts.windows.net/[AAD Tenant ID]/" />
    ```

    Dans le [Décodeur JWT](http://jwt.calebb.net/), **aud** et **iss** apparaissent comme suit dans le jeton JWT :

    ![JWT](./media/design-multi-drm-system-with-access-control/media-services-1st-gotcha.png)

* Ajoutez des autorisations à l’application dans Azure AD sur l’onglet **Configuration** de l’application. Des autorisations sont demandées pour chaque application, aussi bien pour les versions locales que pour les versions déployées.

    ![Autorisations](./media/design-multi-drm-system-with-access-control/media-services-perms-to-other-apps.png)

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

    `objTokenRestrictionTemplate.TokenType = TokenType.JWT;`

    Puisque vous ajoutez la prise en charge de JWT (Azure AD) en plus des SWT (ACS), la valeur par défaut de TokenType est TokenType.JWT. Si vous utilisez SWT/ACS, vous devez définir le jeton sur TokenType.SWT.

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

![Compte de domaine client Azure AD personnalisé 1](./media/design-multi-drm-system-with-access-control/media-services-ad-tenant-domain1.png)

**Compte de domaine Microsoft avec carte à puce** : page de connexion personnalisée par l’informatique d’entreprise Microsoft avec authentification à deux facteurs.

![Compte de domaine client Azure AD personnalisé 2](./media/design-multi-drm-system-with-access-control/media-services-ad-tenant-domain2.png)

**Compte Microsoft** : page de connexion du compte Microsoft pour les consommateurs.

![Compte de domaine client Azure AD personnalisé 3](./media/design-multi-drm-system-with-access-control/media-services-ad-tenant-domain3.png)

### <a name="use-encrypted-media-extensions-for-playready"></a>Utilisation d’Encrypted Media Extensions pour PlayReady

Sur un navigateur moderne prenant en charge Encrypted Media Extensions (EME) pour PlayReady tel qu’Internet Explorer 11 sous Windows 8.1 et version ultérieure et le navigateur Microsoft Edge sous Windows 10, PlayReady sera la DRM sous-jacente d’EME.

![Utilisation d’EME pour PlayReady](./media/design-multi-drm-system-with-access-control/media-services-eme-for-playready1.png)

La zone de lecteur foncée est due au fait que la protection PlayReady empêche d’effectuer une capture d’écran de la vidéo protégée.

La capture d’écran suivante montre les plug-ins de lecteur et les composants Microsoft Security Essentials (MSE)/EME pris en charge :

![Plug-ins de lecteur pour PlayReady](./media/design-multi-drm-system-with-access-control/media-services-eme-for-playready2.png)

EME dans Microsoft Edge et Internet Explorer 11 sur Windows 10 permet d’appeler [PlayReady SL3000](https://www.microsoft.com/playready/features/EnhancedContentProtection.aspx/) sur les appareils Windows 10 compatibles. PlayReady SL3000 déverrouille le flux de contenu premium améliorées (4K, HDR) et les nouveaux modèles de distribution de contenu (pour le contenu amélioré).

Pour vous concentrer uniquement sur les appareils Windows, PlayReady est le seul système DRM dans le matériel disponible sur les appareils Windows (PlayReady SL3000). Un service de diffusion en continu peut utiliser PlayReady via EME ou via une application UWP (Universal Windows Platform), et offrir ainsi une meilleure qualité vidéo à l’aide de PlayReady SL3000 par rapport à un autre DRM. En règle générale, l’utilisation de Chrome ou de Firefox permet de diffuser du contenu jusqu’à 2K, tandis que Microsoft Edge/Internet Explorer 11 ou une application UWP prend en charge du contenu jusqu’à 4K sur le même appareil. La quantité dépend des paramètres de service et de l’implémentation.

#### <a name="use-eme-for-widevine"></a>Utilisation d’EME pour Widevine

Sur un navigateur moderne doté de la prise en charge d’EME/Widevine, telle que Chrome 41 + sous Windows 10, Windows 8.1, Mac OSX Yosemite et Chrome sur Android 4.4.4, la gestion des droits numériques derrière EME est assurée par Google Widevine.

![Utilisation d’EME pour Widevine](./media/design-multi-drm-system-with-access-control/media-services-eme-for-widevine1.png)

Widevine ne vous empêche pas d’effectuer une capture d’écran de la vidéo protégée.

![Plug-ins de lecteur pour Widevine](./media/design-multi-drm-system-with-access-control/media-services-eme-for-widevine2.png)

#### <a name="use-eme-for-fairplay"></a>Utilisation d’EME pour FairPlay

De même, vous pouvez tester le contenu protégé par FairPlay dans ce lecteur de test sur Safari sous MacOS ou iOS 11.2 et versions ultérieures.

Veillez à indiquer « FairPlay » comme protectionInfo.type et entrez l’URL appropriée pour le certificat de votre application dans le chemin FPS AC (certificat d’application de diffusion en continu FairPlay).

### <a name="unentitled-users"></a>Utilisateurs non habilités

Si un utilisateur n’est pas un membre du groupe d’utilisateurs habilités, l’utilisateur échoue à la vérification des droits. Le service de licence multi-DRM refuse alors d’émettre la licence requise comme indiqué. La description détaillée est « L’acquisition de licence a échoué », ce qui correspond à la conception.

![Utilisateurs non habilités](./media/design-multi-drm-system-with-access-control/media-services-unentitledusers.png)

### <a name="run-a-custom-security-token-service"></a>Exécution d’un service d’émission de jeton de sécurité (STS) personnalisé

Si vous exécutez un STS personnalisé, le jeton JWT est émis par le STS personnalisé à l’aide d’une clé symétrique ou asymétrique.

La capture d’écran suivante montre un scénario qui utilise une clé symétrique (à l’aide de Chrome) :

![STS personnalisé avec clé symétrique](./media/design-multi-drm-system-with-access-control/media-services-running-sts1.png)

La capture d’écran suivante montre un scénario qui utilise une clé asymétrique via un certificat X509 (à l’aide d’un navigateur Microsoft moderne) :

![STS personnalisé avec clé asymétrique](./media/design-multi-drm-system-with-access-control/media-services-running-sts2.png)

Dans les deux cas ci-dessus, l’authentification utilisateur reste la même. Elle passe par Azure AD. La seule différence est que les jetons JWT sont émis par le STS personnalisé et non par Azure AD. Lorsque vous configurez la protection CENC dynamique, la restriction du service de distribution de licences spécifie le type de jeton JWT, soit avec une clé symétrique soit avec une clé asymétrique.

## <a name="next-steps"></a>Étapes suivantes

* [Forum Aux Questions (FAQ)](frequently-asked-questions.md)
* [Présentation de la protection du contenu](content-protection-overview.md)
* [Protéger votre contenu avec DRM](protect-with-drm.md)
