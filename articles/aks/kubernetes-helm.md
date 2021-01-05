---
title: Installer des applications existantes avec Helm dans AKS
description: Apprendre à utiliser l’outil d’empaquetage Helm pour déployer des conteneurs dans un cluster AKS (Azure Kubernetes Service)
services: container-service
author: zr-msft
ms.topic: article
ms.date: 12/07/2020
ms.author: zarhoads
ms.openlocfilehash: f12dffe0b538738a8f6dd00cd3d87d44da828f21
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779165"
---
# <a name="install-existing-applications-with-helm-in-azure-kubernetes-service-aks"></a>Installer des applications existantes avec Helm dans Azure Kubernetes service (AKS)

[Helm][helm] est un outil d’empaquetage open source qui vous aide à installer et à gérer le cycle de vie d’applications Kubernetes. À l’instar de gestionnaires de package Linux tels que *APT* et *Yum*, Helm sert à gérer les graphiques Kubernetes, qui sont des packages de ressources Kubernetes préconfigurés.

Cet article vous montre comment configurer et utiliser Helm dans un cluster Kubernetes AKS.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Vous devez aussi avoir installé l’interface CLI Helm, qui est le client s’exécutant sur votre système de développement. Il vous permet de démarrer, arrêter et gérer les applications avec Helm. Si vous utilisez Azure Cloud Shell, l’interface CLI Helm est déjà installée. Pour obtenir les instructions d’installation sur votre plateforme locale, consultez [Installing Helm][helm-install] (Installation de Helm).

> [!IMPORTANT]
> Helm est prévu pour s’exécuter sur des nœuds Linux. Si vous avez des nœuds Windows Server dans votre cluster, vous devez veiller à ce que l’exécution des pods Helm soit planifiée uniquement sur des nœuds Linux. Vous devez aussi veiller à ce que les graphiques Helm que vous installez s’exécutent sur les bons nœuds. Les commandes figurant dans cet article utilisent [node-selectors][k8s-node-selector] pour garantir que les pods sont planifiés sur les nœuds adéquats, mais il se peut que certains graphiques Helm n’exposent pas de sélecteur de nœud. Vous pouvez aussi envisager d’utiliser d’autres options sur votre cluster, comme des [teintes][taints] (« taints »).

## <a name="verify-your-version-of-helm"></a>Vérifier la version de Helm

Utilisez la commande `helm version` pour vérifier que Helm 3 est installé :

```console
helm version
```

L’exemple suivant montre la version 3.0.0 de Helm installée :

```console
$ helm version

version.BuildInfo{Version:"v3.0.0", GitCommit:"e29ce2a54e96cd02ccfce88bee4f58bb6e2a28b6", GitTreeState:"clean", GoVersion:"go1.13.4"}
```

## <a name="install-an-application-with-helm-v3"></a>Installer une application avec Helm v3

### <a name="add-helm-repositories"></a>Ajouter des référentiels Helm

Utilisez la commande [helm repo][helm-repo-add] pour ajouter le référentiel *ingress-nginx*.

```console
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```

### <a name="find-helm-charts"></a>Rechercher des graphiques Helm

Les graphiques Helm servent à déployer des applications dans un cluster Kubernetes. Pour rechercher des graphiques Helm précréés, vous devez utiliser la commande [helm search][helm-search] :

```console
helm search repo ingress-nginx
```

La sortie condensée suivante montre certains des graphiques Helm disponibles :

```console
$ helm search repo ingress-nginx

NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
ingress-nginx/ingress-nginx     2.12.0          0.34.1          Ingress controller for Kubernetes using NGINX a...
```

Pour mettre à jour la liste des graphiques, utilisez la commande [helm repo update][helm-repo-update].

```console
helm repo update
```

L’exemple suivant montre une mise à jour de référentiel réussie :

```console
$ helm repo update

Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "ingress-nginx" chart repository
Update Complete. ⎈ Happy Helming!⎈
```

### <a name="run-helm-charts"></a>Exécuter des graphiques Helm

Pour installer des charts avec Helm, utilisez la commande [helm install][helm-install-command] et spécifiez un nom de version et le nom du chart à installer. Pour vous montrer comment se déroule l’installation d’un graphique Helm, nous allons effectuer un déploiement nginx de base à l’aide d’un graphique Helm.

```console
helm install my-nginx-ingress ingress-nginx/ingress-nginx \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

La sortie condensée suivante montre l’état du déploiement des ressources Kubernetes créées par le graphique Helm :

```console
$ helm install my-nginx-ingress ingress-nginx/ingress-nginx \
>     --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
>     --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux

NAME: my-nginx-ingress
LAST DEPLOYED: Fri Nov 22 10:08:06 2019
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The nginx-ingress controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace default get services -o wide -w my-nginx-ingress-ingress-nginx-controller'
...
```

Utilisez la commande `kubectl get services` pour accéder à l’*EXTERNAL-IP* de votre service.

```console
kubectl --namespace default get services -o wide -w my-nginx-ingress-ingress-nginx-controller
```

Par exemple, la commande ci-dessous affiche l’*EXTERNAL-IP* pour le service *my-nginx-ingress-ingress-nginx-controller* :

```console
$ kubectl --namespace default get services -o wide -w my-nginx-ingress-ingress-nginx-controller

NAME                                        TYPE           CLUSTER-IP   EXTERNAL-IP      PORT(S)                      AGE   SELECTOR
my-nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.2.237   <EXTERNAL-IP>    80:31380/TCP,443:32239/TCP   72s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=my-nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

### <a name="list-releases"></a>Répertorier les versions

Pour afficher la liste des versions installées sur votre cluster, utilisez la commande `helm list`.

```console
helm list
```

L’exemple suivant montre la version *my-nginx-ingress* déployée à l’étape précédente :

```console
$ helm list

NAME                NAMESPACE   REVISION    UPDATED                                 STATUS      CHART                   APP VERSION
my-nginx-ingress    default     1           2019-11-22 10:08:06.048477 -0600 CST    deployed    nginx-ingress-1.25.0    0.26.1 
```

### <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous déployez un graphique Helm, une série de ressources Kubernetes est créée. Ces ressources incluent des pods, des déploiements et des services. Pour nettoyer ces ressources, utilisez la commande [helm uninstall][helm-cleanup] et spécifiez le nom de votre version, tel que fourni par la commande `helm list` précédente.

```console
helm uninstall my-nginx-ingress
```

L’exemple suivant illustre la version nommée *my-nginx-ingress* qui a été désinstallé :

```console
$ helm uninstall my-nginx-ingress

release "my-nginx-ingress" uninstalled
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la gestion des déploiements d’applications Kubernetes, consultez la documentation Helm.

> [!div class="nextstepaction"]
> [Documentation de Helm][helm-documentation]

<!-- LINKS - external -->
[helm]: https://github.com/kubernetes/helm/
[helm-cleanup]: https://helm.sh/docs/intro/using_helm/#helm-uninstall-uninstalling-a-release
[helm-documentation]: https://helm.sh/docs/
[helm-install]: https://helm.sh/docs/intro/install/
[helm-install-command]: https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package
[helm-repo-add]: https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
[helm-search]: https://helm.sh/docs/intro/using_helm/#helm-search-finding-charts
[helm-repo-update]: https://helm.sh/docs/intro/using_helm/#helm-repo-working-with-repositories
            
<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[taints]: operator-best-practices-advanced-scheduler.md
