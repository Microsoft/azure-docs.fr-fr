---
title: Informations de référence sur YAML - ACR Tasks
description: Référence pour la définition de tâches dans YAML pour ACR Tasks, y compris les propriétés de tâche, les types d’étapes, les propriétés d’étape et les variables intégrées.
ms.topic: article
ms.date: 07/08/2020
ms.openlocfilehash: 042310d29f5561c2cd77b0b9cccfc587ca4aa767
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88067581"
---
# <a name="acr-tasks-reference-yaml"></a>Référence ACR Tasks : YAML

La définition de tâche en plusieurs étapes dans ACR Tasks fournit une primitive de calcul orientée conteneur axée sur la génération, le test et la mise à jour corrective des conteneurs. Cet article traite des commandes, des paramètres, des propriétés et de la syntaxe pour les fichiers YAML qui définissent vos tâches en plusieurs étapes.

Cet article contient les informations de référence pour la création des fichiers YAML de tâche en plusieurs étapes pour ACR Tasks. Pour une présentation d’ACR Tasks, consultez la [vue d’ensemble d’ACR Tasks](container-registry-tasks-overview.md).

## <a name="acr-taskyaml-file-format"></a>format de fichier acr-task.yaml

ACR Tasks prend en charge la déclaration de tâche en plusieurs étapes dans la syntaxe YAML standard. Vous définissez les étapes d’une tâche dans un fichier YAML. Vous pouvez ensuite exécuter la tâche manuellement en transmettant le fichier à la commande [az acr run][az-acr-run]. Vous pouvez également utiliser le fichier pour créer une tâche avec [az acr task create][az-acr-task-create] qui se déclenche automatiquement à l’occasion d’une validation Git, d’une mise à jour d’image de base ou d’une planification. Bien que cet article fasse référence à `acr-task.yaml` comme fichier qui contient les étapes, ACR Tasks accepte n’importe quel nom de fichier valide avec une [extension prise en charge](#supported-task-filename-extensions).

Les primitives `acr-task.yaml` de niveau supérieur sont des **propriétés de tâche**, des **types d’étape** et des **propriétés d’étape** :

* Les [propriétés de tâche](#task-properties) s’appliquent à toutes les étapes d’exécution de la tâche. Il existe plusieurs propriétés de tâche globale, notamment :
  * `version`
  * `stepTimeout`
  * `workingDirectory`
* Les [types d’étape de tâche](#task-step-types) représentent les types d’actions qui peuvent être effectuées dans une tâche. Il existe trois types d’étapes :
  * `build`
  * `push`
  * `cmd`
* Les [propriétés d’étape de tâche](#task-step-properties) sont des paramètres qui s’appliquent à une étape individuelle. Il existe plusieurs propriétés d’étape, notamment :
  * `startDelay`
  * `timeout`
  * `when`
  * et plus encore.

Le format de base d’un fichier `acr-task.yaml`, y compris certaines propriétés communes d’étape, suit. Bien qu’il ne s’agisse pas d’une représentation exhaustive de toutes les propriétés d’étape disponibles ou de l’utilisation des types d’étape, il fournit une rapide vue d’ensemble pour le format de fichier de base.

```yml
version: # acr-task.yaml format version.
stepTimeout: # Seconds each step may take.
steps: # A collection of image or container actions.
  - build: # Equivalent to "docker build," but in a multi-tenant environment
  - push: # Push a newly built or retagged image to a registry.
    when: # Step property that defines either parallel or dependent step execution.
  - cmd: # Executes a container, supports specifying an [ENTRYPOINT] and parameters.
    startDelay: # Step property that specifies the number of seconds to wait before starting execution.
```

### <a name="supported-task-filename-extensions"></a>Extensions de nom de fichier de tâche prises en charge

ACR Tasks a plusieurs extensions de nom de fichier réservées, y compris `.yaml`, qui sera traitée comme un fichier de tâche. N’importe quelle extension *ne figurant pas* dans la liste suivante est considérée par ACR Tasks comme un fichier Dockerfile : .yaml, .yml, .toml, .json, .sh, .bash, .zsh, .ps1, .ps, .cmd, .bat, .ts, .js, .php, .py, .rb, .lua

YAML est le seul format de fichier actuellement pris en charge par ACR Tasks. Les autres extensions de nom de fichier sont réservées pour une éventuelle prise en charge future.

## <a name="run-the-sample-tasks"></a>Exécuter les exemples de tâche

Plusieurs exemples de fichiers de tâche sont référencés dans les sections suivantes de cet article. Les exemples de tâche se trouvent dans un référentiel GitHub public, [Azure-Samples/acr-tasks][acr-tasks]. Vous pouvez les exécuter via l’interface de ligne de commande Azure avec la commande [az acr run][az-acr-run]. Les exemples de commandes ressembles à ceci :

```azurecli
az acr run -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

La mise en forme des exemples de commandes part du principe que vous avez configuré un registre par défaut dans l’interface de ligne de commande Azure. Le paramètre `--registry` est donc omis. Pour configurer un registre par défaut, utilisez la commande [az configure][az-configure] avec le paramètre `--defaults`, qui accepte une valeur `acr=REGISTRY_NAME`.

Par exemple, pour configurer l’interface de ligne de commande Azure avec un registre par défaut nommé « myregistry », procédez comme suit :

```azurecli
az configure --defaults acr=myregistry
```

## <a name="task-properties"></a>Propriétés de tâche

Les propriétés de tâche apparaissent généralement dans la partie supérieure d’un fichier `acr-task.yaml` et sont des propriétés globales qui s’appliquent lors de l’exécution complète des étapes de tâche. Certaines de ces propriétés globales peuvent être remplacées au sein d’une étape individuelle.

| Propriété | Type | Facultatif | Description | Remplacement pris en charge | Valeur par défaut |
| -------- | ---- | -------- | ----------- | ------------------ | ------------- |
| `version` | string | Oui | Version du fichier `acr-task.yaml` analysé par le service ACR Tasks. Si ACR Tasks s’efforce de maintenir la compatibilité descendante, cette valeur permet à ACR Tasks d’assurer la compatibilité au sein d’une version définie. En l’absence de version définie explicitement, la dernière version est utilisée par défaut. | Non | None |
| `stepTimeout` | int (secondes) | Oui | Nombre maximal de secondes pendant lesquelles une étape peut être exécutée. Si la propriété est spécifiée sur une tâche, elle définit la propriété `timeout` par défaut de toutes les étapes. Si la propriété `timeout` est spécifiée à une étape, elle remplace la propriété fournie par la tâche. | Oui | 600 (10 minutes) |
| `workingDirectory` | string | Oui | Répertoire de travail du conteneur pendant l’exécution. Si la propriété est spécifiée sur une tâche, elle définit la propriété `workingDirectory` par défaut de toutes les étapes. Si elle est spécifiée à une étape, elle remplace la propriété fournie par la tâche. | Oui | `c:\workspace` sous Windows ou `/workspace` sous Linux |
| `env` | [chaîne, chaîne,...] | Oui |  Tableau de chaînes au format `key=value` qui définissent les variables d’environnement pour la tâche. Si la propriété est spécifiée sur une tâche, elle définit la propriété `env` par défaut de toutes les étapes. Si elle est spécifiée à une étape, elle remplace toutes les variables d’environnement héritées de la tâche. | Oui | None |
| `secrets` | [clé secrète, clé secrète, ...] | Oui | Tableau d’objets [clé secrète](#secret). | Non | None |
| `networks` | [réseau, réseau,...] | Oui | Tableau d’objets [réseau](#network). | Non | None |
| `volumes` | [volume, volume, ...] | Oui | Tableau d’objets [volume](#volume). Spécifie les volumes avec du contenu source à monter dans une étape. | Non | None |

### <a name="secret"></a>secret

L’objet secret comporte les propriétés suivantes.

| Propriété | Type | Facultatif | Description | Valeur par défaut |
| -------- | ---- | -------- | ----------- | ------- |
| `id` | string | Non | L’identificateur du secret. | None |
| `keyvault` | string | Oui | URL de la clé secrète Azure Key Vault. | None |
| `clientID` | string | Oui | ID client de l’[identité managée affectée par l’utilisateur](container-registry-tasks-authentication-managed-identity.md) pour les ressources Azure. | None |

### <a name="network"></a>réseau

L’objet réseau a les propriétés suivantes.

| Propriété | Type | Facultatif | Description | Valeur par défaut |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | string | Non | Nom du réseau. | None |
| `driver` | string | Oui | Pilote de gestion du réseau. | None |
| `ipv6` | bool | Oui | Indique si la mise en réseau IPv6 est activée. | `false` |
| `skipCreation` | bool | Oui | Indique si vous souhaitez ignorer la création du réseau. | `false` |
| `isDefault` | bool | Oui | Indique si le réseau est un réseau par défaut fourni avec Azure Container Registry. | `false` |

### <a name="volume"></a>volume

L’objet volume comporte les propriétés suivantes.

| Propriété | Type | Facultatif | Description | Valeur par défaut |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | string | Non | Nom du volume à monter. Ne peut contenir que des caractères alphanumériques, « - » et « _ ». | None |
| `secret` | map[string]string | Non | Chaque clé du mappage est le nom d’un fichier créé et rempli dans le volume. Chaque valeur est la version de type chaîne du secret. Les valeurs de secret doivent être encodées en base64. | None |

## <a name="task-step-types"></a>Types d’étape de tâche

ACR Tasks prend en charge trois types d’étape. Chaque type d’étape prend en charge plusieurs propriétés, détaillées dans la section correspondante.

| Type d’étape | Description |
| --------- | ----------- |
| [`build`](#build) | Génère une image conteneur à l’aide de la syntaxe `docker build` standard. |
| [`push`](#push) | Exécute un `docker push` d’images tout juste générées ou ré-étiquetées dans un registre de conteneurs. Azure Container Registry, d’autres registres privés et le hub Docker public sont pris en charge. |
| [`cmd`](#cmd) | Exécute un conteneur en tant que commande, avec les paramètres transmis à l’élément `[ENTRYPOINT]` du conteneur. Le type d’étape `cmd` prend en charge les paramètres tels qu’`env`, `detach` et d’autres options de commande `docker run` standard, ce qui permet les tests d’unité et fonctionnels avec l’exécution simultanée de conteneurs. |

## <a name="build"></a>build

Générez une image conteneur. Le type d’étape `build` représente un moyen sécurisé et mutualisé d’exécuter `docker build` dans le cloud en tant que primitive de première classe.

### <a name="syntax-build"></a>Syntaxe : build

```yml
version: v1.1.0
steps:
  - [build]: -t [imageName]:[tag] -f [Dockerfile] [context]
    [property]: [value]
```

Le type d’étape `build` prend en charge les paramètres du tableau suivant. Le type d'étape `build` prend également en charge toutes les options de build de la commande [docker build](https://docs.docker.com/engine/reference/commandline/build/), comme `--build-arg` pour définir des variables de build.

| Paramètre | Description | Facultatif |
| --------- | ----------- | :-------: |
| `-t` &#124; `--image` | Définit l’élément `image:tag` complet de l’image générée.<br /><br />Comme les images peuvent être utilisées pour les validations de tâche internes, telles que les tests fonctionnels, certaines images n’ont pas besoin de `push` vers un registre. Toutefois, l’instanciation d’une image dans une exécution de tâche, l’image a besoin d’un nom comme référence.<br /><br />Contrairement à `az acr build`, l’exécution d’ACR Tasks ne fournit pas de comportement d’envoi (push) par défaut. Avec ACR Tasks, le scénario par défaut suppose la possibilité de générer, valider, puis envoyer une image. Consultez [push](#push) pour savoir comment envoyer (push) les images générées. | Oui |
| `-f` &#124; `--file` | Spécifie le fichier Docker transmis à `docker build`. S’il n’est pas indiqué, le fichier Docker par défaut à la racine du contexte est supposé. Pour spécifier un fichier Docker, transmettez le nom de fichier relatif à la racine du contexte. | Oui |
| `context` | Répertoire racine transmis à `docker build`. Le répertoire racine de chaque tâche est défini pour un [répertoire de travail](#task-step-properties) partagé et inclut la racine du répertoire cloné Git associé. | Non |

### <a name="properties-build"></a>Propriétés : build

Le type d’étape `build` prend en charge les propriétés suivantes : La section [Propriétés d’étape de tâche](#task-step-properties) de cet article comporte les détails de ces propriétés.

| Propriétés | Type | Obligatoire |
| -------- | ---- | -------- |
| `detach` | bool | Facultatif |
| `disableWorkingDirectoryOverride` | bool | Facultatif |
| `entryPoint` | string | Facultatif |
| `env` | [chaîne, chaîne,...] | Facultatif |
| `expose` | [chaîne, chaîne,...] | Facultatif |
| `id` | string | Facultatif |
| `ignoreErrors` | bool | Facultatif |
| `isolation` | string | Facultatif |
| `keep` | bool | Facultatif |
| `network` | object | Facultatif |
| `ports` | [chaîne, chaîne,...] | Facultatif |
| `pull` | bool | Facultatif |
| `repeat` | int | Facultatif |
| `retries` | int | Facultatif |
| `retryDelay` | int (secondes) | Facultatif |
| `secret` | object | Facultatif |
| `startDelay` | int (secondes) | Facultatif |
| `timeout` | int (secondes) | Facultatif |
| `volumeMount` | object | Facultatif |
| `when` | [chaîne, chaîne,...] | Facultatif |
| `workingDirectory` | string | Facultatif |

### <a name="examples-build"></a>Exemples : build

#### <a name="build-image---context-in-root"></a>Générer une image : contexte dans la racine

```azurecli
az acr run -f build-hello-world.yaml https://github.com/AzureCR/acr-tasks-sample.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-hello-world.yaml)]

#### <a name="build-image---context-in-subdirectory"></a>Générer une image : contexte dans un sous-répertoire

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world -f hello-world.dockerfile ./subDirectory
```

## <a name="push"></a>push

Envoie (push) des images générées ou ré-étiquetées vers un registre de conteneurs. Prend en charge l’envoi (push) vers des registres privés comme Azure Container Registry, ou vers le hub Docker public.

### <a name="syntax-push"></a>Syntaxe : push

Le type d’étape `push` prend en charge une collection d’images. La syntaxe de collection YAML prend en charge les formats imbriqués et inline. L’envoi (push) d’une image unique est généralement représenté à l’aide de la syntaxe inline :

```yml
version: v1.1.0
steps:
  # Inline YAML collection syntax
  - push: ["$Registry/hello-world:$ID"]
```

Pour une meilleure lisibilité, utilisez la syntaxe imbriquée lors de l’envoi de plusieurs images :

```yml
version: v1.1.0
steps:
  # Nested YAML collection syntax
  - push:
    - $Registry/hello-world:$ID
    - $Registry/hello-world:latest
```

### <a name="properties-push"></a>Propriétés : push

Le type d’étape `push` prend en charge les propriétés suivantes : La section [Propriétés d’étape de tâche](#task-step-properties) de cet article comporte les détails de ces propriétés.

| Propriété | Type | Obligatoire |
| -------- | ---- | -------- |
| `env` | [chaîne, chaîne,...] | Facultatif |
| `id` | string | Facultatif |
| `ignoreErrors` | bool | Facultatif |
| `startDelay` | int (secondes) | Facultatif |
| `timeout` | int (secondes) | Facultatif |
| `when` | [chaîne, chaîne,...] | Facultatif |

### <a name="examples-push"></a>Exemples : push

#### <a name="push-multiple-images"></a>Envoyer (push) plusieurs images

```azurecli
az acr run -f build-push-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-push-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-push-hello-world.yaml)]

#### <a name="build-push-and-run"></a>Générer, envoyer (push) et exécuter

```azurecli
az acr run -f build-run-hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/build-run-hello-world.yaml -->
[!code-yml[task](~/acr-tasks/build-run-hello-world.yaml)]

## <a name="cmd"></a>cmd

Le type d’étape `cmd` exécute un conteneur.

### <a name="syntax-cmd"></a>Syntaxe : cmd

```yml
version: v1.1.0
steps:
  - [cmd]: [containerImage]:[tag (optional)] [cmdParameters to the image]
```

### <a name="properties-cmd"></a>Propriétés : cmd

Le type d’étape `cmd` prend en charge les propriétés suivantes :

| Propriété | Type | Obligatoire |
| -------- | ---- | -------- |
| `detach` | bool | Facultatif |
| `disableWorkingDirectoryOverride` | bool | Facultatif |
| `entryPoint` | string | Facultatif |
| `env` | [chaîne, chaîne,...] | Facultatif |
| `expose` | [chaîne, chaîne,...] | Facultatif |
| `id` | string | Facultatif |
| `ignoreErrors` | bool | Facultatif |
| `isolation` | string | Facultatif |
| `keep` | bool | Facultatif |
| `network` | object | Facultatif |
| `ports` | [chaîne, chaîne,...] | Facultatif |
| `pull` | bool | Facultatif |
| `repeat` | int | Facultatif |
| `retries` | int | Facultatif |
| `retryDelay` | int (secondes) | Facultatif |
| `secret` | object | Facultatif |
| `startDelay` | int (secondes) | Facultatif |
| `timeout` | int (secondes) | Facultatif |
| `volumeMount` | object | Facultatif |
| `when` | [chaîne, chaîne,...] | Facultatif |
| `workingDirectory` | string | Facultatif |

Vous trouverez les détails de ces propriétés dans la section [Propriétés d’étape de tâche](#task-step-properties) de cet article.

### <a name="examples-cmd"></a>Exemples : cmd

#### <a name="run-hello-world-image"></a>Exécuter l’image hello-world

Cette commande exécute le fichier de tâche `hello-world.yaml`, qui fait référence à l’image [hello-world](https://hub.docker.com/_/hello-world/) sur le hub Docker.

```azurecli
az acr run -f hello-world.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/hello-world.yaml -->
[!code-yml[task](~/acr-tasks/hello-world.yaml)]

#### <a name="run-bash-image-and-echo-hello-world"></a>Exécuter l’image bash et renvoyer « hello world »

Cette commande exécute le fichier de tâche `bash-echo.yaml`, qui fait référence à l’image [bash](https://hub.docker.com/_/bash/) sur le hub Docker.

```azurecli
az acr run -f bash-echo.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo.yaml -->
[!code-yml[task](~/acr-tasks/bash-echo.yaml)]

#### <a name="run-specific-bash-image-tag"></a>Exécuter une balise d’image bash spécifique

Pour exécuter une version d’image spécifique, spécifiez la balise dans `cmd`.

Cette commande exécute le fichier de tâche `bash-echo-3.yaml`, qui fait référence à l’image [bash:3.0](https://hub.docker.com/_/bash/) sur le hub Docker.

```azurecli
az acr run -f bash-echo-3.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/bash-echo-3.yaml -->
[!code-yml[task](~/acr-tasks/bash-echo-3.yaml)]

#### <a name="run-custom-images"></a>Exécuter des images personnalisées

Le type d’étape `cmd` fait référence à des images à l’aide du format `docker run` standard. Les images non précédées d’un registre sont supposées provenir de docker.io. L’exemple précédent peut également être représenté comme suit :

```yml
version: v1.1.0
steps:
  - cmd: docker.io/bash:3.0 echo hello world
```

À l’aide de la convention de référence d’image `docker run` standard, `cmd` peut exécuter des images de n’importe quel registre privé ou dans le Docker Hub public. Si vous faites référence à des images du registre dans lequel ACR Tasks s’exécute, vous n’avez pas besoin de spécifier des informations d’identification de registre.

* Exécutez une image d’un registre de conteneurs Azure. L’exemple suivant part du principe que vous disposez d’un registre nommé `myregistry`, et d’une image personnalisée `myimage:mytag`.

    ```yml
    version: v1.1.0
    steps:
        - cmd: myregistry.azurecr.io/myimage:mytag
    ```

* Généraliser la référence de registre avec une variable d’exécution ou un alias

    Au lieu de coder de manière irréversible le nom de votre registre dans un fichier `acr-task.yaml`, vous pouvez le rendre plus portable en utilisant une [variable d’exécution](#run-variables) ou une [alias](#aliases). La variable `Run.Registry` ou l’alias `$Registry` sont développé lors du runtime pour donner le nom du registre dans lequel la tâche s’exécute.

    Par exemple, pour généraliser la tâche précédente de sorte qu’elle fonctionne dans n’importe quel registre de conteneurs Azure, référencez la variable $Registry dans le nom de l’image :

    ```yml
    version: v1.1.0
    steps:
      - cmd: $Registry/myimage:mytag
    ```

#### <a name="access-secret-volumes"></a>Accéder aux volumes de secrets

La propriété `volumes` permet de spécifier les volumes et les secrets qu’ils contiennent pour les étapes `build` et `cmd` d’une tâche. Dans chaque étape, une propriété facultative `volumeMounts` liste les volumes et les chemins de conteneurs correspondants à monter dans le conteneur à cette étape. Les secrets sont fournis sous forme de fichiers dans le chemin de montage de chaque volume.

Exécutez une tâche et montez deux secrets dans une étape : l’un stocké dans un coffre de clés et l’autre sur la ligne de commande :

```azurecli
az acr run -f mounts-secrets.yaml --set-secret mysecret=abcdefg123456 https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/mounts-secrets.yaml -->
[!code-yml[task](~/acr-tasks/mounts-secrets.yaml)]

## <a name="task-step-properties"></a>Propriétés d’étape de tâche

Chaque type d’étape prend en charge plusieurs propriétés appropriées pour son type. Le tableau suivant définit toutes les propriétés d’étape disponibles. Certains types d’étape ne prennent pas en charge toutes les propriétés. Pour connaître les propriétés disponibles pour chaque type d’étape, consultez les sections de référence sur les types d’étape [cmd](#cmd), [build](#build) et [push](#push).

| Propriété | Type | Facultatif | Description | Valeur par défaut |
| -------- | ---- | -------- | ----------- | ------- |
| `detach` | bool | Oui | Indique si le conteneur doit être détaché lors de l’exécution. | `false` |
| `disableWorkingDirectoryOverride` | bool | Oui | Indique s’il faut désactiver la fonctionnalité de remplacement de `workingDirectory`. Utilisez cette option conjointement avec `workingDirectory` pour avoir un contrôle total sur le répertoire de travail du conteneur. | `false` |
| `entryPoint` | string | Oui | Remplace le `[ENTRYPOINT]` du conteneur d’une étape. | None |
| `env` | [chaîne, chaîne,...] | Oui | Tableau de chaînes au format `key=value` qui définissent les variables d’environnement pour l’étape. | None |
| `expose` | [chaîne, chaîne,...] | Oui | Tableau des ports qui sont exposés à partir du conteneur. |  None |
| [`id`](#example-id) | string | Oui | Identifie l’étape de la tâche de manière unique. Les autres étapes de la tâche peuvent faire référence à l’élément `id` d’une étape (par exemple, pour la vérification des dépendances avec `when`).<br /><br />L’élément `id` est également le nom du conteneur en cours d’exécution. Les processus qui s’exécutent dans d’autres conteneurs de la tâche peuvent faire référence à l’élément `id` avec son nom d’hôte DNS ou pour y accéder avec les journaux d’activité Docker [id], par exemple. | `acb_step_%d`, où `%d` est l’index basé sur 0 de l’étape descendante dans le fichier YAML |
| `ignoreErrors` | bool | Oui | Si vous souhaitez marquer l’état comme réussie, même si une erreur s’est produite pendant l’exécution du conteneur. | `false` |
| `isolation` | string | Oui | Niveau d’isolation du conteneur. | `default` |
| `keep` | bool | Oui | Indique si le conteneur de l’étape doit être conservé après l’exécution. | `false` |
| `network` | object | Oui | Identifie un réseau dans lequel le conteneur s’exécute. | None |
| `ports` | [chaîne, chaîne,...] | Oui | Tableau des ports qui sont publiés sur l’hôte à partir du conteneur. |  None |
| `pull` | bool | Oui | Indique s’il faut ou non forcer une opération d’extraction du conteneur avant son exécution pour éviter tout comportement de mise en cache. | `false` |
| `privileged` | bool | Oui | Indique s’il faut ou non exécuter le conteneur en mode privilégié. | `false` |
| `repeat` | int | Oui | Nombre de tentatives d’exécution d’un conteneur. | 0 |
| `retries` | int | Oui | Nombre de tentatives à renouveler si un conteneur échoue dans son exécution. Une nouvelle tentative n’a lieu que si le code de sortie d’un conteneur est différent de zéro. | 0 |
| `retryDelay` | int (secondes) | Oui | Délai en secondes entre les tentatives d’exécution du conteneur. | 0 |
| `secret` | object | Oui | Identifie une clé secrète Azure Key Vault ou une [identité managée pour les ressources Azure](container-registry-tasks-authentication-managed-identity.md). | None |
| `startDelay` | int (secondes) | Oui | Nombre de secondes de décalage pour l’exécution d’un conteneur. | 0 |
| `timeout` | int (secondes) | Oui | Nombre maximal de secondes pendant lesquelles une étape peut s’exécuter avant d’être terminée. | 600 |
| [`when`](#example-when) | [chaîne, chaîne,...] | Oui | Configure les dépendances d’une étape sur une ou plusieurs autres étapes au sein de la tâche. | None |
| `user` | string | Oui | Nom d’utilisateur ou UID d’un conteneur | None |
| `workingDirectory` | string | Oui | Définit le répertoire de travail pour une étape. Par défaut, ACR Tasks crée un répertoire racine comme répertoire de travail. Toutefois, si votre build dispose de plusieurs étapes, les étapes précédentes peuvent partager des artefacts avec étapes suivantes en spécifiant le même répertoire de travail. | `c:\workspace` sous Windows ou `/workspace` sous Linux |

### <a name="volumemount"></a>volumeMount

L’objet volumeMount comporte les propriétés suivantes.

| Propriété | Type | Facultatif | Description | Valeur par défaut |
| -------- | ---- | -------- | ----------- | ------- | 
| `name` | string | Non | Nom du volume à monter. Doit correspondre exactement au nom d’une propriété `volumes`. | None |
| `mountPath`   | string | non | Chemin absolu pour monter des fichiers dans le conteneur.  | None |

### <a name="examples-task-step-properties"></a>Exemples : Propriétés d’étape de tâche

#### <a name="example-id"></a>Exemple : id

Générez deux images, en instanciant une image de test fonctionnel. Chaque étape est identifiée par un élément `id` unique mentionné dans d’autres étapes de la tâche dans leur propriété `when`.

```azurecli
az acr run -f when-parallel-dependent.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel-dependent.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel-dependent.yaml)]

#### <a name="example-when"></a>Exemple : when

La propriété `when` spécifie les dépendances d’une étape sur d’autres étapes au sein de la tâche. Elle prend en charge deux valeurs de paramètre :

* `when: ["-"]` : indique l’absence de dépendance sur d’autres étapes. Une étape indiquant `when: ["-"]` est exécutée immédiatement et permet l’exécution simultanée des étapes.
* `when: ["id1", "id2"]` : indique que l’étape dépend des étapes avec `id` « id1 » et `id` « id2 ». Cette étape ne sera pas être exécutée tant que les étapes « id1 » et « id2 » ne seront pas terminées.

Si `when` n’est pas spécifié dans une étape, cette étape dépend de l’achèvement de l’étape précédente dans le fichier `acr-task.yaml`.

Exécution séquentielle des étapes sans `when` :

```azurecli
az acr run -f when-sequential-default.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-sequential-default.yaml -->
[!code-yml[task](~/acr-tasks/when-sequential-default.yaml)]

Exécution séquentielle des étapes avec `when` :

```azurecli
az acr run -f when-sequential-id.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-sequential-id.yaml -->
[!code-yml[task](~/acr-tasks/when-sequential-id.yaml)]

Génération d’images en parallèle :

```azurecli
az acr run -f when-parallel.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel.yaml)]

Génération d’images en parallèle et test des dépendances :

```azurecli
az acr run -f when-parallel-dependent.yaml https://github.com/Azure-Samples/acr-tasks.git
```

<!-- SOURCE: https://github.com/Azure-Samples/acr-tasks/blob/master/when-parallel-dependent.yaml -->
[!code-yml[task](~/acr-tasks/when-parallel-dependent.yaml)]

## <a name="run-variables"></a>Exécuter des variables

ACR Tasks inclut un ensemble de variables par défaut qui sont disponibles pour les étapes de tâche lors de l’exécution. Ces variables sont accessibles à l’aide du format `{{.Run.VariableName}}`, où `VariableName` est une des valeurs suivantes :

* `Run.ID`
* `Run.SharedVolume`
* `Run.Registry`
* `Run.RegistryName`
* `Run.Date`
* `Run.OS`
* `Run.Architecture`
* `Run.Commit`
* `Run.Branch`
* `Run.TaskName`

Les noms des variables sont généralement explicites. Des détails suivent pour les variables couramment utilisées. À partir de la version YAML `v1.1.0`, vous pouvez utiliser un [alias de tâche](#aliases) prédéfini abrégé à la place de la plupart des variables d’exécution. Par exemple, à la place de `{{.Run.Registry}}`, utilisez l’alias `$Registry`.

### <a name="runid"></a>Run.ID

Chaque exécution, via `az acr run` ou lancée par un déclencheur, des tâches créées avec `az acr task create` a un ID unique. L’ID représente l’exécution en cours.

Généralement utilisé pour un balisage qui identifie de façon unique une image :

```yml
version: v1.1.0
steps:
    - build: -t $Registry/hello-world:$ID .
```

### <a name="runsharedvolume"></a>Run.SharedVolume

Identificateur unique d’un volume partagé accessible par toutes les étapes de la tâche. Le volume est monté sur `c:\workspace` sous Windows ou `/workspace` sous Linux. 

### <a name="runregistry"></a>Run.Registry

Nom complet du serveur du registre. Généralement utilisé pour référencer de façon générique le registre dans lequel la tâche est en cours d’exécution.

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID .
```

### <a name="runregistryname"></a>Run.RegistryName

Nom du registre de conteneurs. Généralement utilisé dans des étapes de tâche ne requérant pas de nom de serveur complet, par exemple, `cmd` étapes qui exécutent des commandes Azure CLI sur des registres.

```yml
version 1.1.0
steps:
# List repositories in registry
- cmd: az login --identity
- cmd: az acr repository list --name $RegistryName
```

### <a name="rundate"></a>Run.Date

Heure UTC de début de l’exécution.

### <a name="runcommit"></a>Run.Commit

Pour une tâche déclenchée par un commit dans un référentiel GitHub, l’identificateur du commit.

### <a name="runbranch"></a>Run.Branch

Pour une tâche déclenchée par un commit dans un référentiel GitHub, le nom de la branche.

## <a name="aliases"></a>Alias

Depuis `v1.1.0`, Azure Container Registry Tasks prend en charge les alias disponibles pour les étapes de tâche quand elles s’exécutent. Les alias ont un concept similaire à celui des alias (raccourcis de commande) pris en charge dans Bash et d’autres interpréteurs de commandes. 

Avec un alias, vous pouvez lancer toute commande ou tout  groupe de commandes (y compris des options et des noms de fichiers) en entrant un mot.

ACR Tasks prend en charge plusieurs alias prédéfinis ainsi que des alias personnalisés que vous créez.

### <a name="predefined-aliases"></a>Alias prédéfinis

Vous pouvez utiliser les alias de tâches suivants à la place des [variables d’exécution](#run-variables) :

| Alias | Exécuter une variable |
| ----- | ------------ |
| `ID` | `Run.ID` |
| `SharedVolume` | `Run.SharedVolume` |
| `Registry` | `Run.Registry` |
| `RegistryName` | `Run.RegistryName` |
| `Date` | `Run.Date` |
| `OS` | `Run.OS` |
| `Architecture` | `Run.Architecture` |
| `Commit` | `Run.Commit` |
| `Branch` | `Run.Branch` |

Dans les étapes de la tâche, faites précéder un alias de la directive `$`, comme dans cet exemple :

```yml
version: v1.1.0
steps:
  - build: -t $Registry/hello-world:$ID -f hello-world.dockerfile .
```

### <a name="image-aliases"></a>alias d’image

Chacun des points d’alias suivants pointe vers une image stable dans le Registre de conteneurs Microsoft. Vous pouvez faire référence à chacun d’eux dans la section `cmd` d’un fichier de tâches sans utiliser de directive.

| Alias | Image |
| ----- | ----- |
| `acr` | `mcr.microsoft.com/acr/acr-cli:0.1` |
| `az` | `mcr.microsoft.com/acr/azure-cli:a80af84` |
| `bash` | `mcr.microsoft.com/acr/bash:a80af84` |
| `curl` | `mcr.microsoft.com/acr/curl:a80af84` |

L’exemple de tâche suivant utilise plusieurs alias pour [purger](container-registry-auto-purge.md) les balises d’image de plus de 7 jours dans le référentiel `samples/hello-world` dans le registre d’exécution :

```yml
version: v1.1.0
steps:
  - cmd: acr tag list --registry $RegistryName --repository samples/hello-world
  - cmd: acr purge --registry $RegistryName --filter samples/hello-world:.* --ago 7d
```

### <a name="custom-alias"></a>Alias personnalisé

Définissez un alias personnalisé dans votre fichier YAML et utilisez-le comme indiqué dans l’exemple suivant. Un alias ne peut contenir que des caractères alphanumériques. La directive par défaut pour développer un alias est le caractère `$`.

```yml
version: v1.1.0
alias:
  values:
    repo: myrepo
steps:
  - build: -t $Registry/$repo/hello-world:$ID -f Dockerfile .
```

Vous pouvez créer un lien vers un fichier YAML distant ou local pour les définitions d’alias personnalisées. L’exemple suivant établit un lien vers un fichier YAML dans le Stockage Blob Azure :

```yml
version: v1.1.0
alias:
  src:  # link to local or remote custom alias files
    - 'https://link/to/blob/remoteAliases.yml?readSasToken'
[...]
```

## <a name="next-steps"></a>Étapes suivantes

Pour une vue d’ensemble des tâches en plusieurs étapes, consultez la rubrique [Exécuter des tâches de génération, de test et de correction en plusieurs étapes dans les ACR Tasks](container-registry-tasks-multi-step.md).

Pour les builds à une seule étape, consultez la [vue d’ensemble d’ACR Tasks](container-registry-tasks-overview.md).



<!-- IMAGES -->

<!-- LINKS - External -->
[acr-tasks]: https://github.com/Azure-Samples/acr-tasks

<!-- LINKS - Internal -->
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-configure]: /cli/azure/reference-index#az-configure
