---
title: Vue d’ensemble d’Azure Media Services v3
titleSuffix: Azure Media Services
description: Présentation détaillée d’Azure Media Services V3 avec des liens vers des démarrages rapides, des didacticiels et des exemples de code.
services: media-services
documentationcenter: na
author: IngridAtMicrosoft
manager: femila
editor: ''
tags: ''
keywords: azure media services, flux, diffusion, en direct, hors connexion
ms.service: media-services
ms.devlang: multiple
ms.topic: overview
ms.tgt_pltfrm: multiple
ms.workload: media
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: mvc
ms.openlocfilehash: 01ec0c7c0d2d060bbd930a099fe3db54de6cb150
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93308964"
---
# <a name="azure-media-services-v3-overview"></a>Vue d’ensemble d’Azure Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services est une plateforme sur le cloud vous permettant de créer des solutions qui assurent une diffusion de vidéo en continu de qualité, améliorent l’accessibilité et la distribution, analysent le contenu et bien plus encore. Que vous soyez un développeur d’applications, un centre d’appels, un organisme public, une société de divertissement, Media Services vous aide à créer des applications proposant des expériences multimédias de qualité exceptionnelle à un large public sur les navigateurs et les appareils les plus populaires actuellement.

Les kits SDK Media Services v3 sont basés sur la [spécification OpenAPI (Swagger) pour Media Services v3](https://aka.ms/ams-v3-rest-sdk).

> [!NOTE]
> Actuellement, vous pouvez utiliser le [portail Azure](https://portal.azure.com/) pour gérer les [événements en direct](live-events-outputs-concept.md) Media Services v3, voir (mais pas gérer) les [actifs multimédias](assets-concept.md) v3 et [obtenir des informations sur l’accès aux API](./access-api-howto.md). Pour toutes les autres tâches de gestion (par exemple, les [transformations et travaux](transforms-jobs-concept.md) et la [protection de contenu](content-protection-overview.md)), utilisez l’[API REST](/rest/api/media/), l’interface [CLI](/cli/azure/ams) ou l’un des [SDK](media-services-apis-overview.md#sdks) pris en charge.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="compliance-privacy-and-security"></a>Conformité, confidentialité et sécurité

Pour rappel, vous devez respecter toutes les lois applicables dans le cadre de votre utilisation d’Azure Media Services et vous n’êtes pas autorisé à utiliser Media Services ou d’autres services Azure d’une manière qui porterait atteinte aux droits des tiers ou qui leur serait préjudiciable.

Avant de charger une vidéo ou une image vers Media Services, Vous devez disposer de tous les droits appropriés pour utiliser la vidéo ou l’image, y compris, lorsque cela est requis par la Loi, avoir obtenu tous les consentements nécessaires de chaque personne (le cas échéant) apparaissant dans la vidéo ou l’image, pour autoriser l’utilisation, le traitement et le stockage de leurs données dans Media Services et Azure. Certaines juridictions peuvent imposer des obligations légales spéciales pour la collecte, le traitement en ligne et le stockage de catégories de données particulières, comme les données biométriques. Avant d’utiliser Media Services et Azure pour le traitement et le stockage de données soumises à des conditions légales spéciales, vous devez veiller à vous conformer à toutes les conditions légales qui s’appliquent.

Pour en savoir plus sur la conformité, la confidentialité et la sécurité dans Media Services, consultez le [Centre de confidentialité](https://www.microsoft.com/trust-center/?rtc=1) Microsoft. Pour connaître les obligations de Microsoft relatives à la confidentialité et les bonnes pratiques de gestion et de conservation de vos données, consultez la [Déclaration de confidentialité](https://privacy.microsoft.com/PrivacyStatement), les [Conditions d’utilisation des services en ligne](https://www.microsoft.com/licensing/product-licensing/products?rtc=1) (« OST ») et l’[Avenant au traitement des données](https://www.microsoftvolumelicensing.com/DocumentSearch.aspx?Mode=3&DocumentTypeId=67) (« DPA ») de Microsoft. En utilisant Media Services, vous consentez à être lié par les dispositions de l’OST, du DPA et de la Déclaration de confidentialité.
 
## <a name="what-can-i-do-with-media-services"></a>Que puis-je faire avec Media Services ?

Media Services vous permet de créer un grand nombre de flux de travail multimédias dans le cloud. Voici quelques exemples de ce que vous pouvez faire avec Media Services :

* Fournir des vidéos dans différents formats afin qu’elles puissent être lues sur un large éventail de navigateurs et d’appareils. Pour le streaming en direct et à la demande sur différents clients (appareils mobiles, télévisions, PC, etc.), le contenu vidéo et audio doit être codé et empaqueté de façon appropriée. Pour savoir comment livrer et diffuser ce type de contenu, consultez [Démarrage rapide : encoder des fichiers et les diffuser en continu](stream-files-dotnet-quickstart.md).
* Diffuser des événements sportifs en direct à un grand public en ligne, comme le football, le base-ball, le sport universitaire et bien plus encore.
* Diffuser des réunions et des événements publics, comme des réunions de conseil municipal et de corps législatifs.
* Analyser les vidéos ou le contenu audio enregistrés. Par exemple, pour atteindre une satisfaction client optimale, les organisations peuvent extraire une reconnaissance vocale, et générer des index de recherche et des tableaux de bord. Elles peuvent ensuite extraire des informations sur les réclamations, les causes des réclamations et autres données pertinentes.
* Créer un abonnement au service vidéo et diffuser du contenu DRM protégé lorsqu’un client (par exemple, un studio de cinéma) doit limiter l’accès et l’utilisation de l’œuvre protégée par copyright propriétaire.
* Proposer du contenu hors connexion dans des avions, des trains et des voitures. Un client peut avoir besoin de télécharger du contenu sur son téléphone ou sa tablette pour pouvoir le lire lorsqu’il sera déconnecté du réseau.
* Implémenter une plateforme vidéo de formation en ligne éducative avec Azure Media Services et les [API Azure Cognitive Services](../../index.yml?pivot=products&panel=ai) pour le sous-titrage de la reconnaissance vocale, la traduction en plusieurs langues, etc.
* Utilisez Azure Media Services avec les [API Azure Cognitive Services](../../index.yml?pivot=products&panel=ai) pour ajouter des sous-titres et des légendes aux vidéos afin de répondre aux besoins d’une audience plus large (par exemple, les personnes ayant des problèmes d’audition ou souhaitant lire simultanément dans une autre langue).
* Activez Azure CDN pour effectuer un scaling à grande échelle afin d’améliorer la gestion instantanée des charges importantes (par exemple le début d’un événement de lancement d’un produit).

## <a name="how-can-i-get-started-with-v3"></a>Comment bien démarrer avec v3 ?

Découvrez comment encoder et empaqueter du contenu, diffuser des vidéos à la demande, diffuser en direct et analyser vos vidéos avec Media Services v3. Des didacticiels, références d’API et autres documents vous montrent comment diffuser en toute sécurité du contenu vidéo ou audio à la demande et en direct pour des millions d’utilisateurs.

> [!TIP]
> Avant de commencer à développer, passez en revue ce qui suit : L’article [Concepts fondamentaux](concepts-overview.md), qui explique certains concepts importants, comme la création de packages, l’encodage et la protection, et l’article [Développement avec des API Media Services v3](media-services-apis-overview.md), qui fournit des informations sur l’accès aux API et les conventions de nommage, entre autres.

### <a name="sdks"></a>Kits SDK

Commencez à développer avec les [SDK clients Azure Media Services v3](media-services-apis-overview.md#sdks).

### <a name="quickstarts"></a>Démarrages rapides  

Les guides de démarrage rapide montrent les instructions fondamentales du 1er jour pour les nouveaux clients qui testent rapidement Media Services.

* [Diffuser des fichiers vidéo en streaming - .NET](stream-files-dotnet-quickstart.md)
* [Diffuser des fichiers vidéo en streaming - CLI](stream-files-cli-quickstart.md)
* [Diffuser des fichiers vidéo en streaming - Node.js](stream-files-nodejs-quickstart.md)

### <a name="tutorials"></a>Tutoriels

Les tutoriels montrent les procédures de scénarios pour certaines des tâches Media Services principales.

* [Encoder le fichier distant et diffuser la vidéo – REST](stream-files-tutorial-with-rest.md)
* [Encoder le fichier chargé et diffuser la vidéo – REST](stream-files-tutorial-with-api.md)
* [Diffuser en direct - .NET](stream-live-tutorial-with-api.md)
* [Analyser votre vidéo - .NET](analyze-videos-tutorial-with-api.md)
* [Chiffrement dynamique AES-128 - .NET](protect-with-aes128.md)

### <a name="samples"></a>Exemples

Utilisez [ce navigateur d’exemples](/samples/browse/?products=azure-media-services) pour parcourir des exemples de code Azure Media Services.

### <a name="how-to-guides"></a>Guides pratiques

Les guides pratiques contiennent des exemples de code qui montrent comment effectuer une tâche. Dans cette section, vous trouverez de nombreux exemples. En voici quelques-uns :

* [Créer un compte - CLI](./create-account-howto.md)
* [Accéder aux API - CLI](./access-api-howto.md)
* [Encoder avec HTTPS en tant qu’entrée de travail - .NET](job-input-from-http-how-to.md)  
* [Superviser les événements - Portail](monitor-events-portal-how-to.md)
* [Chiffrer dynamiquement avec multi-DRM - .NET](protect-with-drm.md) 
* [Comment encoder avec une transformation personnalisée - CLI](custom-preset-cli-howto.md)

## <a name="ask-questions-give-feedback-get-updates"></a>Poser des questions, envoyer des commentaires, obtenir des mises à jour

Découvrez l’article [Communauté Azure Media Services](media-services-community.md) pour découvrir les différentes façons dont vous pouvez poser des questions, faire des commentaires et obtenir des mises à jour sur Media Services.

## <a name="next-steps"></a>Étapes suivantes

[En savoir plus sur les concepts fondamentaux](concepts-overview.md)
