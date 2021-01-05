---
title: Recettes pour conteneurs Docker
titleSuffix: Azure Cognitive Services
description: Découvrez comment créer, tester et stocker des conteneurs avec tout ou partie de vos paramètres de configuration à des fins de déploiement et de réutilisation.
services: cognitive-services
author: aahill
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: 7380ff58d033a68565de7e419ff318f7bdec121d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80875076"
---
# <a name="create-containers-for-reuse"></a>Créer des conteneurs à réutiliser

Utilisez ces recettes de conteneur pour créer des conteneurs Cognitive Services réutilisables. Les conteneurs peuvent être créés avec tout ou partie des paramètres de configuration de telle sorte qu’ils ne soient _pas_ nécessaires au démarrage des conteneurs.

Dès lors que vous disposez de cette nouvelle couche de conteneur (avec des paramètres) et que vous l’avez testée localement, vous pouvez stocker le conteneur dans un registre de conteneurs. Au moment de démarrer, le conteneur n’aura besoin que des paramètres qui ne sont pas actuellement stockés dans celui-ci. Le conteneur de registre privé offre un espace de configuration qui vous permet de transmettre ces paramètres.

## <a name="docker-run-syntax"></a>Syntaxe d’exécution Docker

Dans les exemples `docker run` de ce document, une console Windows avec un caractère de continuation de ligne `^` est censée être utilisée. Pour votre propre utilisation, tenez compte des points suivants :

* Ne changez pas l’ordre des arguments, sauf si vous avez une connaissance approfondie des conteneurs docker.
* Si vous utilisez un autre système d’exploitation que Windows ou une autre console que la console Windows, utilisez la console et/ou le terminal approprié, la syntaxe de dossier pour les montages et le caractère de continuation de ligne pour votre console et votre système.  Le conteneur Cognitive Services étant un système d’exploitation Linux, le montage cible utilise une syntaxe de dossier de type Linux.
* Les exemples `docker run` utilisent le répertoire du lecteur `c:` pour éviter tout conflit d’autorisation sur Windows. Si vous devez utiliser un répertoire spécifique en tant que répertoire d’entrée, vous devrez peut-être accorder au docker une autorisation de service.

## <a name="store-no-configuration-settings-in-image"></a>Stocker aucun paramètre de configuration dans l’image

Les exemples de commandes `docker run` pour chaque service ne stockent aucun paramètre de configuration dans le conteneur. Quand vous démarrez le conteneur à partir d’une console ou d’un service de registre, ces paramètres de configuration doivent être transmis. Le conteneur de registre privé offre un espace de configuration qui vous permet de transmettre ces paramètres.

## <a name="reuse-recipe-store-all-configuration-settings-with-container"></a>Recette réutilisable : stocker tous les paramètres de configuration avec le conteneur

Pour stocker tous les paramètres de configuration, créez un `Dockerfile` avec ces paramètres.

Problèmes de cette approche :

* Le nom et la balise du nouveau conteneur sont différents de ceux du conteneur d’origine.
* Pour modifier ces paramètres, vous devez modifier les valeurs du Dockerfile, recréer l’image et la republier dans votre registre.
* Si des personnes ont accès à votre registre de conteneurs ou à votre hôte local, elles peuvent exécuter le conteneur et utiliser les points de terminaison Cognitive Services.
* Si votre service cognitif ne nécessite pas de montages d’entrée, n’ajoutez pas les lignes `COPY` à votre Dockerfile.

Créez le Dockerfile, en effectuant une extraction à partir du conteneur Cognitive Services existant que vous voulez utiliser, puis utilisez des commandes docker dans le Dockerfile pour définir ou extraire les informations dont a besoin le conteneur.

Cet exemple :

* Définit le point de terminaison de facturation, `{BILLING_ENDPOINT}` à partir de la clé d’environnement de l’hôte en utilisant `ENV`.
* Définit la clé API de facturation, `{ENDPOINT_KEY}` à partir de la clé d’environnement de l’hôte en utilisant `ENV.

### <a name="reuse-recipe-store-billing-settings-with-container"></a>Recette réutilisable : stocker les paramètres de facturation avec le conteneur

Cet exemple montre comment créer un conteneur de sentiments Analyse de texte à partir d’un Dockerfile.

```Dockerfile
FROM mcr.microsoft.com/azure-cognitive-services/sentiment:latest
ENV billing={BILLING_ENDPOINT}
ENV apikey={ENDPOINT_KEY}
ENV EULA=accept
```

Créez et exécuter le conteneur [localement](#how-to-use-container-on-your-local-host) ou à partir de votre [registre de conteneurs privés](#how-to-add-container-to-private-registry), selon les besoins.

### <a name="reuse-recipe-store-billing-and-mount-settings-with-container"></a>Recette réutilisable : stocker les paramètres de facturation et de montage avec le conteneur

Cet exemple montre comment utiliser Language Understanding, en enregistrant la facturation et les modèles à partir du Dockerfile.

* Copie le fichier de modèle Language Understanding (LUIS) à partir du système de fichiers de l’hôte en utilisant `COPY`.
* Le conteneur LUIS prend en charge plusieurs modèles. Si tous les modèles sont stockés dans le même dossier, vous avez besoin d’une seule instruction `COPY`.
* Exécutez le fichier docker à partir du parent relatif du répertoire d’entrée du modèle. Pour l’exemple suivant, exécutez les commandes `docker build` et `docker run` à partir du parent relatif de `/input`. La première instance de `/input` de la commande `COPY` est le répertoire de l’ordinateur hôte. La deuxième instance de `/input` est le répertoire du conteneur.

```Dockerfile
FROM <container-registry>/<cognitive-service-container-name>:<tag>
ENV billing={BILLING_ENDPOINT}
ENV apikey={ENDPOINT_KEY}
ENV EULA=accept
COPY /input /input
```

Créez et exécuter le conteneur [localement](#how-to-use-container-on-your-local-host) ou à partir de votre [registre de conteneurs privés](#how-to-add-container-to-private-registry), selon les besoins.

## <a name="how-to-use-container-on-your-local-host"></a>Comment utiliser un conteneur sur votre hôte local

Pour créer un fichier Docker, remplacez `<your-image-name>` par le nouveau nom de l’image, puis utilisez :

```console
docker build -t <your-image-name> .
```

Pour exécuter l’image et la supprimer à l’arrêt du conteneur (`--rm`) :

```console
docker run --rm <your-image-name>
```

## <a name="how-to-add-container-to-private-registry"></a>Comment ajouter un conteneur à un registre privé

Suivez ces étapes pour utiliser le Dockerfile et placer la nouvelle image dans votre registre de conteneurs privé.  

1. Créez un `Dockerfile` avec le texte de la recette réutilisable. Un `Dockerfile` n’a pas d’extension.

1. Remplacez les valeurs entre chevrons par vos propres valeurs.

1. Créez le fichier dans une image dans la ligne de commande ou un terminal en utilisant la commande suivante. Remplacez les valeurs entre chevrons, `<>`, par vos propres noms de balise et de conteneur.  

    L’option de balise, `-t`, permet d’indiquer ce que vous avez changé pour le conteneur. Par exemple, le nom de conteneur `modified-LUIS` indique que le conteneur d’origine a été disposé en couches. Le nom de balise `with-billing-and-model` indique comment le conteneur Language Understanding (LUIS) a été modifié.

    ```Bash
    docker build -t <your-new-container-name>:<your-new-tag-name> .
    ```

1. Connectez-vous à Azure CLI à partir d’une console. Cette commande ouvre un navigateur et nécessite une authentification. Une fois authentifié, vous pouvez fermer le navigateur et continuer à travailler dans la console.

    ```azurecli
    az login
    ```

1. Connectez-vous à votre registre privé avec Azure CLI à partir d’une console.

    Remplacez les valeurs entre chevrons, `<my-registry>`, par votre propre nom de registre.  

    ```azurecli
    az acr login --name <my-registry>
    ```

    Vous pouvez aussi vous connecter avec un compte de connexion docker si un principal de service vous a été attribué.

    ```Bash
    docker login <my-registry>.azurecr.io
    ```

1. Balisez le conteneur avec l’emplacement du registre privé. Remplacez les valeurs entre chevrons, `<my-registry>`, par votre propre nom de registre. 

    ```Bash
    docker tag <your-new-container-name>:<your-new-tag-name> <my-registry>.azurecr.io/<your-new-container-name-in-registry>:<your-new-tag-name>
    ```

    Si vous n’utilisez pas de nom de balise, `latest` est suggéré.

1. Envoyez (push) la nouvelle image à votre registre de conteneurs privé. Quand vous affichez votre registre de conteneurs privé, le nom du conteneur utilisé dans la commande CLI suivante devient le nom du dépôt.

    ```Bash
    docker push <my-registry>.azurecr.io/<your-new-container-name-in-registry>:<your-new-tag-name>
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer et utiliser une instance de conteneur Azure](azure-container-instance-recipe.md)

<!--
## Store input and output configuration settings

Bake in input params only

FROM containerpreview.azurecr.io/microsoft/cognitive-services-luis:<tag>
COPY luisModel1 /input/
COPY luisModel2 /input/

## Store all configuration settings

If you are a single manager of the container, you may want to store all settings in the container. The new, resulting container will not need any variables passed in to run. 

Issues with this approach:

* In order to change these settings, you will have to change the values of the Dockerfile and rebuild the file. 
* If someone gets access to your container registry or your local host, they can run the container and use the Cognitive Services endpoints. 

The following _partial_ Dockerfile shows how to statically set the values for billing and model. This example uses the 

```Dockerfile
FROM <container-registry>/<cognitive-service-container-name>:<tag>
ENV billing=<billing value>
ENV apikey=<apikey value>
COPY luisModel1 /input/
COPY luisModel2 /input/
```

->