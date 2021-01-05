---
title: Notes de publication - Service Speech
titleSuffix: Azure Cognitive Services
description: Journal constamment mis à jour des nouvelles fonctionnalités, des améliorations, des correctifs de bogues et des problèmes connus du service Speech.
services: cognitive-services
author: oliversc
manager: jhakulin
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: oliversc
ms.custom: seodec18
ms.openlocfilehash: 2db93ec6806d4b9cc8c8ac17cfb2c15458bf4dc9
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97401149"
---
# <a name="speech-service-release-notes"></a>Notes de publication du service Speech

## <a name="text-to-speech-2020-december-release"></a>Conversion de texte par synthèse vocale – Version de décembre 2020

**Nouvelles voix neuronales en disponibilité générale et en préversion**

Publication de 51 nouvelles voix pour un total de 129 voix neuronales dans 54 langues/paramètres régionaux :

- **46 nouvelles voix dans les paramètres régionaux en disponibilité générale** : Shakir en `ar-EG` arabe (Égypte), Hamed en `ar-SA` arabe (Arabie saoudite), Borislav en `bg-BG` bulgare (Bulgarie), Joana en `ca-ES` catalan (Espagne), Antonin en `cs-CZ` tchèque (République tchèque), Jeppe en `da-DK` danois (Danemark), Jonas en `de-AT` allemand (Autriche), Jan en `de-CH` allemand (Suisse), Nestoras en `el-GR` grec (Grèce), Liam en `en-CA` anglais (Canada), Connor en `en-IE` anglais (Irlande), Madhur en `en-IN` hindi (Inde), Mohan en `en-IN` télougou (Inde), Prabhat en `en-IN` anglais (Inde), Valluvar en `en-IN` tamoul (Inde), Enric en `es-ES` catalan (Espagne), Kert en `et-EE` estonien (Estonie), Harri en `fi-FI` finnois (Finlande), Selma en `fi-FI` finnois (Finlande), Fabrice en `fr-CH` français (Suisse), Colm en `ga-IE` irlandais (Irlande), Avri en `he-IL` hébreu (Israël), Srecko en `hr-HR` croate (Croatie), Tamas en `hu-HU` hongrois (Hongrie), Gadis en `id-ID` indonésien (Indonésie), Leonas en `lt-LT` lituanien (Lituanie), Nils en `lv-LV` letton (Lettonie), Osman en `ms-MY` malais (Malaisie), Joseph en `mt-MT` maltais (Malte), Finn en `nb-NO` norvégien, Bokmål (Norvège), Pernille en `nb-NO` norvégien, Bokmål (Norvège), Fenna en `nl-NL` Néerlandais (Pays-Bas), Maarten en néerlandais `nl-NL` (Pays-Bas), Agnieszka en `pl-PL` polonais (Pologne), Marek en `pl-PL` polonais (Pologne), Duarte `pt-BR` en portugais (Brésil), Raquel `pt-PT` en portugais (Potugal), Emil en `ro-RO` roumain (Roumanie), Dmitry en `ru-RU` russe (Russie), Svetlana en `ru-RU` russe (Russie), Lukas en `sk-SK` slovaque (Slovaquie), Rok en `sl-SI` slovène (Slovénie), Mattias en `sv-SE` suédois (Suède), Sofie en `sv-SE` suédois (Suède), Niwat en `th-TH` thaï (Thaïlande), Ahmet en `tr-TR` turc (Turquie), NamMinh en `vi-VN` vietnamien (Viêt Nam), HsiaoChen en `zh-TW` mandarin taïwanais (Taïwan), YunJhe en `zh-TW` mandarin taïwanais (Taïwan), HiuMaan en `zh-HK` chinois cantonais (Hong Kong), WanLung en `zh-HK` chinois cantonais (Hong Kong).

- **5 nouvelles voix dans les paramètres régionaux en préversion** : Kert en `et-EE` estonien (Estonie), Colm en `ga-IE` irlandais (Irlande), Nils en `lv-LV` letton (Lettonie), Leonas en `lt-LT` lituanien (Lituanie), Joseph en `mt-MT` maltais (Malte).

Avec cette version, nous proposons désormais un total de 129 voix dans 54 langues/paramètres régionaux. En outre, plus de 70 voix standard sont disponibles dans 49 langues/paramètres régionaux. Pour obtenir la liste complète, consultez [Prise en charge linguistique](language-support.md#text-to-speech).

**Mises à jour pour la création de contenu audio**
- Interface utilisateur améliorée de sélection de la voix avec des catégories de voix et des descriptions de voix détaillées. 
- Activation du paramétrage de l’intonation pour toutes les voix neuronales dans différentes langues.
- Automatisation de la localisation de l’interface utilisateur en fonction de la langue du navigateur.
- Activation des contrôles `StyleDegree` pour toutes les voix neuronales `zh-CN`.
Visitez l’[outil de création de contenu audio](https://speech.microsoft.com/audiocontentcreation) pour découvrir les nouvelles fonctionnalités. 

**Mises à jour pour les voix zh-CN**
- Mise à jour de toutes les voix neuronales `zh-CN` pour prendre en charge l’anglais.
- Activation de toutes les voix neuronales `zh-CN` pour prendre en charge le réglage de l’intonation. Possibilité d’utiliser SSML ou l’outil de création de contenu audio pour régler la meilleure intonation.
- Mise à jour de toutes les voix neuronales `zh-CN` disponibles dans plusieurs styles pour la prise en charge du contrôle `StyleDegree`. L’intensité de l’émotion (douce ou forte) est réglable.
- Mise à jour de `zh-CN-YunyeNeural` pour prendre en charge plusieurs styles pouvant refléter différentes émotions.

## <a name="text-to-speech-2020-november-release"></a>Conversion de texte par synthèse vocale – Version de novembre 2020

**Nouveaux paramètres régionaux et voix en préversion**
- **Cinq nouvelles voix et langues** sont introduites dans le portefeuille de synthèse vocale neuronale. Il s'agit de : Grâce en maltais (Malte), Ona en lituanien (Lituanie), Anu en estonien (Estonie), Orla en irlandais (Irlande) et Everita en letton (Lettonie).
- **Cinq nouvelles `zh-CN` voix avec prise en charge de plusieurs styles et rôles** : Xiaohan, Xiaomo, Xiaorui, Xiaoxuan et Yunxi.

> Ces voix sont disponibles en préversion publique da,s trois régions Azure : Eastus, southeastasia et westeurope.

**Disponibilité générale du conteneur neuronal de synthèse vocale neuronale**
- Avec le conteneur de synthèse vocale neuronale, les développeurs peuvent effectuer une synthèse vocale avec les voix numériques les plus naturelles dans leur propre environnement pour des besoins spécifiques en matière de sécurité et de gouvernance des données. Découvrez [comment installer les conteneurs Speech](speech-container-howto.md). 

**Nouvelles fonctionnalités**
- **Custom Voice** : possibilité pour les utilisateurs de copier un modèle vocal d’une région vers une autre ; prise en charge de l’interruption et reprise de point de terminaison. Accédez au [portail](https://speech.microsoft.com/customvoice) ici.
- Prise en charge de la [balise de silence SSML](speech-synthesis-markup.md#add-silence). 
- Améliorations générales de la qualité de la synthèse vocale : Amélioration de la précision de la prononciation des mots dans 5 langues. Réduction de 53 % des erreurs de prononciation.

> Pour plus d’informations, consultez ce [blog sur les technologies](https://techcommunity.microsoft.com/t5/azure-ai/neural-text-to-speech-previews-five-new-languages-with/ba-p/1907604).

## <a name="text-to-speech-2020-october-release"></a>Conversion de texte par synthèse vocale - Version d’août 2020

**Nouvelles fonctionnalités**
- Jenny prend en charge un nouveau style de `newscast`. Consultez [Comment utiliser les styles oraux dans SSML](speech-synthesis-markup.md#adjust-speaking-styles).
- **Voix neuronales mises à niveau vers le vocodeur HiFiNet, avec une fidélité audio supérieure et une vitesse de synthèse plus rapide**. Cela présente des avantages pour les clients dont le scénario repose sur l’audio hi-fi ou de longues interactions, notamment le doublage vidéo, les livres audio ou le matériel pédagogique en ligne. [En savoir plus et écouter les échantillons vocaux sur notre blog Tech Community](https://techcommunity.microsoft.com/t5/azure-ai/azure-neural-tts-upgraded-with-hifinet-achieving-higher-audio/ba-p/1847860)
- **[Custom Voice](https://speech.microsoft.com/customvoice) & [Studio Création de contenu audio](https://speech.microsoft.com/audiocontentcreation) localisé en 17 langues**. Les utilisateurs peuvent facilement changer la langue de l’IU pour une expérience plus conviviale.   
- **Création de contenu audio** : Ajout du contrôle de degré de style pour XiaoxiaoNeural ; amélioration de la fonctionnalité d’arrêt personnalisé pour inclure des arrêts incrémentiels de 50 ms. 

**Améliorations générales de la qualité de la voix TTS**
- Amélioration de la justesse de la prononciation au niveau des mots en `pl-PL` (réduction du taux d’erreur : 51 %) et `fi-FI` (réduction du taux d’erreur : 58 %)
- Amélioration de la lecture d’un seul mot en `ja-JP` pour le scénario de dictionnaire. Réduction des erreurs de prononciation de 80 %.
- `zh-CN-XiaoxiaoNeural` : Amélioration de la qualité vocale du style sentiment/CustomerService/Newscast/Cheerful/Angry.
- `zh-CN` : Amélioration de la prononciation Erhua, de la légèreté de ton et de la prosodie spatiale, ce qui améliore considérablement l’intelligibilité. 

## <a name="speech-sdk-1140-2020-october-release"></a>Kit SDK Speech 1.14.0 : version d’octobre 2020

**Remarque** : Le kit SDK Speech sur Windows dépend du partage de Redistributable Visual C++ pour Visual Studio 2015, 2017 et 2019. Téléchargez-le [ici](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

**Nouvelles fonctionnalités**
- **Linux** : Ajout de la prise en charge de Debian 10 et Ubuntu 20.04 LTS.
- **Python/Objective-C** : Ajout de la prise en charge de l’API `KeywordRecognizer`. La documentation sera disponible [ici](./custom-keyword-basics.md).
- **C++/Java/C#**  : Ajout de la prise en charge pour définir toute clé/valeur `HttpHeader` par le biais de `ServicePropertyChannel::HttpHeader`.
- **JavaScript** : Ajout de la prise en charge de l’API `ConversationTranscriber`. Lisez la documentation [ici](./how-to-use-conversation-transcription.md?pivots=programming-language-javascript). 
- **C++/C#**  : Ajout d’une nouvelle méthode `AudioDataStream FromWavFileInput` (pour lire les fichiers .WAV) [ici (C++)](/cpp/cognitive-services/speech/audiodatastream) et [ici (C#)](/dotnet/api/microsoft.cognitiveservices.speech.audiodatastream?view=azure-dotnet).
-  **C++/C#/Java/Python/Objective-C/Swift** : Ajout d’une méthode `stopSpeakingAsync()` pour arrêter la synthèse vocale. Lisez la documentation de référence [ici (C++)](/cpp/cognitive-services/speech/microsoft-cognitiveservices-speech-namespace), [ici (C#)](/dotnet/api/microsoft.cognitiveservices.speech?view=azure-dotnet), [ici (Java)](/java/api/com.microsoft.cognitiveservices.speech), [ici (Python)](/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech?view=azure-python) et [ici (Objective-C/Swift)](/objectivec/cognitive-services/speech/).
- **C#, C++, Java** : Ajout d’une fonction `FromDialogServiceConnector()` à la classe `Connection` qui peut être utilisée pour superviser les événements de connexion et de déconnexion pour `DialogServiceConnector`. Lisez la documentation de référence [ici (C#)](/dotnet/api/microsoft.cognitiveservices.speech.connection?view=azure-dotnet), [ici (C++)](/cpp/cognitive-services/speech/connection) et [ici (Java)](/java/api/com.microsoft.cognitiveservices.speech.connection).
- **C++/C#/Java/Python/Objective-C/Swift** : Ajout de la prise en charge de l’évaluation de la prononciation, qui évalue la prononciation des entrées vocales et fournit des commentaires aux orateurs sur l’exactitude et la fluidité de l’audio parlé. Lisez la documentation [ici](how-to-pronunciation-assessment.md).

**Modification critique**
- **JavaScript** : PullAudioOutputStream.Read() a un type de retour modifié d’une promesse interne à une promesse JavaScript native.

**Résolution des bogues**
- **Tout** : Correction de la régression 1.13 dans `SetServiceProperty` où les valeurs avec certains caractères spéciaux étaient ignorées.
- **C#**  : Correction des exemples de console Windows sur Visual Studio 2019 qui ne parvenaient pas à trouver les DLL natives.
- **C#**  : Correction d’un plantage avec la gestion de la mémoire si un flux était utilisé en tant qu’entrée `KeywordRecognizer`.
- **ObjectiveC/Swift** : Correction d’un plantage avec la gestion de la mémoire si un flux était utilisé en tant qu’entrée du module de reconnaissance.
- **Windows** : Résolution d’un problème de coexistence avec BT HFP/A2DP sur UWP.
- **JavaScript** : Correction du mappage des ID de session pour améliorer la journalisation et faciliter les corrélations de débogage/service internes.
- **JavaScript** : Ajout d’un correctif à `DialogServiceConnector` pour la désactivation des appels de `ListenOnce` après le premier appel.
- **JavaScript** : Résolution d’un problème où la sortie du résultat était toujours « simple ».
- **JavaScript** : Résolution d’un problème de reconnaissance continue dans Safari sur macOS.
- **JavaScript** : Atténuation de la charge du processeur pour le scénario de débit de requête élevé.
- **JavaScript** : Autorisation de l’accès aux détails de VoiceProfileEnrollmentResult.
- **JavaScript** : Ajout d’un correctif pour la reconnaissance continue dans `IntentRecognizer`.
- **C++/C#/Java/Python/Swift/ObjectiveC** : Correction d’une URL incorrecte pour australiaeast et brazilsouth dans `IntentRecognizer`.
- **C++/C#**  : Ajout de `VoiceProfileType` en tant qu’argument lors de la création d’un objet `VoiceProfile`.
- **C++/C#/Java/Python/Swift/ObjectiveC** : Correction de `SPX_INVALID_ARG` potentiel lors de la tentative de lecture de `AudioDataStream` à partir d’une position donnée.
- **IOS** : Correction d’un plantage de la reconnaissance vocale sur Unity

**Exemples**
- **ObjectiveC** : Ajout d’un exemple de reconnaissance de mot clé [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/objective-c/ios/speech-samples).
- **C#/JavaScript** : Ajout d’un guide de démarrage rapide pour la transcription de conversation [ici (C#)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/conversation-transcription) et [ici (JavaScript)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/conversation-transcription).
- **C++/C#/Java/Python/Swift/ObjectiveC** : Exemple ajouté pour l’évaluation de la prononciation [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples)
- **Xamarin** : Mise à jour du guide de démarrage rapide vers le dernier modèle Visual Studio [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/xamarin).

**Problème connu**
- Le certificat DigiCert Global Root G2 n’est pas pris en charge par défaut dans HoloLens 2 et Android 4.4 (KitKat), et doit être ajouté au système pour que le SDK Speech fonctionne. Le certificat sera ajouté prochainement aux images de système d’exploitation HoloLens 2. Les clients Android 4.4 doivent ajouter le certificat mis à jour au système.

**Test raccourci COVID-19 :** en raison du travail à distance au cours des dernières semaines, nous n’avons pas pu effectuer autant de tests de vérification manuelle que nous le faisons habituellement. Nous n’avons apporté aucune modification qui aurait pu casser quoi que ce soit, et nos tests automatisés ont tous réussi. Dans le cas peu probable où nous aurions manqué quelque chose, veuillez nous en informer sur [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Restez en bonne santé !

## <a name="speech-cli-also-known-as-spx-2020-october-release"></a>Interface CLI Speech (également appelée SPX) : version d’octobre 2020
SPX est l’interface de ligne de commande pour utiliser le service Azure Speech sans écrire de code. Téléchargez la dernière version [ici](./spx-basics.md). <br>

**Nouvelles fonctionnalités**
- `spx csr dataset upload --kind audio|language|acoustic` : créez des jeux de données à partir de données locales, et non uniquement à partir d’URL.
- `spx csr evaluation create|status|list|update|delete` : comparez les nouveaux modèles par rapport à la vérité de référence ou à d’autres modèles.
- `spx * list` : prend en charge l’expérience non paginée (ne nécessite pas --top X --skip X).
- `spx * --http header A=B` : prend en charge les en-têtes personnalisés (ajoutés pour Office pour l’authentification personnalisée). 
- `spx help` : amélioration du codage couleur du texte et des accents graves (bleu).

## <a name="text-to-speech-2020-september-release"></a>Conversion de texte par synthèse vocale - Version de septembre 2020

### <a name="new-features"></a>Nouvelles fonctionnalités

* **Synthèse vocale neuronale** 
    * **Extension de la prise en charge à 18 nouvelles langues/nouveaux paramètres régionaux.** Il s’agit du bulgare, du tchèque, de l’allemand (Autriche), de l’allemand (Suisse), du grec, de l’anglais (Irlande), du français (Suisse), de l’hébreu, du croate, du hongrois, de l’indonésien, du malais, du roumain, du slovaque, du slovène, du tamoul, du télougou et du vietnamien. 
    * **14 nouvelles voix disponibles pour enrichir la variété des langues existantes.** Consultez la [liste complète des langues et des voix](language-support.md#neural-voices).
    * **Nouveaux styles de parole pour les voix `en-US` et `zh-CN`.** Jenny, la nouvelle voix en anglais (États-Unis), prend en charge les styles du chatbot, du service clientèle et de l’assistant. 10 nouveaux styles de parole sont disponibles avec notre voix zh-CN, XiaoXiao. En outre, la voix neuronale XiaoXiao prend en charge le réglage de `StyleDegree`. Consultez [Comment utiliser les styles oraux dans SSML](speech-synthesis-markup.md#adjust-speaking-styles).

* **Conteneurs : Conteneur de synthèse vocale neuronale disponible en préversion publique avec 16 voix disponibles en 14 langues.** En savoir plus sur le [déploiement des conteneurs vocaux pour la synthèse vocale neuronale](speech-container-howto.md)  

Lisez l’[annonce complète des mises à jour de la synthèse vocale (TTS) pour Ignite 2020](https://techcommunity.microsoft.com/t5/azure-ai/ignite-2020-neural-tts-updates-new-language-support-more-voices/ba-p/1698544) 

## <a name="text-to-speech-2020-august-release"></a>Synthèse vocale - version août 2020

### <a name="new-features"></a>Nouvelles fonctionnalités

* **Synthèse vocale neuronale : nouveau style d’élocution pour la voix Aria `en-US`** . AriaNeural peut ressembler à un présentateur de journal télévisé. Le style « newscast-formal » est plus sérieux, tandis que le style « newscast-casual » est plus décontracté et informel. Consultez [Comment utiliser les styles oraux dans SSML](speech-synthesis-markup.md).

* **Voix personnalisée : une nouvelle fonctionnalité est disponible pour vérifier automatiquement la qualité des données de formation**. Lorsque vous chargez vos données, le système examine divers aspects de vos données audio et de transcription, et corrige ou filtre automatiquement les problèmes pour améliorer la qualité du modèle vocal. Cela couvre le volume de votre audio, le niveau de bruit, la précision de la prononciation, l’alignement du texte prononcé avec le texte normalisé, le silence dans l’audio, en plus du format audio et de script. 

* **Création de contenu audio : un ensemble de nouvelles fonctionnalités pour activer des fonctionnalités de gestion audio et de réglage vocal plus puissantes**

    * Prononciation : la fonctionnalité de paramétrage de la prononciation a été mise à jour avec le dernier jeu de phonèmes. Vous pouvez choisir l’élément phonème approprié dans la bibliothèque et affiner la prononciation des mots que vous avez sélectionnés. 

    * Télécharger : La fonctionnalité audio « Télécharger / Exporter » a été améliorée pour prendre en charge la génération d’audio par paragraphe. Vous pouvez modifier le contenu d’un même fichier/SSML tout en générant plusieurs sorties audio. La structure de fichiers du dossier « Download » a également été affinée. Vous pouvez maintenant obtenir facilement tous les fichiers audio dans un même dossier. 

    * État de la tâche : L’expérience d’exportation de plusieurs fichiers a été améliorée. Auparavant, lorsque vous exportiez plusieurs fichiers, si l'un des fichiers échouait, toute la tâche échouait. Mais maintenant, tous les autres fichiers seront exportés avec succès. Le rapport de tâche contient des informations plus détaillées et structurées. Ce rapport vous permet maintenant de consulter les journaux de tous les fichiers et phrases ayant échoué. 

    * Documentation SSML : liée au document SSML pour vous aider à vérifier les règles d’utilisation de toutes les fonctionnalités de paramétrage.

* **L’API Voice List a été mise à jour pour inclure un nom complet convivial ainsi que les styles oraux pris en charge pour les voix neurales**.

### <a name="general-tts-voice-quality-improvements"></a>Améliorations générales de la qualité de la voix TTS

* Réduction du pourcentage d’erreurs de prononciation au niveau du mot pour `ru-RU` (erreurs réduites de 56 %) et `sv-SE` (erreurs réduites de 49 %)

* Amélioration de 40 % de la lecture des mots polyphoniques pour les voix neuronales `en-US`. Exemples de mots polyphoniques : « read », « live », « content », « record », « object », etc. 

* Amélioration de la tonalité naturelle des questions en `fr-FR`. Gain MOS (Mean Opinion Score) : +0.28

* Mise à jour des vocodeurs pour les voix suivantes, avec amélioration de la fidélité et accélération des performances globales de 40 %.

    | Paramètres régionaux | Voix |
    |---|---|    
    | `en-GB` | Mia |
    | `es-MX` | Dalia |
    | `fr-CA` | Sylvie |
    | `fr-FR` | Denise |
    | `ja-JP` | Nanami |
    | `ko-KR` | Sun-Hi |

### <a name="bug-fixes"></a>Résolution des bogues

* Correction de plusieurs bogues liés à l’outil de création de contenu audio 
    * Correction d’un problème lié à l’actualisation automatique. 
    * Correction de problèmes liés aux styles vocaux en zh-CN dans la région Asie Sud-Est.
    * Correction d’un problème de stabilité, notamment une erreur d’exportation avec la balise « break », et d’autres erreurs de ponctuation.

## <a name="new-speech-to-text-locales-2020-august-release"></a>Nouveaux paramètres régionaux de reconnaissance vocale : Version août 2020
Reconnaissance vocale pour 26 nouvelles langues en août : 2 langues européennes (`cs-CZ` et `hu-HU`), 5 langues anglophones, et 19 langues hispaniques couvrant la plupart des pays d’Amérique du Sud. Vous trouverez ci-dessous une liste des nouvelles langues. Cliquez [ici](./language-support.md) pour consulter la liste de toutes les langues.

| Paramètres régionaux  | Langage                          |
|---------|-----------------------------------|
| `cs-CZ` | Tchèque (République tchèque)            | 
| `en-HK` | Anglais (Hong Kong)               | 
| `en-IE` | Anglais (Irlande)                 | 
| `en-PH` | Anglais (Philippines)             | 
| `en-SG` | Anglais (Singapour)               | 
| `en-ZA` | Anglais (Afrique du Sud)            | 
| `es-AR` | Espagnol (Argentine)               | 
| `es-BO` | Espagnol (Bolivie)                 | 
| `es-CL` | Espagnol (Chili)                   | 
| `es-CO` | Espagnol (Colombie)                | 
| `es-CR` | Espagnol (Costa Rica)              | 
| `es-CU` | Espagnol (Cuba)                    | 
| `es-DO` | Espagnol (République dominicaine)      | 
| `es-EC` | Espagnol (Équateur)                 | 
| `es-GT` | Espagnol (Guatemala)               | 
| `es-HN` | Espagnol (Honduras)                | 
| `es-NI` | Espagnol (Nicaragua)               | 
| `es-PA` | Espagnol (Panama)                  | 
| `es-PE` | Espagnol (Pérou)                    | 
| `es-PR` | Espagnol (Porto Rico)             | 
| `es-PY` | Espagnol (Paraguay)                | 
| `es-SV` | Espagnol (Salvador)             | 
| `es-US` | Espagnol (États-Unis)                     | 
| `es-UY` | Espagnol (Uruguay)                 | 
| `es-VE` | Espagnol (Venezuela)               | 
| `hu-HU` | Hongrois (Hongrie)               | 


## <a name="speech-sdk-1130-2020-july-release"></a>Kit SDK Speech 1.13.0 : version de juillet 2020

**Remarque** : Le kit SDK Speech sur Windows dépend du partage de Redistributable Visual C++ pour Visual Studio 2015, 2017 et 2019. Téléchargez et installez l’application à partir d’[ici](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

**Nouvelles fonctionnalités**
- **C#**  : Ajout de la prise en charge de la transcription de conversation asynchrone. Consultez la documentation [ici](./how-to-async-conversation-transcription.md).  
- **JavaScript** : Ajout de la prise en charge de la reconnaissance de l’orateur pour le [navigateur](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/browser/speaker-recognition) et [node.js](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/speaker-recognition).
- **JavaScript** : Ajout de la prise en charge de la détection de langue automatique/ID de langue. Consultez la documentation [ici](./how-to-automatic-language-detection.md?pivots=programming-language-javascript).
- **Objective-C**: Ajout de la prise en charge de la [conversation multi-appareil](./multi-device-conversation.md) et de la [transcription de conversation](./conversation-transcription.md). 
- **Python** : Ajout de la prise en charge des contenus audio compressés pour Python sur Windows et Linux. Consultez la documentation [ici](./how-to-use-codec-compressed-audio-input-streams.md). 

**Résolution des bogues**
- **Tout** : Correction d’un problème qui empêchait KeywordRecognizer de faire avancer les flux après une reconnaissance.
- **Tout** : Correction d’un problème en raison duquel le flux obtenu à partir d’un KeywordRecognitionResult ne contenait pas le mot clé.
- **Tout** : Résolution d’un problème en raison duquel la méthode SendMessageAsync n’envoie pas vraiment le message sur le réseau une fois que les utilisateurs ont fini de l’attendre.
- **Tout** : Correction d’un incident dans les API de reconnaissance de l’orateur lorsque les utilisateurs appellent la méthode VoiceProfileClient::SpeakerRecEnrollProfileAsync plusieurs fois et n’attendent pas la fin des appels.
- **Tout** : Correction de l’activation de la journalisation des fichiers dans les classes VoiceProfileClient et SpeakerRecognizer.
- **JavaScript** : Correction d’un [problème](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/74) de limitation de bande passante lorsque le navigateur est réduit.
- **JavaScript** : Correction d’un [problème](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/78) de fuite de mémoire sur les flux.
- **JavaScript** : Ajout de la mise en cache pour les réponses OCSP à partir de NodeJS.
- **Java** : Correction d’un problème en raison duquel les champs BigInteger retournaient toujours la valeur 0.
- **iOS** : Correction d’un [problème](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/702) avec la publication d’applications basées sur le Kit de développement logiciel (SDK) Speech dans l’App Store iOS.

**Exemples**
- **C++**  : Ajout d’un exemple de code pour Reconnaissance de l’orateur [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/cpp/windows/console/samples/speaker_recognition_samples.cpp).

**Test raccourci COVID-19 :** en raison du travail à distance au cours des dernières semaines, nous n’avons pas pu effectuer autant de tests de vérification manuelle que nous le faisons habituellement. Nous n’avons apporté aucune modification qui aurait pu casser quoi que ce soit, et nos tests automatisés ont tous réussi. Dans le cas peu probable où nous aurions manqué quelque chose, veuillez nous en informer sur [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Restez en bonne santé !

## <a name="text-to-speech-2020-july-release"></a>Synthèse vocale 2020 - version de juillet

### <a name="new-features"></a>Nouvelles fonctionnalités

* **Neural TTS, 15 nouvelles voix neuronales** : Les nouvelles voix ajoutées au portefeuille Neural TTS sont Salma en `ar-EG` arabe (Égypte), Zariyah en `ar-SA` arabe (Arabie saoudite), Alba en `ca-ES` catalan (Espagne), Christel en `da-DK` danois (Danemark), Neerja en `es-IN` anglais (Inde), Noora en `fi-FI` finnois (Finlande), Swara en `hi-IN` hindi (Inde), Colette en `nl-NL` néerlandais (Pays-Bas), Zofia en `pl-PL` polonais (Pologne), Fernanda en `pt-PT` portugais (Portugal), Dariya en `ru-RU` russe (Russie), Hillevi en `sv-SE` suédois (Suède), Achara en `th-TH` thaï (Thaïlande), HiuGaai en `zh-HK` chinois (cantonais, traditionnel) et HsiaoYu en `zh-TW` chinois (mandarin de Taïwan). Vérifier tous les [langages pris en charge](./language-support.md#neural-voices).  

* **Custom Voice, des tests vocaux personnalisés et simplifiés avec le flux d’apprentissage pour simplifier l’expérience utilisateur** : Avec la nouvelle fonctionnalité de test, chaque voix sera automatiquement testée avec un ensemble de tests prédéfini optimisé pour chaque langage afin de couvrir les scénarios de l’assistant général et vocal. Ces ensembles de tests sont soigneusement sélectionnés et testés pour inclure des cas d’usage et des phonèmes typiques dans le langage. En outre, les utilisateurs peuvent toujours choisir de télécharger leurs propres scripts de test lors de l’apprentissage d’un modèle.

* **Création de contenu audio : Un ensemble de nouvelles fonctionnalités est publié pour activer des capacités de gestion audio et de réglage vocal plus puissantes.**

    * `Pitch`, `rate` et `volume` sont améliorés pour prendre en charge le paramétrage avec une valeur prédéfinie, telle que lent, moyen et rapide. Il est désormais simple pour les utilisateurs de choisir une valeur « constante » pour leur édition audio.

    ![Réglage audio](media/release-notes/audio-tuning.png)

    * Les utilisateurs peuvent dorénavant passer en revue les `Audio history` pour leur fichier de travail. Grâce à cette fonctionnalité, les utilisateurs peuvent facilement suivre tous les éléments audio générés associés à un fichier de travail. Ils peuvent vérifier la version de l’historique et comparer la qualité lors du paramétrage. 

    ![Historique audio](media/release-notes/audio-history.png)

    * La fonctionnalité `Clear` est désormais plus flexible. Les utilisateurs peuvent effacer un paramètre de paramétrage spécifique tout en conservant d’autres paramètres disponibles pour le contenu sélectionné.  

    * Un didacticiel a été ajoutée sur la [page d’accueil](https://speech.microsoft.com/audiocontentcreation) pour aider les utilisateurs à prendre rapidement en main le paramétrage de la voix TTS et la gestion audio. 

### <a name="general-tts-voice-quality-improvements"></a>Améliorations générales de la qualité de la voix TTS

* Amélioration du vocodeur TTS pour une plus grande fidélité et une latence moindre.

    * Mise à jour d’Elsa dans `it-IT` à un nouveau vocodeur qui a gagné + 0,464  sur la CMOS (note d’opinion moyenne comparative ) en qualité vocale, 40 % plus rapide en synthèse et réduction de 30 % sur la latence du premier octet. 
    * Mise à jour de Xiaoxiao dans `zh-CN` au nouveau vocodeur avec gain sur la CMOS de + 0148 pour le domaine général, + 0,348 pour le style Newscast et + 0,195 pour le style Lyrique. 

* Mise à jour `de-DE` et `ja-JP` modèles vocaux pour rendre la sortie TTS plus naturelle.
    
    * Mise à jour de Katja dans `de-DE` avec la dernière méthode de modélisation prosodie, le gain MOS (note d’opinion moyenne) est de + 0,13. 
    * Mise à jour de Nanami dans `ja-JP` avec un nouveau modèle d’accentuation prosodie, le gain MOS (note d’opinion moyenne) est de + 0,19 ;  

* Amélioration de la précision de la prononciation au niveau des mots dans cinq langues.

    | Langage | Réduction des erreurs de prononciation |
    |---|---|
    | `en-GB` | 51 % |
    | `ko-KR` | 17 % |
    | `pt-BR` | 39 |
    | `pt-PT` | 77 % |
    | `id-ID` | 46 % |

### <a name="bug-fixes"></a>Résolution des bogues

* Lecture des devises
    * Correction du problème de lecture des devises pour `es-ES` et `es-MX`
     
    | Langage | Entrée | Lecture après amélioration |
    |---|---|---|
    | `es-MX` | 1,58 USD | un peso cincuenta y ocho centavos |
    | `es-ES` | 1,58 USD | un dólar cincuenta y ocho centavos |

    * Prise en charge d’une devise négative (telle que « - 325 € ») dans les paramètres régionaux suivants : `en-US`, `en-GB`, `fr-FR`, `it-IT`, `en-AU`, `en-CA`.

* Amélioration de la lecture des adresses dans `pt-PT`.
* Correction des problèmes de prononciation de Natasha (`en-AU`) et Libby (`en-UK`) sur le mot « for » et « four ».  
* Correction des bogues sur l’outil de création de contenu audio
    * La pause supplémentaire et inattendue après le deuxième paragraphe est corrigée.  
    * La fonctionnalité « aucune interruption » est rajoutée à partir d’un bogue de régression. 
    * Le problème d’actualisation aléatoire de Speech Studio est résolu.  

### <a name="samplessdk"></a>Exemples / SDK

* JavaScript : Correctifs du problème de lecture dans Firefox et Safari sur macOS et iOS. 

## <a name="speech-sdk-1121-2020-june-release"></a>Kit SDK Speech 1.12.1 : version de juin 2020
**Interface CLI Speech (également appelée SPX)**
-   Ajout des fonctionnalités de recherche d’aide dans l’interface CLI :
    -   `spx help find --text TEXT`
    -   `spx help find --topic NAME`
-   Mise à jour pour fonctionner avec les API Batch et Custom Speech v3.0 nouvellement déployées :
    -   `spx help batch examples`
    -   `spx help csr examples`

**Nouvelles fonctionnalités**
-   **C\#, C++**  : Aperçu de reconnaissance de l’orateur : Cette fonctionnalité permet l’identification de l’orateur (qui parle ?) et la vérification de l’orateur (l’orateur est-il la personne qu’il prétend être ?). Commencez par une [vue d’ensemble](./speaker-recognition-overview.md), lisez l’article [Bases de la Reconnaissance de l’orateur](./get-started-speaker-recognition.md) ou la [documentation de référence sur les API](/rest/api/speakerrecognition/).

**Résolution des bogues**
-   **C\#, C++**  : Correction du problème en raison duquel l’enregistrement du microphone ne fonctionnait pas dans la version 1.12 dans la reconnaissance de l’orateur.
-   **JavaScript** : Correctifs pour la synthèse vocale dans Firefox et Safari sur macOS et iOS.
-   Correctif pour l’incident de violation d’accès du vérificateur d’applications Windows lors d’une transcription de conversation, dans le cas de l’utilisation d’un flux de huit canaux.
-   Correction pour l’incident de violation d’accès du vérificateur d’applications Windows lors d’une traduction de conversation entre plusieurs appareils.

**Exemples**
-   **C#**  : [Exemple de code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/speaker-recognition) pour la reconnaissance de l’orateur.
-   **C++**  : [Exemple de code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/speaker-recognition) pour la reconnaissance de l’orateur.
-   **Java** : [Exemple de code](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/intent-recognition) pour la reconnaissance de l’intention sur Android. 

**Test raccourci COVID-19 :** en raison du travail à distance au cours des dernières semaines, nous n’avons pas pu effectuer autant de tests de vérification manuelle que nous le faisons habituellement. Nous n’avons apporté aucune modification qui aurait pu casser quoi que ce soit, et nos tests automatisés ont tous réussi. Dans le cas peu probable où nous aurions manqué quelque chose, veuillez nous en informer sur [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Restez en bonne santé !


## <a name="speech-sdk-1120-2020-may-release"></a>Kit de développement logiciel (SDK) Speech 1.12.0 : version de mai 2020
**Interface CLI Speech (également appelée SPX)**
- **SPX** est un nouvel outil en ligne de commande qui vous permet d’effectuer des tâches de reconnaissance, de synthèse, de traduction, de transcription par lot et de gestion vocale personnalisée à partir de la ligne de commande. Utilisez-le pour tester le service Speech ou générer un script pour les tâches du service Speech que vous devez accomplir. Téléchargez l’outil et lisez la documentation [ici](./spx-overview.md).

**Nouvelles fonctionnalités**
- **Go** : nouvelle prise en charge du langage Go pour la [reconnaissance vocale](./get-started-speech-to-text.md?pivots=programming-language-go) et l’[Assistant vocal personnalisé](./quickstarts/voice-assistants.md?pivots=programming-language-go). Configurez votre environnement de développement [ici](./quickstarts/setup-platform.md?pivots=programming-language-go). Pour un exemple de code, consultez la section Exemples ci-dessous. 
- **JavaScript** : ajout de la prise en charge du navigateur pour la synthèse vocale. Consultez la documentation [ici](./get-started-text-to-speech.md?pivots=programming-language-JavaScript).
- **C++, C#, Java** : Nouvel objet `KeywordRecognizer` et nouvelles API prises en charge sur les plateformes Windows, Android, Linux et iOS. Lisez la documentation [ici](./custom-keyword-overview.md). Pour un exemple de code, consultez la section Exemples ci-dessous. 
- **Java** : ajout de la conversation sur plusieurs appareils avec prise en charge de la traduction. Consultez le document de référence [ici](/java/api/com.microsoft.cognitiveservices.speech.transcription).

**Améliorations et optimisations**
- **JavaScript** : implémentation optimisée du microphone du navigateur améliorant la précision de la reconnaissance vocale.
- **Java** : liaisons refactorisées à l’aide de l’implémentation de JNI directe sans SWIG. Cette modification réduit de 10 fois la taille des liaisons pour tous les packages Java utilisés pour Windows, Android, Linux et Mac, et facilite le développement de l’implémentation Java du Kit de développement logiciel (SDK) Speech.
- **Linux** : mise à jour de la [documentation](./speech-sdk.md?tabs=linux) de support avec les dernières remarques spécifiques de RHEL 7.
- Amélioration de la logique de connexion pour effecteur plusieurs tentatives de connexion lorsque des erreurs de service et de réseau se produisent.
- Mise à jour de la page de démarrage rapide de Speech [portal.azure.com](https://portal.azure.com) pour aider les développeurs à passer à l’étape suivante dans le parcours Azure Speech.

**Résolution des bogues**
- **C#, Java** : correction d’un [problème](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/587) de chargement des bibliothèques de Kit de développement logiciel (SDK) sur Linux ARM (32 bits et 64 bits).
- **C#**  : correction de la suppression explicite des descripteurs natifs pour les objets TranslationRecognizer, IntentRecognizer et Connection.
- **C#**  : correction de la gestion de la durée de vie d’entrée audio pour l’objet ConversationTranscriber.
- Correction d’un problème qui avait pour effet que la raison du résultat de `IntentRecognizer` n’était pas définie correctement lors de la reconnaissance d’intentions à partir d’expressions simples.
- Correction d’un problème qui avait pour effet que le décalage de résultat `SpeechRecognitionEventArgs` n’était pas défini correctement.
- Correction d’une condition de concurrence qui avait pour effet que le Kit de développement logiciel (SDK) essayait d’envoyer un message réseau avant d’ouvrir la connexion websocket. Était reproductible pour `TranslationRecognizer` lors de l’ajout de participants.
- Correction de fuites de mémoire dans le moteur de reconnaissance de mot clé.

**Exemples**
- **Go** : ajout de démarrages rapides pour la [reconnaissance vocale](./get-started-speech-to-text.md?pivots=programming-language-go) et [Assistant vocal personnalisé](./quickstarts/voice-assistants.md?pivots=programming-language-go). Trouvez un exemple de code [ici](https://github.com/microsoft/cognitive-services-speech-sdk-go/tree/master/samples). 
- **JavaScript** : ajout de démarrages rapides pour la [Synthèse vocale](./get-started-text-to-speech.md?pivots=programming-language-javascript), la [Traduction](./get-started-speech-translation.md?pivots=programming-language-csharp&tabs=script) et la [Reconnaissance de l’intention](./quickstarts/intent-recognition.md?pivots=programming-language-javascript).
- Exemples de reconnaissance de mot clé pour [C\#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/uwp/keyword-recognizer) et [Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/keyword-recognizer) (Android).  

**Test raccourci COVID-19 :** en raison du travail à distance au cours des dernières semaines, nous n’avons pas pu effectuer autant de tests de vérification manuelle que nous le faisons habituellement. Nous n’avons apporté aucune modification qui aurait pu casser quoi que ce soit, et nos tests automatisés ont tous réussi. Au cas où nous aurions manqué quelque chose, veuillez nous en informer sur [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Restez en bonne santé !

## <a name="speech-sdk-1110-2020-march-release"></a>Kit de développement logiciel (SDK) Speech 1.11.0 : version de mars 2020
**Nouvelles fonctionnalités**
- Linux : Ajout de la prise en charge de Red Hat Enterprise Linux (RHEL)/CentOS 7 x64 avec [instructions](./how-to-configure-rhel-centos-7.md) sur la configuration du système pour le Kit de développement logiciel (SDK) Speech.
- Linux : Ajout de la prise en charge de .NET Core C# sur Linux ARM32 et ARM64. En savoir plus [ici](./speech-sdk.md?tabs=linux). 
- C#, C++ : Ajout de `UtteranceId` dans `ConversationTranscriptionResult`, un ID cohérent pour tous les intermédiaires et le résultat final de la reconnaissance vocale. Détails pour [C#](/dotnet/api/microsoft.cognitiveservices.speech.transcription.conversationtranscriptionresult?preserve-view=true&view=azure-dotnet) et [C++](/cpp/cognitive-services/speech/transcription-conversationtranscriptionresult).
- Python : Ajout de la prise en charge de `Language ID`. Voir speech_sample.py dans le [référentiel GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/python/console).
- Windows : Ajout de la prise en charge du format d’entrée audio compressé sur la plateforme Windows pour toutes les applications console win32. Détails [ici](./how-to-use-codec-compressed-audio-input-streams.md). 
- JavaScript : Prise en charge de la synthèse vocale (conversion de texte par synthèse vocale) dans NodeJS. En savoir plus [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/text-to-speech). 
- JavaScript : Ajout de nouvelles API pour activer l’inspection de tous les messages envoyés et reçus. En savoir plus [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript). 
        
**Résolution des bogues**
- C#, C++ : Correction d’un problème de sorte que `SendMessageAsync` envoie maintenant un message binaire sous forme de type binaire. Détails pour [C#](/dotnet/api/microsoft.cognitiveservices.speech.connection.sendmessageasync?view=azure-dotnet#Microsoft_CognitiveServices_Speech_Connection_SendMessageAsync_System_String_System_Byte___System_UInt32_) et [C++](/cpp/cognitive-services/speech/connection).
- C#, C++ : Correction d’un problème où l’utilisation de l’événement `Connection MessageReceived` peut provoquer un incident si `Recognizer` est supprimé avant l’objet `Connection`. Détails pour [C#](/dotnet/api/microsoft.cognitiveservices.speech.connection.messagereceived?preserve-view=true&view=azure-dotnet) et [C++](/cpp/cognitive-services/speech/connection#messagereceived).
- Android : La taille de la mémoire tampon audio du microphone a été réduite de 800 ms à 100 ms pour améliorer la latence.
- Android : Correction d’un [problème](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/563) avec l’émulateur Android x86 dans Android Studio.
- JavaScript : Ajout de la prise en charge des régions en Chine avec l’API `fromSubscription`. Détails [ici](/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?preserve-view=true&view=azure-node-latest#fromsubscription-string--string-). 
- JavaScript : Ajout d’autres informations d’erreur pour les échecs de connexion à partir de NodeJS.
        
**Exemples**
- Unity : L’échantillon public de reconnaissance des intentions est corrigé, là où l’importation du fichier json LUIS échouait. Détails [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/369).
- Python : Exemple ajouté pour `Language ID`. Détails [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py).
    
**Tests abrégés en raison du Covid19 :** En raison du travail à distance au cours des dernières semaines, nous n’avons pas pu effectuer autant de tests manuels de vérification des appareils que nous le faisons habituellement. Par exemple, nous n’avons pas pu tester l’entrée du microphone et de la sortie du haut-parleur sous Linux, iOS et macOS. Nous n’avons apporté aucune modification qui aurait pu casser quoi que ce soit sur ces plateformes, et nos tests automatisés ont tous réussi. Dans le cas peu probable où nous aurions manqué quelque chose, veuillez nous en informer sur [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br> Nous vous remercions de votre soutien continu. Comme toujours, publiez vos questions ou vos commentaires sur [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen) ou [Stack Overflow](https://stackoverflow.microsoft.com/questions/tagged/731).<br>
Restez en bonne santé !

## <a name="speech-sdk-1100-2020-february-release"></a>Kit de développement logiciel (SDK) de Speech 1.10.0 : version de février 2020

**Nouvelles fonctionnalités**

 - Ajout de packages Python pour prendre en charge la nouvelle version 3.8 de Python.
 - Prise en charge de Red Hat Enterprise Linux (RHEL)/CentOS 8 x64 (C++, C#, Java, Python).
   > [!NOTE] 
   > Les clients doivent configurer OpenSSL conformément à [ces instructions](./how-to-configure-openssl-linux.md).
 - Prise en charge de Linux ARM32 pour Debian et Ubuntu.
 - DialogServiceConnector prend désormais en charge un paramètre facultatif « bot ID » sur BotFrameworkConfig. Ce paramètre permet d’utiliser plusieurs robots Direct Line Speech avec une seule ressource Azure Speech. Si le paramètre n’est pas spécifié, le robot par défaut (tel que déterminé par la page de configuration du canal Direct Line Speech) sera utilisé.
 - DialogServiceConnector a maintenant une propriété SpeechActivityTemplate. Le contenu de cette chaîne JSON sera utilisé par Direct Line Speech pour préremplir un grand nombre de champs pris en charge dans toutes les activités qui atteignent un robot Direct Line Speech, y compris les activités générées automatiquement en réponse à des événements tels que la reconnaissance vocale.
 - TTS utilise désormais la clé d’abonnement pour l’authentification, ce qui réduit la latence du premier octet du premier résultat de synthèse après la création d’un synthétiseur.
 - Mise à jour des modèles de reconnaissance vocale pour 19 paramètres régionaux pour une réduction du taux d’erreur moyen des mots de 18,6% (es-ES, es-MX, fr-CA, fr-FR, it-IT, ja-JP, ko-KR, pt-BR, zh-CN, zh-HK, nb-NO, fi-FL, ru-RU, pl-PL, ca-ES, zh-TW, th-TH, pt-PT, tr-TR). Les nouveaux modèles apportent des améliorations significatives dans plusieurs domaines, notamment les scénarios de dictée, de transcription des centres d’appels et d’indexation vidéo.

**Résolution des bogues**

 - Correction du bogue dans lequel la transcription de conversation n’a pas attendu correctement dans les API JAVA 
 - Correctif de l’émulateur Android x86 pour le [problème Xamarin GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/363)
 - Ajouter les méthodes (Get|Set)Property manquantes à AudioConfig
 - Correction d’un bogue TTS dans lequel audioDataStream could ne pouvait pas être arrêté lors de l’échec de la connexion
 - L’utilisation d’un point de terminaison sans région entraînerait des défaillances de l’USP pour le traducteur de conversations
 - La génération d’ID dans les applications Windows universelles utilise désormais un algorithme GUID approprié ; elle utilisait précédemment et involontairement par défaut une implémentation faisant l’objet d’un stub qui produisait souvent des collisions sur de grands ensembles d’interactions.
 
 **Exemples**
 
 - Exemple Unity pour l’utilisation du Kit de développement logiciel (SDK) Speech avec [microphone Unity et diffusion en continu en mode push](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/unity/from-unitymicrophone)

**Autres modifications**

 - [Mise à jour de la documentation de configuration OpenSSL pour Linux](./how-to-configure-openssl-linux.md)

## <a name="speech-sdk-190-2020-january-release"></a>SDK Speech 1.9.0 : version de janvier 2020

**Nouvelles fonctionnalités**

- Conversation multi-appareil : connectez plusieurs appareils à la même conversation vocale ou textuelle et traduisez éventuellement les messages échangés. Apprenez-en plus dans [cet article](multi-device-conversation.md). 
- Ajout de la prise en charge de la reconnaissance des mots clés pour le package .aar Android et ajout de la prise en charge des versions x86 et x64. 
- Objective-C : ajout des méthodes `SendMessage` et `SetMessageProperty` à l'objet `Connection`. Consultez la documentation [ici](/objectivec/cognitive-services/speech/spxconnection).
- L'API TTS C++ prend désormais en charge `std::wstring` comme entrée de texte de synthèse, ce qui évite d'avoir à convertir un wstring en string avant de la passer au SDK. Consultez les informations détaillées [ici](/cpp/cognitive-services/speech/speechsynthesizer#speaktextasync). 
- C# : l'[ID de langue](./how-to-automatic-language-detection.md?pivots=programming-language-csharp) et la [configuration de la langue source](./how-to-specify-source-language.md?pivots=programming-language-csharp) sont désormais disponibles.
- JavaScript : ajout d'une fonctionnalité à l'objet `Connection` permettant de transférer les messages personnalisés depuis le service Speech en tant que rappel `receivedServiceMessage`.
- JavaScript : ajout de la prise en charge de `FromHost API` pour faciliter l'utilisation avec les conteneurs locaux et les clouds souverains. Consultez la documentation [ici](speech-container-howto.md).
- JavaScript : nous prenons désormais en considération `NODE_TLS_REJECT_UNAUTHORIZED` grâce à une contribution d'[orgads](https://github.com/orgads). Consultez les informations détaillées [ici](https://github.com/microsoft/cognitive-services-speech-sdk-js/pull/75).

**Dernières modifications**

- `OpenSSL` a été mis à jour vers la version 1.1.1b et est lié de manière statique à la bibliothèque principale du kit SDK Speech pour Linux. Cela peut provoquer un arrêt si votre boîte de réception `OpenSSL` n'a pas été installée dans le répertoire `/usr/lib/ssl` du système. Pour contourner ce problème, consultez [notre documentation](how-to-configure-openssl-linux.md) sous les documents du kit SDK Speech.
- Concernant le type de données renvoyées pour C# `WordLevelTimingResult.Offset`, nous avons remplacé `int` par `long` pour permettre l'accès à `WordLevelTimingResults` lorsque les données vocales dépassent les deux minutes.
- `PushAudioInputStream` et `PullAudioInputStream` envoient désormais des informations d'en-tête wav au service Speech en fonction de `AudioStreamFormat`, éventuellement spécifiés durant leur création. Les clients doivent maintenant utiliser le [format d'entrée audio pris en charge](how-to-use-audio-input-streams.md). Tous les autres formats donneront des résultats de reconnaissance non optimaux ou risquent de causer d’autres problèmes. 

**Résolution des bogues**

- Consultez les informations concernant la mise à jour de `OpenSSL` dans la section Changements cassants ci-dessus. Nous avons résolu un incident intermittent et un problème de performances (contention de verrouillage en condition de charge élevée) dans Linux et Java. 
- Java : améliorations apportées à la fermeture des objets dans les scénarios de forte concurrence.
- Restructuration de notre package NuGet. Nous avons supprimé les trois copies de `Microsoft.CognitiveServices.Speech.core.dll` et `Microsoft.CognitiveServices.Speech.extension.kws.dll` sous les dossiers lib. Plus petit, le package NuGet se télécharge plus rapidement, et nous avons ajouté les en-têtes nécessaires à la compilation de certaines applications natives C++.
- Vous trouverez [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp) des exemples de démarrage rapide corrigés. Ceux-ci se fermaient sans afficher l’exception « microphone introuvable » sous Linux, macOS, Windows.
- Correction de l'incident du kit SDK avec les résultats de reconnaissance vocale de longue durée sur certains chemins de code comme [cet exemple](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/csharp/uwp/speechtotext-uwp).
- Correction de l'erreur de déploiement du kit SDK dans l'environnement d'application web Azure pour résoudre [le problème de ce client](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/396).
- Correction d'une erreur TTS pendant l'utilisation de plusieurs balises `<voice>` ou `<audio>` pour résoudre [le problème de ce client](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/433). 
- Correction d'une erreur TTS 401 qui se déclarait pendant la récupération du kit SDK à la suite d'une interruption.
- JavaScript : correction d'une importation circulaire de données audio grâce à une contribution d'[Euirim](https://github.com/euirim). 
- JavaScript : ajout de la prise en charge de la définition de propriétés de service (ajoutée dans la version 1.7).
- JavaScript : résolution d'un problème où une erreur de connexion pouvait entraîner des tentatives de reconnexion WebSocket continues et infructueuses.

**Exemples**

- Ajout de l'exemple de reconnaissance de mot clé pour Android [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/java/android/sdkdemo).
- Ajout de l'exemple TTS pour le scénario de serveur [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_synthesis_server_scenario_sample.cs).
- Ajout de démarrages rapides pour les conversations multi-appareils pour C# et C++ [ici](quickstarts/multi-device-conversation.md).

**Autres modifications**

- Taille de la bibliothèque principale du kit SDK optimisée sur Android.
- Le kit SDK de la version 1.9.0 et ultérieures prend en charge les types `int` et `string` dans le champ de version de la signature vocale du transcripteur de conversation.

## <a name="speech-sdk-180-2019-november-release"></a>SDK Speech 1.8.0 : Version de novembre 2019

**Nouvelles fonctionnalités**

- Ajout d'une API `FromHost()`, pour faciliter l'utilisation avec des conteneurs locaux et des clouds souverains.
- Ajout de la détection de langue source automatique pour la reconnaissance vocale (en Java et C++)
- Ajout de l'objet `SourceLanguageConfig` pour la reconnaissance vocale, utilisé pour spécifier les langues sources attendues (en Java et C++)
- Ajout de la prise en charge de `KeywordRecognizer` sur Windows (UWP), Android et iOS via les packages NuGet et Unity
- Ajout de l'API Java de conversation à distance pour effectuer une transcription de conversation dans des lots asynchrones.

**Dernières modifications**

- Les fonctionnalités du transcripteur de conversation sont déplacées sous l'espace de noms `Microsoft.CognitiveServices.Speech.Transcription`.
- Certaines parties des méthodes du transcripteur de conversation sont déplacées vers la nouvelle classe `Conversation`.
- Suppression de la prise en charge d'iOS 32 bits (ARMv7 et x86)

**Résolution des bogues**

- Correction de l'incident si le `KeywordRecognizer` local est utilisé sans clé d'abonnement valide au service Speech

**Exemples**

- Exemple Xamarin pour `KeywordRecognizer`
- Exemple Unity pour `KeywordRecognizer`
- Exemples C++ et Java pour la détection automatique de la langue source.

## <a name="speech-sdk-170-2019-september-release"></a>SDK Speech 1.7.0 : version de septembre 2019

**Nouvelles fonctionnalités**

- Ajout de la prise en charge de la version bêta pour Xamarin sur la plateforme Windows universelle (UWP), Android et iOS
- Ajout de la prise en charge d'iOS pour Unity
- Ajout de la prise en charge des entrées `Compressed` pour ALaw, Mulaw, FLAC sous Android, iOS et Linux
- Ajout de `SendMessageAsync` dans la classe `Connection` pour l'envoi d'un message au service
- Ajout de `SetMessageProperty` dans la classe `Connection` pour la propriété de définition d'un message
- Ajout de liaisons TTS pour Java (JRE et Android), Python, Swift et Objective-C
- Ajout de la prise en charge de la lecture TTS pour macOS, iOS et Android
- Ajout d'informations sur la « limite de mot » pour TTS

**Résolution des bogues**

- Correction d'un problème de build IL2CPP sur Unity 2019 pour Android
- Correction d'un problème lié à des en-têtes incorrects entraînant le mauvais traitement de l'entrée d'un fichier WAV
- Correction d'un problème lié à la multiplicité des UUID dans certaines propriétés de connexion
- Correction de quelques avertissements sur les spécificateurs de possibilité de valeur null dans les liaisons Swift (ce qui peut nécessiter de petites modifications de code)
- Correction d'un bogue provoquant la fermeture anormale des connexions WebSocket sous une charge réseau
- Correction d'un problème sur Android pouvant entraîner des ID d'impression en double utilisés par `DialogServiceConnector`
- Améliorations apportées à la stabilité des connexions entre les interactions multitours et au signalement des défaillances (par le biais d’événements `Canceled`) quand elles se produisent avec `DialogServiceConnector`
- Les démarrages de session `DialogServiceConnector` permettent désormais de fournir correctement les événements, y compris lors de l’appel de `ListenOnceAsync()` pendant une activité `StartKeywordRecognitionAsync()`
- Résolution d’un incident associé aux activités `DialogServiceConnector` reçues

**Exemples**

- Guide de démarrage rapide pour Xamarin
- Mise à jour du guide de démarrage rapide CPP avec des informations ARM64 Linux
- Mise à jour du guide de démarrage rapide Unity avec des informations iOS

## <a name="speech-sdk-160-2019-june-release"></a>Kit de développement logiciel (SDK) Speech 1.6.0 : version de juin 2019

**Exemples**

- Exemples de guide de démarrage rapide pour Synthèse vocale sur UWP et Unity
- Exemple de guide de démarrage rapide pour Swift sur iOS
- Exemples Unity supplémentaires pour Speech ainsi que la reconnaissance de l’intention et la traduction
- Exemples de démarrage rapide pour `DialogServiceConnector`

**Améliorations/Modifications**

- Espace de noms de boîte de dialogue :
  - `SpeechBotConnector` a été renommé en `DialogServiceConnector`
  - `BotConfig` a été renommé en `DialogServiceConfig`
  - `BotConfig::FromChannelSecret()` a été remappé à `DialogServiceConfig::FromBotSecret()`
  - Tous les clients existants de Direct Line Speech seront toujours pris en charge après le changement de nom
- Mise à jour de l’adaptateur TTS REST offrant la prise en charge des proxys et de la connexion permanente
- Amélioration du message d’erreur en cas de transmission d’une région non valide
- Swift/Objective-C :
  - Amélioration du signalement des erreurs : Les méthodes susceptibles de provoquer une erreur existent désormais en deux versions : La première expose un objet `NSError` pour la gestion des erreurs, l’autre émet une exception. La première est exposée à Swift. Cette modification doit être adaptée au code Swift existant.
  - Amélioration de la gestion des événements

**Résolution des bogues**

- Correctif pour TTS : où le futur `SpeakTextAsync` est retourné sans attendre que le rendu audio soit terminé
- Correctif pour le marshaling des chaînes dans C# pour activer la prise en charge complète des langages
- Correction d’un problème d’application .NET Core relatif au chargement de la bibliothèque principale avec la version cible de .Net Framework net461 dans les exemples
- Correction de problèmes occasionnels pour déployer des bibliothèques natives dans le dossier de sortie dans les exemples
- Correctif pour fermer le socket web de façon fiable
- Correction des blocages éventuels lors de l’ouverture d’une connexion avec des charges lourdes sur Linux
- Correction des métadonnées manquantes dans le bundle de framework pour macOS
- Correctif pour les problèmes de `pip install --user` sur Windows

## <a name="speech-sdk-151"></a>Kit de développement logiciel (SDK) de reconnaissance vocale 1.5.1

Il s’agit d’une version de correctif de bogue et affecte uniquement le Kit de développement logiciel (SDK) natif/managé. Il n’affecte pas la version JavaScript du SDK.

**Résolution des bogues**

- Correction de FromSubscription si utilisé avec la transcription de conversation.
- Correction d’un bogue dans la détection de mot clé des assistants vocaux.

## <a name="speech-sdk-150-2019-may-release"></a>Kit de développement logiciel (SDK) Speech 1.5.0 : version de mai 2019

**Nouvelles fonctionnalités**

- La détection de mot clé (KWS) est désormais disponible sur Windows et Linux. La fonctionnalité KWS peut fonctionner avec n'importe quel type de microphone, mais la prise en charge officielle de KWS est actuellement limitée aux réseaux de microphones présents dans le matériel Azure Kinect DK ou dans le Kit de développement logiciel (SDK) Speech Devices.
- La fonctionnalité d’indication de phrase est disponible via le Kit de développement logiciel (SDK). Vous pourrez trouver plus d’informations [ici](./get-started-speech-to-text.md).
- La fonctionnalité de transcription de conversation est disponible via le Kit de développement logiciel (SDK). Voir [ici](./conversation-transcription.md).
- Ajoutez la prise en charge des assistants vocaux en utilisant le canal Direct Line Speech.

**Exemples**

- Ajout d’exemples pour les nouvelles fonctionnalités ou les nouveaux services pris en charge par le Kit de développement logiciel (SDK).

**Améliorations/Modifications**

- Ajout de plusieurs propriétés au module de reconnaissance pour ajuster le comportement du service ou les résultats du service (par exemple, le masquage des termes grossiers, entre autres).
- Vous pouvez maintenant configurer le module de reconnaissance via les propriétés de configuration standard, même si vous avez créé le module de reconnaissance `FromEndpoint`.
- Objective-C : la propriété `OutputFormat` a été ajoutée à `SPXSpeechConfiguration`.
- Le Kit de développement logiciel (SDK) prend désormais en charge la distribution Linux Debian 9.

**Résolution des bogues**

- Correction d’un problème dans lequel la ressource de l’intervenant était détruite trop tôt dans la synthèse vocale.

## <a name="speech-sdk-142"></a>Kit de développement logiciel (SDK) Speech 1.4.2

Il s’agit d’une version de correctif de bogue et affecte uniquement le Kit de développement logiciel (SDK) natif/managé. Il n’affecte pas la version JavaScript du SDK.

## <a name="speech-sdk-141"></a>Kit de développement logiciel (SDK) Speech 1.4.1

Il s’agit d’une version JavaScript uniquement. Aucune fonctionnalité n’a été ajoutée. Les correctifs suivants ont été appliqués :

- Empêcher le pack web de charger https-proxy-agent.

## <a name="speech-sdk-140-2019-april-release"></a>Kit de développement logiciel (SDK) Speech 1.4.0 : version d’avril 2019

**Nouvelles fonctionnalités**

- Le SDK prend désormais en charge le service de synthèse vocale en version bêta. Il est pris en charge sur Windows et Linux Desktop à partir de C++ et C#. Pour plus d’informations, consultez la [vue d’ensemble de la synthèse vocale](text-to-speech.md#get-started).
- Le SDK prend désormais en charge les formats audio MP3 et Opus/Ogg comme fichiers d’entrée de flux. Cette fonctionnalité est uniquement disponible sur Linux à partir de C++ et C#, et est actuellement en version bêta (plus de détails [ici](how-to-use-codec-compressed-audio-input-streams.md)).
- Le SDK Speech pour Java, .NET Core, C++ et Objective-C prend désormais en charge macOS. La prise en charge d'Objective-C pour macOS est actuellement en version bêta.
- iOS : Le SDK Speech pour iOS (Objective-C) est désormais également publié en tant que CocoaPod.
- JavaScript : Prise en charge d’un microphone non défini par défaut comme périphérique d’entrée.
- JavaScript : Prise en charge de proxy pour Node.js.

**Exemples**

- Des exemples d’utilisation du SDK Speech avec C++ et Objective-C sur macOS ont été ajoutés.
- Des exemples montrant l’utilisation du service de synthèse vocale ont été ajoutés.

**Améliorations/Modifications**

- Python : Les propriétés supplémentaires des résultats de reconnaissance sont désormais exposées via la propriété `properties`.
- Pour une prise en charge supplémentaire du développement et du débogage, vous pouvez rediriger les informations de journalisation et de diagnostics du SDK vers un fichier journal (plus de détails [ici](how-to-use-logging.md)).
- JavaScript : Améliorer les performances de traitement audio.

**Résolution des bogues**

- Mac/iOS : Correction d’un bogue qui entraînait une longue attente lorsqu’une connexion au service Speech ne pouvait pas être établie.
- Python : améliorer la gestion des erreurs pour les arguments dans les rappels Python.
- JavaScript : Correction d’un rapport d’état incorrect pour une reconnaissance vocale terminée sur RequestSession.

## <a name="speech-sdk-131-2019-february-refresh"></a>Kit de développement logiciel (SDK) Speech 1.3.1 : Actualisation de février 2019

Il s’agit d’une version de correctif de bogue et affecte uniquement le Kit de développement logiciel (SDK) natif/managé. Il n’affecte pas la version JavaScript du SDK.

**Résolution de bogue**

- Correction d’une fuite de mémoire lors de l’utilisation d’une entrée de microphone. N’affecte pas les entrées basées sur un flux ou un fichier.

## <a name="speech-sdk-130-2019-february-release"></a>Kit de développement logiciel (SDK) de reconnaissance vocale 1.3.0 : Version de février 2019

**Nouvelles fonctionnalités**

- Le SDK Speech prend en charge la sélection du microphone d’entrée via la classe `AudioConfig`. Cela vous permet de diffuser en streaming des données audio vers le service Speech à partir d’un microphone qui n’est pas le microphone par défaut. Pour plus d’informations, consultez la documentation décrivant la [sélection du périphérique d’entrée audio](how-to-select-audio-input-devices.md). JavaScript ne propose pas encore cette fonctionnalité.
- Le SDK Speech prend désormais en charge Unity dans une version bêta. Envoyez des commentaires via la section problèmes dans le [dépôt d’exemples GitHub](https://aka.ms/csspeech/samples). Cette version prend en charge Unity sur Windows x86 et x64 (applications de bureau autonome ou plateforme Windows universelle) et Android (ARM32/64, x86). Des informations supplémentaires sont disponibles dans notre [Démarrage rapide Unity](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=unity).
- Le fichier `Microsoft.CognitiveServices.Speech.csharp.bindings.dll` (fourni dans les versions précédentes) n’est plus nécessaire. La fonctionnalité est désormais intégrée au kit SDK principal.

**Exemples**

Le nouveau contenu suivant est disponible dans notre [dépôt d’exemples](https://aka.ms/csspeech/samples) :

- Exemples supplémentaires pour `AudioConfig.FromMicrophoneInput`.
- Exemples Python supplémentaires pour la reconnaissance de l’intention et la traduction.
- Exemples supplémentaires pour l’utilisation de l’objet `Connection` dans iOS.
- Exemples Java supplémentaires pour la traduction avec une sortie audio.
- Nouvel exemple pour l’utilisation de l’[API REST de transcription Batch](batch-transcription.md).

**Améliorations/Modifications**

- Python
  - Vérification de paramètres améliorée et messages d’erreur dans `SpeechConfig`.
  - Ajoutez une prise en charge de l’objet `Connection`.
  - Prise en charge de Python 32 bits (x86) sur Windows.
  - Le SDK Speech pour Python n’est plus en version bêta.
- iOS
  - Le SDK est désormais basé sur le SDK iOS version 12.1.
  - Le SDK prend désormais en charge iOS 9.2 et versions ultérieures.
  - Améliorer la documentation de référence et corriger plusieurs noms de propriété.
- JavaScript
  - Ajoutez une prise en charge de l’objet `Connection`.
  - Ajouter des fichiers de définition de type pour JavaScript en offre groupée
  - Prise en charge initiale et implémentation des conseils.
  - Retourne la collection de propriétés avec le service JSON dans le cadre de la reconnaissance
- Les DLL Windows contiennent à présent une vraie ressource de version.
- Si vous créez un module de reconnaissance `FromEndpoint`, vous pouvez ajouter des paramètres directement à l’URL du point de terminaison. `FromEndpoint` ne vous permet pas de configurer le module de reconnaissance via les propriétés de configuration standard.

**Résolution des bogues**

- Le nom d’utilisateur et le mot de passe du proxy vides n’étaient pas été traités correctement. Avec cette version, si vous définissez un nom d’utilisateur et un mot de passe proxy en tant que chaîne vide, ils ne seront pas soumis lors de la connexion au proxy.
- Les SessionId créés par le SDK n’étaient pas toujours vraiment aléatoires pour certains langages&nbsp;/ environnements. L’ajout de l’initialisation du générateur aléatoire a permis de corriger ce problème.
- Amélioration de la gestion du jeton d’autorisation. Si vous souhaitez utiliser un jeton d’autorisation, spécifiez-le dans le `SpeechConfig` et ne renseignez pas la clé d’abonnement. Créez ensuite le module de reconnaissance comme d’habitude.
- Dans certains cas, la connexion de l’objet `Connection` n’a pas été libérée correctement. Ce problème est à présent résolu.
- L’exemple JavaScript a été corrigé de façon à prendre en charge la sortie audio pour la synthèse de traduction également dans Safari.

## <a name="speech-sdk-121"></a>Kit de développement logiciel (SDK) Speech 1.2.1

Il s’agit d’une version JavaScript uniquement. Aucune fonctionnalité n’a été ajoutée. Les correctifs suivants ont été appliqués :

- Déclenchement du fin de flux au niveau de turn.end, et non pas de speech.end.
- Correction d’un bogue dans la pompe audio qui ne planifiait pas le prochain envoi si l’envoi en cours échouait.
- Correction de la reconnaissance continue avec le jeton d’authentification.
- Correction de bogue pour un module de reconnaissance / des points de terminaison différents.
- Améliorations de la documentation.

## <a name="speech-sdk-120-2018-december-release"></a>SDK Speech 1.2.0 : Version de décembre 2018

**Nouvelles fonctionnalités**

- Python
  - La version bêta de la prise en charge de Python (3.5 et au-delà) est disponible avec cette version. Vous pourrez trouver plus d’informations ici (quickstart-python.md).
- JavaScript
  - Le SDK Speech pour JavaScript est open source. Le code source est disponible sur [GitHub](https://github.com/Microsoft/cognitive-services-speech-sdk-js).
  - Nous prenons désormais en charge Node.js. Pour plus d’informations, [consultez cette page](./get-started-speech-to-text.md).
  - La restriction sur la longueur des sessions audio a été supprimée. La reconnexion se produit automatiquement à l’arrière-plan.
- l'objet `Connection`
  - À partir de la `Recognizer`, vous pouvez accéder à un objet `Connection`. Cet objet vous permet de lancer explicitement la connexion au service et de vous abonner à des événements de connexion et de déconnexion.
    (JavaScript et Python ne proposent pas encore cette fonctionnalité.)
- Prise en charge d’Ubuntu 18.04.
- Android
  - Prise en charge de ProGuard activée durant la génération d’APK.

**Améliorations**

- Améliorations apportées à l’utilisation des threads internes afin de réduire le nombre de threads, de verrous et de mutex.
- Amélioration des rapports d’erreurs et des informations sur les erreurs. Dans plusieurs cas, des messages d’erreur n’ont pas été entièrement propagés.
- Mise à jour des dépendances de développement dans JavaScript pour utiliser des modules à jour.

**Résolution des bogues**

- Résolution des fuites de mémoire causés par une incompatibilité de type dans `RecognizeAsync`.
- Dans certains cas, des exceptions étaient divulguées.
- Résolution des fuites de mémoire dans les arguments d’événement de traduction.
- Résolution d’un problème de verrouillage à la suite d’une reconnexion dans les sessions de longue durée.
- Résolution d’un problème pouvant entraîner l’absence d’un résultat final en cas d’échec de la traduction.
- C# : Si une opération `async` n’était pas attendue dans le thread principal, le module de reconnaissance pouvait être supprimé avant la fin de la tâche asynchrone.
- Java : Résolution d’un problème entraînant un blocage de la machine virtuelle Java.
- Objective-C : Résolution d’un mappage d’enum ; RecognizedIntent était retourné à la place de `RecognizingIntent`.
- JavaScript : Définition du format de sortie par défaut « simple » dans `SpeechConfig`.
- JavaScript : Suppression d’une incohérence au niveau des propriétés sur l’objet config entre JavaScript et d’autres langages.

**Exemples**

- Mise à jour et résolution de plusieurs exemples (par exemple, voix de sortie pour la traduction, etc.).
- Ajout d’exemples Node.js dans le [dépôt d’exemples](https://aka.ms/csspeech/samples).

## <a name="speech-sdk-110"></a>SDK Speech 1.1.0

**Nouvelles fonctionnalités**

- Prise en charge d'Android x86/x64.
- Prise en charge de proxy : dans l’objet `SpeechConfig`, vous pouvez maintenant appeler une fonction pour définir les informations de proxy (nom d’hôte, port, nom d’utilisateur et mot de passe). Cette fonctionnalité n'est pas encore disponible sous iOS.
- Amélioration du code d’erreur et des messages. Si une reconnaissance a renvoyé une erreur, celle-ci a déjà défini `Reason` (dans l’événement annulé) ou `CancellationDetails` (dans le résultat de la reconnaissance) sur `Error`. L’événement annulé contient maintenant deux membres supplémentaires, `ErrorCode` et `ErrorDetails`. Si le serveur a renvoyé des informations d’erreur supplémentaires avec l’erreur signalée, elles sont désormais disponibles dans les nouveaux membres.

**Améliorations**

- Ajout d'une vérification supplémentaire dans la configuration du module de reconnaissance, et ajout d'un message d’erreur supplémentaire.
- Amélioration de la gestion des longs silences au milieu d’un fichier audio.
- Package NuGet : pour les projets .NET Framework, il empêche toute génération avec une configuration AnyCPU.

**Résolution des bogues**

- Correction de plusieurs exceptions détectées dans les modules de reconnaissance. En outre, les exceptions sont interceptées et converties en événement `Canceled`.
- Correction d'une fuite de mémoire dans la gestion des propriétés.
- Correction d’un bogue dans lequel un fichier d’entrée audio pouvait bloquer le module de reconnaissance.
- Correction d’un bogue dans lequel des événements pouvaient être reçus après un événement d’arrêt de session.
- Correction de certaines conditions de concurrence dans le thread.
- Correction d’un problème de compatibilité avec iOS qui pouvait entraîner un blocage.
- Améliorations de la stabilité dans la prise en charge du microphone Android.
- Correction d’un bogue dans lequel un module de reconnaissance de JavaScript ignorait la langue de reconnaissance.
- Correction d’un bogue qui empêchait de définir `EndpointId` (dans certains cas) dans JavaScript.
- Modification de l'ordre des paramètres dans AddIntent dans JavaScript, et ajout de la signature `AddIntent` JavaScript manquante.

**Exemples**

- Ajout d’exemples C++ et C# pour la diffusion en continu dans l’[exemple de référentiel](https://aka.ms/csspeech/samples).

## <a name="speech-sdk-101"></a>SDK Speech 1.0.1

Améliorations de la fiabilité et résolution des bogues :

- Correction d’une erreur irrécupérable potentielle due à une condition de concurrence lors de la suppression du module de reconnaissance
- Correction d’une erreur irrécupérable potentielle en cas d’annulation de propriétés.
- Vérification supplémentaire des erreurs et des paramètres.
- Objective-C : correction d’une erreur irrécupérable possible provoquée par le remplacement d’un nom dans une chaîne NSString.
- Objective-C : réglage de la visibilité de l’API.
- JavaScript : correction des événements et de leurs charges utiles.
- Améliorations de la documentation.

Dans notre [exemple de référentiel](https://aka.ms/csspeech/samples), un nouvel échantillon pour JavaScript a été ajouté.

## <a name="cognitive-services-speech-sdk-100-2018-september-release"></a>SDK Cognitive Services Speech 1.0.0 : version de septembre 2018

**Nouvelles fonctionnalités**

- Prise en charge d’Objective-C sur iOS. Découvrez le [guide de démarrage rapide sur Objective-C pour iOS](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/objectivec/ios/from-microphone).
- Prise en charge de JavaScript dans le navigateur. Découvrez le [guide de démarrage rapide JavaScript](./get-started-speech-to-text.md).

**Dernières modifications**

- Cette version contient plusieurs changements cassants.
  Pour plus d’informations, consultez [cette page](https://aka.ms/csspeech/breakingchanges_1_0_0).

## <a name="cognitive-services-speech-sdk-060-2018-august-release"></a>Cognitive Services Speech SDK 0.6.0 : version d’août 2018

**Nouvelles fonctionnalités**

- Les applications UWP créées à partir du SDK Speech peuvent désormais passer le Kit de certification des applications Windows (WACK).
  Consultez le [guide de démarrage rapide UWP](./get-started-speech-to-text.md?pivots=programming-language-chsarp&tabs=uwp).
- Prise en charge de .NET Standard 2.0 sur Linux (Ubuntu 16.04 x64).
- Expérimental : prise en charge de Java 8 sur Windows (64 bits) et Linux (Ubuntu 16.04 x64).
  Consultez le [guide de démarrage rapide Java Runtime Environment](./get-started-speech-to-text.md?pivots=programming-language-java&tabs=jre).

**Changement fonctionnel**

- Exposition d’informations supplémentaires sur les erreurs de connexion

**Dernières modifications**

- Sur Java (Android), la fonction `SpeechFactory.configureNativePlatformBindingWithDefaultCertificate` ne requiert plus aucun paramètre de chemin d’accès. Le chemin est désormais automatiquement détecté sur toutes les plateformes prises en charge.
- L’élément get-accessor de la propriété `EndpointUrl` dans Java et C# a été supprimé.

**Résolution des bogues**

- Dans Java, le résultat de la synthèse audio sur le module de reconnaissance de traduction est maintenant implémenté.
- Correction d’un bogue qui pouvait provoquer l’inactivité des threads et un plus grand nombre de sockets ouverts et inutilisés
- Correction d’un problème qui provoquait l’arrêt d’une reconnaissance de longue durée au milieu d’une transmission
- Correction d’une condition de concurrence lors de l’arrêt du module de reconnaissance.

## <a name="cognitive-services-speech-sdk-050-2018-july-release"></a>SDK Cognitive Services Speech 0.5.0 : version de juillet 2018

**Nouvelles fonctionnalités**

- Prise en charge de la plateforme Android (API 23 : Android 6.0 Marshmallow ou supérieur). Consultez le [Démarrage rapide Android](./get-started-speech-to-text.md?pivots=programming-language-java&tabs=android).
- Prise en charge de .NET Standard 2.0 sous Windows. Consultez le [Démarrage rapide .NET Core](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnetcore).
- Expérimental : Prise en charge d’UWP sur Windows (version 1709 ou ultérieure).
  - Consultez le [guide de démarrage rapide UWP](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=uwp).
  - Remarque : les applications UWP générées avec le kit SDK Speech ne réussissent pas encore le test du Kit de certification des applications Windows (WACK).
- Prise en charge des reconnaissances de longue durée avec la reconnexion automatique

**Modifications fonctionnelles**

- `StartContinuousRecognitionAsync()` prend en charge les reconnaissances de longue durée.
- Le résultat des reconnaissances contient davantage de champs. Ils sont décalés par rapport au début de l’audio et de la durée (tous les deux en cycles) du texte reconnu et des valeurs supplémentaires représentant l’état de la reconnaissance, par exemple, `InitialSilenceTimeout` et `InitialBabbleTimeout`.
- Prise en charge d’AuthorizationToken pour la création d’instances Data Factory.

**Dernières modifications**

- Événements de reconnaissance : le type d’événement `NoMatch` a été fusionné avec l’événement `Error`.
- Le SpeechOutputFormat du langage C# a été renommé `OutputFormat` pour s’aligner sur le C++.
- Le type de retour de certaines méthodes de l’interface `AudioInputStream` a été légèrement modifié :
  - Dans Java, la méthode `read` retourne désormais `long` au lieu de `int`.
  - Dans C#, la méthode `Read` retourne désormais `uint` au lieu de `int`.
  - Dans C++, les méthodes `Read` et `GetFormat` retournent désormais `size_t` au lieu de `int`.
- C++ : les instances de flux d’entrée audio peuvent maintenant être passées comme `shared_ptr`.

**Résolution des bogues**

- Correction des valeurs de retour incorrectes dans les résultats lorsque `RecognizeAsync()` expire.
- La dépendance aux bibliothèques Media Foundation Windows a été supprimée. Le SDK utilise désormais les API Core Audio.
- Correction de la documentation : ajout de la page [régions](regions.md) pour répertorier les régions prises en charge.

**Problème connu**

- Le SDK Speech pour Android ne signale pas les résultats de la synthèse vocale pour la traduction. Ce problème sera corrigé dans la prochaine version.

## <a name="cognitive-services-speech-sdk-040-2018-june-release"></a>SDK Cognitive Services Speech 0.4.0 : version de juin 2018

**Modifications fonctionnelles**

- AudioInputStream

  Un module de reconnaissance peut désormais consommer un flux en tant que source audio. Pour plus d’informations, consultez ce [guide pratique](how-to-use-audio-input-streams.md).

- Format de sortie détaillé

  Lorsque vous créez un `SpeechRecognizer`, vous pouvez demander le format de sortie `Detailed` ou `Simple`. Le `DetailedSpeechRecognitionResult` contient un score de confiance, le texte reconnu, la forme lexicale brute, la forme normalisée et la forme normalisée avec les blasphèmes masqués.

**Modification critique**

- `SpeechRecognitionResult.Text` a été remplacé par `SpeechRecognitionResult.RecognizedText` pour le langage C#.

**Résolution des bogues**

- Correction d’un problème de rappel possible dans la couche USP qui se produisait lors de l’arrêt.
- Si un module de reconnaissance a utilisé un fichier d’entrée audio, il a été placé sur le descripteur de fichier plus longtemps que nécessaire.
- Suppression de plusieurs blocages entre la pompe de messages et le module de reconnaissance.
- Expiration du délai de déclenchement d’un résultat `NoMatch` lors de la réponse du service.
- Les bibliothèques Media Foundation Windows sont chargées en différé. Cette bibliothèque est nécessaire uniquement pour l’entrée du microphone.
- La vitesse de chargement de données audio est limitée à environ deux fois la vitesse audio d’origine.
- Désormais, les noms d’assemblys C# .NET dans Windows sont forts.
- Correction de la documentation : `Region` est obligatoire pour créer un module de reconnaissance.

D’autres exemples ont été ajoutés et sont constamment mis à jour. Pour obtenir la dernière série d’exemples, accédez au [dépôt GitHub d’exemples pour le SDK Speech](https://aka.ms/csspeech/samples).

## <a name="cognitive-services-speech-sdk-0212733-2018-may-release"></a>SDK Cognitive Services Speech 0.2.12733 : version de mai 2018

Cette version est la première préversion publique du SDK Speech de Cognitive Services.
