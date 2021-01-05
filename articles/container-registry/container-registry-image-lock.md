---
title: Verrouiller des images
description: Définissez des attributs pour une image conteneur ou un référentiel afin qu’il ne puisse pas être supprimé ou remplacé dans un registre de conteneurs Azure.
ms.topic: article
ms.date: 09/30/2019
ms.openlocfilehash: da84767523bb6d948b71b1c1ad2ddaffb628354a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77659694"
---
# <a name="lock-a-container-image-in-an-azure-container-registry"></a>Verrouiller une image dans un registre de conteneurs Azure

Dans un registre de conteneurs Azure, vous pouvez verrouiller une version d’image ou un référentiel afin qu’il ne puisse pas être supprimé ou mis à jour. Pour verrouiller une image ou un référentiel, mettez à jour ses attributs à l’aide de la commande Azure CLI [az acr repository update][az-acr-repository-update]. 

Cet article nécessite que vous exécutiez Azure CLI dans Azure Cloud Shell ou localement (version 2.0.55 ou ultérieure recommandée). Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli].

> [!IMPORTANT]
> Cet article ne s’applique pas au verrouillage d’un registre entier, par exemple à l’aide de **Paramètres > Verrous** dans le Portail Azure, ou des commandes `az lock` dans l’interface de ligne de commande Azure. Le verrouillage d’une ressource du registre ne vous empêche pas de créer, de mettre à jour ou de supprimer des données dans des référentiels. Le verrouillage d’un registre affecte uniquement les opérations de gestion telles que l’ajout ou la suppression de réplications, ou la suppression du registre lui-même. Vous trouverez plus d’informations dans [Verrouiller les ressources pour empêcher les modifications inattendues](../azure-resource-manager/management/lock-resources.md).

## <a name="scenarios"></a>Scénarios

Par défaut, une image avec balise dans Azure Container Registry est *mutable*. Ainsi, avec les autorisations appropriées vous pouvez mettre à jour et envoyer de manière répétée une image avec la même balise vers un registre. Les images conteneur peuvent également être [supprimées](container-registry-delete.md) en fonction des besoins. Ce comportement est utile quand vous développez des images et que vous devez conserver une certaine taille de registre.

En revanche, quand vous déployez une image conteneur en production vous aurez peut-être besoin d’une image conteneur *immuable*. Une image immuable est une image qui ne peut pas être supprimée ou remplacée accidentellement.

Consultez les [Suggestions pour la création de balises et de versions pour les images de conteneurs](container-registry-image-tag-version.md) pour des stratégies de création de balises et de versions pour les images dans votre registre.

Utilisez la commande [az acr repository update][az-acr-repository-update] pour définir les attributs du référentiel afin de pouvoir :

* Verrouiller une version d’image ou un référentiel entier.

* Protéger une version d’image ou un référentiel contre la suppression, mais autoriser les mises à jour.

* Empêcher les opérations de lecture (pull) sur une version d’image ou un référentiel entier.

Pour obtenir des exemples, consultez les sections suivantes. 

## <a name="lock-an-image-or-repository"></a>Verrouiller une image ou un référentiel 

### <a name="show-the-current-repository-attributes"></a>Afficher les attributs actuels d’un référentiel
Pour afficher les attributs actuels d’un référentiel, exécutez la commande [az acr repository show][az-acr-repository-show] suivante :

```azurecli
az acr repository show \
    --name myregistry --repository myrepo \
    --output jsonc
```

### <a name="show-the-current-image-attributes"></a>Afficher les attributs actuels d’une image
Pour afficher les attributs actuels d’une balise, exécutez la commande [az acr repository show][az-acr-repository-show] suivante :

```azurecli
az acr repository show \
    --name myregistry --image image:tag \
    --output jsonc
```

### <a name="lock-an-image-by-tag"></a>Verrouiller une image par balise

Pour verrouiller l’image *myrepo/myimage:tag* dans *myregistry*, exécutez la commande [az acr repository update][az-acr-repository-update] suivante :

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --write-enabled false
```

### <a name="lock-an-image-by-manifest-digest"></a>Verrouiller une image par code de hachage du manifeste

Pour verrouiller une image *myrepo/myimage* identifiée par le code de hachage du manifeste (hachage SHA-256, représenté sous la forme `sha256:...`), exécutez la commande suivante. (Pour connaître le code de hachage du manifeste associé à une ou plusieurs balises d’image, exécutez la commande [az acr repository show-manifests][az-acr-repository-show-manifests].)

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage@sha256:123456abcdefg \
    --write-enabled false
```

### <a name="lock-a-repository"></a>Verrouiller un référentiel

Pour verrouiller le référentiel *myrepo/myimage* et toutes les images qu’il contient, exécutez la commande suivante :

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --write-enabled false
```

## <a name="protect-an-image-or-repository-from-deletion"></a>Protéger une image ou un référentiel contre la suppression

### <a name="protect-an-image-from-deletion"></a>Protéger une image contre la suppression

Pour autoriser la mise à jour de l’image *myrepo/myimage:tag* tout en empêchant sa suppression, exécutez la commande suivante :

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --delete-enabled false --write-enabled true
```

### <a name="protect-a-repository-from-deletion"></a>Protéger un référentiel contre la suppression

La commande suivante configure le référentiel *myrepo/myimage* afin qu’il ne puisse pas être supprimé. Les différentes images peuvent toujours être mises à jour ou supprimées.

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --delete-enabled false --write-enabled true
```

## <a name="prevent-read-operations-on-an-image-or-repository"></a>Empêcher les opérations de lecture sur une image ou un référentiel

Pour empêcher les opérations de lecture (pull) sur l’image *myrepo/myimage:tag*, exécutez la commande suivante :

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --read-enabled false
```

Pour empêcher les opérations de lecture sur toutes les images dans le référentiel *myrepo/myimage*, exécutez la commande suivante :

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --read-enabled false
```

## <a name="unlock-an-image-or-repository"></a>Déverrouiller une image ou un référentiel

Pour restaurer le comportement par défaut de l’image *myrepo/myimage:tag* afin qu’elle puisse être supprimée et mise à jour, exécutez la commande suivante :

```azurecli
az acr repository update \
    --name myregistry --image myrepo/myimage:tag \
    --delete-enabled true --write-enabled true
```

Pour restaurer le comportement par défaut du référentiel et de toutes les images *myrepo/myimage* afin qu’ils puissent être supprimés et mis à jour, exécutez la commande suivante :

```azurecli
az acr repository update \
    --name myregistry --repository myrepo/myimage \
    --delete-enabled true --write-enabled true
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à utiliser la commande [az acr repository update][az-acr-repository-update] pour empêcher la suppression ou la mise à jour des versions d’images dans un référentiel. Pour définir des attributs supplémentaires, consultez la référence de la commande [az acr repository update][az-acr-repository-update].

Pour afficher les attributs définis pour une version d’image ou un référentiel, utilisez la commande [az acr repository show][az-acr-repository-show].

Pour plus d’informations sur les opérations de suppression, consultez [Supprimer des images conteneur dans Azure Container Registry][container-registry-delete].

<!-- LINKS - Internal -->
[az-acr-repository-update]: /cli/azure/acr/repository#az-acr-repository-update
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-show-manifests]: /cli/azure/acr/repository#az-acr-repository-show-manifests
[azure-cli]: /cli/azure/install-azure-cli
[container-registry-delete]: container-registry-delete.md

