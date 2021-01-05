---
title: Installer et exécuter des conteneurs Docker pour les API du service Speech
titleSuffix: Azure Cognitive Services
description: Utilisez les conteneurs Docker pour le service Speech pour effectuer localement la reconnaissance vocale, la transcription, la génération, etc.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: aahi
ms.custom: cog-serv-seo-aug-2020
keywords: local, Docker, conteneur
ms.openlocfilehash: 9ca5229200b39f0a3c68da152f4d89f842d021ca
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95996411"
---
# <a name="install-and-run-docker-containers-for-the-speech-service-apis"></a>Installer et exécuter des conteneurs Docker pour les API du service Speech 

Les conteneurs vous permettent d’exécuter certaines des API du service Speech dans votre propre environnement. Les conteneurs conviennent particulièrement bien à certaines exigences de sécurité et de gouvernance des données. Dans cet article, vous allez apprendre à télécharger, installer et exécuter un conteneur Speech.

Les conteneurs Speech permettent aux clients de créer une architecture d’application Speech optimisée pour tirer parti des fonctionnalités robustes du cloud et de la localité en périphérie. Plusieurs conteneurs sont disponibles ; leurs [tarifs](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) sont identiques à ceux des services Azure Speech basés sur le cloud.


> [!IMPORTANT]
> Les conteneurs Speech suivants sont désormais passés en phase de disponibilité générale :
> * Reconnaissance vocale standard
> * Reconnaissance vocale personnalisée
> * Synthèse vocale standard
> * Synthèse vocale neuronale
>
> Les conteneurs Speech suivants sont en préversion contrôlée.
> * Synthèse vocale personnalisée
> * Détection de langue vocale 
>
> Pour utiliser les conteneurs Speech, vous devez envoyer une demande en ligne et obtenir son approbation. Pour plus d’informations, consultez la section **Demande d’approbation pour l’exécution du conteneur** ci-dessous.

| Conteneur | Fonctionnalités | Latest |
|--|--|--|
| Reconnaissance vocale | Analyse les sentiments et transcrit de façon continue de la parole en temps réel ou des enregistrements audio par lots, avec des résultats intermédiaires.  | 2.7.0 |
| Reconnaissance vocale personnalisée | À l’aide d’un modèle personnalisé issu du [portail Custom Speech](https://speech.microsoft.com/customspeech), transcrit en continu de la parole en temps réel ou des enregistrements audio en texte, avec des résultats intermédiaires. | 2.7.0 |
| Synthèse vocale | Convertit le texte en paroles naturelles par le biais d’une entrée de texte brut ou du langage de balisage SSML (Speech Synthesis Markup Language). | 1.9.0 |
| Synthèse vocale personnalisée | À l’aide d’un modèle personnalisé issu du [portail Custom Voice](https://aka.ms/custom-voice-portal), convertit le texte en paroles naturelles par le biais d’une entrée de texte brut ou du langage de balisage SSML (Speech Synthesis Markup Language). | 1.9.0 |
| Détection de langue vocale | Permet de détecter la langue parlée en fichiers audio. | 1.0 |
| Synthèse vocale neuronale | Convertit du texte en parole naturelle grâce à la technologie de réseau neuronal profond qui permet d’obtenir une parole synthétisée plus naturelle. | 1.3.0 |

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

L’utilisation des conteneurs Speech est soumise aux prérequis suivants :

| Obligatoire | Objectif |
|--|--|
| Moteur Docker | Vous avez besoin d’un moteur Docker installé sur un [ordinateur hôte](#the-host-computer). Docker fournit des packages qui configurent l’environnement Docker sur [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) et [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Pour apprendre les principes de base de Docker et des conteneurs, consultez la [vue d’ensemble de Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Vous devez configurer Docker pour permettre aux conteneurs de se connecter à Azure et de lui envoyer des données de facturation. <br><br> **Sur Windows**, vous devez également configurer Docker pour prendre en charge les conteneurs Linux.<br><br> |
| Bonne connaissance de Docker | Vous devez avoir une compréhension élémentaire des concepts Docker, notamment les registres, référentiels, conteneurs et images conteneurs, ainsi qu’une maîtrise des commandes `docker` de base. |
| Ressource Speech | Pour pouvoir utiliser ces conteneurs, vous devez avoir :<br><br>Une ressource _Speech_ d’Azure afin d’obtenir la clé API et l’URI du point de terminaison associés. Les deux valeurs sont disponibles sur les pages Vue d’ensemble et Clés de la ressource **Speech** du portail Azure. Elles sont nécessaires au démarrage du conteneur.<br><br>**{API_KEY}**  : L’une des deux clés de ressource disponibles à la page **Clés**<br><br>**{ENDPOINT_URI}**  : Le point de terminaison tel qu'il est fourni à la page **Vue d’ensemble** |

[!INCLUDE [Gathering required parameters](../containers/includes/container-gathering-required-parameters.md)]

## <a name="the-host-computer"></a>L’ordinateur hôte

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="advanced-vector-extension-support"></a>Prise en charge d’Advanced Vector Extension

L’**hôte** est l’ordinateur qui exécute le conteneur docker. L’hôte *doit prendre en charge* [Advanced Vector Extensions](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions#CPUs_with_AVX2) (AVX2). Vous pouvez vérifier cette prise en charge AVX2 sur les hôtes Linux avec la commande suivante :

```console
grep -q avx2 /proc/cpuinfo && echo AVX2 supported || echo No AVX2 support detected
```
> [!WARNING]
> L’ordinateur hôte doit *obligatoirement* prendre en charge AVX2. Le conteneur ne fonctionnera *pas* correctement sans prise en charge AVX2.

### <a name="container-requirements-and-recommendations"></a>Exigences et suggestions relatives au conteneur

Le tableau suivant indique l’allocation de ressources minimale et recommandée pour chaque conteneur Speech.

| Conteneur | Minimum | Recommandé |
|-----------|---------|-------------|
| Reconnaissance vocale | 2 cœurs, 2 Go de mémoire | 4 cœurs, 4 Go de mémoire |
| Reconnaissance vocale personnalisée | 2 cœurs, 2 Go de mémoire | 4 cœurs, 4 Go de mémoire |
| Synthèse vocale | 1 cœur, 2 Go de mémoire | 2 cœur, 3 Go de mémoire |
| Synthèse vocale personnalisée | 1 cœur, 2 Go de mémoire | 2 cœur, 3 Go de mémoire |
| Détection de langue vocale | 1 cœur, 1 Go de mémoire | 1 cœur, 1 Go de mémoire |
| Synthèse vocale neuronale | 6 cœurs, 12 Go de mémoire | 8 cœurs, 16 Go de mémoire |

* Chaque cœur doit être cadencé à au moins 2,6 gigahertz (GHz).

Le nombre de cœurs et la quantité de mémoire correspondent aux paramètres `--cpus` et `--memory` qui sont utilisés dans le cadre de la commande `docker run`.

> [!NOTE]
> Les valeurs minimales et recommandées se basent sur les limites Docker, et *non* sur les ressources de la machine hôte. Par exemple, les conteneurs de reconnaissance vocale contenant des portions de mappage de mémoire d’un modèle linguistique volumineux, il est *recommandé* que l’intégralité du fichier tienne dans la mémoire, ce qui correspond à 4 à 6 Go supplémentaires. En outre, la première exécution d’un conteneur peut prendre beaucoup de temps perce que les modèles sont paginés en mémoire.

## <a name="request-approval-to-the-run-the-container"></a>Demande d’approbation pour l’exécution du conteneur

Complétez et envoyez le [formulaire de demande](https://aka.ms/csgate) pour demander l’accès au conteneur. 

[!INCLUDE [Request access to public preview](../../../includes/cognitive-services-containers-request-access.md)]


## <a name="get-the-container-image-with-docker-pull"></a>Obtenir l’image conteneur avec `docker pull`

Les images conteneur pour Speech sont disponibles dans le service Container Registry suivant.

# <a name="speech-to-text"></a>[Reconnaissance vocale](#tab/stt)

| Conteneur | Référentiel |
|-----------|------------|
| Reconnaissance vocale | `mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest` |

# <a name="custom-speech-to-text"></a>[Reconnaissance vocale personnalisée](#tab/cstt)

| Conteneur | Référentiel |
|-----------|------------|
| Reconnaissance vocale personnalisée | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest` |

# <a name="text-to-speech"></a>[Synthèse vocale](#tab/tts)

| Conteneur | Référentiel |
|-----------|------------|
| Synthèse vocale | `mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest` |

# <a name="neural-text-to-speech"></a>[Synthèse vocale neuronale](#tab/ntts)

| Conteneur | Référentiel |
|-----------|------------|
| Synthèse vocale neuronale | `mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest` |

# <a name="custom-text-to-speech"></a>[Synthèse vocale personnalisée](#tab/ctts)

| Conteneur | Référentiel |
|-----------|------------|
| Synthèse vocale personnalisée | `mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech:latest` |

# <a name="speech-language-detection"></a>[Détection de langue vocale](#tab/lid)

> [!TIP]
> Pour obtenir les résultats les plus utiles, nous vous recommandons d’utiliser le conteneur de détection de langue vocale avec les conteneurs de reconnaissance vocale ou de reconnaissance vocale personnalisée. 

| Conteneur | Référentiel |
|-----------|------------|
| Détection de langue vocale | `mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest` |

***

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-speech-containers"></a>Docker pull pour les conteneurs Speech

# <a name="speech-to-text"></a>[Reconnaissance vocale](#tab/stt)

#### <a name="docker-pull-for-the-speech-to-text-container"></a>Docker pull pour le conteneur Reconnaissance vocale

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir du registre de conteneurs Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest
```

> [!IMPORTANT]
> L’étiquette `latest` extrait les paramètres régionaux `en-US`. Pour d’autres paramètres régionaux, consultez [Paramètres régionaux de reconnaissance vocale](#speech-to-text-locales).

#### <a name="speech-to-text-locales"></a>Paramètres régionaux de reconnaissance vocale

Toutes les balises, à l’exception de `latest`, respectent le format suivant et sont sensibles à la casse :

```
<major>.<minor>.<patch>-<platform>-<locale>-<prerelease>
```

La balise suivante illustre le format :

```
2.6.0-amd64-en-us
```

Pour tous les paramètres régionaux pris en charge du conteneur de **reconnaissance vocale**, consultez les [étiquettes d’images de reconnaissance vocale](../containers/container-image-tags.md#speech-to-text).

# <a name="custom-speech-to-text"></a>[Reconnaissance vocale personnalisée](#tab/cstt)

#### <a name="docker-pull-for-the-custom-speech-to-text-container"></a>Docker pull pour le conteneur Reconnaissance vocale personnalisée

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir du registre de conteneurs Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text:latest
```

> [!NOTE]
> Les `locale` et `voice` des conteneurs Speech personnalisés sont déterminés par le modèle personnalisé ingéré par le conteneur.

# <a name="text-to-speech"></a>[Synthèse vocale](#tab/tts)

#### <a name="docker-pull-for-the-text-to-speech-container"></a>Docker pull pour le conteneur Synthèse vocale

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir du registre de conteneurs Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech:latest
```

> [!IMPORTANT]
> La balise `latest` extrait les paramètres régionaux `en-US` et la voix `ariarus`. Pour d’autres paramètres régionaux, consultez [Paramètres régionaux de synthèse vocale](#text-to-speech-locales).

#### <a name="text-to-speech-locales"></a>Paramètres régionaux de synthèse vocale

Toutes les balises, à l’exception de `latest`, respectent le format suivant et sont sensibles à la casse :

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>-<prerelease>
```

La balise suivante illustre le format :

```
1.8.0-amd64-en-us-ariarus
```

Pour tous les paramètres régionaux et voix correspondantes pris en charge du conteneur de **synthèse vocale**, consultez les [étiquettes d’images de synthèse vocale](../containers/container-image-tags.md#text-to-speech).

> [!IMPORTANT]
> Lors de la construction d’une requête HTTP POST de *synthèse vocale*, le message [SSML (Speech Synthesis Markup Language)](speech-synthesis-markup.md) nécessite un élément `voice` avec un attribut `name`. La valeur correspond aux paramètres régionaux et à la voix du conteneur correspondant, également appelés [« nom court »](language-support.md#standard-voices). Par exemple, le nom de la voix pour la balise `latest` est `en-US-AriaRUS`.

# <a name="neural-text-to-speech"></a>[Synthèse vocale neuronale](#tab/ntts)

#### <a name="docker-pull-for-the-neural-text-to-speech-container"></a>Docker pull pour le conteneur de synthèse vocale neuronale

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir du registre de conteneurs Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech:latest
```

> [!IMPORTANT]
> La balise `latest` extrait les paramètres régionaux `en-US` et la voix `arianeural`. Pour d’autres paramètres régionaux, consultez [Paramètres régionaux de synthèse vocale neuronale](#neural-text-to-speech-locales).

#### <a name="neural-text-to-speech-locales"></a>Paramètres régionaux de synthèse vocale neuronale

Toutes les balises, à l’exception de `latest`, respectent le format suivant et sont sensibles à la casse :

```
<major>.<minor>.<patch>-<platform>-<locale>-<voice>
```

La balise suivante illustre le format :

```
1.3.0-amd64-en-us-arianeural
```

Pour tous les paramètres régionaux et les voix correspondantes pris en charge du conteneur de **synthèse vocale neuronale**, consultez les [étiquettes d’images de synthèse vocale neuronale](../containers/container-image-tags.md#neural-text-to-speech).

> [!IMPORTANT]
> Lors de la construction d’une requête HTTP POST de *synthèse vocale neuronale*, le message [SSML (Speech Synthesis Markup Language)](speech-synthesis-markup.md) nécessite un élément `voice` avec un attribut `name`. La valeur correspond aux paramètres régionaux et à la voix du conteneur correspondant, également appelés [« nom court »](language-support.md#neural-voices). Par exemple, le nom de la voix pour la balise `latest` est `en-US-AriaNeural`.

# <a name="custom-text-to-speech"></a>[Synthèse vocale personnalisée](#tab/ctts)

#### <a name="docker-pull-for-the-custom-text-to-speech-container"></a>Docker pull pour le conteneur Synthèse vocale personnalisée

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir du registre de conteneurs Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech:latest
```

> [!NOTE]
> Les `locale` et `voice` des conteneurs Speech personnalisés sont déterminés par le modèle personnalisé ingéré par le conteneur.

# <a name="speech-language-detection"></a>[Détection de langue vocale](#tab/lid)

#### <a name="docker-pull-for-the-speech-language-detection-container"></a>Docker pull du conteneur Détection de langue vocale

Utilisez la commande [docker pull](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir du registre de conteneurs Microsoft.

```Docker
docker pull mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection:latest
```

***

## <a name="how-to-use-the-container"></a>Comment utiliser le conteneur

Une fois que le conteneur est sur l’[ordinateur hôte](#the-host-computer), appliquez la procédure suivante pour travailler avec le conteneur.

1. [Exécutez le conteneur](#run-the-container-with-docker-run) avec les paramètres de facturation requis. D’autres [exemples](speech-container-configuration.md#example-docker-run-commands) de commande `docker run` sont disponibles.
1. [Interrogez le point de terminaison de prédiction du conteneur](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-with-docker-run"></a>Exécuter le conteneur avec `docker run`

Utilisez la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/) pour exécuter le conteneur. Pour plus d’informations sur la façon d’obtenir les valeurs `{Endpoint_URI}` et `{API_Key}`, consultez [Collecte des paramètres requis](#gathering-required-parameters). Des [exemples](speech-container-configuration.md#example-docker-run-commands) supplémentaires de la commande `docker run` sont également disponibles.

# <a name="speech-to-text"></a>[Reconnaissance vocale](#tab/stt)

Pour exécuter le conteneur de *reconnaissance vocale* standard, exécutez la commande `docker run` suivante.

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur *Reconnaissance vocale* à partir de l’image conteneur.
* Alloue 4 cœurs de processeur et 4 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.


#### <a name="analyze-sentiment-on-the-speech-to-text-output"></a>Analyser les sentiments sur la sortie de la reconnaissance vocale 
À partir de la version v2.6.0 du conteneur de reconnaissance vocale, vous devez utiliser le point de terminaison de l’API Analyse de texte 3.0 au lieu de la préversion. Par exemple
* `https://westus2.api.cognitive.microsoft.com/text/analytics/v3.0/sentiment`
* `https://localhost:5000/text/analytics/v3.0/sentiment`

> [!NOTE]
> L’API Analyse de texte `v3.0` n’est pas à compatibilité descendante avec Analyse de texte `v3.0-preview.1`. Pour bénéficier de la prise en charge des dernières fonctionnalités de sentiment, utilisez la version `v2.6.0` de l’image conteneur Reconnaissance vocale et Analyse de texte `v3.0`.

À compter de la version  2.2.0 du conteneur de reconnaissance vocale, vous pouvez appeler l’[API Analyse de sentiments v3](../text-analytics/how-tos/text-analytics-how-to-sentiment-analysis.md) sur la sortie. Pour appeler l’analyse des sentiments, vous avez besoin d’un point de terminaison de ressource de l’API Analyse de texte. Par exemple : 
* `https://westus2.api.cognitive.microsoft.com/text/analytics/v3.0-preview.1/sentiment`
* `https://localhost:5000/text/analytics/v3.0-preview.1/sentiment`

Si vous accédez à un point de terminaison Analyse de texte dans le cloud, vous devez disposer d’une clé. Si vous exécutez l’Analyse de texte localement, il est possible que vous n’ayez pas à la fournir.

La clé et le point de terminaison sont passés au conteneur Speech comme arguments, comme dans l’exemple suivant.

```bash
docker run -it --rm -p 5000:5000 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text:latest \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
CloudAI:SentimentAnalysisSettings:TextAnalyticsHost={TEXT_ANALYTICS_HOST} \
CloudAI:SentimentAnalysisSettings:SentimentAnalysisApiKey={SENTIMENT_APIKEY}
```

Cette commande :

* Effectue les mêmes étapes que la commande ci-dessus.
* Stocke un point de terminaison et une clé d’API Analyse de texte pour l’envoi de demandes d’analyse de sentiments. 

#### <a name="phraselist-v2-on-the-speech-to-text-output"></a>Liste d’expressions v2 sur la sortie de reconnaissance vocale 

À partir de la version v2.6.0 du conteneur de reconnaissance vocale, vous pouvez obtenir la sortie avec vos propres expressions, soit la phrase entière, soit des expressions au milieu. Prenons l’exemple de *l’homme grand* dans la phrase suivante :

* « Voici une phrase **l’homme grand** voici une autre phrase ».

Pour configurer une liste d’expressions, vous avez besoin d’ajouter vos propres expressions quand vous effectuez l’appel. Exemple :

```python
    phrase="the tall man"
    recognizer = speechsdk.SpeechRecognizer(
        speech_config=dict_speech_config,
        audio_config=audio_config)
    phrase_list_grammer = speechsdk.PhraseListGrammar.from_recognizer(recognizer)
    phrase_list_grammer.addPhrase(phrase)

```

Si vous avez plusieurs expressions à ajouter, appelez `.addPhrase()` pour chaque expression pour l’ajouter à la liste d’expressions. 


# <a name="custom-speech-to-text"></a>[Reconnaissance vocale personnalisée](#tab/cstt)

Le conteneur *Reconnaissance vocale personnalisée* s’appuie sur un modèle Speech personnalisé. Le modèle personnalisé doit avoir été [entraîné](how-to-custom-speech-train-model.md) à l’aide du [portail Custom Speech](https://speech.microsoft.com/customspeech).

> [!IMPORTANT]
> Le modèle Custom Speech doit être formé à partir de l’une des versions de modèle suivantes :
> * **20181201 (v3.3 unifiée)**
> * **20190520 (v4.14 unifiée)**
> * **20190701 (v4.17 unifiée)**<br>
> ![Modèle de conteneur formé à Custom Speech](media/custom-speech/custom-speech-train-model-container-scoped.png)

L’**ID du modèle** Custom Speech est nécessaire pour exécuter le conteneur. Il figure dans la page **Entraînement** du portail Custom Speech. À partir du portail Custom Speech, accédez à la page **Entraînement** et sélectionnez le modèle.
<br>

![Page Entraînement de Custom Speech](media/custom-speech/custom-speech-model-training.png)

Obtenez l’**ID du modèle** pour l’utiliser comme argument du paramètre `ModelId` de la commande `docker run`.
<br>

![Détails du modèle Custom Speech](media/custom-speech/custom-speech-model-details.png)

Le tableau suivant présente les différents paramètres `docker run` et leurs descriptions correspondantes :

| Paramètre | Description |
|---------|---------|
| `{VOLUME_MOUNT}` | [Montage de volume](https://docs.docker.com/storage/volumes/) de l’ordinateur hôte, que Docker utilise pour rendre le modèle personnalisé persistant. Par exemple, *C:\CustomSpeech* où le *lecteur C* se trouve sur l’ordinateur hôte. |
| `{MODEL_ID}` | **ID de modèle** Custom Speech issu de la page **Entraînement** du portail Custom Speech. |
| `{ENDPOINT_URI}` | Le point de terminaison est nécessaire pour le comptage et la facturation. Pour plus d’informations, consultez [Collecte des paramètres requis](#gathering-required-parameters). |
| `{API_KEY}` | La clé API est obligatoire. Pour plus d’informations, consultez [Collecte des paramètres requis](#gathering-required-parameters). |

Pour exécuter le conteneur *Reconnaissance vocale personnalisée*, exécutez la commande `docker run` suivante :

```bash
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur *Reconnaissance vocale personnalisée* à partir de l’image conteneur.
* Alloue 4 cœurs de processeur et 4 gigaoctets (Go) de mémoire.
* Charge le modèle *Reconnaissance vocale personnalisée* à partir du montage d’entrée de volume, par exemple *C:\CustomSpeech*.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Télécharge le modèle selon `ModelId` (s’il est introuvable sur le montage de volume).
* Si le modèle personnalisé a été téléchargé auparavant, `ModelId` est ignoré.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.


#### <a name="base-model-download-on-the-custom-speech-to-text-container"></a>Téléchargement des modèles de base sur le conteneur de reconnaissance vocale personnalisée  
À partir de la version v2.6.0 du conteneur de reconnaissance vocale personnalisée, vous pouvez obtenir les informations des modèles de base disponibles à l’aide de l’option `BaseModelLocale=<locale>`. Cette option vous donne la liste des modèles de base disponibles sur ces paramètres régionaux sous votre compte de facturation. Exemple :

```bash
docker run --rm -it \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
BaseModelLocale={LOCALE} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur *Reconnaissance vocale personnalisée* à partir de l’image conteneur.
* Vérifiez et renvoyez les modèles de base disponibles des paramètres régionaux cibles.

La sortie vous donne la liste des modèles de base avec les informations suivantes : paramètres régionaux, ID de modèle et date et heure de création. Vous pouvez utiliser l’ID de modèle pour télécharger et utiliser le modèle de base spécifique que vous préférez. Exemple :
```
Checking available base model for en-us
2020/10/30 21:54:20 [Info] Searching available base models for en-us
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2016-11-04T08:23:42Z, Id: a3d8aab9-6f36-44cd-9904-b37389ce2bfa
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2016-11-04T12:01:02Z, Id: cc7826ac-5355-471d-9bc6-a54673d06e45
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2017-08-17T12:00:00Z, Id: a1f8db59-40ff-4f0e-b011-37629c3a1a53
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-04-16T11:55:00Z, Id: c7a69da3-27de-4a4b-ab75-b6716f6321e5
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-09-21T15:18:43Z, Id: da494a53-0dad-4158-b15f-8f9daca7a412
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-10-19T11:28:54Z, Id: 84ec130b-d047-44bf-a46d-58c1ac292ca7
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-11-26T07:59:09Z, Id: ee5c100f-152f-4ae5-9e9d-014af3c01c56
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2018-11-26T09:21:55Z, Id: d04959a6-71da-4913-9997-836793e3c115
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-01-11T10:04:19Z, Id: 488e5f23-8bc5-46f8-9ad8-ea9a49a8efda
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-02-18T14:37:57Z, Id: 0207b3e6-92a8-4363-8c0e-361114cdd719
2020/10/30 21:54:21 [Info] [Base model] Locale: en-us, CreatedDate: 2019-03-03T17:34:10Z, Id: 198d9b79-2950-4609-b6ec-f52254074a05
2020/10/30 21:54:21 [Fatal] Please run this tool again and assign --modelId '<one above base model id>'. If no model id listed above, it means currently there is no available base model for en-us
```

#### <a name="custom-pronunciation-on-the-custom-speech-to-text-container"></a>Prononciation personnalisée sur le conteneur de reconnaissance vocale personnalisée 
À partir de la version v2.5.0 du conteneur de reconnaissance vocale personnalisée, vous pouvez obtenir un résultat de prononciation personnalisée dans la sortie. Il vous suffit d’avoir vos propres règles de prononciation personnalisées configurées dans votre modèle personnalisé et de monter le modèle sur un conteneur de reconnaissance vocale personnalisée.


# <a name="text-to-speech"></a>[Synthèse vocale](#tab/tts)

Pour exécuter le conteneur de *synthèse vocale* standard, exécutez la commande `docker run` suivante.

```bash
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur de *synthèse vocale* standard à partir de l’image conteneur.
* Alloue 1 cœur de processeur et 2 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

# <a name="neural-text-to-speech"></a>[Synthèse vocale neuronale](#tab/ntts)

Pour exécuter le conteneur *Synthèse vocale neuronale*, exécutez la commande `docker run` suivante.

```bash
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur *Synthèse vocale neuronale* à partir de l’image conteneur.
* Alloue 6 cœurs de processeur et 12 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

# <a name="custom-text-to-speech"></a>[Synthèse vocale personnalisée](#tab/ctts)

Le conteneur *Synthèse vocale personnalisée* s’appuie sur un modèle vocal personnalisé. Le modèle personnalisé doit avoir été [entraîné](how-to-custom-voice-create-voice.md) à l’aide du [portail Custom Voice](https://aka.ms/custom-voice-portal). L’**ID du modèle** Custom Voice est nécessaire pour exécuter le conteneur. Il figure dans la page **Entraînement** du portail Custom Voice. À partir du portail Custom Voice, accédez à la page **Entraînement** et sélectionnez le modèle.
<br>

![Page Entraînement de Custom Voice](media/custom-voice/custom-voice-model-training.png)

Obtenez l’**ID du modèle** pour l’utiliser comme argument du paramètre `ModelId` de la commande docker run.
<br>

![Détails du modèle Custom Voice](media/custom-voice/custom-voice-model-details.png)

Le tableau suivant présente les différents paramètres `docker run` et leurs descriptions correspondantes :

| Paramètre | Description |
|---------|---------|
| `{VOLUME_MOUNT}` | [Montage de volume](https://docs.docker.com/storage/volumes/) de l’ordinateur hôte, que Docker utilise pour rendre le modèle personnalisé persistant. Par exemple, *C:\CustomSpeech* où le *lecteur C* se trouve sur l’ordinateur hôte. |
| `{MODEL_ID}` | **ID de modèle** Custom Voice issu de la page **Entraînement** du portail Custom Voice. |
| `{ENDPOINT_URI}` | Le point de terminaison est nécessaire pour le comptage et la facturation. Pour plus d’informations, consultez [Collecte des paramètres requis](#gathering-required-parameters). |
| `{API_KEY}` | La clé API est obligatoire. Pour plus d’informations, consultez [Collecte des paramètres requis](#gathering-required-parameters). |

Pour exécuter le conteneur *Synthèse vocale personnalisée*, exécutez la commande `docker run` suivante :

```bash
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande :

* Exécute un conteneur *Synthèse vocale personnalisée* à partir de l’image conteneur.
* Alloue 1 cœur de processeur et 2 gigaoctets (Go) de mémoire.
* Charge le modèle *Synthèse vocale personnalisée* à partir du montage d’entrée de volume, par exemple *C:\CustomVoice*.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Télécharge le modèle selon `ModelId` (s’il est introuvable sur le montage de volume).
* Si le modèle personnalisé a été téléchargé auparavant, `ModelId` est ignoré.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

# <a name="speech-language-detection"></a>[Détection de langue vocale](#tab/lid)

Pour exécuter le conteneur *Détection de langue vocale*, exécutez la commande `docker run` suivante.

```bash
docker run --rm -it -p 5003:5003 --memory 1g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

Cette commande : 

* Exécute un conteneur de détection de langue vocale à partir de l’image conteneur. Actuellement, vous n’êtes pas facturé pour l’exécution de cette image.
* Alloue 1 cœur de processeur et 1 gigaoctet (Go) de mémoire.
* Expose le port TCP 5003 et alloue un pseudo-TTY pour le conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

Si vous envoyez uniquement des requêtes Détection de langue Speech, vous devrez définir la valeur `phraseDetection` du client Speech sur `None`.  

```python
speech_config.set_service_property(
      name='speechcontext-phraseDetection.Mode',
      value='None',
      channel=speechsdk.ServicePropertyChannel.UriQueryParameter
   )
```

Si vous souhaitez exécuter ce conteneur avec le conteneur de reconnaissance vocale, vous pouvez utiliser cette [image Docker](https://hub.docker.com/r/antsu/on-prem-client). Après le démarrage des deux conteneurs, utilisez cette commande Docker Run pour exécuter `speech-to-text-with-languagedetection-client`.

```Docker
docker run --rm -v ${HOME}:/root -ti antsu/on-prem-client:latest ./speech-to-text-with-languagedetection-client ./audio/LanguageDetection_en-us.wav --host localhost --lport 5003 --sport 5000
```

> [!NOTE]
> Une augmentation du nombre d’appels simultanés peut avoir un impact sur la fiabilité et la latence. Pour la détection de la langue, nous vous recommandons de ne pas dépasser 4 appels simultanés avec 1 processeur et 1 Go de mémoire. Pour des hôtes dotés de 2 processeurs et de 2 Go de mémoire, nous recommandons un maximum de 6 appels simultanés.

***

> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas.  Pour plus d'informations, consultez [Facturation](#billing).

## <a name="query-the-containers-prediction-endpoint"></a>Interroger le point de terminaison de prédiction du conteneur

> [!NOTE]
> Utilisez un numéro de port unique si vous exécutez plusieurs conteneurs.

| Containers | URL de l’hôte du SDK | Protocol |
|--|--|--|
| Reconnaissance vocale standard et reconnaissance vocale personnalisée | `ws://localhost:5000` | WS |
| Synthèse vocale (standard, personnalisée et neuronale), détection de langue vocale | `http://localhost:5000` | HTTP |

Pour plus d’informations sur l’utilisation des protocoles WSS et HTTPS, consultez [Sécurité des conteneurs](../cognitive-services-container-support.md#azure-cognitive-services-container-security).

### <a name="speech-to-text-standard-and-custom"></a>Reconnaissance vocale (standard et personnalisée)

[!INCLUDE [Query Speech-to-text container endpoint](includes/speech-to-text-container-query-endpoint.md)]

#### <a name="analyze-sentiment"></a>Analyser les sentiments

Si vous avez fourni vos informations d’identification d’API Analyse de texte [au conteneur](#analyze-sentiment-on-the-speech-to-text-output), vous pouvez utiliser le SDK Speech pour envoyer des demandes de reconnaissance vocale avec l’analyse des sentiments. Vous pouvez configurer les réponses de l’API pour qu’elles utilisent un format *simple* ou *détaillé*.
> [!NOTE]
> La version v1.13 du kit de développement logiciel (SDK) Python du service de reconnaissance vocale a identifié un problème avec l’analyse des sentiments. Utilisez la version v1.12.x ou une version antérieure si vous utilisez l’analyse des sentiments dans le Kit de développement logiciel (SDK) Python du service de reconnaissance vocale.

# <a name="simple-format"></a>[Format simple](#tab/simple-format)

Pour configurer le client Speech de sorte qu’il utilise un format simple, ajoutez `"Sentiment"` comme valeur pour `Simple.Extensions`. Si vous voulez choisir une version de modèle d’Analyse de texte spécifique, remplacez `'latest'` dans la configuration de la propriété `speechcontext-phraseDetection.sentimentAnalysis.modelversion`.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Simple.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Simple.Extensions` retourne le résultat des sentiments dans la couche racine de la réponse.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6098574b79434bd4849fee7e0a50f22e",
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

# <a name="detailed-format"></a>[Format détaillé](#tab/detailed-format)

Pour configurer le client Speech de sorte qu’il utilise un format détaillé, ajoutez `"Sentiment"` comme valeur pour `Detailed.Extensions`, `Detailed.Options`, ou les deux. Si vous voulez choisir une version de modèle d’Analyse de texte spécifique, remplacez `'latest'` dans la configuration de la propriété `speechcontext-phraseDetection.sentimentAnalysis.modelversion`.

```python
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Options',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-PhraseOutput.Detailed.Extensions',
    value='["Sentiment"]',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentAnalysis.modelversion',
    value='latest',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

`Detailed.Extensions` fournit le résultat des sentiments dans la couche racine de la réponse. `Detailed.Options` fournit le résultat dans la couche `NBest` de la réponse. Ils peuvent être utilisés séparément ou ensemble.

```json
{
   "DisplayText":"What's the weather like?",
   "Duration":13000000,
   "Id":"6a2aac009b9743d8a47794f3e81f7963",
   "NBest":[
      {
         "Confidence":0.973695,
         "Display":"What's the weather like?",
         "ITN":"what's the weather like",
         "Lexical":"what's the weather like",
         "MaskedITN":"What's the weather like",
         "Sentiment":{
            "Negative":0.03,
            "Neutral":0.79,
            "Positive":0.18
         }
      },
      {
         "Confidence":0.9164971,
         "Display":"What is the weather like?",
         "ITN":"what is the weather like",
         "Lexical":"what is the weather like",
         "MaskedITN":"What is the weather like",
         "Sentiment":{
            "Negative":0.02,
            "Neutral":0.88,
            "Positive":0.1
         }
      }
   ],
   "Offset":4700000,
   "RecognitionStatus":"Success",
   "Sentiment":{
      "Negative":0.03,
      "Neutral":0.79,
      "Positive":0.18
   }
}
```

---

Si vous voulez désactiver complètement l’analyse des sentiments, ajoutez une valeur de `false` à `sentimentanalysis.enabled`.

```python
speech_config.set_service_property(
    name='speechcontext-phraseDetection.sentimentanalysis.enabled',
    value='false',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

### <a name="text-to-speech-standard-neural-and-custom"></a>Synthèse vocale (standard, neuronale et personnalisée)

[!INCLUDE [Query Text-to-speech container endpoint](includes/text-to-speech-container-query-endpoint.md)]

### <a name="run-multiple-containers-on-the-same-host"></a>Exécuter plusieurs conteneurs sur le même hôte

Si vous envisagez d’exécuter plusieurs conteneurs avec les ports exposés, veillez à exécuter chaque conteneur avec un port exposé différent. Par exemple, exécutez le premier conteneur sur le port 5000 et le second conteneur sur le port 5001.

Vous pouvez avoir ce conteneur, et un autre conteneur Azure Cognitive Services qui s’exécutent ensemble sur l’hôte. Vous pouvez également disposer de plusieurs conteneurs du même conteneur Cognitive Services en cours d’exécution.

[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Arrêter le conteneur

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Dépannage

Au démarrage ou pendant l’exécution du conteneur, vous pouvez rencontrer des problèmes. Utilisez une [montage](speech-container-configuration.md#mount-settings) de sortie et activez la journalisation. Ainsi, le conteneur pourra générer des fichiers journaux utiles à la résolution des problèmes.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturation

Les conteneurs Speech envoient des informations de facturation à Azure à l’aide d’une ressource *Speech* sur votre compte Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Pour plus d’informations sur ces options, consultez [Configurer des conteneurs](speech-container-configuration.md).

<!--blogs/samples/video courses -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert des concepts et le flux de travail pour le téléchargement, l’installation et l’exécution des conteneurs Speech. En résumé :

* Speech fournit quatre conteneurs Linux pour Docker, qui encapsulent diverses fonctionnalités :
  * *Reconnaissance vocale*
  * *Reconnaissance vocale personnalisée*
  * *Synthèse vocale*
  * *Synthèse vocale personnalisée*
  * *Synthèse vocale neuronale*
  * *Détection de langue vocale*
* Les images conteneur sont téléchargées à partir du registre de conteneurs dans Azure.
* Les images conteneurs s’exécutent dans Docker.
* Que vous utilisiez l'API REST (synthèse vocale uniquement) ou le kit SDK (reconnaissance vocale ou synthèse vocale), vous spécifiez l'URI de l'hôte du conteneur. 
* Vous devez spécifier les informations de facturation lors de l’instanciation d’un conteneur.

> [!IMPORTANT]
>  Les conteneurs Cognitives Services ne sont pas concédés sous licence pour s’exécuter sans être connectés à Azure pour le contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au service de contrôle à tout moment. Les conteneurs Cognitive Services n’envoient pas les données des clients (p. ex., l’image ou le texte analysés) à Microsoft.

## <a name="next-steps"></a>Étapes suivantes

* Passer en revue [Configurer des conteneurs](speech-container-configuration.md) pour découvrir les paramètres de configuration
* Découvrir comment [utiliser les conteneurs du service Speech avec Kubernetes et Helm](speech-container-howto-on-premises.md)
* Utiliser davantage de [conteneurs Cognitive Services](../cognitive-services-container-support.md)
