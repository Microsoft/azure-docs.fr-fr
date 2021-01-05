---
title: Authentifier avec l’identité gérée
description: Donnez accès aux images de votre registre de conteneurs privé à l’aide d’une identité managée Azure affectée par l’utilisateur ou par le système.
ms.topic: article
ms.date: 01/16/2019
ms.openlocfilehash: 9a144f0e865cfc9bf857752eed65dbe5cda88bd9
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91253460"
---
# <a name="use-an-azure-managed-identity-to-authenticate-to-an-azure-container-registry"></a>Utiliser une identité managée Azure pour s’authentifier auprès d’un registre de conteneurs Azure 

Utilisez une [identité managée pour ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) pour vous authentifier auprès d’un registre de conteneurs Azure à partir d’une autre ressource Azure, sans avoir à fournir ou à gérer les informations d’identification du registre. Par exemple, configurez une identité managée affectée par l’utilisateur ou par le système sur une machine virtuelle Linux pour accéder aux images conteneur à partir de votre registre de conteneurs, comme avec un registre public.

Dans cet article, vous découvrirez les identités managées et apprendrez à :

> [!div class="checklist"]
> * activer une identité affectée par l’utilisateur ou par le système dans une machine virtuelle Azure ;
> * autoriser cette identité à accéder à un registre de conteneurs Azure ;
> * utiliser l’identité managée pour accéder au registre et tirer (pull) une image conteneur. 

Pour créer les ressources Azure, il est nécessaire dans le cadre de cet article de disposer de la version 2.0.55 ou d’une version ultérieure d’Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli].

Docker doit être installé localement pour que vous puissiez configurer un registre de conteneurs et y envoyer une image conteneur. Docker fournit des packages qui le configurent facilement sur n’importe quel système [macOS][docker-mac], [Windows][docker-windows] ou [Linux][docker-linux].

## <a name="why-use-a-managed-identity"></a>Pourquoi utiliser une identité managée ?

Une identité managée pour ressources Azure fournit aux services Azure une identité automatiquement managée dans Azure Active Directory (Azure AD). [Certaines ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md), y compris les machines virtuelles, sont configurables avec une identité managée. Ensuite, utilisez l’identité pour accéder aux autres ressources Azure, sans indiquer les informations d’identification dans le code ou les scripts.

Les identités managées sont de deux types :

* Les *identités affectées par l’utilisateur*, que vous pouvez affecter à plusieurs ressources et conserver aussi longtemps que vous le souhaitez. Les identités attribuées par l’utilisateur sont actuellement en préversion.

* Une *identité affectée par le système*, propre à une ressource spécifique (comme une machine virtuelle), qui possède la même durée de vie que cette ressource.

Après avoir configuré une ressource Azure avec une identité managée, accordez-lui l’accès de votre choix à une autre ressource, tout comme un principal de sécurité. Par exemple, donnez-lui un rôle disposant des autorisations pull (tirage), push/pull (tirage/envoi), etc. sur un registre privé dans Azure. (Pour connaître la liste complète des rôles de registre, voir [Rôles et autorisations Azure Container Registry](container-registry-roles.md).) Vous pouvez accorder à une identité l’accès à une ou plusieurs ressources.

Ensuite, utilisez l’identité pour vous authentifier auprès d’un [service prenant en charge l’authentification Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication), sans aucune information d’identification dans votre code. Pour pouvoir accéder à un registre de conteneurs Azure à partir d’une machine virtuelle à l’aide d’une identité managée, vous vous authentifiez avec Azure Resource Manager. Il existe plusieurs méthodes d’authentification en fonction du scénario :

* [acquérir un jeton d’accès Azure AD](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) programmatiquement à l’aide d’appels HTTP ou REST ;

* utiliser les [kits SDK Azure](../active-directory/managed-identities-azure-resources/how-to-use-vm-sdk.md) ;

* [se connecter à Azure CLI ou à PowerShell](../active-directory/managed-identities-azure-resources/how-to-use-vm-sign-in.md) avec l’identité. 

## <a name="create-a-container-registry"></a>Créer un registre de conteneur

Si vous ne disposez pas déjà d’un registre de conteneurs Azure, créez-en un et envoyez-lui un exemple d’image conteneur. Pour connaître les étapes à suivre, voir [Démarrage rapide : Créer un registre de conteneurs privé avec Azure CLI](container-registry-get-started-azure-cli.md).

Cet article suppose que l’image conteneur `aci-helloworld:v1` est stockée dans le registre. Les exemples utilisent le nom de registre *myContainerRegistry*. Remplacez-le par le nom de votre registre et de vos images dans les étapes suivantes.

## <a name="create-a-docker-enabled-vm"></a>Créer une machine virtuelle compatible Docker

Créez une machine virtuelle Ubuntu compatible Docker. Vous devez également installer [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) sur la machine virtuelle. Si vous disposez déjà d’une machine virtuelle Azure, ignorez l’étape de création correspondante.

Déployez une machine virtuelle Azure Ubuntu par défaut avec [az vm create][az-vm-create]. L’exemple suivant montre la création d’une machine virtuelle nommée *myDockerVM* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myDockerVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Il faut quelques minutes pour que la machine virtuelle soit créée. Après l’exécution de la commande, notez la valeur de `publicIpAddress` qui s’affiche dans l’interface Azure CLI. Utilisez cette adresse pour établir des connexions SSH avec la machine virtuelle.

### <a name="install-docker-on-the-vm"></a>Installer Docker sur la machine virtuelle

Établissez une connexion SSH avec la machine virtuelle dès qu’elle est en cours d’exécution. Remplacez *publicIpAddress* par l’adresse IP publique de votre machine virtuelle.

```bash
ssh azureuser@publicIpAddress
```

Exécutez la commande suivante pour installer Docker sur la machine virtuelle :

```bash
sudo apt update
sudo apt install docker.io -y
```

Après l’installation, exécutez la commande suivante pour vérifier que Docker s’exécute correctement sur la machine virtuelle :

```bash
sudo docker run -it hello-world
```

Sortie :

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

### <a name="install-the-azure-cli"></a>Installer l’interface de ligne de commande Microsoft Azure

Suivez les étapes de la section [Installer Azure CLI avec apt](/cli/azure/install-azure-cli-apt?view=azure-cli-latest) pour installer Azure CLI sur votre machine virtuelle Ubuntu. Dans le cadre de cet article, veillez à installer la version 2.0.55 ou une version ultérieure.

Quittez la session SSH.

## <a name="example-1-access-with-a-user-assigned-identity"></a>Exemple 1 : Accès avec une identité affectée par l’utilisateur

### <a name="create-an-identity"></a>Créer une identité

Créez une identité dans votre abonnement avec la commande [az identity create](/cli/azure/identity?view=azure-cli-latest#az-identity-create). Vous pouvez utiliser le groupe de ressources qui vous a servi à créer le registre de conteneurs ou la machine virtuelle, ou bien un autre.

```azurecli-interactive
az identity create --resource-group myResourceGroup --name myACRId
```

Pour configurer l’identité aux étapes suivantes, utilisez la commande [az identity show][az-identity-show] afin de stocker l’ID de ressource et l’ID de principal de service de l’identité dans des variables.

```azurecli
# Get resource ID of the user-assigned identity
userID=$(az identity show --resource-group myResourceGroup --name myACRId --query id --output tsv)

# Get service principal ID of the user-assigned identity
spID=$(az identity show --resource-group myResourceGroup --name myACRId --query principalId --output tsv)
```

Vous aurez besoin de l’ID de l’identité quand vous vous connecterez à l’interface CLI à partir de votre machine virtuelle. Affichez sa valeur :

```bash
echo $userID
```

L’ID est au format :

```
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myACRId
```

### <a name="configure-the-vm-with-the-identity"></a>Configurer la machine virtuelle avec l’identité

La commande [az vm identity assign][az-vm-identity-assign] configure la machine virtuelle Docker avec l’identité affectée par l’utilisateur :

```azurecli
az vm identity assign --resource-group myResourceGroup --name myDockerVM --identities $userID
```

### <a name="grant-identity-access-to-the-container-registry"></a>Autoriser l’identité à accéder au registre de conteneurs

Configurez maintenant l’identité de façon à ce qu’elle ait accès à votre registre de conteneurs. Tout d’abord, utilisez la commande [az acr show][az-acr-show] pour obtenir l’ID de ressource du registre :

```azurecli
resourceID=$(az acr show --resource-group myResourceGroup --name myContainerRegistry --query id --output tsv)
```

Utilisez la commande [az role assignment create][az-role-assignment-create] pour affecter le rôle AcrPull au registre. Ce rôle lui accorde des [autorisations pull (tirage)](container-registry-roles.md). Pour lui donner à la fois l’autorisation pull et l’autorisation push (envoi), affectez-lui le rôle ACRPush.

```azurecli
az role assignment create --assignee $spID --scope $resourceID --role acrpull
```

### <a name="use-the-identity-to-access-the-registry"></a>Utiliser l’identité pour accéder au registre

Établissez une connexion SSH avec la machine virtuelle Docker configurée avec l’identité. Exécutez les commandes Azure CLI suivantes, avec l’interface Azure CLI installée sur la machine virtuelle.

Tout d’abord, authentifiez-vous auprès d’Azure CLI avec la commande [az login][az-login] et l’identité configurée sur la machine virtuelle. Remplacez `<userID>` par l’ID de l’identité que vous avez récupéré à l’étape précédente. 

```azurecli
az login --identity --username <userID>
```

Ensuite, authentifiez-vous auprès du registre avec la commande [az acr login][az-acr-login]. L’interface CLI utilise alors le jeton Active Directory créé à l’exécution de `az login` pour authentifier en toute transparence votre session auprès du registre de conteneurs. (Selon la configuration de votre machine virtuelle, vous devrez peut-être exécuter cette commande et les commandes docker avec `sudo`.)

```azurecli
az acr login --name myContainerRegistry
```

Un message `Login succeeded` devrait apparaître. Vous pouvez maintenant exécuter des commandes `docker` sans fournir d’informations d’identification. Par exemple, exécutez [docker pull][docker-pull] pour tirer (pull) l’image `aci-helloworld:v1`, en spécifiant le nom du serveur de connexion de votre registre. Il s’agit du nom du registre de conteneurs (tout en minuscules) suivi de `.azurecr.io` (par exemple, `mycontainerregistry.azurecr.io`).

```
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```

## <a name="example-2-access-with-a-system-assigned-identity"></a>Exemple 2 : Accès avec une identité affectée par le système

### <a name="configure-the-vm-with-a-system-managed-identity"></a>Configurer la machine virtuelle avec une identité managée par le système

La commande [az vm identity assign][az-vm-identity-assign] configure la machine virtuelle Docker avec une identité affectée par le système :

```azurecli
az vm identity assign --resource-group myResourceGroup --name myDockerVM 
```

Utilisez la commande [az vm show][az-vm-show] pour donner à une variable la valeur `principalId` (ID du principal de service) de l’identité de la machine virtuelle, qui sera utilisée lors des étapes suivantes.

```azurecli-interactive
spID=$(az vm show --resource-group myResourceGroup --name myDockerVM --query identity.principalId --out tsv)
```

### <a name="grant-identity-access-to-the-container-registry"></a>Autoriser l’identité à accéder au registre de conteneurs

Configurez maintenant l’identité de façon à ce qu’elle ait accès à votre registre de conteneurs. Tout d’abord, utilisez la commande [az acr show][az-acr-show] pour obtenir l’ID de ressource du registre :

```azurecli
resourceID=$(az acr show --resource-group myResourceGroup --name myContainerRegistry --query id --output tsv)
```

Utilisez la commande [az role assignment create][az-role-assignment-create] pour affecter le rôle AcrPull à l’identité. Ce rôle lui accorde des [autorisations pull (tirage)](container-registry-roles.md). Pour lui donner à la fois l’autorisation pull et l’autorisation push (envoi), affectez-lui le rôle ACRPush.

```azurecli
az role assignment create --assignee $spID --scope $resourceID --role acrpull
```

### <a name="use-the-identity-to-access-the-registry"></a>Utiliser l’identité pour accéder au registre

Établissez une connexion SSH avec la machine virtuelle Docker configurée avec l’identité. Exécutez les commandes Azure CLI suivantes, avec l’interface Azure CLI installée sur la machine virtuelle.

Tout d’abord, authentifiez-vous auprès d’Azure CLI avec la commande [az login][az-login] et l’identité affectée par le système sur la machine virtuelle.

```azurecli
az login --identity
```

Ensuite, authentifiez-vous auprès du registre avec la commande [az acr login][az-acr-login]. L’interface CLI utilise alors le jeton Active Directory créé à l’exécution de `az login` pour authentifier en toute transparence votre session auprès du registre de conteneurs. (Selon la configuration de votre machine virtuelle, vous devrez peut-être exécuter cette commande et les commandes docker avec `sudo`.)

```azurecli
az acr login --name myContainerRegistry
```

Un message `Login succeeded` devrait apparaître. Vous pouvez maintenant exécuter des commandes `docker` sans fournir d’informations d’identification. Par exemple, exécutez [docker pull][docker-pull] pour tirer (pull) l’image `aci-helloworld:v1`, en spécifiant le nom du serveur de connexion de votre registre. Il s’agit du nom du registre de conteneurs (tout en minuscules) suivi de `.azurecr.io` (par exemple, `mycontainerregistry.azurecr.io`).

```
docker pull mycontainerregistry.azurecr.io/aci-helloworld:v1
```
> [!NOTE]
> Les identités de service géré affectées par le système peuvent être utilisées pour interagir avec des instances Azure Container Registry et App Service peut utiliser des identités de service géré affectées par le système. Toutefois, vous ne pouvez pas les combiner, car App Service ne peut pas utiliser MSI pour communiquer avec Azure Container Registry. La seule méthode consiste à activer l’administrateur sur Azure Container Registry et à utiliser le nom d’utilisateur/mot de passe de l’administrateur.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert les identités managées avec Azure Container Registry et vous avez appris à :

> [!div class="checklist"]
> * activer une identité affectée par l’utilisateur ou par le système dans une machine virtuelle Azure ;
> * autoriser cette identité à accéder à un registre de conteneurs Azure ;
> * utiliser l’identité managée pour accéder au registre et tirer (pull) une image conteneur.

* En savoir plus sur les [identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/index.yml).


<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-pull]: https://docs.docker.com/engine/reference/commandline/pull/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[az-login]: /cli/azure/reference-index#az-login
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-vm-show]: /cli/azure/vm#az-vm-show
[az-vm-identity-assign]: /cli/azure/vm/identity#az-vm-identity-assign
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-identity-show]: /cli/azure/identity#az-identity-show
[azure-cli]: /cli/azure/install-azure-cli
