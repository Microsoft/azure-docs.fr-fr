---
title: Mises à jour des images de base – Tâches
description: Familiarisez-vous avec les images de base pour les images conteneurs d’application et découvrez comment la mise à jour d’une image de base peut déclencher une tâche Azure Container Registry.
ms.topic: article
ms.date: 01/22/2019
ms.openlocfilehash: df33096830cd7b34a288c38c105aff3610315337
ms.sourcegitcommit: 17e9cb8d05edaac9addcd6e0f2c230f71573422c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/21/2020
ms.locfileid: "97707484"
---
# <a name="about-base-image-updates-for-acr-tasks"></a>À propos des mises à jour des images de base pour ACR Tasks

Cet article fournit des informations générales sur les mises à jour apportées à l’image de base d’une application et sur la façon dont elles peuvent déclencher une tâche Azure Container Registry.

## <a name="what-are-base-images"></a>Qu’est-ce qu’une image de base ?

Les fichiers Dockerfile qui définissent la plupart des images conteneurs spécifient une image parente, souvent appelée *image de base*, sur laquelle sont basées les autres images. Les images de base contiennent en général le système d’exploitation, par exemple [Alpine Linux][base-alpine] ou [Windows Nano Server][base-windows], sur lequel le reste des couches du conteneur est appliqué. Elles peuvent également inclure des infrastructures d’application comme [Node.js][base-node] ou [.NET Core][base-dotnet]. Ces images de base sont elles-mêmes généralement basées sur des images publiques en amont. Plusieurs de vos images d’application peuvent partager une image de base commune.

Une image de base est souvent mise à jour par le chargé de maintenance des images pour y inclure de nouvelles fonctionnalités ou améliorations relatives au système d’exploitation ou à l’infrastructure. Les correctifs de sécurité constituent une autre cause courante conduisant à une mise à jour de l’image de base. Lorsque ces mises à jour en amont se produisent, vous devez également mettre à jour vos images de base de façon à inclure le correctif critique. Chaque image d’application doit ensuite être également recréée pour inclure ces correctifs en amont maintenant inclus dans votre image de base.

Dans certains cas, par exemple une équipe de développement privée, l’image de base spécifie plus que le système d’exploitation ou le framework. Par exemple, l’image de base peut être une image de composant de service partagé pour laquelle un suivi est nécessaire. Les membres d’une équipe peuvent avoir besoin d’effectuer le suivi de cette image de base à des fins de test ou de la mettre à jour régulièrement lors du développement d’images d’application.

## <a name="maintain-copies-of-base-images"></a>Conserver des copies d’images de base

Pour tout contenu de vos registres qui dépend d’un contenu de base conservé dans un registre public tel que Docker Hub, nous vous recommandons de copier le contenu dans un registre de conteneurs Azure ou dans un autre registre privé. Ensuite, assurez-vous de générer vos images d’application en référençant les images de base privées. Azure Container Registry fournit une capacité d’[importation d’images](container-registry-import-images.md) permettant de copier facilement le contenu des registres publics ou d’autres registres de conteneurs Azure. La section suivante décrit l’utilisation d’ACR Tasks pour suivre les mises à jour des images de base lors de la génération des mises à jour des applications. Vous pouvez suivre les mises à jour des images de base dans vos registres de conteneurs Azure et éventuellement dans les registres publics en amont.

## <a name="track-base-image-updates"></a>Suivi des mises à jour des images de base

ACR Tasks vous permet de générer automatiquement des images lorsque l’image de base d’un conteneur est mise à jour. Vous pouvez utiliser cette capacité pour gérer et mettre à jour des copies d’images de base publiques dans vos registres de conteneurs Azure, puis pour régénérer les images d’application qui dépendent des images de base.

ACR Tasks découvre dynamiquement les dépendances des images de base lorsqu’il génère une image conteneur. Par conséquent, il peut détecter quand l’image de base d’une image d’application est mise à jour. Avec une tâche de build préconfigurée, ACR Tasks est capable de reconstruire automatiquement chacune des images d’application faisant référence à l’image de base. Grâce à ces détection et regénération automatiques, ACR Tasks vous permet d’économiser le temps et les efforts normalement nécessaires au suivi et à la mise à jour manuels de chaque image d’application faisant référence à votre image de base mise à jour.

## <a name="base-image-locations"></a>Emplacements des images de base

Pour les générations d’images à partir d’un fichier Dockerfile, une tâche ACR détecte les dépendances sur les images de base aux emplacements suivants :

* Le registre de conteneurs Azure dans lequel s’exécute la tâche
* Un autre registre de conteneurs Azure, dans la même région ou non 
* Un référentiel public dans Docker Hub 
* Un référentiel public dans Microsoft Container Registry

Si l’image de base spécifiée dans l’instruction `FROM` se trouve à l’un de ces emplacements, la tâche ACR ajoute un hook pour que l’image soit bien regénérée à chaque mise à jour de sa base.

## <a name="base-image-notifications"></a>Notifications d’images de base

Le délai entre le moment où une image de base est mise à jour et le moment où la tâche dépendante est déclenchée dépend de l’emplacement de l’image de base :

* **Images de base à partir d’un référentiel public dans Docker Hub ou MCR** : pour les images de base dans des référentiels publics, une tâche ACR vérifie les mises à jour de l’image à un intervalle aléatoire compris entre 10 et 60 minutes. Les tâches dépendantes sont exécutées en conséquence.
* **Images de base à partir d’un registre de conteneurs Azure** : pour les images de base dans des registres de conteneurs Azure, une tâche ACR se déclenche immédiatement lorsque son image de base est mise à jour. L’image de base peut figurer dans le même ACR que celui dans lequel la tâche s’exécute, ou dans un ACR différent d’une autre région.

## <a name="additional-considerations"></a>Considérations supplémentaires

* **Images de base pour les images d’application** : actuellement, une tâche ACR ne suit les mises à jour des images de base que pour les images d’application (*runtime*). Elles ne suivent pas les mises à jour des images de base pour les images intermédiaires (*au moment de la génération*) utilisées dans des Dockerfiles multiétapes.  

* **Activé par défaut** : lorsque vous créez une tâche ACR avec la commande [az acr task create][az-acr-task-create], elle est par défaut *activée* pour se déclencher par la mise à jour de l’image de base. Cela signifie que la propriété `base-image-trigger-enabled` est définie sur True. Si vous voulez désactiver ce comportement dans une tâche, mettez à jour la propriété sur False. Par exemple, exécutez la commande suivante [az acr task update][az-acr-task-update] :

  ```azurecli
  az acr task update --registry myregistry --name mytask --base-image-trigger-enabled False
  ```

* **Déclencher pour le suivi des dépendances** : pour permettre à une tâche ACR de déterminer et de suivre les dépendances d’une image conteneur, y compris son image de base, vous devez d’abord déclencher la génération de l’image **au moins une fois**. Par exemple, déclenchez la tâche manuellement en utilisant la commande [az acr task run][az-acr-task-run].

* **Balise stable pour l’image de base** : pour déclencher une tâche lors de la mise à jour de l’image de base, il faut que cette dernière possède une balise *stable*, par exemple `node:9-alpine`. Cette catégorisation est typique pour une image de base qui est mise à jour avec des correctifs de système d’exploitation et de framework vers une dernière version stable. Si l’image de base est mis à jour avec une nouvelle balise de version, elle ne déclenche pas de tâche. Pour plus d’informations, consultez la catégorisation d’image, voir [Conseils sur les meilleures pratiques](container-registry-image-tag-version.md). 

* **Autres déclencheurs de tâche** : dans le cadre d’une tâche déclenchée par les mises à jour de l’image de base, vous pouvez également activer des déclencheurs selon la [validation du code source](container-registry-tutorial-build-task.md) ou une [planification](container-registry-tasks-scheduled.md). La mise à jour d’une image de base peut également déclencher une [tâche multiétape](container-registry-tasks-multi-step.md).

## <a name="next-steps"></a>Étapes suivantes

Pour connaître les scénarios d’automatisation de la génération des images d’application après la mise à jour d’une image de base, consultez les tutoriels suivants :

* [Automatisation de la génération des images conteneur en cas de mise à jour d’une image de base dans le même registre](container-registry-tutorial-base-image-update.md)

* [Automatisation de la génération des images conteneur en cas de mise à jour d’une image de base dans un autre registre](container-registry-tutorial-base-image-update.md)


<!-- LINKS - External -->
[base-alpine]: https://hub.docker.com/_/alpine/
[base-dotnet]: https://hub.docker.com/r/microsoft/dotnet/
[base-node]: https://hub.docker.com/_/node/
[base-windows]: https://hub.docker.com/r/microsoft/nanoserver/
[sample-archive]: https://github.com/Azure-Samples/acr-build-helloworld-node/archive/master.zip
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-pack-build]: /cli/azure/acr/pack#az-acr-pack-build
[az-acr-task]: /cli/azure/acr/task
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[az-acr-task-update]: /cli/azure/acr/task#az-acr-task-update
[az-login]: /cli/azure/reference-index#az-login
[az-login-service-principal]: /cli/azure/authenticate-azure-cli

<!-- IMAGES -->
[quick-build-01-fork]: ./media/container-registry-tutorial-quick-build/quick-build-01-fork.png
[quick-build-02-browser]: ./media/container-registry-tutorial-quick-build/quick-build-02-browser.png
