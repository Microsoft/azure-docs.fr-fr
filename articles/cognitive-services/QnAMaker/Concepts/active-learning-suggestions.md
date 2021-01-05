---
title: Suggestions d’apprentissage actif – QnA Maker
description: Les suggestions d’apprentissage actif vous permettent d’améliorer la qualité de votre base de connaissances en suggérant d’autres questions à vos paires de question-réponse en fonction des soumissions d’utilisateurs.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: e1a8043912c984be46f85bd384a7049da27028b3
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353236"
---
# <a name="active-learning-suggestions"></a>Suggestions d’apprentissage actif

La fonctionnalité _Suggestions d’apprentissage actif_ vous permet d’améliorer la qualité de votre base de connaissances en suggérant d’autres questions à vos paires de question-réponse en fonction des soumissions d’utilisateurs. Après avoir passé en revue ces suggestions, ajoutez-les aux questions existantes ou rejetez-les.

Votre base de connaissances ne change pas automatiquement. Vous devez accepter les suggestions afin que toute modification prenne effet. Ces suggestions ajoutent des questions, mais elles ne modifient pas les questions existantes pas plus qu’elles ne les suppriment.

## <a name="what-is-active-learning"></a>Présentation de l’apprentissage actif

QnA Maker apprend de nouvelles variantes de question à l’aide des commentaires implicites et explicites.

* [Commentaires implicites](#how-qna-makers-implicit-feedback-works) : l’outil de classement comprend lorsqu’une question d’utilisateur a plusieurs réponses dont les scores sont très proches et la considère comme un commentaire. Vous n’avez pas besoin de faire quoi que ce soit pour que cela se produise.
* [Commentaires explicites](#how-you-give-explicit-feedback-with-the-train-api) : si plusieurs réponses avec peu de variations dans les scores sont renvoyées à partir de la base de connaissances, l’application cliente demande à l’utilisateur quelle question est appropriée. Les commentaires explicites de l’utilisateur sont envoyés à QnA Maker avec [l’API Train](../How-to/improve-knowledge-base.md#train-api).

Les deux méthodes fournissent à l’outil de classement des requêtes similaires qui sont ordonnées en cluster.

## <a name="how-active-learning-works"></a>Fonctionnement de l’apprentissage actif

L’apprentissage actif est déclenché en fonction des scores des principales réponses renvoyées par QnA Maker. Si les différences de scores entre les paires de Q/R qui correspondent à la requête s’inscrivent dans une petite plage, la requête est considérée comme suggestion possible (en tant que question alternative) pour chacune des paires de Q/R possibles. Une fois que vous acceptez la question suggérée pour une paire de question-réponse spécifique, elle est rejetée pour les autres paires. Vous devez penser à enregistrer et effectuer l’apprentissage, après avoir accepté les suggestions.

L’apprentissage actif offre les meilleures suggestions possible dans les cas où les points de terminaison reçoivent une quantité raisonnable de différentes requêtes d’utilisation. Lorsque 5 requêtes similaires ou plus sont ordonnées en cluster, toutes les 30 minutes, QnA Maker suggère les questions de l’utilisateur au concepteur de la base de connaissances pour qu’il les accepte ou les rejette. Toutes les suggestions sont regroupées par similarité, et les principales suggestions pour les questions alternatives sont affichées en fonction de la fréquence des requêtes particulières des utilisateurs finals.

Une fois les questions suggérées dans le portail QnA Maker, vous pouvez examiner, puis vérifier et accepter, ou rejeter, ces suggestions. Il n’existe pas d’API permettant de gérer des suggestions.

## <a name="turn-on-active-learning"></a>Activation de l’apprentissage actif

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Par défaut, l’apprentissage actif est **désactivé**.
Pour utiliser l’apprentissage actif :
* Vous devez [activer l’apprentissage actif](../How-To/use-active-learning.md#turn-on-active-learning-for-alternate-questions) afin que QnA Maker collecte des d’autres questions pour votre base de connaissances.
* Pour voir les autres questions suggérées, [utilisez les options Affichage](../How-To/improve-knowledge-base.md#view-suggested-questions) sur la page Modifier.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Par défaut, l’apprentissage actif est **activé** dans QnA Maker managé (préversion). Pour voir les autres questions suggérées, [utilisez les options Affichage](../How-To/improve-knowledge-base.md#view-suggested-questions) sur la page Modifier.

---

## <a name="how-qna-makers-implicit-feedback-works"></a>Fonctionnement des commentaires implicites de QnA Maker

Les commentaires implicites de QnA Maker utilisent un algorithme pour déterminer la proximité de score, puis pour faire des suggestions d’apprentissage actif. L’algorithme permettant de déterminer la proximité n’est pas un calcul simple. Les plages indiquées dans l’exemple suivant ne sont pas censées être fixes, mais doivent servir de guide pour comprendre l’impact de l’algorithme uniquement.

Lorsque le niveau de confiance du score d’une question est élevé, par exemple 80 %, la plage des scores pris en compte pour l’apprentissage actif est large, environ 10 %. À mesure que le score de confiance diminue, par exemple 40 %, la plage des scores décroît également et passe à environ 4 %.

Dans la réponse JSON suivante issue d’une requête à l’API GenerateAnswer de QnA Maker, les scores pour A, B et C sont proches et sont considérés comme des suggestions.

```json
{
  "activeLearningEnabled": true,
  "answers": [
    {
      "questions": [
        "Q1"
      ],
      "answer": "A1",
      "score": 80,
      "id": 15,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    },
    {
      "questions": [
        "Q2"
      ],
      "answer": "A2",
      "score": 78,
      "id": 16,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    },
    {
      "questions": [
        "Q3"
      ],
      "answer": "A3",
      "score": 75,
      "id": 17,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    },
    {
      "questions": [
        "Q4"
      ],
      "answer": "A4",
      "score": 50,
      "id": 18,
      "source": "Editorial",
      "metadata": [
        {
          "name": "topic",
          "value": "value"
        }
      ]
    }
  ]
}
```

QnA Maker ne saura pas quelle est la meilleure réponse. Utilisez la liste de suggestions du portail QnA Maker pour sélectionner la meilleure réponse et effectuer à nouveau l’apprentissage.


## <a name="how-you-give-explicit-feedback-with-the-train-api"></a>Comment donner des commentaires explicites avec l’API Train

QnA Maker a besoin de commentaires explicites pour savoir quelles réponses sont les meilleures. La façon dont la meilleure réponse est déterminée dépend de vous et peut inclure :

* Les commentaires des utilisateurs, en sélectionnant l’une des réponses.
* La logique métier, par exemple déterminer une plage de score acceptable.
* Une combinaison des commentaires des utilisateurs et de la logique métier.

Utilisez l’[API Train](/rest/api/cognitiveservices/qnamaker4.0/runtime/train) pour envoyer la réponse correcte à QnA Maker une fois que l’utilisateur l’a sélectionnée.

## <a name="next-step"></a>Étape suivante

> [!div class="nextstepaction"]
> [Interroger la base de connaissances](query-knowledge-base.md)