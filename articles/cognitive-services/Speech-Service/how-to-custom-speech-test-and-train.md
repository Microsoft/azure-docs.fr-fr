---
title: Préparer des données pour Custom Speech - Service Speech
titleSuffix: Azure Cognitive Services
description: Lors du test de la précision de la reconnaissance vocale Microsoft ou de l’apprentissage de vos modèles personnalisés, vous aurez besoin de données audio et texte. Dans cette page, nous nous intéressons aux types de données, à la façon dont ils sont utilisés et à leur gestion.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: a5457dc94082f089d3adf02c9614d05d2c5db244
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484003"
---
# <a name="prepare-data-for-custom-speech"></a>Préparer des données pour Custom Speech

Lors du test de la précision de la reconnaissance vocale Microsoft ou de l’apprentissage de vos modèles personnalisés, vous aurez besoin de données audio et texte. Dans cette page, nous abordons les types de données requis par un modèle Custom Speech.

## <a name="data-diversity"></a>Diversité des données

Le texte et l’audio utilisés pour tester et entraîner un modèle personnalisé doivent inclure des échantillons issus d’un ensemble diversifié de haut-parleurs et de scénarios que votre modèle doit reconnaître.
Tenez compte des facteurs suivants lors de la collecte de données pour le test et l’entraînement de modèles personnalisés :

* Vos données texte et audio doivent couvrir les types d’instructions verbales que vos utilisateurs prononceront lorsqu’ils interagiront avec votre modèle. Par exemple, un modèle chargé d’élever et d’abaisser la température nécessite un entraînement sur des instructions possibles permettant de demander de telles modifications.
* Vos données doivent inclure toutes les variantes de langage que votre modèle doit reconnaître. De nombreux facteurs peuvent faire varier l’audio, y compris les accents, les dialectes, le mélange de langues, l’âge, le sexe, la hauteur de la voix, le degré de stress et l’heure de la journée.
* Vous devez inclure des échantillons provenant de différents environnements (en intérieur, en extérieur, avec bruits de route) où votre modèle sera utilisé.
* Le contenu audio doit être collecté à l’aide des périphériques matériels que le système de production utilisera. Si votre modèle doit identifier la parole enregistrée sur des appareils d’enregistrement de diverses qualités, les données audio que vous fournissez pour entraîner votre modèle doivent également être caractéristiques de ces différents scénarios.
* Vous pouvez ajouter ultérieurement des données à votre modèle, mais veillez à utiliser un jeu de données diversifié et représentatif des besoins de votre projet.
* L’insertion de données qui *ne figurent pas* dans les besoins de reconnaissance de modèle personnalisé peut nuire à la qualité globale de la reconnaissance. N’insérez donc pas de données dont votre modèle n’a pas besoin pour transcrire.

Un modèle entraîné dans le cadre d’un sous-ensemble de scénarios ne peut fonctionner correctement que dans ces scénarios. Choisissez soigneusement les données qui représentent l’étendue complète des scénarios que votre modèle personnalisé doit reconnaître.

> [!TIP]
> Commencez avec de petits ensembles d’exemples de données correspondant à la langue et à l’acoustique que votre modèle rencontrera.
> Par exemple, enregistrez un petit échantillon représentatif du contenu audio sur le même matériel et dans le même environnement acoustique que votre modèle rencontrera dans les scénarios de production.
> De petits jeux de données représentatifs peuvent exposer des problèmes avant que vous ayez investi dans la collecte de jeux de données beaucoup plus volumineux à des fins d’entraînement.

## <a name="data-types"></a>Types de données

Ce tableau liste les types de données acceptés, les cas d’utilisation pour chacun d’eux ainsi que la quantité recommandée. Pour créer un modèle, vous n’avez pas besoin de chaque type de données. Les données nécessaires varient selon que vous créez un test ou entraînez un modèle.

| Type de données | Utilisé pour le test | Quantité recommandée | Utilisé pour l’entraînement | Quantité recommandée |
|-----------|-----------------|----------|-------------------|----------|
| [Audio](#audio-data-for-testing) | Oui<br>Utilisé pour l’inspection visuelle | 5 fichiers audio et plus | Non | N/A |
| [Transcriptions audio + étiquetées à la main](#audio--human-labeled-transcript-data-for-testingtraining) | Oui<br>Utilisé pour évaluer la précision | 0,5 - 5 heures d’audio | Oui | 1 -1 000 heures d’audio |
| [Texte associé](#related-text-data-for-training) | Non | n/a | Oui | 1 – 200 Mo de texte associé |

Les fichiers doivent être regroupées par type dans un jeu de données et chargés sous forme de fichier .zip. Chaque jeu de données ne peut contenir qu’un seul type de données.

> [!TIP]
> Pour commencer rapidement, vous pouvez utiliser des exemples de données. Consultez ce dépôt GitHub pour un <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/sampledata/customspeech" target="_target">exemple de données Custom Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="upload-data"></a>Télécharger des données

Pour télécharger vos données, accédez au <a href="https://speech.microsoft.com/customspeech" target="_blank">portail Custom Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a>. Dans le portail, cliquez sur **Upload data** (Charger des données) pour lancer l’Assistant et créer votre premier jeu de données. Vous êtes alors invité à sélectionner un type de données vocales pour votre jeu de données avant d’être autorisé à charger vos données.

![Capture d’écran mettant en évidence l’option Chargement audio dans le portail Speech.](./media/custom-speech/custom-speech-select-audio.png)

Chaque jeu de données que vous chargez doit respecter les exigences associé au type de données choisi. Vos données doivent être correctement mises en forme avant d’être chargées. Des données correctement mises en forme garantissent un traitement précis par le service Custom Speech. Les exigences sont listées dans les sections suivantes.

Une fois que votre jeu de données est chargé, vous disposez de plusieurs options :

* Vous pouvez accéder à l’onglet **Testing** (Tests) et inspecter visuellement du contenu audio uniquement ou du contenu audio + données de transcription étiquetées à la main.
* Vous pouvez accéder à l’onglet **Training** (Entraînement) et utiliser du contenu audio + données de transcription étiquetées à la main ou des données de texte associées pour entraîner un modèle personnalisé.

## <a name="audio-data-for-testing"></a>Données audio pour tester

Les données audio sont optimales pour tester la précision du modèle de reconnaissance vocale de référence de Microsoft ou d’un modèle personnalisé. Gardez à l’esprit que les données audio sont utilisées pour inspecter la précision de la reconnaissance vocale par rapport à un modèle spécifique. Si vous cherchez à quantifier la précision d’un modèle, utilisez du [contenu audio + données de transcription étiquetés à la main](#audio--human-labeled-transcript-data-for-testingtraining).

Servez-vous de ce tableau pour vérifier que le format de vos fichiers audio convient pour une utilisation avec Custom Speech :

| Propriété                 | Valeur                 |
|--------------------------|-----------------------|
| Format de fichier              | RIFF (WAV)            |
| Échantillonnage              | 8 000 Hz ou 16 000 Hz |
| Canaux                 | 1 (mono)              |
| Longueur maximale par fichier audio | 2 heures               |
| Format d’échantillonnage            | PCM, 16 bits           |
| Format d’archive           | .zip                  |
| Taille d’archive maximale     | 2 Go                  |

[!INCLUDE [supported-audio-formats](includes/supported-audio-formats.md)]

> [!TIP]
> Lors du chargement de données de formation et de test, la taille du fichier .zip ne peut pas dépasser 2 Go. Si vous avez besoin de plus de données pour l’entraînement, divisez-le en plusieurs fichiers .zip et chargez-les séparément. Plus tard, vous pouvez choisir d’effectuer l’entraînement à partir de *plusieurs* jeux de données. Cependant, vous ne pouvez tester qu’à partir d’un *seul* jeu de données.

Utilisez <a href="http://sox.sourceforge.net" target="_blank" rel="noopener">SoX <span class="docon docon-navigate-external x-hidden-focus"></span></a> pour vérifier les propriétés audio ou pour convertir les données audio existantes aux formats appropriés. Vous trouverez ci-dessous quelques exemples de la façon dont chacune de ces activités peut être effectuée via la ligne de commande SoX :

| Activité | Description | Commande SoX |
|----------|-------------|-------------|
| Vérifier le format audio | Utilisez cette commande pour vérifier<br>le format du fichier audio. | `sox --i <filename>` |
| Convertir le format audio | Utilisez cette commande pour convertir<br>le fichier audio en un canal unique, 16 bits, 16 KHz. | `sox <input> -b 16 -e signed-integer -c 1 -r 16k -t wav <output>.wav` |

## <a name="audio--human-labeled-transcript-data-for-testingtraining"></a>Contenu audio + données de transcription étiquetées à la main pour tester/entraîner

Pour mesurer la précision de la reconnaissance vocale de Microsoft pendant le traitement de vos fichiers audio, vous devez fournir des transcriptions étiquetées à la main (mot par mot) pour effectuer la comparaison. Si la transcription étiquetée à la main prend souvent beaucoup de temps, elle est nécessaire pour évaluer la précision et entraîner le modèle pour vos cas d’usage. Gardez à l’esprit que les améliorations de la reconnaissance seront proportionnelles à la qualité des données fournies. C’est pourquoi il est important de charger uniquement des transcriptions de grande qualité.

Les fichiers audio peuvent avoir un silence au début et à la fin de l’enregistrement. Si possible, incluez au moins une demi-seconde de silence avant et après Speech dans chaque exemple de fichier. Bien que les données audio avec un faible volume d’enregistrement ou un bruit d’arrière-plan perturbateur ne soient pas utiles, elles ne doivent pas nuire à votre modèle personnalisé. Envisagez toujours de mettre à niveau vos micro et votre matériel de traitement du signal avant de rassembler les échantillons audio.

| Propriété                 | Valeur                               |
|--------------------------|-------------------------------------|
| Format de fichier              | RIFF (WAV)                          |
| Échantillonnage              | 8 000 Hz ou 16 000 Hz               |
| Canaux                 | 1 (mono)                            |
| Longueur maximale par fichier audio | 2 heures (test) /60 s (entraînement) |
| Format d’échantillonnage            | PCM, 16 bits                         |
| Format d’archive           | .zip                                |
| Taille maximale de zip         | 2 Go                                |

[!INCLUDE [supported-audio-formats](includes/supported-audio-formats.md)]

> [!NOTE]
> Lors du chargement de données de formation et de test, la taille du fichier .zip ne peut pas dépasser 2 Go. Vous ne pouvez effectuer des tests qu’à partir d’un *seul* jeu de données : veillez donc à ce que sa taille de fichier reste appropriée. De plus, chaque fichier d’entraînement ne peut pas dépasser 60 secondes. Dans le cas contraire, il génère une erreur.

Pour résoudre les problèmes comme la suppression ou la substitution de mots, une quantité importante de données est nécessaire pour améliorer la reconnaissance. En règle générale, il est recommandé de fournir des transcriptions mot par mot pour environ 10 à 1 000 heures de contenu audio. Les transcriptions de tous les fichiers WAV doivent se trouver dans un seul fichier en texte brut. Chaque ligne du fichier de transcription doit contenir le nom d’un des fichiers audio, suivi de la transcription correspondante. Le nom de fichier et la transcription doivent être séparés par une tabulation (\t).

  Par exemple :
```
  speech01.wav  speech recognition is awesome
  speech02.wav  the quick brown fox jumped all over the place
  speech03.wav  the lazy dog was not amused
```

> [!IMPORTANT]
> La transcription doit être encodée au format UTF-8 marque d'ordre d'octet (BOM).

Les transcriptions doivent être en texte normalisé pour pouvoir être traitées par le système. Une normalisation importante doit cependant être effectuée par l’utilisateur avant de charger les données dans le Studio Speech. Pour savoir quelle langue utiliser pour la préparation de vos transcriptions, consultez [Guide pratique pour créer une transcription étiquetée à la main](how-to-custom-speech-human-labeled-transcriptions.md).

Une fois que vous avez collecté vos fichiers audio et les transcriptions correspondantes, vous devez les packager dans un seul fichier .zip avant de charger ce dernier sur le <a href="https://speech.microsoft.com/customspeech" target="_blank">portail Custom Speech<span class="docon docon-navigate-external x-hidden-focus"></span></a>. Voici un exemple de jeu de données constitué de trois fichiers audio et d’un fichier de transcriptions étiquetées à la main :

> [!div class="mx-imgBorder"]
> ![Sélectionner du contenu audio à partir du portail Speech](./media/custom-speech/custom-speech-audio-transcript-pairs.png)

## <a name="related-text-data-for-training"></a>Données texte associées pour l’entraînement

Les noms de produits ou les fonctionnalités uniques doivent inclure des données texte associées pour l’entraînement. Le texte associé permet de garantir une reconnaissance correcte. Deux types de données texte associées peuvent être fournis pour améliorer la reconnaissance :

| Type de données | Comment ces données améliorent la reconnaissance |
|-----------|------------------------------------|
| Phrases (énoncés) | Améliorent la précision de la reconnaissance des noms de produits ou du vocabulaire d’un domaine spécifique dans le contexte d’une phrase. |
| Prononciations | Améliorent la prononciation des termes peu courants, des acronymes ou d’autres mots dont la prononciation n’est pas définie. |

Les phrases peuvent être fournies dans un ou plusieurs fichiers texte. Pour améliorer la précision, utilisez des données texte plus proches des énoncés prononcés attendus. Les prononciations doivent être fournies sous forme de fichier texte unique. L’ensemble peut être packagé en un seul fichier zip et chargé sur le <a href="https://speech.microsoft.com/customspeech" target="_blank">portail Custom Speech <span class="docon docon-navigate-external x-hidden-focus"></span></a>.

### <a name="guidelines-to-create-a-sentences-file"></a>Recommandations pour créer un fichier de phrases

Pour créer un modèle personnalisé en utilisant des phrases, vous devez fournir une liste d’exemples d’énoncés. Ces énoncés ne doivent _pas_ nécessairement être des phrases complètes ou grammaticalement correctes. En revanche, elles doivent refléter précisément l’entrée parlée que vous attendez en production. Pour donner plus de poids à certains termes, ajoutez plusieurs phrases contenant ces termes spécifiques.

En règle générale, l’adaptation du modèle est plus efficace quand le texte d’entraînement est le plus proche possible du texte réel attendu en production. Le jargon et les expressions spécifiques à un domaine que vous voulez améliorer doivent être inclus dans le texte d’entraînement. Dans la mesure du possible, essayez d’avoir une phrase ou un mot clé contrôlé sur une ligne distincte. Pour les mots clés et les expressions qui sont importants pour vous (par exemple des noms de produits), vous pouvez les copier plusieurs fois. Gardez cependant à l’esprit qu’il ne faut pas les copier un trop grand nombre de fois, car cela pourrait affecter le taux de reconnaissance global.

Servez-vous de ce tableau pour vérifier que votre fichier de données associées présente le bon format pour les énoncés :

| Propriété | Valeur |
|----------|-------|
| Encodage de texte | UTF-8 BOM |
| Nb d’énoncés par ligne | 1 |
| Taille maximale du fichier | 200 Mo |

Par ailleurs, vous devez prendre en compte les restrictions suivantes :

* Évitez de répéter un même caractère plus de 4 fois. Par exemple : « aaaa » ou « uuuu ».
* N’utilisez pas de caractères spéciaux ou de caractères UTF-8 au-delà de `U+00A1`.
* Les URI sont rejetés.

### <a name="guidelines-to-create-a-pronunciation-file"></a>Recommandations pour créer un fichier de prononciation

Si vos utilisateurs sont appelés à rencontrer ou à utiliser des termes peu communs qui se prononcent d’une façon particulière, vous pouvez fournir un fichier de prononciation personnalisé pour améliorer la reconnaissance.

> [!IMPORTANT]
> Il est déconseillé d’utiliser des fichiers de prononciation personnalisée pour altérer la prononciation des mots communs.

Ce tableau contient des exemples d’énoncés oraux et une prononciation personnalisée pour chacun d’eux :

| Forme reconnue/affichée | Forme orale (en anglais) |
|--------------|--------------------------|
| 3CPO | trois c p o |
| CNTK (Computational Network Toolkit de Microsoft Research) | c n t k |
| IEEE | i triple e |

La forme orale est la séquence phonétique décomposée. Elle peut être constituée de lettres, de mots, de syllabes ou d’une combinaison des trois.

La prononciation personnalisée est disponible en anglais (`en-US`) et en allemand (`de-DE`). Ce tableau présente les caractères pris en charge par langue :

| Langage | Paramètres régionaux | Caractères |
|----------|--------|------------|
| Anglais | `en-US` | `a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z` |
| Allemand | `de-DE` | `ä, ö, ü, a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z` |

Utilisez le tableau suivant pour vérifier que votre fichier de données associées est correctement mis en forme. Les fichiers de prononciation sont petits et doivent se limiter à quelques Ko.

| Propriété | Valeur |
|----------|-------|
| Encodage de texte | UTF-8 BOM (ANSI est également pris en charge pour l’anglais) |
| Nbre de prononciations par ligne | 1 |
| Taille maximale du fichier | 1 Mo (1 Ko pour le niveau gratuit) |

## <a name="next-steps"></a>Étapes suivantes

* [Inspecter les données](how-to-custom-speech-inspect-data.md)
* [Évaluer les données](how-to-custom-speech-evaluate-data.md)
* [Entraîner un modèle](how-to-custom-speech-train-model.md)
* [Déployer un modèle](./how-to-custom-speech-train-model.md)