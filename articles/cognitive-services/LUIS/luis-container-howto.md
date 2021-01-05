---
title: Installer et exécuter des conteneurs Docker pour LUIS
titleSuffix: Azure Cognitive Services
description: Utilisez le conteneur LUIS pour charger votre application formée ou publiée, et accéder à ses prédictions localement.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18, cog-serv-seo-aug-2020
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 09/28/2020
ms.author: aahi
keywords: local, Docker, conteneur
ms.openlocfilehash: 778fe388ae3db68d836384299a8a1c7c06e31f41
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96001799"
---
# <a name="install-and-run-docker-containers-for-luis"></a>Installer et exécuter des conteneurs Docker pour LUIS

[!INCLUDE [container image location note](../containers/includes/image-location-note.md)]

Les conteneurs vous permettent d’utiliser LUIS dans votre propre environnement. Les conteneurs conviennent particulièrement bien à certaines exigences de sécurité et de gouvernance des données. Cet article explique comment télécharger, installer et exécuter un conteneur LUIS.

Le conteneur Language Understanding (LUIS) charge votre modèle Language Understanding qui a été entraîné ou publié. En tant qu’[application LUIS](https://www.luis.ai), le conteneur Docker fournit l’accès aux prédictions de requêtes à partir des points de terminaison d’API du conteneur. Vous pouvez collecter les journaux de requête du conteneur et les charger à nouveau dans l’application Language Understanding pour améliorer la précision de prédiction de l’application.

La vidéo suivante illustre l’utilisation de ce conteneur.

[![Démonstration d’un conteneur pour Cognitive Services](./media/luis-container-how-to/luis-containers-demo-video-still.png)](https://aka.ms/luis-container-demo)

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Voici les prérequis à l’exécution du conteneur LUIS :

|Obligatoire|Objectif|
|--|--|
|Moteur Docker| Vous avez besoin d’un moteur Docker installé sur un [ordinateur hôte](#the-host-computer). Docker fournit des packages qui configurent l’environnement Docker sur [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) et [Linux](https://docs.docker.com/engine/installation/#supported-platforms). Pour apprendre les principes de base de Docker et des conteneurs, consultez la [vue d’ensemble de Docker](https://docs.docker.com/engine/docker-overview/).<br><br> Vous devez configurer Docker pour permettre aux conteneurs de se connecter à Azure et de lui envoyer des données de facturation. <br><br> **Sur Windows**, vous devez également configurer Docker pour prendre en charge les conteneurs Linux.<br><br>|
|Bonne connaissance de Docker | Vous devez avoir une compréhension élémentaire des concepts Docker, notamment les registres, référentiels, conteneurs et images conteneurs, ainsi qu’une maîtrise des commandes `docker` de base.|
|La ressource Azure `Cognitive Services` et le fichier d’[application empaquetée](luis-how-to-start-new-app.md) LUIS |Pour pouvoir utiliser le conteneur, vous devez disposer des éléments suivants :<br><br>* Une ressource Azure _Cognitive Services_ et la clé de facturation de l’URI du point de terminaison de facturation associée. Les deux valeurs, disponibles dans les pages Vue d’ensemble et Clés de la ressource, sont nécessaires au démarrage du conteneur. <br>* Une application entraînée ou publiée empaquetée en tant qu’entrée montée dans le conteneur avec son ID d’application associé. Vous pouvez obtenir le fichier empaqueté à partir du portail LUIS ou des API de création. Si vous obtenez des applications empaquetées LUIS à partir des [API de création](#authoring-apis-for-package-file), vous aurez également besoin de votre _clé de création_.<br><br>Ces prérequis sont utilisés pour passer des arguments de ligne de commande aux variables suivantes :<br><br>**{AUTHORING_KEY}**  : cette clé sert à obtenir l’application empaquetée à partir du service LUIS dans le cloud et à charger les journaux d’activité de requêtes vers le cloud. Le format est `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`.<br><br>**{APP_ID}**  : cet ID sert à sélectionner l’application. Le format est `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.<br><br>**{API_KEY}**  : cette clé sert à démarrer le conteneur. La clé de point de terminaison est disponible à deux endroits. Le premier est le portail Azure, dans la liste des clés de la ressource _Cognitive Services_. Elle est également disponible dans le portail LUIS, dans la page Keys and Endpoint settings (Paramètres des clés et du point de terminaison). N’utilisez pas la clé de démarrage.<br><br>**{ENDPOINT_URI}**  : le point de terminaison tel qu'il est fourni à la page Vue d’ensemble.<br><br>La [clé de création et la clé de point de terminaison](luis-limits.md#key-limits) ont différentes fonctions. Ne les utilisez pas de manière interchangeable. |

[!INCLUDE [Gathering required container parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="authoring-apis-for-package-file"></a>API de création de fichier de package

API de création d’applications empaquetées :

* [API de package publiée](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagepublishedapplicationasgzip)
* [API de package non publiée et uniquement entraînée](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagetrainedapplicationasgzip)

### <a name="the-host-computer"></a>L’ordinateur hôte

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Exigences et suggestions relatives au conteneur

Le tableau ci-dessous répertorie les valeurs minimales et recommandées pour l’hôte de conteneur. Vos besoins peuvent varier en fonction du volume du trafic.

|Conteneur| Minimum | Recommandé | TPS<br>(Minimum, maximum)|
|-----------|---------|-------------|--|
|LUIS|1 cœur, 2 Go de mémoire|1 cœur, 4 Go de mémoire|20, 40|

* Chaque cœur doit être cadencé à au moins 2,6 gigahertz (GHz).
* TPS - transactions par seconde

Le nombre de cœurs et la quantité de mémoire correspondent aux paramètres `--cpus` et `--memory` qui sont utilisés dans le cadre de la commande `docker run`.

## <a name="get-the-container-image-with-docker-pull"></a>Obtenir l’image conteneur avec `docker pull`

Utilisez la commande [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) pour télécharger une image conteneur à partir du référentiel `mcr.microsoft.com/azure-cognitive-services/language/luis` :

```
docker pull mcr.microsoft.com/azure-cognitive-services/language/luis:latest
```

Pour obtenir une description complète des balises disponibles utilisées dans la commande précédente, telles que `latest`, consultez [LUIS](https://go.microsoft.com/fwlink/?linkid=2043204) sur Docker Hub.

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

## <a name="how-to-use-the-container"></a>Comment utiliser le conteneur

Une fois que le conteneur est sur l’[ordinateur hôte](#the-host-computer), appliquez la procédure suivante pour travailler avec le conteneur.

![Procédure pour l’utilisation du conteneur LUIS (Language Understanding)](./media/luis-container-how-to/luis-flow-with-containers-diagram.jpg)

1. [Exportez le package](#export-packaged-app-from-luis) pour le conteneur à partir du portail LUIS ou des API LUIS.
1. Déplacez le fichier de package dans le répertoire d’**entrée** requis sur l’[ordinateur hôte](#the-host-computer). Vous ne devez ni renommer, ni modifier, ni remplacer, ni décompresser le fichier de package LUIS.
1. [Exécutez le conteneur](#run-the-container-with-docker-run) avec le _montage d’entrée_ et les paramètres de facturation requis. D’autres [exemples](luis-container-configuration.md#example-docker-run-commands) de commande `docker run` sont disponibles.
1. [Interrogation du point de terminaison de prédiction du conteneur](#query-the-containers-prediction-endpoint).
1. Quand vous en avez terminé avec le conteneur, [importez les journaux d’activité du point de terminaison](#import-the-endpoint-logs-for-active-learning) à partir du montage de sortie dans le portail LUIS et [arrêtez](#stop-the-container) le conteneur.
1. Utilisez l’[apprentissage actif](luis-how-to-review-endpoint-utterances.md) du portail LUIS dans la page **Review endpoint utterances** (Passer en revue les énoncés du point de terminaison) afin d’améliorer l’application.

L’application en cours d’exécution dans le conteneur ne peut pas être modifiée. Pour changer l’application dans le conteneur, vous devez changer l’application dans le service LUIS à l’aide du portail [LUIS](https://www.luis.ai) ou utiliser les [API de création](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c2f) LUIS. Ensuite, entraînez et/ou publiez, puis téléchargez un nouveau package et réexécutez le conteneur.

L’application LUIS à l’intérieur du conteneur ne peut pas être réexportée vers le service LUIS. Seuls les journaux d’activité de requêtes peuvent être chargés.

## <a name="export-packaged-app-from-luis"></a>Exporter l’application empaquetée à partir de LUIS

Le conteneur LUIS a besoin d’une application LUIS entraînée ou publiée pour répondre aux requêtes de prédiction des énoncés d’utilisateur. Pour accéder à l’application LUIS, utilisez l’API du package entraîné ou publié.

L’emplacement par défaut est le sous-répertoire `input` correspondant à l’emplacement où vous exécutez la commande `docker run`.

Placez le fichier de package dans un répertoire et référencez ce répertoire en tant que montage d’entrée quand vous exécutez le conteneur docker.

### <a name="package-types"></a>Types de packages

Le répertoire de montage d’entrée peut contenir simultanément les modèles **Production**, **Staging** (intermédiaire) et **Versioned** (versionné) de l’application. Tous les packages sont montés.

|Type de package|API de point de terminaison de requête|Disponibilité des requêtes|Format du nom de fichier de package|
|--|--|--|--|
|Versions gérées|GET, POST|Conteneur uniquement|`{APP_ID}_v{APP_VERSION}.gz`|
|Staging|GET, POST|Azure et conteneur|`{APP_ID}_STAGING.gz`|
|Production|GET, POST|Azure et conteneur|`{APP_ID}_PRODUCTION.gz`|

> [!IMPORTANT]
> Ne renommez pas, ne modifiez pas, ne remplacez pas et ne décompressez pas les fichiers de package LUIS.

### <a name="packaging-prerequisites"></a>Prérequis pour l’empaquetage

Avant d’empaqueter une application LUIS, vous devez disposer des éléments suivants :

|Conditions requises pour l’empaquetage|Détails|
|--|--|
|Instance de ressource Azure _Cognitive Services_|Exemples de régions prises en charge :<br><br>USA Ouest (`westus`)<br>Europe Ouest (`westeurope`)<br>Australie Est (`australiaeast`)|
|Application LUIS entraînée ou publiée|Sans aucune [dépendance non prise en charge][unsupported-dependencies]. |
|Accès au système de fichiers de l’[ordinateur hôte](#the-host-computer) |L’ordinateur hôte doit autoriser un [montage d’entrée](luis-container-configuration.md#mount-settings).|

### <a name="export-app-package-from-luis-portal"></a>Exporter le package d’application à partir du portail LUIS

Le [portail](https://www.luis.ai) LUIS offre la possibilité d’exporter le package de l’application entraînée ou publiée.

### <a name="export-published-apps-package-from-luis-portal"></a>Exporter le package de l’application publiée à partir du portail LUIS

Le package de l’application publiée est disponible à partir de la page de liste **My Apps**.

1. Connectez-vous au portail [LUIS](https://www.luis.ai).
1. Cochez la case à gauche du nom de l’application dans la liste.
1. Sélectionnez l’élément **Export** dans la barre d’outils contextuelle au-dessus de la liste.
1. Sélectionnez **Export for container (GZIP)** .
1. Sélectionnez l’environnement **Production slot** (Emplacement de production) ou **Staging slot** (Emplacement intermédiaire).
1. Le package est téléchargé à partir du navigateur.

![Exporter le package publié pour le conteneur à partir du menu Export de la page d’application](./media/luis-container-how-to/export-published-package-for-container.png)

### <a name="export-versioned-apps-package-from-luis-portal"></a>Exporter le package de l’application versionné à partir du portail LUIS

Le package de l’application versionné est disponible à partir de la page de liste **Versions**.

1. Connectez-vous au portail [LUIS](https://www.luis.ai).
1. Sélectionnez l’application dans la liste.
1. Sélectionnez **Manage** (Gérer) dans la barre de navigation de l’application.
1. Sélectionnez **Versions** dans la barre de navigation gauche.
1. Cochez la case à gauche du nom de la version dans la liste.
1. Sélectionnez l’élément **Export** dans la barre d’outils contextuelle au-dessus de la liste.
1. Sélectionnez **Export for container (GZIP)** .
1. Le package est téléchargé à partir du navigateur.

![Exporter le package entraîné pour le conteneur à partir du menu Export de la page Versions](./media/luis-container-how-to/export-trained-package-for-container.png)

### <a name="export-published-apps-package-from-api"></a>Exporter le package de l’application publiée à partir de l’API

Utilisez l’API REST suivante pour empaqueter une application LUIS que vous avez déjà [publiée](luis-how-to-publish-app.md). Remplacez les espaces réservés dans l’appel d’API par vos propres valeurs appropriées, en utilisant le tableau sous la spécification HTTP.

```http
GET /luis/api/v2.0/package/{APP_ID}/slot/{SLOT_NAME}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| Espace réservé | Valeur |
|-------------|-------|
| **{APP_ID}** | ID d’application de l’application LUIS publiée. |
| **{SLOT_NAME}** | Environnement de l’application LUIS publiée. Utilisez l’une des valeurs suivantes :<br/>`PRODUCTION`<br/>`STAGING` |
| **{AUTHORING_KEY}** | Clé de création du compte LUIS pour l’application LUIS publiée.<br/>Vous pouvez obtenir votre clé de création à partir de la page **User Settings** (Paramètres utilisateur) dans le portail LUIS. |
| **{AZURE_REGION}** | Région Azure appropriée :<br/><br/>`westus` - USA Ouest<br/>`westeurope` - Europe Ouest<br/>`australiaeast` - Australie Est |

Pour télécharger le package publié, reportez-vous à la [documentation sur les API][download-published-package]. En cas de téléchargement réussi, la réponse est un fichier de package LUIS. Enregistrez le fichier à l’emplacement de stockage spécifié pour le montage d’entrée du conteneur.

### <a name="export-versioned-apps-package-from-api"></a>Exporter le package de l’application versionné à partir de l’API

Utilisez l’API REST suivante pour empaqueter une application LUIS que vous avez déjà [entraînée](luis-how-to-train.md). Remplacez les espaces réservés dans l’appel d’API par vos propres valeurs appropriées, en utilisant le tableau sous la spécification HTTP.

```http
GET /luis/api/v2.0/package/{APP_ID}/versions/{APP_VERSION}/gzip HTTP/1.1
Host: {AZURE_REGION}.api.cognitive.microsoft.com
Ocp-Apim-Subscription-Key: {AUTHORING_KEY}
```

| Espace réservé | Valeur |
|-------------|-------|
| **{APP_ID}** | ID d’application de l’application LUIS entraînée. |
| **{APP_VERSION}** | Version de l’application LUIS entraînée. |
| **{AUTHORING_KEY}** | Clé de création du compte LUIS pour l’application LUIS publiée.<br/>Vous pouvez obtenir votre clé de création à partir de la page **User Settings** (Paramètres utilisateur) dans le portail LUIS. |
| **{AZURE_REGION}** | Région Azure appropriée :<br/><br/>`westus` - USA Ouest<br/>`westeurope` - Europe Ouest<br/>`australiaeast` - Australie Est |

Pour télécharger le package versionné, reportez-vous à la [documentation sur les API][download-versioned-package]. En cas de téléchargement réussi, la réponse est un fichier de package LUIS. Enregistrez le fichier à l’emplacement de stockage spécifié pour le montage d’entrée du conteneur.

## <a name="run-the-container-with-docker-run"></a>Exécuter le conteneur avec `docker run`

Utilisez la commande [docker run](https://docs.docker.com/engine/reference/commandline/run/) pour exécuter le conteneur. Pour plus d’informations sur la façon d’obtenir les valeurs `{ENDPOINT_URI}` et `{API_KEY}`, consultez [Collecte des paramètres requis](#gathering-required-parameters).

[Exemples ](luis-container-configuration.md#example-docker-run-commands) de la commande `docker run` sont disponibles.

```console
docker run --rm -it -p 5000:5000 ^
--memory 4g ^
--cpus 2 ^
--mount type=bind,src=c:\input,target=/input ^
--mount type=bind,src=c:\output\,target=/output ^
mcr.microsoft.com/azure-cognitive-services/language/luis ^
Eula=accept ^
Billing={ENDPOINT_URI} ^
ApiKey={API_KEY}
```

* Cet exemple utilise le répertoire du lecteur `C:` pour éviter tout conflit d’autorisation sur Windows. Si vous devez utiliser un répertoire spécifique en tant que répertoire d’entrée, vous devrez peut-être accorder au docker une autorisation de service.
* Ne changez pas l’ordre des arguments, sauf si vous connaissez les conteneurs Docker.
* Si vous utilisez un autre système d’exploitation, utilisez la console et/ou le terminal approprié, la syntaxe de dossier pour les montages et le caractère de continuation de ligne pour votre système. Dans ces exemples, une console Windows avec un caractère de continuation de ligne `^` est censée être utilisée. Le conteneur étant un système d’exploitation Linux, le montage cible utilise une syntaxe de dossier de type Linux.

Cette commande :

* Exécute un conteneur à partir de l’image conteneur LUIS.
* Charge l’application LUIS à partir du montage d’entrée à l’emplacement *C:\input* situé sur l’hôte de conteneur.
* Alloue deux cœurs de processeur et 4 gigaoctets (Go) de mémoire.
* Expose le port TCP 5000 et alloue un pseudo-TTY pour le conteneur
* Enregistre les journaux d’activité LUIS et de conteneur dans le montage de sortie à l’emplacement *C:\output* situé sur l’hôte de conteneur.
* Supprime automatiquement le conteneur après sa fermeture. L’image conteneur est toujours disponible sur l’ordinateur hôte.

D’autres [exemples](luis-container-configuration.md#example-docker-run-commands) de commande `docker run` sont disponibles.

> [!IMPORTANT]
> Vous devez spécifier les options `Eula`, `Billing` et `ApiKey` pour exécuter le conteneur, sinon il ne démarrera pas.  Pour plus d'informations, consultez [Facturation](#billing).
> La valeur ApiKey correspond à la **clé** mentionnée sur la page **Ressources Azure** du portail LUIS. Elle est également disponible sur la page des clés de ressources Azure `Cognitive Services`.

[!INCLUDE [Running multiple containers on the same host](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

## <a name="endpoint-apis-supported-by-the-container"></a>API de point de terminaison prises en charge par le conteneur

Les versions V2 et [V3](luis-migration-api-v3.md) de l’API sont disponibles avec le conteneur.

## <a name="query-the-containers-prediction-endpoint"></a>Interroger le point de terminaison de prédiction du conteneur

Le conteneur fournit des API de point de terminaison de prédiction de requête basées sur REST. Les points de terminaison pour les applications publiées (intermédiaires ou production) ont un itinéraire _différent_ de celui des points de terminaison pour les applications versionnées.

Utilisez l’hôte, `http://localhost:5000`, pour les API de conteneur.

# <a name="v3-prediction-endpoint"></a>[Point de terminaison de prédiction V3](#tab/v3)

|Type de package|Verbe HTTP|Routage|Paramètres de requête|
|--|--|--|--|
|Publié|GET, POST|`/luis/v3.0/apps/{appId}/slots/{slotName}/predict?`|`query={query}`<br>[`&verbose`]<br>[`&log`]<br>[`&show-all-intents`]|
|Versions gérées|GET, POST|`/luis/v3.0/apps/{appId}/versions/{versionId}/predict?`|`query={query}`<br>[`&verbose`]<br>[`&log`]<br>[`&show-all-intents`]|

Les paramètres de requête configurent ce qui est retourné dans la réponse de requête, et de quelle manière :

|Paramètre de requête.|Type|Objectif|
|--|--|--|
|`query`|string|Énoncé de l’utilisateur.|
|`verbose`|boolean|Valeur booléenne indiquant s’il faut retourner toutes les métadonnées pour les modèles prédits. La valeur par défaut est false.|
|`log`|boolean|Enregistre les requêtes, qui peuvent être utilisées ultérieurement pour l’[apprentissage actif](luis-how-to-review-endpoint-utterances.md). La valeur par défaut est false.|
|`show-all-intents`|boolean|Valeur booléenne indiquant s’il faut retourner toutes les intentions ou uniquement l’intention au score le plus élevé. La valeur par défaut est false.|

# <a name="v2-prediction-endpoint"></a>[Point de terminaison de prédiction V2](#tab/v2)

|Type de package|Verbe HTTP|Routage|Paramètres de requête|
|--|--|--|--|
|Publié|[GET](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee78), [POST](https://westus.dev.cognitive.microsoft.com/docs/services/5819c76f40a6350ce09de1ac/operations/5819c77140a63516d81aee79)|`/luis/v2.0/apps/{appId}?`|`q={q}`<br>`&staging`<br>[`&timezoneOffset`]<br>[`&verbose`]<br>[`&log`]<br>|
|Versions gérées|GET, POST|`/luis/v2.0/apps/{appId}/versions/{versionId}?`|`q={q}`<br>[`&timezoneOffset`]<br>[`&verbose`]<br>[`&log`]|

Les paramètres de requête configurent ce qui est retourné dans la réponse de requête, et de quelle manière :

|Paramètre de requête.|Type|Objectif|
|--|--|--|
|`q`|string|Énoncé de l’utilisateur.|
|`timezoneOffset`|nombre|timezoneOffset vous permet de [changer le fuseau horaire](luis-concept-data-alteration.md#change-time-zone-of-prebuilt-datetimev2-entity) utilisé par l’entité prédéfinie datetimeV2.|
|`verbose`|boolean|Retourne toutes les intentions et leurs scores quand la valeur est true. La valeur par défaut est false, ce qui retourne uniquement la première intention.|
|`staging`|boolean|Retourne une requête à partir des résultats de l’environnement intermédiaire si la valeur est true. |
|`log`|boolean|Enregistre les requêtes, qui peuvent être utilisées ultérieurement pour l’[apprentissage actif](luis-how-to-review-endpoint-utterances.md). La valeur par défaut est true.|

**_

### <a name="query-the-luis-app"></a>Interroger l’application LUIS

Voici un exemple de commande CURL pour interroger le conteneur d’une application publiée :

# <a name="v3-prediction-endpoint"></a>[Point de terminaison de prédiction V3](#tab/v3)

Pour interroger un modèle dans un emplacement, utilisez l’API suivante :

```bash
curl -G \
-d verbose=false \
-d log=true \
--data-urlencode "query=turn the lights on" \
"http://localhost:5000/luis/v3.0/apps/{APP_ID}/slots/production/predict"
```

Pour adresser des requêtes à l’environnement _ *Intermédiaire**, remplacez `production` dans l’itinéraire par `staging` :

`http://localhost:5000/luis/v3.0/apps/{APP_ID}/slots/staging/predict`

Pour interroger un modèle versionné, utilisez l’API suivante :

```bash
curl -G \
-d verbose=false \
-d log=false \
--data-urlencode "query=turn the lights on" \
"http://localhost:5000/luis/v3.0/apps/{APP_ID}/versions/{APP_VERSION}/predict"
```

# <a name="v2-prediction-endpoint"></a>[Point de terminaison de prédiction V2](#tab/v2)

Pour interroger un modèle dans un emplacement, utilisez l’API suivante :

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APP_ID}?q=turn%20on%20the%20lights&staging=false&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
Pour effectuer des requêtes sur l’environnement intermédiaire (**Staging**), affectez la valeur true au paramètre de chaîne de requête **staging** :

`staging=true`

Pour interroger un modèle versionné, utilisez l’API suivante :

```bash
curl -X GET \
"http://localhost:5000/luis/v2.0/apps/{APP_ID}/versions/{APP_VERSION}?q=turn%20on%20the%20lights&timezoneOffset=0&verbose=false&log=true" \
-H "accept: application/json"
```
Le nom de version a un maximum de 10 caractères et contient uniquement des caractères autorisés dans une URL.

**_

## <a name="import-the-endpoint-logs-for-active-learning"></a>Importer les journaux d’activité de point de terminaison pour l’apprentissage actif

Si un montage de sortie est spécifié pour le conteneur LUIS, les fichiers journaux de requêtes d’application sont enregistrés dans le répertoire de sortie, où `{INSTANCE_ID}` correspond à l’ID du conteneur. Le journal des requêtes d’application contient la requête, la réponse et les horodateurs de chaque requête de prédiction soumise au conteneur LUIS.

L’emplacement suivant montre la structure de répertoires imbriqués pour les fichiers journaux du conteneur.
```
/output/luis/{INSTANCE_ID}/
```

Dans le portail LUIS, sélectionnez votre application, puis _ *Importer les journaux de point de terminaison** pour charger ces journaux.

![Importer les fichiers journaux du conteneur pour l’apprentissage actif](./media/luis-container-how-to/upload-endpoint-log-files.png)

Une fois le journal chargé, [passez en revue les énoncés de point de terminaison](./luis-concept-review-endpoint-utterances.md) dans le portail LUIS.

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="stop-the-container"></a>Arrêter le conteneur

Pour arrêter le conteneur, dans l’environnement de ligne de commande où le conteneur est en cours d’exécution, appuyez sur **Ctrl+C**.

## <a name="troubleshooting"></a>Dépannage

Si vous exécutez le conteneur avec un [montage](luis-container-configuration.md#mount-settings) de sortie et la journalisation activée, il génère des fichiers journaux qui sont utiles pour résoudre les problèmes qui se produisent lors du démarrage ou de l’exécution du conteneur.

[!INCLUDE [Cognitive Services FAQ note](../containers/includes/cognitive-services-faq-note.md)]

## <a name="billing"></a>Facturation

Le conteneur LUIS envoie des informations de facturation à Azure à l’aide d’une ressource _Cognitive Services_ correspondante sur votre compte Azure.

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

Pour plus d’informations sur ces options, consultez [Configurer des conteneurs](luis-container-configuration.md).

<!--blogs/samples/video courses -->
[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>Résumé

Dans cet article, vous avez découvert des concepts et le flux de travail pour le téléchargement, l’installation et l’exécution des conteneurs LUIS (Language Understanding). En résumé :

* LUIS (Language Understanding) fournit un conteneur Linux pour Docker offrant des prédictions de requête de point de terminaison d’énoncés.
* Les images conteneurs sont téléchargées à partir de Microsoft Container Registry (MCR).
* Les images conteneurs s’exécutent dans Docker.
* Vous pouvez utiliser l’API REST pour interroger les points de terminaison de conteneur en spécifiant l’URI hôte du conteneur.
* Vous devez spécifier les informations de facturation lors de l’instanciation d’un conteneur.

> [!IMPORTANT]
> Les conteneurs Cognitives Services ne sont pas concédés sous licence pour s’exécuter sans être connectés à Azure pour le contrôle. Les clients doivent configurer les conteneurs de manière à ce qu’ils communiquent les informations de facturation au service de contrôle à tout moment. Les conteneurs Cognitive Services n’envoient pas de données relatives aux clients (par exemple, l’image ou le texte en cours d’analyse) à Microsoft.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir les paramètres de configuration, passez en revue [Configurer des conteneurs](luis-container-configuration.md).
* Consultez [Limitations des conteneurs LUIS](luis-container-limitations.md) pour connaître les restrictions de capacité connues.
* Reportez-vous à la section [Résolution des problèmes](troubleshooting.md) pour résoudre les problèmes liés à la fonctionnalité LUIS.
* Utiliser davantage de [conteneurs Cognitive Services](../cognitive-services-container-support.md)

<!-- Links - external -->
[download-published-package]: https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagepublishedapplicationasgzip
[download-versioned-package]: https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-packagetrainedapplicationasgzip

[unsupported-dependencies]: luis-container-limitations.md#unsupported-dependencies-for-latest-container