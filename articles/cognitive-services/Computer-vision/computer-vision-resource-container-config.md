---
title: Configurer des conteneurs OCR Read - Vision par ordinateur
titleSuffix: Azure Cognitive Services
description: Cet article explique comment configurer les paramètres obligatoires et facultatifs pour les conteneurs OCR Read dans Vision par ordinateur.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 11/23/2020
ms.author: aahi
ms.custom: seodec18
ms.openlocfilehash: 0539f37fe15f68d8bfd47bf426333f9d5c67c37d
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96006868"
---
# <a name="configure-read-ocr-docker-containers"></a>Configurer des conteneurs Docker OCR Read

Configurez l'environnement d'exécution du conteneur OCR Read de Vision par ordinateur à l'aide des arguments de la commande `docker run`. Ce conteneur a plusieurs paramètres obligatoires et quelques paramètres facultatifs. Plusieurs [exemples](#example-docker-run-commands) de commande sont disponibles. Les paramètres propres aux conteneurs correspondent aux paramètres de facturation. 

## <a name="configuration-settings"></a>Paramètres de configuration

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> Les paramètres [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) et [`Eula`](#eula-setting) sont utilisés conjointement, et vous devez fournir des valeurs valides pour les trois ; à défaut, votre conteneur ne démarrera pas. Pour plus d’informations sur l’instanciation d’un conteneur à l’aide de ces paramètres de configuration, consultez [Facturation](computer-vision-how-to-install-containers.md).

Les paramètres de configuration spécifiques au conteneur sont les suivants :

|Obligatoire|Paramètre|Objectif|
|--|--|--|
|Non|ReadEngineConfig:ResultExpirationPeriod| Conteneurs v2.0 uniquement. Période d’expiration du résultat, en heures. L'intervalle par défaut est de 48 heures. Le paramètre spécifie à quel moment le système doit effacer les résultats de la reconnaissance. Par exemple, si `resultExpirationPeriod=1`, le système efface le résultat de la reconnaissance 1 heure après le processus. Si `resultExpirationPeriod=0`, le système efface le résultat de la reconnaissance après récupération du résultat.|
|Non|Cache:Redis| Conteneurs v2.0 uniquement. Active le stockage Redis pour le stockage des résultats. Un cache est *obligatoire* si plusieurs conteneurs de lecture sont placés derrière un équilibreur de charge.|
|Non|Queue:RabbitMQ|Conteneurs v2.0 uniquement. Active RabbitMQ pour la répartition des tâches. Ce paramètre est utile lorsque plusieurs conteneurs de lecture sont placés derrière un équilibreur de charge.|
|Non|Queue:Azure:QueueVisibilityTimeoutInMilliseconds | Conteneurs v3.x uniquement. Délai à l'issue duquel un message devient invisible car un rôle de travail est en train de le traiter. |
|Non|Storage::DocumentStore::MongoDB|Conteneurs v2.0 uniquement. Active MongoDB pour le stockage permanent des résultats. |
|Non|Storage:ObjectStore:AzureBlob:ConnectionString| Conteneurs v3.x uniquement. Chaîne de connexion de stockage d'objets blob Azure. |

## <a name="apikey-configuration-setting"></a>Paramètre de configuration ApiKey

Le paramètre `ApiKey` spécifie la clé de ressource Azure `Cognitive Services` utilisée pour effectuer le suivi des informations de facturation pour le conteneur. Vous devez donner une valeur à ApiKey, qui doit être une clé valide pour la ressource _Cognitive Services_ spécifiée dans le paramètre de configuration [`Billing`](#billing-configuration-setting).

Vous trouverez ce paramètre à l’emplacement suivant :

* Portail Azure : Gestion des ressources **Cognitive Services**, sous **Clés**

## <a name="applicationinsights-setting"></a>Paramètre ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Paramètre de configuration Billing

Le paramètre `Billing` permet de spécifier l’URI de point de terminaison de la ressource _Cognitive Services_ sur Azure servant à effectuer l’analyse des informations de facturation du conteneur. Vous devez donner une valeur à ce paramètre de configuration, qui doit être un URI de point de terminaison valide pour une ressource _Cognitive Services_ dans Azure. Le conteneur crée des rapports sur l’utilisation toutes les 10 à 15 minutes.

Vous trouverez ce paramètre à l’emplacement suivant :

* Portail Azure : Vue d’ensemble **Cognitive Services**, étiquetée `Endpoint`

Pensez à ajouter le routage `vision/v1.0` à l’URI de point de terminaison, comme dans le tableau suivant. 

|Obligatoire| Nom | Type de données | Description |
|--|------|-----------|-------------|
|Oui| `Billing` | String | URI du point de terminaison de facturation<br><br>Exemple :<br>`Billing=https://westcentralus.api.cognitive.microsoft.com/vision/v1.0` |

## <a name="eula-setting"></a>Paramètre Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Paramètres Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Paramètres des informations d’identification du proxy HTTP

[!INCLUDE [Container shared configuration HTTP proxy settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Paramètres de journalisation
 
[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]

## <a name="mount-settings"></a>Paramètres de montage

Utilisez des montages de liaisons pour lire et écrire des données vers et à partir du conteneur. Vous pouvez spécifier un montage d’entrée ou de sortie en spécifiant l’option `--mount` dans la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/).

Les conteneurs Vision par ordinateur n’utilisent pas de montage d’entrée ou de sortie pour stocker des données d’entraînement ou de service. 

La syntaxe exacte de l’emplacement de montage d’hôte varie en fonction du système d’exploitation hôte. De plus, l’emplacement de montage de l’[ordinateur hôte](computer-vision-how-to-install-containers.md#the-host-computer) peut ne pas être accessible en raison d’un conflit entre les autorisations utilisées par le compte de service Docker et les autorisations de l’emplacement de montage de l’hôte. 

|Facultatif| Nom | Type de données | Description |
|-------|------|-----------|-------------|
|Non autorisé| `Input` | String | Les conteneurs Vision par ordinateur n’utilisent pas cet élément.|
|Facultatif| `Output` | String | Cible du montage de sortie. La valeur par défaut est `/output`. Il s’agit de l’emplacement des journaux d’activité. Les journaux d’activité de conteneur sont inclus. <br><br>Exemple :<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Exemples de commandes docker run

Les exemples suivants utilisent les paramètres de configuration pour illustrer comment écrire et utiliser des commandes `docker run`.  Une fois en cours d’exécution, le conteneur continue à s’exécuter jusqu’à ce que vous l’[arrêtiez](computer-vision-how-to-install-containers.md#stop-the-container).

* **Caractère de continuation de ligne** : Les commandes Docker dans les sections suivantes utilisent la barre oblique inverse, `\`, comme caractère de continuation de ligne. Remplacez-la ou supprimez-la en fonction des exigences de votre système d’exploitation hôte. 
* **Ordre des arguments** : Ne changez pas l’ordre des arguments, sauf si vous avez une connaissance approfondie des conteneurs Docker.

Remplacez {_argument_name_} par vos propres valeurs :

| Espace réservé | Valeur | Format ou exemple |
|-------------|-------|---|
| **{API_KEY}** | Clé de point de terminaison de la ressource `Computer Vision` dans la page Clés Azure `Computer Vision`. | `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` |
| **{ENDPOINT_URI}** | La valeur de point de terminaison de facturation est disponible dans la page Vue d’ensemble Azure `Computer Vision`.| Pour obtenir des exemples explicites, consultez [Collecte des paramètres requis](computer-vision-how-to-install-containers.md#gathering-required-parameters). |

[!INCLUDE [subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas.  Pour plus d'informations, consultez [Facturation](computer-vision-how-to-install-containers.md#billing).
> La valeur ApiKey est la **Clé** de la page Clés des ressources Azure `Cognitive Services`.

## <a name="container-docker-examples"></a>Exemples de conteneur Docker

Les exemples Docker suivants s’appliquent au conteneur Lire.


# <a name="version-32-preview"></a>[Version 3.2-preview](#tab/version-3-2)

### <a name="basic-example"></a>Exemple de base

```bash
docker run --rm -it -p 5000:5000 --memory 18g --cpus 8 \
mcr.microsoft.com/azure-cognitive-services/vision/read:3.2-preview.1 \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}

```

### <a name="logging-example"></a>Exemple de journalisation 

```bash
docker run --rm -it -p 5000:5000 --memory 18g --cpus 8 \
mcr.microsoft.com/azure-cognitive-services/vision/read:3.2-preview.1 \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
Logging:Console:LogLevel:Default=Information
```

# <a name="version-20-preview"></a>[Version 2.0-preview](#tab/version-2)

### <a name="basic-example"></a>Exemple de base

```bash
docker run --rm -it -p 5000:5000 --memory 18g --cpus 8 \
mcr.microsoft.com/azure-cognitive-services/vision/read:2.0-preview \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}

```

### <a name="logging-example"></a>Exemple de journalisation 

```bash
docker run --rm -it -p 5000:5000 --memory 18g --cpus 8 \
mcr.microsoft.com/azure-cognitive-services/vision/read:2.0-preview \
Eula=accept \
Billing={ENDPOINT_URI} \
ApiKey={API_KEY}
Logging:Console:LogLevel:Default=Information
```

---

## <a name="next-steps"></a>Étapes suivantes

* Consultez [Guide pratique pour installer et exécuter des conteneurs](computer-vision-how-to-install-containers.md).
