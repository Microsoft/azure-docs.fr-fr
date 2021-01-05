---
title: 'Nouveauté : service Language Understanding (LUIS)'
description: Cet article est régulièrement mis à jour avec des informations nouvelles sur l’API Language Understanding Azure Cognitive Services.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: overview
ms.date: 06/15/2020
ms.openlocfilehash: fffe337a59ff343164a155fcd0f4d5616a32bf41
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91334614"
---
# <a name="whats-new-in-language-understanding"></a>Nouveauté du service Language Understanding

Découvrir les nouveautés du service. Ces éléments incluent des notes de publication, des vidéos, des billets de blog et d’autres types d’informations. Marquez cette page pour rester au fait des nouveautés du service.

## <a name="release-notes"></a>Notes de publication

### <a name="june-2020"></a>Juin 2020

* Kit SDK [Preview 3.0 Authoring](luis-migration-authoring-entities.md)
    * Version 3.2.0-preview.3 - [.NET - NuGet](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.LUIS.Authoring/)
    * Version 4.0.0-preview.3 - [JS - NPM](https://www.npmjs.com/package/@azure/cognitiveservices-luis-authoring)
* Application des pratiques DevOps avec LUIS
    * Concepts
        * [Pratiques DevOps pour LUIS](luis-concept-devops-sourcecontrol.md)
        * [Workflows d’intégration continue et de livraison continue pour LUIS DevOps](luis-concept-devops-automation.md)
        * [Test de LUIS DevOps](luis-concept-devops-testing.md)
    * Procédures
        * [Appliquer DevOps au développement d’applications LUIS à l’aide de GitHub Actions](luis-how-to-devops-with-github.md)
    * [Dépôt GitHub code complet](https://github.com/Azure-Samples/LUIS-DevOps-Template)

### <a name="may-2020---build"></a>Mai 2020 – //Build

* Publié en tant que **version en disposition générale** :
    * [Conteneur Language Understanding](luis-container-howto.md)
    * La préversion du portail est promue en [version actuelle du portail](https://www.luis.ai), la version [précédente](https://previous.luis.ai) du portail reste toujours disponible
    * Nouvelle expérience de création d’entités issues du machine learning
    * [Processus de mise à niveau](migrate-from-composite-entity.md) des entités composites et simples aux entités issues du machine learning
    * Prise en charge du [paramètre de normalisation des variantes de mot](how-to-application-settings-portal.md)
* Modifications apportées aux API de création
    * Schéma d’application 7.x pour les entités issues du machine learning imbriquées
    * [Migration vers la fonctionnalité requise](luis-migration-authoring-entities.md#api-change-constraint-replaced-with-required-feature)
* Nouvelles ressources pour les développeurs
    * [Outils d’intégration continue](developer-reference-resource.md#continuous-integration-tools)
    * Atelier : découvrir les bonnes pratiques pour la [_compréhension du langage naturel_ à l’aide de LUIS](developer-reference-resource.md#workshops)
* [Clés gérées par le client](luis-encryption-of-data-at-rest.md) : chiffrez toutes les données que vous utilisez dans LUIS à l’aide de votre propre clé
* [Présentation de l’IA](https://channel9.msdn.com/Shows/AI-Show/New-Features-in-Language-Understanding) (vidéo) : découvrez les nouvelles fonctionnalités de LUIS



### <a name="march-2020"></a>Mars 2020

* La sécurité TLS 1.2 est maintenant appliquée pour toutes les requêtes HTTP adressées à ce service. Pour plus d’informations, voir [Sécurité Azure Cognitive Services](../cognitive-services-security.md).

### <a name="november-4-2019---ignite"></a>4 novembre 2019 – Ignite

* Vidéo - [Modèles NLU (Natural Language Understanding) avancés avec LUIS et Azure Cognitive Services | BRK2188](https://www.youtube.com/watch?v=JdJEV2jV0_Y)

* Amélioration de la productivité des développeurs
    * Disponibilité générale de notre [point de terminaison de prédiction V3](luis-migration-api-v3.md).
    * Possibilité d’importer et d’exporter des applications avec le format `.lu` ([LUDown](https://github.com/microsoft/botbuilder-tools/tree/master/packages/Ludown)). Cela ouvre la voie à un processus d’intégration/livraison continues (CI/CD) efficace.
* Extension linguistique
    * [Arabe et hindi](luis-language-support.md) en préversion publique.
* Modèles prédéfinis
    * Des [domaines prédéfinis](luis-reference-prebuilt-domains.md) sont désormais en disponibilité générale (GA)
    * Les [entités prédéfinies](luis-reference-prebuilt-entities.md#japanese-entity-support) japonaises (âge, devise, nombre et pourcentage) ne sont pas prises en charge dans la version V3.
    * Les [entités prédéfinies](luis-reference-prebuilt-entities.md#italian-entity-support) italiennes (âge, devise, dimension, nombre et résolution de pourcentage) ont changé depuis la version V2.
* Amélioration de l’expérience utilisateur dans le [portail preview.luis.ai](https://preview.luis.ai) (expérience d’étiquetage remaniée pour permettre la génération et le débogage de modèles complexes). Essayez les tutoriels du portail en préversion :
    * [Intentions uniquement](tutorial-intents-only.md)
    * [Entité issue du machine learning décomposable](tutorial-machine-learned-entity.md)
* Fonctionnalités avancées de compréhension de la langue ([génération de modèles linguistiques sophistiqués](luis-concept-entity-types.md) avec moins d’efforts).
* Définition de fonctionnalités de machine learning au niveau du modèle et activation de modèles à utiliser en tant que signaux pour d’autres modèles, comme l’utilisation d’entités en tant que fonctionnalités pour des intentions et d’autres entités.
* Nouvelles [limites](luis-limits.md) étendues (maximum plus élevé pour les listes d’expressions et le nombre total d’expressions, ainsi que nouveau modèle en tant que limites de fonctionnalité)
* Extraction d’informations à partir de texte au format de structure hiérarchique profonde, qui rend les applications de conversation plus puissantes.

    ![Image d’entité issue du machine learning](./media/whats-new/deep-entity-extraction-example.png)

### <a name="september-3-2019"></a>3 septembre 2019

* Ressource de création Azure : [migrer maintenant](luis-migration-authoring.md).
    * 500 applications par ressource Azure
    * 100 versions par application
* Prise en charge de la langue turque pour les entités prédéfinies
* Prise en charge de la langue italienne pour datetimeV2

### <a name="july-23-2019"></a>23 juillet 2019

* Mise à jour de [Recognizers-Text](https://github.com/microsoft/Recognizers-Text/releases/tag/dotnet-v1.2.3) vers 1.2.3
    * Reconnaissances de l’âge, de la température, de la dimension et des devises en italien.
    * Amélioration de la reconnaissance des jour fériés en anglais pour calculer correctement les dates basées sur la Thanksgiving.
    * Améliorations de DateTime en français pour réduire les faux positifs des entités non-date et non-heure.
    * Prise en charge des années civiles/scolaires/fiscales et des acronymes dans DateRange en anglais.
    * Reconnaissance de PhoneNumber améliorée en chinois et japonais.
    * Prise en charge améliorée de NumberRange en anglais.
    * Amélioration des performances.

### <a name="june-24-2019"></a>24 juin 2019

* [Entité prédéfinie OrdinalV2](luis-reference-prebuilt-ordinal-v2.md) pour la prise en charge de l’ordre, comme « suivant », « précédent » et « dernier ». Anglais uniquement.

### <a name="may-6-2019---build-conference"></a>6 mai 2019 - //Conférence Build

Les fonctionnalités suivantes ont été publiées lors de la conférence Build 2019 :

* [Aperçu du guide de migration API V3](luis-migration-api-v3.md)
* [Tableau de bord d’analyse amélioré](luis-how-to-use-dashboard.md)
* [Domaines prédéfinis améliorés](luis-reference-prebuilt-domains.md)
* [Entités de liste dynamique](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [Entités externes](schema-change-prediction-runtime.md#external-entities-passed-in-at-prediction-time)

## <a name="blogs"></a>Blogs

[Bot Framework](https://blog.botframework.com/)

## <a name="videos"></a>Vidéos

### <a name="2019-ignite-videos"></a>Vidéos Ignite 2019

[Modèles NLU (Natural Language Understanding) avancés avec LUIS et Azure Cognitive Services | BRK2188](https://www.youtube.com/watch?v=JdJEV2jV0_Y)

### <a name="2019-build-videos"></a>Vidéos Build 2019

[Comment utiliser l’intelligence artificielle conversationnelle Azure pour préparer votre entreprise à la prochaine génération](https://www.youtube.com/watch?v=_k97jd-csuk&feature=youtu.be)

## <a name="service-updates"></a>Mises à jour de service

[Annonces de mise à jour Azure pour Cognitive Services](https://azure.microsoft.com/updates/?product=cognitive-services)
