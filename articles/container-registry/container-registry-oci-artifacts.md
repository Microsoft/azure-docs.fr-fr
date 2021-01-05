---
title: Envoyer (push) et tirer (pull) un artefact OCI
description: Envoyer (push) et tirer (pull) des artefacts OCI (Open Container Initiative) à l’aide d’un registre de conteneurs privé dans Azure
author: SteveLasker
manager: gwallace
ms.topic: article
ms.date: 08/12/2020
ms.author: stevelas
ms.openlocfilehash: 7c95766cc12b281521fa52ab113fadd4321d0815
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89485001"
---
# <a name="push-and-pull-an-oci-artifact-using-an-azure-container-registry"></a>Envoyer (push) et tirer (pull) un artefact OCI à l’aide d’un registre de conteneurs Azure

Vous pouvez utiliser un registre de conteneurs Azure pour stocker et gérer des [artefacts OCI (Open Container Initiative)](container-registry-image-formats.md#oci-artifacts) ainsi que des images de conteneur Docker et compatibles Docker.

Pour illustrer cette fonctionnalité, cet article montre comment utiliser l’outil [ORAS (OCI Registry as Storage)](https://github.com/deislabs/oras) pour envoyer (push) un exemple d’artefact (un fichier texte) vers un registre de conteneurs Azure. Ensuite, l’artefact est tiré (pull) du registre. Vous pouvez gérer divers artefacts OCI dans un registre de conteneurs Azure à l’aide de différents outils en ligne de commande adaptés à chaque artefact.

## <a name="prerequisites"></a>Conditions préalables requises

* **Azure Container Registry** : créez un Registre de conteneur dans votre abonnement Azure. Par exemple, utilisez le [portail Azure](container-registry-get-started-portal.md) ou [Azure CLI](container-registry-get-started-azure-cli.md).
* **Outil ORAS** : téléchargez et installez une version ORAS à jour pour votre système d’exploitation à partir du [dépôt GitHub](https://github.com/deislabs/oras/releases). L’outil est publié en tant que tarball compressé (fichier `.tar.gz`). Extrayez et installez le fichier à l’aide des procédures standard pour votre système d’exploitation.
* **Principal de service Azure Active Directory (facultatif)**  : pour vous authentifier directement auprès d’ORAS, créez un [principal de service](container-registry-auth-service-principal.md) pour accéder à votre registre. Assurez-vous que le principal de service se voit attribuer un rôle tel qu’AcrPush afin qu’il dispose d’autorisations pour envoyer (push) et tirer (pull) des artefacts.
* **Azure CLI (facultatif)**  : pour utiliser une identité individuelle, vous avez besoin d’une installation locale d’Azure CLI. La version 2.0.71 ou ultérieure est recommandée. Exécutez `az --version ` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).
* **Docker (facultatif)**  : pour utiliser une identité individuelle, vous devez également disposer de Docker localement afin de vous authentifier auprès du registre. Docker fournit des packages qui le configurent facilement sur n’importe quel système [macOS][docker-mac], [Windows][docker-windows] ou [Linux][docker-linux].


## <a name="sign-in-to-a-registry"></a>Se connecter à un registre

Cette section présente deux workflows suggérés pour se connecter au registre, en fonction de l’identité utilisée. Choisissez la méthode appropriée pour votre environnement.

### <a name="sign-in-with-oras"></a>Se connecter avec ORAS

À l’aide d’un [principal de service](container-registry-auth-service-principal.md) doté de droits d’envoi (push), exécutez la commande `oras login` pour vous connecter au registre au moyen de l’ID d’application et du mot de passe du principal de service. Spécifiez le nom de registre complet (tout en minuscules), dans ce cas *myregistry.azurecr.io*. L’ID d’application du principal de service est transmis dans la variable d’environnement `$SP_APP_ID`, et le mot de passe dans la variable `$SP_PASSWD`.

```bash
oras login myregistry.azurecr.io --username $SP_APP_ID --password $SP_PASSWD
```

Pour lire le mot de passe à partir de Stdin, utilisez `--password-stdin`.

### <a name="sign-in-with-azure-cli"></a>Se connecter avec Azure CLI

[Connectez-vous](/cli/azure/authenticate-azure-cli) à Azure CLI avec votre identité pour envoyer (push) et tirer (pull) des artefacts à partir du registre de conteneurs.

Ensuite, utilisez la commande Azure CLI [az acr login](/cli/azure/acr?view=azure-cli-latest#az-acr-login) pour accéder au registre. Par exemple, pour vous authentifier auprès d’un registre nommé *myregistry* :

```azurecli
az login
az acr login --name myregistry
```

> [!NOTE]
> `az acr login` utilise le client Docker pour définir un jeton Azure Active Directory dans le fichier `docker.config`. Le client Docker doit être installé et en cours d’exécution pour terminer le flux d’authentification individuel.

## <a name="push-an-artifact"></a>Envoyer (push) un artefact

Créez un fichier texte dans un répertoire de travail local avec un exemple de texte. Par exemple, dans un interpréteur de commandes bash :

```bash
echo "Here is an artifact!" > artifact.txt
```

Utilisez la commande `oras push` pour envoyer (push) ce fichier texte vers votre registre. L’exemple suivant envoie l’exemple de fichier texte vers le dépôt `samples/artifact`. Le registre est identifié par le nom de registre complet *myregistry.azurecr.io* (tout en minuscules). L’artefact est étiqueté `1.0`. L’artefact a un type non défini, par défaut, identifié par la chaîne de *type de média* qui suit le nom de fichier `artifact.txt`. Consultez [OCI Artifacts](https://github.com/opencontainers/artifacts) pour obtenir d’autres types. 

**Linux**

```bash
oras push myregistry.azurecr.io/samples/artifact:1.0 \
    --manifest-config /dev/null:application/vnd.unknown.config.v1+json \
    ./artifact.txt:application/vnd.unknown.layer.v1+txt
```

**Windows**

```cmd
.\oras.exe push myregistry.azurecr.io/samples/artifact:1.0 ^
    --manifest-config NUL:application/vnd.unknown.config.v1+json ^
    .\artifact.txt:application/vnd.unknown.layer.v1+txt
```

La sortie d’un envoi (push) exécuté avec succès ressemble à ce qui suit :

```console
Uploading 33998889555f artifact.txt
Pushed myregistry.azurecr.io/samples/artifact:1.0
Digest: sha256:xxxxxxbc912ef63e69136f05f1078dbf8d00960a79ee73c210eb2a5f65xxxxxx
```

Pour gérer les artefacts dans votre registre, si vous utilisez Azure CLI, exécutez des commandes `az acr` standard pour gérer les images. Par exemple, récupérez les attributs de l’artefact à l’aide de la commande [az acr repository show][az-acr-repository-show] :

```azurecli
az acr repository show \
    --name myregistry \
    --image samples/artifact:1.0
```

Le résultat ressemble à ce qui suit :

```json
{
  "changeableAttributes": {
    "deleteEnabled": true,
    "listEnabled": true,
    "readEnabled": true,
    "writeEnabled": true
  },
  "createdTime": "2019-08-28T20:43:31.0001687Z",
  "digest": "sha256:xxxxxxbc912ef63e69136f05f1078dbf8d00960a79ee73c210eb2a5f65xxxxxx",
  "lastUpdateTime": "2019-08-28T20:43:31.0001687Z",
  "name": "1.0",
  "signed": false
}
```

## <a name="pull-an-artifact"></a>Tirer (pull) un artefact

Exécutez la commande `oras pull` pour tirer (pull) l’artefact de votre registre.

Tout d’abord, supprimez le fichier texte de votre répertoire de travail local :

```bash
rm artifact.txt
```

Exécutez `oras pull` pour tirer (pull) l’artefact en spécifiant le type de média utilisé pour envoyer (push) l’artefact :

```bash
oras pull myregistry.azurecr.io/samples/artifact:1.0 \
    --media-type application/vnd.unknown.layer.v1+txt
```

Vérifiez que le tirage (pull) a réussi :

```bash
$ cat artifact.txt
Here is an artifact!
```

## <a name="remove-the-artifact-optional"></a>Supprimer l’artefact (facultatif)

Pour supprimer l’artefact de votre registre de conteneurs Azure, utilisez la commande [az acr repository delete][az-acr-repository-delete]. L’exemple suivant supprime l’artefact que vous y avez stocké :

```azurecli
az acr repository delete \
    --name myregistry \
    --image samples/artifact:1.0
```

## <a name="example-build-docker-image-from-oci-artifact"></a>Exemple : Générer l’image Docker à partir de l’artefact OCI

Le code source et les fichiers binaires pour générer une image conteneur peuvent être stockés en tant qu’artefacts OCI dans un registre de conteneurs Azure. Vous pouvez référencer un artefact source comme contexte de génération pour une [tâche ACR](container-registry-tasks-overview.md). Cet exemple montre comment stocker un fichier Dockerfile en tant qu’artefact OCI, puis référencer l’artefact pour générer une image conteneur.

Par exemple, créez un fichier Dockerfile avec une seule ligne :

```bash
echo "FROM hello-world" > hello-world.dockerfile
```

Connectez-vous au registre de conteneurs de destination.

```azurecli
az login
az acr login --name myregistry
```

Créez et envoyez un nouvel artefact OCI au registre de destination en utilisant la commande `oras push`. Cet exemple définit le type de média par défaut pour l’artefact.

```bash
oras push myregistry.azurecr.io/hello-world:1.0 hello-world.dockerfile
```

Exécutez la commande [az acr build](/cli/azure/acr#az-acr-build) pour générer l’image hello-world en utilisant le nouvel artefact comme contexte de génération :

```azurecli
az acr build --registry myregistry --file hello-world.dockerfile \
  oci://myregistry.azurecr.io/hello-world:1.0
```

## <a name="next-steps"></a>Étapes suivantes

* Découvrir la [bibliothèque ORAS](https://github.com/deislabs/oras/tree/master/docs), notamment la configuration d’un manifeste pour un artefact
* Visiter le dépôt [OCI Artifacts](https://github.com/opencontainers/artifacts) pour obtenir des informations de référence sur les nouveaux types d’artefacts



<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-acr-repository-show]: /cli/azure/acr/repository?#az-acr-repository-show
[az-acr-repository-delete]: /cli/azure/acr/repository#az-acr-repository-delete
