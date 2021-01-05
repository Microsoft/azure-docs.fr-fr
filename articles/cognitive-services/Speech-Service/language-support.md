---
title: Prise en charge des langues – Service Speech
titleSuffix: Azure Cognitive Services
description: Le service Speech prend en charge de nombreuses langues, que ce soit pour la reconnaissance vocale, la synthèse vocale ou la traduction vocale. Cet article fournit une liste complète des langues prise en charge, par fonctionnalité de service.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/26/2020
ms.author: trbye
ms.custom: references_regions
ms.openlocfilehash: 12c9777449d3f360b8f08d99aca7b916856139d4
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400180"
---
# <a name="language-and-voice-support-for-the-speech-service"></a>Prise en charge des langues et de la voix pour le service Speech

La prise en charge des langues varie selon les fonctionnalités du service Speech. Les tableaux suivants résument la prise en charge des langues pour les offres de service [Reconnaissance vocale](#speech-to-text), [Synthèse vocale](#text-to-speech) et [Traduction vocale](#speech-translation).

## <a name="speech-to-text"></a>Reconnaissance vocale

Le Kit de développement logiciel (SDK) et l’API REST Speech de Microsoft prennent en charge les langages suivants (paramètres régionaux). 

Afin d'améliorer la précision, la personnalisation est proposée pour une partie des langues en chargeant des **transcriptions audio + étiquetées à la main** ou des **textes associés : Phrases.** Pour en savoir plus sur la personnalisation, consultez [Bien démarrer avec Custom Speech](./custom-speech-overview.md).

<!--
To get the AM and ML bits:
https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetSupportedLocalesForModels

To get pronunciation bits:
https://cris.ai -> Click on Adaptation Data -> scroll down to section "Pronunciation Datasets" -> Click on Import -> Locale: the list of locales there correspond to the supported locales
-->

| Langage                          | Paramètres régionaux (BCP-47) | Personnalisations                                   |
|-----------------------------------|--------|--------------------------------------------------|
|Arabe (Bahreïn), standard moderne  |`ar-BH` | Modèle de langage                                   |
|Arabe (Égypte)                     |`ar-EG` | Modèle de langage                                   |
|Arabe (Irak)                      |`ar-IQ` | Modèle de langage                                   |
|Arabe (Israël)                              |`ar-IL` | Modèle de langage                                                   |
|Arabe (Jordanie)                    |`ar-JO` | Modèle de langage                                   |
|Arabe (Koweït)                    |`ar-KW` | Modèle de langage                                   |
|Arabe (Liban)                   |`ar-LB` | Modèle de langage                                   |
|Arabe (Oman)                      |`ar-OM` | Modèle de langage                                   |
|Arabe (Qatar)                     |`ar-QA` | Modèle de langage                                   |
|Arabe (Arabie saoudite)              |`ar-SA` | Modèle de langage                                   |
|Arabe (État de Palestine)            |`ar-PS` | Modèle de langage                                   |
|Arabe (Syrie)                     |`ar-SY` | Modèle de langage                                   |
|Arabe (Émirats arabes unis)      |`ar-AE` | Modèle de langage                                   |
|Bulgare (Bulgarie)               |`bg-BG` | Modèle de langage                                   |
|Catalan (Espagne)                    |`ca-ES` | Modèle de langage                                   |
|Chinois (cantonais, traditionnel)   |`zh-HK` | Modèle de langage                                   |
|Chinois (mandarin, simplifié)     |`zh-CN` | Modèle acoustique<br>Modèle de langage                 |
|Chinois (mandarin, taïwanais)       |`zh-TW` | Modèle de langage                                   |
|Croate (Croatie)                 |`hr-HR` | Modèle de langage                                   |
|Tchèque (République tchèque)             |`cs-CZ` | Modèle de langage                                   |
|Danois (Danemark)                   |`da-DK` | Modèle de langage                                   |
|Néerlandais (Pays-Bas)                |`nl-NL` | Modèle de langage                                   |
|Anglais (Australie)                |`en-AU` | Modèle acoustique<br>Modèle de langage                 |
|Anglais (Canada)                   |`en-CA` | Modèle acoustique<br>Modèle de langage                 |
|Anglais (Hong Kong)                |`en-HK` | Modèle de langage                                   |
|Anglais (Inde)                    |`en-IN` | Modèle acoustique<br>Modèle de langage                 |
|Anglais (Irlande)                  |`en-IE` | Modèle de langage                                   |
|Anglais (Nouvelle-Zélande)              |`en-NZ` | Modèle acoustique<br>Modèle de langage                 |
|Anglais (Nigeria)                          |`en-NG` | Modèle de langage                                                     |
|Anglais (Philippines)              |`en-PH` | Modèle de langage                                   |
|Anglais (Singapour)                |`en-SG` | Modèle de langage                                   |
|Anglais (Afrique du Sud)             |`en-ZA` | Modèle de langage                                   |
|Anglais (Royaume-Uni)           |`en-GB` | Modèle acoustique<br>Modèle de langage<br>Prononcer|
|Anglais (États-Unis)            |`en-US` | Modèle acoustique<br>Modèle de langage<br>Prononcer|
|Estonien (Estonie)                  |`et-EE` | Modèle de langage                                   |
|Finnois (Finlande)                  |`fi-FI` | Modèle de langage                                   |
|Français (Canada)                    |`fr-CA` | Modèle acoustique<br>Modèle de langage                 |
|Français (France)                    |`fr-FR` | Modèle acoustique<br>Modèle de langage<br>Prononcer|
|Allemand (Allemagne)                   |`de-DE` | Modèle acoustique<br>Modèle de langage<br>Prononcer|
|Grec (Grèce)                     |`el-GR` | Modèle de langage                                   |
|Goudjrati (Inde)                  |`gu-IN` | Modèle de langage                                   |
|Hindi (Inde)                      |`hi-IN` | Modèle acoustique<br>Modèle de langage                 |
|Hongrois (Hongrie)                |`hu-HU` | Modèle de langage                                   |
|Irlandais (Irlande)                     |`ga-IE` | Modèle de langage                                   |
|Italien (Italie)                    |`it-IT` | Modèle acoustique<br>Modèle de langage<br>Prononcer|
|Japonais (Japon)                   |`ja-JP` | Modèle de langage                                   |
|Coréen (Corée)                     |`ko-KR` | Modèle de langage                                   |
|Letton (Lettonie)                   |`lv-LV` | Modèle de langage                                   |
|Lituanien (Lituanie)             |`lt-LT` | Modèle de langage                                   |
|Maltais (Malte)                     |`mt-MT` | Modèle de langage                                   |
|Marathi (Inde)                    |`mr-IN` | Modèle de langage                                   |
|Norvégien (bokmål, Norvège)         |`nb-NO` | Modèle de langage                                   |
|Polonais (Pologne)                    |`pl-PL` | Modèle de langage                                   |
|Portugais (Brésil)                |`pt-BR` | Modèle acoustique<br>Modèle de langage<br>Prononcer|
|Portugais (Portugal)              |`pt-PT` | Modèle de langage                                   |
|Roumain (Roumanie)                 |`ro-RO` | Modèle de langage                                   |
|Russe (Russie)                   |`ru-RU` | Modèle acoustique<br>Modèle de langage                 |
|Slovaque (Slovaquie)                  |`sk-SK` | Modèle de langage                                   |
|Slovène (Slovénie)               |`sl-SI` | Modèle de langage                                   |
|Espagnol (Argentine)                |`es-AR` | Modèle de langage                                   |
|Espagnol (Bolivie)                  |`es-BO` | Modèle de langage                                   |
|Espagnol (Chili)                    |`es-CL` | Modèle de langage                                   |
|Espagnol (Colombie)                 |`es-CO` | Modèle de langage                                   |
|Espagnol (Costa Rica)               |`es-CR` | Modèle de langage                                   |
|Espagnol (Cuba)                     |`es-CU` | Modèle de langage                                   |
|Espagnol (République dominicaine)       |`es-DO` | Modèle de langage                                   |
|Espagnol (Équateur)                  |`es-EC` | Modèle de langage                                   |
|Espagnol (Salvador)              |`es-SV` | Modèle de langage                                   |
|Espagnol (Guinée équatoriale)            |`es-GQ` | Modèle de langage                                   |
|Espagnol (Guatemala)                |`es-GT` | Modèle de langage                                   |
|Espagnol (Honduras)                 |`es-HN` | Modèle de langage                                   |
|Espagnol (Mexique)                   |`es-MX` | Modèle acoustique<br>Modèle de langage                 |
|Espagnol (Nicaragua)                |`es-NI` | Modèle de langage                                   |
|Espagnol (Panama)                   |`es-PA` | Modèle de langage                                   |
|Espagnol (Paraguay)                 |`es-PY` | Modèle de langage                                   |
|Espagnol (Pérou)                     |`es-PE` | Modèle de langage                                   |
|Espagnol (Porto Rico)              |`es-PR` | Modèle de langage                                   |
|Espagnol (Espagne)                    |`es-ES` | Modèle acoustique<br>Modèle de langage                 |
|Espagnol (Uruguay)                  |`es-UY` | Modèle de langage                                   |
|Espagnol (États-Unis)                      |`es-US` | Modèle de langage                                   |
|Espagnol (Venezuela)                |`es-VE` | Modèle de langage                                   |
|Suédois (Suède)                   |`sv-SE` | Modèle de langage                                   |
|Tamoul (Inde)                      |`ta-IN` | Modèle de langage                                   |
|Télougou (Inde)                     |`te-IN` | Modèle de langage                                   |
|Thaï (Thaïlande)                    |`th-TH` | Modèle de langage                                   |
|Turc (Turquie)                   |`tr-TR` | Modèle de langage                                   |

## <a name="text-to-speech"></a>Synthèse vocale

Le Kit de développement logiciel (SDK) Speech de Microsoft et les API REST prennent en charge ces voix qui, à leur tour, prennent chacune en charge une langue et un dialecte spécifiques, identifiés par les paramètres régionaux. Vous pouvez également obtenir la liste complète des langues et des voix prises en charge pour chaque région/point de terminaison spécifique via l’API [voices/list](rest-text-to-speech.md#get-a-list-of-voices). 

> [!IMPORTANT]
> Les prix varient pour les voix standard, personnalisées et neurales. Pour plus d’informations, visitez la page [Tarification](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

### <a name="neural-voices"></a>Voix neurales

La synthèse vocale neuronale est un nouveau type de synthèse vocale alimentée par les réseaux neuronaux profonds. Lorsque vous utilisez une voix neuronale, il est presque impossible de distinguer la voix synthétisée des enregistrements humains.

Les voix neurales peuvent être utilisées pour rendre les interactions avec les chatbots et des assistants vocaux plus naturelles et agréables, convertir des textes numériques comme les livres électroniques en livres audio et améliorer les systèmes de navigation embarqués. Grâce à la prosodie naturelle quasi humaine et à la bonne articulation des mots, les voix neuronales réduisent considérablement la fatigue d’écoute des utilisateurs qui interagissent avec les systèmes d’intelligence artificielle.

| Langue | Paramètres régionaux | Sexe | Nom de la voix | Prise en charge du style |
|---|---|---|---|---|
| Arabe (Égypte) | `ar-EG` | Female | `ar-EG-SalmaNeural` | Général |
| Arabe (Égypte) | `ar-EG` | Male | `ar-EG-ShakirNeural` <sup>Nouveau</sup> | Général |
| Arabe (Arabie saoudite) | `ar-SA` | Female | `ar-SA-ZariyahNeural` | Général |
| Arabe (Arabie saoudite) | `ar-SA` | Male | `ar-SA-HamedNeural` <sup>Nouveau</sup> | Général |
| Bulgare (Bulgarie) | `bg-BG` | Female | `bg-BG-KalinaNeural` | Général |
| Bulgare (Bulgarie) | `bg-BG` | Male | `bg-BG-BorislavNeural` <sup>Nouveau</sup> | Général |
| Catalan (Espagne) | `ca-ES` | Female | `ca-ES-AlbaNeural` | Général |
| Catalan (Espagne) | `ca-ES` | Female | `ca-ES-JoanaNeural` <sup>Nouveau</sup> | Général |
| Catalan (Espagne) | `ca-ES` | Male | `ca-ES-EnricNeural` <sup>Nouveau</sup> | Général |
| Chinois (cantonais, traditionnel) | `zh-HK` | Female | `zh-HK-HiuGaaiNeural` | Général |
| Chinois (cantonais, traditionnel) | `zh-HK` | Female | `zh-HK-HiuMaanNeural` <sup>Nouveau</sup> | Général |
| Chinois (cantonais, traditionnel) | `zh-HK` | Male | `zh-HK-WanLungNeural` <sup>Nouveau</sup> | Général |
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-XiaoxiaoNeural` | Général, plusieurs styles de voix disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles)  |
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-XiaoyouNeural` | Voix d’enfant, optimisée pour raconter des histoires |
| Chinois (mandarin, simplifié) | `zh-CN` | Male | `zh-CN-YunyangNeural` | Optimisée pour la lecture de bulletins d’informations,<br /> plusieurs styles de voix disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chinois (mandarin, simplifié) | `zh-CN` | Male | `zh-CN-YunyeNeural` | Optimisée pour raconter des histoires  |
| Chinois (mandarin, taïwanais) | `zh-TW` | Female | `zh-TW-HsiaoChenNeural` <sup>Nouveau</sup> | Général |
| Chinois (mandarin, taïwanais) | `zh-TW` | Female | `zh-TW-HsiaoYuNeural` | Général |
| Chinois (mandarin, taïwanais) | `zh-TW` | Male | `zh-TW-YunJheNeural` <sup>Nouveau</sup> | Général |
| Croate (Croatie) | `hr-HR` | Female | `hr-HR-GabrijelaNeural` | Général |
| Croate (Croatie) | `hr-HR` | Male | `hr-HR-SreckoNeural` <sup>Nouveau</sup> | Général |
| Tchèque (Tchéquie) | `cs-CZ` | Female | `cs-CZ-VlastaNeural` | Général |
| Tchèque (Tchéquie) | `cs-CZ` | Male | `cs-CZ-AntoninNeural` <sup>Nouveau</sup> | Général |
| Danois (Danemark) | `da-DK` | Female | `da-DK-ChristelNeural` | Général |
| Danois (Danemark) | `da-DK` | Male | `da-DK-JeppeNeural` <sup>Nouveau</sup> | Général |
| Néerlandais (Pays-Bas) | `nl-NL` | Female | `nl-NL-ColetteNeural` | Général |
| Néerlandais (Pays-Bas) | `nl-NL` | Female | `nl-NL-FennaNeural` <sup>Nouveau</sup> | Général |
| Néerlandais (Pays-Bas) | `nl-NL` | Male | `nl-NL-MaartenNeural` <sup>Nouveau</sup> | Général |
| Anglais (Australie) | `en-AU` | Female | `en-AU-NatashaNeural` | Général |
| Anglais (Australie) | `en-AU` | Male | `en-AU-WilliamNeural` | Général |
| Anglais (Canada) | `en-CA` | Female | `en-CA-ClaraNeural` | Général |
| Anglais (Canada) | `en-CA` | Male | `en-CA-LiamNeural` <sup>Nouveau</sup> | Général |
| Anglais (Inde) | `en-IN` | Female | `en-IN-NeerjaNeural` | Général |
| Anglais (Inde) | `en-IN` | Male | `en-IN-PrabhatNeural` <sup>Nouveau</sup> | Général |
| Anglais (Irlande) | `en-IE` | Female | `en-IE-EmilyNeural` | Général |
| Anglais (Irlande) | `en-IE` | Male | `en-IE-ConnorNeural` <sup>Nouveau</sup> | Général |
| Anglais (Royaume-Uni) | `en-GB` | Female | `en-GB-LibbyNeural` | Général |
| Anglais (Royaume-Uni) | `en-GB` | Female | `en-GB-MiaNeural` | Général |
| Anglais (Royaume-Uni) | `en-GB` | Male | `en-GB-RyanNeural` | Général |
| Anglais (États-Unis) | `en-US` | Female | `en-US-AriaNeural` | Général, plusieurs styles de voix disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles)  |
| Anglais (États-Unis) | `en-US` | Female | `en-US-JennyNeural` | Général |
| Anglais (États-Unis) | `en-US` | Male | `en-US-GuyNeural` | Général |
| Finnois (Finlande) | `fi-FI` | Female | `fi-FI-NooraNeural` | Général |
| Finnois (Finlande) | `fi-FI` | Female | `fi-FI-SelmaNeural` <sup>Nouveau</sup> | Général |
| Finnois (Finlande) | `fi-FI` | Male | `fi-FI-HarriNeural` <sup>Nouveau</sup> | Général |
| Français (Canada) | `fr-CA` | Female | `fr-CA-SylvieNeural` | Général |
| Français (Canada) | `fr-CA` | Male | `fr-CA-JeanNeural` | Général |
| Français (France) | `fr-FR` | Female | `fr-FR-DeniseNeural` | Général |
| Français (France) | `fr-FR` | Male | `fr-FR-HenriNeural` | Général |
| Français (Suisse) | `fr-CH` | Female | `fr-CH-ArianeNeural` | Général |
| Français (Suisse) | `fr-CH` | Male | `fr-CH-FabriceNeural` <sup>Nouveau</sup> | Général |
| Allemand (Autriche) | `de-AT` | Female | `de-AT-IngridNeural` | Général |
| Allemand (Autriche) | `de-AT` | Male | `de-AT-JonasNeural` <sup>Nouveau</sup> | Général |
| Allemand (Allemagne) | `de-DE` | Female | `de-DE-KatjaNeural` | Général |
| Allemand (Allemagne) | `de-DE` | Male | `de-DE-ConradNeural` | Général |
| Allemand (Suisse) | `de-CH` | Female | `de-CH-LeniNeural` | Général |
| Allemand (Suisse) | `de-CH` | Male | `de-CH-JanNeural` <sup>Nouveau</sup> | Général |
| Grec (Grèce) | `el-GR` | Female | `el-GR-AthinaNeural` | Général |
| Grec (Grèce) | `el-GR` | Male | `el-GR-NestorasNeural` <sup>Nouveau</sup> | Général |
| Hébreu (Israël) | `he-IL` | Female | `he-IL-HilaNeural` | Général |
| Hébreu (Israël) | `he-IL` | Male | `he-IL-AvriNeural` <sup>Nouveau</sup> | Général |
| Hindi (Inde) | `hi-IN` | Female | `hi-IN-SwaraNeural` | Général |
| Hindi (Inde) | `hi-IN` | Male | `hi-IN-MadhurNeural` <sup>Nouveau</sup> | Général |
| Hongrois (Hongrie) | `hu-HU` | Female | `hu-HU-NoemiNeural` | Général |
| Hongrois (Hongrie) | `hu-HU` | Male | `hu-HU-TamasNeural` <sup>Nouveau</sup> | Général |
| Indonésien (Indonésie) | `id-ID` | Female | `id-ID-GadisNeural` <sup>Nouveau</sup> | Général |
| Indonésien (Indonésie) | `id-ID` | Male | `id-ID-ArdiNeural` | Général |
| Italien (Italie) | `it-IT` | Female | `it-IT-ElsaNeural` | Général |
| Italien (Italie) | `it-IT` | Female | `it-IT-IsabellaNeural` | Général |
| Italien (Italie) | `it-IT` | Male | `it-IT-DiegoNeural` | Général |
| Japonais (Japon) | `ja-JP` | Female | `ja-JP-NanamiNeural` | Général |
| Japonais (Japon) | `ja-JP` | Male | `ja-JP-KeitaNeural` | Général |
| Coréen (Corée) | `ko-KR` | Female | `ko-KR-SunHiNeural` | Général |
| Coréen (Corée) | `ko-KR` | Male | `ko-KR-InJoonNeural` | Général |
| Malais (Malaisie) | `ms-MY` | Female | `ms-MY-YasminNeural` | Général |
| Malais (Malaisie) | `ms-MY` | Male | `ms-MY-OsmanNeural` <sup>Nouveau</sup> | Général |
| Norvégien (bokmål, Norvège) | `nb-NO` | Female | `nb-NO-IselinNeural` | Général |
| Norvégien (bokmål, Norvège) | `nb-NO` | Female | `nb-NO-PernilleNeural` <sup>Nouveau</sup> | Général |
| Norvégien (bokmål, Norvège) | `nb-NO` | Male | `nb-NO-FinnNeural` <sup>Nouveau</sup> | Général |
| Polonais (Pologne) | `pl-PL` | Female | `pl-PL-AgnieszkaNeural` <sup>Nouveau</sup> | Général |
| Polonais (Pologne) | `pl-PL` | Female | `pl-PL-ZofiaNeural` | Général |
| Polonais (Pologne) | `pl-PL` | Male | `pl-PL-MarekNeural` <sup>Nouveau</sup> | Général |
| Portugais (Brésil) | `pt-BR` | Female | `pt-BR-FranciscaNeural` | Général, plusieurs styles de voix disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles)  |
| Portugais (Brésil) | `pt-BR` | Male | `pt-BR-AntonioNeural` | Général |
| Portugais (Portugal) | `pt-PT` | Female | `pt-PT-FernandaNeural` | Général |
| Portugais (Portugal) | `pt-PT` | Female | `pt-PT-RaquelNeural` <sup>Nouveau</sup> | Général |
| Portugais (Portugal) | `pt-PT` | Male | `pt-PT-DuarteNeural` <sup>Nouveau</sup> | Général |
| Roumain (Roumanie) | `ro-RO` | Female | `ro-RO-AlinaNeural` | Général |
| Roumain (Roumanie) | `ro-RO` | Male | `ro-RO-EmilNeural` <sup>Nouveau</sup> | Général |
| Russe (Russie) | `ru-RU` | Female | `ru-RU-DariyaNeural` | Général |
| Russe (Russie) | `ru-RU` | Female | `ru-RU-SvetlanaNeural` <sup>Nouveau</sup> | Général |
| Russe (Russie) | `ru-RU` | Male | `ru-RU-DmitryNeural` <sup>Nouveau</sup> | Général |
| Slovaque (Slovaquie) | `sk-SK` | Female | `sk-SK-ViktoriaNeural` | Général |
| Slovaque (Slovaquie) | `sk-SK` | Male | `sk-SK-LukasNeural` <sup>Nouveau</sup> | Général |
| Slovène (Slovénie) | `sl-SI` | Female | `sl-SI-PetraNeural` | Général |
| Slovène (Slovénie) | `sl-SI` | Male | `sl-SI-RokNeural` <sup>Nouveau</sup> | Général |
| Espagnol (Mexique) | `es-MX` | Female | `es-MX-DaliaNeural` | Général |
| Espagnol (Mexique) | `es-MX` | Male | `es-MX-JorgeNeural` | Général |
| Espagnol (Espagne) | `es-ES` | Female | `es-ES-ElviraNeural` | Général |
| Espagnol (Espagne) | `es-ES` | Male | `es-ES-AlvaroNeural` | Général |
| Suédois (Suède) | `sv-SE` | Female | `sv-SE-HilleviNeural` | Général |
| Suédois (Suède) | `sv-SE` | Female | `sv-SE-SofieNeural` <sup>Nouveau</sup> | Général |
| Suédois (Suède) | `sv-SE` | Male | `sv-SE-MattiasNeural` <sup>Nouveau</sup> | Général |
| Tamoul (Inde) | `ta-IN` | Female | `ta-IN-PallaviNeural` | Général |
| Tamoul (Inde) | `ta-IN` | Male | `ta-IN-ValluvarNeural` <sup>Nouveau</sup> | Général |
| Télougou (Inde) | `te-IN` | Female | `te-IN-ShrutiNeural` | Général |
| Télougou (Inde) | `te-IN` | Male | `te-IN-MohanNeural` <sup>Nouveau</sup> | Général |
| Thaï (Thaïlande) | `th-TH` | Female | `th-TH-AcharaNeural` | Général |
| Thaï (Thaïlande) | `th-TH` | Female | `th-TH-PremwadeeNeural` | Général |
| Thaï (Thaïlande) | `th-TH` | Male | `th-TH-NiwatNeural` <sup>Nouveau</sup> | Général |
| Turc (Turquie) | `tr-TR` | Female | `tr-TR-EmelNeural` | Général |
| Turc (Turquie) | `tr-TR` | Male | `tr-TR-AhmetNeural` <sup>Nouveau</sup> | Général |
| Vietnamien (Vietnam) | `vi-VN` | Female | `vi-VN-HoaiMyNeural` | Général |
| Vietnamien (Vietnam) | `vi-VN` | Male | `vi-VN-NamMinhNeural` <sup>Nouveau</sup> | Général |

#### <a name="neural-voices-in-preview"></a>Voix neuronales en préversion

Les voix neuronales ci-dessous sont en préversion publique. 

| Langage                         | Paramètres régionaux  | Sexe | Nom de la voix                             | Prise en charge du style |
|----------------------------------|---------|--------|----------------------------------------|---------------|
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-XiaohanNeural` | Général, plusieurs styles disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-XiaomoNeural` | Général, plusieurs jeux de rôle et styles disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-XiaoruiNeural` | Voix de senior, plusieurs styles disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-XiaoxuanNeural` | Général, plusieurs jeux de rôle et styles disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Chinois (mandarin, simplifié) | `zh-CN` | Male   | `zh-CN-YunxiNeural` | Général, plusieurs styles disponibles [à l’aide de SSML](speech-synthesis-markup.md#adjust-speaking-styles) |
| Estonien (Estonie) | `et-EE` | Female | `et-EE-AnuNeural` | Général |
| Estonien (Estonie) | `et-EE` | Male | `et-EE-KertNeural` <sup>Nouveau</sup> | Général |
| Irlandais (Irlande) | `ga-IE` | Female | `ga-IE-OrlaNeural` | Général |
| Irlandais (Irlande) | `ga-IE` | Male | `ga-IE-ColmNeural` <sup>Nouveau</sup> | Général |
| Letton (Lettonie) | `lv-LV` | Female | `lv-LV-EveritaNeural` | Général |
| Letton (Lettonie) | `lv-LV` | Male | `lv-LV-NilsNeural` <sup>Nouveau</sup> | Général |
| Lituanien (Lituanie) | `lt-LT` | Female | `lt-LT-OnaNeural` | Général |
| Lituanien (Lituanie) | `lt-LT` | Male | `lt-LT-LeonasNeural` <sup>Nouveau</sup> | Général |
| Maltais (Malte) | `mt-MT` | Female | `mt-MT-GraceNeural` | Général |
| Maltais (Malte) | `mt-MT` | Male | `mt-MT-JosephNeural` <sup>Nouveau</sup> | Général |

> [!IMPORTANT]
> Les voix en préversion publique sont uniquement disponibles dans 3 régions de service : USA Est, Europe Ouest et Asie Sud-Est.

Pour plus d’informations sur la disponibilité régionale, consultez [régions](regions.md#standard-and-neural-voices).

Pour savoir comment configurer et ajuster les voix neuronales, comme les styles de diction, consultez [Langage de balisage de synthèse vocale](speech-synthesis-markup.md#adjust-speaking-styles).

> [!IMPORTANT]
> La voix `en-US-JessaNeural` a basculé sur `en-US-AriaNeural`. Si vous utilisiez « Jessa », procédez à une conversion vers « Aria ».

> [!TIP]
> Vous pouvez continuer à utiliser le mappage de nom de service complet comme « Voix Microsoft Server Speech Text to Speech (en-US, AriaNeural) » dans vos requêtes de synthèse vocale.

### <a name="standard-voices"></a>Voix standard

Plus de 75 voix standard sont disponibles dans plus de 45 langues et paramètres régionaux, ce qui vous permet de convertir le texte en parole synthétisée. Pour plus d’informations sur la disponibilité régionale, consultez [régions](regions.md#standard-and-neural-voices).

| Langage | Paramètres régionaux (BCP-47) | Sexe | Nom de la voix |
|--|--|--|--|
| Arabe (arabe) | `ar-EG` | Female | `ar-EG-Hoda`|
| Arabe (Arabie saoudite) | `ar-SA` | Male | `ar-SA-Naayf`|
| Bulgare (Bulgarie) | `bg-BG` | Male | `bg-BG-Ivan`|
| Catalan (Espagne) | `ca-ES` | Female | `ca-ES-HerenaRUS`|
| Chinois (cantonais, traditionnel) | `zh-HK` | Male | `zh-HK-Danny`|
| Chinois (cantonais, traditionnel) | `zh-HK` | Female | `zh-HK-TracyRUS`|
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-HuihuiRUS`|
| Chinois (mandarin, simplifié) | `zh-CN` | Male | `zh-CN-Kangkang`|
| Chinois (mandarin, simplifié) | `zh-CN` | Female | `zh-CN-Yaoyao`|
| Chinois (mandarin, taïwanais) |  `zh-TW` | Female | `zh-TW-HanHanRUS`|
| Chinois (mandarin, taïwanais) |  `zh-TW` | Female | `zh-TW-Yating`|
| Chinois (mandarin, taïwanais) |  `zh-TW` | Male | `zh-TW-Zhiwei`|
| Croate (Croatie) | `hr-HR` | Male | `hr-HR-Matej`|
| Tchèque (République tchèque) | `cs-CZ` | Male | `cs-CZ-Jakub`|
| Danois (Danemark) | `da-DK` | Female | `da-DK-HelleRUS`|
| Néerlandais (Pays-Bas) | `nl-NL` | Female | `nl-NL-HannaRUS`|
| Anglais (Australie) | `en-AU` | Female | `en-AU-Catherine`|
| Anglais (Australie) | `en-AU` | Female | `en-AU-HayleyRUS`|
| Anglais (Canada) | `en-CA` | Female | `en-CA-HeatherRUS`|
| Anglais (Canada) | `en-CA` | Female | `en-CA-Linda`|
| Anglais (Inde) | `en-IN` | Female | `en-IN-Heera`|
| Anglais (Inde) | `en-IN` | Female | `en-IN-PriyaRUS`|
| Anglais (Inde) | `en-IN` | Male | `en-IN-Ravi`|
| Anglais (Irlande) | `en-IE` | Male | `en-IE-Sean`|
| Anglais (Royaume-Uni) | `en-GB` | Male | `en-GB-George`|
| Anglais (Royaume-Uni) | `en-GB` | Female | `en-GB-HazelRUS`|
| Anglais (Royaume-Uni) | `en-GB` | Female | `en-GB-Susan`|
| Anglais (États-Unis) | `en-US` | Male | `en-US-BenjaminRUS`|
| Anglais (États-Unis) | `en-US` | Male | `en-US-GuyRUS`|
| Anglais (États-Unis) | `en-US` | Female | `en-US-JessaRUS`|
| Anglais (États-Unis) | `en-US` | Female | `en-US-ZiraRUS`|
| Finnois (Finlande) | `fi-FI` | Female | `fi-FI-HeidiRUS`|
| Français (Canada) | `fr-CA` | Female | `fr-CA-Caroline`|
| Français (Canada) | `fr-CA` | Female | `fr-CA-HarmonieRUS`|
| Français (France) | `fr-FR` | Female | `fr-FR-HortenseRUS`|
| Français (France) | `fr-FR` | Female | `fr-FR-Julie`|
| Français (France) | `fr-FR` | Male | `fr-FR-Paul`|
| Français (Suisse) | `fr-CH` | Male | `fr-CH-Guillaume`|
| Allemand (Autriche) | `de-AT` | Male | `de-AT-Michael`|
| Allemand (Allemagne) | `de-DE` | Female | `de-DE-HeddaRUS`|
| Allemand (Allemagne) | `de-DE` | Male | `de-DE-Stefan`|
| Allemand (Suisse) | `de-CH` | Male | `de-CH-Karsten`|
| Grec (Grèce) | `el-GR` | Male | `el-GR-Stefanos`|
| Hébreu (Israël) | `he-IL` | Male | `he-IL-Asaf`|
| Hindi (Inde) | `hi-IN` | Male | `hi-IN-Hemant`|
| Hindi (Inde) | `hi-IN` | Female | `hi-IN-Kalpana`|
| Hongrois (Hongrie) | `hu-HU` | Male | `hu-HU-Szabolcs`|
| Indonésien (Indonésie) | `id-ID` | Male | `id-ID-Andika`|
| Italien (Italie) | `it-IT` | Male | `it-IT-Cosimo`|
| Italien (Italie) | `it-IT` | Female | `it-IT-LuciaRUS`|
| Japonais (Japon) | `ja-JP` | Female | `ja-JP-Ayumi`|
| Japonais (Japon) | `ja-JP` | Female | `ja-JP-HarukaRUS`|
| Japonais (Japon) | `ja-JP` | Male | `ja-JP-Ichiro`|
| Coréen (Corée) | `ko-KR` | Female | `ko-KR-HeamiRUS`|
| Malais (Malaisie) | `ms-MY` | Male | `ms-MY-Rizwan`|
| Norvégien (bokmål, Norvège) | `nb-NO` | Female | `nb-NO-HuldaRUS`|
| Polonais (Pologne) | `pl-PL` | Female | `pl-PL-PaulinaRUS`|
| Portugais (Brésil) | `pt-BR` | Male | `pt-BR-Daniel`|
| Portugais (Brésil) | `pt-BR` | Female | `pt-BR-HeloisaRUS`|
| Portugais (Portugal) | `pt-PT` | Female | `pt-PT-HeliaRUS`|
| Roumain (Roumanie) | `ro-RO` | Male | `ro-RO-Andrei`|
| Russe (Russie) | `ru-RU` | Female | `ru-RU-EkaterinaRUS`|
| Russe (Russie) | `ru-RU` | Female | `ru-RU-Irina`|
| Russe (Russie) | `ru-RU` | Male | `ru-RU-Pavel`|
| Slovaque (Slovaquie) | `sk-SK` | Male | `sk-SK-Filip`|
| Slovène (Slovénie) | `sl-SI` | Male | `sl-SI-Lado`|
| Espagnol (Mexique) | `es-MX` | Female | `es-MX-HildaRUS`|
| Espagnol (Mexique) | `es-MX` | Male | `es-MX-Raul`|
| Espagnol (Espagne) | `es-ES` | Female | `es-ES-HelenaRUS`|
| Espagnol (Espagne) | `es-ES` | Female | `es-ES-Laura`|
| Espagnol (Espagne) | `es-ES` | Male | `es-ES-Pablo`|
| Suédois (Suède) | `sv-SE` | Female | `sv-SE-HedvigRUS`|
| Tamoul (Inde) | `ta-IN` | Male | `ta-IN-Valluvar`|
| Télougou (Inde) | `te-IN` | Female | `te-IN-Chitra`|
| Thaï (Thaïlande) | `th-TH` | Male | `th-TH-Pattara`|
| Turc (Turquie) | `tr-TR` | Female | `tr-TR-SedaRUS`|
| Vietnamien (Vietnam) | `vi-VN` | Male | `vi-VN-An` |

> [!IMPORTANT]
> La voix `en-US-Jessa` a basculé sur `en-US-Aria`. Si vous utilisiez « Jessa », procédez à une conversion vers « Aria ».

> [!TIP]
> Vous pouvez continuer à utiliser le mappage de nom de service complet comme « Voix Microsoft Server Speech Text to Speech (en-US, AriaRUS) » dans vos requêtes de synthèse vocale.

### <a name="customization"></a>Personnalisation

La personnalisation vocale est disponible pour `de-DE`, `en-GB`, `en-IN`, `en-US`, `es-MX`, `fr-FR`, `it-IT`, `pt-BR`et `zh-CN`. Sélectionnez les paramètres régionaux qui correspondent aux données d’apprentissage servant à entraîner un modèle vocal personnalisé. Par exemple, si vos données d’enregistrement sont en anglais avec un accent britannique, sélectionnez `en-GB`.

> [!NOTE]
> À l’exception du chinois-anglais, nous ne gérons pas l’apprentissage de modèles bilingues dans Custom Voice. Sélectionnez « Bilingue chinois-anglais » si vous souhaitez effectuer l’apprentissage d’une voix en chinois qui parle aussi anglais. Dans tous les paramètres régionaux, la formation vocale commence par un jeu de données de plus de 2 000 énoncés, à l’exception de `en-US` et `zh-CN`, pour lesquels la taille des données de formation de départ n’a pas d’importance.

## <a name="speech-translation"></a>Traduction vocale

L’API **Traduction vocale** prend en charge différentes langues pour la traduction de parole en parole et de parole en texte. La langue source doit toujours provenir du tableau des langues de reconnaissance vocale. Les langues cibles disponibles dépendent selon que cible de la traduction est de la parole ou du texte. Vous pouvez traduire les conversations entrantes dans plus de [60 langues](https://www.microsoft.com/translator/business/languages/). Un sous-ensemble de langues est disponible pour la [synthèse vocale](language-support.md#text-languages).

### <a name="text-languages"></a>Langues de texte

| Langue de texte           | Code langue |
|:------------------------|:-------------:|
| Afrikaans               | `af`          |
| Arabe                  | `ar`          |
| Bangla                  | `bn`          |
| Bosniaque (latin)         | `bs`          |
| Bulgare               | `bg`          |
| Cantonais (traditionnel) | `yue`         |
| Catalan                 | `ca`          |
| Chinois (simplifié)      | `zh-Hans`     |
| Chinois traditionnel     | `zh-Hant`     |
| Croate                | `hr`          |
| Tchèque                   | `cs`          |
| Danois                  | `da`          |
| Néerlandais                   | `nl`          |
| Anglais                 | `en`          |
| Estonien                | `et`          |
| Fidjien                  | `fj`          |
| Filipino                | `fil`         |
| Finnois                 | `fi`          |
| Français                  | `fr`          |
| Allemand                  | `de`          |
| Grec                   | `el`          |
| Goudjrati                | `gu`          |
| Créole haïtien          | `ht`          |
| Hébreu                  | `he`          |
| Hindi                   | `hi`          |
| Hmong blanc               | `mww`         |
| Hongrois               | `hu`          |
| Indonésien              | `id`          |
| Irlandais                   | `ga`          |
| Italien                 | `it`          |
| Japonais                | `ja`          |
| Kannada                 | `kn`          |
| Kiswahili               | `sw`          |
| Klingon                 | `tlh-Latn`    |
| Klingon (plqaD)         | `tlh-Piqd`    |
| Coréen                  | `ko`          |
| Letton                 | `lv`          |
| Lituanien              | `lt`          |
| Malgache                | `mg`          |
| Malais                   | `ms`          |
| Malayalam               | `ml`          |
| Maltais                 | `mt`          |
| Maori                   | `mi`          |
| Marathi                 | `mr`          |
| Norvégien               | `nb`          |
| Persan                 | `fa`          |
| Polonais                  | `pl`          |
| Portugais (Brésil)     | `pt-br`       |
| Portugais (Portugal)   | `pt-pt`       |
| Pendjabi                 | `pa`          |
| Queretaro Otomi         | `otq`         |
| Roumain                | `ro`          |
| Russe                 | `ru`          |
| Samoan                  | `sm`          |
| Serbe (cyrillique)      | `sr-Cyrl`     |
| Serbe (latin)         | `sr-Latn`     |
| Slovaque                  | `sk`          |
| Slovène               | `sl`          |
| Espagnol                 | `es`          |
| Suédois                 | `sv`          |
| Tahitien                | `ty`          |
| Tamoul                   | `ta`          |
| Télougou                  | `te`          |
| Thaï                    | `th`          |
| Tonga                  | `to`          |
| Turc                 | `tr`          |
| Ukrainien               | `uk`          |
| Ourdou                    | `ur`          |
| Vietnamien              | `vi`          |
| Gallois                   | `cy`          |
| Yucatec Maya            | `yua`         |

## <a name="speaker-recognition"></a>Reconnaissance de l’orateur

Consultez le tableau suivant qui répertorie les langues prises en charge pour les différentes API Reconnaissance de l’orateur. Consultez la [vue d’ensemble](speaker-recognition-overview.md) pour plus d’informations sur Reconnaissance de l’orateur.

| Langage | Paramètres régionaux (BCP-47) | Vérification dépendante du texte | Vérification indépendante du texte | Identification indépendante du texte |
|----|----|----|----|----|
|Anglais (US)  |  en-US  |  Oui  |  Oui  |  Oui |
|Chinois (mandarin, simplifié) | zh-CN     |     n/a |     Oui |     Oui|
|Anglais (Australie)     | en-AU     | n/a     | Oui     | Oui|
|Anglais (Canada)     | en-CA     | n/a |     Oui |     Oui|
|Anglais (Royaume-Uni)     | en-GB     | n/a     | Oui     | Oui|
|Français (Canada)     | fr-CA     | n/a     | Oui |     Oui|
|Français (France)     | fr-FR     | n/a     | Oui     | Oui|
|Allemand (Allemagne)     | de-DE     | n/a     | Oui     | Oui|
|Italien | it-IT     |     n/a     | Oui |     Oui|
|Japonais     | ja-JP | n/a     | Oui     | Oui|
|Portugais (Brésil) | pt-br |     n/a |     Oui |     Oui|
|Espagnol (Mexique)     | es-MX     | n/a |     Oui |     Oui|
|Espagnol (Espagne)     | es-ES | n/a     | Oui |     Oui|

## <a name="next-steps"></a>Étapes suivantes

* [Créez un compte Azure gratuit](https://azure.microsoft.com/free/cognitive-services/)
* [Découvrir comment utiliser la reconnaissance vocale en C#](./get-started-speech-to-text.md?pivots=programming-language-chsarp)
