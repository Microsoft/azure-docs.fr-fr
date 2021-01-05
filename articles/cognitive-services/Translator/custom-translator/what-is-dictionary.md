---
title: Qu’est un dictionnaire ? - Custom Translator
titleSuffix: Azure Cognitive Services
description: Un dictionnaire est un document aligné spécifiant une liste d’expressions ou de phrases (avec leur traduction) que Microsoft Translator devra traduire toujours de la même façon. Les dictionnaires sont parfois appelés glossaires ou bases terminologiques.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: conceptual
ms.openlocfilehash: 14ff76e090e628a880290bb0e9f90026260de0c7
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368982"
---
# <a name="what-is-a-dictionary"></a>Qu’est un dictionnaire ?

Un dictionnaire est une paire alignée de documents qui spécifie une liste d’expressions ou de phrases avec les traductions correspondantes. Utilisez un dictionnaire dans votre apprentissage si vous souhaitez que Microsoft Translator traduise toujours toutes les instances de l’expression ou de la phrase source selon la traduction que vous avez indiquée dans le dictionnaire. Les dictionnaires sont parfois appelés glossaires ou bases terminologiques. On peut considérer le dictionnaire comme un « copier-remplacer » en force pour tous les termes listés. En outre, le service Microsoft Custom Translator crée et utilise ses propres dictionnaires à usage général pour améliorer la qualité de sa traduction. Toutefois, un dictionnaire fourni par le client est prioritaire et fera en premier l’objet de recherches pour des mots ou des phrases.

Les dictionnaires ne fonctionnent que pour les projets des paires de langues qui s’appuient sur un modèle de réseau neuronal global Microsoft entièrement pris en charge. [Affichez la liste complète des langues](../language-support.md#customization).

## <a name="phrase-dictionary"></a>Dictionnaire d’expressions
Le dictionnaire d’expressions respecte la casse. Il s’agit d’une opération de recherche et de remplacement exacts. Si un dictionnaire d’expressions est ajouté dans le cadre de l’apprentissage du modèle, tous les mots et expressions listés sont traduits de la manière spécifiée. La traduction du reste de la phrase reste inchangée. Vous pouvez utiliser un dictionnaire d’expressions pour spécifier des expressions à ne pas traduire : indiquez pour cela la même expression non traduite dans le fichier source et dans le fichier cible du dictionnaire.

## <a name="sentence-dictionary"></a>Dictionnaire de phrases
Le dictionnaire de phrases ne respecte pas la casse. Le dictionnaire de phrases permet de spécifier la traduction cible exacte d’une phrase source. La totalité (et pas seulement une partie) de la phrase soumise doit être identique à l’entrée de dictionnaire source Si l’entrée du dictionnaire source se termine par un signe de ponctuation, elle est ignorée pendant la comparaison. pour qu’il y ait correspondance.  En cas de correspondance, l’entrée cible du dictionnaire de phrases est retournée.

## <a name="dictionary-only-trainings"></a>Apprentissage sur dictionnaire uniquement
Il est possible d’entraîner un modèle en n’utilisant que des données de dictionnaire. Pour cela, sélectionnez seulement le ou les documents de type dictionnaire que vous souhaitez inclure, puis appuyez sur Créer un modèle. Comme il s’agit d’un apprentissage sur dictionnaire uniquement, il n’y a pas de nombre minimal de phrases d’apprentissage nécessaires. L’apprentissage est généralement beaucoup plus rapide pour ce type de modèle que dans le cas standard.  Les modèles obtenus utilisent les modèles de référence de Microsoft pour la traduction, en y joignant les dictionnaires ajoutés.  Aucun rapport de test n’est généré.

>[!Note]
>Dans la mesure où Custom Translator n’aligne pas les fichiers de dictionnaire en phrases, il est important qu’il y ait le même nombre d’expressions/phrases sources et cibles dans les documents de dictionnaire et qu’elles soient précisément alignées.

## <a name="recommendations"></a>Recommandations

- Les dictionnaires ne peuvent pas remplacer l’entraînement d’un modèle avec les données d’entraînement. Il est préférable de laisser le système apprendre à partir de vos données d’entraînement plutôt que d’utiliser un dictionnaire. Toutefois, lorsque des phrases ou des noms composés doivent être rendus tels quels, utilisez un dictionnaire.
- Le dictionnaire d’expressions doit être utilisé avec parcimonie. Par conséquent, gardez à l’esprit que quand une expression est remplacée au sein d’une phrase, le contexte de cette phrase est perdu ou limité pour traduire le reste de la phrase. D’un côté, l’expression ou le mot dans la phrase est traduit en fonction du dictionnaire fourni mais, de l’autre, la qualité de traduction globale de la phrase s’en trouve souvent diminuée.
- Le dictionnaire d’expressions fonctionne bien pour les noms composés comme les noms de produits (« Microsoft SQL Server »), les noms propres (« ville de Hambourg ») et les fonctionnalités de produit (« tableau croisé dynamique »). Il n’est pas aussi performant pour les verbes et les adjectifs, car ceux-ci sont généralement fléchis dans la langue source ou la langue cible. Une bonne pratique consiste à éviter les entrées de dictionnaire d’expressions pour tout ce qui n’est pas un nom composé.
- Lorsque vous utilisez un dictionnaire d’expressions, la casse et la ponctuation sont importantes. Les entrées de dictionnaire ne correspondent qu’à des mots et des expressions dans la phrase d’entrée qui utilisent exactement la même casse et la même ponctuation que celles spécifiées dans le fichier de dictionnaire source. En outre, les traductions reflètent la casse et la ponctuation fournies dans le fichier de dictionnaire cible. Par exemple, si vous avez entraîné un système traduisant de l’anglais en espagnol qui utilise un dictionnaire d’expressions spécifiant « US » dans le fichier source et « EE.UU. » dans le fichier cible, quand vous demandez la traduction d’une phrase qui comprend le mot « us » (en minuscules), il n’existe PAS de correspondance dans le dictionnaire. Toutefois, si vous demandez la traduction d’une phrase qui contient le mot « US » (en majuscules), une correspondance est trouvée dans le dictionnaire et la traduction contient « EE.UU. » Notez que la casse et la ponctuation de la traduction peuvent être différentes de celles spécifiées dans le fichier cible du dictionnaire et peuvent être différentes de la casse et de la ponctuation dans la source. Elles suivent les règles de la langue cible.
- Lorsque vous utilisez un dictionnaire de phrases, la ponctuation finale de la phrase est ignorée. Par exemple, si votre dictionnaire source contient « Cette phrase se termine par un signe de ponctuation ! », toutes les requêtes de traduction contenant « Cette phrase se termine par un signe de ponctuation » correspondent.
- Si un mot apparaît plusieurs fois dans un fichier de dictionnaire, le système utilise toujours la dernière entrée indiquée. Par conséquent, votre dictionnaire ne doit pas contenir plusieurs traductions pour un même mot.

## <a name="next-steps"></a>Étapes suivantes

- Lisez les [instructions sur les formats de documents](document-formats-naming-convention.md).