---
title: Modération de texte - Content Moderator
titleSuffix: Azure Cognitive Services
description: Utilisez la modération de texte pour détecter un texte indésirable, des données à caractère personnel et des listes de termes personnalisées.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 05/18/2020
ms.author: pafarley
ms.openlocfilehash: ae49a8738ba711ac6c77f2e299852ad61f70be56
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92912903"
---
# <a name="learn-text-moderation-concepts"></a>Familiarisez-vous avec les concepts de modération de texte

Utilisez les modèles de modération de texte de Content Moderator pour analyser le contenu du texte.

Vous pouvez bloquer, approuver ou passer en revue le contenu en fonction de vos stratégies et seuils (Consultez [Révisions, flux de travail et travaux](./review-api.md) pour savoir comment configurer les révisions humaines). Utilisez les modèles de modération de texte pour augmenter la modération humaine dans des environnements où des partenaires, des employés et des consommateurs génèrent du contenu de texte. Ces environnements incluent notamment les salles de conversation, les forums de discussion, les bots conversationnels, les catalogues d’e-commerce et les documents.

La réponse du service inclut les informations suivantes :

- Vulgarité : correspondance basée sur des termes avec une liste prédéfinie de termes injurieux dans différentes langues
- Classification : classification en trois catégories assistée par ordinateur
- Données à caractère personnel
- Texte corrigé automatiquement
- Texte d’origine
- Langage

## <a name="profanity"></a>Termes vulgaires

Si l’API détecte des termes injurieux dans l’une des [langues prises en charge](./language-support.md), ces termes sont inclus dans la réponse. La réponse contient également leur emplacement (`Index`) dans le texte d’origine. La valeur `ListId` dans l’exemple JSON suivant fait référence à des termes se trouvant dans les [listes de termes personnalisées](try-terms-list-api.md), le cas échéant.

```json
"Terms": [
    {
        "Index": 118,
        "OriginalIndex": 118,
        "ListId": 0,
        "Term": "crap"
    }
```

> [!NOTE]
> Pour le paramètre **langue** , attribuez `eng` ou laissez-le vide pour voir la réponse de la **classification** assistée par ordinateur (fonctionnalité en préversion). **Cette fonctionnalité prend en charge uniquement l’anglais**.
>
> Pour la détection des **termes injurieux** , utilisez le [code ISO 639-3](http://www-01.sil.org/iso639-3/codes.asp) des langues prises en charge répertoriées dans cet article, ou laissez le paramètre vide.

## <a name="classification"></a>classification ;

La **fonctionnalité de classification de texte** assistée par ordinateur de Content Moderator, qui prend en charge **uniquement l’anglais** , permet de détecter le contenu potentiellement indésirable. Le contenu marqué d’un indicateur peut être considéré comme inapproprié en fonction du contexte. La classification de texte informe de cette probabilité pour chaque catégorie et peut vous recommander une révision par des humains. Cette fonctionnalité utilise un modèle entraîné pour identifier un langage potentiellement abusif, dépréciatif ou discriminatoire. Cela inclut l’argot, les mots abrégés, les mots offensants et les mots intentionnellement mal orthographiés à réviser. 

L’extrait suivant de l’extrait de code JSON présente un exemple de sortie :

```json
"Classification": {
    "ReviewRecommended": true,
    "Category1": {
        "Score": 1.5113095059859916E-06
    },
    "Category2": {
        "Score": 0.12747249007225037
    },
    "Category3": {
        "Score": 0.98799997568130493
    }
}
```

### <a name="explanation"></a>Explication

- `Category1` se réfère à la présence potentielle de langage pouvant être considéré comme sexuellement explicite ou réservé aux adultes dans certaines situations.
- `Category2` se réfère à la présence potentielle de langage pouvant être considéré comme sexuellement suggestif ou réservé aux adultes dans certaines situations.
- `Category3` se réfère à la présence potentielle de langage pouvant être considéré comme choquant dans certaines situations.
- `Score` est compris entre 0 et 1. Plus le score est élevé, plus le modèle prédit que la catégorie peut être applicable. Cette fonctionnalité s’appuie sur un modèle statistique plutôt que sur des résultats codés manuellement. Nous vous recommandons d’effectuer le test avec votre propre contenu afin de déterminer comment chaque catégorie correspond à vos besoins.
- La valeur de `ReviewRecommended` est true ou false, selon les seuils de score internes. Les clients doivent évaluer l’opportunité d’utiliser cette valeur ou choisir des seuils personnalisés en fonction de leurs stratégies de contenu.

## <a name="personal-data"></a>Données à caractère personnel

La fonctionnalité de données à caractère personnel détecte la présence potentielle des informations suivantes :

- Adresse de messagerie
- Adresse postale aux États-Unis
- Adresse IP
- Numéro de téléphone aux États-Unis

L’exemple suivant illustre une réponse :

```json
"pii":{
  "email":[
      {
        "detected":"abcdef@abcd.com",
        "sub_type":"Regular",
        "text":"abcdef@abcd.com",
        "index":32
      }
  ],
  "ssn":[

  ],
  "ipa":[
      {
        "sub_type":"IPV4",
        "text":"255.255.255.255",
        "index":72
      }
  ],
  "phone":[
      {
        "country_code":"US",
        "text":"6657789887",
        "index":56
      }
  ],
  "address":[
      {
        "text":"1 Microsoft Way, Redmond, WA 98052",
        "index":89
      }
  ]
}
```

## <a name="auto-correction"></a>Correction automatique

Supposons que le texte d’entrée soit le suivant (les fautes « lzay » et « f0x » sont intentionnelles) :

> The qu!ck brown f0x jumps over the lzay dog.

Si vous demandez la correction automatique, la réponse contient la version corrigée du texte :

> The quick brown fox jumps over the lazy dog.

## <a name="creating-and-managing-your-custom-lists-of-terms"></a>Création et gestion de vos listes personnalisées de termes

Bien que la liste globale par défaut de termes fonctionne parfaitement pour la plupart des cas, vous pouvez passer au crible un texte pour détecter des termes qui sont spécifiques à vos besoins professionnels. Vous pouvez, par exemple, filtrer tous les noms de marques de concurrents dans des posts d’utilisateurs.

> [!NOTE]
> Il existe une limite maximale de **5 listes de termes** , chaque liste ne devant **pas dépasser 10 000 termes**.
>

L’exemple suivant présente l’ID de liste correspondant :

```json
"Terms": [
    {
        "Index": 118,
        "OriginalIndex": 118,
        "ListId": 231.
        "Term": "crap"
    }
```

Content Moderator fournit une [API de liste de termes](https://westus.dev.cognitive.microsoft.com/docs/services/57cf755e3f9b070c105bd2c2/operations/57cf755e3f9b070868a1f67f) avec des opérations pour la gestion des listes de termes personnalisées. Commencez avec la [Console d’API de listes de termes personnalisées](try-terms-list-api.md) et utilisez les exemples de code API REST. Consultez également le [démarrage rapide .NET des listes de termes](term-lists-quickstart-dotnet.md) si vous connaissez déjà Visual Studio et C#.

## <a name="next-steps"></a>Étapes suivantes

Testez les API à l’aide de la [console API de modération de texte](try-text-api.md). Consultez également [Révisions, flux de travail et travaux](./review-api.md) pour en savoir plus sur le mode de configuration des révisions humaines.