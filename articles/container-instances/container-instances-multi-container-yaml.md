---
title: Didacticiel - déployer un groupe de plusieurs conteneurs - YAML
description: Dans ce didacticiel, vous découvrez comment déployer un groupe multiconteneur dans Azure Container Instances à l’aide d’Azure CLI et d’un fichier YAML.
ms.topic: article
ms.date: 07/01/2020
ms.openlocfilehash: 6f9dda7735587dfee1dde86c85375efcf057daa7
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605159"
---
# <a name="tutorial-deploy-a-multi-container-group-using-a-yaml-file"></a>Tutoriel : Déployer un groupe multiconteneur à l’aide d’un fichier YAML

> [!div class="op_single_selector"]
> * [YAML](container-instances-multi-container-yaml.md)
> * [Resource Manager](container-instances-multi-container-group.md)
>

Azure Container Instances prend en charge le déploiement de plusieurs conteneurs sur un seul hôte à l’aide d’un [groupe de conteneurs](container-instances-container-groups.md). Un groupe de conteneurs est utile lors de la création d’une annexe d’application pour la journalisation, la surveillance ou toute autre configuration dans laquelle un service a besoin d’un deuxième processus associé.

Dans ce didacticiel, vous suivez des étapes pour exécuter une configuration d'annexe simple à deux conteneurs en déployant un [fichier YAML](container-instances-reference-yaml.md) à l'aide d'Azure CLI. Un fichier YAML fournit un format concis destiné à spécifier les paramètres d’instance. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Configurer un fichier YAML
> * Déployer le groupe de conteneurs
> * Afficher les journaux des conteneurs

> [!NOTE]
> Les groupes à plusieurs conteneurs sont actuellement restreints aux conteneurs Linux.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

## <a name="configure-a-yaml-file"></a>Configurer un fichier YAML

Pour déployer un groupe multiconteneur avec la commande [az container create][az-container-create] dans Azure CLI, vous devez spécifier la configuration du groupe de conteneurs dans un fichier YAML. Vous devez ensuite transmettre le fichier YAML à la commande en tant que paramètre.

Commencez par copier le YAML suivant dans un nouveau fichier nommé **deploy-aci.yaml**. Dans Azure Cloud Shell, vous pouvez utiliser Visual Studio Code pour créer le fichier dans votre répertoire de travail :

```
code deploy-aci.yaml
```

Ce fichier YAML définit un groupe de conteneurs nommé « myContainerGroup », qui comprend deux conteneurs, une adresse IP publique et deux ports exposés. Les conteneurs sont déployés à partir d’images publiques de Microsoft. Le premier conteneur du groupe exécute une application web accessible sur Internet. Le second conteneur, le side-car, envoie régulièrement des requêtes HTTP à l’application web en cours d’exécution dans le premier conteneur par le biais du réseau local du groupe de conteneurs.

```YAML
apiVersion: 2019-12-01
location: eastus
name: myContainerGroup
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
      ports:
      - port: 80
      - port: 8080
  - name: aci-tutorial-sidecar
    properties:
      image: mcr.microsoft.com/azuredocs/aci-tutorial-sidecar
      resources:
        requests:
          cpu: 1
          memoryInGb: 1.5
  osType: Linux
  ipAddress:
    type: Public
    ports:
    - protocol: tcp
      port: 80
    - protocol: tcp
      port: 8080
tags: {exampleTag: tutorial}
type: Microsoft.ContainerInstance/containerGroups
```

Pour utiliser un registre d’images conteneur privé, ajoutez la propriété `imageRegistryCredentials` au groupe de conteneurs en modifiant les valeurs pour votre environnement :

```YAML
  imageRegistryCredentials:
  - server: imageRegistryLoginServer
    username: imageRegistryUsername
    password: imageRegistryPassword
```

## <a name="deploy-the-container-group"></a>Déployer le groupe de conteneurs

Créez un groupe de ressources avec la commande [az group create][az-group-create] :

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Déployez le groupe de conteneurs avec la commande [az container create][az-container-create], en transmettant le fichier YAML en tant qu’argument :

```azurecli-interactive
az container create --resource-group myResourceGroup --file deploy-aci.yaml
```

Après quelques secondes, vous devriez recevoir une réponse initiale d’Azure.

## <a name="view-deployment-state"></a>Afficher l’état du déploiement

Pour afficher l’état du déploiement, utilisez la commande [az container show][az-container-show] suivante :

```azurecli-interactive
az container show --resource-group myResourceGroup --name myContainerGroup --output table
```

Pour voir l’application en cours d’exécution, accédez à son adresse IP dans votre navigateur. Par exemple, l’adresse IP est `52.168.26.124` dans cet exemple de sortie :

```console
Name              ResourceGroup    Status    Image                                                                                               IP:ports              Network    CPU/Memory       OsType    Location
----------------  ---------------  --------  --------------------------------------------------------------------------------------------------  --------------------  ---------  ---------------  --------  ----------
myContainerGroup  danlep0318r      Running   mcr.microsoft.com/azuredocs/aci-tutorial-sidecar,mcr.microsoft.com/azuredocs/aci-helloworld:latest  20.42.26.114:80,8080  Public     1.0 core/1.5 gb  Linux     eastus
```

## <a name="view-container-logs"></a>Afficher les journaux d’activité du conteneur

Consultez la sortie du journal d’un conteneur à l’aide de la commande [az container logs][az-container-logs]. L’argument `--container-name` spécifie le conteneur à partir duquel extraire les journaux d’activité. Dans cet exemple, le conteneur `aci-tutorial-app` est spécifié.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-app
```

Sortie :

```console
listening on port 80
::1 - - [02/Jul/2020:23:17:48 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [02/Jul/2020:23:17:51 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
::1 - - [02/Jul/2020:23:17:54 +0000] "HEAD / HTTP/1.1" 200 1663 "-" "curl/7.54.0"
```

Pour afficher les journaux du conteneur annexe, exécutez une commande similaire en spécifiant le conteneur `aci-tutorial-sidecar`.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name myContainerGroup --container-name aci-tutorial-sidecar
```

Sortie :

```console
Every 3s: curl -I http://localhost                          2020-07-02 20:36:41

  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0  1663    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
HTTP/1.1 200 OK
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Wed, 29 Nov 2017 06:40:40 GMT
ETag: W/"67f-16006818640"
Content-Type: text/html; charset=UTF-8
Content-Length: 1663
Date: Thu, 02 Jul 2020 20:36:41 GMT
Connection: keep-alive
```

Comme vous pouvez le voir, l’annexe envoie régulièrement une requête HTTP à l’application web principale via le réseau local du groupe pour s’assurer qu’il fonctionne. Cet exemple d’annexe peut être développé afin de déclencher une alerte après réception d’un code de réponse HTTP autre que `200 OK`.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez utilisé un fichier YAML pour déployer un groupe multiconteneur dans Azure Container Instances. Vous avez appris à :

> [!div class="checklist"]
> * Configurer un fichier YAML pour un groupe multiconteneur
> * Déployer le groupe de conteneurs
> * Afficher les journaux des conteneurs

Vous pouvez aussi spécifier un groupe multiconteneur à l’aide d’un [modèle Resource Manager](container-instances-multi-container-group.md). Un modèle Resource Manager peut être facilement adapté pour les scénarios lorsque vous avez besoin de déployer des ressources de service Azure supplémentaires avec le groupe de conteneurs.

<!-- LINKS - External -->

<!-- LINKS - Internal -->
[aci-tutorial]: ./container-instances-tutorial-prepare-app.md
[az-container-create]: /cli/azure/container#az-container-create
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[az-group-create]: /cli/azure/group#az-group-create
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
[template-reference]: /azure/templates/microsoft.containerinstance/containergroups
