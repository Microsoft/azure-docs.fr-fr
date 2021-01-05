---
title: Guide pratique pour installer et exécuter un conteneur pour Form Recognizer
titleSuffix: Azure Cognitive Services
description: Cet article explique comment utiliser le conteneur Form Recognizer Azure Cognitive Services pour analyser des données de formulaire et de table.
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 07/14/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: 3aee0497f79b57699a44641488c1f09bbae79960
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913141"
---
# <a name="install-and-run-form-recognizer-containers-preview"></a>Installer et exécuter des conteneurs Form Recognizer (préversion)

[!INCLUDE [Form Recognizer containers limit](includes/container-limit.md)]

Le service Azure Form Recognizer utilise la technologie Machine Learning pour identifier et extraire des tables et des paires clé-valeur à partir de formulaires. Il associe à celles-ci des valeurs et des entrées de table, puis génère des données structurées qui incluent les relations dans le fichier d’origine. 

Pour réduire la complexité et intégrer facilement un modèle Form Recognizer personnalisé dans votre processus d’automatisation de flux de travail ou une autre application, vous pouvez appeler le modèle à l’aide d’une simple API REST. Seuls cinq formulaires (ou un formulaire vide et deux formulaires complétés) sont nécessaires pour obtenir des résultats rapides, précis et adaptés à votre contenu. Cela ne nécessite pas d’intervention manuelle et aucune expertise en science des données n’est requise. Cela ne nécessite pas davantage d’étiquetage ou d’annotation de données.

| Fonction | Fonctionnalités |
|----------|----------|
| Form Recognizer | <li>Traite les fichiers PDF, PNG et JPG.<li>Effectue l’apprentissage de modèles personnalisés avec un minimum de 5 formulaires présentant la même disposition. <li>Extrait des paires clé-valeur et des informations de table. <li>Utilise la fonction Reconnaître le texte de l’API Vision par ordinateur de Cognitive Services pour détecter et extraire du texte imprimé à partir d’images figurant dans des formulaires.<li>Ne requiert ni étiquetage ni annotation. |

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour pouvoir utiliser des conteneurs Form Recognizer, les conditions préalables suivantes doivent être réunies :

| Obligatoire | Objectif |
|----------|---------|
| Moteur Docker | Vous avez besoin d’un moteur Docker installé sur un [ordinateur hôte](#the-host-computer). Docker fournit des packages qui configurent l’environnement Docker sur [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) et [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Pour apprendre les principes de base de Docker et des conteneurs, consultez la [vue d’ensemble de Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Vous devez configurer Docker pour permettre aux conteneurs de se connecter à Azure et de lui envoyer des données de facturation. <br><br> Sous Windows, vous devez également configurer Docker pour prendre en charge les conteneurs Linux.<br><br> |
| Bonne connaissance de Docker | Vous devez avoir une compréhension élémentaire des concepts de Docker, notamment les registres, référentiels, conteneurs et images conteneurs, ainsi qu’une bonne connaissance des commandes `docker` de base. |
| L’interface Azure CLI | Installez [Azure CLI](/cli/azure/install-azure-cli) sur votre hôte. |
| Ressource API Vision par ordinateur | Pour pouvoir traiter des documents et images numérisés, vous devez disposer d’une ressource Vision par ordinateur. Vous pouvez accéder à la fonctionnalité Reconnaître le texte en tant que ressource Azure (API REST ou SDK) ou [conteneur](../Computer-vision/computer-vision-how-to-install-containers.md#get-the-container-image-with-docker-pull) *cognitive-services-Recognize-Text*. Les frais de facturation habituels s’appliquent. <br><br>Transmettez la clé API et les points de terminaison de votre ressource Vision par ordinateur (cloud Azure ou conteneur Cognitive Services). Utilisez cette clé API et le point de terminaison en tant que **{COMPUTER_VISION_API_KEY}** et **{COMPUTER_VISION_BILLING_ENDPOINT_URI}** .<br><br> Si vous utilisez le conteneur *cognitive-services-recognize-text* , vérifiez les points suivants :<br><br>Votre clé Vision par ordinateur pour le conteneur Form Recognizer est la clé spécifiée dans la commande `docker run` de Vision par ordinateur pour le conteneur *cognitive-services-recognize-text*.<br>Votre point de terminaison de facturation est le point de terminaison du conteneur (par exemple, `http://localhost:5000`). Si vous utilisez le conteneur Vision par ordinateur et le conteneur Form Recognizer ensemble sur le même hôte, ceux-ci ne peuvent pas être démarrés tous les deux avec le port par défaut *5000*. |
| Ressource Form Recognizer | Pour pouvoir utiliser ces conteneurs, vous devez disposer des éléments suivants :<br><br>Une ressource **Form Recognizer** d’Azure afin d’obtenir la clé API et l’URI du point de terminaison associés. Les deux valeurs, disponibles dans les pages Clés et Vue d’ensemble de **Form Recognizer** du portail Azure, sont nécessaires au démarrage du conteneur.<br><br>**{FORM_RECOGNIZER_API_KEY}**  : l’une des deux clés de ressource disponibles à la page Clés<br><br>**{FORM_RECOGNIZER_ENDPOINT_URI}**  : le point de terminaison tel qu'il est fourni à la page Vue d’ensemble |

> [!NOTE]
> Le nom de la ressource de vision par ordinateur doit être un mot unique, sans trait d’union `-` ni aucun autre caractère spécial. Cette restriction est en place pour garantir la compatibilité des conteneurs Form Recognizer et Reconnaître le texte.

## <a name="gathering-required-parameters"></a>Collecte des paramètres requis

Il existe trois paramètres principaux pour tous les conteneurs de Cognitive Services requis. Le contrat de licence utilisateur final (CLUF) doit être présent avec une valeur de `accept`. En outre, une URL de point de terminaison et une clé API sont nécessaires.

### <a name="endpoint-uri-computer_vision_endpoint_uri-and-form_recognizer_endpoint_uri"></a>`{COMPUTER_VISION_ENDPOINT_URI}` et `{FORM_RECOGNIZER_ENDPOINT_URI}` de l’URI du point de terminaison

La valeur URI du **point de terminaison** est disponible sur la page *Vue d’ensemble* du portail Azure de la ressource Cognitive Service correspondante. Accédez à la page *Vue d’ensemble* , pointez sur le point de terminaison et une icône `Copy to clipboard` <span class="docon docon-edit-copy x-hidden-focus"></span> s’affiche. Copiez et utilisez si nécessaire.

![Collecter l’URI de point de terminaison pour une utilisation ultérieure](../containers/media/overview-endpoint-uri.png)

### <a name="keys-computer_vision_api_key-and-form_recognizer_api_key"></a>Clés `{COMPUTER_VISION_API_KEY}` et `{FORM_RECOGNIZER_API_KEY}`

Cette clé est utilisée pour démarrer le conteneur et est disponible sur la page Clés de la ressource Cognitive Service correspondante sur le portail Azure. Accédez à la page *Clés* , puis cliquez sur l’icône `Copy to clipboard` <span class="docon docon-edit-copy x-hidden-focus"></span>.

![Obtenir l’une des deux clés pour une utilisation ultérieure](../containers/media/keys-copy-api-key.png)

> [!IMPORTANT]
> Ces clés d’abonnement sont utilisées pour accéder à votre API Cognitive Service. Ne partagez pas vos clés. Stockez-les en toute sécurité, par exemple, à l’aide de Azure Key Vault. Nous vous recommandons également de régénérer ces clés régulièrement. Une seule clé est nécessaire pour effectuer un appel d’API. Lors de la régénération de la première clé, vous pouvez utiliser la deuxième clé pour un accès continu au service.

## <a name="the-host-computer"></a>L’ordinateur hôte

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Exigences et suggestions relatives au conteneur

Les quantités minimale et recommandée de cœurs de processeur et de mémoire à allouer pour chaque conteneur Form Recognizer sont indiquées dans le tableau suivant :

| Conteneur | Minimum | Recommandé |
|-----------|---------|-------------|
| Form Recognizer | 2 cœurs, 4 Go de mémoire | 4 cœurs, 8 Go de mémoire |
| Reconnaître le texte | 1 cœur, 8 Go de mémoire | 2 cœurs, 8 Go de mémoire |

* Chaque cœur doit être cadencé à au moins 2,6 gigahertz (GHz).
* Le nombre de cœurs et la quantité de mémoire correspondent aux paramètres `--cpus` et `--memory` qui sont utilisés dans le cadre de la commande `docker run`.

> [!Note]
> Les valeurs minimale et recommandée sont basées sur les limites de Docker, *pas* sur les ressources de la machine hôte.

## <a name="get-the-container-images-with-the-docker-pull-command"></a>Obtenir les images conteneur avec la commande docker pull

Les images conteneur des offres **Form Recognizer** et **Reconnaître le texte** sont disponibles dans le registre de conteneurs suivant :

| Conteneur | Nom complet de l’image |
|-----------|------------|
| Form Recognizer | `containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest` |
| Reconnaître le texte | `containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text:latest` |

Vous aurez besoin des deux conteneurs. Pour plus de détails sur le conteneur **Reconnaître le texte** , consultez [cet article](../Computer-vision/computer-vision-how-to-install-containers.md#get-the-container-image-with-docker-pull).

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-form-recognizer-container"></a>Commande docker pull du conteneur Form Recognizer

#### <a name="form-recognizer"></a>Form Recognizer

Pour obtenir le conteneur Form Recognizer, utilisez la commande suivante :

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest
```
### <a name="docker-pull-for-the-recognize-text-container"></a>Commande docker pull du conteneur Reconnaître le texte

#### <a name="recognize-text"></a>Reconnaître le texte

Pour obtenir le conteneur Reconnaître le texte, utilisez la commande suivante :

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text:latest
```

## <a name="how-to-use-the-container"></a>Comment utiliser le conteneur

Une fois que le conteneur se trouve sur l’[ordinateur hôte](#the-host-computer), procédez comme suit pour utiliser le conteneur.

1. [Exécutez le conteneur](#run-the-container-by-using-the-docker-run-command) avec les paramètres de facturation requis. D’autres [exemples](form-recognizer-container-configuration.md#example-docker-run-commands) de commande `docker run` sont disponibles.
1. [Interrogez le point de terminaison de prédiction du conteneur](#query-the-containers-prediction-endpoint).

## <a name="run-the-container-by-using-the-docker-run-command"></a>Exécuter le conteneur à l’aide de la commande docker run

Utilisez la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/) pour exécuter le conteneur. Consultez [Collecte des paramètres requis](#gathering-required-parameters) pour plus d’informations sur la façon d’obtenir les valeurs `{COMPUTER_VISION_ENDPOINT_URI}`, `{COMPUTER_VISION_API_KEY}`, `{FORM_RECOGNIZER_ENDPOINT_URI}` et `{FORM_RECOGNIZER_API_KEY}`.

[Exemples ](form-recognizer-container-configuration.md#example-docker-run-commands) de la commande `docker run` sont disponibles.

### <a name="form-recognizer"></a>Form Recognizer

> [!NOTE]
> Les répertoires utilisés pour `--mount` dans ces exemples sont des chemins d’accès aux répertoires Windows. Si vous utilisez Linux ou macOS, modifiez le paramètre pour votre environnement. 

```bash
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

Cette commande :

* Exécute un conteneur Form Recognizer à partir de l’image conteneur.
* Alloue 2 cœurs de processeur et 8 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.
* Monte un volume /input et un volume /output sur le conteneur.

[!INCLUDE [Running multiple containers on the same host H2](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

### <a name="run-separate-containers-as-separate-docker-run-commands"></a>Exécuter des conteneurs distincts sous forme de commandes docker run distinctes

Pour la combinaison Form Recognizer et Reconnaissance de texte hébergée localement sur le même hôte, utilisez les deux exemples de commandes de l’interface de ligne de commande Docker suivants :

Exécutez le premier conteneur sur le port 5000. 

```bash 
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY}
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

Exécutez le deuxième conteneur sur le port 5001.

```bash 
docker run --rm -it -p 5001:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text \
Eula=accept \
Billing={COMPUTER_VISION_ENDPOINT_URI} \
ApiKey={COMPUTER_VISION_API_KEY}
```
Tous les conteneurs suivants doivent être sur un port différent. 

### <a name="run-separate-containers-with-docker-compose"></a>Exécuter des conteneurs distincts avec Docker Compose

Pour la combinaison Form Recognizer et Reconnaissance de texte hébergée localement sur le même hôte, utilisez l’exemple de fichier YAML Docker Compose suivant. `{COMPUTER_VISION_API_KEY}` de la Reconnaissance de texte doit être identique pour les deux conteneurs `formrecognizer` et `ocr`. La valeur `{COMPUTER_VISION_ENDPOINT_URI}` est utilisée uniquement dans le conteneur `ocr`, car le conteneur `formrecognizer` utilise le nom et le port `ocr`. 

```docker
version: '3.3'
services:   
  ocr:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{COMPUTER_VISION_ENDPOINT_URI}"
      apikey: "{COMPUTER_VISION_API_KEY}"

  formrecognizer:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{FORM_RECOGNIZER_ENDPOINT_URI}"
      apikey: "{FORM_RECOGNIZER_API_KEY}"
      FormRecognizer__ComputerVisionApiKey: {COMPUTER_VISION_API_KEY}
      FormRecognizer__ComputerVisionEndpointUri: "http://ocr:5000"
      FormRecognizer__SyncProcessTaskCancelLimitInSecs: 75
    links:
      - ocr
    volumes:
      - type: bind
        source: c:\output
        target: /output
      - type: bind
        source: c:\input
        target: /input
    ports:
      - "5000:5000"
```

> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey`, ainsi que les options `FormRecognizer:ComputerVisionApiKey` et `FormRecognizer:ComputerVisionEndpointUri`, pour exécuter le conteneur, sans quoi il ne démarrera pas. Pour plus d'informations, consultez [Facturation](#billing).

## <a name="query-the-containers-prediction-endpoint"></a>Interroger le point de terminaison de prédiction du conteneur

|Conteneur|Point de terminaison|
|--|--|
|form-recognizer|http://localhost:5000

### <a name="form-recognizer"></a>Form Recognizer

Le conteneur fournit des API de point de terminaison de requête s’appuyant sur WebSocket, qui figurent dans la [documentation du Kit de développement logiciel (SDK) des services Form Recognizer](./index.yml).

Par défaut, le Kit SDK Form Recognizer utilise les services en ligne. Pour utiliser le conteneur, vous devez changer la méthode d’initialisation. Consultez les exemples ci-dessous.

#### <a name="for-c"></a>Pour C#

Passer de l’utilisation de cet appel d’initialisation du cloud Azure :

```csharp
var config =
    FormRecognizerConfig.FromSubscription(
        "YourSubscriptionKey",
        "YourServiceRegion");
```
à cet appel, qui utilise le point de terminaison du conteneur :

```csharp
var config =
    FormRecognizerConfig.FromEndpoint(
        "ws://localhost:5000/formrecognizer/v1.0-preview/custom",
        "YourSubscriptionKey");
```

#### <a name="for-python"></a>Pour Python

Passer de l’utilisation de cet appel d’initialisation du cloud Azure :

```python
formrecognizer_config =
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key, region=service_region)
```

à cet appel, qui utilise le point de terminaison du conteneur :

```python
formrecognizer_config = 
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key,
        endpoint="ws://localhost:5000/formrecognizer/v1.0-preview/custom"
```

### <a name="form-recognizer"></a>Form Recognizer

Le conteneur fournit les API de point de terminaison REST que vous trouverez sur la page [API Form Recognizer](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api/operations/AnalyzeWithCustomModel).


[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]


## <a name="stop-the-container"></a>Arrêter le conteneur

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>Dépannage

Si vous exécutez le conteneur avec un [montage](form-recognizer-container-configuration.md#mount-settings) de sortie et la journalisation activée, il génère des fichiers journaux qui sont utiles pour résoudre les problèmes qui se produisent lors du démarrage ou de l’exécution du conteneur.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturation

Les conteneurs Form Recognizer envoient des informations de facturation à Azure en utilisant une ressource _Form Recognizer_ sur votre compte Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Pour plus d’informations sur ces options, consultez [Configurer des conteneurs](form-recognizer-container-configuration.md).

<!--blogs/samples/video courses -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert les concepts et le workflow du téléchargement, de l’installation et de l’exécution des conteneurs Form Recognizer. En résumé :

* Form Recognizer fournit un conteneur Linux pour Docker.
* Les images conteneur sont téléchargées à partir du registre de conteneurs privé dans Azure.
* Les images conteneurs s’exécutent dans Docker.
* Vous pouvez utiliser l’API REST ou le Kit de développement logiciel (SDK) REST pour appeler des opérations dans le conteneur Form Recognizer en spécifiant l’URI hôte du conteneur.
* Vous devez spécifier les informations de facturation lors de l’instanciation d’un conteneur.

> [!IMPORTANT]
>  Les conteneurs Cognitives Services ne sont pas concédés sous licence pour s’exécuter sans être connectés à Azure pour le contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au service de contrôle à tout moment. Les conteneurs Cognitive Services n’envoient pas de données relatives aux clients (par exemple, l’image ou le texte en cours d’analyse) à Microsoft.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir les paramètres de configuration, passez en revue [Configurer des conteneurs](form-recognizer-container-configuration.md).
* Utilisez davantage de [conteneurs Cognitive Services](../cognitive-services-container-support.md).