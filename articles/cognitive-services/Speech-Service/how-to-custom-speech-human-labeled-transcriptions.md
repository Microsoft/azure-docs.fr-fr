---
title: Recommandations concernant les transcriptions étiquetées à la main – Service Speech
titleSuffix: Azure Cognitive Services
description: Pour améliorer la précision de la reconnaissance vocale, par exemple lorsque des mots sont supprimés ou remplacés de manière incorrecte, vous pouvez utiliser des transcriptions étiquetées à la main avec vos données audio. Les transcriptions étiquetées à la main sont des transcriptions mot pour mot et littérales d’un fichier audio.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: erhopf
ms.openlocfilehash: 4516bbddd3fda593021288a440e1b354d4d7f1e3
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96340235"
---
# <a name="how-to-create-human-labeled-transcriptions"></a>Comment créer des transcriptions étiquetées à la main

Si vous cherchez à améliorer la précision de la reconnaissance, en particulier les problèmes liés à la suppression de mots ou à leur substitution inappropriée, vous pouvez utiliser des transcriptions étiquetées à la main avec vos données audio. En quoi consistent les transcriptions étiquetées à la main ? Il s’agit tout simplement de transcriptions textuelles, mot par mot, d’un fichier audio.

Sachant que l’amélioration de la reconnaissance nécessite un échantillon volumineux de données de transcription, nous vous conseillons de fournir entre 10 et 1 000 heures de données de transcription. Dans cette page, nous allons passer en revue les principes à respecter pour créer des transcriptions de grande qualité. Ce guide est divisé par paramètres régionaux, avec des sections pour l’anglais des États-Unis, le chinois mandarin et l’allemand.

## <a name="us-english-en-us"></a>Anglais (États-Unis, en-US)

Les transcriptions étiquetées à la main pour le contenu audio en anglais doivent être fournies sous forme de texte brut, uniquement avec des caractères ASCII. Évitez d’utiliser des caractères de ponctuation Latin-1 ou Unicode. Ces caractères sont souvent ajoutés par inadvertance pendant la copie de texte dans une application de traitement de texte la capture des données dans des pages web. Si ces caractères sont présents, veillez à les mettre à jour par une substitution ASCII appropriée.

Voici quelques exemples :

| Caractères à éviter | Substitution | Notes |
| ------------------- | ------------ | ----- |
| “Hello world” | "Hello world" | Les guillemets typographiques ouvrants et fermants ont été substitués par des caractères ASCII appropriés. |
| John’s day | John's day | L’apostrophe a été substituée par le caractère ASCII approprié. |
| it was good—no, it was great! | it was good--no, it was great! | Le tiret cadratin a été substitué par deux traits d’union. |

### <a name="text-normalization-for-us-english"></a>Normalisation du texte pour l’anglais des États-Unis

La normalisation du texte consiste à transformer les mots dans un format cohérent utilisé pendant l’entraînement d’un modèle. Si certaines règles de normalisation sont automatiquement appliquées au texte, nous vous recommandons cependant de suivre ces recommandations pendant la phase de préparation de données de transcription étiquetées à la main :

- Développez les abréviations.
- Écrivez les chaînes numériques non standard en lettres (comme les termes de comptabilité).
- Les caractères non alphabétiques ou les caractères alphanumériques mixtes doivent être retranscrits comme ils se prononcent.
- Les abréviations prononcées comme des mots n’ont pas besoin d’être modifiés (par exemple, « radar », « laser », « RAM » ou « OTAN »).
- Écrivez les abréviations dont les lettres se prononcent en les séparant avec un espace.

Voici quelques exemples de normalisation à effectuer au niveau de la transcription :

| Texte d’origine               | Texte après la normalisation              |
| --------------------------- | ------------------------------------- |
| Dr. Bruce Banner            | Docteur Bruce Banner                   |
| James Bond, 007             | James Bond, double oh seven           |
| Ke$ha                       | Kesha                                 |
| How long is the 2x4         | How long is the two by four           |
| The meeting goes from 1-3pm | The meeting goes from one to three pm |
| My blood type is O+         | My blood type is O positive           |
| Water is H20                | Water is H 2 O                        |
| Play OU812 by Van Halen     | Play O U 8 1 2 by Van Halen           |
| UTF-8 with BOM              | U T F 8 with BOM                      |

Les règles de normalisation suivantes sont automatiquement appliquées aux transcriptions :

- Utilisation de lettres minuscules
- Suppression de tous les signes de ponctuation, à l’exception des apostrophes dans les mots
- Écriture des nombres en lettres/forme parlée, comme les montants en dollars

Voici quelques exemples de normalisation effectuée automatiquement au niveau de la transcription :

| Texte d’origine                          | Texte après la normalisation          |
| -------------------------------------- | --------------------------------- |
| "Holy cow!" said Batman.               | holy cow said batman              |
| "What?" said Batman’s sidekick, Robin. | what said batman’s sidekick robin |
| Go get -em!                            | go get em                         |
| I’m double-jointed                     | I'm double jointed                |
| 104 Elm Street                         | one oh four Elm street            |
| Tune to 102.7                          | tune to one oh two point seven    |
| Pi is about 3.14                       | pi is about three point one four  |
| It costs \$3.14                        | it costs three fourteen           |

## <a name="mandarin-chinese-zh-cn"></a>Mandarin chinois (zh-CN)

Les transcriptions étiquetées à la main pour le contenu audio en chinois mandarin doivent être encodées en UTF-8 avec un indicateur d’ordre des octets. Évitez l’utilisation des signes de ponctuation de demi-largeur. Ces caractères peuvent être inclus par inadvertance pendant la préparation des données dans un programme de traitement ou la récupération de données dans des pages web. Si ces caractères sont présents, veillez à les mettre à jour par une substitution appropriée avec des caractères à pleine chasse.

Voici quelques exemples :

| Caractères à éviter | Substitution   | Notes |
| ------------------- | -------------- | ----- |
| "你好" | "你好" | Les guillemets typographiques ouvrants et fermants ont été substitués par des caractères appropriés. |
| 需要什么帮助? | 需要什么帮助？| Le point d’interrogation a été substitué par un caractère approprié. |

### <a name="text-normalization-for-mandarin-chinese"></a>Normalisation de texte pour le chinois Mandarin

La normalisation du texte consiste à transformer les mots dans un format cohérent utilisé pendant l’entraînement d’un modèle. Si certaines règles de normalisation sont automatiquement appliquées au texte, nous vous recommandons cependant de suivre ces recommandations pendant la phase de préparation de données de transcription étiquetées à la main :

- Développez les abréviations.
- Écrivez les chaînes numériques sous leur forme orale.

Voici quelques exemples de normalisation à effectuer au niveau de la transcription :

| Texte d’origine | Texte après la normalisation |
| ------------- | ------------------------ |
| 我今年 21 | 我今年二十一 |
| 3 号楼 504 | 三号 楼 五 零 四 |

Les règles de normalisation suivantes sont automatiquement appliquées aux transcriptions :

- Suppression de tous les signes de ponctuation
- Écriture des nombres en forme parlée
- Conversion des lettres à pleine chasse en lettres à demi-chasse
- Utilisation de lettres majuscules pour tous les mots anglais

Voici quelques exemples de normalisation effectuée automatiquement au niveau de la transcription :

| Texte d’origine | Texte après la normalisation |
| ------------- | ------------------------ |
| 3.1415 | 三 点 一 四 一 五 |
| ￥ 3.5 | 三 元 五 角 |
| w f y z | W F Y Z |
| 1992 年 8 月 8 日 | 一 九 九 二 年 八 月 八 日 |
| 你吃饭了吗? | 你 吃饭 了 吗 |
| 下午 5:00 的航班 | 下午 五点 的 航班 |
| 我今年 21 岁 | 我 今年 二十 一 岁 |

## <a name="german-de-de-and-other-languages"></a>Allemand (de-DE) et autres langages

Les transcriptions étiquetées à la main pour le contenu audio en allemand (et dans d’autres langues que l’anglais et le chinois mandarin) doivent être encodées en UTF-8 avec un indicateur d’ordre des octets. Une transcription étiquetée à la main doit être fournie pour chaque fichier audio.

### <a name="text-normalization-for-german"></a>Normalisation de texte pour l’allemand

La normalisation du texte consiste à transformer les mots dans un format cohérent utilisé pendant l’entraînement d’un modèle. Si certaines règles de normalisation sont automatiquement appliquées au texte, nous vous recommandons cependant de suivre ces recommandations pendant la phase de préparation de données de transcription étiquetées à la main :

- Écrire les décimales sous la forme « , » et non « . ».
- Écrire des séparateurs d’heure sous la forme « : » et non « . » (par exemple : 12:00 Uhr).
- Les abréviations telles que « ca ». ne sont pas remplacées. Nous vous recommandons d’utiliser la forme parlée complète.
- Les quatre principaux opérateurs mathématiques (+, -, \*, /) sont supprimés. Nous vous recommandons de les remplacer par leur forme écrite : « plus », « minus », « mal » et « geteilt ».
- Les opérateurs de comparaison sont supprimés (=, <, et >). Nous vous recommandons de les remplacer par « gleich », « kleiner als » et « grösser als ».
- Écrivez les fractions, telles que 3/4, sous forme écrite (par exemple, « drei viertel » au lieu de 3/4).
- Remplacez le symbole « € » par la forme écrite « Euro ».

Voici quelques exemples de normalisation à effectuer au niveau de la transcription :

| Texte d’origine    | Texte après la normalisation utilisateur | Texte après la normalisation système       |
| ---------------- | ----------------------------- | ------------------------------------- |
| Es ist 12.23 Uhr | Es ist 12:23 Uhr              | es ist zwölf uhr drei und zwanzig uhr |
| {12.45}          | {12,45}                       | zwölf komma vier fünf                 |
| 2 + 3 - 4        | 2 plus 3 minus 4              | zwei plus drei minus vier             |

Les règles de normalisation suivantes sont automatiquement appliquées aux transcriptions :

- Utilisation de lettres minuscules pour tout le texte
- Suppression de toute la ponctuation, notamment les différents types de guillemets ("test", ’test’, "test„ et «test» sont acceptés)
- Suppression des lignes contenant un des caractères spéciaux suivants : ¢ ¤ ¥ ¦ § © ª ¬ ® ° ± ² µ × ÿ Ø¬¬
- Écriture des nombres en forme parlée, y compris les montants en dollars ou en euros
- Acceptation des trémas uniquement pour a, o et u. Les autres seront remplacées par « th » ou ignorés.

Voici quelques exemples de normalisation effectuée automatiquement au niveau de la transcription :

| Texte d’origine    | Texte après la normalisation |
| ---------------- | ------------------------ |
| Frankfurter Ring | frankfurter ring         |
| ¡Eine Frage!     | eine frage               |
| wir, haben       | wir haben                |

## <a name="next-steps"></a>Étapes suivantes

- [Préparer et tester les données](./how-to-custom-speech-test-and-train.md)
- [Inspecter les données](how-to-custom-speech-inspect-data.md)
- [Évaluer les données](how-to-custom-speech-evaluate-data.md)
- [Entraîner un modèle](how-to-custom-speech-train-model.md)
- [Déployer un modèle](./how-to-custom-speech-train-model.md)