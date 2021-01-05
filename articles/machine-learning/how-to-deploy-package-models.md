---
title: Modèles de package
titleSuffix: Azure Machine Learning
description: Empaquetez un modèle. Les modèles peuvent être empaquetés en tant qu’image Docker à télécharger ensuite, mais vous pouvez aussi créer un Dockerfile et l’utiliser pour générer l’image.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: gopalv
author: gvashishtha
ms.date: 07/31/2020
ms.topic: conceptual
ms.reviewer: larryfr
ms.custom: deploy
ms.openlocfilehash: 299eb316b534ddc9d5eee934cc15eae841276038
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93312636"
---
# <a name="how-to-package-a-registered-model-with-docker"></a>Procédure d’empaquetage d’un modèle inscrit avec Docker

Cet article explique la procédure d’empaquetage d’un modèle Azure Machine Learning inscrit avec Docker.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez déjà formé et inscrit un modèle dans votre espace de travail Machine Learning. Pour apprendre à former et inscrire un modèlescikit-learn, [suivez ce didacticiel](how-to-train-scikit-learn.md).


## <a name="package-models"></a>Modèles de package

Dans certains cas, vous pouvez être amené à créer une image Docker sans déployer le modèle (par exemple, si vous envisagez un [déploiement sur Azure App Service](how-to-deploy-app-service.md)). Vous pouvez également télécharger l’image et l’exécuter sur une installation locale de Docker. Vous pouvez même télécharger les fichiers utilisés pour générer l’image, les inspecter, les modifier, puis générer l’image manuellement.

L’empaquetage de modèle vous permet d’effectuer ces deux opérations. Il empaquette toutes les ressources nécessaires pour héberger un modèle en tant que service web, et vous permet de télécharger une image Docker entièrement générée ou les fichiers nécessaires pour en générer une. Vous pouvez utiliser l’empaquetage de modèle de deux façons :

**Télécharger un modèle empaqueté :** téléchargez une image Docker contenant le modèle et les autres fichiers nécessaires pour l’héberger en tant que service web.

**Générer un fichier Docker :** téléchargez le fichier Docker, le modèle, le script d’entrée et les autres ressources nécessaires pour générer une image Docker. Vous pouvez ensuite inspecter les fichiers ou apporter des modifications avant de générer l’image localement.

Les deux packages permettent d’obtenir une image Docker locale.

> [!TIP]
> La création d’un package est similaire au déploiement d’un modèle. Vous utilisez un modèle inscrit et une configuration d’inférence.

> [!IMPORTANT]
> Pour télécharger une image entièrement générée ou générer une image localement, [Docker](https://www.docker.com) doit être installé dans votre environnement de développement.

### <a name="download-a-packaged-model"></a>Télécharger un modèle empaqueté

L’exemple suivant génère une image qui est inscrite dans le registre de conteneurs Azure pour votre espace de travail :

```python
package = Model.package(ws, [model], inference_config)
package.wait_for_creation(show_output=True)
```

Après avoir créé un package, vous pouvez utiliser `package.pull()` pour extraire l’image dans votre environnement Docker local. La sortie de cette commande affiche le nom de l’image. Par exemple : 

`Status: Downloaded newer image for myworkspacef78fd10.azurecr.io/package:20190822181338`. 

Une fois le modèle téléchargé, utilisez la commande `docker images` pour lister les images locales :

```text
REPOSITORY                               TAG                 IMAGE ID            CREATED             SIZE
myworkspacef78fd10.azurecr.io/package    20190822181338      7ff48015d5bd        4 minutes ago       1.43 GB
```

Pour démarrer un conteneur local basé sur cette image, utilisez la commande suivante afin de démarrer un conteneur nommé à partir de l’interpréteur de commandes ou de la ligne de commande. Remplacez la valeur `<imageid>` par l’ID d’image retourné par la commande `docker images`.

```bash
docker run -p 6789:5001 --name mycontainer <imageid>
```

Cette commande démarre la version la plus récente de l’image nommée `myimage`. Elle mappe le port local 6789 au port du conteneur sur lequel le service web écoute (5001). Elle affecte également le nom `mycontainer` au conteneur, ce qui en facilite l’arrêt. Une fois le conteneur démarré, vous pouvez envoyer des demandes à `http://localhost:6789/score`.

### <a name="generate-a-dockerfile-and-dependencies"></a>Générer un fichier Docker et des dépendances

L’exemple suivant montre comment télécharger le fichier Docker, le modèle et les autres ressources nécessaires pour générer une image localement. Le paramètre `generate_dockerfile=True` indique que vous souhaitez les fichiers, et non une image entièrement générée.

```python
package = Model.package(ws, [model], inference_config, generate_dockerfile=True)
package.wait_for_creation(show_output=True)
# Download the package.
package.save("./imagefiles")
# Get the Azure container registry that the model/Dockerfile uses.
acr=package.get_container_registry()
print("Address:", acr.address)
print("Username:", acr.username)
print("Password:", acr.password)
```

Ce code télécharge les fichiers nécessaires à la création de l’image dans le répertoire `imagefiles`. Le fichier Docker inclus dans les fichiers enregistrés référence une image de base stockée dans un registre de conteneurs Azure. Quand vous générez l’image sur votre installation Docker locale, vous devez utiliser l’adresse, le nom d’utilisateur et le mot de passe pour vous authentifier auprès du registre. Pour générer l’image à l’aide d’une installation Docker locale, effectuez les étapes suivantes :

1. À partir d’un interpréteur de commandes ou d’une session de ligne de commande, utilisez la commande suivante pour authentifier Docker auprès du registre de conteneurs Azure. Remplacez `<address>`, `<username>` et `<password>` par les valeurs récupérées par `package.get_container_registry()`.

    ```bash
    docker login <address> -u <username> -p <password>
    ```

2. Pour créer l’image, utilisez la commande suivante. Remplacez `<imagefiles>` par le chemin du répertoire dans lequel la commande `package.save()` a enregistré les fichiers.

    ```bash
    docker build --tag myimage <imagefiles>
    ```

    Cette commande définit le nom de l’ image sur `myimage`.

Pour vérifier que l’image est générée, utilisez la commande `docker images`. L’image `myimage` doit figurer dans la liste :

```text
REPOSITORY      TAG                 IMAGE ID            CREATED             SIZE
<none>          <none>              2d5ee0bf3b3b        49 seconds ago      1.43 GB
myimage         latest              739f22498d64        3 minutes ago       1.43 GB
```

Pour démarrer un conteneur basé sur cette image, utilisez la commande suivante :

```bash
docker run -p 6789:5001 --name mycontainer myimage:latest
```

Cette commande démarre la version la plus récente de l’image nommée `myimage`. Elle mappe le port local 6789 au port du conteneur sur lequel le service web écoute (5001). Elle affecte également le nom `mycontainer` au conteneur, ce qui en facilite l’arrêt. Une fois le conteneur démarré, vous pouvez envoyer des demandes à `http://localhost:6789/score`.

### <a name="example-client-to-test-the-local-container"></a>Exemple de client pour tester le conteneur local

Le code suivant est un exemple de client Python utilisable avec le conteneur :

```python
import requests
import json

# URL for the web service.
scoring_uri = 'http://localhost:6789/score'

# Two sets of data to score, so we get two results back.
data = {"data":
        [
            [ 1,2,3,4,5,6,7,8,9,10 ],
            [ 10,9,8,7,6,5,4,3,2,1 ]
        ]
        }
# Convert to JSON string.
input_data = json.dumps(data)

# Set the content type.
headers = {'Content-Type': 'application/json'}

# Make the request and display the response.
resp = requests.post(scoring_uri, input_data, headers=headers)
print(resp.text)
```

Pour obtenir des exemples de clients dans d’autres langages de programmation, consultez [Utiliser des modèles déployés en tant que services web](how-to-consume-web-service.md).

### <a name="stop-the-docker-container"></a>Arrêter le conteneur Docker

Pour arrêter le conteneur, utilisez la commande suivante à partir d’un autre interpréteur de commandes ou d’une autre ligne de commande :

```bash
docker kill mycontainer
```

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre des problèmes d’échec de déploiement](how-to-troubleshoot-deployment.md)
* [Déployer dans Azure Kubernetes Service](how-to-deploy-azure-kubernetes-service.md)
* [Créer des applications clientes pour utiliser des services web](how-to-consume-web-service.md)
* [Mettre à jour un service web](how-to-deploy-update-web-service.md)
* [Guide pratique pour déployer un modèle à l’aide d’une image Docker personnalisée](how-to-deploy-custom-docker-image.md)
* [Utiliser TLS pour sécuriser un service web par le biais d’Azure Machine Learning](how-to-secure-web-service.md)
* [Superviser vos modèles Azure Machine Learning avec Application Insights](how-to-enable-app-insights.md)
* [Collecter des données pour des modèles en production](how-to-enable-data-collection.md)
* [Créer des alertes d’événement et des déclencheurs pour les déploiements de modèle](how-to-use-event-grid.md)
