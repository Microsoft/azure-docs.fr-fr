---
title: Création d’un conteneur Windows Server sur un cluster AKS à l’aide d’Azure CLI
description: Découvrez comment créer rapidement un cluster Kubernetes et déployer une application dans un conteneur Windows Server dans Azure Kubernetes Service (AKS) à l’aide d’Azure CLI.
services: container-service
ms.topic: article
ms.date: 07/16/2020
ms.openlocfilehash: a14659b64bbc86cfc50cbf8a377c0245fba25065
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886241"
---
# <a name="create-a-windows-server-container-on-an-azure-kubernetes-service-aks-cluster-using-the-azure-cli"></a>Créer un conteneur Windows Server sur un cluster Azure Kubernetes Service (AKS) à l’aide d’Azure CLI

AKS (Azure Kubernetes Service) est un service Kubernetes managé qui vous permet de déployer et de gérer rapidement des clusters. Dans cet article, vous déployez un cluster AKS à l’aide d’Azure CLI. Vous déployez également un exemple d’application ASP.NET dans un conteneur Windows Server vers le cluster.

![Image de la navigation vers l’exemple d’application ASP.NET](media/windows-container/asp-net-sample-app.png)

Cet article suppose une compréhension élémentaire des concepts liés à Kubernetes. Pour plus d’informations, consultez [Concepts de base de Kubernetes pour AKS (Azure Kubernetes Service)][kubernetes-concepts].

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

### <a name="limitations"></a>Limites

Les limitations suivantes s’appliquent lorsque vous créez et gérez les clusters AKS prenant en charge plusieurs pools de nœuds :

* Vous ne pouvez pas supprimer le premier pool de nœuds.

Les limitations supplémentaires suivantes s’appliquent aux pools de nœuds Windows Server :

* Le cluster AKS peut comprendre au maximum 10 pools de nœuds.
* Le cluster AKS peut comprendre au maximum 100 nœuds dans chaque pool de nœuds.
* Le nom de pool de nœud Windows Server a une limite de 6 caractères.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un groupe logique dans lequel des ressources Azure sont déployées et gérées. Lorsque vous créez un groupe de ressources, vous devez spécifier un emplacement. Il s’agit de l’emplacement de stockage des métadonnées de groupe de ressources. C’est également là que vos ressources s’exécutent dans Azure si vous ne spécifiez pas une autre région lors de la création de ressources. Créez un groupe de ressources avec la commande [az group create][az-group-create].

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

> [!NOTE]
> Cet article utilise la syntaxe Bash pour les commandes dans ce didacticiel.
> Si vous utilisez Azure Cloud Shell, assurez-vous que la liste déroulante dans le coin supérieur gauche de la fenêtre de Cloud Shell est définie sur **Bash**.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

L’exemple de sortie suivant montre que le groupe de ressources a été créé correctement :

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/myResourceGroup",
  "location": "eastus",
  "managedBy": null,
  "name": "myResourceGroup",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": null
}
```

## <a name="create-an-aks-cluster"></a>Créer un cluster AKS

Pour pouvoir exécuter un cluster AKS qui prend en charge des pools de nœuds pour les conteneurs Windows Server, votre cluster doit appliquer une stratégie de réseau qui utilise un plug-in réseau [Azure CNI][azure-cni-about] (avancé). Pour plus d’informations sur la planification des plages de sous-réseau nécessaires et les considérations réseau, consultez [Configurer le réseau Azure CNI][use-advanced-networking]. Utilisez la commande [az aks create][az-aks-create] pour créer un cluster AKS nommé *myAKSCluster*. Cette commande crée les ressources réseau nécessaires si elles n’existent pas.

* Le cluster est configuré avec deux nœuds
* Les paramètres *windows-admin-password* et *windows-admin-username* définissent les informations d’identification administrateur pour les conteneurs Windows Server créés sur le cluster et doivent répondre aux [exigences relatives aux mots de passe de Windows Server][windows-server-password].
* Le pool de nœuds utilise `VirtualMachineScaleSets`

> [!NOTE]
> Pour garantir un fonctionnement fiable de votre cluster, vous devez exécuter au moins 2 (deux) nœuds dans le pool de nœuds par défaut.

Fournissez votre propre *PASSWORD_WIN* sécurisé (n’oubliez pas que les commandes dans cet article sont entrées dans un interpréteur de commandes BASH) :

```azurecli-interactive
PASSWORD_WIN="P@ssw0rd1234"

az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 2 \
    --enable-addons monitoring \
    --generate-ssh-keys \
    --windows-admin-password $PASSWORD_WIN \
    --windows-admin-username azureuser \
    --vm-set-type VirtualMachineScaleSets \
    --network-plugin azure
```

> [!NOTE]
> Si vous recevez une erreur de validation de mot de passe, vérifiez que le paramètre *windows-admin-password* répond aux [exigences en matière de mot de passe de Windows Server][windows-server-password]. Si votre mot de passe répond aux conditions, essayez de créer votre groupe de ressources dans une autre région. Puis essayez de créer le cluster avec le nouveau groupe de ressources.

Au bout de quelques minutes, la commande se termine et retourne des informations au format JSON sur le cluster. Parfois, le provisionnement du cluster peut prendre plus de quelques minutes. Autorisez jusqu’à 10 minutes dans ces cas de figure.

## <a name="add-a-windows-server-node-pool"></a>Ajouter un pool de nœuds Windows Server

Par défaut, un cluster AKS est créé avec un pool de nœuds qui peut exécuter des conteneurs Linux. Utilisez la commande `az aks nodepool add` pour ajouter un pool de nœuds supplémentaire qui peut exécuter des conteneurs Windows Server en même temps que le pool de nœuds Linux.

```azurecli
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --os-type Windows \
    --name npwin \
    --node-count 1
```

La commande ci-dessus crée un pool de nœuds nommé *npwin* et l’ajoute à *myAKSCluster*. Lorsque vous créez un pool de nœuds pour exécuter des conteneurs Windows Server, la valeur par défaut de *node-vm-size* est *Standard_D2s_v3*. Si vous choisissez de définir le paramètre *node-vm-size*, veuillez consulter la liste des [tailles de machines virtuelles limitées][restricted-vm-sizes]. La taille minimale recommandée est *Standard_D2s_v3*. La commande ci-dessus utilise également le sous-réseau par défaut dans le réseau virtuel par défaut créé lors de l’exécution de `az aks create`.

## <a name="connect-to-the-cluster"></a>Se connecter au cluster

Pour gérer un cluster Kubernetes, vous utilisez [kubectl][kubectl], le client de ligne de commande Kubernetes. Si vous utilisez Azure Cloud Shell, `kubectl` est déjà installé. Pour installer `kubectl` en local, utilisez la commande [az aks install-cli][az-aks-install-cli] :

```azurecli
az aks install-cli
```

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials][az-aks-get-credentials]. Cette commande télécharge les informations d’identification et configure l’interface CLI Kubernetes pour les utiliser.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Pour vérifier la connexion à votre cluster, utilisez la commande [kubectl get][kubectl-get] pour retourner une liste des nœuds du cluster.

```console
kubectl get nodes
```

L’exemple de sortie suivant montre tous les nœuds du cluster. Vérifiez que l’état de tous les nœuds est *Prêt* :

```output
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.16.9
aksnpwin987654                      Ready    agent   108s   v1.16.9
```

## <a name="run-the-application"></a>Exécution de l'application

Un fichier manifeste Kubernetes définit un état souhaité pour le cluster, notamment les images conteneur à exécuter. Dans cet article, un manifeste est utilisé pour créer tous les objets nécessaires pour exécuter l’exemple d’application ASP.NET dans un conteneur Windows Server. Ce manifeste comprend un [déploiement Kubernetes][kubernetes-deployment] pour l’exemple d’application ASP.NET et un [service Kubernetes][kubernetes-service] externe pour accéder à l’application à partir d’Internet.

L’exemple d’application ASP.NET est fourni dans le cadre des [Exemples .NET Framework][dotnet-samples] et s’exécute dans un conteneur Windows Server. AKS exige que les conteneurs Windows Server soient basés sur des images de *Windows Server 2019* ou versions supérieures. Le fichier manifeste Kubernetes doit également définir un [sélecteur de nœud][node-selector] pour indiquer à votre cluster AKS d’exécuter le pod de votre exemple d’application ASP.NET sur un nœud qui peut exécuter des conteneurs Windows Server.

Créez un fichier nommé `sample.yaml`, et copiez-y la définition YAML suivante. Si vous utilisez Azure Cloud Shell, vous pouvez créer ce fichier à l’aide de `vi` ou de `nano` comme si vous travailliez sur un système virtuel ou physique :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample
  labels:
    app: sample
spec:
  replicas: 1
  template:
    metadata:
      name: sample
      labels:
        app: sample
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": windows
      containers:
      - name: sample
        image: mcr.microsoft.com/dotnet/framework/samples:aspnetapp
        resources:
          limits:
            cpu: 1
            memory: 800M
          requests:
            cpu: .1
            memory: 300M
        ports:
          - containerPort: 80
  selector:
    matchLabels:
      app: sample
---
apiVersion: v1
kind: Service
metadata:
  name: sample
spec:
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
  selector:
    app: sample
```

Déployez l’application à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f sample.yaml
```

L’exemple de sortie suivant montre que le déploiement et le service ont été créés correctement :

```output
deployment.apps/sample created
service/sample created
```

## <a name="test-the-application"></a>Test de l’application

Quand l’application s’exécute, un service Kubernetes expose le front-end de l’application sur Internet. L’exécution de ce processus peut prendre plusieurs minutes. Parfois, le provisionnement du service peut prendre plus de quelques minutes. Autorisez jusqu’à 10 minutes dans ces cas de figure.

Pour surveiller la progression, utilisez la commande [kubectl get service][kubectl-get] avec l’argument `--watch`.

```console
kubectl get service sample --watch
```

Dans un premier temps, la valeur *EXTERNAL-IP* pour *l’exemple* de service apparaît comme étant *en attente*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
sample             LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Quand l’adresse *EXTERNAL-IP* passe de l’état *pending* à une adresse IP publique réelle, utilisez `CTRL-C` pour arrêter le processus de surveillance `kubectl`. L’exemple de sortie suivant montre une adresse IP publique valide affectée au service :

```output
sample  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

Pour voir l’exemple d’application en action, ouvrez un navigateur web en utilisant l’adresse IP externe de votre service.

![Image de la navigation vers l’exemple d’application ASP.NET](media/windows-container/asp-net-sample-app.png)

> [!Note]
> Si vous dépassez le délai d’expiration en tentant de charger la page, vous devez vérifier que l’exemple d’application est prêt avec la commande [kubectl get pods --watch] suivante. Parfois, le conteneur Windows n’est pas démarré au moment où votre adresse IP externe est disponible.

## <a name="delete-cluster"></a>Supprimer un cluster

Lorsque vous n’avez plus besoin du cluster, utilisez la commande [az group delete][az-group-delete] pour supprimer le groupe de ressources, le service conteneur et toutes les ressources associées.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

> [!NOTE]
> Lorsque vous supprimez le cluster, le principal de service Azure Active Directory utilisé par le cluster AKS n’est pas supprimé. Pour obtenir des instructions sur la façon de supprimer le principal de service, consultez [Considérations et suppression du principal de service AKS][sp-delete]. Si vous avez utilisé une identité managée, l’identité est managée par la plateforme et n’a pas besoin d’être supprimée.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez déployé un cluster Kubernetes et déployé un exemple d’application ASP.NET dans un conteneur Windows Server vers ce cluster. [Accédez au tableau de bord web Kubernetes][kubernetes-dashboard] pour le cluster que vous venez de créer.

Pour en savoir plus sur AKS et parcourir le code complet de l’exemple de déploiement, passez au tutoriel sur le cluster Kubernetes.

> [!div class="nextstepaction"]
> [Didacticiel AKS][aks-tutorial]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[dotnet-samples]: https://hub.docker.com/_/microsoft-dotnet-framework-samples/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md

<!-- LINKS - internal -->
[kubernetes-concepts]: concepts-clusters-workloads.md
[aks-monitor]: ../azure-monitor/insights/container-insights-onboard.md
[aks-tutorial]: ./tutorial-kubernetes-prepare-app.md
[az-aks-browse]: /cli/azure/aks?view=azure-cli-latest#az-aks-browse
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-aks-install-cli]: /cli/azure/aks?view=azure-cli-latest#az-aks-install-cli
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[az-provider-register]: /cli/azure/provider#az-provider-register
[azure-cli-install]: /cli/azure/install-azure-cli
[azure-cni-about]: concepts-network.md#azure-cni-advanced-networking
[sp-delete]: kubernetes-service-principal.md#additional-considerations
[azure-portal]: https://portal.azure.com
[kubernetes-deployment]: concepts-clusters-workloads.md#deployments-and-yaml-manifests
[kubernetes-service]: concepts-network.md#services
[kubernetes-dashboard]: kubernetes-dashboard.md
[restricted-vm-sizes]: quotas-skus-regions.md#restricted-vm-sizes
[use-advanced-networking]: configure-azure-cni.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[windows-server-password]: /windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference