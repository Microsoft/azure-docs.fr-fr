---
title: Prise en charge linguistique - API Analyse de texte
titleSuffix: Azure Cognitive Services
description: Liste des langages naturels pris en charge par l’API Analyse de texte. Cet article décrit les langues prises en charge pour chaque opération.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 11/19/2020
ms.author: aahi
ms.openlocfilehash: b90fe6b7e0937a9ac3b300e904085122610d5645
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94957076"
---
# <a name="text-analytics-api-v3-language-support"></a>Prise en charge linguistique de l’API Analyse de texte v3 

[!INCLUDE [v3 region availability](includes/v3-region-availability.md)]


#### <a name="sentiment-analysis"></a>[Analyse des sentiments](#tab/sentiment-analysis)

| Langage              | Code langue | Prise en charge de v2 | Prise en charge de v3 | À partir de la version de modèle v3 : |              Notes |
|:----------------------|:-------------:|:----------:|:----------:|:--------------------------:|-------------------:|
| Chinois simplifié    |   `zh-hans`   |     ✓      |     ✓      |         2019-10-01         | `zh` également accepté |
| Chinois traditionnel   |   `zh-hant`   |            |     ✓      |         2019-10-01         |                    |
| Danois               |     `da`      |     ✓      |            |                            |                    |
| Néerlandais                 |     `nl`      |     ✓      |            |                            |                    |
| Anglais               |     `en`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Finnois               |     `fi`      |     ✓      |            |                            |                    |
| Français                |     `fr`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Allemand                |     `de`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Grec                 |     `el`      |     ✓      |            |                            |                    |
| Hindi                 |     `hi`      |            |      ✓     |          2020-04-01        |                    |
| Italien               |     `it`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Japonais              |     `ja`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Coréen                |     `ko`      |            |     ✓      |         2019-10-01         |                    |
| Norvégien (Bokmål)   |     `no`      |     ✓      |     ✓      |         2020-07-01         |                    |
| Polonais                |     `pl`      |     ✓      |            |                            |                    |
| Portugais (Brésil)   |    `pt-BR`    |            |     ✓      |         2020-04-01         |                    |
| Portugais (Portugal) |    `pt-PT`    |     ✓      |     ✓      |         2019-10-01         | `pt` également accepté |
| Russe               |     `ru`      |     ✓      |            |                            |                    |
| Espagnol               |     `es`      |     ✓      |     ✓      |         2019-10-01         |                    |
| Suédois               |     `sv`      |     ✓      |            |                            |                    |
| Turc               |     `tr`      |     ✓      |     ✓       |         2020-07-01        |                    |

### <a name="opinion-mining-v31-preview-only"></a>Exploration des opinions (v3.1-preview uniquement)

| Langage              | Code langue | À partir de la version de modèle v3 : |              Notes |
|:----------------------|:-------------:|:------------------------------------:|-------------------:|
| Anglais               |     `en`      |              2020-04-01              |                    |


#### <a name="named-entity-recognition-ner"></a>[Reconnaissance d’entité nommée (NER)](#tab/named-entity-recognition)

> [!NOTE]
> * NER v3 prend actuellement en charge seulement la langue anglaise et la langue espagnole. Si vous appelez NER v3 avec une autre langue, l’API retourne les résultats de la version 2.1 à condition que la langue soit prise en charge dans v2.1.
> * v2.1 retourne l’ensemble complet des entités disponibles uniquement en anglais, chinois simplifié, français, allemand et espagnol.  Les entités « Person », « Location » et « Organization » sont retournées pour les autres langues prises en charge.

| Langage               | Code langue | Prise en charge de v2.1 | Prise en charge de v3 | À partir de la version de modèle v3 : |       Notes        |
|:-----------------------|:-------------:|:----------:|:----------:|:-------------------------------:|:------------------:|
| Arabe                |     `ar`      |     ✓      |            |                                 |                    |
| Tchèque                 |     `cs`      |     ✓      |            |                                 |                    |
| Chinois simplifié     |   `zh-hans`   |     ✓      |            |                                 | `zh` également accepté |
| Chinois traditionnel   |   `zh-hant`   |     ✓      |            |                                 |                    |
| Danois                |     `da`      |     ✓      |            |                                 |                    |
| Néerlandais                 |     `nl`      |     ✓      |            |                                 |                    |
| Anglais                |     `en`      |     ✓      |     ✓      |           2019-10-01            |                    |
| Finnois               |     `fi`      |     ✓      |            |                                 |                    |
| Français                 |     `fr`      |     ✓      |            |                                 |                    |
| Allemand                 |     `de`      |     ✓      |            |                                 |                    |
| Hébreu                |     `he`      |     ✓      |            |                                 |                    |
| Hongrois             |     `hu`      |     ✓      |            |                                 |                    |
| Italien               |     `it`      |     ✓      |            |                                 |                    |
| Japonais              |     `ja`      |     ✓      |            |                                 |                    |
| Coréen                |     `ko`      |     ✓      |            |                                 |                    |
| Norvégien (Bokmål)   |     `no`      |     ✓      |            |                                 | `nb` également accepté |
| Polonais                |     `pl`      |     ✓      |            |                                 |                    |
| Portugais (Portugal) |    `pt-PT`    |     ✓      |            |                                 | `pt` également accepté |
| Portugais (Brésil)   |    `pt-BR`    |     ✓      |            |                                 |                    |
| Russe              |     `ru`      |     ✓      |            |                                 |                    |
| Espagnol               |     `es`      |     ✓      |     ✓       |              2020-04-01                   |                    |
| Suédois               |     `sv`      |     ✓      |            |                                 |                    |
| Turc               |     `tr`      |     ✓      |            |                                 |                    |

#### <a name="key-phrase-extraction"></a>[Extraction de phrases clés](#tab/key-phrase-extraction)

> [!NOTE]
> Les versions de modèle d’extraction de phrases clés antérieures à 2020-07-01 ont une limite de 64 caractères. Cette limite n’est pas présente dans les versions de modèle ultérieures.

| Langage              | Code langue | Prise en charge de v2 | Prise en charge de v3 | Disponible à partir de la version de modèle v3 : |       Notes        |
|:----------------------|:-------------:|:----------:|:----------:|:-----------------------------------------:|:------------------:|
| Néerlandais                 |     `nl`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Anglais               |     `en`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Finnois               |     `fi`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Français                |     `fr`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Allemand                |     `de`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Italien               |     `it`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Japonais              |     `ja`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Coréen                |     `ko`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Norvégien (Bokmål)   |     `no`      |     ✓      |     ✓      |                2019-10-01                 | `nb` également accepté |
| Polonais                |     `pl`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Portugais (Portugal) |    `pt-PT`    |     ✓      |     ✓      |                2019-10-01                 | `pt` également accepté |
| Portugais (Brésil)   |    `pt-BR`    |     ✓      |     ✓      |                2019-10-01                 |                    |
| Russe               |     `ru`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Espagnol               |     `es`      |     ✓      |     ✓      |                2019-10-01                 |                    |
| Suédois               |     `sv`      |     ✓      |     ✓      |                2019-10-01                 |                    |

#### <a name="entity-linking"></a>[Liaison d’entités](#tab/entity-linking)

| Langage | Code langue | Prise en charge de v2 | Prise en charge de v3 | Disponible à partir de la version de modèle v3 : | Notes |
|:---------|:-------------:|:----------:|:----------:|:-----------------------------------------:|:-----:|
| Anglais  |     `en`      |     ✓      |     ✓      |                2019-10-01                 |       |
| Espagnol  |     `es`      |     ✓      |     ✓      |                2019-10-01                 |       |

#### <a name="language-detection"></a>[Détection de la langue](#tab/language-detection)

L’API Analyse de texte peut détecter une grande variété de langues, de variantes, de dialectes ainsi que certaines langues régionales/culturelles et peut retourner les langues détectées avec leur nom et leur code. Les paramètres du code de langue de la fonctionnalité Détection de langue de cette API sont conformes à la norme [BCP-47](https://tools.ietf.org/html/bcp47), avec la plupart d’entre eux conformes aux identificateurs [ISO-639-1](https://www.iso.org/iso-639-language-codes.html). 

Si vous avez du contenu exprimé dans une langue moins fréquemment utilisée, vous pouvez essayer Détection de langue pour voir si elle retourne un code. La réponse pour les langues qui ne peuvent pas être détectées est `unknown`.

| Langage | Code de langue |  Prise en charge de v3 | Disponible à partir de la version de modèle v3 : |
|:---------|:-------------:|:----------:|:-----------------------------------------:|
|Afrikaans|`af`|✓|    |
|Albanais|`sq`|✓|    |
|Arabe|`ar`|✓|    |
|Arménien|`hy`|✓|    |
|Basque|`eu`|✓|    |
|Biélorusse|`be`|✓|    |
|Bengali|`bn`|✓|    |
|Bosniaque|`bs`|✓|2020-09-01|
|Bulgare|`bg`|✓|    |
|Birman|`my`|✓|    |
|Catalan, valencien|`ca`|✓|    |
|Khmer central|`km`|✓|    |
|Chinois|`zh`|✓|    |
|Chinois (simplifié)|`zh_chs`|✓|    |
|Chinois traditionnel|`zh_cht`|✓|    |
|Croate|`hr`|✓|    |
|Tchèque|`cs`|✓|    |
|Danois|`da`|✓|    |
|Dari|`prs`|✓|2020-09-01|
|Divehi, Dhivehi, maldivien|`dv`|✓|    |
|Néerlandais, flamand|`nl`|✓|    |
|Anglais|`en`|✓|    |
|Espéranto|`eo`|✓|    |
|Estonien|`et`|✓|    |
|Fidjien|`fj`|✓|2020-09-01|
|Finnois|`fi`|✓|    |
|Français|`fr`|✓|    |
|Galicien|`gl`|✓|    |
|Géorgien|`ka`|✓|    |
|Allemand|`de`|✓|    |
|Grec|`el`|✓|    |
|Goudjrati|`gu`|✓|    |
|Haïtien, créole haïtien|`ht`|✓|    |
|Hébreu|`he`|✓|    |
|Hindi|`hi`|✓|    |
|Hmong blanc|`mww`|✓|2020-09-01|
|Hongrois|`hu`|✓|    |
|Islandais|`is`|✓|    |
|Indonésien|`id`|✓|    |
|Inuktitut|`iu`|✓|    |
|Irlandais|`ga`|✓|    |
|Italien|`it`|✓|    |
|Japonais|`ja`|✓|    |
|Kannada|`kn`|✓|    |
|Kazakh|`kk`|✓|2020-09-01|
|Coréen|`ko`|✓|    |
|Kurde|`ku`|✓|    |
|Lao|`lo`|✓|    |
|Latin|`la`|✓|    |
|Letton|`lv`|✓|    |
|Lituanien|`lt`|✓|    |
|Macédonien|`mk`|✓|    |
|Malgache|`mg`|✓|2020-09-01|
|Malais|`ms`|✓|    |
|Malayalam|`ml`|✓|    |
|Maltais|`mt`|✓|    |
|Maori|`mi`|✓|2020-09-01|
|Marathi|`mr`|✓|2020-09-01|
|Norvégien|`no`|✓|    |
|Norvégien (Nynorsk)|`nn`|✓|    |
|Odia|`or`|✓|    |
|Pachto, pachtou|`ps`|✓|    |
|Persan|`fa`|✓|    |
|Polonais|`pl`|✓|    |
|Portugais|`pt`|✓|    |
|Pendjabi, penjabi|`pa`|✓|    |
|Queretaro Otomi|`otq`|✓|2020-09-01|
|Roumain, moldave|`ro`|✓|    |
|Russe|`ru`|✓|    |
|Samoan|`sm`|✓|2020-09-01|
|Serbe|`sr`|✓|    |
|Singhalais, cingalais|`si`|✓|    |
|Slovaque|`sk`|✓|    |
|Slovène|`sl`|✓|    |
|Somali|`so`|✓|    |
|Espagnol, castillan|`es`|✓|    |
|Swahili|`sw`|✓|    |
|Suédois|`sv`|✓|    |
|Tagalog|`tl`|✓|    |
|Tahitien|`ty`|✓|2020-09-01|
|Tamoul|`ta`|✓|    |
|Télougou|`te`|✓|    |
|Thaï|`th`|✓|    |
|Tonga|`to`|✓|2020-09-01|
|Turc|`tr`|✓|    |
|Ukrainien|`uk`|✓|    |
|Ourdou|`ur`|✓|    |
|Ouzbek|`uz`|✓|    |
|Vietnamien|`vi`|✓|    |
|Gallois|`cy`|✓|    |
|Yiddish|`yi`|✓|    |
|Yucatec Maya|`yua`|✓|    |


---


## <a name="see-also"></a>Voir aussi

* [Qu’est-ce que l’API Analyse de texte ?](overview.md)   
