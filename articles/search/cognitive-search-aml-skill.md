---
title: Compétence AML personnalisée dans des ensembles de compétences
titleSuffix: Azure Cognitive Search
description: Étendez les capacités des ensembles de compétences Recherche cognitive Azure à l’aide de modèles Azure Machine Learning.
manager: nitinme
author: mattmsft
ms.author: magottei
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/12/2020
ms.openlocfilehash: 6cefe543ea8ba992b028448070bf041a77bfec64
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97630273"
---
# <a name="aml-skill-in-an-azure-cognitive-search-enrichment-pipeline"></a>Compétence AML dans un pipeline d’enrichissement Recherche cognitive Azure

> [!IMPORTANT] 
> Cette compétence est actuellement en préversion publique. Les fonctionnalités en préversion sont fournies sans contrat de niveau de service et ne sont pas recommandées pour les charges de travail de production. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Le SDK .NET n’est actuellement pas pris en charge.

La compétence **AML** vous permet d’étendre l’enrichissement par IA à l’aide d’un modèle [Azure Machine Learning](../machine-learning/overview-what-is-azure-ml.md) (AML) personnalisé. Une fois qu’un modèle AML est [formé et déployé](../machine-learning/concept-azure-machine-learning-architecture.md#workspace), une compétence **AML** l’intègre dans l’enrichissement par IA.

À l’instar des compétences intégrées, une compétence **AML** a des entrées et des sorties. Les entrées sont envoyées à votre service AML déployé sous la forme d’un objet JSON, qui génère une charge utile JSON en réponse avec un code d’état de réussite. La réponse est censée avoir les sorties spécifiées par votre compétence **AML**. Toute autre réponse est considérée comme une erreur et aucun enrichissement n’est effectué.

> [!NOTE]
> L’indexeur réessaie deux fois pour certains codes d’état HTTP standard retournés par le service AML. Ces codes d’état HTTP sont les suivants :
> * `503 Service Unavailable`
> * `429 Too Many Requests`

## <a name="prerequisites"></a>Prérequis

* Un [espace de travail AML](../machine-learning/concept-workspace.md)
* Une [cible de calcul AML Azure Kubernetes Service](../machine-learning/concept-compute-target.md) dans cet espace de travail avec un [modèle déployé](../machine-learning/how-to-deploy-azure-kubernetes-service.md)
  * La [cible de calcul doit permettre l’utilisation du protocole SSL](../machine-learning/how-to-secure-web-service.md#deploy-on-azure-kubernetes-service). Recherche cognitive Azure autorise uniquement l’accès aux points de terminaison **https**.
  * Les certificats autosignés ne peuvent pas être utilisés.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Custom.AmlSkill

## <a name="skill-parameters"></a>Paramètres de la compétence

Les paramètres respectent la casse. Les paramètres que vous choisissez d’utiliser dépendent de [l’authentification dont votre service AML a besoin, le cas échéant](#WhatSkillParametersToUse).

| Nom du paramètre | Description |
|--------------------|-------------|
| `uri` | (Requis [en l’absence d’authentification ou pour une authentification par clé](#WhatSkillParametersToUse)) [URI de scoring du service AML](../machine-learning/how-to-consume-web-service.md) auquel la charge utile _JSON_ sera envoyée. Seul le schéma d’URI **https** est autorisé. |
| `key` | (Requis pour l’[authentification par clé](#WhatSkillParametersToUse)) [Clé du service AML](../machine-learning/how-to-consume-web-service.md#authentication-with-keys). |
| `resourceId` | (Requis pour l’[authentification par jeton](#WhatSkillParametersToUse)). ID de ressource Azure Resource Manager du service AML. Il doit être au format subscriptions/{guid}/resourceGroups/{resource-group-name}/Microsoft.MachineLearningServices/workspaces/{workspace-name}/services/{service_name}. |
| `region` | (Facultatif pour l’[authentification par jeton](#WhatSkillParametersToUse)). [Région](https://azure.microsoft.com/global-infrastructure/regions/) dans laquelle le service AML est déployé. |
| `timeout` | (Facultatif) Si spécifié, indique le délai d’expiration pour le client http qui effectue l’appel d’API. Il doit être formaté en tant que valeur « dayTimeDuration » XSD (un sous-ensemble limité d'une valeur de [durée ISO 8601](https://www.w3.org/TR/xmlschema11-2/#dayTimeDuration) ). Par exemple, `PT60S` pour 60 secondes. S’il n’est pas défini, une valeur par défaut de 30 secondes est choisie. Le délai d’expiration peut être défini sur 230 secondes maximum et 1 seconde minimum. |
| `degreeOfParallelism` | (Facultatif) Lorsqu’il est spécifié, indique le nombre d’appels que l’indexeur effectuera en parallèle au point de terminaison que vous avez fourni. Vous pouvez réduire cette valeur si votre point de terminaison échoue avec une charge de requête trop élevée, ou l’augmenter si votre point de terminaison est en mesure d’accepter plus de requêtes et si vous souhaitez augmenter les performances de l’indexeur.  S’il n’est pas défini, une valeur par défaut de 5 secondes est utilisée. Le degreeOfParallelism peut avoir une valeur maximale de 10 et un minimum de 1.

<a name="WhatSkillParametersToUse"></a>

## <a name="what-skill-parameters-to-use"></a>Paramètres de compétence à utiliser

Les paramètres de compétence AML requis dépendent de l’authentification utilisée par votre service AML, le cas échéant. Les services AML fournissent trois options d’authentification :

* [Authentification basée sur une clé](../machine-learning/how-to-authenticate-web-service.md#key-based-authentication). Une clé statique est fournie pour authentifier les demandes de scoring des compétences AML
  * Utiliser les paramètres _uri_ et _key_
* [Authentification basée sur un jeton](../machine-learning/how-to-authenticate-web-service.md#token-based-authentication). Le service AML est [déployé à l’aide de l’authentification basée sur un jeton](../machine-learning/how-to-authenticate-web-service.md#token-based-authentication). L’[identité managée](../active-directory/managed-identities-azure-resources/overview.md) du service Recherche cognitive Azure reçoit le [rôle Lecteur](../machine-learning/how-to-assign-roles.md) dans l’espace de travail du service AML. La compétence AML utilise ensuite l’identité managée du service Recherche cognitive Azure pour s’authentifier auprès du service AML, sans qu’aucune clé statique soit nécessaire.
  * Utiliser le paramètre _resourceId_
  * Si le service Recherche cognitive Azure se trouve dans une autre région que celle de l’espace de travail AML, utilisez le paramètre _region_ pour définir la région dans laquelle le service AML a été déployé.
* Aucune authentification. Aucune authentification n’est requise pour utiliser le service AML
  * Utiliser le paramètre _uri_

## <a name="skill-inputs"></a>Entrées de la compétence

Il n’y a pas d’entrée « prédéfinie » pour cette compétence. Si vous choisissez comme entrées un ou plusieurs champs déjà disponibles au moment de l’exécution de cette compétence, la charge utile _JSON_ envoyée au service AML aura des champs différents.

## <a name="skill-outputs"></a>Sorties de la compétence

Il n’y a pas de sortie « prédéfinie » pour cette compétence. En fonction de la réponse envoyée par votre service AML, ajoutez des champs de sortie à récupérer dans la réponse _JSON_.

## <a name="sample-definition"></a>Exemple de définition

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
    "description": "A sample model that detects the language of sentence",
    "uri": "https://contoso.count-things.com/score",
    "context": "/document",
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "detected_language_code"
      }
    ]
  }
```

## <a name="sample-input-json-structure"></a>Exemple de structure JSON d’entrée

Cette structure _JSON_ représente la charge utile à envoyer à votre service AML. Les champs de haut niveau de la structure correspondront aux « noms » spécifiés dans la section `inputs` de la définition de compétence. La valeur de ces champs provient de la `source` de ces champs (qui peut être un champ dans le document ou éventuellement une autre compétence)

```json
{
  "text": "Este es un contrato en Inglés"
}
```

## <a name="sample-output-json-structure"></a>Exemple de structure JSON de sortie

Le terme « output » correspond à la réponse renvoyée par votre service AML. Le service AML ne doit retourner qu’une charge utile _JSON_ (vérifiée en examinant l’en-tête de réponse `Content-Type`) et doit être un objet dans lequel les champs sont des enrichissements correspondant aux « noms » dans la section `output` et dont la valeur est l’enrichissement.

```json
{
    "detected_language_code": "es"
}
```

## <a name="inline-shaping-sample-definition"></a>Exemple de définition de mise en forme inlined

```json
  {
    "@odata.type": "#Microsoft.Skills.Custom.AmlSkill",
    "description": "A sample model that detects the language of sentence",
    "uri": "https://contoso.count-things.com/score",
    "context": "/document",
    "inputs": [
      {
        "name": "shapedText",
        "sourceContext": "/document",
        "inputs": [
            {
              "name": "content",
              "source": "/document/content"
            }
        ]
      }
    ],
    "outputs": [
      {
        "name": "detected_language_code"
      }
    ]
  }
```

## <a name="inline-shaping-input-json-structure"></a>Structure JSON d’entrée de mise en forme inlined

```json
{
  "shapedText": { "content": "Este es un contrato en Inglés" }
}
```

## <a name="inline-shaping-sample-output-json-structure"></a>Structure JSON d’exemple de sortie de mise en forme inlined

```json
{
    "detected_language_code": "es"
}
```

## <a name="error-cases"></a>Cas d’erreur
En plus de la non-disponibilité de votre service AML ou de l’envoi de codes d’état non réussis, les cas suivants sont considérés comme erronés :

* Si le service AML retourne un code d’état de réussite, mais que la réponse indique que ce n’est pas `application/json`, la réponse est considérée non valide et aucun enrichissement n’est effectué.
* Si le service AML retourne un JSON non valide

Quand le service AML n’est pas disponible ou retourne une erreur HTTP, une erreur conviviale avec tous les détails disponibles sur l’erreur HTTP est ajoutée à l’historique des exécutions de l’indexeur.

## <a name="see-also"></a>Voir aussi

+ [Guide pratique pour définir un ensemble de compétences](cognitive-search-defining-skillset.md)
+ [Résolution des problèmes liés au service AML](../machine-learning/how-to-troubleshoot-deployment.md)