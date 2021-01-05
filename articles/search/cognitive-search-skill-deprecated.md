---
title: Compétences cognitives déconseillées
titleSuffix: Azure Cognitive Search
description: Cette page contient la liste des compétences cognitives qui sont considérées comme déconseillées et qui ne seront pas prises en charge dans un avenir proche dans les ensembles de compétences de Recherche cognitive Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 85f3b9862bd8155c1a4b11860dc82d92a2f9e810
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936093"
---
# <a name="deprecated-cognitive-skills-in-azure-cognitive-search"></a>Compétences cognitives déconseillées dans Recherche cognitive Azure

Ce document décrit les compétences cognitives qui sont considérées comme déconseillées. Utilisez le guide suivant pour le contenu :

* Nom de la compétence : nom de la compétence qui sera déconseillée, il est mappé sur l’attribut @odata.type.
* Dernière version de l’API disponible : Dernière version de l’API publique Recherche cognitive Azure via laquelle des ensembles de compétences contenant la compétence déconseillée correspondante peuvent être créés/mis à jour.
* Fin de la prise en charge : dernier jour après lequel la compétence correspondante est considérée non prise en charge. Les compétences créées précédemment doivent continuer à fonctionner, mais il est recommandé aux utilisateurs de migrer hors d’une compétence déconseillée.
* Recommandations : chemin de migration vers l’avant pour utiliser une compétence prise en charge. Il est conseillé aux utilisateurs de suivre ces suggestions pour continuer à bénéficier du support technique.

## <a name="microsoftskillstextnamedentityrecognitionskill"></a>Microsoft.Skills.Text.NamedEntityRecognitionSkill

### <a name="last-available-api-version"></a>Dernière version d’API disponible

2017-11-11-Preview

### <a name="end-of-support"></a>Fin de la prise en charge

15 février 2019

### <a name="recommendations"></a>Recommandations 

Utilisez [Microsoft.Skills.Text.EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md) à la place. Il fournit la plupart des fonctionnalités de NamedEntityRecognitionSkill à un meilleur niveau de qualité. Il détient également des informations plus riches dans ses champs de sortie complexes.

Pour migrer vers la [compétence de reconnaissance des entités](cognitive-search-skill-entity-recognition.md), vous devrez apporter une ou plusieurs des modifications suivantes à votre définition de compétence. Vous pouvez mettre à jour la définition de compétence à l’aide de l’[API de mise à jour de compétences](/rest/api/searchservice/update-skillset).

> [!NOTE]
> actuellement, le score de confiance comme concept n’est pas pris en charge. Le paramètre `minimumPrecision` existe sur `EntityRecognitionSkill` pour une utilisation ultérieure et la compatibilité descendante.

1. *(Obligatoire)* Modifiez `@odata.type` en remplaçant `"#Microsoft.Skills.Text.NamedEntityRecognitionSkill"` par `"#Microsoft.Skills.Text.EntityRecognitionSkill"`.

2. *(Facultatif)* Si vous utilisez la sortie `entities`, utilisez la sortie de collection complexe `namedEntities` issue de `EntityRecognitionSkill` à la place. Vous pouvez utiliser `targetName` dans la définition de compétence pour le mapper à une annotation appelée `entities`.

3. *(Facultatif)* Si vous ne spécifiez pas explicitement `categories`, `EntityRecognitionSkill` peut retourner un type différent de catégories en plus de celles qui étaient prises en charge par `NamedEntityRecognitionSkill`. Si ce comportement est indésirable, veillez à définir explicitement le paramètre `categories` sur `["Person", "Location", "Organization"]`.

    _Exemples de définitions de migration_

    * Migration simple

        _(Avant) Définition de compétence NamedEntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
            "categories": [ "Person"],
            "defaultLanguageCode": "en",
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            }
            ]
        }
        ```
        _(Après) Définition de compétence EntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "categories": [ "Person"],
            "defaultLanguageCode": "en",
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            }
            ]
        }
        ```
    
    * Migration légèrement compliquée

        _(Avant) Définition de compétence NamedEntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
            "defaultLanguageCode": "en",
            "minimumPrecision": 0.1,
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            },
            {
                "name": "entities"
            }
            ]
        }
        ```
        _(Après) Définition de compétence EntityRecognition_
        ```json
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "categories": [ "Person", "Location", "Organization" ],
            "defaultLanguageCode": "en",
            "minimumPrecision": 0.1,
            "inputs": [
            {
                "name": "text",
                "source": "/document/content"
            }
            ],
            "outputs": [
            {
                "name": "persons",
                "targetName": "people"
            },
            {
                "name": "namedEntities",
                "targetName": "entities"
            }
            ]
        }
        ```

## <a name="see-also"></a>Voir aussi

+ [Compétences prédéfinies](cognitive-search-predefined-skills.md)
+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Compétence de reconnaissance des entités](cognitive-search-skill-entity-recognition.md)