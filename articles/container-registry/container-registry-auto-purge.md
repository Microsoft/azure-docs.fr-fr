---
title: Vider les étiquettes et les manifestes
description: Utilisez une commande de vidage pour supprimer plusieurs balises et manifestes d’un registre de conteneurs Azure en fonction d’un filtre d’âge et d’étiquette, et planifiez éventuellement des opérations de vidage.
ms.topic: article
ms.date: 11/10/2020
ms.openlocfilehash: 406a1f231af57407e9475a8888b68aad9d88dcb3
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94445113"
---
# <a name="automatically-purge-images-from-an-azure-container-registry"></a>Purger automatiquement les images d’un registre de conteneurs Azure

Lorsque vous utilisez un registre de conteneurs Azure dans le cadre d’un workflow de développement, le registre peut rapidement se remplir d’images ou d’autres artefacts qui ne sont pas nécessaires après une brève période. Vous souhaiterez peut-être supprimer toutes les étiquettes antérieures à une certaine durée ou correspondant à un filtre de nom spécifié. Pour supprimer rapidement plusieurs artefacts, cet article présente la commande `acr purge` que vous pouvez exécuter en tant que tâche ACR à la demande ou [planifiée](container-registry-tasks-scheduled.md). 

La commande `acr purge` est actuellement distribuée dans une image de conteneur public (`mcr.microsoft.com/acr/acr-cli:0.3`), générée à partir du code source dans le référentiel [acr-cli](https://github.com/Azure/acr-cli) dans GitHub.

Pour utiliser les exemples de tâche ACR de cet article, vous pouvez utiliser Azure Cloud Shell ou une installation locale d’Azure CLI. Si vous souhaitez l’utiliser en local, la version 2.0.76 ou une version ultérieure est requise. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install]. 

> [!IMPORTANT]
> Actuellement, cette fonctionnalité est uniquement disponible en tant que version préliminaire. Les préversions sont à votre disposition, à condition que vous acceptiez les [conditions d’utilisation supplémentaires][terms-of-use]. Certains aspects de cette fonctionnalité sont susceptibles d’être modifiés avant la mise à disposition générale.

> [!WARNING]
> Utilisez la commande `acr purge` avec précaution : les données d'image supprimées ne sont PAS RÉCUPÉRABLES. Si vous disposez de systèmes qui tirent (pull) les images en fonction du code de hachage du manifeste (et non en fonction du nom de l'image), vous ne devez pas purger les images sans étiquette. La suppression des images sans étiquette va empêcher ces systèmes de tirer les images à partir du registre. Au lieu de tirer en fonction du manifeste, envisagez d’adopter un schéma d’*étiquetage unique* (il s’agit là d’une [bonne pratique](container-registry-image-tag-version.md)).

Si vous souhaitez supprimer des étiquettes d’image unique ou des manifestes à l'aide des commandes Azure CLI, consultez [Supprimer des images conteneur dans Azure Container Registry](container-registry-delete.md).

## <a name="use-the-purge-command"></a>Utiliser la commande de vidage

La commande de conteneur `acr purge` supprime les images par étiquette dans un référentiel qui correspondent à un filtre de nom et qui sont antérieures à une durée spécifiée. Par défaut, seules les références des étiquettes sont supprimées, et non les [manifestes](container-registry-concepts.md#manifest) et les données de couche sous-jacents. La commande a une option pour supprimer aussi les manifestes. 

> [!NOTE]
> `acr purge` ne supprime pas une étiquette d'image ou de référentiel si l’attribut `write-enabled` a la valeur `false`. Pour plus d’informations, consultez [Verrouiller une image dans un registre de conteneurs Azure](container-registry-image-lock.md).

`acr purge` est conçu pour s’exécuter en tant que commande de conteneur dans une [tâche ACR](container-registry-tasks-overview.md), afin qu’il s’authentifie automatiquement avec le registre sur lequel la tâche s’exécute et effectue des actions à cet emplacement. Les exemples de tâches mentionnés dans cet article utilisent l’[alias](container-registry-tasks-reference-yaml.md#aliases) de commande `acr purge` à la place d’une commande d’image conteneur complète.

Au minimum, spécifiez les éléments suivants lorsque vous exécutez `acr purge` :

* `--filter` : un référentiel et une *expression régulière* pour filtrer les étiquettes dans le référentiel. Exemples : `--filter "hello-world:.*"` correspond à toutes les étiquettes du référentiel `hello-world` et `--filter "hello-world:^1.*"` correspond aux étiquettes commençant par `1`. Transmettez plusieurs paramètres `--filter` pour purger plusieurs référentiels.
* `--ago` - Une [chaîne de durée](https://golang.org/pkg/time/) de style Go pour indiquer une durée au-delà de laquelle les images sont supprimées. La durée se compose d’une séquence d’un ou de plusieurs nombres décimaux, chacun avec un suffixe d’unité. Les unités de temps valides incluent « d » pour les jours, « h » pour les heures et « m » pour les minutes. Par exemple, `--ago 2d3h6m` sélectionne toutes les images filtrées dont la dernière modification remonte à plus de 2 jours, 3 heures et 6 minutes, et `--ago 1.5h` sélectionne les images dont la dernière modification remonte à il y a plus de 1,5 heure.

`acr purge` prend en charge plusieurs paramètres facultatifs. Les deux valeurs suivantes sont utilisées dans les exemples de cet article :

* `--untagged` : Spécifie que les manifestes qui n’ont pas d’étiquettes associées (*manifestes non étiquetés*) sont supprimés.
* `--dry-run` : Spécifie qu’aucune donnée n’est supprimée, mais la sortie est la même que si la commande était exécutée sans cet indicateur. Ce paramètre est utile pour tester une commande de vidage afin de s’assurer qu’elle ne supprime pas par inadvertance les données que vous envisagez de conserver.
* `--keep` : spécifie que le dernier nombre x de balises à supprimer est conservé.

Pour les paramètres supplémentaires, exécutez `acr purge --help`. 

`acr purge` prend en charge d’autres fonctionnalités des commandes de tâches ACR, notamment les [variables d’exécution](container-registry-tasks-reference-yaml.md#run-variables) et les [journaux d’exécution des tâches](container-registry-tasks-logs.md) qui sont diffusés en continu et également enregistrés pour une récupération ultérieure.

### <a name="run-in-an-on-demand-task"></a>Exécuter dans une tâche à la demande

L’exemple suivant utilise la commande [az acr run][az-acr-run] pour exécuter la commande `acr purge` à la demande. Cet exemple supprime toutes les balises d’image et les manifestes du référentiel `hello-world` dans *myregistry* qui ont été modifiés il y a plus de 1 jour. La commande de conteneur est passée à l’aide d’une variable d’environnement. La tâche s’exécute sans contexte source.

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge --filter 'hello-world:.*' \
  --untagged --ago 1d"

az acr run \
  --cmd "$PURGE_CMD" \
  --registry myregistry \
  /dev/null
```

### <a name="run-in-a-scheduled-task"></a>Exécuter dans une tâche planifiée

L’exemple suivant utilise la commande [az acr task create][az-acr-task-create] pour créer une [tâche ACR planifiée](container-registry-tasks-scheduled.md) quotidienne. La tâche vide les étiquettes modifiées il y a plus de 7 jours dans le référentiel `hello-world`. La commande de conteneur est passée à l’aide d’une variable d’environnement. La tâche s’exécute sans contexte source.

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge --filter 'hello-world:.*' \
  --ago 7d"

az acr task create --name purgeTask \
  --cmd "$PURGE_CMD" \
  --schedule "0 0 * * *" \
  --registry myregistry \
  --context /dev/null
```

Exécutez la commande [az acr task show][az-acr-task-show] pour vérifier que le déclencheur de minuteur est configuré.

### <a name="purge-large-numbers-of-tags-and-manifests"></a>Vider un grand nombre d’étiquettes et de manifestes

La purge d’un grand nombre d’étiquettes et de manifestes peut prendre plusieurs minutes ou plus. Pour vider des milliers d’étiquettes et de manifestes, la commande devra peut-être s’exécuter plus longtemps que le délai d’attente par défaut de 600 secondes pour une tâche à la demande, ou de 3600 secondes pour une tâche planifiée. Si le délai d’expiration est dépassé, seul un sous-ensemble d’étiquettes et de manifestes est supprimé. Pour vous assurer qu’un vidage à grande échelle se termine, transmettez le paramètre `--timeout` pour augmenter la valeur. 

Par exemple, la tâche à la demande suivante définit un délai d’expiration de 3600 secondes (1 heure) :

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge --filter 'hello-world:.*' \
  --ago 1d --untagged"

az acr run \
  --cmd "$PURGE_CMD" \
  --registry myregistry \
  --timeout 3600 \
  /dev/null
```

## <a name="example-scheduled-purge-of-multiple-repositories-in-a-registry"></a>Exemple : Vidage planifié de plusieurs dépôts dans un registre

Cet exemple explique comment utiliser `acr purge` pour nettoyer périodiquement plusieurs référentiels dans un registre. Par exemple, vous pouvez avoir un pipeline de développement qui pousse les images vers les référentiels `samples/devimage1` et `samples/devimage2`. Vous importez régulièrement des images de développement dans un référentiel de production pour vos déploiements, vous n’avez donc plus besoin des images de développement. Chaque semaine, vous videz les dépôts `samples/devimage1` et `samples/devimage2`, en préparation du travail de la semaine à venir.

### <a name="preview-the-purge"></a>Afficher un aperçu de la purge

Avant de supprimer les données, nous vous recommandons d’exécuter une tâche de vidage à la demande à l’aide du paramètre `--dry-run`. Cette option vous permet de voir les étiquettes et manifestes que la commande videra, sans supprimer les données. 

Dans l’exemple suivant, le filtre de chaque référentiel sélectionne toutes les étiquettes. Le paramètre `--ago 0d` correspond aux images de tous les âges des référentiels qui correspondent aux filtres. Modifiez les critères de sélection en fonction des besoins de votre scénario. Le paramètre `--untagged` indique de supprimer des manifestes en plus des étiquettes. La commande de conteneur est passée à la commande [az acr run][az-acr-run] à l’aide d’une variable d’environnement.

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge \
  --filter 'samples/devimage1:.*' --filter 'samples/devimage2:.*' \
  --ago 0d --untagged --dry-run"

az acr run \
  --cmd "$PURGE_CMD" \
  --registry myregistry \
  /dev/null
```

Examinez la sortie de la commande pour voir les étiquettes et les manifestes qui correspondent aux paramètres de sélection. Étant donné que la commande est exécutée avec `--dry-run`, aucune donnée n’est supprimée.

Exemple de sortie :

```console
[...]
Deleting tags for repository: samples/devimage1
myregistry.azurecr.io/samples/devimage1:232889b
myregistry.azurecr.io/samples/devimage1:a21776a
Deleting manifests for repository: samples/devimage1
myregistry.azurecr.io/samples/devimage1@sha256:81b6f9c92844bbbb5d0a101b22f7c2a7949e40f8ea90c8b3bc396879d95e788b
myregistry.azurecr.io/samples/devimage1@sha256:3ded859790e68bd02791a972ab0bae727231dc8746f233a7949e40f8ea90c8b3
Deleting tags for repository: samples/devimage2
myregistry.azurecr.io/samples/devimage2:5e788ba
myregistry.azurecr.io/samples/devimage2:f336b7c
Deleting manifests for repository: samples/devimage2
myregistry.azurecr.io/samples/devimage2@sha256:8d2527cde610e1715ad095cb12bc7ed169b60c495e5428eefdf336b7cb7c0371
myregistry.azurecr.io/samples/devimage2@sha256:ca86b078f89607bc03ded859790e68bd02791a972ab0bae727231dc8746f233a

Number of deleted tags: 4
Number of deleted manifests: 4
[...]
```

### <a name="schedule-the-purge"></a>Planifier le vidage

Une fois que vous avez vérifié le test, créez une tâche planifiée pour automatiser le vidage. L’exemple suivant planifie une tâche hebdomadaire le dimanche à 1 h UTC pour exécuter la commande de vidage précédente :

```azurecli
# Environment variable for container command line
PURGE_CMD="acr purge \
  --filter 'samples/devimage1:.*' --filter 'samples/devimage2:.*' \
  --ago 0d --untagged"

az acr task create --name weeklyPurgeTask \
  --cmd "$PURGE_CMD" \
  --schedule "0 1 * * Sun" \
  --registry myregistry \
  --context /dev/null
```

Exécutez la commande [az acr task show][az-acr-task-show] pour vérifier que le déclencheur de minuteur est configuré.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les autres options permettant de [supprimer des données d’image](container-registry-delete.md) dans Azure Container Registry.

Pour plus d’informations sur le stockage des images, consultez [Stockage des images conteneur dans Azure Container Registry](container-registry-storage.md).

<!-- LINKS - External -->

[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-show]: /cli/azure/acr/task#az-acr-task-show

