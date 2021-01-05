---
title: Qu’est-ce que Custom Translator ?
titleSuffix: Azure Cognitive Services
description: Custom Translator offre des fonctions similaires à celles de Microsoft Translator Hub pour la traduction automatique statistique (SMT), mais exclusivement pour les systèmes de traduction automatique neuronaux (NMT).
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 12/09/2019
ms.author: swmachan
ms.topic: overview
ms.openlocfilehash: 6a7301fd878eb65e4c1dd540275611bcf9348afc
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95241741"
---
# <a name="what-is-custom-translator"></a>Qu’est-ce que Custom Translator ?

[Custom Translator](https://portal.customtranslator.azure.ai) est une fonctionnalité du service Microsoft Translator qui permet aux entreprises de traduction, aux développeurs d’applications et aux fournisseurs de services linguistiques de créer des systèmes de traduction automatique neuronaux (NMT) personnalisés. Les systèmes de traduction personnalisés s’intègrent de manière fluide aux applications, aux workflows et aux sites web existants.

Les systèmes de traduction créés avec [Custom Translator](https://portal.customtranslator.azure.ai) sont disponibles via la même [API de traduction de texte Translator Text V3](../reference/v3-0-translate.md?tabs=curl) cloud [sécurisée](https://cognitive.uservoice.com/knowledgebase/articles/1147537-api-and-customization-confidentiality) extrêmement performante et scalable de Microsoft qui génère des milliards de traductions tous les jours.

Custom Translator prend en charge plus de trente-six langues et mappe directement vers les langues disponibles pour les systèmes NMT. Pour une liste complète, consultez [Langues Microsoft Translator](../language-support.md#customization).

## <a name="features"></a>Fonctionnalités

Custom Translator offre différentes fonctionnalités pour créer un système de traduction personnalisé et y accéder.

|Fonctionnalité  |Description  |
|---------|---------|
|[Tirer parti de la technologie de traduction automatique neuronale](https://www.microsoft.com/translator/blog/2016/11/15/microsoft-translator-launching-neural-network-based-translations-for-all-its-speech-languages/)     |  Améliorez votre traduction en tirant parti de la traduction automatique neuronale (NMT) fournie par Custom Translator.       |
|[Créer des systèmes qui connaissent la terminologie de votre activité](what-are-parallel-documents.md)     |  Personnalisez et créez des systèmes de traduction en utilisant des documents parallèles qui comprennent la terminologie utilisée dans votre entreprise et votre secteur d’activité.       |
|[Utiliser un dictionnaire pour générer vos modèles](what-is-dictionary.md)     |   Si vous n’avez pas de jeu de données d’entraînement, vous pouvez entraîner un modèle avec uniquement des données de dictionnaire.       |
|[Collaborer avec d’autres personnes](how-to-manage-settings.md#share-your-workspace)     |   Collaborez avec votre équipe en partageant votre travail avec différentes personnes.     |
|[Accéder à votre modèle de traduction personnalisé](../reference/v3-0-translate.md?tabs=curl)     |  Vos applications/programmes peuvent accéder à votre modèle de traduction personnalisé à tout moment via l’API de traduction de texte Translator Text V3 de Microsoft.       |

## <a name="get-better-translations"></a>Obtenir de meilleures traductions

Microsoft Translator a publié un service de [Traduction automatique neuronale (NTM)](https://www.microsoft.com/translator/blog/2016/11/15/microsoft-translator-launching-neural-network-based-translations-for-all-its-speech-languages/) en 2016. Le service NMT a présenté une avancée majeure en termes de qualité de traduction par rapport à la technologie standard de [traduction automatique statistique (SMT)](https://en.wikipedia.org/wiki/Statistical_machine_translation). Dans la mesure où le service NMT saisit mieux le contexte des phrases dans leur ensemble avant de les traduire, il offre de meilleures traductions dans un style plus naturel et plus fluide. [Custom Translator](https://portal.customtranslator.azure.ai) fournit le service NMT pour vos modèles personnalisés, ce qui donnera lieu à une amélioration de la qualité de traduction.

Vous pouvez utiliser des documents déjà traduits pour créer un système de traduction. Ces documents présentent un style et une terminologie spécifiques à un domaine, qui sont meilleurs qu’un système de traduction standard. Les utilisateurs peuvent charger des documents ALIGN, PDF, LCL, HTML, HTM, XLF, TMX, XLIFF, TXT, DOCX et XLSX.

Custom Translator accepte également les données qui sont parallèles au niveau document pour collecter et préparer les données de façon plus efficace. Si les utilisateurs ont accès à des versions du même contenu dans plusieurs langues, mais dans des documents distincts, Custom Translator est capable de faire correspondre automatiquement les phrases dans les documents.

Si vous fournissez le type et la quantité appropriés de données d’entraînement, il n’est pas rare de voir une augmentation du [score BLEU](what-is-bleu-score.md) de 5 à 10 points grâce à Custom Translator.

## <a name="be-productive-and-cost-effective"></a>Soyez productif et efficace

Avec [Custom Translator](https://portal.customtranslator.azure.ai), l’entraînement et le déploiement d’un système personnalisé ne nécessitent pas de compétences en programmation.

À l’aide du portail sécurisé de [Custom Translator](https://portal.customtranslator.azure.ai), les utilisateurs peuvent charger des données d’entraînement, entraîner les systèmes, tester les systèmes et les déployer dans un environnement de production via une interface utilisateur intuitive. Le système pourra être ensuite utilisé à grande échelle en quelques heures (l’heure réelle dépend de la taille des données d’entraînement).

[Custom Translator](https://portal.customtranslator.azure.ai) peut également être accessible programmatiquement via une [API dédiée](https://custom-api.cognitive.microsofttranslator.com/swagger/) (actuellement en préversion). L’API permet aux utilisateurs de gérer la création ou la mise à jour régulière d’entraînements depuis leur propre application ou service web.

Le coût d’utilisation d’un modèle personnalisé pour traduire du contenu est fonction du niveau tarifaire de l’API de traduction de texte Translator Text de l’utilisateur. Pour les détails des niveaux tarifaires, consultez la [page web des tarifs de l’API de traduction de texte Translator Text](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/) de Cognitive Services.

## <a name="securely-translate-anytime-anywhere-on-all-your-apps-and-services"></a>Traduisez sans crainte où et quand vous voulez sur toutes vos applications et tous vos services

Des systèmes personnalisés peuvent être accessibles et intégrés de manière fluide à n’importe quel produit ou workflow métier et sur n’importe quel appareil, par le biais de l’API de traduction de texte Translator Text Microsoft et d’une technologie REST standard.

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [détails de tarification](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/).

- Avec ce guide de [Démarrage rapide](quickstart-build-deploy-custom-model.md), apprenez à créer un modèle de traduction dans Custom Translator.
