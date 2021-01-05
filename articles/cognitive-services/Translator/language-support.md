---
title: Prise en charge linguistique – Translator
titleSuffix: Azure Cognitive Services
description: Cognitive Services Translator prend en charge les langues suivantes pour la traduction de texte par traduction automatique neuronale.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 06/10/2020
ms.author: swmachan
ms.openlocfilehash: c7e859b05450b95cf9c75421599dd9c54a639cdc
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97606842"
---
# <a name="language-and-region-support-for-text-and-speech-translation"></a>Langues et régions prises en charge pour la traduction de texte et vocale

Utilisez Translator pour traduire vers et depuis plus de 70 langues de traduction de texte. La traduction automatique neuronale (NMT) constitue la nouvelle référence en matière de traductions automatiques de grande qualité reposant sur l’intelligence artificielle, et est proposée par défaut dans Translator v3 lorsqu’un système neuronal est disponible.

Vous pouvez aussi utiliser Translator en association avec Custom Translator pour créer des systèmes de traduction neuronaux qui comprennent la terminologie utilisée dans votre entreprise et votre secteur d’activité, et en association avec le service Microsoft Speech pour ajouter la traduction vocale à votre application.

[En savoir plus sur le fonctionnement de la traduction automatique](https://www.microsoft.com/translator/mt.aspx)

## <a name="text-translation"></a>Traduction de texte
La traduction de texte est disponible à l’aide de l’opération Translate vers ou à partir de l’une des langues disponibles dans Translator. L’API offre également la détection de langue à l’aide de l’opération Detect, la translittération à l’aide de l’opération Transliterate et des dictionnaires bilingues à l’aide des opérations Dictionary Lookup et Dictionary Examples. Les langues disponibles pour chacune de ces opérations sont répertoriées ci-dessous. 

### <a name="translate"></a>Translate

Translator prend en charge les langues suivantes pour la traduction de texte en texte. 

[Voir la documentation de référence de l’opération Translate](reference/v3-0-translate.md)

| Langage | Code langue |
|:-|:-:|
| Afrikaans | `af` |
| Arabe | `ar` |
| Assamais | `as` |
| Bangla | `bn` |
| Bosniaque (latin) | `bs` |
| Bulgare | `bg` |
| Cantonais (traditionnel) | `yue` |
| Catalan | `ca` |
| Chinois (simplifié) | `zh-Hans` |
| Chinois traditionnel | `zh-Hant` |
| Croate | `hr` |
| Tchèque | `cs` |
| Dari | `prs` |
| Danois | `da` |
| Néerlandais | `nl` |
| Anglais | `en` |
| Estonien | `et` |
| Fidjien | `fj` |
| Filipino | `fil` |
| Finnois | `fi` |
| Français | `fr` |
| Français (Canada) | `fr-ca` |
| Allemand | `de` |
| Grec | `el` |
| Goudjrati | `gu` |
| Créole haïtien | `ht` |
| Hébreu | `he` |
| Hindi | `hi` |
| Hmong blanc | `mww` |
| Hongrois | `hu` |
| Islandais | `is` |
| Indonésien | `id` |
| Irlandais | `ga` |
| Italien | `it` |
| Japonais | `ja` |
| Kannada | `kn` |
| Kazakh | `kk` |
| Klingon | `tlh-Latn` |
| Klingon (plqaD) | `tlh-Piqd` |
| Coréen | `ko` |
| Kurde (central) | `ku` |
| Kurde (Nord) | `kmr` |
| Letton | `lv` |
| Lituanien | `lt` |
| Malgache | `mg` |
| Malais | `ms` |
| Malayalam | `ml` |
| Maltais | `mt` |
| Maori | `mi` |
| Marathi | `mr` |
| Norvégien | `nb` |
| Odia | `or` |
| Pachto | `ps` |
| Persan | `fa` |
| Polonais | `pl` |
| Portugais (Brésil) | `pt-br` |
| Portugais (Portugal) | `pt-pt` |
| Pendjabi | `pa` |
| Queretaro Otomi | `otq` |
| Roumain | `ro` |
| Russe | `ru` |
| Samoan | `sm` |
| Serbe (cyrillique) | `sr-Cyrl` |
| Serbe (latin) | `sr-Latn` |
| Slovaque | `sk` |
| Slovène | `sl` |
| Espagnol | `es` |
| Swahili | `sw` |
| Suédois | `sv` |
| Tahitien | `ty` |
| Tamoul | `ta` |
| Télougou | `te` |
| Thaï | `th` |
| Tonga | `to` |
| Turc | `tr` |
| Ukrainien | `uk` |
| Ourdou | `ur` |
| Vietnamien | `vi` |
| Gallois | `cy` |
| Yucatec Maya | `yua` |

> [!NOTE]
> Par défaut, le code de langue `pt` sera `pt-br`, Portugais (Brésil).

### <a name="detect"></a>Detect

Translator détecte les langues suivantes pour la traduction et la translittération.

[Voir la documentation de référence de l’opération Detect](reference/v3-0-detect.md)

| Langage | Code langue |
|:-|:-:|
| Afrikaans | `af` |
| Arabe | `ar` |
| Bulgare | `bg` |
| Catalan | `ca` |
| Chinois (simplifié) | `zh-Hans` |
| Chinois traditionnel | `zh-Hant` |
| Croate | `hr` |
| Tchèque | `cs` |
| Danois | `da` |
| Néerlandais | `nl` |
| Anglais | `en` |
| Estonien | `et` |
| Finnois | `fi` |
| Français | `fr` |
| Allemand | `de` |
| Grec | `el` |
| Goudjrati | `gu` |
| Créole haïtien | `ht` |
| Hébreu | `he` |
| Hindi | `hi` |
| Hongrois | `hu` |
| Islandais | `is` |
| Indonésien | `id` |
| Irlandais | `ga` |
| Italien | `it` |
| Japonais | `ja` |
| Klingon | `tlh-Latn` |
| Coréen | `ko` |
| Kurde (central) | `ku-Arab` |
| Letton | `lv` |
| Lituanien | `lt` |
| Malais | `ms` |
| Maltais | `mt` |
| Norvégien | `nb` |
| Pachto | `ps` |
| Persan | `fa` |
| Polonais | `pl` |
| Portugais | `pt` |
| Roumain | `ro` |
| Russe | `ru` |
| Serbe (cyrillique) | `sr-Cyrl` |
| Serbe (latin) | `sr-Latn` |
| Slovaque | `sk` |
| Slovène | `sl` |
| Espagnol | `es` |
| Swahili | `sw` |
| Suédois | `sv` |
| Tahitien | `ty` |
| Thaï | `th` |
| Turc | `tr` |
| Ukrainien | `uk` |
| Ourdou | `ur` |
| Vietnamien | `vi` |
| Gallois | `cy` |
| Yucatec Maya | `yua` |

### <a name="transliterate"></a>Transliterate

La méthode Transliterate prend en charge les langues ci-après. Dans la colonne « Vers/De », le symbole « <--> » indique que la langue peut être translittérée aussi bien à partir des scripts indiqués que vers ces derniers. Le symbole « --> » signifie que la langue peut uniquement être translittérée du premier script vers le second.

[Voir la documentation de référence de l’opération Transliterate](reference/v3-0-translate.md)


| Langage    | Code langue | Script | Vers/De | Script|
|:----------- |:-------------:|:-------------:|:-------------:|:-------------:|
| Arabe | `ar` | Arabe `Arab` | <--> | Latin `Latn` |
| Bangla  | `bn` | Bengali `Beng` | <--> | Latin `Latn` |
| Chinois (simplifié) | `zh-Hans` | Chinois (simplifié) `Hans`| <--> | Latin `Latn` |
| Chinois (simplifié) | `zh-Hans` | Chinois (simplifié) `Hans`| <--> | Chinois (traditionnel) `Hant`|
| Chinois (traditionnel) | `zh-Hant` | Chinois (traditionnel) `Hant`| <--> | Latin `Latn` |
| Chinois (traditionnel) | `zh-Hant` | Chinois (traditionnel) `Hant`| <--> | Chinois (simplifié) `Hans` |
| Goudjrati | `gu`  | Goudjrati `Gujr` | <--> | Latin `Latn` |
| Hébreu | `he` | Hébreu `Hebr` | <--> | Latin `Latn` |
| Hindi | `hi` | Dévanâgarî `Deva` | <--> | Latin `Latn` |
| Japonais | `ja` | Japonais `Jpan` | <--> | Latin `Latn` |
| Kannada | `kn` | Kannada `Knda` | <--> | Latin `Latn` |
| Malayalam | `ml` | Malayalam `Mlym` | <--> | Latin `Latn` |
| Marathi | `mr` | Dévanâgarî `Deva` | <--> | Latin `Latn` |
| Odia | `or` | Odia `Orya` | <--> | Latin `Latn` |
| Pendjabi | `pa` | Gurmukhi `Guru`  | <--> | Latin `Latn`  |
| Serbe (cyrillique) | `sr-Cyrl` | Cyrillique `Cyrl`  | --> | Latin `Latn` |
| Serbe (latin) | `sr-Latn` | Latin `Latn` | --> | Cyrillique `Cyrl`|
| Tamoul | `ta` | Tamoul `Taml` | <--> | Latin `Latn` |
| Télougou | `te` | Télougou `Telu` | <--> | Latin `Latn` |
| Thaï | `th` | Thaï `Thai` | --> | Latin `Latn` |

### <a name="dictionary"></a>Dictionnaire

Le dictionnaire prend en charge les langues ci-après de ou vers l’anglais à l’aide des méthodes Lookup et Examples.

Consultez la documentation de référence pour les opérations [Dictionary Lookup](reference/v3-0-dictionary-lookup.md) et [Dictionary Examples](reference/v3-0-dictionary-examples.md).

| Langage    | Code langue |
|:----------- |:-------------:|
| Afrikaans      | `af`          |
| Arabe       | `ar`          |
| Bangla      | `bn`          |
| Bosniaque (latin)      | `bs`          |
| Bulgare      | `bg`          |
| Catalan      | `ca`          |
| Chinois (simplifié)      | `zh-Hans`          |
| Croate      | `hr`          |
| Tchèque      | `cs`          |
| Danois      | `da`          |
| Néerlandais      | `nl`          |
| Estonien      | `et`          |
| Finnois      | `fi`          |
| Français      | `fr`          |
| Allemand      | `de`          |
| Grec      | `el`          |
| Créole haïtien      | `ht`          |
| Hébreu      | `he`          |
| Hindi      | `hi`          |
| Hmong blanc      | `mww`          |
| Hongrois      | `hu`          |
| Islandais    | `is`  |
| Indonésien      | `id`          |
| Italien      | `it`          |
| Japonais      | `ja`          |
| Klingon      | `tlh`          |
| Coréen      | `ko`          |
| Letton      | `lv`          |
| Lituanien      | `lt`          |
| Malais      | `ms`          |
| Maltais      | `mt`          |
| Norvégien      | `nb`          |
| Persan      | `fa`          |
| Polonais      | `pl`          |
| Portugais (Brésil)     | `pt-br`          |
| Roumain      | `ro`          |
| Russe      | `ru`          |
| Serbe (latin)      | `sr-Latn`          |
| Slovaque     | `sk`          |
| Slovène      | `sl`          |
| Espagnol      | `es`          |
| Swahili      | `sw`          |
| Suédois      | `sv`          |
| Tamoul      | `ta`          |
| Thaï      | `th`          |
| Turc      | `tr`          |
| Ukrainien      | `uk`          |
| Ourdou      | `ur`          |
| Vietnamien      | `vi`          |
| Gallois      | `cy`          |

### <a name="access-the-translator-language-list-programmatically"></a>Accéder par programme à la liste de langues de Translator

Vous pouvez utiliser la méthode Languages pour récupérer la liste des langues prises en charge par Translator. Vous pouvez visualiser la liste par fonctionnalité, par code de langue, ainsi que par le nom de langue en anglais ou dans toute autre langue prise en charge. Cette liste est automatiquement mise à jour par le service Microsoft Translator lorsque de nouvelles langues deviennent disponibles.

[Visualiser la documentation de référence sur l’opération Languages](reference/v3-0-languages.md)

## <a name="customization"></a>Personnalisation

Les langues suivantes sont disponibles pour la personnalisation vers ou depuis l'anglais à l'aide de [Custom Translator](https://aka.ms/CustomTranslator).

| Langage    | Code langue |
|:----------- |:-------------:|
|Afrikaans| `af`|
| Arabe       | `ar`          |
| Bangla      | `bn`          |
| Bosniaque (latin)      | `bs`          |
| Bulgare      | `bg`          |
|Catalan|   `ca`    |
| Chinois (simplifié)      | `zh-Hans`          |
|Chinois traditionnel|   `zh-Hant`   |
| Croate      | `hr`          |
| Tchèque      | `cs`          |
| Danois      | `da`          |
| Néerlandais      | `nl`          |
| Anglais    | `en`     |
| Estonien      | `et`          |
|Fidjien|    `fj`    |
|Filipino|  `fil`   |
| Finnois      | `fi`          |
| Français      | `fr`          |
| Allemand      | `de`          |
| Grec      | `el`          |
| Goudjrati| `gu`    |
| Hébreu      | `he`          |
| Hindi      | `hi`          |
| Hongrois      | `hu`          |
| Islandais | `is` |
| Indonésien|   `id`    |
| Irlandais | `ga`  |
| Italien      | `it`          |
| Japonais      | `ja`          |
|Kannada|`kn`|
| Coréen      | `ko`          |
| Letton      | `lv`          |
| Lituanien      | `lt`          |
| Malgache| `mg`    |
| Malais|    `ms` |
|Maltais|   `mt`    |
| Maori| `mi`  |
| Marathi| `mr`  |
| Norvégien      | `nb`          |
| Persan      | `fa`          |
| Polonais      | `pl`          |
| Portugais (Brésil) | `pt-br` |
| Pendjabi|`pa`|
| Roumain      | `ro`          |
| Russe      | `ru`          |
| Samoan|   `sm`    |
| Serbe (latin)      | `sr-Latn`          |
| Slovaque     | `sk`          |
| Slovène      | `sl`          |
| Espagnol      | `es`          |
| Swahili|  `sw`    |
| Suédois      | `sv`          |
|Tahitien|  `ty`    |
| Thaï      | `th`          |
|Tonga|    `to`    |
| Turc      | `tr`          |
| Ukrainien      | `uk`          |
| Ourdou| `ur`    |
| Vietnamien      | `vi`          |
| Gallois | `cy` |

## <a name="speech-translation"></a>Traduction vocale
La traduction vocale est disponible à l’aide de Translator avec le service Speech de Cognitive Services. Consultez la [documentation du service Speech](../speech-service/index.yml) pour en savoir plus sur l’utilisation de la traduction vocale et pour afficher toutes les [options de langues disponibles](../speech-service/language-support.md).

### <a name="speech-to-text"></a>Reconnaissance vocale
Convertissez la parole en texte afin d’effectuer la traduction dans la langue textuelle de votre choix. La reconnaissance vocale est utilisée pour la traduction de la voix en texte, ou pour traduire la voix en voix lorsqu’elle est utilisée conjointement avec la synthèse vocale.

| Langage    |
|:----------- |
|Arabe|
|Cantonais (traditionnel)|
|Catalan|
|Chinois (simplifié)|
|Chinois traditionnel|
|Danois|
|Néerlandais|
|Anglais|
|Finnois|
|Français|
|Français (Canada)|
|Allemand|
|Goudjrati|
|Hindi|
|Italien|
|Japonais|
|Coréen|
|Marathi|
|Norvégien|
|Polonais|
|Portugais (Brésil)|
|Portugais (Portugal)|
|Russe|
|Espagnol|
|Suédois|
|Tamoul|
|Télougou|
|Thaï|
|Turc|

### <a name="text-to-speech"></a>Synthèse vocale
Convertir du texte en parole. La synthèse vocale est utilisée pour ajouter une sortie audible des résultats de la traduction, ou pour traduire la voix en voix lorsqu’elle est utilisée avec la reconnaissance vocale. 

| Langage |
|:-|
| Arabe |
| Bulgare |
| Cantonais (traditionnel) |
| Catalan |
| Chinois (simplifié) |
| Chinois traditionnel |
| Croate |
| Tchèque |
| Danois |
| Néerlandais |
| Anglais |
| Finnois |
| Français |
| Français (Canada) |
| Allemand |
| Grec |
| Hébreu |
| Hindi |
| Hongrois |
| Indonésien |
| Italien |
| Japonais |
| Coréen |
| Malais |
| Norvégien |
| Polonais |
| Portugais (Brésil) |
| Portugais (Portugal) |
| Roumain |
| Russe |
| Slovaque |
| Slovène |
| Espagnol |
| Suédois |
| Tamoul |
| Télougou |
| Thaï |
| Turc |
| Vietnamien |

## <a name="view-the-language-list-on-the-microsoft-translator-website"></a>Afficher la liste des langues sur le site web de Microsoft Translator

Si vous souhaitez examiner rapidement les langues, le site web de Microsoft Translator présente toutes les langues prises en charge par Translator pour la traduction de texte et par le service Speech pour la traduction vocale. Cette liste ne comporte pas d’informations propres aux développeurs, comme les codes de langue.

[Voir la liste des langues](https://www.microsoft.com/translator/languages.aspx)