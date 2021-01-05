---
title: Autorisations pour des référentiels dans Azure Container Registry
description: Créer un jeton avec des autorisations étendues à des référentiels spécifiques dans un registre Premium pour tirer (pull) ou envoyer (push) des images ou effectuer d’autres actions
ms.topic: article
ms.date: 05/27/2020
ms.openlocfilehash: b65b1bf69337cb172a17043490a5d13c7bd7afc2
ms.sourcegitcommit: 8a1ba1ebc76635b643b6634cc64e137f74a1e4da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/09/2020
ms.locfileid: "94381233"
---
# <a name="create-a-token-with-repository-scoped-permissions"></a>Créer un jeton avec des autorisations délimitées par le référentiel

Cet article explique comment créer des jetons et des mappages d’étendue pour gérer les autorisations délimitées par le référentiel dans votre registre de conteneurs. En créant des jetons, le propriétaire du registre peut fournir aux utilisateurs ou aux services un accès étendu et limité dans le temps aux référentiels pour tirer (pull) ou envoyer (push) des images ou effectuer d’autres actions. Un jeton fournit des autorisations plus précises que les autres [options d’authentification](container-registry-authentication.md) du registre, qui délimitent les autorisations à un registre entier. 

Les scénarios de création d’un jeton sont les suivants :

* Autoriser les appareils IoT dotés de jetons individuels pour tirer (pull) une image d’un référentiel
* Fournir à une organisation externe des autorisations sur un référentiel spécifique 
* Limiter l’accès au référentiel à différents groupes d’utilisateurs de votre organisation. Par exemple, fournissez un accès en écriture et en lecture aux développeurs qui génèrent des images ciblant des référentiels spécifiques, ainsi qu’un accès en lecture aux équipes qui déploient à partir de ces référentiels.

Cette fonctionnalité est disponible uniquement au niveau de service **Premium** de registre de conteneurs. Pour plus d’informations sur les niveaux de service et les limites de registre, consultez [Niveaux de service d’Azure Container Registry](container-registry-skus.md).

> [!IMPORTANT]
> Cette fonctionnalité est actuellement en préversion et certaines [limitations s’appliquent](#preview-limitations). Les préversions sont à votre disposition, à condition que vous acceptiez les [conditions d’utilisation supplémentaires][terms-of-use]. Certains aspects de cette fonctionnalité sont susceptibles d’être modifiés avant la mise à disposition générale.

## <a name="preview-limitations"></a>Limitations de la version préliminaire

* Vous ne pouvez pas actuellement attribuer des autorisations délimites par le référentiel à une identité Azure Active Directory telle qu’un principal de service ou une identité managée.
* Vous ne pouvez pas créer de mappage d’étendue dans un registre activé pour l’[accès en extraction anonyme](container-registry-faq.md#how-do-i-enable-anonymous-pull-access).

## <a name="concepts"></a>Concepts

Pour configurer des autorisations délimitées par le référentiel, vous créez un *jeton* avec un *mappage d’étendue* associé. 

* Un **jeton** accompagné d’un mot de passe généré permet à l’utilisateur de s’authentifier auprès du registre. Vous pouvez définir une date d’expiration pour un mot de passe de jeton ou désactiver un jeton à tout moment.  

  Après l’authentification à l’aide d’un jeton, l’utilisateur ou le service peut effectuer une ou plusieurs *actions* délimitées par un ou plusieurs référentiels.

  |Action  |Description  | Exemple |
  |---------|---------|--------|
  |`content/delete`    | Supprimer des données du référentiel  | Supprimer un référentiel ou un manifeste |
  |`content/read`     |  Lire des données du référentiel |  Tirer (pull) un artefact |
  |`content/write`     |  Écrire des données dans le référentiel     | Utiliser avec `content/read` pour envoyer (push) un artefact |
  |`metadata/read`    | Lire des métadonnées du référentiel   | Répertorier des balises ou des manifestes |
  |`metadata/write`     |  Écrire des métadonnées dans le référentiel  | Activer ou désactiver les opérations de lecture, d’écriture ou de suppression |

* Un **mappage d’étendue** regroupe les autorisations de référentiel que vous appliquez à un jeton et peut s’appliquer à nouveau à d’autres jetons. Chaque jeton est associé à un mappage d’étendue unique. 

   Avec un mappage d’étendue :

    * Configurer plusieurs jetons avec des autorisations identiques sur un ensemble de référentiels
    * Mettre à jour les autorisations de jeton lorsque vous ajoutez ou supprimez des actions de référentiel dans le mappage d’étendue, ou appliquer un autre mappage d’étendue 

  Azure Container Registry fournit également des mappages d’étendues définis par le système que vous pouvez appliquer lors de la création de jetons. Les autorisations des mappages d’étendues définis par le système s’appliquent à tous les référentiels de votre registre.

L’image suivante illustre la relation entre les jetons et les mappages d’étendue. 

![Jetons de registre et mappages d’étendue](media/container-registry-repository-scoped-permissions/token-scope-map-concepts.png)

## <a name="prerequisites"></a>Prérequis

* **Azure CLI**  : les commandes Azure CLI permettant de créer et gérer des jetons sont disponibles dans la version 2.0.76 ou ultérieure d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).
* **Docker**  : pour vous authentifier auprès du registre afin de tirer (pull) ou d’envoyer (push) des images, vous avez besoin d’une installation locale de Docker. Docker fournit des instructions d’installation pour les systèmes [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/) et [Linux](https://docs.docker.com/engine/installation/#supported-platforms).
* **Registre de conteneurs**  : créez un registre de conteneurs Premium dans votre abonnement Azure si vous n’en avez pas ou mettez à niveau un registre existant. Par exemple, utilisez le [portail Azure](container-registry-get-started-portal.md) ou [Azure CLI](container-registry-get-started-azure-cli.md). 

## <a name="create-token---cli"></a>Créer un jeton – CLI

### <a name="create-token-and-specify-repositories"></a>Créer un jeton et spécifier des référentiels

Créez un jeton à l’aide de la commande [az acr token create][az-acr-token-create]. Lorsque vous créez un jeton, vous pouvez spécifier un ou plusieurs référentiels et actions associées sur chaque référentiel. Il n’est pas nécessaire que les référentiels soient déjà dans le registre de conteneurs. Pour créer un jeton en spécifiant un mappage d’étendues existant, consultez la [section suivante](#create-token-and-specify-scope-map).

L’exemple suivant crée un jeton dans le registre *myregistry* avec les autorisations suivantes sur le référentiel `samples/hello-world` : `content/write` et `content/read`. Par défaut, la commande définit l’état du jeton sur `enabled` par défaut, mais vous pouvez mettre à jour l’état sur `disabled` à tout moment.

```azurecli
az acr token create --name MyToken --registry myregistry \
  --repository samples/hello-world \
  content/write content/read
```

La sortie affiche des détails sur le jeton. Par défaut, deux mots de passe sont générés. Il est recommandé d’enregistrer les mots de passe dans un endroit sûr pour les utiliser ultérieurement à des fins d’authentification. Les mots de passe ne peuvent pas être récupérés à nouveau, mais d’autres peuvent être générés.

```console
{
  "creationDate": "2020-01-18T00:15:34.066221+00:00",
  "credentials": {
    "certificates": [],
    "passwords": [
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password1",
        "value": "uH54BxxxxK7KOxxxxRbr26dAs8JXxxxx"
      },
      {
        "creationTime": "2020-01-18T00:15:52.837651+00:00",
        "expiry": null,
        "name": "password2",
        "value": "kPX6Or/xxxxLXpqowxxxxkA0idwLtmxxxx"
      }
    ],
    "username": "MyToken"
  },
  "id": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/tokens/MyToken",
  "name": "MyToken",
  "objectId": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "myresourcegroup",
  "scopeMapId": "/subscriptions/xxxxxxxx-adbd-4cb4-c864-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.ContainerRegistry/registries/myregistry/scopeMaps/MyToken-scope-map",
  "status": "enabled",
  "type": "Microsoft.ContainerRegistry/registries/tokens"
```

> [!NOTE]
> Si vous souhaitez régénérer les mots de passe des jetons et définir des périodes d’expiration des mots de passe, consultez [Régénérer les mots de passe de jeton](#regenerate-token-passwords) plus loin dans cet article.

La sortie contient des détails sur le mappage d’étendue que la commande a créé. Vous pouvez utiliser le mappage d’étendue, nommé ici `MyToken-scope-map`, pour appliquer les mêmes actions de référentiel à d’autres jetons. Ou mettez à jour le mappage d’étendue ultérieurement pour modifier les autorisations des jetons associés.

### <a name="create-token-and-specify-scope-map"></a>Créer un jeton et spécifier le mappage d’étendue

Une autre façon de créer un jeton consiste à spécifier un mappage d’étendue existant. Si vous n’avez pas encore de mappage d’étendue, commencez par en créer un en spécifiant des référentiels et des actions associées. Ensuite, spécifiez le mappage d’étendue lors de la création d’un jeton. 

Pour créer un mappage d’étendue, utilisez la commande [az acr scope-map create][az-acr-scope-map-create]. La commande suivante crée un mappage d’étendue avec les mêmes autorisations que celles du référentiel `samples/hello-world` utilisé précédemment. 

```azurecli
az acr scope-map create --name MyScopeMap --registry myregistry \
  --repository samples/hello-world \
  content/write content/read \
  --description "Sample scope map"
```

Exécutez la commande [az acr token create][az-acr-token-create] pour créer un jeton, en spécifiant le mappage d’étendue *MyScopeMap*. Comme dans l’exemple précédent, la commande définit l’état du jeton sur `enabled` par défaut.

```azurecli
az acr token create --name MyToken \
  --registry myregistry \
  --scope-map MyScopeMap
```

La sortie affiche des détails sur le jeton. Par défaut, deux mots de passe sont générés. Il est recommandé d’enregistrer les mots de passe dans un endroit sûr pour les utiliser ultérieurement à des fins d’authentification. Les mots de passe ne peuvent pas être récupérés à nouveau, mais d’autres peuvent être générés.

> [!NOTE]
> Si vous souhaitez régénérer les mots de passe des jetons et définir des périodes d’expiration des mots de passe, consultez [Régénérer les mots de passe de jeton](#regenerate-token-passwords) plus loin dans cet article.

## <a name="create-token---portal"></a>Créer un jeton – Portail

Vous pouvez utiliser le Portail Azure pour créer des jetons et des mappages d’étendue. Comme avec la commande CLI `az acr token create`, vous pouvez appliquer un mappage d’étendue existant ou en créer un lorsque vous créez un jeton en spécifiant un ou plusieurs référentiels et actions associées. Il n’est pas nécessaire que les référentiels soient déjà dans le registre de conteneurs. 

L’exemple suivant crée un jeton, puis crée un mappage d’étendue avec les autorisations suivantes sur le référentiel `samples/hello-world` : `content/write` et `content/read`.

1. Dans le portail, accédez à votre registre de conteneurs.
1. Sous **Autorisations du référentiel** , sélectionnez **Jetons (préversion) > +Ajouter**.

      :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-token-add.png" alt-text="Créer un jeton dans le portail":::
1. Entrez un nom de jeton.
1. Sous **Scope map** (Mappage d’étendue), sélectionnez **Create new** (Créer).
1. Configurer le mappage d’étendue :
    1. Entrez un nom et une description pour le mappage d’étendue. 
    1. Sous **Repositories** (Référentiels), entrez `samples/hello-world` et, sous **Permissions** (Autorisations), sélectionnez `content/read` et `content/write`. Sélectionnez ensuite **+Add** (+Ajouter).  

        :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-scope-map-add.png" alt-text="Créer un mappage d’étendue dans le portail":::

    1. Après avoir ajouté des référentiels et des autorisations, sélectionnez **Add** (Ajouter) pour ajouter le mappage d’étendue.
1. Acceptez le **Status** (État) par défaut **Enabled** (Activé) du jeton, puis sélectionnez **Create** (Créer).

Une fois le jeton validé et créé, les détails du jeton s’affichent dans l’écran **Tokens** (Jetons).

### <a name="add-token-password"></a>Ajouter un mot de passe de jeton

Pour utiliser un jeton créé dans le portail, vous devez générer un mot de passe. Vous pouvez générer un ou deux mots de passe et définir une date d’expiration pour chacun d’entre eux. 

1. Dans le portail, accédez à votre registre de conteneurs.
1. Sous **Autorisations du référentiel** , sélectionnez **Jetons (préversion)** et sélectionnez jeton.
1. Dans les détails du jeton, sélectionnez **password1** ou **password2** , puis sélectionnez l’icône de génération.
1. Dans l’écran du mot de passe, définissez éventuellement une date d’expiration pour le mot de passe, puis sélectionnez **Generate** (Générer). Il est recommandé de définir une date d’expiration.
1. Après avoir généré un mot de passe, copiez-le et enregistrez-le dans un emplacement sûr. Vous ne pouvez pas récupérer un mot de passe généré après avoir fermé l’écran, mais vous pouvez en générer un nouveau.

    :::image type="content" source="media/container-registry-repository-scoped-permissions/portal-token-password.png" alt-text="Créer un mot de passe de jeton dans le portail":::

## <a name="authenticate-with-token"></a>S’authentifier avec un jeton

Lorsqu’un utilisateur ou un service utilise un jeton pour s’authentifier auprès du registre cible, il fournit le nom du jeton comme nom d’utilisateur et l’un de ses mots de passe générés. 

La méthode d’authentification utilisée dépend des actions configurées associées au jeton.

|Action  |Comment s’authentifier  |
  |---------|---------|
  |`content/delete`    | `az acr repository delete` dans Azure CLI<br/><br/>Exemple : `az acr repository delete --name myregistry --repository myrepo --username MyToken --password xxxxxxxxxx`|
  |`content/read`     |  `docker login`<br/><br/>`az acr login` dans Azure CLI<br/><br/>Exemple : `az acr login --name myregistry --username MyToken --password xxxxxxxxxx`  |
  |`content/write`     |  `docker login`<br/><br/>`az acr login` dans Azure CLI     |
  |`metadata/read`    | `az acr repository show`<br/><br/>`az acr repository show-tags`<br/><br/>`az acr repository show-manifests` dans Azure CLI   |
  |`metadata/write`     |  `az acr repository untag`<br/><br/>`az acr repository update` dans Azure CLI |

## <a name="examples-use-token"></a>Exemples : Utiliser un jeton

Les exemples suivants utilisent le jeton créé plus haut dans cet article pour effectuer des opérations courantes sur un référentiel : envoyer (push) et tirer (pull) des images, supprimer des images et répertorier des balises de référentiel. Le jeton a été configuré initialement avec des autorisations d’envoi (push) [actions `content/write` et `content/read`] sur le référentiel `samples/hello-world`.

### <a name="pull-and-tag-test-images"></a>Tirer (pull) et baliser des images de test

Pour les exemples suivants, tirez (pull) les images `hello-world` et `alpine` de Docker Hub, puis balisez-les pour votre registre et votre référentiel.

```bash
docker pull hello-world
docker pull alpine
docker tag hello-world myregistry.azurecr.io/samples/hello-world:v1
docker tag alpine myregistry.azurecr.io/samples/alpine:v1
```

### <a name="authenticate-using-token"></a>Authentifier à l'aide du jeton

Exécutez `docker login` ou `az acr login` pour vous authentifier auprès du registre afin d’envoyer (push) ou de tirer (pull) des images. Entrez le nom du jeton en tant que nom d’utilisateur et fournissez l’un de ses mots de passe. Le jeton doit avoir l’état `Enabled`.

L’exemple suivant est mis en forme pour l’interpréteur de commandes Bash et fournit les valeurs à l’aide de variables d’environnement.

```bash
TOKEN_NAME=MyToken
TOKEN_PWD=<token password>

echo $TOKEN_PWD | docker login --username $TOKEN_NAME --password-stdin myregistry.azurecr.io
```

La sortie doit afficher l’authentification réussie :

```console
Login Succeeded
```

### <a name="push-images-to-registry"></a>Envoyer des images au registre

Une fois la connexion établie, tentez d’envoyer (push) les images balisées dans le registre. Étant donné que le jeton dispose d’autorisations pour envoyer (push) des images vers le référentiel `samples/hello-world`, l’envoi (push) suivant fonctionne :

```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

Le jeton ne dispose pas des autorisations d’accès au `samples/alpine`. Par conséquent, la tentative d’envoi (push) suivant échoue avec une erreur semblable à `requested access to the resource is denied` :

```bash
docker push myregistry.azurecr.io/samples/alpine:v1
```

### <a name="update-token-permissions"></a>Mettre à jour des autorisations de jeton

Pour mettre à jour les autorisations d’un jeton, mettez à jour les autorisations dans le mappage d’étendue associé. Le mappage d’étendue mis à jour est appliqué immédiatement à tous les jetons associés. 

Par exemple, mettez à jour `MyToken-scope-map` avec les actions `content/write` et `content/read` sur le référentiel `samples/alpine`, et supprimez l’action `content/write` sur le référentiel `samples/hello-world`.  

Pour utiliser Azure CLI, exécutez [az acr scope-map update][az-acr-scope-map-update] pour mettre à jour le mappage d’étendue :

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add samples/alpine content/write content/read \
  --remove samples/hello-world content/write 
```

Dans le portail Azure :

1. Accédez à votre registre de conteneurs.
1. Sous **Autorisations de référentiel** , sélectionnez **Mappages d’étendues (préversion)** s, puis sélectionnez le mappage d’étendue à mettre à jour.
1. Sous **Repositories** (Référentiels), entrez `samples/alpine` et, sous **Permissions** (Autorisations), sélectionnez `content/read` et `content/write`. Sélectionnez ensuite **+Add** (+Ajouter).
1. Sous **Repositories** (Référentiels), sélectionnez `samples/hello-world` et, sous **Permissions** (Autorisations), désélectionnez `content/write`. Ensuite, sélectionnez **Enregistrer**.

Après la mise à jour du mappage d’étendue, l’envoi (push) suivant est réussi :

```bash
docker push myregistry.azurecr.io/samples/alpine:v1
```

Étant donné que le mappage d’étendue dispose uniquement de l’autorisation `content/read` sur le référentiel `samples/hello-world`, une tentative d’envoi (push) vers le référentiel `samples/hello-world` échoue désormais :
 
```bash
docker push myregistry.azurecr.io/samples/hello-world:v1
```

L’extraction d’images des deux pensions est réussie, car le mappage d’étendue fournit des autorisations `content/read` sur les deux référentiels :

```bash
docker pull myregistry.azurecr.io/samples/alpine:v1
docker pull myregistry.azurecr.io/samples/hello-world:v1
```
### <a name="delete-images"></a>Supprimer des images

Mettez à jour le mappage d’étendue en ajoutant l’action `content/delete` au référentiel `alpine`. Cette action permet de supprimer des images dans le référentiel ou de supprimer l’intégralité du référentiel.

Par souci de concision, nous affichons uniquement la commande [az acr scope-map update][az-acr-scope-map-update] pour mettre à jour le mappage d’étendue :

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add samples/alpine content/delete
``` 

Pour mettre à jour le mappage d’étendues à l’aide du portail, consultez la [section précédente](#update-token-permissions).

Utilisez la commande [az acr repository delete][az-acr-repository-delete] suivante pour supprimer le référentiel `samples/alpine`. Pour supprimer des images ou des référentiels, transmettez le nom et le mot de passe du jeton à la commande. L’exemple suivant utilise les variables d’environnement créées plus haut dans l’article :

```azurecli
az acr repository delete \
  --name myregistry --repository samples/alpine \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

### <a name="show-repo-tags"></a>Afficher les balises de référentiel 

Mettez à jour le mappage d’étendue en ajoutant l’action `metadata/read` au référentiel `hello-world`. Cette action permet de lire les données de manifeste et de balise dans le référentiel.

Par souci de concision, nous affichons uniquement la commande [az acr scope-map update][az-acr-scope-map-update] pour mettre à jour le mappage d’étendue :

```azurecli
az acr scope-map update \
  --name MyScopeMap \
  --registry myregistry \
  --add samples/hello-world metadata/read 
```  

Pour mettre à jour le mappage d’étendues à l’aide du portail, consultez la [section précédente](#update-token-permissions).

Pour lire les métadonnées dans le référentiel `samples/hello-world`, exécutez la commande [az acr repository show-manifests][az-acr-repository-show-manifests] ou [az acr repository show-tags][az-acr-repository-show-tags]. 

Pour lire des métadonnées, transmettez le nom et le mot de passe du jeton à l’une ou l’autre des commandes. L’exemple suivant utilise les variables d’environnement créées plus haut dans l’article :

```azurecli
az acr repository show-tags \
  --name myregistry --repository samples/hello-world \
  --username $TOKEN_NAME --password $TOKEN_PWD
```

Exemple de sortie :

```console
[
  "v1"
]
```

## <a name="manage-tokens-and-scope-maps"></a>Gérer les jetons et les mappages d’étendue

### <a name="list-scope-maps"></a>Répertorier les mappages d’étendue

Utilisez la commande [az acr scope-map list][az-acr-scope-map-list] ou l’écran **Scope maps (Preview)** [Mappages d’étendue (préversion)] dans le portail pour répertorier tous les mappages d’étendue configurés dans un registre. Par exemple :

```azurecli
az acr scope-map list \
  --registry myregistry --output table
```

La sortie se compose des trois mappages d’étendues définis par le système et d’autres mappages d’étendues générés par vous. Les jetons peuvent être configurés avec l’un de ces mappages d’étendues.

```
NAME                 TYPE           CREATION DATE         DESCRIPTION
-------------------  -------------  --------------------  ------------------------------------------------------------
_repositories_admin  SystemDefined  2020-01-20T09:44:24Z  Can perform all read, write and delete operations on the ...
_repositories_pull   SystemDefined  2020-01-20T09:44:24Z  Can pull any repository of the registry
_repositories_push   SystemDefined  2020-01-20T09:44:24Z  Can push to any repository of the registry
MyScopeMap           UserDefined    2019-11-15T21:17:34Z  Sample scope map
```

### <a name="show-token-details"></a>Afficher les détails du jeton

Pour afficher les détails d’un jeton, tels que son état et les dates d’expiration de ses mots de passe, exécutez la commande [az acr token show][az-acr-token-show] ou sélectionnez le jeton dans l’écran **Tokens (Preview)** [Jetons (préversion)] dans le portail. Par exemple :

```azurecli
az acr scope-map show \
  --name MyScopeMap --registry myregistry
```

Utilisez la commande [az acr token list][az-acr-token-list] ou l’écran **Tokens (Preview)** [Jetons (préversion)] dans le portail pour répertorier tous les jetons configurés dans un registre. Par exemple :

```azurecli
az acr token list --registry myregistry --output table
```

### <a name="regenerate-token-passwords"></a>Régénérer les mots de passe des jetons

Si vous n’avez pas généré de mot de passe de jeton ou si vous souhaitez générer de nouveaux mots de passe, exécutez la commande [az acr token credential generate][az-acr-token-credential-generate]. 

L’exemple suivant génère une nouvelle valeur password1 pour le jeton *MyToken* , avec une période d’expiration de 30 jours. Cela stocke le mot de passe dans la variable d’environnement `TOKEN_PWD`. Cet exemple est mis en forme pour l’interpréteur de commandes Bash.

```azurecli
TOKEN_PWD=$(az acr token credential generate \
  --name MyToken --registry myregistry --days 30 \
  --password1 --query 'passwords[0].value' --output tsv)
```

Pour utiliser le Portail Azure afin de générer un mot de passe de jeton, consultez les étapes décrites dans [Créer un jeton – Portail](#create-token---portal) plus haut dans cet article.

### <a name="update-token-with-new-scope-map"></a>Mettre à jour le jeton avec le nouveau mappage d’étendue

Si vous souhaitez mettre à jour un jeton avec un mappage d’étendue différent, exécutez la commande [az acr token update][az-acr-token-update] et spécifiez le nouveau mappage d’étendue. Par exemple :

```azurecli
az acr token update --name MyToken --registry myregistry \
  --scope-map MyNewScopeMap
```

Sur le portail, dans l’écran **Tokens (preview)** [Jetons (préversion)], sélectionnez le jeton, puis sous **Scope map** (Mappage d’étendue), sélectionnez un autre mappage d’étendue.

> [!TIP]
> Après la mise à jour d’un jeton avec un nouveau mappage d’étendue, vous souhaiterez peut-être générer de nouveaux mots de passe de jeton. Utilisez la commande [az acr token credential generate][az-acr-token-credential-generate] ou régénérez un mot de passe de jeton dans le Portail Azure.

## <a name="disable-or-delete-token"></a>Désactiver ou supprimer un jeton

Vous devrez peut-être désactiver temporairement l’utilisation des informations d’identification du jeton pour un utilisateur ou un service. 

À l’aide d’Azure CLI, exécutez la commande [az acr token update][az-acr-token-update] pour définir le paramètre `status` sur `disabled` :

```azurecli
az acr token update --name MyToken --registry myregistry \
  --status disabled
```

Dans le portail, sélectionnez le jeton dans l’écran **Tokens (Preview)** [Jetons (préversion)], puis sélectionnez **Disabled** (Désactivé) sous **Status** (État).

Pour supprimer un jeton afin d’invalider l’accès de manière permanente pour toute personne utilisant ses informations d’identification, exécutez la commande [az acr token delete][az-acr-token-delete]. 

```azurecli
az acr token delete --name MyToken --registry myregistry
```

Dans le portail, sélectionnez le jeton dans l’écran **Tokens (Preview)** [Jetons (préversion)], puis sélectionnez **Discard** (Abandonner).

## <a name="next-steps"></a>Étapes suivantes

* Pour gérer les mappages d’étendue et les jetons, utilisez des commandes supplémentaires dans les groupes de commande [az acr scope-map][az-acr-scope-map] et [az acr token][az-acr-token].
* Consultez la [page de présentation de l’authentification](container-registry-authentication.md) pour obtenir d’autres options d’authentification avec un registre de conteneurs Azure, notamment à l’aide d’une identité Azure Active Directory, d’un principal de service ou d’un compte administrateur.


<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-repository]: /cli/azure/acr/repository/
[az-acr-repository-show-tags]: /cli/azure/acr/repository/#az-acr-repository-show-tags
[az-acr-repository-show-manifests]: /cli/azure/acr/repository/#az-acr-repository-show-manifests
[az-acr-repository-delete]: /cli/azure/acr/repository/#az-acr-repository-delete
[az-acr-scope-map]: /cli/azure/acr/scope-map/
[az-acr-scope-map-create]: /cli/azure/acr/scope-map/#az-acr-scope-map-create
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az-acr-scope-map-show
[az-acr-scope-map-show]: /cli/azure/acr/scope-map/#az-acr-scope-map-list
[az-acr-scope-map-update]: /cli/azure/acr/scope-map/#az-acr-scope-map-update
[az-acr-scope-map-list]: /cli/azure/acr/scope-map/#az-acr-scope-map-list
[az-acr-token]: /cli/azure/acr/token/
[az-acr-token-show]: /cli/azure/acr/token/#az-acr-token-show
[az-acr-token-list]: /cli/azure/acr/token/#az-acr-token-list
[az-acr-token-delete]: /cli/azure/acr/token/#az-acr-token-delete
[az-acr-token-create]: /cli/azure/acr/token/#az-acr-token-create
[az-acr-token-update]: /cli/azure/acr/token/#az-acr-token-update
[az-acr-token-credential-generate]: /cli/azure/acr/token/credential/#az-acr-token-credential-generate
