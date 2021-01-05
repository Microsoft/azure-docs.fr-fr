---
title: Définition de schéma d’application
description: L’application LUIS est représentée dans le `.json` ou `.lu` et comprend toutes les intentions, les entités, les exemples de énoncés, les fonctionnalités et les paramètres.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 08/22/2020
ms.openlocfilehash: db4fd52dad82542f20e58ebb3b8526c5be7f2f88
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91327321"
---
# <a name="app-schema-definition"></a>Définition de schéma d’application

L’application LUIS est représentée dans le `.json` ou `.lu` et comprend toutes les intentions, les entités, les exemples de énoncés, les fonctionnalités et les paramètres.

## <a name="format"></a>Format

Lorsque vous importez et exportez l’application, choisissez `.json` ou `.lu`.

|Format|Information|
|--|--|
|`.json`| Format de programmation standard|
|`.lu`|Pris en charge par les outils [Bot Builder](https://github.com/microsoft/botbuilder-tools/blob/master/packages/Ludown/docs/lu-file-format.md) de Bot Framework.|

## <a name="version-7x"></a>Version 7.x

* Si vous passez à la version 7.x, les entités sont représentées en tant qu’entités issues du Machine Learning imbriquées.
* Prise en charge de la création d’entités issues du Machine Learning imbriquées avec la propriété `enableNestedChildren` sur les API de création suivantes :
    * [Ajouter une étiquette](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c08)
    * [Ajouter une étiquette de lot](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c09)
    * [Examiner les étiquettes](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c0a)
    * [Suggérer des requêtes de point de terminaison pour les entités](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2e)
    * [Suggérer des requêtes de point de terminaison pour les intentions](https://westus.dev.cognitive.microsoft.com/docs/services/luis-programmatic-apis-v3-0-preview/operations/5890b47c39e2bb052c5b9c2d)

```json
{
  "luis_schema_version": "7.0.0",
  "intents": [
    {
      "name": "None",
      "features": []
    }
  ],
  "entities": [],
  "hierarchicals": [],
  "composites": [],
  "closedLists": [],
  "prebuiltEntities": [],
  "utterances": [],
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "patternAnyEntities": [],
  "regex_entities": [],
  "phraselists": [
  ],
  "regex_features": [],
  "patterns": [],
  "settings": []
}
```

| element                  | Commentaire                              |
|--------------------------|--------------------------------------|
| "hierarchicals" : [],     | Déprécié, utilisez [entités de machine-learning](luis-concept-entity-types.md).   |
| "composites" : [],        | Déprécié, utilisez [entités de machine-learning](luis-concept-entity-types.md). Référence d’[entité composite](reference-entity-composite.md). |
| "closedLists" : [],       | Référence d’[entités de liste](reference-entity-list.md), principalement utilisée comme fonctionnalités pour les entités.    |
| "versionId" : "0.1",      | Version d’une application LUIS.|
| "name" : "example-app",   | Nom de l’application LUIS. |
| "desc" : "",              | Description facultative de l’application LUIS.  |
| "culture" : "en-us",      | [Langue](luis-language-support.md) de l’application, a un impact sur les fonctionnalités sous-jacentes telles que les entités prédéfinies, le machine learning et le générateur de jetons.  |
| "tokenizerVersion" : "1.0.0", | [Générateur de jetons](luis-language-support.md#tokenization)  |
| "patternAnyEntities" : [],   | [Entité Pattern.any](reference-entity-pattern-any.md)    |
| "regex_entities" : [],    |  [Entité d’expression régulière](reference-entity-regular-expression.md)   |
| "phraselists" : [],       |  [Listes d’expressions (fonctionnalité)](luis-concept-feature.md#create-a-phrase-list-for-a-concept)   |
| "regex_features" : [],    |  Déprécié, utilisez [entités de machine-learning](luis-concept-entity-types.md). |
| "patterns" : [],          |  [Les modèles améliorent la précision de la prédiction](luis-concept-patterns.md) avec la [syntaxe de modèle](reference-pattern-syntax.md)   |
| "settings" : []           | [Paramètres de l’application](luis-reference-application-settings.md)|

## <a name="version-6x"></a>Version 6.x

* En passant à la version 6.x, utilisez la nouvelle [entité issue du Machine Learning](reference-entity-machine-learned-entity.md) pour représenter vos entités.

```json
{
  "luis_schema_version": "6.0.0",
  "intents": [
    {
      "name": "None",
      "features": []
    }
  ],
  "entities": [],
  "hierarchicals": [],
  "composites": [],
  "closedLists": [],
  "prebuiltEntities": [],
  "utterances": [],
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "patternAnyEntities": [],
  "regex_entities": [],
  "phraselists": [],
  "regex_features": [],
  "patterns": [],
  "settings": []
}
```

## <a name="version-4x"></a>Version 4.x

```json
{
  "luis_schema_version": "4.0.0",
  "versionId": "0.1",
  "name": "example-app",
  "desc": "",
  "culture": "en-us",
  "tokenizerVersion": "1.0.0",
  "intents": [
    {
      "name": "None"
    }
  ],
  "entities": [],
  "composites": [],
  "closedLists": [],
  "patternAnyEntities": [],
  "regex_entities": [],
  "prebuiltEntities": [],
  "model_features": [],
  "regex_features": [],
  "patterns": [],
  "utterances": [],
  "settings": []
}
```

## <a name="next-steps"></a>Étapes suivantes

* Migrer vers la [V3 des API de création](luis-migration-authoring-entities.md)