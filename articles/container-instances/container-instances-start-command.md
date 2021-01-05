---
title: Écraser le point d’entrée dans l’instance de conteneur
description: Définir une ligne de commande pour écraser le point d’entrée dans une image conteneur lorsque vous déployez une instance de conteneur Azure
ms.topic: article
ms.date: 04/15/2019
ms.openlocfilehash: 23221de3dc91c37c2e6fb96489539d3954efcd87
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86169627"
---
# <a name="set-the-command-line-in-a-container-instance-to-override-the-default-command-line-operation"></a>Définir la ligne de commande dans une instance de conteneur pour remplacer l’opération de ligne de commande par défaut

Lorsque vous créez une instance de conteneur, vous pouvez spécifier une commande pour remplacer l’instruction de ligne de commande par défaut intégrée à l’image conteneur. Ce comportement équivaut à définir l’argument de ligne de commande `--entrypoint` sur `docker run`.

À l’instar de la définition de [variables d’environnement](container-instances-environment-variables.md) pour les instances de conteneur, la spécification d’une ligne de commande initiale est utile pour les programmes de traitement par lots qui nécessitent que vous prépariez chaque conteneur de manière dynamique avec une configuration propre à une tâche.

## <a name="command-line-guidelines"></a>Recommandations relatives aux lignes de commande

* Par défaut, la ligne de commande spécifie un *processus unique qui démarre sans interpréteur de commandes* dans le conteneur. Par exemple, la ligne de commande peut exécuter un script Python ou un fichier exécutable. Le processus peut spécifier d'autres paramètres ou arguments.

* Pour exécuter plusieurs commandes, commencez votre ligne de commande en définissant un environnement d’interpréteur de commandes qui est pris en charge dans le système d’exploitation du conteneur. Exemples :

  |Système d’exploitation  |Interpréteur de commandes par défaut  |
  |---------|---------|
  |Ubuntu     |   `/bin/bash`      |
  |Alpine     |   `/bin/sh`      |
  |Windows     |    `cmd`     |

  Suivez les conventions de l’interpréteur de commandes pour combiner plusieurs commandes à exécuter l’une après l’autre.

* Selon la configuration du conteneur, vous devrez peut-être définir un chemin d’accès complet à l’exécutable de ligne de commande ou des arguments.

* Définissez une [stratégie de redémarrage](container-instances-restart-policy.md) appropriée pour l’instance de conteneur, selon que la ligne de commande spécifie une tâche de longue durée ou une tâche à exécution unique. Par exemple, l’utilisation d’une stratégie de redémarrage `Never` ou `OnFailure` est recommandée pour une tâche à exécution unique. 

* Si vous avez besoin d’informations sur le point d’entrée par défaut défini dans une image conteneur, utilisez la commande [docker image inspect](https://docs.docker.com/engine/reference/commandline/image_inspect/).

## <a name="command-line-syntax"></a>Syntaxe de la ligne de commande

La syntaxe de ligne de commande varie en fonction de l’API Azure ou de l’outil utilisés pour la création des instances. Si vous spécifiez un environnement d’interpréteur de commandes, respectez également les conventions de syntaxe de commande de l’interpréteur.

* Commande [az container create][az-container-create] : transmettez une chaîne avec le paramètre `--command-line`. Exemple : `--command-line "python myscript.py arg1 arg2"`.

* Cmdlet Azure PowerShell [New-AzureRmContainerGroup][new-azurermcontainergroup] : transmettez une chaîne avec le paramètre `-Command`. Exemple : `-Command "echo hello"`.

* Portail Azure : Dans la propriété **Remplacement de commande** de la configuration du conteneur, fournissez une liste de chaînes séparées par des virgules, sans guillemets. Exemple : `python, myscript.py, arg1, arg2`. 

* Modèle Resource Manager ou fichier YAML, ou l’un des Kits de développement logiciel (SDK) Azure : Spécifiez la propriété de ligne de commande sous la forme d’un tableau de chaînes. Exemple : tableau JSON `["python", "myscript.py", "arg1", "arg2"]` dans un modèle Resource Manager. 

  Si vous connaissez bien la syntaxe [Dockerfile](https://docs.docker.com/engine/reference/builder/), ce format s’apparente à la forme *exec* de l’instruction CMD.

### <a name="examples"></a>Exemples

|    |  Azure CLI   | Portail | Modèle | 
| ---- | ---- | --- | --- |
| **Commande unique** | `--command-line "python myscript.py arg1 arg2"` | **Remplacement de commande** : `python, myscript.py, arg1, arg2` | `"command": ["python", "myscript.py", "arg1", "arg2"]` |
| **Commandes multiples** | `--command-line "/bin/bash -c 'mkdir test; touch test/myfile; tail -f /dev/null'"` |**Remplacement de commande** : `/bin/bash, -c, mkdir test; touch test/myfile; tail -f /dev/null` | `"command": ["/bin/bash", "-c", "mkdir test; touch test/myfile; tail -f /dev/null"]` |

## <a name="azure-cli-example"></a>Exemple Azure CLI

À titre d’exemple, vous allez modifier le comportement de l’image conteneur [microsoft/aci-wordcount][aci-wordcount], qui analyse le texte de la pièce *Hamlet* de Shakespeare pour y rechercher les mots qui reviennent le plus fréquemment. Au lieu d’analyser *Hamlet*, vous pourriez définir une ligne de commande pointant vers une autre source de texte.

Pour visualiser la sortie du conteneur [microsoft/aci-wordcount][aci-wordcount] lorsque ce dernier analyse le texte par défaut, exécutez-le avec la commande [az container create][az-container-create] ci-après. Aucune ligne de commande initiale n’étant spécifiée, la commande de conteneur par défaut s’exécute. À titre d’illustration, cet exemple définit des [variables d’environnement](container-instances-environment-variables.md) afin de rechercher les 3 premiers mots qui comportent au moins cinq lettres :

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer1 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --environment-variables NumWords=3 MinLength=5 \
    --restart-policy OnFailure
```

Une fois que le conteneur présente l’état *Terminated* (vérifiez l’état à l’aide de la commande [az container show][az-container-show]), affichez le journal avec la commande [az container logs][az-container-logs] pour consulter la sortie.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer1
```

Sortie :

```console
[('HAMLET', 386), ('HORATIO', 127), ('CLAUDIUS', 120)]
```

Maintenant, vous allez configurer un second exemple de conteneur afin d’analyser un autre texte en spécifiant une autre ligne de commande. Le script Python exécuté par le conteneur (*wordcount.py*) accepte une URL comme argument et traite le contenu de cette page au lieu de celui défini par défaut.

Par exemple, pour déterminer les 3 premiers mots qui comportent au moins cinq lettres dans la pièce *Roméo et Juliette* :

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer2 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables NumWords=3 MinLength=5 \
    --command-line "python wordcount.py http://shakespeare.mit.edu/romeo_juliet/full.html"
```

Là encore, lorsque l’état du conteneur est *Terminé*, affichez les journaux d’activité du conteneur pour consulter la sortie :

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer2
```

Sortie :

```console
[('ROMEO', 177), ('JULIET', 134), ('CAPULET', 119)]
```

## <a name="next-steps"></a>Étapes suivantes

Les scénarios basés sur des tâches, comme le traitement par lots d’un jeu de données volumineux avec plusieurs conteneurs, peuvent bénéficier de l’utilisation de lignes de commande personnalisées lors de l’exécution. Pour plus d’informations sur l’exécution des conteneurs basés sur des tâches, consultez l’article [Exécuter des tâches conteneurisées avec des stratégies de redémarrage](container-instances-restart-policy.md).

<!-- LINKS - External -->
[aci-wordcount]: https://hub.docker.com/_/microsoft-azuredocs-aci-wordcount

<!-- LINKS Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[new-azurermcontainergroup]: /powershell/module/azurerm.containerinstance/new-azurermcontainergroup
[portal]: https://portal.azure.com
