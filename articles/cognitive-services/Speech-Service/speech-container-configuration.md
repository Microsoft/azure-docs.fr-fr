---
title: Configurer des conteneurs Speech
titleSuffix: Azure Cognitive Services
description: Le service Speech fournit pour chaque conteneur un framework de configuration commun qui vous permet de configurer et de gérer facilement les paramètres de stockage, de journalisation, de télémétrie et de sécurité de vos conteneurs.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: aahi
ms.openlocfilehash: e65bb7c7d8fc04baec6b50a53519e689e748fbe1
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012167"
---
# <a name="configure-speech-service-containers"></a>Configurer des conteneurs Speech

Les conteneurs Speech permettent aux clients de créer une architecture d’application Speech optimisée pour tirer parti des fonctionnalités robustes du cloud et de la localité en périphérie. Les cinq conteneurs Speech que nous prenons en charge sont désormais la **reconnaissance vocale**, la **reconnaissance vocale personnalisée**, la **synthèse vocale**, la **synthèse vocale neuronale** et la **synthèse vocale personnalisée**.

L’environnement d’exécution de conteneur **Speech** est configuré à l’aide des arguments de la commande `docker run`. Ce conteneur a plusieurs paramètres obligatoires et quelques paramètres facultatifs. Plusieurs [exemples](#example-docker-run-commands) de commande sont disponibles. Les paramètres propres aux conteneurs correspondent aux paramètres de facturation.

## <a name="configuration-settings"></a>Paramètres de configuration

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> Les paramètres [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) et [`Eula`](#eula-setting) sont utilisés conjointement, et vous devez fournir des valeurs valides pour les trois ; à défaut, votre conteneur ne démarrera pas. Pour plus d’informations sur l’instanciation d’un conteneur à l’aide de ces paramètres de configuration, consultez [Facturation](speech-container-howto.md#billing).

## <a name="apikey-configuration-setting"></a>Paramètre de configuration ApiKey

Le paramètre `ApiKey` spécifie la clé de ressource Azure utilisée pour effectuer le suivi des informations de facturation pour le conteneur. Vous devez spécifier une valeur pour ApiKey, et il doit s’agir d’une clé valide pour la ressource _Speech_ spécifiée pour le paramètre de configuration [`Billing`](#billing-configuration-setting).

Vous trouverez ce paramètre à l’emplacement suivant :

- Portail Azure : Gestion des ressources de **Speech**, sous **Clés**

## <a name="applicationinsights-setting"></a>Paramètre ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Paramètre de configuration Billing

Le paramètre `Billing` permet de spécifier l’URI de point de terminaison de la ressource _Speech_ sur Azure servant à mesurer les informations de facturation du conteneur. Vous devez affecter à ce paramètre de configuration une valeur correspondant à un URI de point de terminaison valide pour une ressource _Speech_ sur Azure. Le conteneur crée des rapports sur l’utilisation toutes les 10 à 15 minutes.

Vous trouverez ce paramètre à l’emplacement suivant :

- Portail Azure : Vue d’ensemble de **Speech**, étiqueté `Endpoint`

| Obligatoire | Nom | Type de données | Description |
| -------- | ---- | --------- | ----------- |
| Oui | `Billing` | String | URI de point de terminaison de facturation. Pour plus d’informations sur la façon d’obtenir l’URI de facturation, consultez [Collecte des paramètres requis](speech-container-howto.md#gathering-required-parameters). Pour obtenir plus d’informations et une liste complète des points de terminaison régionaux, consultez [Noms de sous-domaines personnalisés pour Cognitive Services](../cognitive-services-custom-subdomains.md). |

## <a name="eula-setting"></a>Paramètre Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Paramètres Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Paramètres des informations d’identification du proxy HTTP

[!INCLUDE [Container shared HTTP proxy settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Paramètres de journalisation

[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Paramètres de montage

Utilisez des montages de liaisons pour lire et écrire des données vers et à partir du conteneur. Vous pouvez spécifier un montage d’entrée ou de sortie en spécifiant l’option `--mount` dans la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Les conteneurs Speech standard n’utilisent pas de montage d’entrée ou de sortie pour stocker des données d’entraînement ou de service. Toutefois, les conteneurs de reconnaissance vocale personnalisée s’appuient sur des montages de volume.

La syntaxe exacte de l’emplacement de montage d’hôte varie en fonction du système d’exploitation hôte. De plus, l’emplacement de montage de l’[ordinateur hôte](speech-container-howto.md#the-host-computer) peut ne pas être accessible en raison d’un conflit entre les autorisations utilisées par le compte de service docker et les autorisations de l’emplacement de montage de l’hôte.

| Facultatif | Nom | Type de données | Description |
| -------- | ---- | --------- | ----------- |
| Non autorisé | `Input` | String | Les conteneurs Speech standard n’utilisent pas cet élément. Toutefois, les conteneurs de reconnaissance vocale personnalisée utilisent des [montages de volume](#volume-mount-settings).                                                                                    |
| Facultatif | `Output` | String | Cible du montage de sortie. La valeur par défaut est `/output`. Il s’agit de l’emplacement des journaux d’activité. Les journaux d’activité de conteneur sont inclus. <br><br>Exemple :<br>`--mount type=bind,src=c:\output,target=/output` |

## <a name="volume-mount-settings"></a>Paramètres de montage de volume

Les conteneurs de reconnaissance vocale personnalisée utilisent des [montages de volume](https://docs.docker.com/storage/volumes/) pour conserver les modèles personnalisés. Vous pouvez spécifier un montage de volume en ajoutant l’option `-v` (ou `--volume`) à la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Les modèles personnalisés sont téléchargés la première fois qu’un nouveau modèle est ingéré dans le cadre de la commande docker run de conteneur de reconnaissance vocale personnalisée. Les exécutions séquentielles du même `ModelId` pour un conteneur de reconnaissance vocale utilise le modèle téléchargé précédemment. Si le montage de volume n’est pas fourni, les modèles personnalisés ne peuvent pas être persistants.

Le paramètre de montage de volume est constitué de trois champs de couleur `:` séparés :

1. Le premier champ est le nom du volume sur l’ordinateur hôte, par exemple _C:\Input_.
2. Le deuxième champ est le répertoire sur le conteneur, par exemple _/usr/local/Models_.
3. Le troisième champ (facultatif) est une liste d’options séparées par des virgules. Pour plus d’informations, consultez [Utiliser des volumes](https://docs.docker.com/storage/volumes/).

### <a name="volume-mount-example"></a>Exemple de montage de volume

```bash
-v C:\input:/usr/local/models
```

Cette commande monte le répertoire _C:\Input_ de l’ordinateur hôte sur le répertoire _/usr/local/Models_ des conteneurs.

> [!IMPORTANT]
> Les paramètres de montage de volume s’appliquent uniquement aux conteneurs de **reconnaissance vocale personnalisée** et de **synthèse vocale personnalisée**. Les conteneurs de **reconnaissance vocale**, de **synthèse vocale neuronale** et de **synthèse vocale** n’utilisent pas de montages de volume.

## <a name="example-docker-run-commands"></a>Exemples de commandes docker run

Les exemples suivants utilisent les paramètres de configuration pour illustrer comment écrire et utiliser des commandes `docker run`. Une fois en cours d’exécution, le conteneur continue à s’exécuter jusqu’à ce que vous l’[arrêtiez](speech-container-howto.md#stop-the-container).

- **Caractère de continuation de ligne** : Les commandes Docker dans les sections suivantes utilisent la barre oblique inverse, `\`, comme caractère de continuation de ligne. Remplacez-la ou supprimez-la en fonction des exigences de votre système d’exploitation hôte.
- **Ordre des arguments** : Ne changez pas l’ordre des arguments, sauf si vous connaissez les conteneurs Docker.

Remplacez {_argument_name_} par vos propres valeurs :

| Espace réservé | Valeur | Format ou exemple |
| ----------- | ----- | ----------------- |
| **{API_KEY}** | Clé de point de terminaison de la ressource `Speech` dans la page Clés Azure `Speech`.   | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`                                                                                  |
| **{ENDPOINT_URI}** | La valeur de point de terminaison de facturation est disponible dans la page Vue d’ensemble Azure `Speech`. | Pour obtenir des exemples explicites, consultez [Collecte des paramètres requis](speech-container-howto.md#gathering-required-parameters). |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas. Pour plus d'informations, consultez [Facturation](#billing-configuration-setting).
> La valeur ApiKey est la **Clé** de la page Clés des ressources Azure Speech.

## <a name="speech-container-docker-examples"></a>Exemples Docker de conteneur Speech

Les exemples Docker suivants s’appliquent au conteneur Speech.

## <a name="speech-to-text"></a>[Reconnaissance vocale](#tab/stt)

### <a name="basic-example-for-speech-to-text"></a>Exemple de base de reconnaissance vocale

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-speech-to-text"></a>Exemple de journalisation pour la reconnaissance vocale

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-speech-to-text"></a>[Reconnaissance vocale personnalisée](#tab/cstt)

### <a name="basic-example-for-custom-speech-to-text"></a>Exemple de base de reconnaissance vocale personnalisée

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-speech-to-text"></a>Exemple de base de reconnaissance vocale personnalisée

```Docker
docker run --rm -it -p 5000:5000 --memory 4g --cpus 4 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-speech-to-text \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="text-to-speech"></a>[Synthèse vocale](#tab/tss)

### <a name="basic-example-for-text-to-speech"></a>Exemple de base de synthèse vocale

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-text-to-speech"></a>Exemple de journalisation pour la synthèse vocale

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
mcr.microsoft.com/azure-cognitive-services/speechservices/text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="custom-text-to-speech"></a>[Synthèse vocale personnalisée](#tab/ctts)

### <a name="basic-example-for-custom-text-to-speech"></a>Exemple de base de synthèse vocale personnalisée

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-custom-text-to-speech"></a>Exemple de journalisation pour la synthèse vocale personnalisée

```Docker
docker run --rm -it -p 5000:5000 --memory 2g --cpus 1 \
-v {VOLUME_MOUNT}:/usr/local/models \
mcr.microsoft.com/azure-cognitive-services/speechservices/custom-text-to-speech \
ModelId={MODEL_ID} \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="neural-text-to-speech"></a>[Synthèse vocale neuronale](#tab/ntts)

### <a name="basic-example-for-neural-text-to-speech"></a>Exemple de base pour la synthèse vocale neuronale

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-neural-text-to-speech"></a>Exemple de journalisation pour la synthèse vocale neurale

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/neural-text-to-speech \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

## <a name="speech-language-detection"></a>[Détection de langue vocale](#tab/lid)

### <a name="basic-example-for-speech-language-detection"></a>Exemple de base pour la détection des langues vocales

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
```

### <a name="logging-example-for-speech-language-detection"></a>Exemple de journalisation pour la détection des langues vocales

```Docker
docker run --rm -it -p 5000:5000 --memory 12g --cpus 6 \
mcr.microsoft.com/azure-cognitive-services/speechservices/language-detection \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY} \
Logging:Console:LogLevel:Default=Information
```

---

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Guide pratique pour installer et exécuter des conteneurs](speech-container-howto.md).
