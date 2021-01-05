---
title: Protéger votre contenu avec Azure Media Services | Microsoft Docs
description: Cet article donne une vue d’ensemble de la protection du contenu avec Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 81bc00e1-dcda-4d69-b9ab-8768b793422b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 08ebda8bdd17cc4d620792934e1299365d0ced71
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89257923"
---
# <a name="content-protection-overview"></a>Présentation de la protection du contenu

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)] 

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Vous pouvez utiliser Azure Media Services pour sécuriser votre contenu multimédia du moment où il quitte votre ordinateur jusqu’à la remise, en passant par le stockage et le traitement. Media Services vous permet de transmettre votre contenu dynamique ou à la demande chiffré dynamiquement avec la norme Advanced Encryption Standard (AES-128) ou un des principaux systèmes de gestion des droits numériques (DRM) : Microsoft PlayReady, Google Widevine et Apple FairPlay. Media Services fournit également un service de distribution de clés AES et licences (PlayReady, Widevine et FairPlay) DRM aux clients autorisés. 

L’image suivante illustre le flux de travail de protection du contenu Media Services : 

![Protéger avec PlayReady](./media/media-services-content-protection-overview/media-services-content-protection-with-multi-drm.png)

Cet article explique les concepts et la terminologie pertinents pour comprendre la protection du contenu avec Media Services. Il fournit également des liens aux articles qui abordent la protection du contenu. 

## <a name="dynamic-encryption"></a>Chiffrement dynamique

Vous pouvez utiliser Media Services pour transmettre du contenu chiffré de manière dynamique avec le chiffrement à clé en clair AES ou DRM en utilisant PlayReady, Widevine ou Apple FairPlay. Si le contenu est chiffré avec une clé en clair AES et est envoyé via HTTPS, il ne s’affiche en clair qu’une fois qu’il atteint le client. 

Chaque méthode de chiffrement prend en charge les protocoles de diffusion en continu suivants :
 
- AES : MPEG-DASH, Smooth Streaming et HLS
- PlayReady : MPEG-DASH, Smooth Streaming et HLS
- Widevine : MPEG-DASH
- FairPlay : HLS

Le chiffrement sur des téléchargements progressifs n’est pas pris en charge. 

Pour chiffrer un élément, vous devez associer une clé de contenu de chiffrement à votre élément et configurer également une stratégie d'autorisation pour la clé. Les clés de chiffrement peuvent être spécifiées ou générées automatiquement par Media Services.

Vous devez également configurer la stratégie de remise de l’élément multimédia. Si vous souhaitez diffuser en continu un élément avec chiffrement de stockage, assurez-vous de spécifier comment vous souhaitez le remettre en configurant la stratégie de remise d’éléments.

Lorsqu’un flux est demandé par un lecteur, Media Services utilise la clé spécifiée pour chiffrer dynamiquement votre contenu à l’aide du chiffrement de clé en clair AES ou DRM. Pour déchiffrer le flux, le lecteur demande la clé au service de remise des clés Media Services. Pour déterminer si l’utilisateur est autorisé à obtenir la clé, le service évalue les stratégies d’autorisation que vous avez spécifiées pour la clé.

## <a name="aes-128-clear-key-vs-drm"></a>Clé en clair AES-128 et DRM
Les clients se demandent souvent s’ils devraient utiliser un chiffrement AES ou un système DRM. La principale différence entre les deux systèmes est qu’avec le chiffrement AES, la clé de contenu est transmise au client dans un format non chiffré (« en clair »). Par conséquent, la clé utilisée pour chiffrer le contenu peut être affichée dans la trace réseau du client en texte en clair. Le chiffrement avec une clé en clair AES-128 convient à des cas d’utilisation dans lesquels l’observateur est fiable (par exemple, le chiffrement de vidéos d’entreprise distribuées dans une société et destinées aux employés).

PlayReady, Widevine et FairPlay offrent tous un haut niveau de chiffrement comparé au chiffrement à clé en clair AES-128. La clé de contenu est transmise dans un format chiffré. De plus, le déchiffrement est pris en charge dans un environnement sécurisé au niveau du système d’exploitation, où il est plus difficile pour un utilisateur malintentionné de commettre une attaque. DRM est recommandé dans des cas d’utilisation où l’observateur peut ne pas être fiable et où donc un niveau de sécurité supérieur est requis.

## <a name="storage-encryption"></a>Chiffrement du stockage
Vous pouvez utiliser le chiffrement de stockage pour chiffrer votre contenu en clair en local en utilisant le chiffrement AES 256 bits. Vous pouvez ensuite le télécharger vers le stockage Azure, où il est stocké à l’état chiffré au repos. Les éléments multimédias protégés par le chiffrement de stockage sont automatiquement déchiffrés et placés dans un système de fichiers chiffré avant d’être encodés. Les éléments multimédias sont éventuellement rechiffrés avant d’être rechargés sous la forme d’un nouvel élément multimédia de sortie. Le principal cas d'utilisation du chiffrement de stockage concerne la sécurisation de fichiers multimédias d'entrée de haute qualité avec un chiffrement renforcé au repos sur le disque.

Pour fournir un élément multimédia avec chiffrement de stockage, vous devez configurer la stratégie de remise de l'élément multimédia afin que Media Services sache comment vous souhaitez remettre votre contenu. Pour que votre élément puisse être diffusé en continu, le serveur de diffusion en continu supprime le chiffrement et diffuse en continu votre contenu à l’aide de la stratégie de remise spécifiée (par exemple AES, chiffrement commun ou aucun chiffrement).

## <a name="types-of-encryption"></a>Types de chiffrement
PlayReady et Widevine utilisent le chiffrement commun (mode CTR AES). FairPlay utilise le chiffrement en mode CBS AES. Le chiffrement à clé en clair AES-128 utilise le chiffrement d’enveloppe.

## <a name="licenses-and-keys-delivery-service"></a>Service de remise de licences et de clés
Media Services fournit un service de remise de licences DRM (PlayReady, Widevine, FairPlay) et de clés AES aux clients autorisés. Vous pouvez utiliser le [portail Azure](media-services-portal-protect-content.md), l’API REST ou le kit de développement logiciel (SDK) Media Services pour .NET pour configurer des stratégies d’authentification et d’autorisation pour vos licences et vos clés.

## <a name="control-content-access"></a>Contrôle de l’accès au contenu
Vous pouvez contrôler qui a accès à votre contenu en configurant la stratégie d’autorisation de clé de contenu. La stratégie d’autorisation de clé de contenu prend en charge la restriction ouverte ou par jeton.

### <a name="open-authorization"></a>Autorisation ouverte
Avec une stratégie d’autorisation ouverte, la clé de contenu est envoyée à un client, quel qu’il soit (sans restriction).

### <a name="token-authorization"></a>Autorisation de jeton
Avec une stratégie d’autorisation de jeton, la clé de contenu n’est envoyée qu’à un client qui présente un JSON Web Token (JWT) ou Simple Web Token (SWT) valide dans la requête de clé/licence. Ce jeton doit être émis par un service d’émission de jeton de sécurité (STS). Vous pouvez utiliser Azure Active Directory en tant que STS ou déployer un STS personnalisé. Le STS doit être configuré pour créer un jeton signé avec la clé spécifiée et émettre les revendications spécifiées dans la configuration de restriction de jeton. Le service de remise de clé Media Services retourne la clé/licence demandée au client si le jeton est valide et que les revendications du jeton correspondent à celles configurées pour la clé/licence.

Quand vous configurez la stratégie de restriction par jeton, vous devez définir les paramètres de clé de vérification, émetteur et audience principaux. La clé de vérification principale contient le jeton avec lequel la clé a été signée. L’émetteur est le service STS qui émet le jeton. L’audience, parfois appelé étendue, décrit l’objectif du jeton ou la ressource à laquelle le jeton autorise l’accès. Le service de remise de clé Media Services valide le fait que les valeurs du jeton correspondent aux valeurs du modèle.

### <a name="token-replay-prevention"></a>Prévention de réexécution de jeton

La fonctionnalité de *prévention de relecture de jeton* permet aux clients de Media Services de limiter le nombre de fois qu’un même jeton peut être utilisé pour demander une clé ou une licence. Le client peut ajouter une revendication de type `urn:microsoft:azure:mediaservices:maxuses` dans le jeton, sont la valeur indique le nombre de fois que le jeton peut être utilisé pour acquérir une licence ou une clé. Toutes les demandes suivantes de remise de clé effectuées à l’aide du même jeton renvoient une réponse Non autorisé. Découvrez comment ajouter la revendication dans l’[exemple DRM](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L601).
 
#### <a name="considerations"></a>Considérations

* Les clients doivent pouvoir contrôler la génération de jetons. La revendication doit être placée dans le jeton lui-même.
* Lors de l’utilisation de cette fonctionnalité, les demandes comportant des jetons dont le délai d’expiration est supérieur à une heure après le moment de réception de la demande sont rejetées avec une réponse Non autorisé.
* Les jetons sont identifiés de façon unique par leur signature. Toute modification apportée à la charge utile (par exemple, une mise à jour de l’heure d’expiration ou de la revendication) a pour effet de modifier la signature du jeton et est considérée comme un nouveau jeton que la fonctionnalité de remise de clé n’a pas encore rencontré.
* La lecture échoue si le jeton a été exécuté un nombre fois supérieur à la valeur `maxuses` définie par le client.
* Cette fonctionnalité peut être utilisée pour tout contenu protégé existant (seul le jeton émis doit être modifié).
* Cette fonctionnalité opère avec JWT et SWT.

## <a name="streaming-urls"></a>URL de diffusion
Si votre ressource a été chiffrée avec plusieurs DRM, utilisez une balise de chiffrement dans l’URL de streaming : (format=’m3u8-aapl’, encryption=’xxx’).

Les considérations suivantes s'appliquent :

* Pas plus d’un seul type de chiffrement peut être spécifié.
* Le type de chiffrement ne doit pas être spécifié dans l’URL si un seul chiffrement a été appliqué à la ressource.
* Le type de chiffrement ne tient pas compte de la casse.
* Les types de chiffrement suivants peuvent être spécifiés :

  * **cenc** : pour PlayReady ou Widevine (chiffrement commun)
  * **cbcs-aapl** : pour FairPlay (chiffrement CBC AES)
  * **cbc** : chiffrement de l'enveloppe AES

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="next-steps"></a>Étapes suivantes
Les articles ci-dessous décrivent les étapes suivantes que vous pouvez exécuter pour mieux comprendre la protection du contenu :

* [Protection avec le chiffrement du stockage](media-services-rest-storage-encryption.md)
* [Protection avec le chiffrement AES](media-services-protect-with-aes128.md)
* [Protection avec PlayReady et/ou Widevine](media-services-protect-with-playready-widevine.md)
* [Protection avec FairPlay](media-services-protect-hls-with-FairPlay.md)

## <a name="related-links"></a>Liens connexes

* [Authentification par jeton JWT](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/)
* [Intégration d'une application Azure Media Services basée sur OWIN MVC avec Azure Active Directory et remise de clés de contenu basée sur les revendications JWT](http://www.gtrifonov.com/2015/01/24/mvc-owin-azure-media-services-ad-integration/)

[content-protection]: ./media/media-services-content-protection-overview/media-services-content-protection.png
