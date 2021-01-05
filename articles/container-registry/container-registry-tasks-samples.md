---
title: Exemples de tâches ACR
description: Exemples de tâches Azure Container Registry (ACR Tasks) pour créer des images conteneur, les exécuter et leur appliquer une mise à jour corrective
ms.topic: article
ms.date: 11/14/2019
ms.openlocfilehash: 49df3bf565052a729ac3c587bd2ba11a299d05f1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74456087"
---
# <a name="acr-tasks-samples"></a>Exemples ACR Tasks

Cet article contient des liens vers des exemples de fichiers `task.yaml` et de fichiers Dockerfile associés pour plusieurs scénarios [Azure Container Registry Tasks](container-registry-tasks-overview.md) (tâches ACR). 

Pour obtenir des exemples supplémentaires, consultez le dépôt [Exemples Azure][task-examples].

## <a name="scenarios"></a>Scénarios

* **Créer une image** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-hello-world.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Exécuter un conteneur** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo.yaml)

* **Créer et envoyer (push) une image** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-push-hello-world.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Créer et exécuter une image** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-run-hello-world.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Créer et envoyer (push) plusieurs images** -  [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/build-push-hello-world-multi.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Créer et tester des images en parallèle** -  [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.dockerfile)

* **Créer et envoyer (push) des images à plusieurs registres** - [YAML](https://github.com/Azure-Samples/acr-tasks/blob/master/multipleRegistries/testtask.yaml), [Dockerfile](https://github.com/Azure-Samples/acr-tasks/blob/master/multipleRegistries/hello-world.dockerfile)


## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur ACR Tasks :

* [Tâches multiétapes](container-registry-tasks-multi-step.md) : workflows basés sur une tâche ACR pour la génération, le test et la mise à jour corrective d’images conteneur dans le cloud.
* [Tâches de référence](container-registry-tasks-reference-yaml.md) : types d’étapes de tâches, leurs propriétés et leur utilisation.
* [Dépôt Cmd](https://github.com/AzureCR/cmd) : collection de conteneurs agissant en tant que commandes pour les ACR Tasks.


<!-- LINKS - External -->
[task-examples]: https://github.com/Azure-Samples/acr-tasks
