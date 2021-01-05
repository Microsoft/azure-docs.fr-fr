---
title: Nombre de caractères - Translator
titleSuffix: Azure Cognitive Services
description: Cet article explique comment le service Translator d’Azure Cognitive Services compte les caractères pour vous permettre de comprendre comment il ingère le contenu.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 05/26/2020
ms.author: swmachan
ms.openlocfilehash: 7fad90cf47591b5ffab4232c2be3180138738f01
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83992827"
---
# <a name="how-the-translator-counts-characters"></a>Comment le service Translator compte les caractères

Le service Translator comptabilise chaque point de code Unicode du texte entré comme un caractère. Chaque traduction d’un texte dans une autre langue est comptabilisée comme une seule traduction, même si la requête a été effectuée dans un même appel d’API effectuant une traduction dans plusieurs langues. La longueur de la réponse n’a pas d’importance.

Voici ce qui est pris en compte :

* Texte transmis au service Translator dans le corps de la requête
   * `Text` lorsque vous utilisez les méthodes Translate, Transliterate et Dictionary Lookup
   * `Text` et `Translation` lorsque vous utilisez la méthode Dictionary Examples
* Toutes les balises : balises HTML, XML, etc. dans le champ de texte du corps de la requête. La notation JSON utilisée pour générer la requête (par exemple « Text: ») n’est pas prise en compte.
* Une lettre seule
* Ponctuation
* Un espace, une tabulation, une balise et tout type d’espace blanc
* Chaque point de code défini au format Unicode
* Une répétition de traduction, même si vous avez traduit ce même texte précédemment

Pour les scripts basés sur des idéogrammes tels que les idéogrammes chinois et les kanji japonais, le service Translator comptabilise toujours le nombre de points de code Unicode, en comptant un caractère par idéogramme. Exception : Les substituts Unicode comptent comme deux caractères.

Le nombre de requêtes, de mots, d’octets ou de phrases est sans importance dans la comptabilisation des caractères.

Les appels aux méthodes Detect et BreakSentence ne sont pas comptés dans l’utilisation de caractères. Toutefois, nous nous attendons à ce que la quantité d’appels aux méthodes Detect et BreakSentence soit raisonnable par rapport à l’utilisation des autres fonctions comptées. Si le nombre d’appels Detect ou BreakSentence que vous effectuez dépasse le nombre des autres méthodes comptabilisées multiplié par 100, Microsoft se réserve le droit de limiter votre utilisation des méthodes Detect et BreakSentence.

Vous trouverez plus d’informations sur les comptes de caractères dans la [FAQ du service Translator](https://www.microsoft.com/en-us/translator/faq.aspx).
