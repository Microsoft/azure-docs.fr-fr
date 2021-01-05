---
title: Configuration d’un conteneur pour Form Recognizer
titleSuffix: Azure Cognitive Services
description: Découvrez comment configurer le conteneur Form Recognizer pour analyser des données de formulaire et de table.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: aahi
ms.openlocfilehash: 324b70fc810acc4faba4f488f821049f7eb0875e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86538001"
---
# <a name="configure-form-recognizer-containers"></a>Configurer des conteneurs Form Recognizer

[!INCLUDE [Form Recognizer containers limit](includes/container-limit.md)]

En utilisant les conteneurs Azure Form Recognizer, vous pouvez créer une architecture d’application optimisée pour tirer parti des fonctionnalités robustes du cloud et de la localité en périphérie.

Configurez l’environnement d’exécution du conteneur Form Recognizer à l’aide des arguments de la commande `docker run`. Ce conteneur a plusieurs paramètres obligatoires et quelques paramètres facultatifs. Pour obtenir des exemples, consultez la section [Exemples de commandes docker run](#example-docker-run-commands). Les paramètres propres aux conteneurs correspondent aux paramètres de facturation.

## <a name="configuration-settings"></a>Paramètres de configuration

[!INCLUDE [Container shared configuration settings table](../../../includes/cognitive-services-containers-configuration-shared-settings-table.md)]

> [!IMPORTANT]
> Les paramètres [`ApiKey`](#apikey-configuration-setting), [`Billing`](#billing-configuration-setting) et [`Eula`](#eula-setting) sont utilisés ensemble. Vous devez fournir des valeurs valides pour les trois ; à défaut, votre conteneur ne démarre pas. Pour plus d’informations sur l’instanciation d’un conteneur à l’aide de ces paramètres de configuration, consultez [Facturation](form-recognizer-container-howto.md#billing).

## <a name="apikey-configuration-setting"></a>Paramètre de configuration ApiKey

Le paramètre `ApiKey` spécifie la clé de ressource Azure utilisée pour effectuer le suivi des informations de facturation pour le conteneur. La valeur pour la clé API doit être une clé valide pour la ressource _Form Recognizer_ spécifiée pour `Billing` dans la section Paramètre de configuration Billing.

Ce paramètre figure dans le portail Azure, dans **Form Recognizer Resource Management (Gestion des ressources Form Recognizer)** , sous **Clés**.

## <a name="applicationinsights-setting"></a>Paramètre ApplicationInsights

[!INCLUDE [Container shared configuration ApplicationInsights settings](../../../includes/cognitive-services-containers-configuration-shared-settings-application-insights.md)]

## <a name="billing-configuration-setting"></a>Paramètre de configuration Billing

Le paramètre `Billing` permet de spécifier l’URI de point de terminaison de la ressource _Form Recognizer_ sur Azure servant à mesurer les informations de facturation du conteneur. La valeur de ce paramètre de configuration doit être un URI de point de terminaison valide pour une ressource _Form Recognizer_ sur Azure. Le conteneur crée des rapports sur l’utilisation toutes les 10 à 15 minutes.

Ce paramètre figure dans le portail Azure, dans **Form Recognizer Overview (Présentation de Form Recognizer)** , sous **Point de terminaison**.

|Obligatoire| Nom | Type de données | Description |
|--|------|-----------|-------------|
|Oui| `Billing` | String | URI de point de terminaison de facturation. Pour plus d’informations sur la façon d’obtenir l’URI de facturation, consultez [Collecte des paramètres requis](form-recognizer-container-howto.md#gathering-required-parameters). Pour obtenir plus d’informations et une liste complète des points de terminaison régionaux, consultez [Noms de sous-domaines personnalisés pour Cognitive Services](../cognitive-services-custom-subdomains.md). |

## <a name="eula-setting"></a>Paramètre Eula

[!INCLUDE [Container shared configuration eula settings](../../../includes/cognitive-services-containers-configuration-shared-settings-eula.md)]

## <a name="fluentd-settings"></a>Paramètres Fluentd

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-fluentd.md)]

## <a name="http-proxy-credentials-settings"></a>Paramètres des informations d’identification du proxy HTTP

[!INCLUDE [Container shared configuration fluentd settings](../../../includes/cognitive-services-containers-configuration-shared-settings-http-proxy.md)]

## <a name="logging-settings"></a>Paramètres de journalisation

[!INCLUDE [Container shared configuration logging settings](../../../includes/cognitive-services-containers-configuration-shared-settings-logging.md)]


## <a name="mount-settings"></a>Paramètres de montage

Utilisez des montages de liaisons pour lire et écrire des données vers et à partir du conteneur. Vous pouvez spécifier un montage d’entrée ou de sortie en spécifiant l’option `--mount` dans la commande [`docker run`](https://docs.docker.com/engine/reference/commandline/run/).

Le conteneur Form Recognizer nécessite un montage d’entrée et de sortie. Le montage d’entrée peut être en lecture seule. Il est indispensable pour accéder aux données qui sont utilisées pour l’entraînement et le scoring. Le montage de sortie doit être accessible en écriture. Il sert à stocker les modèles et les données temporaires.

La syntaxe exacte de l’emplacement de montage d’hôte varie en fonction du système d’exploitation hôte. De plus, l’emplacement de montage de l’[ordinateur hôte](form-recognizer-container-howto.md#the-host-computer) peut ne pas être accessible en raison d’un conflit entre les autorisations de compte de service Docker et les autorisations d’emplacement du montage d’hôte.

|Facultatif| Nom | Type de données | Description |
|-------|------|-----------|-------------|
|Obligatoire| `Input` | String | Cible du montage d’entrée. La valeur par défaut est `/input`.    <br><br>Exemple :<br>`--mount type=bind,src=c:\input,target=/input`|
|Obligatoire| `Output` | String | Cible du montage de sortie. La valeur par défaut est `/output`.  <br><br>Exemple :<br>`--mount type=bind,src=c:\output,target=/output`|

## <a name="example-docker-run-commands"></a>Exemples de commandes docker run

Les exemples suivants utilisent les paramètres de configuration pour illustrer comment écrire et utiliser des commandes `docker run`. Une fois en cours d’exécution, le conteneur continue à s’exécuter jusqu’à ce que vous l’[arrêtiez](form-recognizer-container-howto.md#stop-the-container).

* **Caractère de continuation de ligne** : Les commandes Docker dans les sections suivantes utilisent la barre oblique inverse (\\) comme caractère de continuation de ligne. Remplacez ou supprimez ce caractère en fonction des exigences de votre système d’exploitation hôte.
* **Ordre des arguments** : Ne changez pas l’ordre des arguments, sauf si vous avez une connaissance approfondie des conteneurs Docker.

Remplacez {_argument_name_} dans le tableau suivant par vos propres valeurs :

| Espace réservé | Valeur |
|-------------|-------|
| **{FORM_RECOGNIZER_API_KEY}** | Clé qui sert à démarrer le conteneur. Elle est disponible dans la page Clés Form Recognizer dans le portail Azure. |
| **{FORM_RECOGNIZER_ENDPOINT_URI}** | La valeur d’URI de point de terminaison de facturation est disponible dans la page de présentation de Form Recognizer du portail Azure.|
| **{COMPUTER_VISION_API_KEY}** | La clé est disponible dans la page Clés de l’API Vision par ordinateur du portail Azure.|
| **{COMPUTER_VISION_ENDPOINT_URI}** | Le point de terminaison de facturation. Si vous utilisez une ressource informatique Vision par ordinateur, la valeur de l’URI est disponible dans la page de vue d’ensemble de l’API Vision par ordinateur du portail Azure. Si vous utilisez un conteneur *cognitive-services-recognize-text*, utilisez l’URL du point de terminaison de facturation passée au conteneur dans la commande `docker run`. |

Consultez [Collecte des paramètres requis](form-recognizer-container-howto.md#gathering-required-parameters) pour plus d’informations sur la façon d’obtenir ces valeurs.

[!INCLUDE [cognitive-services-custom-subdomains-note](../../../includes/cognitive-services-custom-subdomains-note.md)]

> [!IMPORTANT]
> Pour exécuter le conteneur, spécifiez les options `Eula`, `Billing` et `ApiKey` ; sinon, le conteneur ne démarre pas. Pour plus d'informations, consultez [Facturation](#billing-configuration-setting).

## <a name="form-recognizer-container-docker-examples"></a>Exemples Docker de conteneurs Form Recognizer

Les exemples Docker suivants s’appliquent au conteneur Form Recognizer.

### <a name="basic-example-for-form-recognizer"></a>Exemple de base pour Form Recognizer

```Docker
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

### <a name="logging-example-for-form-recognizer"></a>Exemple de journalisation pour Form Recognizer

```Docker
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
Logging:Console:LogLevel:Default=Information
```

## <a name="next-steps"></a>Étapes suivantes

* Passez en revue la page sur l’[installation et l’exécution des conteneurs](form-recognizer-container-howto.md).
