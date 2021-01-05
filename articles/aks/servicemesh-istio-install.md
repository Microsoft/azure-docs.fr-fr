---
title: Installer Istio dans AKS (Azure Kubernetes Service)
description: Découvrez comment installer et utiliser Istio pour créer un maillage de services (service mesh) dans un cluster AKS (Azure Kubernetes Service)
author: paulbouwer
ms.topic: article
ms.date: 10/02/2020
ms.author: pabouwer
zone_pivot_groups: client-operating-system
ms.openlocfilehash: c42e9c31397e9313898d7029366bc8de169d368e
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94683826"
---
# <a name="install-and-use-istio-in-azure-kubernetes-service-aks"></a>Installer et utiliser Istio dans AKS (Azure Kubernetes Service)

[Istio][istio-github] est un maillage de services open source qui fournit un jeu de fonctionnalités de base dans les microservices d’un cluster Kubernetes. Ces fonctionnalités incluent la gestion du trafic, l’identité et la sécurité des services, l’application des stratégies et l’observabilité. Pour plus d’informations sur Istio, consultez la documentation officielle [What is Istio?][istio-docs-concepts].

Cet article vous montre comment installer Istio. Le binaire client Istio, `istioctl`, est installé sur votre machine cliente et les composants Istio sont installés dans un cluster Kubernetes sur AKS.

> [!NOTE]
> Les instructions suivantes référencent la version d’Istio `1.7.3`.
>
> Les versions Istio `1.7.x` ont été testées par l’équipe Istio par rapport à la version Kubernetes`1.16+`. Vous trouverez d’autres versions d’Istio dans la [page GitHub dédiée aux versions d’Istio][istio-github-releases], des informations sur chaque version sur le [site des actualités Istio][istio-release-notes] ainsi que les versions de Kubernetes prises en charge dans le [FAQ général sur Istio][istio-faq].

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Télécharger et installer le binaire client istioctl d’Istio
> * Installer Istio sur AKS
> * Valider l’installation d’Istio
> * Accéder aux modules complémentaires
> * Désinstaller Istio d’AKS

## <a name="before-you-begin"></a>Avant de commencer

Les étapes détaillées dans cet article supposent que vous avez créé un cluster AKS (Kubernetes `1.16` et ultérieur, avec RBAC Kubernetes activé) et que vous avez établi une connexion `kubectl` avec le cluster. Si vous avez besoin d’aide avec l’un quelconque de ces éléments, consultez le [guide de démarrage rapide d’AKS][aks-quickstart].

Assurez-vous de lire la documentation relative aux [niveaux de performance et à l’extensibilité d’Istio](https://istio.io/docs/concepts/performance-and-scalability/) pour comprendre quelles sont les ressources supplémentaires nécessaires à l’exécution d’Istio dans votre cluster AKS. Les exigences de base et de mémoire varient en fonction de votre charge de travail précise. Choisissez un nombre de nœuds et une taille de machine virtuelle appropriés pour votre configuration.

Cet article répartit les instructions d’installation d’Istio en plusieurs étapes discrètes. Le résultat final est, par sa structure, identique à celui obtenu avec le [guide][istio-install-istioctl] officiel d’installation d’Istio.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download and install client binary](includes/servicemesh/istio/install-client-binary-linux.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [macOS - download and install client binary](includes/servicemesh/istio/install-client-binary-macos.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download and install client binary](includes/servicemesh/istio/install-client-binary-windows.md)]

::: zone-end

## <a name="install-the-istio-operator-on-aks"></a>Installer l’opérateur Istio sur AKS

Istio fournit un [opérateur][istio-install-operator] pour gérer l’installation et les mises à jour des composants Istio au sein de votre cluster AKS. Nous allons installer l’opérateur Istio à l’aide du fichier binaire du client `istioctl`.

```bash
istioctl operator init
```

Vous devriez voir une sortie semblable à la sortie suivante pour confirmer que l’opérateur Istio a été installé.

```console
Using operator Deployment image: docker.io/istio/operator:1.7.3
✔ Istio operator installed
✔ Installation complete
```

L’opérateur Istio est installé dans l’espace de noms `istio-operator`. Interrogez l’espace de noms.

```bash
kubectl get all -n istio-operator
```

Vous devez voir les composants suivants déployés.

```console
NAME                                  READY   STATUS    RESTARTS   AGE
pod/istio-operator-6d7958b7bf-wxgdc   1/1     Running   0          2m43s

NAME                     TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/istio-operator   ClusterIP   10.0.8.57    <none>        8383/TCP   2m43s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-operator   1/1     1            1           2m43s

NAME                                        DESIRED   CURRENT   READY   AGE
replicaset.apps/istio-operator-6d7958b7bf   1         1         1       2m43s
```

Vous pouvez en savoir plus sur le modèle d’opérateur et sur la manière dont il peut aider à automatiser des tâches complexes via [kubernetes.io][kubernetes-operator].


### <a name="install-istio-components"></a>Installer les composants Istio

Maintenant que nous avons correctement installé l’opérateur Istio dans notre cluster AKS, il est temps d’installer les composants Istio. 

Nous allons tirer parti du `default` [profil de configuration Istio][istio-configuration-profiles] pour créer les [spécifications de l’opérateur Istio][istio-control-plane].

Vous pouvez exécuter la commande `istioctl` suivante pour afficher la configuration du profil de configuration Istio `default`.

```bash
istioctl profile dump default
```

> [!NOTE]
> Istio doit être actuellement planifié pour s’exécuter sur des nœuds Linux. Si vous avez des nœuds Windows Server dans votre cluster, vous devez veiller à ce que l’exécution des pods Istio soit planifiée uniquement sur des nœuds Linux. Nous allons utiliser des [sélecteurs de nœud][kubernetes-node-selectors] pour vérifier que les pods sont planifiés sur les nœuds appropriés.

> [!CAUTION]
> Les fonctionnalités d’[Istio CNI][istio-feature-cni] sont actuellement en [version alpha][istio-feature-stages], réfléchissez avant de les activer. 

Créez un fichier appelé `istio.aks.yaml` en utilisant le contenu suivant. Ce fichier contiendra la [spécification de l’opérateur Istio][istio-control-plane] pour la configuration d’Istio.

```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istio-control-plane
spec:
  # Use the default profile as the base
  # More details at: https://istio.io/docs/setup/additional-setup/config-profiles/
  profile: default
  # Enable the addons that we will want to use
  addonComponents:
    grafana:
      enabled: true
    prometheus:
      enabled: true
    tracing:
      enabled: true
    kiali:
      enabled: true
  values:
    global:
      # Ensure that the Istio pods are only scheduled to run on Linux nodes
      defaultNodeSelector:
        beta.kubernetes.io/os: linux
    kiali:
      dashboard:
        auth:
          strategy: anonymous 
```

Créez l’`istio-system`espace de noms et déployez la spécification de l’opérateur Istio dans cet espace de noms. L’opérateur Istio surveillera la spécification de l’opérateur Istio et l’utilisera pour installer et configurer Istio dans votre cluster AKS.

```bash
kubectl create ns istio-system

kubectl apply -f istio.aks.yaml 
```

À ce stade, vous avez déployé Istio dans votre cluster AKS. Pour nous assurer que nous avons un déploiement réussi d’Istio, nous allons passer à la section suivante pour [valider l’installation d’Istio](#validate-the-istio-installation).

## <a name="validate-the-istio-installation"></a>Valider l’installation d’Istio

Interrogez l’espace de noms `istio-system`, où les composants Istio et de module complémentaire ont été installés par l’opérateur Istio :

```bash
kubectl get all -n istio-system
```

Vous devez voir les composants suivants :

- `istio*` - les composants Istio
- `jaeger-*`, `tracing` et `zipkin` - le module complémentaire de suivi
- `prometheus` - le module complémentaire de métriques
- `grafana` - le module complémentaire du tableau de bord d’analyse et de surveillance
- `kiali` - le module complémentaire du tableau de bord de maillage de service

```console
NAME                                        READY   STATUS    RESTARTS   AGE
pod/grafana-7cf9794c74-mpfbp                1/1     Running   0          5m53s
pod/istio-ingressgateway-86b5dbdcb9-ndrp5   1/1     Running   0          5m57s
pod/istio-tracing-c98f4b8fc-zqklg           1/1     Running   0          82s
pod/istiod-6965c56995-4ph9h                 1/1     Running   0          6m15s
pod/kiali-7b44985d68-p87zh                  1/1     Running   0          81s
pod/prometheus-6868989549-5ghzz             1/1     Running   0          81s

NAME                                TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                                                      AGE
service/grafana                     ClusterIP      10.0.226.39    <none>         3000/TCP                                                     5m54s
service/istio-ingressgateway        LoadBalancer   10.0.143.56    20.53.72.254   15021:32166/TCP,80:31684/TCP,443:31302/TCP,15443:30863/TCP   5m57s
service/istiod                      ClusterIP      10.0.211.228   <none>         15010/TCP,15012/TCP,443/TCP,15014/TCP,853/TCP                6m16s
service/jaeger-agent                ClusterIP      None           <none>         5775/UDP,6831/UDP,6832/UDP                                   82s
service/jaeger-collector            ClusterIP      10.0.7.62      <none>         14267/TCP,14268/TCP,14250/TCP                                82s
service/jaeger-collector-headless   ClusterIP      None           <none>         14250/TCP                                                    82s
service/jaeger-query                ClusterIP      10.0.52.172    <none>         16686/TCP                                                    82s
service/kiali                       ClusterIP      10.0.71.179    <none>         20001/TCP                                                    82s
service/prometheus                  ClusterIP      10.0.171.151   <none>         9090/TCP                                                     82s
service/tracing                     ClusterIP      10.0.195.137   <none>         80/TCP                                                       82s
service/zipkin                      ClusterIP      10.0.136.111   <none>         9411/TCP                                                     82s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/grafana                1/1     1            1           5m54s
deployment.apps/istio-ingressgateway   1/1     1            1           5m58s
deployment.apps/istio-tracing          1/1     1            1           83s
deployment.apps/istiod                 1/1     1            1           6m16s
deployment.apps/kiali                  1/1     1            1           83s
deployment.apps/prometheus             1/1     1            1           82s

NAME                                              DESIRED   CURRENT   READY   AGE
replicaset.apps/grafana-7cf9794c74                1         1         1       5m54s
replicaset.apps/istio-ingressgateway-86b5dbdcb9   1         1         1       5m58s
replicaset.apps/istio-tracing-c98f4b8fc           1         1         1       83s
replicaset.apps/istiod-6965c56995                 1         1         1       6m16s
replicaset.apps/kiali-7b44985d68                  1         1         1       82s
replicaset.apps/prometheus-6868989549             1         1         1       82s

NAME                                                       REFERENCE                         TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/istio-ingressgateway   Deployment/istio-ingressgateway   7%/80%    1         5         1          5m57s
horizontalpodautoscaler.autoscaling/istiod                 Deployment/istiod                 1%/80%    1         5         1          6m16s
```

Vous pouvez également obtenir des informations supplémentaires sur l’installation en regardant les journaux de l’opérateur Istio.

```bash
kubectl logs -n istio-operator -l name=istio-operator -f
```

Si `istio-ingressgateway` indique `<pending>` en guise d’adresse IP externe, patientez quelques minutes, le temps qu’une adresse IP soit attribuée par le réseau Azure.

Tous les pods doivent indiquer l’état `Running`. Si vos pods n’ont pas cet état, patientez une minute ou deux. Si des pods signalent un problème, utilisez la commande [kubectl describe pod][kubectl-describe] pour examiner leur sortie et leur état.

## <a name="accessing-the-add-ons"></a>Accès aux modules complémentaires

Une série de modules complémentaires fournissant des fonctionnalités supplémentaires a été installée par l’opérateur Istio. Les applications web des modules complémentaires **ne sont pas** exposées publiquement par le biais d’une adresse IP externe. 

Pour accéder aux interfaces utilisateur des modules complémentaires, utilisez la commande `istioctl dashboard`. Cette commande utilise [kubectl port-forward][kubectl-port-forward] et un port aléatoire pour créer une connexion sécurisée entre votre machine cliente et le pod approprié dans votre cluster AKS. L’application web du module complémentaire s’ouvre ensuite automatiquement dans votre navigateur par défaut.

### <a name="grafana"></a>Grafana

Les tableaux de bord d’analyse et de surveillance pour Istio sont fournis par [Grafana][grafana]. Pensez à utiliser les informations d’identification que vous avez créées via le secret Grafana précédemment lorsque vous y êtes invité. Ouvrez le tableau de bord Grafana de façon sécurisée, comme ceci :

```console
istioctl dashboard grafana
```

### <a name="prometheus"></a>Prometheus

Les indicateurs de performance pour Istio sont fournis par [Prometheus][prometheus]. Ouvrez le tableau de bord Prometheus de façon sécurisée, comme ceci :

```console
istioctl dashboard prometheus
```

### <a name="jaeger"></a>Jaeger

Le suivi au sein d’Istio est assuré par [Jaeger][jaeger]. Ouvrez le tableau de bord Jaeger de façon sécurisée, comme ceci :

```console
istioctl dashboard jaeger
```

### <a name="kiali"></a>Kiali

Un tableau de bord d’observation du maillage de services est fourni par [Kiali][kiali]. Pensez à utiliser les informations d’identification que vous avez créées via le secret Kiali précédemment lorsque vous y êtes invité. Ouvrez le tableau de bord Kiali de façon sécurisée, comme ceci :

```console
istioctl dashboard kiali
```

### <a name="envoy"></a>Envoy

Une interface simple est disponible pour les proxys [Envoy][envoy]. Elle fournit des informations de configuration et des métriques pour un proxy Envoy s’exécutant dans un pod spécifié. Ouvrez l’interface Envoy de façon sécurisée, comme ceci :

```console
istioctl dashboard envoy <pod-name>.<namespace>
```

## <a name="uninstall-istio-from-aks"></a>Désinstaller Istio d’AKS

> [!WARNING]
> La suppression d’Istio à partir d’un système en cours d’exécution peut entraîner des problèmes de trafic entre vos services. Vérifiez que vous avez mis en place des dispositions pour que votre système continue de fonctionner correctement sans Istio avant de continuer.

### <a name="remove-istio"></a>Supprimer Istio

Pour supprimer Istio de votre cluster AKS, supprimez la ressource `IstioOperator` nommée `istio-control-plane` que nous avons ajoutée précédemment. L’opérateur Istio reconnaîtra que la spécification de l’opérateur Istio a été supprimée, puis supprimera tous les composants Istio associés.

```bash
kubectl delete istiooperator istio-control-plane -n istio-system
```

Vous pouvez exécuter la commande suivante pour vérifier le moment où tous les composants Istio ont été supprimés.

```bash
kubectl get all -n istio-system
```

### <a name="remove-istio-operator"></a>Supprimer l’opérateur Istio

Une fois Istio correctement désinstallé, vous pouvez également supprimer l’opérateur Istio.

```bash
istioctl operator remove
```

Enfin, supprimez les espaces de noms `istio-`.

```bash
kubectl delete ns istio-system
kubectl delete ns istio-operator
```

## <a name="next-steps"></a>Étapes suivantes

Pour explorer davantage d’options de configuration et d’installation pour Istio, consultez la documentation Istio officielle suivante :

- [Istio - guides d’installation][istio-installation-guides]

Pour suivre des scénarios supplémentaires, consultez :

- [Exemple d’application Istio BookInfo][istio-bookinfo-example]

<!-- LINKS - external -->
[istio]: https://istio.io
[helm]: https://helm.sh

[istio-faq]: https://istio.io/faq/general/
[istio-docs-concepts]: https://istio.io/docs/concepts/what-is-istio/
[istio-github]: https://github.com/istio/istio
[istio-github-releases]: https://github.com/istio/istio/releases
[istio-release-notes]: https://istio.io/news/
[istio-installation-guides]: https://istio.io/docs/setup/install/
[istio-install-download]: https://istio.io/docs/setup/kubernetes/download-release/
[istio-install-istioctl]: https://istio.io/docs/setup/install/istioctl/
[istio-install-operator]: https://istio.io/latest/docs/setup/install/operator/
[istio-configuration-profiles]: https://istio.io/docs/setup/additional-setup/config-profiles/
[istio-control-plane]: https://istio.io/docs/reference/config/istio.operator.v1alpha1/
[istio-bookinfo-example]: https://istio.io/docs/examples/bookinfo/

[istio-feature-stages]: https://istio.io/about/feature-stages/
[istio-feature-sds]: https://istio.io/docs/tasks/traffic-management/ingress/secure-ingress-sds/
[istio-feature-cni]: https://istio.io/docs/setup/additional-setup/cni/

[kubernetes-operator]: https://kubernetes.io/docs/concepts/extend-kubernetes/operator/
[kubernetes-feature-sa-projected-volume]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#service-account-token-volume-projection
[kubernetes-crd]: https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions
[kubernetes-secrets]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-node-selectors]: ./concepts-clusters-workloads.md#node-selectors
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/
[jaeger]: https://www.jaegertracing.io/
[kiali]: https://www.kiali.io/
[envoy]: https://www.envoyproxy.io/

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
