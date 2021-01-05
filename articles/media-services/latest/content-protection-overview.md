---
title: Protéger votre contenu à l’aide du chiffrement dynamique Media Services v3
titleSuffix: Azure Media Services
description: En savoir plus sur la protection du contenu avec le chiffrement dynamique, les protocoles de diffusion en continu et les types de chiffrement dans Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 5d6530cf7b8d8611ff23a3517112cb0aa7442d6d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95971166"
---
# <a name="protect-your-content-with-media-services-dynamic-encryption"></a>Protéger votre contenu à l’aide du chiffrement dynamique de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Utilisez Azure Media Services pour sécuriser votre contenu multimédia du moment où il quitte votre ordinateur jusqu’à sa remise, en passant par le stockage et le traitement. Media Services vous permet de transmettre votre contenu dynamique ou à la demande chiffré dynamiquement avec la norme Advanced Encryption Standard (AES-128) ou un des principaux systèmes de gestion des droits numériques (DRM) : Microsoft PlayReady, Google Widevine et Apple FairPlay. Media Services fournit également un service de distribution de clés AES et licences (PlayReady, Widevine et FairPlay) DRM aux clients autorisés. Si le contenu est chiffré avec une clé en clair AES et est envoyé via HTTPS, il ne s’affiche en clair qu’une fois qu’il atteint le client.

[!INCLUDE [Widevine is not available in the GovCloud region.](./includes/widevine-not-available-govcloud.md)]

Dans Media Services v3, une clé de contenu est associée à un localisateur de streaming (consultez [cet exemple](protect-with-aes128.md)). Si vous utilisez le service de remise de clé Azure Media Services, vous pouvez demander à ce service de générer la clé de contenu pour vous. Vous devez générer la clé de contenu vous-même si vous utilisez votre propre service de remise de clé, ou si vous avez besoin de gérer un scénario de haute disponibilité au cours duquel vous avez besoin d’avoir la même clé de contenu dans deux centres de données.

Lorsqu’un flux est demandé par un lecteur, Media Services utilise la clé spécifiée pour chiffrer dynamiquement votre contenu à l’aide du chiffrement de clé en clair AES ou DRM. Pour déchiffrer le flux, le lecteur demande la clé au service de remise des clés Media Services, ou au service de remise des clés que vous avez spécifié. Pour déterminer si l’utilisateur est autorisé ou non à obtenir la clé, le service évalue la stratégie de clé de contenu que vous avez spécifiée pour la clé.

Vous pouvez utiliser l’API REST ou le la bibliothèque de client Media Services pour configurer des stratégies d’authentification et d’autorisation pour vos licences et vos clés.

L’image suivante illustre le flux de travail de protection du contenu de Media Services :

![Flux de travail pour la protection du contenu de Media Services](./media/content-protection/content-protection.svg)
  
&#42; *Le chiffrement dynamique prend en charge les standards AES-128 « clé en clair », CBCS et CENC. Pour plus d’informations, voir la [matrice de prise en charge](#streaming-protocols-and-encryption-types).*

Cet article explique les concepts et la terminologie pertinents pour comprendre la protection du contenu avec Media Services.

## <a name="main-components-of-a-content-protection-system"></a>Principaux composants du système de protection du contenu

Pour implémenter correctement votre système de protection de contenu, vous devez parfaitement comprendre l’étendue de l’effort. Les sections suivantes fournissent un aperçu des trois éléments que vous devez implémenter.

> [!NOTE]
> Nous vous recommandons vivement de vous concentrer et de tester entièrement chaque élément des sections suivantes avant de passer à l’élément suivant. Pour tester votre système de protection du contenu, utilisez les outils spécifiés dans les sections.

### <a name="media-services-code"></a>Code Media Services
  
L’[exemple DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs) montre comment implémenter un système multi-DRM avec Media Services v3 en utilisant .NET. Il montre également comment utiliser le service de remise de clé ou de licences de Media Services.
  
Vous pouvez chiffrer chaque ressource avec plusieurs types de chiffrement (AES-128, PlayReady, Widevine, FairPlay). Pour identifier la meilleure combinaison, voir [Types de protocoles de diffusion en continu et de chiffrements](#streaming-protocols-and-encryption-types).

Cet exemple indique comment effectuer les opérations suivantes :

1. Créer et configurer une [stratégie de clé de contenu](content-key-policy-concept.md).

   Vous créez une stratégie de clé de contenu pour configurer le mode de remise de clé de contenu (qui fournit un accès sécurisé à vos ressources) aux clients finaux :  

   * Définissez une autorisation de livraison de licence. Spécifiez la logique du contrôle d’autorisation en fonction des revendications figurant dans le JSON Web Token (JWT).
   * Configurez les licences associées à [PlayReady](playready-license-template-overview.md), [Widevine](widevine-license-template-overview.md) et/ou [FairPlay](fairplay-license-overview.md). Les modèles vous permettent de configurer les droits et autorisations pour chacun des DRM.

     ```
     ContentKeyPolicyPlayReadyConfiguration playReadyConfig = ConfigurePlayReadyLicenseTemplate();
     ContentKeyPolicyWidevineConfiguration widevineConfig = ConfigureWidevineLicenseTempate();
     ContentKeyPolicyFairPlayConfiguration fairPlayConfig = ConfigureFairPlayPolicyOptions();
     ```

2. Créez un [localisateur de streaming](streaming-locators-concept.md) configuré pour diffuser en continu une ressource chiffrée.
  
   Le localisateur de streaming doit être associé à une [stratégie de diffusion en continu](streaming-policy-concept.md). Dans cet exemple, nous définissons `StreamingLocator.StreamingPolicyName` sur la stratégie « Predefined_MultiDrmCencStreaming ».

   Les chiffrements PlayReady et Widevine sont appliqués, et la clé est remise au client de lecture en fonction des licences DRM configurées. Si vous voulez aussi chiffrer votre flux avec CBCS (FairPlay), utilisez la stratégie « Predefined_MultiDrmStreaming ».

   Le localisateur de streaming est également associé à la stratégie de clé de contenu que vous avez définie.

3. Créer un jeton de test.

   La méthode `GetTokenAsync` indique comment créer un jeton de test.
4. Générer l’URL de diffusion en continu.

   La méthode `GetDASHStreamingUrlAsync` indique comment générer l’URL de diffusion en continu. En l’occurrence, l’URL diffuse en continu le contenu DASH.

### <a name="player-with-an-aes-or-drm-client"></a>Lecteur avec client AES ou DRM

Une application de lecteur vidéo basée sur un kit de développement logiciel du lecteur (natif ou dans le navigateur) doit remplir les conditions suivantes :

* Le Kit de développement logiciel (SDK) du lecteur prend en charge les clients DRM nécessaires.
* Le Kit de développement logiciel du lecteur prend en charge les protocoles de diffusion en continu requis : Smooth, DASH et/ou HTTP Live Streaming (HLS).
* Le Kit de développement logiciel (SDK) du lecteur peut gérer la transmission d’un jeton JWT dans une demande d’acquisition de licence.

Vous pouvez créer un lecteur à l’aide de [l’API Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/). Utilisez [l’API ProtectionInfo d’Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/) pour spécifier la technologie DRM à utiliser sur d’autres plateformes DRM.

Pour tester du contenu chiffré par AES ou CENC (Widevine et/ou PlayReady), vous pouvez utiliser le [lecteur multimédia Azure](https://aka.ms/azuremediaplayer). Veillez à sélectionner **Options avancées** et vérifiez vos options de chiffrement.

Si vous souhaitez tester le contenu chiffré de FairPlay, utilisez [ce lecteur test](https://aka.ms/amtest). Le lecteur prend en charge les DRM Widevine, PlayReady et FairPlay, ainsi que le chiffrement « clé en clair » AES-128.

Choisissez le navigateur approprié pour tester les différents systèmes DRM :

* Chrome, Opera ou Firefox pour Widevine.
* Microsoft Edge ou Internet Explorer 11 pour PlayReady.
* Safari sur macOS pour FairPlay.

### <a name="security-token-service"></a>Service d’émission de jeton de sécurité

Un service d’émission de jeton de sécurité (STS) émet un JWT en tant que jeton d’accès pour l’accès aux ressources principales. Vous pouvez utiliser le service de remise de clé ou de licence Azure Media Services en tant que ressource principale. Un service STS doit définir les éléments suivants :

* L’émetteur et l’audience (ou l’étendue).
* Les revendications, qui dépendent des exigences de l’activité concernant la protection du contenu.
* La vérification symétrique ou asymétrique pour vérifier la signature.
* La prise en charge de substitution de clé (si nécessaire).

Vous pouvez vous servir de cet [outil STS](https://openidconnectweb.azurewebsites.net/DRMTool/Jwt) pour tester le service d’émission de jeton de sécurité. Il prend en charge les trois types de clés de vérification : symétrique, asymétrique ou Azure Active Directory (Azure AD) avec substitution de clé.

## <a name="streaming-protocols-and-encryption-types"></a>Types de protocoles de diffusion en continu et de chiffrement

Vous pouvez utiliser Media Services pour transmettre du contenu chiffré de manière dynamique avec le chiffrement à clé en clair AES ou DRM en utilisant PlayReady, Widevine ou Apple FairPlay. Actuellement, vous pouvez chiffrer les formats HLS, MPEG-DASH et Smooth Streaming. Chaque protocole prend en charge les méthodes de chiffrement suivantes.

### <a name="hls"></a>HLS

Le protocole HLS prend en charge les formats de conteneur et schémas de chiffrement ci-après :

|Format de conteneur|Schéma de chiffrement|Exemple d’URL|
|---|---|---|
|Tous|AES|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-aapl,encryption=cbc)`|
|MPG2-TS |CBCS (FairPlay) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-aapl,encryption=cbcs-aapl)`|
|CMAF(fmp4) |CBCS (FairPlay) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-cmaf,encryption=cbcs-aapl)`|
|MPG2-TS |CENC (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-aapl,encryption=cenc)`|
|CMAF(fmp4) |CENC (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=m3u8-cmaf,encryption=cenc)`|

Le chiffrement HLS/CMAF + FairPlay (y compris HEVC/H.265) est pris en charge sur les appareils suivants :

* iOS 11 ou version ultérieure.
* iPhone 8 ou version ultérieure.
* macOS High Sierra avec un processeur Intel de 7e génération.

### <a name="mpeg-dash"></a>MPEG-DASH

Le protocole MPEG-DASH prend en charge les formats de conteneur et schémas de chiffrement ci-après :

|Format de conteneur|Schéma de chiffrement|Exemples d’URL
|---|---|---|
|Tous|AES|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=mpd-time-csf,encryption=cbc)`|
|CSF(fmp4) |CENC (Widevine + PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=mpd-time-csf,encryption=cenc)`|
|CMAF(fmp4)|CENC (Widevine + PlayReady)|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(format=mpd-time-cmaf,encryption=cenc)`|

### <a name="smooth-streaming"></a>Smooth Streaming

Le protocole Smooth Streaming prend en charge les formats de conteneur et schémas de chiffrement ci-après.

|Protocol|Format de conteneur|Schéma de chiffrement|
|---|---|---|
|fMP4|AES|`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=cbc)`|
|fMP4 | CENC (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=cenc)`|
|fMP4 | PIFF 1.1 (PlayReady) |`https://amsv3account-usw22.streaming.media.azure.net/00000000-0000-0000-0000-000000000000/ignite.ism/manifest(encryption=piff)`|

> [!NOTE]
> La prise en charge de PIFF 1.1 est fournie en tant que solution de compatibilité descendante pour les Smart TV (Samsung, LG) qui implémentaient la version « Silverlight » précoce de la norme CENC. Nous vous recommandons d’utiliser le format PIFF uniquement là où il est nécessaire pour la prise en charge des Smart TV Samsung ou LG livrés entre 2009 et 2015 qui prenaient en charge la version PIFF 1.1 du chiffrement PlayReady. 

### <a name="browsers"></a>Navigateurs

Les navigateurs courants prennent en charge les clients DRM suivants :

|Browser|Chiffrement|
|---|---|
|Chrome|Widevine|
|Microsoft Edge, Internet Explorer 11|PlayReady|
|Firefox|Widevine|
|Opera|Widevine|
|Safari|FairPlay|

## <a name="controlling-content-access"></a>Contrôle de l’accès au contenu

Vous pouvez contrôler qui a accès à votre contenu en configurant la stratégie d’autorisation de clé de contenu. Media Services prend en charge plusieurs méthodes d’autorisation des utilisateurs effectuant des demandes de clé. Le client (lecteur) doit répondre à la stratégie avant que la clé de contenu ne puisse lui être remise. La stratégie de clé de contenu peut contenir une restriction *ouverte* ou *de jeton*.

Vous pouvez utiliser une stratégie de clé de contenu limitée à l’ouverture si vous souhaitez attribuer une licence à une personne dépourvue d’autorisation. Par exemple, si vos recettes sont basées sur la publicité plutôt que sur des abonnements.  

Avec une stratégie de clé de contenu limitée au jeton, la clé de contenu n’est envoyée qu’à un client qui présente un JSON Web Token ou un simple jeton web (SWT) valides dans la demande de clé ou de licence. Ce jeton doit être émis par un service d’émission de jeton de sécurité (STS).

Vous pouvez utiliser Azure AD en tant que STS ou déployer un [STS personnalisé](#using-a-custom-sts). Le STS doit être configuré pour créer un jeton signé avec la clé spécifiée et émettre les revendications spécifiées dans la configuration de restriction de jeton. Le service de remise de clé ou de licence Media Services retourne la licence ou la clé demandées au client si les deux conditions suivantes sont réunies :

* Le jeton est valide.
* Les revendications dans le jeton correspondent à celles configurées pour la licence ou la clé.

Quand vous configurez la stratégie de restriction par jeton, vous devez définir les paramètres de clé de vérification, émetteur et audience principaux. La clé de vérification principale contient le jeton avec lequel la clé a été signée. L’émetteur est le service STS qui émet le jeton. L’audience, parfois appelé étendue, décrit l’objectif du jeton ou la ressource à laquelle le jeton autorise l’accès. Le service de remise de clé ou de licence Media Services vérifie que les valeurs dans le jeton correspondent aux valeurs du modèle.

### <a name="token-replay-prevention"></a>Prévention de réexécution de jeton

La fonctionnalité de *prévention de relecture de jeton* permet aux clients de Media Services de limiter le nombre de fois qu’un même jeton peut être utilisé pour demander une clé ou une licence. Le client peut ajouter une revendication de type `urn:microsoft:azure:mediaservices:maxuses` dans le jeton, sont la valeur indique le nombre de fois que le jeton peut être utilisé pour acquérir une licence ou une clé. Toutes les demandes suivantes de remise de clé effectuées à l’aide du même jeton renvoient une réponse Non autorisé. Découvrez comment ajouter la revendication dans l’[exemple DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L601).
 
#### <a name="considerations"></a>Considérations

* Les clients doivent pouvoir contrôler la génération de jetons. La revendication doit être placée dans le jeton lui-même.
* Lors de l’utilisation de cette fonctionnalité, les demandes comportant des jetons dont le délai d’expiration est supérieur à une heure après le moment de réception de la demande sont rejetées avec une réponse Non autorisé.
* Les jetons sont identifiés de façon unique par leur signature. Toute modification apportée à la charge utile (par exemple, une mise à jour de l’heure d’expiration ou de la revendication) a pour effet de modifier la signature du jeton et est considérée comme un nouveau jeton que la fonctionnalité de remise de clé n’a pas encore rencontré.
* La lecture échoue si le jeton a été exécuté un nombre fois supérieur à la valeur `maxuses` définie par le client.
* Cette fonctionnalité peut être utilisée pour tout contenu protégé existant (seul le jeton émis doit être modifié).
* Cette fonctionnalité opère avec JWT et SWT.

## <a name="using-a-custom-sts"></a>Utilisation d’un STS personnalisé

Un client peut choisir d’utiliser un STS personnalisé pour fournir des jetons. En voici plusieurs raisons :

* Le fournisseur d’identité (IDP) utilisé par le client ne prend pas en charge STS. Dans ce cas, un service STS personnalisé peut être une solution.
* Le client peut avoir besoin de contrôler de manière plus souple ou plus stricte l’intégration de STS avec le système de facturation des abonnés du client.

   Par exemple, un service [OTT](https://en.wikipedia.org/wiki/Over-the-top_media_services) peut proposer plusieurs offres d’abonné, par exemple des offres premium, de base et sports. L’opérateur peut chercher à associer les revendications d’un jeton avec l’offre d’un abonné afin que seul le contenu d’une offre spécifique soit mis à disposition. Dans ce cas, un STS personnalisé offre la souplesse et la maîtrise nécessaires.

* Pour inclure des revendications personnalisées dans le jeton afin de pouvoir opérer une sélection parmi différentes ContentKeyPolicyOptions avec différents paramètres de licence DRM (licence d’abonnement ou licence de location).
* Pour inclure une revendication représentant l’identificateur de clé de contenu de la clé à laquelle le jeton accorde l’accès.

Lorsque vous utilisez un STS personnalisé, vous devez effectuer deux modifications :

* Lorsque vous configurez le service de distribution de licences pour un élément multimédia, vous devez spécifier la clé de sécurité utilisée par le STS personnalisé, et non la clé actuelle d’Azure AD.
* Une fois le jeton JTW généré, une clé de sécurité est spécifiée à la place de la clé privée du certificat X509 courant dans Azure AD.

Il existe deux types de clés de sécurité :

* Clé symétrique : la même clé est utilisée pour générer et vérifier un jeton JWT.
* Clé asymétrique : une paire de clés publique-privée dans un certificat X509 est utilisée avec une clé privée pour chiffrer/générer un jeton JWT et avec la clé publique pour vérifier le jeton.

Si vous utilisez .NET Framework/C# en tant que plateforme de développement, le certificat X509 utilisé pour la clé de sécurité asymétrique doit avoir une longueur d’au moins 2 048 bits. La longueur de clé constitue une exigence de la classe System.IdentityModel.Tokens.X509AsymmetricSecurityKey dans .NET Framework. Dans le cas contraire, l’exception suivante est générée : IDX10630 : La longueur de la signature « System.IdentityModel.Tokens.X509AsymmetricSecurityKey » ne peut pas être inférieure à « 2048 » bits.

## <a name="custom-key-and-license-acquisition-url"></a>URL d’acquisition de licences et de clés personnalisées

Utilisez les modèles suivants si vous souhaitez spécifier un service de remise de clé ou de licence autre que Media Services. Les deux champs modifiables présents dans les modèles sont là pour vous permettre de partager votre stratégie de diffusion en continu entre plusieurs ressources afin d’éviter de créer une stratégie par ressource. 

* `EnvelopeEncryption.CustomKeyAcquisitionUrlTemplate`: Modèle pour l’URL du service personnalisé qui remet les clés aux lecteurs de l’utilisateur final. Il n’est pas requis lorsque vous utilisez Azure Media Services pour émettre des clés. 

   Ce modèle prend en charge les jetons remplaçables que le service met à jour lors de l’exécution, en utilisant la valeur spécifique à la requête.  Les valeurs de jeton actuellement prises en charge sont les suivantes :
   * `{AlternativeMediaId}` qui est remplacée par la valeur de StreamingLocatorId.AlternativeMediaId.
   * `{ContentKeyId}` qui est remplacée par la valeur de l’identificateur de la clé demandée.
* `StreamingPolicyPlayReadyConfiguration.CustomLicenseAcquisitionUrlTemplate`: Modèle pour l’URL du service personnalisé qui remet les licences aux lecteurs de l’utilisateur final. Il n’est pas requis lorsque vous utilisez Azure Media Services pour émettre des licences.

   Ce modèle prend en charge les jetons remplaçables que le service met à jour lors de l’exécution, en utilisant la valeur spécifique à la requête. Les valeurs de jeton actuellement prises en charge sont les suivantes :  
   * `{AlternativeMediaId}` qui est remplacée par la valeur de StreamingLocatorId.AlternativeMediaId.
   * `{ContentKeyId}` qui est remplacée par la valeur de l’identificateur de la clé demandée. 
* `StreamingPolicyWidevineConfiguration.CustomLicenseAcquisitionUrlTemplate`: Identique au modèle précédent, uniquement pour Widevine. 
* `StreamingPolicyFairPlayConfiguration.CustomLicenseAcquisitionUrlTemplate`: Identique au modèle précédent, uniquement pour FairPlay.  

Par exemple :

```csharp
streamingPolicy.EnvelopEncryption.customKeyAcquisitionUrlTemplate = "https://mykeyserver.hostname.com/envelopekey/{AlternativeMediaId}/{ContentKeyId}";
```

`ContentKeyId` a une valeur de la clé demandée. Vous pouvez utiliser `AlternativeMediaId` si vous souhaitez mapper la demande à une entité de votre côté. Par exemple, `AlternativeMediaId` peut être utilisé pour vous aider à rechercher des autorisations.

Pour accéder à des exemples REST qui utilisent des URL d’acquisition de clé ou de licence personnalisées, voir [Stratégies de diffusion en continu – Créer](/rest/api/media/streamingpolicies/create).

> [!NOTE]
> Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="troubleshoot"></a>Dépanner

Si vous obtenez l’erreur `MPE_ENC_ENCRYPTION_NOT_SET_IN_DELIVERY_POLICY`, veillez à spécifier la stratégie de diffusion en continu appropriée.

Si vous obtenez des erreurs qui se terminent par `_NOT_SPECIFIED_IN_URL`, vérifiez que vous avez spécifié le format de chiffrement dans l’URL. par exemple `…/manifest(format=m3u8-cmaf,encryption=cbcs-aapl)`. Voir [Types de protocoles de diffusion en continu et de chiffrement](#streaming-protocols-and-encryption-types).

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

* [Protection avec le chiffrement AES](protect-with-aes128.md)
* [Protéger avec DRM](protect-with-drm.md)
* [Concevoir un système de protection de contenu multi-DRM avec contrôle d’accès](design-multi-drm-system-with-access-control.md)
* [Chiffrement côté stockage](storage-account-concept.md#storage-side-encryption)
* [Forum Aux Questions](frequently-asked-questions.md)
* [JSON Web Token Handler](/dotnet/framework/security/json-web-token-handler)
