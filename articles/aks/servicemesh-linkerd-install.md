---
title: Installer Linkerd dans AKS (Azure Kubernetes Service)
description: Découvrir comment installer et utiliser Linkerd pour créer un maillage de services (service mesh) dans un cluster AKS (Azure Kubernetes Service)
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
zone_pivot_groups: client-operating-system
ms.openlocfilehash: d5b2ec220bbf14a79247ed2c78a0e84f4717fae3
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94684234"
---
# <a name="install-linkerd-in-azure-kubernetes-service-aks"></a>Installer Linkerd dans AKS (Azure Kubernetes Service)

[Linkerd][linkerd-github] est un maillage de services open source et un [projet d’incubation CNCF][linkerd-cncf]. Linkerd est un maillage de services ultraléger qui offre des fonctionnalités axées notamment sur les aspects suivants : gestion du trafic, identité et sécurité du service, fiabilité et observabilité. Pour obtenir des informations officielles, consultez le [FAQ sur Linkerd][linkerd-faq] et la documentation sur l’[architecture de Linkerd][linkerd-architecture].

Cet article vous montre comment installer Linkerd. Le binaire client Linkerd, `linkerd`, est installé sur votre machine cliente tandis que les composants Linkerd sont installés dans un cluster Kubernetes sur AKS.

> [!NOTE]
> Ces instructions font référence à la version `stable-2.6.0` de Linkerd.
>
> Vous pouvez exécuter la version `stable-2.6.x` de Linkerd sur les versions `1.13+` de Kubernetes. D’autres versions de Linkerd (edge et stable) sont disponibles sur [GitHub][linkerd-github-releases].

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Télécharger et installer le fichier binaire client linkerd de Linkerd
> * Installer Linkerd sur AKS
> * Valider l’installation de Linkerd
> * Accéder au tableau de bord
> * Désinstaller Linkerd d’AKS

## <a name="before-you-begin"></a>Avant de commencer

Les étapes détaillées dans cet article supposent que vous avez créé un cluster AKS (Kubernetes `1.13` et ultérieur, avec RBAC Kubernetes activé) et que vous avez établi une connexion `kubectl` avec le cluster. Si vous avez besoin d’aide avec l’un quelconque de ces éléments, consultez le [guide de démarrage rapide d’AKS][aks-quickstart].

Tous les pods Linkerd doivent être planifiés pour s’exécuter sur des nœuds Linux. Cette configuration est la valeur par défaut dans la méthode d’installation détaillée ci-dessous et ne nécessite aucune action supplémentaire.

Cet article répartit les instructions d’installation de Linkerd en plusieurs étapes discrètes. Le résultat est, par sa structure, identique à celui obtenu avec le [guide][linkerd-getting-started] de démarrage officiel de Linkerd.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download and install client binary](includes/servicemesh/linkerd/install-client-binary-linux.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [MacOS - download and install client binary](includes/servicemesh/linkerd/install-client-binary-macos.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download and install client binary](includes/servicemesh/linkerd/install-client-binary-windows.md)]

::: zone-end

## <a name="install-linkerd-on-aks"></a>Installer Linkerd sur AKS

Avant d’installer Linkerd, nous allons exécuter des vérifications de pré-installation pour déterminer si le plan de contrôle peut être installé sur notre cluster AKS :

```console
linkerd check --pre
```

Vous devriez obtenir une sortie similaire à la suivante, ce qui indique que votre cluster AKS est une cible d’installation valide pour Linkerd :

```console
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version
√ is running the minimum kubectl version

pre-kubernetes-setup
--------------------
√ control plane namespace does not already exist
√ can create Namespaces
√ can create ClusterRoles
√ can create ClusterRoleBindings
√ can create CustomResourceDefinitions
√ can create PodSecurityPolicies
√ can create ServiceAccounts
√ can create Services
√ can create Deployments
√ can create CronJobs
√ can create ConfigMaps
√ no clock skew detected

pre-kubernetes-capability
-------------------------
√ has NET_ADMIN capability
√ has NET_RAW capability

pre-linkerd-global-resources
----------------------------
√ no ClusterRoles exist
√ no ClusterRoleBindings exist
√ no CustomResourceDefinitions exist
√ no MutatingWebhookConfigurations exist
√ no ValidatingWebhookConfigurations exist
√ no PodSecurityPolicies exist

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

Status check results are √
```

Il est maintenant temps d’installer les composants Linkerd. Utilisez les binaires `linkerd` et `kubectl` pour installer les composants Linkerd dans votre cluster AKS. Un espace de noms `linkerd` est automatiquement créé, et les composants sont installés dans cet espace de noms.

```console
linkerd install | kubectl apply -f -
```

Linkerd déploie plusieurs objets. Vous pouvez voir la liste de ces objets dans la sortie de votre commande `linkerd install` ci-dessus. Le déploiement des composants Linkerd prend environ 1 minute selon votre environnement de cluster.

À ce stade, vous avez déployé Linkerd sur votre cluster AKS. Pour veiller au bon déploiement de Linkerd, passons à la section suivante pour [valider l’installation de Linkerd](#validate-the-linkerd-installation).

## <a name="validate-the-linkerd-installation"></a>Valider l’installation de Linkerd

Confirmez que les ressources ont été créées. Utilisez les commandes [kubectl get svc][kubectl-get] et [kubectl get pod][kubectl-get] pour interroger l’espace de noms `linkerd` où les composants Linkerd ont été installés par la commande `linkerd install` :

```console
kubectl get svc --namespace linkerd --output wide
kubectl get pod --namespace linkerd --output wide
```

L’exemple de sortie suivant montre les services et les pods (planifiés sur des nœuds Linux) qui doivent maintenant être en cours d’exécution :

```console
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE  SELECTOR
linkerd-controller-api   ClusterIP   10.0.110.67    <none>        8085/TCP            66s  linkerd.io/control-plane-component=controller
linkerd-destination      ClusterIP   10.0.224.29    <none>        8086/TCP            66s  linkerd.io/control-plane-component=controller
linkerd-dst              ClusterIP   10.0.225.148   <none>        8086/TCP            66s  linkerd.io/control-plane-component=destination
linkerd-grafana          ClusterIP   10.0.61.124    <none>        3000/TCP            65s  linkerd.io/control-plane-component=grafana
linkerd-identity         ClusterIP   10.0.6.104     <none>        8080/TCP            67s  linkerd.io/control-plane-component=identity
linkerd-prometheus       ClusterIP   10.0.27.168    <none>        9090/TCP            65s  linkerd.io/control-plane-component=prometheus
linkerd-proxy-injector   ClusterIP   10.0.100.133   <none>        443/TCP             64s  linkerd.io/control-plane-component=proxy-injector
linkerd-sp-validator     ClusterIP   10.0.221.5     <none>        443/TCP             64s  linkerd.io/control-plane-component=sp-validator
linkerd-tap              ClusterIP   10.0.18.14     <none>        8088/TCP,443/TCP    64s  linkerd.io/control-plane-component=tap
linkerd-web              ClusterIP   10.0.37.108    <none>        8084/TCP,9994/TCP   66s  linkerd.io/control-plane-component=web

NAME                                      READY   STATUS    RESTARTS   AGE   IP            NODE                            NOMINATED NODE   READINESS GATES
linkerd-controller-66ddc9f94f-cm9kt       3/3     Running   0          66s   10.240.0.50   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-destination-c94bc454-qpkng        2/2     Running   0          66s   10.240.0.78   aks-linux-16165125-vmss000002   <none>           <none>
linkerd-grafana-6868fdcb66-4cmq2          2/2     Running   0          65s   10.240.0.69   aks-linux-16165125-vmss000002   <none>           <none>
linkerd-identity-74d8df4b85-tqq8f         2/2     Running   0          66s   10.240.0.48   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-prometheus-699587cf8-k8ghg        2/2     Running   0          65s   10.240.0.41   aks-linux-16165125-vmss000001   <none>           <none>
linkerd-proxy-injector-6556447f64-n29wr   2/2     Running   0          64s   10.240.0.32   aks-linux-16165125-vmss000000   <none>           <none>
linkerd-sp-validator-56745cd567-v4x7h     2/2     Running   0          64s   10.240.0.6    aks-linux-16165125-vmss000000   <none>           <none>
linkerd-tap-5cd9fc566-ct988               2/2     Running   0          64s   10.240.0.15   aks-linux-16165125-vmss000000   <none>           <none>
linkerd-web-774c79b6d5-dhhwf              2/2     Running   0          65s   10.240.0.70   aks-linux-16165125-vmss000002   <none>           <none>
```

Linkerd fournit une commande par le biais du binaire client `linkerd` pour vérifier que le plan de contrôle Linkerd est correctement installé et configuré.

```console
linkerd check
```

Vous devriez obtenir une sortie similaire à la suivante, ce qui indique que votre installation a réussi :

```console
kubernetes-api
--------------
√ can initialize the client
√ can query the Kubernetes API

kubernetes-version
------------------
√ is running the minimum Kubernetes API version
√ is running the minimum kubectl version

linkerd-config
--------------
√ control plane Namespace exists
√ control plane ClusterRoles exist
√ control plane ClusterRoleBindings exist
√ control plane ServiceAccounts exist
√ control plane CustomResourceDefinitions exist
√ control plane MutatingWebhookConfigurations exist
√ control plane ValidatingWebhookConfigurations exist
√ control plane PodSecurityPolicies exist

linkerd-existence
-----------------
√ 'linkerd-config' config map exists
√ heartbeat ServiceAccount exist
√ control plane replica sets are ready
√ no unschedulable pods
√ controller pod is running
√ can initialize the client
√ can query the control plane API

linkerd-api
-----------
√ control plane pods are ready
√ control plane self-check
√ [kubernetes] control plane can talk to Kubernetes
√ [prometheus] control plane can talk to Prometheus
√ no invalid service profiles

linkerd-version
---------------
√ can determine the latest version
√ cli is up-to-date

control-plane-version
---------------------
√ control plane is up-to-date
√ control plane and cli versions match

Status check results are √
```

## <a name="access-the-dashboard"></a>Accéder au tableau de bord

Linkerd est fourni avec un tableau de bord qui fournit des insights sur le maillage de services et les charges de travail. Pour accéder au tableau de bord, utilisez la commande `linkerd dashboard`. Cette commande utilise [kubectl port-forward][kubectl-port-forward] pour créer une connexion sécurisée entre votre machine cliente et les pods appropriés dans votre cluster AKS. Le tableau de bord s’ouvre ensuite automatiquement dans votre navigateur par défaut.

```console
linkerd dashboard
```

La commande crée également un port-forward et retourne un lien pour les tableaux de bord Grafana.

```console
Linkerd dashboard available at:
http://127.0.0.1:50750
Grafana dashboard available at:
http://127.0.0.1:50750/grafana
Opening Linkerd dashboard in the default browser
```

## <a name="uninstall-linkerd-from-aks"></a>Désinstaller Linkerd d’AKS

> [!WARNING]
> La suppression de Linkerd d’un système en cours d’exécution peut entraîner des problèmes de trafic entre vos services. Avant d’aller plus loin, vérifiez que des dispositions sont en place pour que votre système continue de fonctionner correctement sans Linkerd.

Vous devez d’abord supprimer les proxys du plan de données. Supprimez toutes les [annotations][linkerd-automatic-proxy-injection] d’injection de proxy automatique des espaces de noms de charge de travail et lancez vos déploiements de charge de travail. Vos charges de travail ne doivent plus être associées à des composants de plan de données.

Enfin, supprimez le plan de contrôle comme suit :

```console
linkerd install --ignore-cluster | kubectl delete -f -
```

## <a name="next-steps"></a>Étapes suivantes

Pour explorer d’autres options de configuration et d’installation pour Linkerd, consultez les pages d’aide officielles suivantes :

- [Linkerd - Installation de Helm][linkerd-install-with-helm]
- [Linkerd - Installation en plusieurs étapes pour tenir compte des privilèges de rôle][linkerd-multi-stage-installation]

Pour suivre des scénarios supplémentaires, consultez :

- [Démo Linkerd : emojivoto][linkerd-demo-emojivoto]
- [Démo Linkerd : books][linkerd-demo-books]

<!-- LINKS - external -->

[linkerd]: https://linkerd.io/
[linkerd-cncf]: https://landscape.cncf.io/selected=linkerd
[linkerd-faq]: https://linkerd.io/2/faq/
[linkerd-architecture]: https://linkerd.io/2/reference/architecture/
[linkerd-getting-started]: https://linkerd.io/2/getting-started/
[linkerd-overview]: https://linkerd.io/2/overview/
[linkerd-github]: https://github.com/linkerd/linkerd2
[linkerd-github-releases]: https://github.com/linkerd/linkerd2/releases/

[linkerd-install-with-helm]: https://linkerd.io/2/tasks/install-helm/
[linkerd-multi-stage-installation]: https://linkerd.io/2/tasks/install/#multi-stage-install
[linkerd-automatic-proxy-injection]: https://linkerd.io/2/features/proxy-injection/

[linkerd-demo-emojivoto]: https://linkerd.io/2/getting-started/#step-5-install-the-demo-app
[linkerd-demo-books]: https://linkerd.io/2/tasks/books/

[helm]: https://helm.sh

[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
