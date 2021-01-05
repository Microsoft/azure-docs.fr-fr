---
title: Bases de l’interface CLI Speech
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser le service Speech avec l’outil de commande de l’interface CLI Speech, sans code et avec une configuration minimale.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 04/04/2020
ms.author: trbye
ms.openlocfilehash: 1b92d1b5853d6b794ebdcf0e2052b8f15081d608
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507572"
---
# <a name="learn-the-basics-of-the-speech-cli"></a>Présentation des bases de l’interface CLI Speech

Cet article présente les modèles d’utilisation de base de l’interface de ligne de commande (CLI) Speech, un outil en ligne de commande qui permet d’utiliser le service Speech sans écrire de code. Vous pouvez tester tout de suite les principales fonctionnalités du service Speech, sans créer d’environnement de développement ni écrire de code, pour voir s’il correspond bien à vos cas d’utilisation. L’interface CLI Speech est pleinement utilisable en production et peut servir à automatiser des workflows simples dans le service Speech, à l’aide de scripts `.bat` ou shell.

Cet article suppose que vous avez une bonne connaissance de l’invite de commandes, du terminal ou de PowerShell.

[!INCLUDE [](includes/spx-setup.md)]

## <a name="basic-usage"></a>Utilisation de base

Cette section présente quelques commandes SPX de base qui sont souvent utiles pour les premiers tests et les premières expérimentations. Commencez par afficher l'aide intégrée à l'outil en exécutant la commande suivante.

```shell
spx
```

Remarquez les rubriques d’aide **Voir aussi** listées à droite des paramètres de commande. Vous pouvez entrer ces commandes pour obtenir une aide détaillée sur les sous-commandes.

Vous pouvez rechercher des rubriques d’aide par mot clé. Par exemple, entrez la commande suivante pour afficher la liste des exemples d’utilisation de l’interface CLI Speech :

```shell
spx help find --topics "examples"
```

Entrez la commande suivante pour afficher les options de la commande Recognize :

```shell
spx help recognize
```

À présent, nous allons utiliser l’interface CLI de Speech pour obtenir une reconnaissance vocale en utilisant le micro par défaut de votre système. 

>[!WARNING]
> Si vous utilisez un conteneur Docker, cette commande ne fonctionnera pas.

Exécutez cette commande :

```shell
spx recognize --microphone
```

Avec l’interface CLI de Speech, vous pouvez également effectuer une reconnaissance vocale à partir d’un fichier audio.

```shell
spx recognize --file /path/to/file.wav
```
> [!TIP]
> Si vous effectuez une reconnaissance vocale à partir d’un fichier audio situé dans un conteneur Docker, vérifiez que le fichier audio se trouve dans le répertoire que vous avez monté à l’étape précédente.

Une fois la commande entrée, SPX commence à écouter l’audio sur le périphérique d’entrée actif. Il s’arrête lorsque vous appuyez sur `ENTER`. La parole enregistrée est ensuite reconnue et convertie en texte dans la sortie de la console. La synthèse vocale est également facile à effectuer à l’aide de l’interface CLI Speech. 

La commande suivante prend en entrée le texte saisi et donne en sortie la synthèse vocale sur le périphérique de sortie actif.

```shell
spx synthesize --text "Testing synthesis using the Speech CLI" --speakers
```

Outre la reconnaissance et la synthèse vocales, l’interface CLI Speech permet d’effectuer une traduction vocale. Comme pour la reconnaissance vocale ci-dessus, exécutez la commande suivante pour capturer le flux audio de votre microphone par défaut et effectuer la traduction en texte dans la langue cible.

```shell
spx translate --microphone --source en-US --target ru-RU --output file C:\some\file\path\russian_translation.txt
```

Dans cette commande, vous spécifiez à la fois la langue source (**avant** traduction) et la langue cible (**après** traduction). L’argument `--microphone` permet d’écouter l’audio sur le périphérique d’entrée actif et de s’arrêter lorsque vous appuyez sur `ENTER`. La sortie est une traduction de texte dans la langue cible, écrite dans un fichier texte.

> [!NOTE]
> Pour connaître la liste de toutes les langues prises en charge avec le code de paramètres régionaux correspondant, consultez l’article [Langues et paramètres régionaux](language-support.md).

### <a name="configuration-files-in-the-datastore"></a>Fichiers de configuration dans le magasin de données

Le comportement de l’interface CLI Speech peut s’appuyer sur les paramètres des fichiers de configuration, que vous pouvez référencer dans les appels de l’interface CLI Speech à l’aide d’un symbole \@.
L’interface CLI Speech enregistre un nouveau paramètre dans un sous-répertoire `./spx/data` qu’elle crée dans le répertoire de travail actuel.
Lors de la recherche d’une valeur de configuration, l’interface CLI Speech recherche dans votre répertoire de travail actuel, puis dans le magasin de données dans `./spx/data`, puis dans d’autres magasins de données, y compris un magasin de données en lecture seule final dans le fichier binaire `spx`.
Avant, comme vous utilisiez le magasin de données pour enregistrer vos valeurs `@key` et `@region`, vous n’aviez pas besoin de les spécifier avec chaque appel de ligne de commande.
Vous pouvez également utiliser des fichiers de configuration pour stocker vos propres paramètres de configuration, voire même pour transmettre des URL ou d’autres contenus dynamiques générés au moment de l’exécution.

Cette section montre comment utiliser un fichier de configuration dans le magasin de données local pour stocker et extraire des paramètres de commande à l’aide de `spx config`, et comment stocker la sortie de l’interface CLI Speech à l’aide de l’option `--output`.

L’exemple suivant efface le fichier de configuration `@my.defaults`, ajoute des paires clé-valeur pour **key** et **region** dans le fichier et utilise la configuration dans un appel à `spx recognize`.

```shell
spx config @my.defaults --clear
spx config @my.defaults --add key 000072626F6E20697320636F6F6C0000
spx config @my.defaults --add region westus

spx config @my.defaults

spx recognize --nodefaults @my.defaults --file hello.wav
```

Vous pouvez également écrire du contenu dynamique dans un fichier de configuration. Par exemple, la commande suivante crée un modèle vocal personnalisé et stocke l’URL du nouveau modèle dans un fichier de configuration. La commande suivante attend que le modèle défini à cette URL soit prêt à être utilisé pour passer la main.

```shell
spx csr model create --name "Example 4" --datasets @my.datasets.txt --output url @my.model.txt
spx csr model status --model @my.model.txt --wait
```

L’exemple suivant écrit deux URL dans le fichier de configuration `@my.datasets.txt`.
Dans ce scénario, `--output` peut inclure un mot clé **add** facultatif pour créer un fichier de configuration ou compléter l’existant.


```shell
spx csr dataset create --name "LM" --kind Language --content https://crbn.us/data.txt --output url @my.datasets.txt
spx csr dataset create --name "AM" --kind Acoustic --content https://crbn.us/audio.zip --output add url @my.datasets.txt

spx config @my.datasets.txt
```

Pour plus d’informations sur les fichiers de magasin de données, notamment sur l’utilisation des fichiers de configuration par défaut (`@spx.default`, `@default.config`et `@*.default.config` pour les paramètres par défaut propres à une commande), entrez la commande suivante :

```shell
spx help advanced setup
```

## <a name="batch-operations"></a>Opérations de traitement par lots

Les commandes de la section précédente sont idéales pour découvrir rapidement le fonctionnement du service Speech. Toutefois, pour évaluer s’il peut correspondre à vos cas d’utilisation, vous devez probablement effectuer des opérations de traitement par lots sur une série d’entrées dont vous disposez déjà, afin de voir comment le service gère différents scénarios. Cette section montre comment effectuer les opérations suivantes :

* Exécuter une reconnaissance vocale par lots sur un répertoire de fichiers audio
* Exécuter une synthèse vocale par lots en effectuant des itérations sur un fichier `.tsv`

## <a name="batch-speech-recognition"></a>Reconnaissance vocale par lots

Si vous disposez d’un répertoire de fichiers audio, il est facile d’exécuter rapidement une reconnaissance vocale par lots avec l’interface CLI Speech. Exécutez simplement la commande suivante, en pointant sur votre répertoire avec la commande `--files`. Dans cet exemple, `\*.wav` est ajouté au répertoire pour reconnaître tous les fichiers `.wav` présents dans le répertoire. Spécifiez également l’argument `--threads` pour exécuter la reconnaissance sur 10 threads parallèles.

> [!NOTE]
> L’argument `--threads` peut également être utilisé dans la section suivante pour les commandes `spx synthesize`, et les threads disponibles dépendent du processeur et du pourcentage de charge actuel.

```shell
spx recognize --files C:\your_wav_file_dir\*.wav --output file C:\output_dir\speech_output.tsv --threads 10
```

La sortie vocale reconnue est écrite dans `speech_output.tsv` à l’aide de l’argument `--output file`. Voici un exemple de structure de fichier de sortie.

```output
audio.input.id    recognizer.session.started.sessionid    recognizer.recognized.result.text
sample_1    07baa2f8d9fd4fbcb9faea451ce05475    A sample wave file.
sample_2    8f9b378f6d0b42f99522f1173492f013    Sample text synthesized.
```

## <a name="synthesize-speech-to-a-file"></a>Synthétiser la voix dans un fichier

Exécutez la commande suivante pour convertir la sortie de votre haut-parleur en fichier `.wav`.

```bash
spx synthesize --text "The speech synthesizer greets you!" --audio output greetings.wav
```

L’interface CLI Speech génère un langage naturel en anglais dans le fichier audio `greetings.wav`.
Dans Windows, vous pouvez lire le fichier audio en entrant `start greetings.wav`.


## <a name="batch-text-to-speech-synthesis"></a>Synthèse vocale par lots

Le moyen le plus simple d’exécuter une synthèse vocale consiste à créer un fichier `.tsv` (valeurs séparées par des tabulations) et à tirer parti de la commande `--foreach` de l’interface CLI Speech. Prenons le fichier `text_synthesis.tsv` suivant :

```output
audio.output    text
C:\batch_wav_output\wav_1.wav    Sample text to synthesize.
C:\batch_wav_output\wav_2.wav    Using the Speech CLI to run batch-synthesis.
C:\batch_wav_output\wav_3.wav    Some more text to test capabilities.
```

 Ensuite, exécutez une commande pour pointer vers `text_synthesis.tsv`, effectuer une synthèse sur chaque champ `text` et écrire le résultat sur le chemin `audio.output` correspondant dans un fichier `.wav`. 

```shell
spx synthesize --foreach in @C:\your\path\to\text_synthesis.tsv
```

Cette commande équivaut à exécuter `spx synthesize --text Sample text to synthesize --audio output C:\batch_wav_output\wav_1.wav` **pour chaque** enregistrement du fichier `.tsv`. Points à prendre en compte :

* Les en-têtes de colonne, `audio.output` et `text`, correspondent respectivement aux arguments de ligne de commande `--audio output` et `--text`. Les arguments de ligne de commande en plusieurs parties, comme `--audio output`, doivent être mis en forme dans le fichier sans espaces, ni tirets de début, ni points de séparation entre les chaînes, par exemple `audio.output`. Tous les autres arguments de ligne de commande existants peuvent être ajoutés au fichier sous forme de colonnes supplémentaires suivant ce modèle.
* Quand le fichier est mis en forme de cette manière, aucun argument supplémentaire n’est obligatoire pour `--foreach`.
* Veillez à séparer les valeurs du `.tsv` par une **tabulation**.

Toutefois, si votre fichier `.tsv` comporte des en-têtes de colonnes qui **ne correspondent pas** aux arguments de ligne de commande, comme dans l’exemple suivant :

```output
wav_path    str_text
C:\batch_wav_output\wav_1.wav    Sample text to synthesize.
C:\batch_wav_output\wav_2.wav    Using the Speech CLI to run batch-synthesis.
C:\batch_wav_output\wav_3.wav    Some more text to test capabilities.
```

Vous pouvez remplacer ces noms de champs par les bons arguments selon la syntaxe suivante dans l’appel `--foreach`. Il s’agit du même appel que ci-dessus.

```shell
spx synthesize --foreach audio.output;text in @C:\your\path\to\text_synthesis.tsv
```

## <a name="next-steps"></a>Étapes suivantes

* Suivez les guides de démarrage rapide de [reconnaissance vocale](get-started-speech-to-text.md?pivots=programmer-tool-spx) et de [synthèse vocale](get-started-text-to-speech.md?pivots=programmer-tool-spx) à l’aide de l’interface CLI Speech.
