---
title: Déployer des conteneurs avec Helm dans Kubernetes sur Azure
description: Apprendre à utiliser l’outil d’empaquetage Helm pour déployer des conteneurs dans un cluster AKS (Azure Kubernetes Service)
services: container-service
author: zr-msft
ms.service: container-service
ms.topic: article
ms.date: 05/23/2019
ms.author: zarhoads
ms.openlocfilehash: bc74ac660c5bba0624416d0a1724d959a4c385a7
ms.sourcegitcommit: f176e5bb926476ec8f9e2a2829bda48d510fbed7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/04/2019
ms.locfileid: "70305271"
---
# <a name="install-applications-with-helm-in-azure-kubernetes-service-aks"></a>Installer des applications avec Helm dans Azure Kubernetes Service (AKS)

[Helm][helm] est un outil d’empaquetage open source qui vous aide à installer et à gérer le cycle de vie d’applications Kubernetes. À l’instar de gestionnaires de package Linux tels que *APT* et *Yum*, Helm sert à gérer des packages d'application (charts), qui sont des definitions de ressources Kubernetes préconfigurés.

Cet article vous montre comment configurer et utiliser Helm dans un cluster Kubernetes AKS.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Vous devez aussi avoir installé l’interface CLI Helm, qui est le client s’exécutant sur votre système de développement. Il vous permet de démarrer, arrêter et gérer les applications avec Helm. Si vous utilisez Azure Cloud Shell, l’interface CLI Helm est déjà installée. Pour obtenir les instructions d’installation sur votre plateforme local, consultez [Installing Helm][helm-install] (Installation de Helm).

> [!IMPORTANT]
> Helm est prévu pour s’exécuter sur des nœuds Linux. Si vous avez des nœuds Windows Server dans votre cluster, vous devez veiller à ce que l’exécution des pods Helm soit planifiée uniquement sur des nœuds Linux. Vous devez aussi veiller à ce que les charts Helm que vous installez s’exécutent sur les bons nœuds. Les commandes figurant dans cet article utilisent [node-selectors][k8s-node-selector] pour garantir que les pods sont planifiés sur les nœuds adéquats, mais il se peut que certains charts Helm n’exposent pas de sélecteur de nœud. Vous pouvez aussi envisager d’utiliser d’autres options sur votre cluster, comme des [teintes][taints] (« taints »).

## <a name="create-a-service-account"></a>Créer un compte de service

Avant de déployer Helm dans un cluster AKS où RBAC est activé, vous avez besoin d’un compte de service et d’une liaison de rôle pour le service Tiller. Pour plus d’informations sur la sécurisation de Helm / Tiller dans un cluster où RBAC est activé, consultez [Tiller, Namespaces and RBAC (Tiller, espaces de noms et RBAC)][tiller-rbac]. Si RBAC n’est pas activé sur votre cluster AKS, passez cette étape.

Créez un fichier nommé `helm-rbac.yaml`, et copiez-y le code YAML suivant :

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

Créez le compte de service et la liaison de rôle avec la commande `kubectl apply` :

```console
kubectl apply -f helm-rbac.yaml
```

## <a name="secure-tiller-and-helm"></a>Sécuriser Tiller et Helm

Le client Helm et le service Tiller s’authentifient et communiquent entre eux à l’aide des protocoles TLS/SSL. Cette méthode d’authentification permet de sécuriser le cluster Kubernetes et les services qui peuvent être déployés. Pour améliorer la sécurité, vous pouvez générer vos propres certificats signés. Chaque utilisateur Helm reçoit son propre certificat client et Tiller est initialisé dans le cluster Kubernetes avec les certificats appliqués. Pour plus d’informations, consultez [Using TLS/SSL between Helm and Tiller][helm-ssl].

Lorsque vous utilisez un cluster Kubernetes où RBAC est activé, vous pouvez contrôler le niveau d’accès au cluster de Tiller. Vous pouvez définir l’espace de noms Kubernetes dans lequel Tiller est déployé, et restreindre les espaces de noms dans lesquels Tiller peut ensuite déployer des ressources. Cette méthode vous permet de créer des instances de Tiller dans différents espaces de noms, de créer des limites de déploiement et de limiter les utilisateurs du client Helm à certains espaces de noms. Pour plus d’informations, consultez [Helm role-based access controls][helm-rbac].

## <a name="configure-helm"></a>Configurer Helm

Pour déployer une version basique de Tiller dans un cluster AKS, utilisez la commande [helm init][helm-init]. Si RBAC n’est pas activé sur votre cluster, supprimez l’argument et la valeur `--service-account`. Les exemples suivants définissent également [history-max][helm-history-max] sur 200.

Si vous avez configuré TLS/SSL pour Tiller et Helm, ignorez cette étape d’initialisation et fournissez le `--tiller-tls-` nécessaire, comme indiqué dans l’exemple suivant.

```console
helm init --history-max 200 --service-account tiller --node-selectors "beta.kubernetes.io/os=linux"
```

Si vous avez configuré TLS/SSL entre Helm et Tiller, fournissez les paramètres et les noms `--tiller-tls-*` de vos propres certificats, comme indiqué dans l’exemple suivant :

```console
helm init \
    --tiller-tls \
    --tiller-tls-cert tiller.cert.pem \
    --tiller-tls-key tiller.key.pem \
    --tiller-tls-verify \
    --tls-ca-cert ca.cert.pem \
    --history-max 200 \
    --service-account tiller \
    --node-selectors "beta.kubernetes.io/os=linux"
```

## <a name="find-helm-charts"></a>Rechercher des charts Helm

Les charts Helm servent à déployer des applications dans un cluster Kubernetes. Pour rechercher des charts Helm précréés, vous devez utiliser la commande [helm search][helm-search] :

```console
helm search
```

La sortie condensée suivante montre certains des charts Helm disponibles :

```
$ helm search

NAME                           CHART VERSION    APP VERSION  DESCRIPTION
stable/aerospike               0.1.7            v3.14.1.2    A Helm chart for Aerospike in Kubernetes
stable/anchore-engine          0.1.7            0.1.10       Anchore container analysis and policy evaluatio...
stable/apm-server              0.1.0            6.2.4        The server receives data from the Elastic APM a...
stable/ark                     1.0.1            0.8.2        A Helm chart for ark
stable/artifactory             7.2.1            6.0.0        Universal Repository Manager supporting all maj...
stable/artifactory-ha          0.2.1            6.0.0        Universal Repository Manager supporting all maj...
stable/auditbeat               0.1.0            6.2.4        A lightweight shipper to audit the activities o...
stable/aws-cluster-autoscaler  0.3.3                         Scales worker nodes within autoscaling groups.
stable/bitcoind                0.1.3            0.15.1       Bitcoin is an innovative payment network and a ...
stable/buildkite               0.2.3            3            Agent for Buildkite
stable/burrow                  0.4.4            0.17.1       Burrow is a permissionable smart contract machine
stable/centrifugo              2.0.1            1.7.3        Centrifugo is a real-time messaging server.
stable/cerebro                 0.1.0            0.7.3        A Helm chart for Cerebro - a web admin tool tha...
stable/cert-manager            v0.3.3           v0.3.1       A Helm chart for cert-manager
stable/chaoskube               0.7.0            0.8.0        Chaoskube periodically kills random pods in you...
stable/chartmuseum             1.5.0            0.7.0        Helm Chart Repository with support for Amazon S...
stable/chronograf              0.4.5            1.3          Open-source web application written in Go and R...
stable/cluster-autoscaler      0.6.4            1.2.2        Scales worker nodes within autoscaling groups.
stable/cockroachdb             1.1.1            2.0.0        CockroachDB is a scalable, survivable, strongly...
stable/concourse               1.10.1           3.14.1       Concourse is a simple and scalable CI system.
stable/consul                  3.2.0            1.0.0        Highly available and distributed service discov...
stable/coredns                 0.9.0            1.0.6        CoreDNS is a DNS server that chains plugins and...
stable/coscale                 0.2.1            3.9.1        CoScale Agent
stable/dask                    1.0.4            0.17.4       Distributed computation in Python with task sch...
stable/dask-distributed        2.0.2                         DEPRECATED: Distributed computation in Python
stable/datadog                 0.18.0           6.3.0        DataDog Agent
...
```

Pour mettre à jour la liste des charts, utilisez la commande [helm repo update][helm-repo-update]. L’exemple suivant montre une mise à jour de référentiel réussie :

```console
$ helm repo update

Hold tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete.
```

## <a name="run-helm-charts"></a>Exécuter des charts Helm

Pour installer des charts avec Helm, utilisez la commande [helm install][helm-install] et spécifiez le nom du chart à installer. Pour vous montrer comment se déroule l’installation d’un chart Helm, nous allons effectuer un déploiement nginx de base à l’aide d’un chart Helm. Si vous avez configuré TLS/SSL, ajoutez le paramètre `--tls` pour utiliser le certificat client Helm.

```console
helm install stable/nginx-ingress \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

La sortie condensée suivante montre l’état du déploiement des ressources Kubernetes créées par le chart Helm :

```
$ helm install stable/nginx-ingress --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux

NAME:   flailing-alpaca
LAST DEPLOYED: Thu May 23 12:55:21 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                                      DATA  AGE
flailing-alpaca-nginx-ingress-controller  1     0s

==> v1/Pod(related)
NAME                                                            READY  STATUS             RESTARTS  AGE
flailing-alpaca-nginx-ingress-controller-56666dfd9f-bq4cl       0/1    ContainerCreating  0         0s
flailing-alpaca-nginx-ingress-default-backend-66bc89dc44-m87bp  0/1    ContainerCreating  0         0s

==> v1/Service
NAME                                           TYPE          CLUSTER-IP  EXTERNAL-IP  PORT(S)                     AGE
flailing-alpaca-nginx-ingress-controller       LoadBalancer  10.0.109.7  <pending>    80:31219/TCP,443:32421/TCP  0s
flailing-alpaca-nginx-ingress-default-backend  ClusterIP     10.0.44.97  <none>       80/TCP                      0s
...
```

Pour accéder au service nginx-ingress-controller avec un navigateur web, attendez une à deux minutes que son adresse *EXTERNAL-IP* soit renseignée.

## <a name="list-helm-releases"></a>Liste des publications de Helm

Pour afficher la liste des publications installées sur votre cluster, utilisez la commande [helm list][helm-list]. L’exemple suivant montre la version de nginx-ingress déployée à l’étape précédente. Si vous avez configuré TLS/SSL, ajoutez le paramètre `--tls` pour utiliser le certificat client Helm.

```console
$ helm list

NAME                REVISION    UPDATED                     STATUS      CHART                 APP VERSION   NAMESPACE
flailing-alpaca   1         Thu May 23 12:55:21 2019    DEPLOYED    nginx-ingress-1.6.13    0.24.1      default
```

## <a name="clean-up-resources"></a>Supprimer des ressources

Quand vous déployez un chart Helm, une série de ressources Kubernetes est créée. Ces ressources incluent des pods, des déploiements et des services. Pour nettoyer ces ressources, utilisez la commande `helm delete` et spécifiez le nom de votre version, tel que fourni par la commande `helm list` précédente. L’exemple suivant supprime la version nommée *flailing-alpaca* :

```console
$ helm delete flailing-alpaca

release "flailing-alpaca" deleted
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la gestion des déploiements d’applications Kubernetes, consultez la documentation Helm.

> [!div class="nextstepaction"]
> [Documentation de Helm][helm-documentation]

<!-- LINKS - external -->
[helm]: https://github.com/kubernetes/helm/
[helm-documentation]: https://docs.helm.sh/
[helm-init]: https://docs.helm.sh/helm/#helm-init
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm
[helm-install-options]: https://github.com/kubernetes/helm/blob/master/docs/install.md
[helm-list]: https://docs.helm.sh/helm/#helm-list
[helm-history-max]: https://helm.sh/docs/using_helm/#initialize-helm-and-install-tiller
[helm-rbac]: https://docs.helm.sh/using_helm/#role-based-access-control
[helm-repo-update]: https://docs.helm.sh/helm/#helm-repo-update
[helm-search]: https://docs.helm.sh/helm/#helm-search
[tiller-rbac]: https://docs.helm.sh/using_helm/#tiller-namespaces-and-rbac
[helm-ssl]: https://docs.helm.sh/using_helm/#using-ssl-between-helm-and-tiller

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
[taints]: operator-best-practices-advanced-scheduler.md
