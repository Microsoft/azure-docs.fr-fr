---
title: Redémarrer la stratégie pour des tâches à exécution unique
description: Découvrez comment utiliser Azure Container Instances pour exécuter des tâches jusqu’à complétion, telles que des tâches de génération, de test ou de rendu d’image.
ms.topic: article
ms.date: 08/11/2020
ms.openlocfilehash: 336a31a03cdc9dfdfebe79ef47b59ef90053f523
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88798939"
---
# <a name="run-containerized-tasks-with-restart-policies"></a>Exécuter des tâches conteneurisées avec des stratégies de redémarrage

La facilité et la vitesse de déploiement des conteneurs d’Azure Container Instances en font une plateforme incontournable pour les tâches à exécution unique, comme les tâches de génération, de test et de rendu d’image, dans une instance de conteneur.

À l’aide d’une stratégie de redémarrage configurable, vous pouvez spécifier l’arrêt de vos conteneurs lorsque leurs processus sont terminés. Étant donné que les instances de conteneur sont facturées à la seconde, vous êtes facturé uniquement pour les ressources de calcul utilisées au moment où le conteneur qui exécute votre tâche est exécuté.

Les exemples présentés dans cet article utilisent l’interface de ligne de commande Azure. Vous devez disposer d’Azure CLI version 2.0.21 ou ultérieure [en local][azure-cli-install], ou utiliser l’interface de ligne de commande Azure dans [Azure Cloud Shell](../cloud-shell/overview.md).

## <a name="container-restart-policy"></a>Stratégie de redémarrage des conteneurs

Lorsque vous créez un [groupe de conteneurs](container-instances-container-groups.md) dans Azure Container Instances, vous pouvez spécifier l’un des trois paramètres de stratégie de redémarrage disponibles.

| Stratégie de redémarrage   | Description |
| ---------------- | :---------- |
| `Always` | Les conteneurs du groupe de conteneurs sont toujours redémarrés. Il s’agit du paramètre appliqué **par défaut** lorsqu’aucune stratégie de redémarrage n’est spécifiée au moment de la création du conteneur. |
| `Never` | Les conteneurs du groupe de conteneurs ne sont jamais redémarrés. Les conteneurs sont exécutés au maximum une fois. |
| `OnFailure` | Les conteneurs du groupe de conteneurs sont redémarrés uniquement en cas d’échec des processus qui y sont exécutés (lorsque ceux-ci se terminent par un code de sortie différent de zéro). Les conteneurs sont exécutés au moins une fois. |

[!INCLUDE [container-instances-restart-ip](../../includes/container-instances-restart-ip.md)]

## <a name="specify-a-restart-policy"></a>Spécifier une stratégie de redémarrage

La façon dont vous spécifiez une stratégie de redémarrage dépend de la manière dont vous créez vos instances de conteneurs (par exemple, avec l’interface de ligne de commande Azure, avec les applets de commande Azure PowerShell ou dans le portail Azure). Dans l’interface de ligne de commande Azure, spécifiez le paramètre `--restart-policy` lorsque vous appelez [az container create][az-container-create].

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mycontainerimage \
    --restart-policy OnFailure
```

## <a name="run-to-completion-example"></a>Exemple d’exécution jusqu’à achèvement

Pour voir la stratégie de redémarrage à l’œuvre, créez une instance de conteneur à partir de l’image Microsoft [aci-wordcount][aci-wordcount-image], puis spécifiez la stratégie de redémarrage `OnFailure`. Cet exemple de conteneur exécute un script Python qui, par défaut, analyse le texte de [Hamlet](http://shakespeare.mit.edu/hamlet/full.html) de Shakespeare, écrit les 10 mots les plus fréquents dans STDOUT, puis se termine.

Exécutez l’exemple de conteneur avec la commande [az container create][az-container-create] suivante :

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure
```

Azure Container Instances démarre le conteneur, puis l’arrête lorsque l’exécution de son application (ou de son script, dans ce cas) est terminée. Quand Azure Container Instances arrête un conteneur dont la stratégie de redémarrage est `Never` ou `OnFailure`, l’état du conteneur est défini sur **Terminé**. Vous pouvez vérifier l’état du conteneur à l’aide de la commande [az container show][az-container-show] :

```azurecli-interactive
az container show \
    --resource-group myResourceGroup \
    --name mycontainer \
    --query containers[0].instanceView.currentState.state
```

Exemple de sortie :

```bash
"Terminated"
```

Lorsque l’état de l’exemple de conteneur est *Terminé*, vous pouvez voir la sortie de sa tâche en consultant les journaux d’activité du conteneur. Exécutez la commande [az container logs][az-container-logs] pour afficher la sortie du script :

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer
```

Sortie :

```bash
[('the', 990),
 ('and', 702),
 ('of', 628),
 ('to', 610),
 ('I', 544),
 ('you', 495),
 ('a', 453),
 ('my', 441),
 ('in', 399),
 ('HAMLET', 386)]
```

Cet exemple montre la sortie du script envoyé à STDOUT. Toutefois, il est possible que vos tâches en conteneur écrivent leur sortie dans un stockage persistant, en vue d’une récupération ultérieure. Par exemple, vers un [partage de fichiers Azure](./container-instances-volume-azure-files.md).

## <a name="next-steps"></a>Étapes suivantes

Les scénarios basés sur des tâches, telles que le traitement par lots d’un jeu de données volumineux avec plusieurs conteneurs, peuvent tirer parti de l’utilisation de [lignes de commande](container-instances-start-command.md) ou de [variables d’environnement](container-instances-environment-variables.md) personnalisées au moment de l’exécution.

Pour plus d’informations sur la conservation de la sortie de vos conteneurs qui s’exécutent jusqu’à achèvement, consultez [Montage d’un partage de fichiers Azure avec Azure Container Instances](./container-instances-volume-azure-files.md).

<!-- LINKS - External -->
[aci-wordcount-image]: https://hub.docker.com/_/microsoft-azuredocs-aci-wordcount

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container?view=azure-cli-latest#az-container-create
[az-container-logs]: /cli/azure/container?view=azure-cli-latest#az-container-logs
[az-container-show]: /cli/azure/container?view=azure-cli-latest#az-container-show
[azure-cli-install]: /cli/azure/install-azure-cli
