---
title: Monter le volume gitRepo pour le groupe de conteneurs
description: Découvrez comment monter un volume GitRepo pour cloner un référentiel Git dans vos instances de conteneurs
ms.topic: article
ms.date: 06/15/2018
ms.openlocfilehash: 5f361b0bbd4a02f754e47c8321a664af05036601
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260716"
---
# <a name="mount-a-gitrepo-volume-in-azure-container-instances"></a>Monter un volume GitRepo dans Azure Container Instances

Découvrez comment monter un volume *GitRepo* pour cloner un référentiel Git dans vos instances de conteneurs.

> [!NOTE]
> Le montage d’un volume *GitRepo* est actuellement limité aux conteneurs Linux. Nous travaillons actuellement à proposer toutes ces fonctionnalités dans des conteneurs Windows. En attendant, nous vous invitons à découvrir les différences actuelles de la plateforme dans la [Vue d’ensemble](container-instances-overview.md#linux-and-windows-containers).

## <a name="gitrepo-volume"></a>Volumes GitRepo

Le volume *GitRepo* monte un répertoire et clone le référentiel Git spécifié dans celui-ci au démarrage du conteneur. L’utilisation d’un volume *GitRepo* dans vos instances de conteneur permet d’éviter d’ajouter du code à cette fin dans vos applications.

Lorsque vous montez un volume *GitRepo*, vous pouvez définir trois propriétés pour le configurer :

| Propriété | Obligatoire | Description |
| -------- | -------- | ----------- |
| `repository` | Oui | URL complète incluant `http://` ou `https://` du référentiel Git à cloner.|
| `directory` | Non | Répertoire dans lequel le référentiel doit être cloné. Le chemin d’accès ne peut pas contenir ou commencer par « `..` ».  Si vous spécifiez « `.` », le référentiel est cloné dans le répertoire du volume. Autrement, le référentiel Git est cloné dans un sous-répertoire du nom spécifié à l’intérieur du répertoire de volume. |
| `revision` | Non | Hachage de validation de la révision à cloner. À défaut de spécification, la révision `HEAD` est clonée. |

## <a name="mount-gitrepo-volume-azure-cli"></a>Monter un volume gitRepo : Azure CLI

Pour monter un volume gitRepo lorsque vous déployez des instances de conteneurs avec [Azure CLI](/cli/azure), fournissez les paramètres `--gitrepo-url` et `--gitrepo-mount-path` à la commande [az container create][az-container-create]. Vous pouvez éventuellement spécifier le répertoire dans le volume où effectuer le clonage (`--gitrepo-dir`) et le hachage de validation de la révision à cloner (`--gitrepo-revision`).

Cet exemple de commande clone l’exemple d’application Microsoft [aci-helloworld][aci-helloworld] dans `/mnt/aci-helloworld` au sein de l’instance de conteneur :

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name hellogitrepo \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --dns-name-label aci-demo \
    --ports 80 \
    --gitrepo-url https://github.com/Azure-Samples/aci-helloworld \
    --gitrepo-mount-path /mnt/aci-helloworld
```

Pour vérifier que le volume gitRepo a été monté, lancez un interpréteur de commandes dans le conteneur avec [az container exec][az-container-exec] et dressez la liste du répertoire :

```azurecli
az container exec --resource-group myResourceGroup --name hellogitrepo --exec-command /bin/sh
```

```output
/usr/src/app # ls -l /mnt/aci-helloworld/
total 16
-rw-r--r--    1 root     root           144 Apr 16 16:35 Dockerfile
-rw-r--r--    1 root     root          1162 Apr 16 16:35 LICENSE
-rw-r--r--    1 root     root          1237 Apr 16 16:35 README.md
drwxr-xr-x    2 root     root          4096 Apr 16 16:35 app
```

## <a name="mount-gitrepo-volume-resource-manager"></a>Monter un volume gitRepo : Gestionnaire de ressources

Pour monter un volume gitRepo lorsque vous déployez des instances de conteneurs avec un [modèle Azure Resource Manager](/azure/templates/microsoft.containerinstance/containergroups), commencez par renseigner le tableau `volumes` dans la section `properties` du groupe de conteneurs du modèle. Ensuite, pour chaque conteneur du groupe de conteneurs dans lequel vous souhaitez monter le volume *GitRepo*, renseignez le tableau `volumeMounts` dans la section `properties` de la définition de conteneur.

Par exemple, le modèle Resource Manager suivant crée un groupe de conteneurs consistant en un seul conteneur. Le conteneur clone deux référentiels GitHub spécifiés par les blocs de volume *GitRepo*. Le deuxième volume inclut des propriétés supplémentaires spécifiant un répertoire vers lequel cloner, ainsi que le hachage de validation d’une révision spécifique à cloner.

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-gitrepo.json -->
[!code-json[volume-gitrepo](~/azure-docs-json-samples/container-instances/aci-deploy-volume-gitrepo.json)]

La structure ainsi obtenue de répertoire des deux référentiels clonés définis dans le modèle précédent est la suivante :

```
/mnt/repo1/aci-helloworld
/mnt/repo2/my-custom-clone-directory
```

Pour voir un exemple de déploiement d’instance de conteneur avec un modèle Azure Resource Manager, consultez [Déployer des groupes de plusieurs conteneurs dans Azure Container Instances](container-instances-multi-container-group.md).

## <a name="private-git-repo-authentication"></a>Authentification des dépôts Git privés

Pour monter un volume gitRepo pour un dépôt Git privé, spécifiez les informations d’identification dans l’URL du dépôt. En règle générale, les informations d’identification se présentent sous la forme d’un nom d’utilisateur et d’un jeton d’accès personnel (PAT) qui accorde un accès limité au dépôt.

Par exemple, le paramètre `--gitrepo-url` de l’interface de ligne de commande Azure pour un dépôt GitHub privé doit ressembler à ce qui suit (où « gituser » est le nom d’utilisateur GitHub et « abcdef1234fdsa4321abcdef » est le jeton d’accès personnel de l’utilisateur) :

```console
--gitrepo-url https://gituser:abcdef1234fdsa4321abcdef@github.com/GitUser/some-private-repository
```

Pour un dépôt Git Azure Repos, spécifiez n’importe quel nom d’utilisateur (vous pouvez utiliser « azurereposuser » comme dans l’exemple suivant) en association avec un jeton PAT valide :

```console
--gitrepo-url https://azurereposuser:abcdef1234fdsa4321abcdef@dev.azure.com/your-org/_git/some-private-repository
```

Pour plus d’informations sur les jetons d’accès personnel pour GitHub et Azure Repos, consultez les articles suivants :

GitHub : [Création d'un jeton d'accès personnel pour la ligne de commande][pat-github]

Azure Repos : [Créer des jetons d’accès personnel pour authentifier l’accès][pat-repos]

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment monter d’autres types de volumes dans Azure Container Instances :

* [Monter un partage de fichiers Azure dans Azure Container Instances](container-instances-volume-azure-files.md)
* [Monter un volume emptyDir dans Azure Container Instances](container-instances-volume-emptydir.md)
* [Monter un volume secret dans Azure Container Instances](container-instances-volume-secret.md)

<!-- LINKS - External -->
[aci-helloworld]: https://github.com/Azure-Samples/aci-helloworld
[pat-github]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/
[pat-repos]: /azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
