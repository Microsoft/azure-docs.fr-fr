---
title: Qu’est-ce que le service Vision par ordinateur ?
titleSuffix: Azure Cognitive Services
description: Le service Vision par ordinateur vous donne accès à des algorithmes avancés pour le traitement d’images et le renvoi d’informations.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: overview
ms.date: 11/23/2020
ms.author: pafarley
ms.custom:
- seodec18
- cog-serv-seo-aug-2020
- contperf-fy21q2
keywords: vision par ordinateur, applications de vision par ordinateur, service vision par ordinateur
ms.openlocfilehash: 62d82388d975f36c422f7eb1e9feff8ce2725d54
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033287"
---
# <a name="what-is-computer-vision"></a>Qu’est-ce que le service Vision par ordinateur ?

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Le service Vision par ordinateur d’Azure vous donne accès à des algorithmes avancés permettant de traiter des images et de retourner des informations en fonction des caractéristique visuelles qui vous intéressent. Par exemple, Vision par ordinateur peut déterminer si une image contient du contenu pour adultes, identifier des marques ou des objets spécifiques ou trouver des visages humains.

Vous pouvez créer des applications Vision par ordinateur par le biais d’un [kit SDK de bibliothèque de client](./quickstarts-sdk/client-library.md) ou en appelant directement l’[API REST](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/5d986960601faab4bf452005). Cette page couvre globalement ce que vous pouvez faire avec le service Vision par ordinateur.

## <a name="optical-character-recognition-ocr"></a>Reconnaissance optique des caractères (OCR)

Vision par ordinateur comprend des [fonctionnalités de reconnaissance optique de caractères (OCR)](concept-recognizing-text.md). Vous pouvez utiliser la nouvelle API Lire pour extraire du texte imprimé et manuscrit d’images et de documents. Elle utilise les modèles les plus récents et fonctionne avec du texte sur diverses surfaces et fonds. Ces derniers comprennent les reçus, les affiches, les cartes de visite, les lettres et les tableaux blancs. Les deux API de reconnaissance optique de caractères prennent en charge l’extraction de texte imprimé en [plusieurs langues](./language-support.md). Pour bien démarrer, suivez un [guide de démarrage rapide](./quickstarts-sdk/client-library.md).

## <a name="computer-vision-for-digital-asset-management"></a>Vision par ordinateur pour la gestion des ressources numériques

Vision par ordinateur peut contribuer à de nombreux scénarios de gestion des actifs numériques (DAM). La gestion des actifs numériques est le processus métier d’organisation, de stockage et de récupération des éléments multimédias enrichis, et de gestion des droits et autorisations numériques. Par exemple, une entreprise peut décider de regrouper et d’identifier des images sur la base de logos, de visages, d’objets, de couleurs, ou d’autres éléments visibles. Vous pouvez également [générer automatiquement des légendes pour des images](./Tutorials/storage-lab-tutorial.md) et y associer des mots clés afin qu’elles puissent faire l’objet d’une recherche. Pour une solution gestion des actifs numériques tout-en-un utilisant Cognitive Services, Recherche cognitive Azure et une fonctionnalité de création de rapports intelligents, consultez le [Guide de l’accélérateur de solution d’exploration des connaissances](https://github.com/Azure-Samples/azure-search-knowledge-mining) sur GitHub. Pour d’autres exemples de Vision par ordinateur, consultez le dépôt [Modèles de solution Vision par ordinateur](https://github.com/Azure-Samples/Cognitive-Services-Vision-Solution-Templates).

## <a name="analyze-images-for-insight"></a>Analyser des images pour obtenir des informations

Vous pouvez analyser des images pour fournir des insights sur leurs caractéristiques visuelles. Toutes les fonctionnalités listées dans le tableau ci-dessous sont fournies par l’API [Analyser l’image](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-1-ga/operations/56f91f2e778daf14a499f21b). Pour bien démarrer, suivez un [guide de démarrage rapide](./quickstarts-sdk/client-library.md).


### <a name="tag-visual-features"></a>Identifier les composants visuels à l’aide de balises

Identifier les composants visuels d’une image et les marquer à l’aide de balises à partir d’un ensemble de milliers d’objets, d’êtres vivants, de scènes et d’actions reconnaissables. Quand les étiquettes sont ambigües ou inhabituelles, la réponse de l’API fournit des conseils pour clarifier le contexte de l’étiquette. Le balisage ne se limite pas au sujet principal, comme une personne au premier plan, mais il inclut également le décor (intérieur ou extérieur), le mobilier, les outils, les plantes, les animaux, les accessoires, les gadgets, etc. [Identifier les composants visuels à l’aide de balises](concept-tagging-images.md)

### <a name="detect-objects"></a>Détecter des objets

La détection d’objets est similaire au balisage, mais l’API retourne les coordonnées de cadre englobant pour chaque balise appliquée. Par exemple, si une image contient un chien, un chat et une personne, l’opération de détection liste ces objets ainsi que leurs coordonnées dans l’image. Vous pouvez utiliser cette fonctionnalité pour traiter d’autres relations entre les objets dans une image. Elle vous permet également de savoir quand il existe plusieurs instances de la même balise dans une image. [Détecter des objets](concept-object-detection.md)

### <a name="detect-brands"></a>Détecter les marques

Identifiez les marques commerciales dans les images ou vidéos à partir d’une base de données de milliers de logos internationaux. Vous pouvez utiliser cette fonctionnalité, par exemple, pour déterminer quelles marques sont les plus populaires sur les réseaux sociaux, et lesquelles prédominent dans les médias. [Détecter les marques](concept-brand-detection.md)

### <a name="categorize-an-image"></a>Catégoriser une image

Identifier et catégoriser une image dans son ensemble en utilisant une [taxonomie des catégories](Category-Taxonomy.md) comprenant des hiérarchies héréditaires parent/enfant. Les catégories peuvent être utilisées seules ou avec nos nouveaux modèles de balisage.<br/>Actuellement, l’anglais est la seule langue prise en charge pour le balisage et la catégorisation des images. [Catégoriser une image](concept-categorizing-images.md)

### <a name="describe-an-image"></a>Décrire une image

Générer une description de l’intégralité d’une image dans un langage lisible utilisant des phrases complètes. Les algorithmes du service Vision par ordinateur génèrent différentes descriptions selon les objets identifiés dans l’image. Chacune des descriptions est évaluée, et un score de confiance est généré. Une liste est ensuite renvoyée, classée du score de confiance plus élevé au plus bas. [Décrire une image](concept-describing-images.md)

### <a name="detect-faces"></a>Détecter des visages

Détecter les visages dans une image et fournir des informations sur chaque visage détecté. Le service Vision par ordinateur indique les coordonnées, le rectangle, le sexe et l’âge pour chaque visage détecté.<br/>Le service Vision par ordinateur fournit un sous-ensemble des fonctionnalités du service [Visage](../face/index.yml). Utilisez le service Visage pour effectuer une analyse plus détaillée (reconnaissance faciale, détection de la posture, etc.). [Détecter des visages](concept-detecting-faces.md)

### <a name="detect-image-types"></a>Détecter les types d’images

Détecter les caractéristiques relatives à une image, par exemple si une image est un dessin au trait ou s’il pourrait s’agir d’une image clipart. [Détecter les types d’images](concept-detecting-image-types.md)

### <a name="detect-domain-specific-content"></a>Détecter le contenu spécifique à un domaine

Utiliser des modèles de domaine pour détecter et identifier le contenu spécifique à un domaine dans une image, notamment pour reconnaître des célébrités ou des éléments géographiques. Par exemple, si une image contient des célébrités, le service Vision par ordinateur peut utiliser un modèle de domaine pour célébrités afin de déterminer si les personnes détectées dans l’image correspondent à des célébrités connues. [Détecter le contenu spécifique à un domaine](concept-detecting-domain-content.md)

### <a name="detect-the-color-scheme"></a>Détecter le jeu de couleurs

Analyser l’utilisation des couleurs dans une image. Le service Vision par ordinateur peut déterminer si une image est en noir et blanc ou en couleur. Pour les images en couleur, il peut également identifier les couleurs dominantes et d’accentuation. [Détecter le jeu de couleurs](concept-detecting-color-schemes.md)

### <a name="generate-a-thumbnail"></a>Générer une miniature

Analyser le contenu d’une image pour en générer une miniature. Le service Vision par ordinateur commence par générer une miniature de haute qualité, puis analyse les objets contenus dans l’image pour déterminer la *zone d’intérêt*. Il rogne ensuite l’image pour conserver uniquement la zone d’intérêt. La miniature générée peut être présentée à l’aide de proportions différentes de celles de l’image d’origine selon les besoins de chacun. [Générer une miniature](concept-generating-thumbnails.md)

### <a name="get-the-area-of-interest"></a>Obtenir la zone d’intérêt

Analyser le contenu d’une image pour retourner les coordonnées de la *zone d’intérêt*. Au lieu de rogner l’image et de générer une miniature, le service Vision par ordinateur retourne les coordonnées de cadre englobant de la région pour que l’application appelante puisse modifier l’image d’origine de la manière souhaitée. [Obtenir la zone d’intérêt](concept-generating-thumbnails.md#area-of-interest)

## <a name="moderate-content-in-images"></a>Modérer du contenu dans les images

Vous pouvez utiliser Vision par ordinateur pour [détecter des contenus pour adultes](concept-detecting-adult-content.md) dans une image et retourner des scores de confiance pour différentes classifications. Le seuil de marquage du contenu peut être défini sur une échelle pour prendre en compte vos préférences.

## <a name="deploy-on-premises-using-docker-containers"></a>Déployer localement en utilisant des conteneurs Docker

Utilisez des conteneurs Vision par ordinateur pour déployer localement des fonctionnalités d’API. Ces conteneurs Docker vous donnent la possibilité de rapprocher le service plus près de vos données, ce qui peut être souhaitable pour des raisons de conformité, de sécurité ou opérationnelles. Vision par ordinateur offre les conteneurs suivants :

* Le [conteneur OCR de lecture de Vision par ordinateur (préversion)](computer-vision-how-to-install-containers.md) vous permet de reconnaître du texte imprimé et manuscrit dans les images.
* Le [conteneur d’analyse spatiale de Vision par ordinateur (préversion)](spatial-analysis-container.md) vous permet d’analyser les vidéos en streaming temps réel afin de comprendre les relations spatiales entre les personnes et leurs déplacements dans des environnements physiques.

## <a name="image-requirements"></a>Exigences des images

Le service Vision par ordinateur peut analyser des images qui répondent aux exigences suivantes :

- L’image doit être au format JPEG, PNG, GIF ou BMP
- La taille de fichier de l’image doit être inférieure à 4 mégaoctets (Mo)
- Les dimensions de l’image doivent être supérieures à 50 x 50 pixels
  - Pour l’API Read, la taille de l’image doit être comprise entre 50 x 50 et 10 000 x 10 000 pixels.

## <a name="data-privacy-and-security"></a>Sécurité et confidentialité des données

Comme avec tous les services Cognitive Services, les développeurs utilisant le service Vision par ordinateur doivent connaître les politiques de Microsoft relatives aux données client. Pour en savoir plus, consultez la [page Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) dans le Centre de gestion de la confidentialité Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Commencez avec Vision par ordinateur en suivant le guide de démarrage rapide dans votre langage de développement préféré :

- [Démarrage rapide : API REST ou bibliothèques de client Vision par ordinateur](./quickstarts-sdk/client-library.md)
