---
title: Guide pratique sur la préparation des données pour Custom Voice - Service Speech
titleSuffix: Azure Cognitive Services
description: Créez une voix personnalisée pour votre marque avec le service Speech. Vous fournissez des enregistrements en studio et les scripts associés et le service génère un modèle de voix unique à partir de la voix enregistrée. Cette voix assure alors la synthèse vocale dans vos produits, outils et applications.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: erhopf
ms.openlocfilehash: 5427e9f996fb77d455aa8064fc7cb1c65e1fcf7e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74805975"
---
# <a name="prepare-data-to-create-a-custom-voice"></a>Préparer des données en vue de créer une voix personnalisée

Dès que vous êtes prêt à créer une voix personnalisée pour la synthèse vocale dans votre application, vous devez dans un premier temps rassembler les enregistrements audio et les scripts associés pour commencer l’entraînement du modèle de voix. Le service Speech se sert de ces données pour créer une voix unique correspondant à la voix des enregistrements. Après avoir entraîné la voix, vous pouvez commencer la synthèse vocale dans vos applications.

Vous pouvez commencer par un petit volume de données pour créer une preuve de concept. Cependant, plus vous fournirez de données, plus votre voix personnalisée semblera naturelle. Avant d’entraîner votre propre modèle vocal pour la synthèse vocale, vous avez besoin d’enregistrements audio et des transcriptions de texte associées. Dans cette page, nous allons passer en revue les différents types de données, leurs utilisations et leur gestion respective.

## <a name="data-types"></a>Types de données

Un jeu de données d’entraînement vocal comprend des enregistrements audio et un fichier texte contenant les transcriptions associées. Chaque fichier audio doit contenir un seul énoncé (une seule phrase ou un seul tour pour un système de dialogues) et durer moins de 15 secondes.

Dans certains cas, vous n’aurez peut-être pas à disposition le jeu de données approprié et souhaiterez tester l’entraînement vocal personnalisé avec les fichiers audio disponibles, courts ou longs, avec ou sans transcriptions. Nous mettons à disposition des outils (en version bêta) destinés à faciliter la segmentation du contenu audio en énoncés et à préparer les transcriptions à l’aide de l’[API de transcription Batch](batch-transcription.md).

Ce tableau liste les types de données et la façon dont chacun est utilisé pour créer un modèle vocal personnalisé pour la synthèse vocale.

| Type de données | Description | Quand l’utiliser | Autre service nécessaire | Quantité nécessaire pour l’entraînement d’un modèle | Paramètres régionaux |
| --------- | ----------- | ----------- | --------------------------- | ----------------------------- | --------- |
| **Énoncés individuels + transcription correspondante** | Collection (.zip) de fichiers audio (.wav) correspondant à des énoncés individuels. Chaque fichier audio est limité à 15 secondes et est associé à une transcription formatée (.txt). | Enregistrements professionnels avec transcriptions correspondantes | Prêt pour l’entraînement. | Pas d’exigence spécifique pour les langues en-US et zh-CN. Plus de 2 000 énoncés distinctes pour les autres paramètres régionaux. | [Tous les paramètres régionaux de Custom Voice](language-support.md#customization) |
| **Contenu audio long + transcription (bêta)** | Collection (.zip) de fichiers audio longs et non segmentés (plus de 20 secondes), associés à une transcription (.txt) qui contient tous les mots prononcés. | Vous disposez de fichiers audio et des transcriptions correspondantes, mais ils ne sont pas segmentés en énoncés. | Segmentation (à l’aide de la transcription Batch).<br>Transformation du format audio, si nécessaire. | Pas d’exigence précise  | [Tous les paramètres régionaux de Custom Voice](language-support.md#customization) |
| **Audio uniquement (bêta)** | Collection (.zip) de fichiers audio sans transcription. | Vous disposez uniquement de fichiers audio, sans transcriptions. | Segmentation + génération de transcriptions (à l’aide de la transcription Batch).<br>Transformation du format audio, si nécessaire.| Pas d’exigence précise | [Tous les paramètres régionaux de Custom Voice](language-support.md#customization) |

Les fichiers doivent être regroupées par type dans un jeu de données et chargés sous forme de fichier zip. Chaque jeu de données ne peut contenir qu’un seul type de données.

> [!NOTE]
> Le nombre maximal de jeux de données qu’il est autorisé d’importer par abonnement est de 10 fichiers .zip pour les utilisateurs disposant d’un abonnement gratuit (F0) et de 500 pour ceux qui disposent d’un abonnement standard (S0).

## <a name="individual-utterances--matching-transcript"></a>Énoncés individuels + transcription correspondante

Vous pouvez préparer les enregistrements d’énoncés individuels et la transcription correspondante de deux façons différentes : soit en écrivant un script et en le faisant lire par une voix professionnelle, soit en utilisant un enregistrement audio disponible publiquement et en le transcrivant en texte. Dans ce dernier cas, éliminez les disfluences dans les fichiers audio telles que les « euh » et autres sons de remplissage, bégaiements, mots marmonnés ou erreurs de prononciation.

Pour produire une police de voix de qualité satisfaisante, créez les enregistrements dans une pièce silencieuse avec un microphone de grande qualité. Il est essentiel de faire attention à l’homogénéité du volume, au débit, à la tonalité et de s’exprimer de manière expressive.

> [!TIP]
> Pour créer une voix à des fins de production, nous vous recommandons de faire appel aux services d’un studio d’enregistrement et d’une voix professionnelle. Pour plus d’informations, consultez le [Guide pratique pour enregistrer des exemples de voix pour une voix personnalisée](record-custom-voice-samples.md).

### <a name="audio-files"></a>Fichiers audio

Chaque fichier audio doit contenir un seul énoncé (une seule phrase ou un seul tour pour un système de dialogues) et durer moins de 15 secondes. La langue parlée doit être identique dans tous les fichiers. Les voix de synthèse vocale personnalisées multilingues ne sont pas prises en charge, à l’exception de la combinaison chinois-anglais. Chaque fichier audio doit avoir un nom de fichier numérique unique avec l’extension de nom de fichier .wav.

Suivez ces recommandations pendant la préparation du contenu audio.

| Propriété | Valeur |
| -------- | ----- |
| Format de fichier | RIFF (.wav), regroupé dans un fichier .zip |
| Taux d’échantillonnage | Au moins 16 000 Hz |
| Format d’échantillonnage | PCM, 16 bits |
| Nom de fichier | Numérique, avec l’extension .wav. Noms de fichiers en double non autorisés. |
| Durée du contenu audio | Moins de 15 secondes |
| Format d’archive | .zip |
| Taille d’archive maximale | 2 048 Mo |

> [!NOTE]
> Les fichiers .wav dont le taux d’échantillonnage est inférieur à 16 000 Hz sont rejetés. Si un fichier .zip contient des fichiers .wav dont le taux d’échantillonnage est différent, seuls ceux dont ce taux est supérieur ou égal à 16 000 Hz sont importés. Actuellement, le portail importe les archives .zip jusqu’à 200 Mo. Toutefois, il est possible de charger plusieurs archives.

### <a name="transcripts"></a>Transcriptions

Le fichier de transcription est un fichier texte brut. Suivez ces recommandations pour préparer votre transcriptions.

| Propriété | Valeur |
| -------- | ----- |
| Format de fichier | Texte brut (.txt) |
| Format d’encodage | ANSI/ASCII, UTF-8, UTF-8-BOM, UTF-16-LE ou UTF-16-BE. Pour zh-CN, les encodages ANSI/ASCII et UTF-8 ne sont pas pris en charge. |
| Nb d’énoncés par ligne | **Un** – Chaque ligne du fichier de transcription doit contenir le nom d’un des fichiers audio, suivi de la transcription correspondante. Le nom de fichier et la transcription doivent être séparés par une tabulation (\t). |
| Taille maximale du fichier | 2 048 Mo |

Voici un exemple d’organisation des transcriptions, énoncé par énoncé, dans un même fichier .txt :

```
0000000001[tab] This is the waistline, and it's falling.
0000000002[tab] We have trouble scoring.
0000000003[tab] It was Janet Maslin.
```
Il est important que les transcriptions soient parfaitement fidèles au contenu audio correspondant. Les erreurs de transcription entraînent une perte de qualité pendant l’entraînement.

> [!TIP]
> Quand vous créez des voix pour une synthèse vocale de production, sélectionnez des énoncés (ou rédigez des scripts) qui conjuguent efficacité et couverture phonétique. Des difficultés à obtenir les résultats que vous souhaitez ? [Contactez l’équipe Custom Voice](mailto:speechsupport@microsoft.com) pour savoir comment obtenir des conseils.

## <a name="long-audio--transcript-beta"></a>Contenu audio long + transcription (bêta)

Dans certains cas, vous n’aurez peut-être pas de contenu audio segmenté à disposition. Nous proposons un service (en version bêta) via le Custom Voice destiné à vous aider à segmenter les fichiers audio longs et à créer des transcriptions. Gardez à l’esprit que l’utilisation de ce service vous sera facturée au titre de votre abonnement de reconnaissance vocale.

> [!NOTE]
> Le service de segmentation de contenu audio de longue durée exploite la fonctionnalité de transcription Batch de la reconnaissance vocale, qui prend uniquement en charge les utilisateurs disposant d’un abonnement standard (S0). Pendant le traitement de la segmentation, vos fichiers audio et les transcriptions seront aussi être envoyés au service Custom Speech de façon à affiner le modèle de reconnaissance et ainsi améliorer la précision de vos données. Aucune donnée ne sera conservée pendant ce processus. À l’issue de la segmentation, seuls les énoncés segmentés et leurs transcriptions correspondantes seront stockés pour vos besoins de téléchargement et d’entraînement.

### <a name="audio-files"></a>Fichiers audio

Suivez ces recommandations au moment de préparer le contenu audio à segmenter.

| Propriété | Valeur |
| -------- | ----- |
| Format de fichier | RIFF (.wav) avec un taux d’échantillonnage d’au moins 16 khz 16 bits en PCM ou .mp3 avec une vitesse de transmission d’au moins 256 Kbits/s, le tout regroupé dans un fichier .zip |
| Nom de fichier | Prise en charge des caractères Unicode et ASCII. Les noms de fichiers en double ne sont pas autorisés. |
| Durée du contenu audio | Plus de 20 secondes |
| Format d’archive | .zip |
| Taille d’archive maximale | 2 048 Mo |

Tous les fichiers audio doivent être regroupés dans un fichier zip. Il est possible de placer des fichiers .wav et des fichiers .mp3 dans un même fichier zip audio. Par exemple, vous pouvez charger un fichier zip contenant un fichier audio nommé « kingstory.wav », d’une durée de 45 secondes, et un autre nommé « queenstory.mp3 », d’une durée de 200 secondes. Tous les fichiers .mp3 seront convertis au format .wav à l’issue du traitement.

### <a name="transcripts"></a>Transcriptions

Les transcriptions doivent être préparées selon les spécifications listées dans ce tableau. Chaque fichier audio doit être mis en correspondance avec une transcription.

| Propriété | Valeur |
| -------- | ----- |
| Format de fichier | Texte brut (.txt), regroupé dans un fichier .zip |
| Nom de fichier | Utilisez le nom du fichier audio correspondant |
| Format d’encodage | UTF-8-BOM uniquement |
| Nb d’énoncés par ligne | Aucune limite |
| Taille maximale du fichier | 2 048 Mo |

Tous les fichiers de transcriptions de ce type de données doivent être regroupés dans un fichier zip. Par exemple, supposons que vous avez chargé un fichier zip contenant un fichier audio nommé « kingstory.wav », d’une durée de 45 secondes, et un autre nommé « queenstory.mp3 », d’une durée de 200 secondes. Vous devez charger un autre fichier zip contenant deux transcriptions, l’une nommée « kingstory.txt » et l’autre « queenstory.txt ». Dans chaque fichier texte brut, vous fournirez la transcription complète correcte pour le contenu audio correspondant.

Une fois le jeu de données chargé, nous vous aiderons à segmenter le fichier audio en énoncés sur la base de la transcription fournie. Vous pouvez vérifier les énoncés segmentés et les transcriptions correspondantes en téléchargeant le jeu de données. Des ID uniques seront attribués automatiquement aux énoncés segmentés. Il est important de vérifier que les transcriptions que vous fournissez sont précises à 100 %. La présence d’erreurs dans les transcriptions peut réduire la précision pendant la segmentation audio et occasionner des pertes de qualité pendant la phase d’entraînement qui vient après.

## <a name="audio-only-beta"></a>Audio uniquement (bêta)

Si vous n’avez pas de transcriptions pour vos enregistrements audio, utilisez l’option **Audio uniquement** pour charger vos données. Notre système peut vous aider à segmenter et transcrire vos fichiers audio. Gardez à l’esprit que l’utilisation de ce service vous sera décomptée de votre abonnement de reconnaissance vocale.

Suivez ces recommandations pendant la préparation du contenu audio.

> [!NOTE]
> Le service de segmentation de contenu audio de longue durée exploite la fonctionnalité de transcription Batch de la reconnaissance vocale, qui prend uniquement en charge les utilisateurs disposant d’un abonnement standard (S0).

| Propriété | Valeur |
| -------- | ----- |
| Format de fichier | RIFF (.wav) avec un taux d’échantillonnage d’au moins 16 khz 16 bits en PCM ou .mp3 avec une vitesse de transmission d’au moins 256 Kbits/s, le tout regroupé dans un fichier .zip |
| Nom de fichier | Prise en charge des caractères Unicode et ASCII. Aucun nom en double autorisé. |
| Durée du contenu audio | Plus de 20 secondes |
| Format d’archive | .zip |
| Taille d’archive maximale | 2 048 Mo |

Tous les fichiers audio doivent être regroupés dans un fichier zip. Une fois le jeu de données chargé, nous vous aiderons à segmenter le fichier audio en énoncés à partir de notre service de transcription Batch. Des ID uniques seront attribués automatiquement aux énoncés segmentés. Les transcriptions correspondantes seront générées via la reconnaissance vocale. Tous les fichiers .mp3 seront convertis au format .wav à l’issue du traitement. Vous pouvez vérifier les énoncés segmentés et les transcriptions correspondantes en téléchargeant le jeu de données.

## <a name="next-steps"></a>Étapes suivantes

- [Créer une voix personnalisée Custom Voice](how-to-custom-voice-create-voice.md)
- [Guide : Enregistrer des échantillons vocaux](record-custom-voice-samples.md)
