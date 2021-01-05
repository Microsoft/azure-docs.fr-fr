---
title: Monter le volume secret pour le groupe de conteneurs
description: Découvrez comment monter un volume secret pour stocker des informations sensibles et y accéder à partir de vos instances de conteneur
ms.topic: article
ms.date: 07/02/2020
ms.openlocfilehash: ea82ba5994feaf102d4622eada284df431e004d0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86169559"
---
# <a name="mount-a-secret-volume-in-azure-container-instances"></a>Monter un volume secret dans Azure Container Instances

Vous pouvez utiliser un volume *secret* pour fournir des informations sensibles aux conteneurs d’un groupe de conteneurs. Le volume *secret* stocke vos secrets spécifiés dans les fichiers du volume, auxquels les conteneurs de votre groupe de conteneurs peuvent accéder. En stockant les secrets d’un volume *secret*, vous pouvez éviter de placer des données sensibles telles que des clés SSH ou des informations d’identification sur une base de données dans votre code d’application.

* Une fois déployé avec des secrets dans un groupe de conteneurs, un volume secret est en *lecture seule*.
* Tous les volumes secrets sont sauvegardés par [tmpfs][tmpfs], un système de fichiers reposant sur la RAM ; leur contenu n’est jamais écrit sur un stockage non volatile.

> [!NOTE]
> Les volumes *secrets* sont actuellement restreints aux conteneurs Linux. Découvrez comment transmettre des variables d’environnement sécurisées pour les conteneurs Windows et Linux dans [Définir des variables d’environnement](container-instances-environment-variables.md). Nous travaillons actuellement à proposer toutes ces fonctionnalités dans des conteneurs Windows. En attendant, nous vous invitons à découvrir les différences actuelles de la plateforme dans la [Vue d’ensemble](container-instances-overview.md#linux-and-windows-containers).

## <a name="mount-secret-volume---azure-cli"></a>Monter un volume secret : Azure CLI

Pour déployer un conteneur avec un ou plusieurs secrets à l’aide de l’interface Azure CLI, incluez les paramètres `--secrets` et `--secrets-mount-path` dans la commande [az container create][az-container-create]. Cet exemple monte un volume *secret* constitué de deux fichiers contenant des secrets, « mysecret1 » et « mysecret2 », sur `/mnt/secrets` :

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name secret-volume-demo \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --secrets mysecret1="My first secret FOO" mysecret2="My second secret BAR" \
    --secrets-mount-path /mnt/secrets
```

La sortie de la commande [az container exec][az-container-exec] suivante indique l’ouverture d’un interpréteur de commandes dans le conteneur en cours d’exécution, affichant les fichiers dans le volume secret, puis affiche leur contenu :

```azurecli
az container exec \
  --resource-group myResourceGroup \
  --name secret-volume-demo --exec-command "/bin/sh"
```

```output
/usr/src/app # ls /mnt/secrets
mysecret1
mysecret2
/usr/src/app # cat /mnt/secrets/mysecret1
My first secret FOO
/usr/src/app # cat /mnt/secrets/mysecret2
My second secret BAR
/usr/src/app # exit
Bye.
```

## <a name="mount-secret-volume---yaml"></a>Monter un volume secret : YAML

Vous pouvez également déployer des groupes de conteneurs avec Azure CLI et un [modèle YAML](container-instances-multi-container-yaml.md). Le déploiement avec un modèle YAML est la méthode privilégiée lorsqu’il s’agit de déployer des groupes de conteneurs, composés de conteneurs multiples.

Lorsque vous déployez avec un modèle YAML, les valeurs de secret doivent être **codées en Base64** dans le modèle. Toutefois, les valeurs de secret apparaissent en texte brut dans les fichiers du conteneur.

Le modèle YAML suivant définit un groupe de conteneurs avec un seul conteneur montant un volume *secret* sur `/mnt/secrets`. Le volume secret a deux fichiers contenant des secrets, « mysecret1 » et « mysecret2 ».

```yaml
apiVersion: '2019-12-01'
location: eastus
name: secret-volume-demo
properties:
  containers:
  - name: aci-tutorial-app
    properties:
      environmentVariables: []
      image: mcr.microsoft.com/azuredocs/aci-helloworld:latest
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - mountPath: /mnt/secrets
        name: secretvolume1
  osType: Linux
  restartPolicy: Always
  volumes:
  - name: secretvolume1
    secret:
      mysecret1: TXkgZmlyc3Qgc2VjcmV0IEZPTwo=
      mysecret2: TXkgc2Vjb25kIHNlY3JldCBCQVIK
tags: {}
type: Microsoft.ContainerInstance/containerGroups
```

Pour effectuer un déploiement avec le modèle YAML, enregistrez le YAML précédent dans un fichier nommé `deploy-aci.yaml`, puis exécutez la commande [az container create][az-container-create] avec le paramètre `--file` :

```azurecli-interactive
# Deploy with YAML template
az container create \
  --resource-group myResourceGroup \
  --file deploy-aci.yaml
```

## <a name="mount-secret-volume---resource-manager"></a>Monter un volume secret : Gestionnaire des ressources

En plus du déploiement de l’interface CLI et de YAML, vous pouvez déployer un groupe de conteneurs à l’aide d’un [modèle Resource Manager](/azure/templates/microsoft.containerinstance/containergroups) Azure.

Tout d’abord, remplissez le tableau `volumes` dans la section `properties` du groupe de conteneurs du modèle. Lorsque vous déployez avec un modèle Resource Manager, les valeurs du secret doivent être **Codées en base64** dans le modèle. Toutefois, les valeurs de secret apparaissent en texte brut dans les fichiers du conteneur.

Ensuite, pour chaque conteneur du groupe de conteneurs dans lequel vous souhaitez monter le volume *secret*, remplissez le tableau `volumeMounts` dans la section `properties` de la définition de conteneur.

Le modèle Resource Manager suivant définit un groupe de conteneurs avec un conteneur montant un volume *secret* sur `/mnt/secrets`. Le volume secret possède deux secrets, « mysecret1 » et « mysecret2 ».

<!-- https://github.com/Azure/azure-docs-json-samples/blob/master/container-instances/aci-deploy-volume-secret.json -->
[!code-json[volume-secret](~/azure-docs-json-samples/container-instances/aci-deploy-volume-secret.json)]

Pour effectuer un déploiement avec le modèle Resource Manager, enregistrez le JSON précédent dans un fichier nommé `deploy-aci.json`, puis exécutez la commande [az deployment group create][az-deployment-group-create] avec le paramètre `--template-file` :

```azurecli-interactive
# Deploy with Resource Manager template
az deployment group create \
  --resource-group myResourceGroup \
  --template-file deploy-aci.json
```

## <a name="next-steps"></a>Étapes suivantes

### <a name="volumes"></a>Volumes

Découvrez comment monter d’autres types de volumes dans Azure Container Instances :

* [Monter un partage de fichiers Azure dans Azure Container Instances](container-instances-volume-azure-files.md)
* [Monter un volume emptyDir dans Azure Container Instances](container-instances-volume-emptydir.md)
* [Monter un volume gitRepo dans Azure Container Instances](container-instances-volume-gitrepo.md)

### <a name="secure-environment-variables"></a>Protection des variables d’environnement

Une autre méthode pour fournir des informations sensibles aux conteneurs (y compris les conteneurs Windows) s’effectue à l’aide de la [protection des variables d’environnement](container-instances-environment-variables.md#secure-values).

<!-- LINKS - External -->
[tmpfs]: https://wikipedia.org/wiki/Tmpfs

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
