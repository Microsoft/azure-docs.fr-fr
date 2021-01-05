---
title: Concepts Azure Media Services | Microsoft Docs
description: Cet article fournit une brève vue d’ensemble des concepts de Microsoft Azure Media Services et des liens vers d’autres articles pour plus d’informations.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.openlocfilehash: 2a251ef426d41413cf81f46ca05b11640f01e87a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91333764"
---
# <a name="azure-media-services-concepts"></a>Concepts Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Cette rubrique fournit une vue d'ensemble des concepts les plus importants liés à Media Services.

## <a name="assets-and-storage"></a><a name="assets"></a>Éléments multimédias et stockage
### <a name="assets"></a>Éléments multimédias
Un [élément multimédia](/rest/api/media/operations/asset) contient des fichiers numériques (y compris vidéo, audio, images, collections de miniatures, légendes et fichiers de sous-titres) et les métadonnées associées à ces fichiers. Une fois les fichiers numériques chargés dans un élément multimédia, ils peuvent être utilisés dans des flux de travail de diffusion et d’encodage Media Services.

Un élément multimédia est mappé à un conteneur d’objets blob dans le compte de stockage Azure et les fichiers de l’élément multimédia sont stockés en tant qu’objets blob de blocs dans ce conteneur. Les objets blob de pages ne sont pas pris en charge par Azure Media Services.

Lors du choix du contenu multimédia à télécharger et à stocker dans un élément multimédia, vous devez prendre en compte les considérations suivantes :

* Un élément multimédia doit contenir une seule et unique instance de contenu multimédia. Par exemple, une seule version d’un épisode d’une série TV, d’un film ou d’une publicité.
* Un élément multimédia ne doit pas contenir plusieurs rendus ou versions d’un fichier audiovisuel. Une tentative d’enregistrement de plusieurs épisodes d’une série TV, plusieurs publicités ou plusieurs angles de caméra à partir d’une production unique dans un élément multimédia constitue un exemple d’utilisation incorrect d’un élément multimédia. Le stockage de plusieurs rendus ou versions d’un fichier audiovisuel dans un élément multimédia peut entraîner des difficultés lors de l’envoi des travaux d’encodage, de la diffusion en continu et de la sécurisation de la remise de l’élément multimédia ultérieurement dans le flux de travail.  

### <a name="asset-file"></a>Fichier d’élément multimédia
Un [AssetFile](/rest/api/media/operations/assetfile) représente un fichier vidéo ou audio réel stocké dans un conteneur d’objets blob. Un fichier d’élément multimédia est toujours associé à un élément multimédia et un élément multimédia peut contenir un ou plusieurs fichiers La tâche de Media Services Encoder échoue si un objet de fichier de ressources n’est pas associé à un fichier numérique dans un conteneur d’objets blob.

L'instance **AssetFile** et le fichier multimédia réel sont deux objets distincts. L’instance AssetFile contient des métadonnées concernant le fichier multimédia, tandis que le fichier multimédia contient le contenu multimédia réel.

Vous ne devez pas essayer de modifier le contenu des conteneurs d’objets blob qui ont été générés par Media Services sans utiliser les API Media Services.

### <a name="asset-encryption-options"></a>Options de chiffrement d’élément multimédia
Selon le type de contenu que vous souhaitez télécharger, stocker et remettre, Media Services met à votre disposition différentes options de chiffrement.

>[!NOTE]
>Aucun chiffrement n’est utilisé. Il s’agit de la valeur par défaut. Quand vous utilisez cette option, votre contenu n'est pas protégé pendant le transit ou le repos dans le stockage.

Si vous prévoyez de remettre du contenu MP4 via un téléchargement progressif, utilisez cette option pour télécharger votre contenu.

**StorageEncrypted** : utilisez cette option pour chiffrer votre contenu localement à l’aide du chiffrement AES 256 bits, puis chargez-le vers Azure Storage où il est stocké au repos sous forme chiffrée. Les éléments multimédias protégés par le chiffrement de stockage sont automatiquement déchiffrés et placés dans un système de fichiers chiffré avant d’être encodés, puis éventuellement rechiffrés avant d’être rechargés sous la forme d’un nouvel élément multimédia de sortie. Le principal cas d’utilisation du chiffrement de stockage concerne la sécurisation des fichiers multimédias d’entrée de haute qualité avec un chiffrement renforcé au repos sur le disque. 

Pour fournir un élément multimédia avec chiffrement de stockage, vous devez configurer la stratégie de remise de l'élément multimédia afin que Media Services sache comment vous souhaitez remettre votre contenu. Pour que votre élément multimédia puisse être diffusé en continu, le serveur de diffusion supprime le chiffrement de stockage et diffuse votre contenu à l'aide de la stratégie de remise spécifiée (par exemple AES, PlayReady ou aucun chiffrement). 

**CommonEncryptionProtected** : utilisez cette option si vous souhaitez chiffrer du contenu (ou charger du contenu déjà chiffré) avec Common Encryption ou PlayReady DRM (par exemple Smooth Streaming protégé par PlayReady DRM).

**EnvelopeEncryptionProtected** : utilisez cette option si vous souhaitez protéger du contenu (ou charger du contenu déjà protégé) HTTP Live Streaming (HLS) chiffré avec AES (Advanced Encryption Standard). Si vous téléchargez du contenu HLS déjà chiffré avec AES, il doit avoir été chiffré par Transform Manager.

### <a name="access-policy"></a>Stratégie d’accès
La stratégie d’accès ( [AccessPolicy](/rest/api/media/operations/accesspolicy) ) définit les autorisations (telles que lecture, écriture et énumération) et la durée d’accès à un élément multimédia. En général, vous passez un objet AccessPolicy à un localisateur qui est ensuite utilisé pour accéder aux fichiers contenus dans un élément multimédia.

>[!NOTE]
>Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Vous devez utiliser le même ID de stratégie si vous utilisez toujours les mêmes jours / autorisations d’accès, par exemple, les stratégies pour les localisateurs destinées à demeurer en place pendant une longue période (stratégies sans chargement). Pour plus d’informations, consultez [cette rubrique](media-services-dotnet-manage-entities.md#limit-access-policies) .

### <a name="blob-container"></a>Conteneur d’objets blob
Un conteneur d’objets blob regroupe un ensemble d’objets blob. Les conteneurs d’objets blob sont utilisés dans Media Services comme points de limite de contrôle d’accès et comme localisateurs de signature d’accès partagé (SAS, Shared Access Signature) sur les éléments multimédias. Un compte de stockage Azure peut contenir un nombre illimité de conteneurs d’objets blob. Un conteneur peut stocker un nombre illimité d’objets blob.

>[!NOTE]
> Vous ne devez pas essayer de modifier le contenu des conteneurs d’objets blob qui ont été générés par Media Services sans utiliser les API Media Services.
> 
> 

### <a name="locators"></a><a name="locators"></a>Localisateurs
Les [localisateur](/rest/api/media/operations/locator)s fournissent un point d’entrée pour accéder aux fichiers contenus dans une ressource. Une stratégie d’accès est utilisée pour définir les autorisations et la durée pendant laquelle un client a accès à un élément multimédia donné. Les localisateurs peuvent avoir une relation plusieurs-à-un avec une stratégie d’accès, de telle sorte que différents localisateurs puissent fournir différentes heures de démarrage et différents types de connexions à différents clients tout en utilisant tous les mêmes paramètres d’autorisation et de durée. Toutefois, en raison d’une restriction de stratégie d’accès partagé définie par les services de stockage Azure, vous ne pouvez pas avoir plus de cinq localisateurs uniques associés simultanément à un élément multimédia donné. 

Media Services prend en charge deux types de localisateurs : Les localisateurs OnDemandOrigin, utilisés pour diffuser du contenu multimédia (par exemple, MPEG DASH, HLS ou Smooth Streaming) ou télécharger progressivement du contenu multimédia et des localisateurs d’URL SAS, qui servent à charger ou à télécharger des fichiers multimédias vers/depuis le stockage Azure. 

>[!NOTE]
>L’autorisation de liste (AccessPermissions.List) ne doit pas être utilisée lors de la création d’un localisateur OnDemandOrigin. 

### <a name="storage-account"></a>Compte de stockage
Tous les accès à Azure Storage passent par un compte de stockage. Un compte Media Services peut être associé à un ou plusieurs comptes de stockage. Un compte peut contenir un nombre illimité de conteneurs, tant que la taille totale ne dépasse pas 500 To par compte de stockage.  Media Services fournit des outils de niveau SDK pour vous permettre de gérer plusieurs comptes de stockage et d’équilibrer la charge de distribution de vos éléments multimédia pendant le téléchargement vers ces comptes en fonction de métriques ou de la distribution aléatoire. Pour plus d’informations, consultez la page [Utilisation d’Azure Storage](/previous-versions/azure/dn767951(v=azure.100)). 

## <a name="jobs-and-tasks"></a>Travaux et tâches
Un [travail](/rest/api/media/operations/job) sert généralement à traiter (par exemple à indexer ou encoder) une présentation audio/vidéo. Si vous traitez plusieurs vidéos, créez un travail pour chaque vidéo à encoder.

Un travail contient des métadonnées concernant le traitement à effectuer. Chaque travail contient une ou plusieurs [tâche](/rest/api/media/operations/task)s qui spécifient une tâche de traitement atomique, ses éléments multimédias d’entrée, ses éléments multimédias de sortie, un processeur multimédia et ses paramètres associés. Les tâches d’un travail peuvent être chaînées, c’est-à-dire que la ressource de sortie d’une tâche peut être transmise comme ressource d’entrée à la tâche suivante. De cette façon, un travail peut contenir tout le traitement nécessaire à une présentation multimédia.

## <a name="encoding"></a><a id="encoding"></a>Encodage
Azure Media Services fournit plusieurs options pour l’encodage de fichiers multimédias dans le cloud.

Quand vous commencez à utiliser Media Services, il est important de bien comprendre la différence entre les codecs et les formats de fichiers.
Les codecs sont les logiciels qui implémentent les algorithmes de compression/décompression, tandis que les formats de fichiers sont des conteneurs qui contiennent la vidéo compressée.

Media Services fournit l’empaquetage dynamique qui permet de distribuer un contenu en diffusion continue en MP4 ou Smooth Streaming dans un format pris en charge par Media Services (MPEG DASH, HLS, Smooth Streaming) sans avoir à recréer de nouveaux packages dans ces formats.

Pour tirer parti de l’[empaquetage dynamique](media-services-dynamic-packaging-overview.md), vous devez encoder votre fichier mezzanine (source) en un ensemble de fichiers MP4 à débit adaptatif ou de fichiers Smooth Streaming à débit adaptatif et avoir au moins un point de terminaison de streaming standard ou premium à l’état Démarré.

Media Services prend en charge les éléments suivants sur l'encodeur à la demande décrit dans cet article :

* [Media Encoder Standard](media-services-encode-asset.md#media-encoder-standard)

Pour plus d'informations sur l'encodeur pris en charge, consultez [Encodeur](media-services-encode-asset.md).

## <a name="live-streaming"></a>Vidéo en flux continu
Dans Azure Media Services, un canal représente un pipeline de traitement du contenu de diffusion en direct. Un canal reçoit des flux d’entrée en direct de l’une des deux manières suivantes :

* Un encodeur en direct local envoie au canal un paquet RTMP ou Smooth Streaming (MP4 fragmenté) à débit binaire multiple. Vous pouvez utiliser les encodeurs live suivants qui produisent un flux Smooth Streaming multidébit : MediaExcel, Ateme, Imagine Communications, Envivio, Cisco et Elemental. Les encodeurs live suivants produisent un flux au format RTMP : Adobe Flash Live Encoder, [Telestream Wirecast](media-services-configure-wirecast-live-encoder.md), Teradek et Haivision. Les flux reçus transitent par les canaux sans transcodage et encodage supplémentaires. Lorsqu’il y est invité, Media Services fournit le flux aux clients.
* Un flux à débit unique (dans l’un des formats suivants : RTMP ou Smooth Streaming [MP4 fragmenté]) est envoyé au canal qui est activé pour effectuer un encodage en direct avec Media Services. Le canal procède ensuite à l’encodage en temps réel du flux à débit unique entrant en flux vidéo multidébit (adaptatif). Lorsqu’il y est invité, Media Services fournit le flux aux clients.

### <a name="channel"></a>Channel
Dans Media Services, les [canaux](/rest/api/media/operations/channel)sont responsables du traitement du contenu de vidéo en flux continu. Un canal fournit un point de terminaison d’entrée (URL de réception) que vous fournissez ensuite à un transcodeur live. Le canal reçoit des flux d’entrée live en provenance du transcodeur et les met à disposition pour la diffusion en continu via un ou plusieurs StreamingEndpoints. Les canaux fournissent également un point de terminaison d’aperçu (URL d’aperçu) que vous utilisez pour obtenir un aperçu et valider votre flux avant tout traitement et remise supplémentaires.

Vous pouvez obtenir l’URL de réception et l’URL d’aperçu lors de la création du canal. Pour obtenir ces URL, il n’est pas obligatoire que le canal soit à l’état démarré. Lorsque vous êtes prêt à commencer l’envoi de données à partir d’un transcodeur live dans le canal, celui-ci doit être démarré. Une fois que le transcodeur live a commencé l’ingestion des données, vous pouvez prévisualiser votre flux.

Chaque compte Media Services peut contenir plusieurs canaux, plusieurs programmes et plusieurs StreamingEndpoints. Selon les besoins en matière de bande passante et de sécurité, les services StreamingEndpoint peuvent être affectés à un ou plusieurs canaux. N’importe quel StreamingEndpoint peut assurer l’extraction à partir de n’importe quel canal.

### <a name="program-event"></a>Programme (événement)
Un [programme (événement)](/rest/api/media/operations/program) vous permet de contrôler la publication et le stockage des segments dans un flux live. Les canaux gèrent les programmes (événements). La relation entre canal et programme est similaire au contenu multimédia traditionnel où un canal a un flux de contenu constant et un programme est limité à un événement minuté sur ce canal.
Vous pouvez spécifier le nombre d’heures pendant lesquelles vous souhaitez conserver le contenu enregistré pour le programme en définissant la propriété **ArchiveWindowLength** . Cette valeur peut être comprise entre 5 minutes et 25 heures.

ArchiveWindowLength détermine également la durée maximale que les clients peuvent rechercher en arrière à partir de la position dynamique actuelle. Les programmes peuvent durer davantage que le laps de temps spécifié, mais le contenu qui se situe en dehors de la longueur de fenêtre est ignoré en permanence. La valeur de cette propriété détermine également la longueur maximale que les manifestes de client peuvent atteindre.

Chaque programme (événement) est associé à un élément multimédia. Pour publier le programme, vous devez créer un localisateur pour l’élément multimédia associé. Le fait de posséder ce localisateur vous permettra de générer une URL de diffusion en continu que vous pourrez fournir à vos clients.

Un canal prend en charge jusqu’à trois programmes exécutés simultanément, ce qui rend possible la création de plusieurs archives du même flux entrant. Cela vous permet de publier et d’archiver différentes parties d’un événement en fonction des besoins. Par exemple, imaginez que vous devez archiver 6 heures d’un programme, mais diffuser uniquement les 10 dernières minutes. Pour ce faire, vous devez créer deux programmes exécutés simultanément. Un programme est configuré pour archiver 6 heures de l’événement, mais il n’est pas publié. L’autre programme est configuré pour archiver pendant 10 minutes et il est publié.

Pour plus d'informations, consultez les pages suivantes :

* [Utilisation de canaux activés pour effectuer un encodage en temps réel avec Azure Media Services](media-services-manage-live-encoder-enabled-channels.md)
* [Utilisation des canaux recevant un flux dynamique à débit binaire multiple provenant d’encodeurs locaux](media-services-live-streaming-with-onprem-encoders.md)
* [Quotas et limitations](media-services-quotas-and-limitations.md)

## <a name="protecting-content"></a>Protection du contenu
### <a name="dynamic-encryption"></a>Chiffrement dynamique
Azure Media Services vous permet de sécuriser votre contenu multimédia dès lors qu’il quitte votre ordinateur via le stockage, le traitement et la remise. Media Services permet de transmettre un contenu chiffré dynamiquement avec la norme AES (Advanced Encryption Standard) (à l’aide de clés de chiffrement 128 bits) et le chiffrement commun CENC (Common Encryption) en utilisant PlayReady et/ou Widevine DRM. Media Services fournit également un service de distribution de clés AES et licences PlayReady aux clients autorisés.

Actuellement, vous pouvez chiffrer les formats de diffusion en continu suivants : HLS, MPEG DASH et Smooth Streaming. Vous ne pouvez pas chiffrer les téléchargements progressifs.

Si vous souhaitez que Media Services chiffre un élément multimédia, vous devez associer une clé de chiffrement (CommonEncryption ou EnvelopeEncryption) à votre élément multimédia et configurer des stratégies d’autorisation pour la clé.

Si vous souhaitez diffuser un élément multimédia avec chiffrement de stockage, vous devez configurer la stratégie de remise de l’élément multimédia de manière à indiquer la façon dont il sera remis.

Quand un lecteur demande un flux de données, Media Services utilise la clé spécifiée pour chiffrer dynamiquement votre contenu à l’aide d’un chiffrement d’enveloppe (avec AES) ou d’un chiffrement commun (avec PlayReady ou Widevine). Pour déchiffrer le flux de données, le lecteur demande la clé au service de remise de clé. Pour déterminer si l’utilisateur est autorisé à obtenir la clé, le service évalue les stratégies d’autorisation que vous avez spécifiées pour la clé.

### <a name="token-restriction"></a>Restriction à jeton
La stratégie d’autorisation des clés de contenu peut avoir une ou plusieurs restrictions d’autorisations : ouvert, restriction par jeton ou restriction IP. La stratégie de restriction à jeton doit être accompagnée d’un jeton émis par un service de jeton sécurisé (STS). Media Services prend en charge les jetons aux formats SWT (Simple Web Tokens) et JWT (JSON Web Token). Media Services ne fournit pas de services de jeton sécurisé. Vous pouvez créer un service STS personnalisé. Le STS doit être configuré pour créer un jeton signé avec la clé spécifiée et émettre les revendications spécifiées dans la configuration de restriction de jeton. Le service de remise de clé Media Services retourne la clé (ou licence) demandée au client si le jeton est valide et que les revendications du jeton correspondent à celles configurées pour la clé (ou licence).

Lorsque vous configurez la stratégie de restriction par jeton, vous devez définir les paramètres principaux de clé de vérification, émetteur et public. La clé de vérification principale contient la clé utilisée pour signer le jeton, l’émetteur est le service de jeton sécurisé qui émet le jeton. Le public (parfois appelé l’étendue) décrit l’objectif du jeton ou la ressource à laquelle le jeton autorise l’accès. Le service de remise de clé Media Services valide le fait que les valeurs du jeton correspondent aux valeurs du modèle.

Pour plus d’informations, consultez les articles suivants :
- [Vue d’ensemble de la protection de contenu](media-services-content-protection-overview.md)
- [Protéger avec AES-128](media-services-protect-with-aes128.md)
- [Protéger avec PlayReady/Widevine](media-services-protect-with-playready-widevine.md)

## <a name="delivering"></a>Remise
### <a name="dynamic-packaging"></a><a name="dynamic_packaging"></a>Empaquetage dynamique
Lors de l’utilisation de Media Services, il est recommandé de toujours encoder vos fichiers mezzanine en un ensemble de fichiers MP4 à vitesse de transmission adaptative, puis de convertir le jeu au format souhaité en utilisant l’ [empaquetage dynamique](media-services-dynamic-packaging-overview.md).

### <a name="streaming-endpoint"></a>point de terminaison de diffusion en continu
Un StreamingEndpoint représente un service de diffusion en continu qui peut fournir du contenu directement à une application de lecteur cliente ou à un réseau de distribution de contenu (CDN, Content Delivery Network) en vue d’une distribution supplémentaire (Azure Media Services fournit désormais l’intégration CDN Azure.) Le flux sortant d’un service de point de terminaison de streaming peut être un flux dynamique ou une ressource de vidéo à la demande dans votre compte Media Services. En fonction de leurs besoins, les clients Media Services choisissent un point de terminaison de streaming **Standard** ou un ou plusieurs points de terminaison de streaming **Premium**. Le point de terminaison de streaming standard convient à la plupart des charges de travail de diffusion en continu. 

Le point de terminaison de streaming standard convient à la plupart des charges de travail de diffusion en continu. Les points de terminaison de streaming standard offrent la flexibilité nécessaire pour fournir votre contenu à presque tous les types d’appareils grâce à l’empaquetage dynamique dans HLS, MPEG-DASH et Smooth Streaming, ainsi que le chiffrement dynamique pour Microsoft PlayReady, Google Widevine, Apple Fairplay et AES128.  Ils peuvent également être mis à l’échelle pour des publics très réduits ou très nombreux, comptant des milliers d’utilisateurs simultanés grâce à l’intégration d’Azure CDN. Si votre charge de travail est avancée, si vos besoins en capacité de diffusion en continu ne correspondent pas aux cibles de débit du point de terminaison de streaming standard ou si vous souhaitez contrôler la capacité du service StreamingEndpoint afin de gérer les besoins croissants en termes de bande passante, nous vous recommandons d’attribuer des unités d’échelle, également appelées « unités de diffusion premium ».

Il est recommandé d’utiliser l’empaquetage dynamique et/ou le chiffrement dynamique.

>[!NOTE]
>Une fois votre compte AMS créé, un point de terminaison de diffusion continue **par défaut** est ajouté à l’état **Arrêté**. Pour démarrer la diffusion en continu de votre contenu et tirer parti de l’empaquetage et du chiffrement dynamiques, le point de terminaison de streaming à partir duquel vous souhaitez diffuser du contenu doit se trouver à l’état **En cours d’exécution**. 

Pour plus d’informations, consultez [cette rubrique](media-services-portal-manage-streaming-endpoints.md) .

Par défaut, vous bénéficiez au maximum de deux points de terminaison de diffusion en continu dans votre compte Media Services. Pour demander une limite plus élevée, consultez la rubrique [Quotas et limitations](media-services-quotas-and-limitations.md).

Vous êtes facturé uniquement lorsque votre StreamingEndpoint est en cours d’exécution.

### <a name="asset-delivery-policy"></a>Stratégie de remise d’élément multimédia
L’une des étapes du flux de travail de remise de contenu Media Services consiste à configurer les [stratégies de remise pour les éléments multimédias](/rest/api/media/operations/assetdeliverypolicy) que vous souhaitez diffuser en continu. La stratégie de remise de ressources indique à Media Services comment vous souhaitez distribuer vos ressources : dans quel protocole de diffusion en continu votre ressource doit être empaquetée dynamiquement (par exemple, MPEG DASH, HLS, Smooth Streaming ou tous), si vous souhaitez chiffrer dynamiquement votre ressource ou non et comment (chiffrement commun ou d’enveloppe).

Si vous avez un élément multimédia avec chiffrement de stockage, avant de pouvoir le diffuser en continu, le serveur de diffusion supprime le chiffrement de stockage et diffuse en continu votre contenu à l’aide de la stratégie de remise spécifiée. Par exemple, pour remettre votre élément multimédia chiffré avec une clé de chiffrement AES (Advanced Encryption Standard), définissez le type de stratégie sur DynamicEnvelopeEncryption. Pour supprimer le chiffrement de stockage et diffuser la ressource en clair, définissez le type de stratégie sur NoDynamicEncryption.

### <a name="progressive-download"></a>Téléchargement progressif
Le téléchargement progressif vous permet de commencer la lecture multimédia avant que l’intégralité du fichier ait été téléchargée. Seul un fichier MP4 peut être téléchargé progressivement.

>[!NOTE]
>Vous devez déchiffrer les éléments multimédias chiffrés si vous souhaitez qu’ils soient disponibles pour le téléchargement progressif.

Pour fournir aux utilisateurs des URL de téléchargement progressif, vous devez d’abord créer un localisateur OnDemandOrigin. La création du localisateur vous donne le chemin d’accès de base à l’élément multimédia. Vous devez ensuite ajouter le nom du fichier MP4. Par exemple :

`http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4`

### <a name="streaming-urls"></a>URL de diffusion
Diffusion en continu de votre contenu aux clients. Pour fournir aux utilisateurs des URL de diffusion en continu, vous devez d’abord créer un localisateur OnDemandOrigin. La création du localisateur vous donne le chemin d’accès de base à l’élément multimédia qui contient le contenu que vous souhaitez diffuser. Toutefois, pour pouvoir diffuser ce contenu vous devez modifier ce chemin d’accès. Pour construire une URL complète vers le fichier manifeste de diffusion en continu, vous devez concaténer la valeur de chemin d’accès du localisateur et le nom du fichier manifeste (nom_fichier.ISM). Ensuite, ajoutez /Manifest et un format approprié (si nécessaire) au chemin d’accès du localisateur.

Vous pouvez aussi diffuser votre contenu via une connexion TSL. Pour ce faire, assurez-vous que votre URL de diffusion commence par HTTPS. Actuellement, AMS ne prend pas en charge le protocole TLS avec les domaines personnalisés.  

>[!NOTE]
>Vous ne pouvez transmettre en continu avec le protocole TLS que si le point de terminaison de streaming à partir duquel vous distribuez votre contenu a été créé après le 10 septembre 2014. Si vos URL de diffusion sont basées sur des points de terminaison créés après le 10 septembre, l’URL contient « streaming.mediaservices.windows.net » (le nouveau format). Les URL de diffusion qui contiennent « origin.mediaservices.windows.net » (ancien format) ne sont pas compatibles avec le protocole TLS. Si votre URL suit l’ancien format et que vous souhaitez être en mesure de diffuser via le protocole TLS, créez un point de terminaison de diffusion en continu. L’utilisation d’URL créées à partir du nouveau point de terminaison permet de diffuser votre contenu via le protocole TLS.

La liste suivante décrit les différents formats de diffusion en continu et donne des exemples :

* Smooth Streaming

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest

* MPEG DASH

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest(format=mpd-time-csf)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)

* Apple HTTP Live Streaming (HLS) V4

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest(format=m3u8-aapl)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)

* Apple HTTP Live Streaming (HLS) V3

{nom du point de terminaison de diffusion en continu-nom du compte media services}.streaming.mediaservices.windows.net/{ID_de_localisateur}/{nom_de_fichier}.ISM/Manifest(format=m3u8-aapl-v3)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
