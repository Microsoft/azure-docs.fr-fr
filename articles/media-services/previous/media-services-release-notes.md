---
title: Notes de publication Azure Media Services | Microsoft Docs
description: Cet article traite des notes de publication de Microsoft Azure Media Services v2.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: media
ms.devlang: dotnet
ms.topic: article
ms.date: 10/01/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: bd4f50365ced61e35ae6d10a9763a0f9815098d7
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92014101"
---
# <a name="azure-media-services-release-notes"></a>Notes de publication d'Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Ces notes de publication pour Azure Media Services récapitulent les modifications par rapport aux précédentes versions et les problèmes connus.

> [!NOTE]
> Aucune nouvelle fonctionnalité n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Nous souhaitons connaître vos impressions afin de pouvoir nous consacrer à la résolution des problèmes que vous rencontrez. Pour signaler un problème ou poser des questions, publiez un post sur le [Forum MSDN sur Azure Media Services]. 

## <a name="known-issues"></a><a name="issues"></a>Problèmes connus
### <a name="media-services-general-issues"></a><a name="general_issues"></a>Problèmes généraux concernant Media Services

| Problème | Description |
| --- | --- |
| Plusieurs en-têtes HTTP courants ne sont pas fournis dans l’API REST. |Si vous développez des applications Media Services à l’aide de l’API REST, vous constaterez que certains champs d’en-tête HTTP courants (notamment CLIENT-REQUEST-ID, REQUEST-ID et RETURN-CLIENT-REQUEST-ID) ne sont pas pris en charge. Les en-têtes seront ajoutés dans une prochaine mise à jour. |
| L’encodage par pourcentage n’est pas autorisé. |Media Services utilise la valeur de la propriété IAssetFile.Name pour générer des liens URL pour la diffusion de contenu en continu (par exemple, `http://{AMSAccount}.origin.mediaservices.windows.net/{GUID}/{IAssetFile.Name}/streamingParameters`). Pour cette raison, l’encodage par pourcentage n’est pas autorisé. La valeur de la propriété Name ne peut pas comporter les [caractères réservés à l’encodage en pourcentage suivants](https://en.wikipedia.org/wiki/Percent-encoding#Percent-encoding_reserved_characters) : !* ’();:@&=+$,/?%#[]". En outre, il ne peut exister qu’un « . » pour l’extension de nom de fichier. |
| La méthode ListBlobs intégrée à la version 3.x du Kit de développement logiciel (SDK) d'Azure Storage échoue. |Media Services génère des URL SAS basées sur la version du [02/12/2012](/rest/api/storageservices/version-2012-02-12) . Si vous voulez utiliser le SDK d’Azure Storage pour répertorier les objets blob dans un conteneur d’objets blob, utilisez la méthode [CloudBlobContainer.ListBlobs](/dotnet/api/microsoft.azure.storage.blob.cloudblobcontainer.listblobs) intégrée à la version 2.x de ce SDK. |
| Le mécanisme de limitation de Media Services restreint l’utilisation des ressources pour les applications qui recourent de manière excessive au service. Le service peut renvoyer le code d’état HTTP 503 « Service indisponible ». |Pour plus d’informations, consultez la description du code d’état HTTP 503 dans [Codes d’erreur d’Azure Media Services](media-services-encoding-error-codes.md). |
| Quand vous interrogez des entités, il existe une limite de 1000 entités retournées simultanément, car l’API REST version 2 publique limite les résultats des requêtes à 1000 résultats. |Utilisez Skip et Take (.NET)/ top (REST) comme décrit dans [cet exemple .NET](media-services-dotnet-manage-entities.md#enumerating-through-large-collections-of-entities) et [cet exemple d’API REST](media-services-rest-manage-entities.md#enumerating-through-large-collections-of-entities). |
| Certains clients peuvent rencontrer un problème de répétition de balise dans le manifeste de diffusion en continu lisse. |Pour plus d’informations, consultez [cette section](media-services-deliver-content-overview.md#known-issues). |
| Les objets du kit SDK Media Services .NET ne peuvent pas être sérialisés et, par conséquent, ne fonctionnent pas avec le cache Azure pour Redis. |Si vous essayez de sérialiser l’objet AssetCollection du SDK pour l’ajouter au cache Azure pour Redis, une exception est levée. |
|L’API REST répond avec un message d’erreur indiquant que le filtre n’est pas accessible par cette version de l’API REST lors de la tentative d’obtention d’un filtre au niveau de l’actif multimédia ou du compte.|Le filtre a été créé ou modifié avec une version d’API plus récente que celle utilisée pour essayer d’obtenir le filtre. Cela peut se produire si deux versions d’API sont utilisées par le code ou les outils employés par le client.  La meilleure solution ici consiste à mettre à niveau le code ou les outils pour utiliser la version plus récente ou les deux versions d’API.|

## <a name="rest-api-version-history"></a><a name="rest_version_history"></a>Historique des versions de l’API REST
Pour obtenir des informations sur l’historique des versions de l’API REST, consultez la [Référence de l’API REST d’Azure Media Services].

## <a name="september-2020"></a>Septembre 2020

Les propriétés v2 suivantes ne seront plus remplies avec les données historiques de progression du travail :

* [HistoricalEvents](/dotnet/api/microsoft.windowsazure.mediaservices.client.itask.historicalevents)
* [PerfMessage](/dotnet/api/microsoft.windowsazure.mediaservices.client.itask.perfmessage)

Pour afficher l’historique des tâches, utilisez les notifications de travail v2 au moyen de webhooks ou de messages de file d’attente à l’aide de points de terminaison de notification. Pour plus d'informations, consultez les pages suivantes :

* [Utiliser le stockage de files d’attente Azure pour surveiller les notifications de tâches Media Services](media-services-dotnet-check-job-progress-with-queues.md)
* [Utiliser Azure Webhooks pour superviser les notifications de travaux Media Services](media-services-dotnet-check-job-progress-with-webhooks.md)

## <a name="february-2020"></a>Février 2020

Certains Processeurs multimédias Analytics seront mis hors service. Pour connaître les dates de mise hors service, voir la rubrique [Composants hérités](legacy-components.md).

## <a name="september-2019"></a>Septembre 2019

### <a name="deprecation-of-media-processors"></a>Dépréciation des processeurs multimédias

Nous annonçons la dépréciation d’*Azure Media Indexer* et d’*Azure Media Indexer 2 Preview*. [Azure Media Services Video Indexer](../video-indexer/index.yml) remplace ces processeurs multimédias hérités.

Pour connaître les dates de mise hors service, consultez la rubrique [Composants hérités](legacy-components.md).

Consultez également [Effectuer une migration depuis Azure Media Indexer et Azure Media Indexer 2 vers Azure Media Services Video Indexer](migrate-indexer-v1-v2.md).

## <a name="august-2019"></a>Août 2019

### <a name="deprecation-of-media-processors"></a>Dépréciation des processeurs multimédias

Nous annonçons la dépréciation des processeurs multimédias *Windows Azure Media Encoder* (WAME) et *Azure Media Encoder* (AME). Pour les dates de mise hors service, consultez cette rubrique consacrée aux [composants hérités](legacy-components.md).

Pour plus d'informations, consultez [Migration de WAME vers Media Encoder Standard](./migrate-windows-azure-media-encoder.md) et [Migration d'AME vers Media Encoder Standard](./migrate-azure-media-encoder.md).

## <a name="march-2019"></a>Mars 2019

L’utilisation de la fonctionnalité d’évaluation Media Hyperlapse d’Azure Media Services est déconseillée.

## <a name="december-2018"></a>Décembre 2018

La fonctionnalité d’évaluation Media Hyperlapse d’Azure Media Services a récemment été mise hors service. Depuis le 19 décembre 2018, Media Services n’apporte plus de changements ou d’améliorations à Media Hyperlapse. Le 29 mars 2019, elle sera mise hors service et ne sera plus disponible.

## <a name="october-2018"></a>Octobre 2018

### <a name="cmaf-support"></a>Prise en charge de CMAF

Prise en charge du chiffrement CMAF et « cbcs » pour les lecteurs Apple HLS (iOS 11+) et MPEG-DASH qui prennent en charge CMAF.

### <a name="web-vtt-thumbnail-sprites"></a>Sprites de miniatures WebVTT

Vous pouvez désormais utiliser Media Services pour générer des sprites de miniatures WebVTT à l’aide de nos API v2. Pour plus d’informations, consultez [Générer un sprite de miniatures](generate-thumbnail-sprite.md).

## <a name="july-2018"></a>Juillet 2018

La dernière version du service comprend des changements mineurs de mise en forme des messages d’erreur retournés par le service en cas d’échec d’un travail, du point de vue de sa décomposition en deux ou plusieurs lignes.

## <a name="may-2018"></a>Mai 2018 

À partir du 12 mai 2018, les canaux en direct ne prendront plus en charge le protocole de réception du flux de transport RTP/MPEG-2. Effectuez la migration à partir de RTP/MPEG-2 vers le protocole de réception RTMP ou MP4 fragmenté (Smooth Streaming).

## <a name="october-2017-release"></a>Version d’octobre 2017
> [!IMPORTANT] 
> Media Services déprécie la prise en charge des clés d’authentification Azure Access Control Service. Le 22 juin 2018, vous ne pourrez plus vous authentifier auprès du backend Media Services par le biais de code, à l’aide de clés Access Control Service. Vous devez mettre à jour votre code pour utiliser Azure Active Directory (Azure AD) comme l’indique l’article [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md). Surveillez les avertissements relatifs à ce changement dans le portail Azure.

### <a name="updates-for-october-2017"></a>Mises à jour d’octobre 2017
#### <a name="sdks"></a>Kits SDK
* Le SDK .NET a été mis à jour pour prendre en charge l’authentification Azure AD. La prise en charge de l’authentification Access Control Service a été supprimée dans la dernière version du SDK .NET sur Nuget.org afin d’accélérer la migration vers Azure AD. 
* Le SDK Java a mis à jour pour prendre en charge l’authentification Azure AD. La prise en charge de l’authentification Azure AD a été ajoutée au SDK Java. Pour plus d’informations sur la façon d’utiliser le SDK Java avec Media Services, consultez [Prise en main du Kit de développement logiciel du client Java pour Azure Media Services](media-services-java-how-to-use.md).

#### <a name="file-based-encoding"></a>Encodage basé sur un fichier
* Vous pouvez maintenant utiliser l’encodeur Premium pour encoder votre contenu avec le codec vidéo HEVC (High-Efficiency Video Coding) H.265. Choisir H.265 n’a aucun impact sur le prix par rapport à d’autres codecs comme H.264. Pour plus d’informations sur les licences de brevet HEVC, consultez les [Conditions de Services en ligne](https://azure.microsoft.com/support/legal/).
* Pour une source vidéo encodée avec le codec vidéo H.265 (HEVC), par exemple une vidéo capturée à l’aide d’iOS 11 ou de GoPro Hero 6, vous pouvez maintenant utiliser l’encodeur Premium ou l’encodeur Standard pour encoder ces vidéos. Pour plus d’informations sur les licences de brevet, consultez les [Conditions de Services en ligne](https://azure.microsoft.com/support/legal/).
* Pour le contenu qui contient des pistes audio en plusieurs langues, les valeurs de langue doivent être libellées correctement conformément à la spécification de format de fichier correspondante (par exemple, ISO MP4). Vous pouvez ensuite utiliser l’encodeur Standard afin d’encoder le contenu pour le streaming. Le localisateur de streaming liste les langues audio disponibles.
* L’encodeur Standard prend désormais en charge deux nouvelles présélections de système audio uniquement, « Audio AAC » et « Bonne qualité audio AAC ». Ces deux options génèrent une sortie stéréo AAC (Advanced Audio Coding), respectivement à des débits de 128 kbit/s et 192 kbit/s.
* L’encodeur Premium prend désormais en charge les formats de fichier QuickTime/MOV comme entrée. Le codec vidéo doit être l’un des [types Apple ProRes répertoriés dans cet article GitHub](./media-services-media-encoder-standard-formats.md). Le son doit être au format AAC ou PCM (Pulse Code Modulation). L’encodeur Premium ne prend pas en charge comme entrée la vidéo DVC/DVCPro encapsulée dans des fichiers QuickTime/MOV (par exemple). L’encodeur Standard, en revanche, prend en charge ces codecs vidéo.
* Les correctifs de bogue suivants ont été apportés dans les encodeurs :

    * Vous pouvez maintenant envoyer des travaux à l’aide d’un actif multimédia d’entrée. Une fois ces travaux terminés, vous pouvez modifier l’actif multimédia (par exemple ajouter, supprimer ou renommer des fichiers au sein de l’actif multimédia) et soumettre des travaux supplémentaires.
    * La qualité des miniatures JPEG produites par l’encodeur Standard a été améliorée.
    * L’encodeur Standard gère mieux les métadonnées d’entrée et la génération de miniatures dans les vidéos de très courte durée.
    * Des améliorations apportées au décodeur H.264 utilisé dans l’encodeur Standard éliminent certains artefacts rares. 

#### <a name="media-analytics"></a>Media Analytics
Disponibilité générale d’Azure Media Redactor : ce processeur multimédia assure l’anonymat en ajoutant un effet de flou sur les visages des personnes sélectionnées, et son utilisation est idéale dans le domaine de la sécurité publique et pour les médias d’information. 

Pour obtenir une vue d’ensemble de ce nouveau processeur, consultez [ce billet de blog](https://azure.microsoft.com/blog/azure-media-redactor/). Pour plus d’informations sur la documentation et les paramètres, consultez [Éditer les visages avec Azure Media Analytics](media-services-face-redaction.md).



## <a name="june-2017-release"></a>Version de juin 2017

Media Services prend désormais en charge [l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md).

> [!IMPORTANT]
> À l’heure actuelle, Media Services prend en charge le modèle d’authentification Access Control Service. L’autorisation Access Control Service sera dépréciée à compter du 1er juin 2018. Nous vous recommandons de migrer vers le modèle d’authentification Azure AD dès que possible.

## <a name="march-2017-release"></a>Version de mars 2017

Vous pouvez désormais utiliser l’encodeur Standard pour [générer automatiquement une échelle de vitesse de transmission](media-services-autogen-bitrate-ladder-with-mes.md) en spécifiant la chaîne prédéfinie « Streaming adaptatif » quand vous créez une tâche d’encodage. Pour encoder une vidéo pour du streaming avec Media Services, utilisez la valeur prédéfinie « Streaming adaptatif ». Pour personnaliser un encodage prédéfini pour votre scénario spécifique, vous pouvez commencer par [ces options prédéfinies](media-services-mes-presets-overview.md).

Vous pouvez désormais utiliser Media Encoder Standard ou Media Encoder Premium Workflow pour [créer une tâche d’encodage qui génère des segments fMP4](media-services-generate-fmp4-chunks.md). 

## <a name="february-2017-release"></a>Version de février 2017

Depuis le 1er avril 2017, les enregistrements de travaux de votre compte de plus de 90 jours sont supprimés automatiquement, ainsi que leurs enregistrements de tâches associés. La suppression se produit même si le nombre total d’enregistrements est inférieur au quota maximal. Pour archiver les informations des travaux/tâches, vous pouvez utiliser le code décrit dans [Gérer les actifs multimédias et les entités connexes avec le SDK Media Services .NET](media-services-dotnet-manage-entities.md).

## <a name="january-2017-release"></a>Version de janvier 2017

Dans Media Services, un point de terminaison de streaming représente un service de streaming qui peut fournir du contenu directement à une application de lecteur cliente ou à un réseau de diffusion de contenu (CDN) pour être redistribué. Media Services fournit également une intégration transparente au CDN Azure. Le flux sortant d’un service StreamingEndpoint peut être un flux dynamique, une vidéo à la demande ou un téléchargement progressif de votre ressource dans votre compte Media Services. Chaque compte Media Services comprend un point de terminaison de streaming par défaut. Vous pouvez créer d’autres points de terminaison de streaming sous votre compte. 

Il existe deux versions de point de terminaison de streaming : 1.0 et 2.0. À compter du 10 janvier 2017, les nouveaux comptes Media Services incluent le point de terminaison de streaming par défaut de version 2.0. Les autres points de terminaison que vous ajoutez à ce compte sont également de la version 2.0. Ce changement n’affecte pas les comptes existants. Les points de terminaison de streaming existants sont de version 1.0 et peuvent être mis à niveau vers la version 2.0. Ce changement s’accompagne de modifications en matière de fonctionnalités, de facturation et de comportement. Pour plus d’informations, consultez [Vue d’ensemble des points de terminaison de streaming](media-services-streaming-endpoints-overview.md).

À partir de la version 2.15, Media Services a ajouté les propriétés suivantes à l’entité de point de terminaison de streaming :

* CdnProvider 
* CdnProfile
* FreeTrialEndTime 
* StreamingEndpointVersion 

Pour plus d’informations sur ces propriétés, consultez [StreamingEndpoint](/rest/api/media/operations/streamingendpoint). 

## <a name="december-2016-release"></a>Version de décembre 2016

 Vous pouvez désormais utiliser Media Services pour accéder aux données de télémétrie/métriques pour ses services. Vous pouvez utiliser la version actuelle de Media Services pour recueillir des données de télémétrie pour les entités en temps réel de canal, de point de terminaison de streaming et d’archive. Pour plus d’informations, consultez [Télémétrie Azure Media Services](media-services-telemetry-overview.md).

## <a name="july-2016-release"></a><a name="july_changes16"></a>Version de juillet 2016
### <a name="updates-to-the-manifest-file-ism-generated-by-encoding-tasks"></a>Mises à jour apportées au fichier manifeste (*.ISM) généré par les tâches d’encodage
Quand une tâche d’encodage est soumise à Media Encoder Standard ou à Media Encoder Premium, elle génère un [fichier manifeste de streaming](media-services-deliver-content-overview.md) (*.ism) dans la ressource de sortie. Dans la dernière version de service, la syntaxe de ce fichier manifeste de streaming a été mise à jour.

> [!NOTE]
> La syntaxe du fichier manifeste de streaming (.ism) est réservée à un usage interne. Elle est susceptible de changer dans les versions à venir. Ne pas modifier ni manipuler le contenu de ce fichier.
> 
> 

### <a name="a-new-client-manifest-ismc-file-is-generated-in-the-output-asset-when-an-encoding-task-outputs-one-or-more-mp4-files"></a>Un nouveau fichier manifeste client (*.ISMC) est généré dans l’élément de sortie quand une tâche d’encodage génère un ou plusieurs fichiers MP4.
À compter de la version de service la plus récente, une fois que la tâche d’encodage a généré plusieurs fichiers MP4, l’élément de sortie contient également un fichier manifeste de streaming (*.ismc) client. Le fichier .ismc vous aide à améliorer les performances de diffusion en continu dynamique. 

> [!NOTE]
> La syntaxe du fichier manifeste client (.ism) est réservée à un usage interne. Elle est susceptible de changer dans les versions à venir. Ne pas modifier ni manipuler le contenu de ce fichier.
> 
> 

Pour plus d’informations, consultez [ce blog](/archive/blogs/randomnumber/encoder-changes-within-azure-media-services-now-create-ismc-file).

### <a name="known-issues"></a>Problèmes connus
Certains clients peuvent rencontrer un problème de répétition de balise dans le manifeste de diffusion en continu lisse. Pour plus d’informations, consultez [cette section](media-services-deliver-content-overview.md#known-issues).

## <a name="april-2016-release"></a><a id="apr_changes16"></a>Version d’avril 2016
### <a name="media-analytics"></a>Media Analytics
 Media Services a intégré Media Analytics pour offrir des fonctionnalités vidéo performantes. Pour plus d’informations, consultez [Media Analytics sur la plateforme Media Services](./legacy-components.md).

### <a name="apple-fairplay-preview"></a>Apple FairPlay (préversion)
Vous pouvez maintenant utiliser Media Services pour chiffrer de manière dynamique votre contenu HLS (HTTP Live Streaming) avec Apple FairPlay. Vous pouvez aussi utiliser le service de remise de licence Media Services pour fournir des licences FairPlay aux clients. Pour plus d’informations, consultez « Utiliser Azure Media Services pour diffuser votre contenu HLS protégé avec Apple FairPlay ».

## <a name="february-2016-release"></a><a id="feb_changes16"></a>Version de février 2016
La dernière version du SDK Media Services pour .NET (3.5.3) contient un correctif de bogue relatif à Google Widevine. Il était impossible de réutiliser AssetDeliveryPolicy pour plusieurs actifs multimédias chiffrés avec Widevine. Dans le cadre de ce correctif de bogue, la propriété suivante a été ajoutée au SDK : WidevineBaseLicenseAcquisitionUrl.

```csharp
Dictionary<AssetDeliveryPolicyConfigurationKey, string> assetDeliveryPolicyConfiguration =
    new Dictionary<AssetDeliveryPolicyConfigurationKey, string>
{
    {AssetDeliveryPolicyConfigurationKey.WidevineBaseLicenseAcquisitionUrl,"http://testurl"},

};
```

## <a name="january-2016-release"></a><a id="jan_changes_16"></a>Version de janvier 2016
Les unités réservées d’encodage ont été renommées pour éviter la confusion avec les noms d’encodeurs.

Les unités réservées d’encodage De base, Standard et Premium ont été renommées respectivement S1, S2 et S3. Les clients qui utilisent des unités réservées d’encodage De base voient aujourd’hui S1 comme étiquette dans le portail Azure (et dans la nomenclature). Les clients qui utilisent Standard et Premium voient respectivement les étiquettes S2 et S3. 

## <a name="december-2015-release"></a><a id="dec_changes_15"></a>Version de décembre 2015

### <a name="media-encoder-deprecation-announcement"></a>Annonce de dépréciation de Media Encoder

 Media Encoder sera déprécié environ 12 mois après le lancement de la version Media Encoder Standard.

### <a name="azure-sdk-for-php"></a>Kit SDK Azure pour PHP
L’équipe du SDK Azure a publié une nouvelle version du package [SDK Azure pour PHP](https://github.com/Azure/azure-sdk-for-php) qui contient des mises à jour et nouvelles fonctionnalités pour Media Services. En particulier, le SDK Media Services pour PHP prend désormais en charge la dernière version des fonctionnalités de [protection du contenu](media-services-content-protection-overview.md). Ces fonctionnalités sont le chiffrement dynamique avec AES et DRM (PlayReady et Widevine) avec et sans restrictions de jetons. Il prend également en charge la mise à l’échelle des [unités d’encodage](media-services-dotnet-encoding-units.md).

Pour plus d'informations, consultez les pages suivantes :

* Les [exemples de code](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices) suivants vous aident à démarrer rapidement :
  * **vodworkflow_aes.php** : ce fichier PHP indique comment utiliser le chiffrement dynamique AES 128 et le service de remise de clé. Il est basé sur l’exemple .NET décrit dans [Utilisation du chiffrement dynamique AES-128 et du service de distribution des clés](media-services-protect-with-aes128.md).
  * **vodworkflow_aes.php** : ce fichier PHP indique comment utiliser le chiffrement dynamique PlayReady et le service de remise de licence. Il est basé sur l’exemple .NET décrit dans [Utilisation du chiffrement commun dynamique PlayReady et/ou Widevine](media-services-protect-with-playready-widevine.md).
  * **scale_encoding_units.php** : ce fichier PHP indique comment mettre à l’échelle des unités réservées d’encodage.

## <a name="november-2015-release"></a><a id="nov_changes_15"></a>Version de novembre 2015
 Media Services propose désormais un service de remise de licence Widevine dans le cloud. Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/). Consultez également [ce didacticiel](media-services-protect-with-playready-widevine.md) et le [dépôt GitHub](https://github.com/Azure-Samples/media-services-dotnet-dynamic-encryption-with-drm). 

Les services de remise de licence Widevine fournis par Media Services sont en préversion. Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/announcing-google-widevine-license-delivery-services-public-preview-in-azure-media-services/).

## <a name="october-2015-release"></a><a id="oct_changes_15"></a>Version d’octobre 2015
Media Services est désormais disponible dans les centres de données suivants : Brésil Sud, Inde Ouest, Inde Sud et Inde Centre. Vous pouvez maintenant utiliser le portail Azure pour [créer des comptes Media Service](media-services-portal-create-account.md) et effectuer les différentes tâches décrites dans la [page web de documentation de Media Services](https://azure.microsoft.com/documentation/services/media-services/). La fonctionnalité Live Encoding n’est pas activée dans ces centres de données. En outre, tous les types d’unités réservées d’encodage ne sont pas disponibles dans ces centres de données.

* Brésil Sud :                                          seules les unités réservées d’encodage Standard et De base sont disponibles.
* Inde Ouest, Inde Sud et Inde Centre :             seules les unités réservées d’encodage De base sont disponibles.

## <a name="september-2015-release"></a><a id="september_changes_15"></a>Version de septembre 2015
Media Services offre désormais la possibilité de protéger la vidéo à la demande et les flux en direct avec la technologie DRM modulaire Widevine. Vous pouvez utiliser les partenaires des services de remise suivants pour fournir des licences Widevine :
* [Axinom](https://www.axinom.com) 
* [EZDRM](https://ezdrm.com/) 
* [castLabs](https://castlabs.com/company/partners/azure/) 

Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/azure-media-services-adds-google-widevine-packaging-for-delivering-multi-drm-stream/).
  
Vous pouvez utiliser le [SDK .NET Media Services](https://www.nuget.org/packages/windowsazure.mediaservices/) (à compter de la version 3.5.1) ou une API REST pour configurer votre AssetDeliveryConfiguration afin d’utiliser Widevine. 
* Media Services a ajouté la prise en charge des vidéos Apple ProRes. Vous pouvez à présent télécharger vos fichiers vidéos sources QuickTime qui utilisent Apple ProRes ou d’autres codecs. Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/announcing-support-for-apple-prores-videos-in-azure-media-services/).
* Vous pouvez à présent utiliser Media Encoder Standard pour effectuer un sous-découpage et une extraction d’archive dynamique. Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/sub-clipping-and-live-archive-extraction-with-media-encoder-standard/).
* Les mises à jour de filtrage suivantes ont été effectuées : 
  
  * Vous pouvez désormais utiliser le format Apple HLS avec filtre audio uniquement. Vous pouvez utiliser cette mise à jour pour supprimer une piste audio uniquement en spécifiant (audio-only=false) dans l’URL.
  * Quand vous définissez des filtres pour vos ressources, vous pouvez désormais combiner plusieurs filtres (jusqu’à trois) dans une même URL.
    
    Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/).
* Media Services prend désormais en charge les I-frames dans HLS version 4. La prise en charge des I-frames optimise les opérations d’avance rapide et de retour arrière. Par défaut, toutes les sorties HLS version 4 incluent la sélection I-frame (EXT-X-I-FRAME-STREAM-INF).
Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/azure-media-services-release-dynamic-manifest-composition-remove-hls-audio-only-track-and-hls-i-frame-track-support/).

## <a name="august-2015-release"></a><a id="august_changes_15"></a>Version d’août 2015
* Le SDK Media Services pour Java version 0.8.0 et de nouveaux exemples sont désormais disponibles. Pour plus d'informations, consultez les pages suivantes :
    
* Azure Media Player a été mis à jour avec la prise en charge des flux audio multiples. Pour plus d’informations, consultez [ce billet de blog](https://azure.microsoft.com/blog/2015/08/13/azure-media-player-update-with-multi-audio-stream-support/)

## <a name="july-2015-release"></a><a id="july_changes_15"></a>Version de juillet 2015
* La disponibilité générale de Media Encoder Standard a été annoncée. Pour plus d’informations, consultez [ce billet de blog](https://azure.microsoft.com/blog/2015/07/16/announcing-the-general-availability-of-media-encoder-standard/)
  
    Media Encoder Standard utilise les présélections, comme décrit dans [cette section](./media-services-mes-presets-overview.md). Quand vous utilisez une présélection pour les encodages 4K, obtenez le type d’unité réservée Premium. Pour plus d’informations, consultez [Mise à l’échelle de l’encodage](media-services-scale-media-processing-overview.md).
* Les légendes en temps réel en direct ont été utilisées avec Media Services et Media Player. Pour plus d’informations, consultez [ce billet de blog](https://azure.microsoft.com/blog/2015/07/08/live-real-time-captions-with-azure-media-services-and-player/)

### <a name="media-services-net-sdk-updates"></a>Mises à jour du SDK .NET Media Services
Le SDK Media Services en est maintenant à la version 3.4.0.0. Les mises à jour suivantes ont été effectuées : 

* La prise en charge d’une archive en direct a été implémentée. Vous ne pouvez pas télécharger un élément contenant une archive en direct.
* La prise en charge des filtres dynamiques a été implémentée.
* Une fonctionnalité a été implémentée afin de permettre aux utilisateurs de conserver un conteneur de stockage pendant qu’ils suppriment un actif multimédia.
* Des correctifs de bogues liés aux stratégies de nouvelle tentative sur les canaux ont été effectués.
* Media Encoder Premium Workflow a été activé.

## <a name="june-2015-release"></a><a id="june_changes_15"></a>Version de juin 2015
### <a name="media-services-net-sdk-updates"></a>Mises à jour du SDK .NET Media Services
Le SDK Media Services en est maintenant à la version 3.3.0.0. Les mises à jour suivantes ont été effectuées : 

* La prise en charge de la spécification de découverte d’OpenID Connect a été ajoutée.
* La prise en charge de la gestion du renouvellement de clés côté fournisseur d’identité a été ajoutée.

Si vous utilisez un fournisseur d’identité qui expose le document de découverte de OpenID Connect (comme Azure AD, Google et Salesforce), vous pouvez demander à Media Services d’obtenir des clés de signature pour la validation des jetons JWT (JSON Web Tokens) provenant de la spécification de découverte d’OpenID Connect. 

Pour plus d’informations, consultez [Use JSON web keys from the OpenID Connect discovery spec to work with JWT authentication in Media Services](http://gtrifonov.com/2015/06/07/using-json-web-keys-from-openid-connect-discovery-spec-to-work-with-jwt-token-authentication-in-azure-media-services/).

## <a name="may-2015-release"></a><a id="may_changes_15"></a>Version de mai 2015
Les nouvelles fonctionnalités suivantes ont été annoncées :

* [Une préversion de l’encodage en temps réel avec Media Services](media-services-manage-live-encoder-enabled-channels.md)
* [Un manifeste dynamique](media-services-dynamic-manifest-overview.md)

## <a name="april-2015-release"></a><a id="april_changes_15"></a>Version d’avril 2015
### <a name="general-media-services-updates"></a>Mises à jour générales de Media Services
* [Media Player](https://azure.microsoft.com/blog/2015/04/15/announcing-azure-media-player/) a été annoncé.
* À compter de Media Services REST 2.10, les canaux configurés pour recevoir un protocole RTMP (Real-Time Messaging Protocol) sont créés avec des URL de réception principale et secondaire. Pour plus d’informations, consultez [Configurations de l’entrée (réception) des canaux](media-services-live-streaming-with-onprem-encoders.md#channel_input).
* Azure Media Indexer a été mis à jour.
* La prise en charge de l’espagnol a été ajoutée.
* Une nouvelle configuration pour le format XML a été ajoutée.

Pour plus d’informations, consultez [ce blog](https://azure.microsoft.com/blog/2015/04/13/azure-media-indexer-spanish-v1-2/).

### <a name="media-services-net-sdk-updates"></a>Mises à jour du SDK .NET Media Services
Le SDK Media Services en est maintenant à la version 3.2.0.0. Les mises à jour suivantes ont été effectuées :

* Changement cassant : TokenRestrictionTemplate.Issuer et TokenRestrictionTemplate.Audience ont été changés pour être de type chaîne.
* Des mises à jour relatives à la création de stratégies de nouvelle tentative personnalisées ont été effectuées.
* Des correctifs liés au téléchargement/chargement de fichiers ont été effectués.
* La classe MediaServicesCredentials accepte désormais les points de terminaison de contrôle d’accès principal et secondaire pour l’authentification.

## <a name="march-2015-release"></a><a id="march_changes_15"></a>Version de mars 2015
### <a name="general-media-services-updates"></a>Mises à jour générales de Media Services
* Media Services fournit désormais l’intégration de réseau de distribution de contenu (CDN). Pour prendre en charge cette intégration, la propriété CdnEnabled a été ajoutée à StreamingEndpoint. Vous pouvez utiliser CdnEnabled avec les API REST à compter de la version 2.9. Pour plus d’informations, consultez la rubrique [StreamingEndpoint](/rest/api/media/operations/streamingendpoint). Vous pouvez utiliser CdnEnabled avec le SDK .NET à compter de la version 3.1.0.2. Pour plus d’informations, consultez la rubrique [StreamingEndpoint](/archive/blogs/randomnumber/encoder-changes-within-azure-media-services-now-create-ismc-file).
* Media Encoder Premium Workflow a été annoncé. Pour plus d’informations, consultez [Introducing Premium Encoding in Azure Media Services (Présentation de l’encodage Premium dans Azure Media Services)](https://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services/).

## <a name="february-2015-release"></a><a id="february_changes_15"></a>Version de février 2015
### <a name="general-media-services-updates"></a>Mises à jour générales de Media Services
La dernière version de l’API REST de Media Services est la version 2.9. À compter de cette version, vous pouvez activer l’intégration de réseau de distribution de contenu (CDN) aux points de terminaison de streaming. Pour plus d’informations, consultez la rubrique [StreamingEndpoint](/rest/api/media/operations/streamingendpoint).

## <a name="january-2015-release"></a><a id="january_changes_15"></a>Version de janvier 2015
### <a name="general-media-services-updates"></a>Mises à jour générales de Media Services
La disponibilité générale de la protection du contenu avec chiffrement dynamique a été annoncée. Pour plus d’informations, consultez [Azure Media Services enhances streaming security with General Availability of DRM technology](https://azure.microsoft.com/blog/2015/01/29/azure-media-services-enhances-streaming-security-with-general-availability-of-drm-technology/).

### <a name="media-services-net-sdk-updates"></a>Mises à jour du SDK .NET Media Services
Le SDK Media Services en est maintenant à la version 3.1.0.1.

Cette version a marqué le constructeur Microsoft.WindowsAzure.MediaServices.Client.ContentKeyAuthorization.TokenRestrictionTemplate par défaut comme obsolète. Le nouveau constructeur prend TokenType comme argument.

```csharp
TokenRestrictionTemplate template = new TokenRestrictionTemplate(TokenType.SWT);
```


## <a name="december-2014-release"></a><a id="december_changes_14"></a>Version de décembre 2014
### <a name="general-media-services-updates"></a>Mises à jour générales de Media Services
* Certaines mises à jour et nouvelles fonctionnalités ont été ajoutées à Media Indexer. Pour plus d’informations, consultez les [Notes de publication d’Azure Media Indexer version 1.1.6.7](https://azure.microsoft.com/blog/2014/12/03/azure-media-indexer-version-1-1-6-7-release-notes/).
* Une nouvelle API REST permettant de mettre à jour les unités réservées d’encodage a été ajoutée. Pour plus d’informations, consultez [EncodingReservedUnitType avec REST](/rest/api/media/operations/encodingreservedunittype).
* La prise en charge de CORS a été ajoutée pour le service de remise de clé.
* Les performances d’interrogation des options de stratégie d’autorisation ont été améliorées.
* Dans le centre de données en Chine, [l’URL de remise de clé](/rest/api/media/operations/contentkey#get_delivery_service_url) fonctionne désormais par client (comme dans les autres centres de données).
* La durée cible HLS automatique a été ajoutée. Lors de la diffusion en continu, TLS est toujours empaquetée de façon dynamique. Par défaut, Media Services calcule automatiquement le coefficient d’empaquetage de segment HLS (FragmentsPerSegment) en fonction de l’intervalle d’image clé (KeyFrameInterval). On emploie également le terme de groupe d’images (GOP) reçu à partir de l’encodeur en direct. Pour plus d’informations, consultez [Vue d’ensemble du streaming en direct à l’aide d’Azure Media Services](/previous-versions/azure/dn783466(v=azure.100)).

### <a name="media-services-net-sdk-updates"></a>Mises à jour du SDK .NET Media Services
Le [SDK Media Services](https://www.nuget.org/packages/windowsazure.mediaservices/) en est maintenant à la version 3.1.0.0. Les mises à jour suivantes ont été effectuées :

* La dépendance du SDK .NET a été mise à niveau vers le .NET Framework 4.5.
* Une nouvelle API permettant de mettre à jour les unités réservées d’encodage a été ajoutée. Pour plus d’informations, consultez [Mise à jour du type d’unité réservé et augmentation des unités réservées d’encodage à l’aide de .NET](media-services-dotnet-encoding-units.md).
* La prise en charge de JSON pour l’authentification de jeton a été ajoutée. Pour plus d’informations, consultez [JWT token authentication in Media Services and dynamic encryption](http://www.gtrifonov.com/2015/01/03/jwt-token-authentication-in-azure-media-services-and-dynamic-encryption/).
* Des décalages relatifs pour BeginDate et ExpirationDate dans le modèle de licence PlayReady ont été ajoutés.

## <a name="november-2014-release"></a><a id="november_changes_14"></a>Version de novembre 2014
* Vous pouvez désormais utiliser Media Services pour ingérer du contenu Smooth Streaming (fMP4) en direct sur une connexion TLS. Pour assurer la réception via TLS, veillez à mettre à jour l’URL de réception pour HTTPS. Actuellement, Media Services ne prend pas en charge TLS avec les domaines personnalisés. Pour plus d’informations sur le streaming en direct, consultez [Vue d’ensemble du streaming en direct à l’aide d’Azure Media Services](/previous-versions/azure/dn783466(v=azure.100)).
* Actuellement, vous ne pouvez pas ingérer un flux RTMP en direct sur une connexion TLS.
* Vous ne pouvez transmettre en continu avec le protocole TLS que si le point de terminaison de streaming à partir duquel vous distribuez votre contenu a été créé après le 10 septembre 2014. Si vos URL de streaming sont basées sur des points de terminaison créés après le 10 septembre 2014, l’URL contient « streaming.mediaservices.windows.net » (le nouveau format). Les URL de streaming qui contiennent « origin.mediaservices.windows.net » (ancien format) ne sont pas compatibles avec le protocole TLS. Si votre URL suit l’ancien format et que vous souhaitez pouvoir diffuser par le biais du protocole TLS, [créez un point de terminaison de streaming](media-services-portal-manage-streaming-endpoints.md). Pour diffuser votre contenu avec le protocole TLS, utilisez des URL basées sur le nouveau point de terminaison de streaming.

### <a name="media-services-net-sdk"></a><a id="oct_sdk"></a>Kit de développement logiciel (SDK) .NET de Media Services
La dernière version des extensions du SDK Media Services pour .NET est la version 2.0.0.3.

La dernière version du SDK Media Services pour .NET est la version 3.0.0.8. Les mises à jour suivantes ont été effectuées :

* La refactorisation a été implémentée dans les classes de stratégie de nouvelle tentative.
* Une chaîne d’agent utilisateur a été ajoutée pour les en-têtes de requête HTTP.
* Une étape de génération de restauration NuGet a été ajoutée.
* Des tests de scénario ont été corrigés pour utiliser la certification x509 du dépôt.
* Des paramètres de validation ont été ajoutés en cas de mise à jour des extrémités de canal et de streaming.

### <a name="new-github-repository-to-host-media-services-samples"></a>Nouveau dépôt GitHub avec exemples Media Services
Ces exemples se trouvent dans le [dépôt GitHub d’exemples Media Services](https://github.com/Azure/Azure-Media-Services-Samples).

## <a name="september-2014-release"></a><a id="september_changes_14"></a>Version de septembre 2014
La dernière version des métadonnées REST de Media Services est la version 2.7. Pour plus d’informations sur les dernières mises à jour REST, consultez [Référence de l’API REST Media Services](/rest/api/media/operations/azure-media-services-rest-api-reference).

La dernière version du SDK Media Services pour .NET est la version 3.0.0.7.

### <a name="breaking-changes"></a><a id="sept_14_breaking_changes"></a>Dernières modifications
* Origin a été renommé [StreamingEndpoint].
* Le comportement par défaut, quand vous utilisez le portail Azure pour encoder puis publier des fichiers MP4, a changé.

### <a name="new-featuresscenarios-that-are-part-of-the-general-availability-release"></a><a id="sept_14_GA_changes"></a>Nouvelles fonctionnalités/scénarios qui font partie de la version en disponibilité générale
* Le processeur multimédia Media Indexer a été introduit. Pour plus d’informations, consultez [Indexation de fichiers multimédias avec Azure Media Indexer](/previous-versions/azure/dn783455(v=azure.100)).
* Vous pouvez utiliser l’entité [StreamingEndpoint] pour ajouter des noms de domaine (d’hôte) personnalisés.
  
    Pour utiliser un nom de domaine personnalisé comme nom de point de terminaison de streaming Media Services, ajoutez des noms d’hôte personnalisés à votre point de terminaison de streaming. Utilisez les API REST de Media Services ou le SDK .NET pour ajouter des noms d’hôte personnalisés.
  
    Les considérations suivantes s'appliquent :
  
  * Vous devez être propriétaire du nom de domaine personnalisé.
  * La propriété du nom de domaine doit être validée par Media Services. Pour valider le domaine, créez un enregistrement CName qui mappe le domaine parent MediaServicesAccountId pour vérifier DNS mediaservices-dns-zone.
  * Vous devez créer un autre enregistrement CName qui mappe le nom d’hôte personnalisé (par exemple, sports.contoso.com) au nom d’hôte de votre StreamingEndpont Media Services (par exemple, amstest.streaming.mediaservices.windows.net).

    Pour plus d’informations, consultez la propriété CustomHostNames dans l’article [StreamingEndpoint](/rest/api/media/operations/streamingendpoint).

### <a name="new-featuresscenarios-that-are-part-of-the-public-preview-release"></a><a id="sept_14_preview_changes"></a>Nouvelles fonctionnalités/nouveaux scénarios intégrés à la version préliminaire publique
* Streaming en direct (préversion). Pour plus d’informations, consultez [Vue d’ensemble du streaming en direct à l’aide d’Azure Media Services](/previous-versions/azure/dn783466(v=azure.100)).
* Service de remise de clé. Pour plus d’informations, consultez [Utilisation du chiffrement dynamique AES-128 et du service de distribution des clés](/previous-versions/azure/dn783457(v=azure.100)).
* Chiffrement dynamique AES. Pour plus d’informations, consultez [Utilisation du chiffrement dynamique AES-128 et du service de distribution des clés](/previous-versions/azure/dn783457(v=azure.100)).
* Service de remise de licence PlayReady. 
* Chiffrement dynamique PlayReady. 
* Modèle de licence PlayReady de Media Services. Pour plus d’informations, consultez [Présentation du modèle de licence PlayReady de Media Services].
* Diffuser des actifs multimédias chiffrés dans le stockage. Pour plus d’informations, consultez [Diffuser du contenu chiffré dans le stockage](/previous-versions/azure/dn783451(v=azure.100)).

## <a name="august-2014-release"></a><a id="august_changes_14"></a>Version d’août 2014
Quand vous encodez un actif multimédia, un actif multimédia de sortie est créé à la fin du travail d’encodage. Jusqu’à cette version, l’encodeur Media Services produisait des métadonnées sur les actifs multimédias de sortie. À partir de cette version, l’encodeur produit également des métadonnées sur les actifs multimédias d’entrée. Pour plus d’informations, consultez [Métadonnées d’entrée] et [Métadonnées de sortie].

## <a name="july-2014-release"></a><a id="july_changes_14"></a>Version de juillet 2014
Plusieurs bogues ont été résolus pour le gestionnaire de package et le chiffreur d'Azure Media Services :

* Quand un actif multimédia d’archive en direct est transmis à HLS, seul le son est lu : ce problème a été résolu et la lecture audio et vidéo est maintenant possible.
* Quand un actif multimédia est empaqueté avec HLS et le chiffrement d’enveloppe AES 128 bits, les flux empaquetés ne sont pas lus sur les appareils Android : Ce bogue a été résolu et le flux empaqueté est lu sur les appareils Android qui prennent en charge HLS.

## <a name="may-2014-release"></a><a id="may_changes_14"></a>Version de mai 2014
### <a name="general-media-services-updates"></a><a id="may_14_changes"></a>Mises à jour générales de Media Services
Vous pouvez maintenant utiliser [l’empaquetage dynamique] pour le streaming HLS version 3. Pour le streaming HLS version 3, ajoutez le format suivant au chemin du localisateur d’origine : * .ism/manifest(format=m3u8-aapl-v3). Pour plus d’informations, consultez [ce forum](https://social.msdn.microsoft.com/Forums/en-US/13b8a776-9519-4145-b9ed-d2b632861fde/dynamic-packaging-to-hls-v3).

Désormais, l’empaquetage dynamique prend également en charge la transmission du format HLS (version 3 et version 4) chiffré avec PlayReady sur la base du Smooth Streaming statiquement chiffré avec PlayReady. Pour plus d’informations sur la façon de chiffrer Smooth Streaming avec PlayReady, consultez [Protéger Smooth Streaming avec PlayReady](/previous-versions/azure/dn189154(v=azure.100)).

### <a name="media-services-net-sdk-updates"></a><a name="may_14_donnet_changes"></a>Mises à jour du SDK .NET Media Services
Le SDK Media Services en est maintenant à la version 3.0.0.5. Les mises à jour suivantes ont été effectuées :

* La vitesse et la résilience sont meilleures quand vous chargez et téléchargez des fichiers multimédias.
* Des améliorations ont été apportées à la logique de nouvelle tentative et à la gestion des exceptions temporaires : 
  
  * La détection des erreurs temporaires et la logique de nouvelle tentative ont été améliorées pour les exceptions déclenchées quand vous interrogez, enregistrez des modifications et chargez ou téléchargez des fichiers. 
  * Quand vous recevez des exceptions web (par exemple lors d’une demande de jeton Access Control Service), les erreurs irrécupérables échouent plus rapidement maintenant.

Pour plus d’informations, consultez [Logique de nouvelle tentative dans le SDK Media Services pour .NET].

## <a name="januaryfebruary-2014-releases"></a><a id="jan_feb_changes_14"></a>Versions de janvier/février 2014
### <a name="media-services-net-sdk-3001-3002-and-3003"></a><a name="jan_fab_14_donnet_changes"></a>SDK .NET Media Services 3.0.0.1, 3.0.0.2 et 3.0.0.3
Les modifications introduites dans les versions 3.0.0.1 et 3.0.0.2 sont les suivantes :

* Des problèmes concernant l’utilisation de requêtes LINQ avec des instructions OrderBy ont été résolus.
* Des solutions de test dans [GitHub] ont été divisées en tests unitaires et tests basés sur des scénarios.

Pour plus d’informations sur les modifications, consultez le [SDK Media Services pour .NET 3.0.0.1 et 3.0.0.2](http://gtrifonov.com/2014/02/07/windows-azure-media-services-net-sdk-3-0-0-2-release/index.html).

Les modifications suivantes ont été apportées à la version 3.0.0.3 :

* Les dépendances du stockage Azure ont été mises à niveau pour utiliser la version 3.0.3.0.
* Un problème de compatibilité descendante a été résolu pour les versions 3.0. *.* .

## <a name="december-2013-release"></a><a id="december_changes_13"></a>Version de décembre 2013
### <a name="media-services-net-sdk-3000"></a><a name="dec_13_donnet_changes"></a>SDK .NET Media Services 3.0.0.0
> [!NOTE]
> Les versions 3.0.x.x ne présentent pas de compatibilité descendante avec les versions 2.4.x.x.
> 
> 

La dernière version du Kit de développement logiciel (SDK) Media Services est maintenant la version 3.0.0.0.0. Vous pouvez télécharger le dernier package à partir de NuGet ou obtenir les différents composants sur [GitHub].

À compter de version 3.0.0.0 du SDK Media Services, vous pouvez réutiliser les jetons [Azure AD Access Control Service](/previous-versions/azure/azure-services/hh147631(v=azure.100)). Pour plus d’informations, consultez la section « Réutiliser des jetons Access Control Service » dans [Se connecter à Media Services à l’aide du SDK Media Services pour .NET](/previous-versions/azure/jj129571(v=azure.100)).

### <a name="media-services-net-sdk-extensions-2000"></a><a name="dec_13_donnet_ext_changes"></a>Extensions du SDK Media Services pour .NET 2.0.0.0
 Les extensions du SDK Media Services pour .NET sont un ensemble de méthodes d’extension et de fonctions d’assistance qui simplifient votre code et le développement avec Media Services. Pour obtenir les dernières informations disponibles, consultez [Extensions du SDK Media Services pour .NET](https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev).

## <a name="november-2013-release"></a><a id="november_changes_13"></a>Version de novembre 2013
### <a name="media-services-net-sdk-changes"></a><a name="nov_13_donnet_changes"></a>Modifications apportées au SDK Media Services pour .NET
À compter de cette version, le SDK Media Services pour .NET gère toutes les erreurs temporaires qui peuvent se produire lors des appels adressés à la couche API REST de Media Services.

## <a name="august-2013-release"></a><a id="august_changes_13"></a>Version d’août 2013
### <a name="media-services-powershell-cmdlets-included-in-azure-sdk-tools"></a><a name="aug_13_powershell_changes"></a>Applets de commande PowerShell Media Services incluses dans les outils du SDK Azure
Les applets de commande PowerShell Media Services suivantes sont désormais incluses dans les [outils du SDK Azure](https://github.com/Azure/azure-sdk-tools) :

* Get-AzureMediaServices 

    Par exemple : `Get-AzureMediaServicesAccount`
* New-AzureMediaServicesAccount 
  
    Par exemple : `New-AzureMediaServicesAccount -Name "MediaAccountName" -Location "Region" -StorageAccountName "StorageAccountName"`
* New-AzureMediaServicesKey 
  
    Par exemple : `New-AzureMediaServicesKey -Name "MediaAccountName" -KeyType Secondary -Force`
* Remove-AzureMediaServicesAccount 
  
    Par exemple : `Remove-AzureMediaServicesAccount -Name "MediaAccountName" -Force`

## <a name="june-2013-release"></a><a id="june_changes_13"></a>Version de juin 2013
### <a name="media-services-changes"></a><a name="june_13_general_changes"></a>Modifications apportées à Media Services
Les changements suivants mentionnés dans cette section correspondent aux mises à jour incluses dans les versions de Media Services de juin 2013 :

* Possibilité de lier plusieurs comptes de stockage à un compte Media Service. 
    * StorageAccount
    * Asset.StorageAccountName and Asset.StorageAccount
* Possibilité de mettre à jour Job.Priority. 
* Entités et propriétés liées aux notifications : 
    * JobNotificationSubscription
    * NotificationEndPoint
    * Travail
* Asset.Uri 
* Locator.Name 

### <a name="media-services-net-sdk-changes"></a><a name="june_13_dotnet_changes"></a>Modifications apportées au SDK Media Services pour .NET
Les modifications suivantes sont incluses dans les versions du SDK Media Services de juin 2013. Le dernier Kit de développement logiciel (SDK) Media Services est disponible sur GitHub.

* À partir de la version 2.3.0.0, le SDK Media Services prend en charge la liaison de plusieurs comptes de stockage à un compte Media Services. Les API suivantes prennent en charge cette fonctionnalité :
  
    * Type IStorageAccount
    * Propriété Microsoft.WindowsAzure.MediaServices.Client.CloudMediaContext.StorageAccounts
    * Propriété StorageAccount
    * Propriété StorageAccountName
  
      Pour plus d’informations, consultez [Gérer les actifs Media Services sur plusieurs comptes de stockage](/previous-versions/azure/dn271889(v=azure.100)).
* API liées aux notifications. À compter de la version 2.2.0.0, vous pouvez écouter les notifications du service de stockage de files d’attente Azure. Pour plus d’informations, consultez [Gérer les notifications de travaux de Media Services](/previous-versions/azure/dn261241(v=azure.100)).
  
    * Propriété Microsoft.WindowsAzure.MediaServices.Client.IJob.JobNotificationSubscriptions
    * Type Microsoft.WindowsAzure.MediaServices.Client.INotificationEndPoint
    * Type Microsoft.WindowsAzure.MediaServices.Client.IJobNotificationSubscription
    * Type Microsoft.WindowsAzure.MediaServices.Client.NotificationEndPointCollection
    * Type Microsoft.WindowsAzure.MediaServices.Client.NotificationEndPointType
* Dépendance par rapport au SDK du client Storage 2.0 (Microsoft.WindowsAzure.StorageClient.dll)
* Dépendance par rapport à OData 5.5 (Microsoft.Data.OData.dll)

## <a name="december-2012-release"></a><a id="december_changes_12"></a>Version de décembre 2012
### <a name="media-services-net-sdk-changes"></a><a name="dec_12_dotnet_changes"></a>Modifications apportées au SDK Media Services pour .NET
* IntelliSense : la documentation IntelliSense manquante a été ajoutée pour de nombreux types.
* Microsoft.Practices.TransientFaultHandling.Core : résolution d’un problème dû au fait que le SDK dépendait encore d’une ancienne version de cet assembly. Il fait dorénavant référence à la version 5.1.1209.1 de cet assembly.

Correctifs pour les problèmes détectés dans le Kit de développement logiciel (SDK) de novembre 2012 :

* IAsset.Locators.Count : ce nombre est maintenant indiqué correctement sur les nouvelles interfaces IAsset après la suppression de tous les localisateurs.
* IAssetFile.ContentFileSize : cette valeur est maintenant définie correctement après un chargement par IAssetFile.Upload(filepath).
* IAssetFile.ContentFileSize : cette propriété peut désormais être définie pendant la création d’un fichier d’actifs. Elle était auparavant en lecture seule.
* IAssetFile.Upload(filepath) : un problème a été résolu avec cette méthode de chargement synchrone qui générait l’erreur suivante quand plusieurs fichiers étaient chargés sur l’actif. « Le serveur n'a pas pu authentifier la demande. Vérifiez que la valeur de l'en-tête d'autorisation est formée correctement, avec la signature. »
* IAssetFile.UploadAsync : un problème qui limitait à cinq le nombre de fichiers pouvant être chargés simultanément a été résolu.
* IAssetFile.UploadProgressChanged : cet événement est désormais fourni par le kit SDK.
* IAssetFile.DownloadAsync(string, BlobTransferClient, ILocator, CancellationToken) : cette surcharge de méthode est désormais fournie.
* IAssetFile.DownloadAsync : un problème qui limitait à cinq le nombre de fichiers pouvant être téléchargés simultanément a été résolu.
* IAssetFile.Delete() : résolution d’un problème concernant l’appel de la méthode delete, qui pouvait lever une exception si aucun fichier n’était chargé pour IAssetFile.
* Travaux  : résolution d’un problème concernant le chaînage d’une « tâche MP4 vers Smooth Streams » avec une « tâche de protection PlayReady » à l’aide d’un modèle de travail, qui n’aboutissait à la création d’aucune tâche.
* EncryptionUtils.GetCertificateFromStore() : cette méthode ne lève plus une exception de référence null à la suite de l’échec de la recherche du certificat en raison de problèmes de configuration du certificat.

## <a name="november-2012-release"></a><a id="november_changes_12"></a>Version de novembre 2012
Les changements mentionnés dans cette section étaient des mises à jour incluses dans le Kit de développement logiciel (SDK) de novembre 2012 (version 2.0.0.0). Ces changements peuvent nécessiter une modification ou une réécriture du code écrit pour la préversion du SDK de juin 2012.

* Éléments multimédias
  
    * IAsset.Create(assetName) est la *seule* fonction de création d’actifs multimédias. IAsset.Create ne télécharge plus les fichiers dans le cadre de l'appel de la méthode. Utilisez IAssetFile pour le téléchargement.
    * La méthode IAsset.Publish et la valeur d’énumération AssetState.Publish ont été supprimées du SDK Services. Tous les codes qui dépendent de cette valeur doivent être réécrits.
* FileInfo
  
    * Cette classe a été supprimée et remplacée par IAssetFile.
  
* IAssetFiles
  
    * IAssetFile remplace FileInfo et présente un comportement différent. Pour l’utiliser, vous devez instancier l’objet IAssetFiles, puis effectuer un téléchargement de fichier à l’aide du SDK Media Services ou du SDK Storage. Les surcharges IAssetFile.Upload suivantes peuvent être utilisées :
  
        * IAssetFile.Upload(filePath) : cette méthode synchrone bloque le thread. Nous vous recommandons de l’utiliser uniquement quand vous chargez un seul fichier.
        * IAssetFile.UploadAsync(filePath, blobTransferClient, locator, cancellationToken) : cette méthode asynchrone est le mécanisme de chargement privilégié. 
    
            Bogue connu : si vous utilisez le jeton d’annulation, le chargement est annulé. Les tâches peuvent avoir de nombreux états d’annulation. Vous devez saisir et traiter les exceptions correctement.
* Localisateurs
  
    * Les versions propres à l’origine ont été supprimées. L’élément context.Locators.CreateSasLocator (asset, accessPolicy) propre aux associations de sécurité sera marqué comme déprécié ou supprimé dès la mise à disposition générale. Consultez la section relative aux localisateurs sous « Nouvelles fonctionnalités » pour plus d’informations sur le nouveau comportement.

## <a name="june-2012-preview-release"></a><a id="june_changes_12"></a>Préversion de juin 2012
La fonctionnalité suivante est une nouveauté de la version de novembre du SDK :

* Suppression des entités
  
    * Les objets IAsset, IAssetFile, ILocator, IAccessPolicy, IContentKey sont désormais supprimés au niveau de l’objet, c’est-à-dire IObject.Delete(), au lieu d’exiger une suppression dans la Collection, c’est-à-dire cloudMediaContext.ObjCollection.Delete(objInstance).
* Localisateurs
  
    * Vous devez désormais créer les localisateurs à l’aide de la méthode CreateLocator. Ils doivent utiliser les valeurs d’énumération LocatorType.SAS ou LocatorType.OnDemandOrigin comme argument pour le type particulier de localisateur que vous souhaitez créer.
    * De nouvelles propriétés ont été ajoutées aux localisateurs afin de faciliter l’obtention d’URI utilisables pour votre contenu. Cette nouvelle conception des localisateurs fournit une plus grande flexibilité pour une extensibilité tierce ultérieure, et simplifie l’utilisation pour les applications clientes multimédias.
* Prise en charge de la méthode asynchrone
  
    * La prise en charge asynchrone a été ajoutée à toutes les méthodes.

## <a name="additional-notes"></a>Remarques supplémentaires

* Widevine est un service fourni par Google Inc. soumis aux conditions de service et à la politique de confidentialité de Google, Inc.

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

<!-- Anchors. -->

<!-- Images. -->

<!--- URLs. --->
[Microsoft Q&A question page for Azure Media Services]: /answers/topics/azure-media-services.html
[Référence de l’API REST d’Azure Media Services]: /rest/api/media/operations/azure-media-services-rest-api-reference
[Media Services pricing details]: https://azure.microsoft.com/pricing/details/media-services/
[Métadonnées d’entrée]: ./media-services-input-metadata-schema.md
[Métadonnées de sortie]: ./media-services-output-metadata-schema.md
[Deliver content]: /previous-versions/azure/hh973618(v=azure.100)
[Index media files with the Azure Media Indexer]: /previous-versions/azure/dn783455(v=azure.100)
[StreamingEndpoint]: /rest/api/media/operations/streamingendpoint
[Work with Media Services live streaming]: /previous-versions/azure/dn783466(v=azure.100)
[Use AES-128 dynamic encryption and the key delivery service]: /previous-versions/azure/dn783457(v=azure.100)
[Use PlayReady dynamic encryption and the license delivery service]: /previous-versions/azure/dn783467(v=azure.100)
[Preview features]: https://azure.microsoft.com/services/preview/
[Présentation du modèle de licence PlayReady de Media Services]: /previous-versions/azure/dn783459(v=azure.100)
[Stream storage-encrypted content]: /previous-versions/azure/dn783451(v=azure.100)
[Azure portal]: https://portal.azure.com
[l’empaquetage dynamique]: /previous-versions/azure/jj889436(v=azure.100)
[Nick Drouin's blog]: http://blog-ndrouin.azurewebsites.net/hls-v3-new-old-thing/
[Protect Smooth Streaming with PlayReady]: /previous-versions/azure/dn189154(v=azure.100)
[Logique de nouvelle tentative dans le SDK Media Services pour .NET]: ./media-services-retry-logic-in-dotnet-sdk.md
[Grass Valley announces EDIUS 7 streaming through the cloud]: https://www.streamingmedia.com/Producer/Articles/ReadArticle.aspx?ArticleID=96351&utm_source=dlvr.it&utm_medium=twitter
[Control Media Services Encoder output file names]: /previous-versions/azure/dn303341(v=azure.100)
[Create overlays]: /previous-versions/azure/dn640496(v=azure.100)
[Stitch video segments]: /previous-versions/azure/dn640504(v=azure.100)
[Azure Media Services .NET SDK 3.0.0.1 and 3.0.0.2 releases]: http://www.gtrifonov.com/2014/02/07/windows-azure-media-services-.net-sdk-3.0.0.2-release/
[Azure AD Access Control Service]: /previous-versions/azure/azure-services/hh147631(v=azure.100)
[Connect to Media Services with the Media Services SDK for .NET]: /previous-versions/azure/jj129571(v=azure.100)
[Media Services .NET SDK extensions]: https://github.com/Azure/azure-sdk-for-media-services-extensions/tree/dev
[Azure SDK tools]: https://github.com/Azure/azure-sdk-tools
[GitHub]: https://github.com/Azure/azure-sdk-for-media-services
[Manage Media Services assets across multiple Storage accounts]: /previous-versions/azure/dn271889(v=azure.100)
[Handle Media Services job notifications]: /previous-versions/azure/dn261241(v=azure.100)