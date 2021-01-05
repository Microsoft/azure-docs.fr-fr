---
title: Questions fréquentes (FAQ) sur Video Indexer - Azure
titleSuffix: Azure Media Services
description: Cet article donne des réponses aux questions les plus fréquemment posées sur Azure Media Services Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 05/12/2020
ms.author: juliako
ms.openlocfilehash: 721af069a76497576b92c4203a93656d19c283aa
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92217726"
---
# <a name="video-indexer-frequently-asked-questions"></a>Forum Aux Questions sur Video Indexer

Cet article répond aux questions fréquentes sur Video Indexer.

## <a name="general-questions"></a>Questions générales

### <a name="what-is-video-indexer"></a>Qu’est-ce que Video Indexer ?

Video Indexer est un service d’intelligence artificielle qui fait partie de Microsoft Azure Media Services. Video Indexer offre une orchestration de plusieurs modèles Machine Learning qui vous permettent d’extraire facilement un insight recherché à partir d’une vidéo. Pour fournir des insights avancés et précis, Video Indexer se sert de plusieurs canaux de la vidéo : l’audio, la voix et le visuel. Les insights de Video Indexer peuvent être utilisés de bien des façons, par exemple pour améliorer la découvertibilité et l’accessibilité de contenus, concevoir de nouvelles opportunités de monétisation ou créer de nouvelles expériences qui exploitent les insights. Video Indexer fournit une interface web pour les tests, la configuration et la personnalisation de modèles dans votre compte. Les développeurs peuvent utiliser une API REST pour intégrer Video Indexer à un système de production. 

### <a name="what-can-i-do-with-video-indexer"></a>Que puis-je faire avec Video Indexer ?

Voici certaines opérations que Video Indexer peut effectuer sur les fichiers multimédias :

* Identification et extraction de la voix et identification des intervenants.
* Identification et extraction du texte à l’écran dans une vidéo.
* Détection d’objets dans un fichier vidéo.
* Identification des marques (par exemple : Microsoft) dans les pistes audio et le texte à l’écran dans une vidéo.
* Détection et identification de visages à partir d’une base de données de célébrités et d’une base de données de visages définie par l’utilisateur.
* Extraction des sujets abordés, mais pas nécessairement mentionnés dans le contenu audio et vidéo.
* Création de sous-titres à partir d’une piste audio.

Pour obtenir plus d’informations et découvrir d’autres fonctionnalités Video Indexer, consultez [Vue d’ensemble](video-indexer-overview.md).

### <a name="how-do-i-get-started-with-video-indexer"></a>Comment bien démarrer avec Video Indexer ?

Video Indexer comprend une offre d’essai gratuit qui vous attribue 600 minutes dans l’interface web et 2 400 minutes via l’API. Vous pouvez vous [connecter à l’interface web de Video Indexer](https://www.videoindexer.ai/) et l’essayer par vous-même à l’aide d’une simple identité web et sans avoir à configurer un abonnement Azure. Suivez [ce lab de présentation facile](https://github.com/Azure-Samples/media-services-video-indexer/blob/master/IntroToVideoIndexer.md) pour mieux comprendre comment utiliser Video Indexer.

Pour indexer des fichiers vidéo et audio à grande échelle, vous pouvez connecter Video Indexer à un abonnement payant Microsoft Azure. Pour plus d’informations sur les prix, consultez la [page sur la tarification](https://azure.microsoft.com/pricing/details/cognitive-services/video-indexer/).

Pour plus d’informations pour bien démarrer, consultez la [page correspondante](video-indexer-get-started.md).

### <a name="do-i-need-coding-skills-to-use-video-indexer"></a>Ai-je besoin de compétences de codage pour utiliser Video Indexer ?

Vous pouvez utiliser l’interface web Video Indexer pour évaluer, configurer et gérer votre compte **sans codage nécessaire**.  Lorsque vous êtes prêt à développer des applications plus complexes, vous pouvez utiliser l’[API Video Indexer](https://api-portal.videoindexer.ai/) pour intégrer Video Indexer à vos propres applications, sites web ou [workflows personnalisés à l’aide de technologies serverless, telles que Azure Logic Apps](https://azure.microsoft.com/blog/logic-apps-flow-connectors-will-make-automating-video-indexer-simpler-than-ever/) ou Azure Functions.

### <a name="do-i-need-machine-learning-skills-to-use-video-indexer"></a>Ai-je besoin de compétences Machine Learning pour utiliser Video Indexer ?

Non, Video Indexer fournit l’intégration de plusieurs modèles Machine Learning dans un pipeline. L’indexation d’un fichier vidéo ou audio par l’intermédiaire de Video Indexer récupère un ensemble complet d’insights extraits d’une chronologie partagée, sans qu’aucune compétence en machine learning ni connaissance des algorithmes ne soit nécessaire de la part du client.

### <a name="what-media-formats-does-video-indexer-support"></a>Quels formats multimédias sont pris en charge par Video Indexer ?

Video Indexer prend en charge les formats multimédias les plus courants. Reportez-vous à la liste des [formats standard Azure Media Encoder](../latest/media-encoder-standard-formats.md) pour plus d’informations.

### <a name="how-do-i-upload-a-media-file-into-video-indexer-and-what-are-the-limitations"></a>Comment charger un fichier multimédia dans Video Indexer et quelles sont les limitations ?

Dans le portail web Video Indexer, vous pouvez charger un fichier multimédia par le biais de la boîte de dialogue de chargement de fichier, ou en faisant pointer vers une URL qui héberge directement le fichier source (voir l’[exemple](https://nimbuscdn-nimbuspm.streaming.mediaservices.windows.net/2b533311-b215-4409-80af-529c3e853622/Ignite-short.mp4)). Une URL qui héberge le contenu multimédia à l’aide d’un iFrame ou d’un code incorporé ne fonctionne pas (voir l’[exemple](https://www.videoindexer.ai/accounts/7e1282e8-083c-46ab-8c20-84cae3dc289d/videos/5cfa29e152/?t=4.11)). 

Pour plus d’informations, consultez ce [guide pratique](./upload-index-videos.md).

#### <a name="limitations"></a>Limites

* Le nom de la vidéo ne peut pas dépasser 80 caractères.
* Si vous téléchargez une vidéo à l’aide d’un tableau d’octets, la taille de la vidéo est limitée à 2 Go (et à 30 Go lors de l’utilisation d’une URL). 

Pour obtenir une liste complète, consultez [Considérations et limitations relatives au chargement](upload-index-videos.md#uploading-considerations-and-limitations).

### <a name="how-long-does-it-take-video-indexer-to-extract-insights-from-media"></a>Combien de temps faut-il à Video Indexer pour extraire des insights à partir d’un fichier multimédia ?

Le temps nécessaire pour indexer un fichier vidéo ou audio, que ce soit à l’aide de l’API Video Indexer ou de l’interface web Video Indexer, dépend de plusieurs paramètres, tels que la longueur et la qualité du fichier, le nombre d’insights trouvés dans le fichier, le nombre d’[unités réservées](../previous/media-services-scale-media-processing-overview.md) disponibles et l’activation ou non du [point de terminaison de streaming](../previous/media-services-streaming-endpoints-overview.md). Nous vous recommandons d’exécuter quelques fichiers de test avec votre propre contenu et de prendre une moyenne pour obtenir une meilleure idée.

### <a name="can-i-create-customized-workflows-to-automate-processes-with-video-indexer"></a>Puis-je créer des workflows personnalisés pour automatiser des processus avec Video Indexer ?

Oui, vous pouvez intégrer Video Indexer à des technologies serverless, telles que Logic Apps, Flow et [Azure Functions](https://azure.microsoft.com/services/functions/). De plus amples informations sur les connecteurs [Logic App](https://azure.microsoft.com/services/logic-apps/) et [Flow](https://flow.microsoft.com/en-us/) de Video Indexer sont à votre disposition [ici](https://azure.microsoft.com/blog/logic-apps-flow-connectors-will-make-automating-video-indexer-simpler-than-ever/). Vous pouvez voir des projets d’automatisation effectués par les partenaires dans le référentiel [Exemples Video Indexer](https://github.com/Azure-Samples/media-services-video-indexer).

### <a name="in-which-azure-regions-is-video-indexer-available"></a>Dans quelles régions Azure Video Indexer est-il disponible ?

Vous pouvez voir les régions Azure dans lesquelles Video Indexer est disponible sur la page consacrée aux [régions](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services&regions=all).

### <a name="can-i-customize-video-indexer-models-for-my-specific-use-case"></a>Puis-je personnaliser des modèles Video Indexer pour mon cas d’usage spécifique ? 

Oui. Dans Video Indexer, vous pouvez personnaliser certains des modèles disponibles pour mieux répondre à vos besoins. 

Par exemple, notre modèle Personne prend en charge 1 000 000 visages de célébrités par défaut, mais vous pouvez également le former à reconnaître d’autres visages qui ne figurent pas dans cette base de données. 

Pour plus d’informations, consultez les articles sur la personnalisation des modèles [Personne](customize-person-model-overview.md), [Marques](customize-brands-model-overview.md) et [Langue](customize-language-model-overview.md). 

###  <a name="can-i-edit-the-videos-in-my-library"></a>Puis-je modifier les vidéos de ma bibliothèque ?

Oui. Appuyez sur le bouton **Modifier la vidéo** à partir de la vue de la bibliothèque, ou sur le bouton **Ouvrir dans l’éditeur** à partir de la vue du lecteur pour accéder à l’onglet **Projets**. Vous pouvez créer un nouveau projet et ajouter d’autres vidéos à partir de votre bibliothèque pour les modifier. Une fois que vous avez terminé, vous pouvez afficher votre vidéo et la télécharger. 

Si vous souhaitez obtenir des insights sur votre nouvelle vidéo, indexez-la avec Video Indexer et elle apparaîtra dans votre bibliothèque avec ses insights.

### <a name="can-i-index-multiple-audio-streams-or-channels"></a>Puis-je indexer plusieurs flux ou canaux audio ?

S’il existe plusieurs flux audio, Video Indexer prend le premier qu’il rencontre et ne traite que celui-là. Dans n’importe quel flux audio qu’il traite, Video Indexer prend les différents canaux (s’il y en a) et les traite ensemble comme mono. Pour la manipulation de flux/canaux, vous pouvez utiliser des commandes ffmpeg sur le fichier avant de l’indexer.

### <a name="what-is-the-sla-for-video-indexer"></a>Quel est le contrat SLA pour Video Indexer ?

Le contrat SLA d’Azure Media Service couvre Video Indexer et se trouve dans la page [SLA](https://azure.microsoft.com/support/legal/sla/media-services/v1_2/). Le contrat SLA ne s’applique qu’aux comptes payants Video Indexer et ne s’applique pas à l’essai gratuit.

## <a name="privacy-questions"></a>Questions relatives à la confidentialité

### <a name="are-video-and-audio-files-indexed-by-video-indexer-stored"></a>Les fichiers vidéo et audio indexés par Video Indexer sont-ils stockés ?

Oui, sauf si vous les supprimez de Video Indexer au moyen de l’API ou du site web Video Indexer, vos fichiers vidéo et audio sont stockés. Dans le cadre de l’essai gratuit, les fichiers vidéo et audio que vous indexez sont stockés dans la région Azure USA Est. Sinon, vos fichiers vidéos et audio sont stockés dans le compte de stockage de votre abonnement Azure.

### <a name="can-i-delete-my-files-that-are-stored-in-video-indexer-portal"></a>Puis-je supprimer mes fichiers qui sont stockés dans le portail Video Indexer ?

Oui. Vous pouvez toujours supprimer vos fichiers vidéo et audio, ainsi que les métadonnées et les insights qui ont été extraits par Video Indexer. Lorsque vous supprimez un fichier à partir de Video Indexer, le fichier, ses métadonnées et ses insights sont définitivement supprimés de Video Indexer. Toutefois, si vous avez implémenté votre propre solution de sauvegarde dans le stockage Azure, le fichier demeure dans votre stockage Azure.

### <a name="can-i-control-user-access-to-my-video-indexer-account"></a>Puis-je contrôler l’accès des utilisateurs à mon compte Video Indexer ?

Oui, seuls les administrateurs de compte peuvent inviter ou ne pas inviter des utilisateurs sur leurs comptes, et décider également de qui dispose de privilèges de modification ou d’un accès en lecture seule.

### <a name="who-has-access-to-my-video-and-audio-files-that-have-been-indexed-andor-stored-by-video-indexer-and-the-metadata-and-insights-that-were-extracted"></a>Qui a accès à mes fichiers vidéo et audio indexés et/ou stockés par Video Indexer, ainsi qu’aux métadonnées et aux insights qui ont été extraits ?

Votre contenu vidéo ou audio, pour lequel vous avez défini le paramètre de confidentialité sur Public, est accessible par toute personne qui dispose du lien vers votre contenu vidéo ou audio ainsi qu’à ses insights. Votre contenu vidéo ou audio, pour lequel vous avez défini le paramètre de confidentialité sur Privé, est uniquement accessible par les utilisateurs qui ont été invités à participer au compte associé au contenu vidéo ou audio. Le paramètre de confidentialité de votre contenu s’applique également aux métadonnées et aux insights que Video Indexer extrait. Vous affectez le paramètre de confidentialité lorsque vous chargez votre fichier vidéo ou audio. Vous pouvez également modifier le paramètre de confidentialité après l’indexation.

### <a name="what-access-does-microsoft-have-to-my-video-or-audio-files-that-have-been-indexed-andor-stored-by-video-indexer-and-the-metadata-and-insights-that-were-extracted"></a>Quel accès Microsoft a-t-il à mes fichiers vidéo ou audio indexés et/ou stockés par Video Indexer, et aux métadonnées et insights qui ont été extraits ?

Selon les [Conditions d’utilisation des services en ligne Azure](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=31) (OST), votre contenu vous appartient en intégralité, et Microsoft accède uniquement à ce contenu ainsi qu’aux métadonnées et aux insights que Video Indexer extrait à partir de votre contenu, conformément aux Conditions d’utilisation des services en ligne et à la Déclaration de confidentialité de Microsoft.

### <a name="are-the-custom-models-that-i-build-in-my-video-indexer-account-available-to-other-accounts"></a>Les modèles personnalisés que je crée dans sur mon compte Video Indexer sont-ils disponibles pour d'autres comptes ?

 Non, les modèles personnalisés que vous créez sur votre compte ne sont disponibles pour aucun autre compte. Actuellement, Video Indexer vous permet de créer les modèles personnalisés suivants sur votre compte : [marques](customize-brands-model-overview.md), [langue](customize-language-model-overview.md) et [personne](customize-person-model-overview.md). Ces modèles sont uniquement disponibles sur le compte sur lequel vous les avez créés.
  
### <a name="is-the-content-indexed-by-video-indexer-kept-within-the-azure-region-where-i-am-using-video-indexer"></a>Le contenu indexé par Video Indexer est-il conservé au sein de la région Azure où j’utilise Video Indexer ?

Oui. Le contenu et les insights associés sont conservés dans la région Azure, sauf s’il existe une configuration manuelle dans votre abonnement Azure qui utilise plusieurs régions Azure. 

### <a name="what-is-the-privacy-policy-for-video-indexer"></a>Quelle est la politique de confidentialité Video Indexer ?

Video Indexer est couvert par la [Déclaration de confidentialité Microsoft](https://privacy.microsoft.com/privacystatement). La Déclaration de confidentialité décrit les données personnelles que Microsoft traite, comment et dans quel but Microsoft les traite. Pour en savoir plus sur la confidentialité, visitez le site [Microsoft Trust Center](https://www.microsoft.com/trustcenter).

### <a name="what-certifications-does-video-indexer-have"></a>De quelles certifications le service Video Indexer dispose-t-il ?

Actuellement, Video Indexer a la certification SOC. Pour consulter la certification de Video Indexer, reportez-vous au site [Microsoft Trust Center](https://www.microsoft.com/trustcenter/compliance/complianceofferings?product=Azure).

### <a name="what-is-the-difference-between-private-and-public-videos"></a>Quelle est la différence entre les vidéos privées et publiques ? 

Lorsque des vidéos sont chargées dans Video Indexer, vous pouvez choisir entre deux paramètres de confidentialité : privé et public. Les vidéos publiques sont accessibles à tous, y compris aux utilisateurs anonymes et non identifiés. Les vidéos privées sont réservées aux membres du compte. 

### <a name="i-tried-to-upload-a-video-as-public-and-it-was-flagged-for-inappropriate-or-offensive-content-what-does-that-mean"></a>J'ai essayé de charger une vidéo en tant que vidéo publique, et son contenu a été signalé comme inapproprié ou offensant. Qu'est-ce que cela signifie ? 

Lors du chargement d'une vidéo dans Video Indexer, une analyse automatique du contenu est effectuée par des algorithmes et des modèles afin de s'assurer qu'aucun contenu inapproprié ne sera diffusé publiquement. Si une vidéo est jugée suspecte car elle contient un contenu explicite, vous ne pourrez pas la définir comme étant publique. Mais les membres du compte peuvent toujours y accéder en tant que vidéo privée (la visionner, télécharger les aperçus et les artefacts extraits, et effectuer d'autres opérations réservées aux membres du compte).   

Pour rendre la vidéo accessible au public, vous pouvez soit : 

* Créer votre propre couche d'interface (par exemple une application ou un site web) et l’utiliser pour interagir avec le service Video Indexer. De cette façon, la vidéo reste privée dans notre portail et vos utilisateurs peuvent interagir avec elle via votre interface. Par exemple, vous pouvez toujours obtenir des informations ou autoriser le visionnage de la vidéo dans votre propre interface. 
* Demander un examen humain du contenu, ce qui entraînerait la suppression de la restriction, en supposant que le contenu n'est pas explicite. 

    Cette option peut être envisagée si le site web Video Indexer est utilisé directement par vos utilisateurs comme couche d'interface, et pour un visionnage public (non authentifié). 

## <a name="api-questions"></a>Questions sur l’API

### <a name="what-apis-does-video-indexer-offer"></a>Quelles sont les API offertes par Video Indexer ?

Les API de Video Indexer permettent l’indexation, l’extraction de métadonnées, la gestion des ressources, la translation, l’incorporation, la personnalisation de modèles, et bien plus encore. Des informations détaillées sur l’utilisation de l’API Video Indexer sont disponibles sur le portail [Video Indexer Developer Portal](https://api-portal.videoindexer.ai/).

### <a name="what-client-sdks-does-video-indexer-offer"></a>Quels sont les kits SDK proposés par Video Indexer ?

Il n’y a aucun kit SDK client disponible actuellement. L’équipe de Video Indexer travaille en ce moment à l’élaboration des kits SDK et prévoit de les diffuser prochainement.

### <a name="how-do-i-get-started-with-video-indexers-api"></a>Comment bien démarrer avec l’API Video Indexer ?

Suivez le [tutoriel sur la prise en main de l’API Video Indexer](video-indexer-use-apis.md).

### <a name="what-is-the-difference-between-the-video-indexer-api-and-the-azure-media-service-v3-api"></a>Quelle différence y a-t-il entre l’API Video Indexer et l’API Azure Media Services v3 ?

Actuellement, des fonctionnalités sont disponibles à la fois dans l’API Video Indexer et l’API Azure Media Services v3. Des informations supplémentaires sur la façon de comparer ces deux services sont disponibles [ici](compare-video-indexer-with-media-services-presets.md).

### <a name="what-is-an-api-access-token-and-why-do-i-need-it"></a>Qu’est-ce qu’un jeton d’accès API et pourquoi en ai-je besoin ?

L’API Video Indexer contient une API Autorisations et une API Opérations. L’API Autorisations contient des appels qui vous donnent un jeton d’accès. Chaque appel à l’API Opérations doit être associé à un jeton d’accès correspondant à l’étendue d’autorisation de l’appel.

Les jetons d’accès sont nécessaires pour utiliser les API Video Indexer pour des raisons de sécurité. Cela garantit que tous les appels proviennent de vous ou des personnes qui ont des autorisations d’accès à votre compte. 

### <a name="what-is-the-difference-between-account-access-token-user-access-token-and-video-access-token"></a>Quelle est la différence entre le jeton d’accès Compte, le jeton d’accès Utilisateur et le jeton d’accès Vidéo ?

* Niveau Compte : les jetons d’accès de niveau compte vous permettent d’effectuer des opérations au niveau du compte ou de la vidéo. Par exemple, charger une vidéo, lister toutes les vidéos, obtenir des insights de vidéo.
* Niveau Utilisateur : les jetons d’accès de niveau utilisateur vous permettent d’effectuer des opérations au niveau de l’utilisateur. Par exemple, obtenir les comptes associés.
* Niveau Vidéo : les jetons d’accès de niveau vidéo vous permettent d’effectuer des opérations au niveau d’une vidéo spécifique. Par exemple, obtenir des aperçus de la vidéo, télécharger les sous-titres, obtenir des widgets, etc.

### <a name="how-often-do-i-need-to-get-a-new-access-token-when-do-access-tokens-expire"></a>À quelle fréquence dois-je demander un nouveau jeton d’accès ? Quand les jetons d’accès expirent-ils ?

Les jetons d’accès expirent toutes les heures. Vous devez donc générer un nouveau jeton d’accès chaque heure. 

### <a name="what-are-the-login-options-to-video-indexer-developer-portal"></a>Quelles sont les options de connexion au portail des développeurs Video Indexer ?

Vous pouvez vous connecter à l’aide d’Azure AD, d’un compte Microsoft, d’un compte Google ou d’un compte Facebook. 

Une fois que vous avez inscrit votre compte de courrier à l’aide d’un fournisseur d’identité, vous ne pouvez pas utiliser ce compte de courrier avec un autre fournisseur d’identité.

## <a name="billing-questions"></a>Questions sur la facturation

### <a name="how-much-does-video-indexer-cost"></a>Combien coûte Video Indexer ?

Video Indexer utilise un modèle tarifaire simple de paiement à l’utilisation, basé sur la durée de l’entrée de contenu que vous indexez. Des frais supplémentaires peuvent s’appliquer pour le codage, le streaming, le stockage, l’utilisation du réseau et les unités réservées Multimédia. Pour plus d’informations, consultez la page des [tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/video-indexer/).

### <a name="when-am-i-billed-for-using-video-indexer"></a>Quand suis-je facturé pour l’utilisation de Video Indexer ?

Lors de l’envoi d’une vidéo à indexer, l’utilisateur définit l’indexation comme étant une analyse vidéo, une analyse audio ou les deux. Cela détermine quelles références (SKU) seront ensuite facturées. Si une erreur de niveau critique se produit pendant le traitement, un code d’erreur est retourné en réponse. Dans ce cas, aucune facturation n’a lieu.  Une erreur critique peut être causée par un bogue dans notre code ou une défaillance critique dans une dépendance interne du service. Lorsque des erreurs non critiques, telles que l’identification erronée ou l’extraction d’insights, surviennent, une réponse est retournée. Dans tous les cas où une réponse valide (sans code d’erreur) est retournée, la facturation a lieu.
 
### <a name="does-video-indexer-offer-a-free-trial"></a>Un essai gratuit de Video Indexer est-il disponible ?

Oui. Video Indexer propose un essai gratuit qui offre des fonctionnalités complètes de service et d’API. Il existe un quota de 600 minutes de vidéos pour les utilisateurs de l’interface web et de 2 400 minutes pour les utilisateurs d’API. 

## <a name="next-steps"></a>Étapes suivantes

* [Vue d'ensemble](video-indexer-overview.md)
* [Stack Overflow](https://stackoverflow.com/search?q=video-indexer)
