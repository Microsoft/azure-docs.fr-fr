---
title: Installer Consul dans AKS (Azure Kubernetes Service)
description: Découvrez comment installer et utiliser Consul pour créer un maillage de services (service mesh) dans un cluster AKS (Azure Kubernetes Service)
author: dstrebel
ms.topic: article
ms.date: 10/09/2019
ms.author: dastrebe
zone_pivot_groups: client-operating-system
ms.openlocfilehash: 7c5ad53c0040009e9ed1f28072540b46ce7b0b9a
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94683917"
---
# <a name="install-and-use-consul-in-azure-kubernetes-service-aks"></a>Installer et utiliser Consul dans AKS (Azure Kubernetes Service)

[Consul][consul-github] est un maillage de services open source qui fournit un jeu de fonctionnalités de base dans les microservices d’un cluster Kubernetes. Ces fonctionnalités incluent la détection des services, la vérification de l’intégrité, la segmentation des services et l’observabilité. Pour plus d’informations sur Consul, consultez la documentation officielle [What is Consul?][consul-docs-concepts].

Cet article vous montre comment installer Consul. Les composants Consul sont installés dans un cluster Kubernetes sur AKS.

> [!NOTE]
> Ces instructions font référence à la version `1.6.0` de Consul et utilisent la version `2.14.2` ou ultérieure de Helm.
>
> Vous pouvez exécuter les versions `1.6.x` de Consul sur les versions `1.13+` de Kubernetes. Vous trouverez d’autres versions de Consul dans la [page GitHub dédiée aux versions de Consul][consul-github-releases] et des informations sur chaque version sur le site [Consul - Release Notes][consul-release-notes].

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Installer les composants Consul sur AKS
> * Valider l’installation de Consul
> * Désinstaller Consul à partir d’AKS

## <a name="before-you-begin"></a>Avant de commencer

Les étapes détaillées dans cet article supposent que vous avez créé un cluster AKS (Kubernetes `1.13` et ultérieur, avec RBAC Kubernetes activé) et que vous avez établi une connexion `kubectl` avec le cluster. Si vous avez besoin d’aide avec l’un quelconque de ces éléments, consultez le [guide de démarrage rapide d’AKS][aks-quickstart]. Vérifiez que votre cluster dispose au minimum de 3 nœuds dans le pool de nœuds Linux.

Vous avez besoin de [Helm][helm] pour suivre ces instructions et installer Consul. Nous vous recommandons d’installer et de configurer correctement la dernière version stable dans votre cluster. Si vous avez besoin d’aide pour installer Helm, consultez les [consignes d’installation de Helm dans AKS][helm-install]. Tous les pods Consul doivent également être planifiés pour s’exécuter sur des nœuds Linux.

Cet article répartit les instructions d’installation de Consul en plusieurs étapes discrètes. Le résultat final est, par sa structure, identique à celui obtenu avec le [guide][consul-install-k8] officiel d’installation de Consul.

### <a name="install-the-consul-components-on-aks"></a>Installer les composants Consul sur AKS

Nous allons commencer par télécharger la version `v0.10.0` du chart Helm Consul. Cette version du chart inclut la version `1.6.0` de Consul.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download](includes/servicemesh/consul/download-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [macOS - download](includes/servicemesh/consul/download-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download](includes/servicemesh/consul/download-powershell.md)]

::: zone-end

Utilisez Helm et le chart `consul-helm` téléchargé pour installer les composants Consul dans l’espace de noms `consul` de votre cluster AKS. 

> [!NOTE]
> **Options d’installation**
> 
> Nous utilisons les options suivantes dans le cadre de notre installation :
> - `connectInject.enabled=true` : permet d’injecter des proxys dans les pods
> - `client.enabled=true` : permet d’exécuter des clients Consul sur chaque nœud
> - `client.grpc=true` : active l’écouteur gRPC pour connectInject
> - `syncCatalog.enabled=true` : synchronise les services Kubernetes et Consul
>
> **Sélecteurs de nœud**
>
> Consul doit actuellement être planifié pour s’exécuter sur des nœuds Linux. Si vous avez des nœuds Windows Server dans votre cluster, vous devez veiller à ce que l’exécution des pods Consul soit planifiée uniquement sur des nœuds Linux. Nous allons utiliser des [sélecteurs de nœud][kubernetes-node-selectors] pour vérifier que les pods sont planifiés sur les nœuds appropriés.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Bash - install Istio components](includes/servicemesh/consul/install-components-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [Bash - install Istio components](includes/servicemesh/consul/install-components-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [PowerShell - install Istio components](includes/servicemesh/consul/install-components-powershell.md)]

::: zone-end

Le chart Helm `Consul` déploie plusieurs objets. Vous pouvez voir la liste à partir de la sortie de votre commande `helm install` ci-dessus. Le déploiement des composants Consul peut prendre environ 3 minutes, selon votre environnement de cluster.

À ce stade, vous avez déployé Consul sur votre cluster AKS. Pour nous assurer que nous avons un déploiement réussi de Consul, nous allons passer à la section suivante pour valider l’installation de Consul.

## <a name="validate-the-consul-installation"></a>Valider l’installation de Consul

Confirmez que les ressources ont été créées. Utilisez les commandes [kubectl get svc][kubectl-get] et [kubectl get pod][kubectl-get] pour interroger l’espace de noms `consul` où les composants Consul ont été installés par la commande `helm install` :

```console
kubectl get svc --namespace consul --output wide
kubectl get pod --namespace consul --output wide
```

L’exemple de sortie suivant montre les services et les pods (planifiés sur des nœuds Linux) qui doivent maintenant être en cours d’exécution :

```output
NAME                                 TYPE           CLUSTER-IP    EXTERNAL-IP             PORT(S)                                                                   AGE     SELECTOR
consul                               ExternalName   <none>        consul.service.consul   <none>                                                                    38s     <none>
consul-consul-connect-injector-svc   ClusterIP      10.0.98.102   <none>                  443/TCP                                                                   3m26s   app=consul,component=connect-injector,release=consul
consul-consul-dns                    ClusterIP      10.0.46.194   <none>                  53/TCP,53/UDP                                                             3m26s   app=consul,hasDNS=true,release=consul
consul-consul-server                 ClusterIP      None          <none>                  8500/TCP,8301/TCP,8301/UDP,8302/TCP,8302/UDP,8300/TCP,8600/TCP,8600/UDP   3m26s   app=consul,component=server,release=consul
consul-consul-ui                     ClusterIP      10.0.50.188   <none>                  80/TCP                                                                    3m26s   app=consul,component=server,release=consul

NAME                                                              READY   STATUS    RESTARTS   AGE    IP            NODE                            NOMINATED NODE   READINESS GATES
consul-consul-connect-injector-webhook-deployment-99f74fdbcr5zj   1/1     Running   0          3m9s   10.240.0.68   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-jbksc                                               1/1     Running   0          3m9s   10.240.0.44   aks-linux-92468653-vmss000001   <none>           <none>
consul-consul-jkwtq                                               1/1     Running   0          3m9s   10.240.0.70   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-server-0                                            1/1     Running   0          3m9s   10.240.0.91   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-server-1                                            1/1     Running   0          3m9s   10.240.0.38   aks-linux-92468653-vmss000001   <none>           <none>
consul-consul-server-2                                            1/1     Running   0          3m9s   10.240.0.10   aks-linux-92468653-vmss000000   <none>           <none>
consul-consul-sync-catalog-d846b79c-8ssr8                         1/1     Running   2          3m9s   10.240.0.94   aks-linux-92468653-vmss000002   <none>           <none>
consul-consul-tz2t5                                               1/1     Running   0          3m9s   10.240.0.12   aks-linux-92468653-vmss000000   <none>           <none>
```

Tous les pods doivent indiquer l’état `Running`. Si vos pods n’ont pas cet état, patientez une minute ou deux. Si des pods signalent un problème, utilisez la commande [kubectl describe pod][kubectl-describe] pour examiner leur sortie et leur état.

## <a name="accessing-the-consul-ui"></a>Accès à l’interface utilisateur Consul

L’interface utilisateur Consul installée précédemment permet de configurer Consul. L’interface utilisateur pour Consul n’est pas exposée publiquement via une adresse IP externe. Pour accéder à l’interface utilisateur Consul, utilisez la commande [kubectl port-forward][kubectl-port-forward]. Cette commande crée une connexion sécurisée entre votre machine cliente et le pod approprié dans votre cluster AKS.

```console
kubectl port-forward -n consul svc/consul-consul-ui 8080:80
```

Vous pouvez maintenant ouvrir un navigateur et le faire pointer vers `http://localhost:8080/ui` pour ouvrir l’interface utilisateur Consul. Les éléments suivants doivent s’afficher quand vous ouvrez l’interface utilisateur :

![Interface utilisateur Consul](./media/servicemesh/consul/consul-ui.png)

## <a name="uninstall-consul-from-aks"></a>Désinstaller Consul à partir d’AKS

> [!WARNING]
> La suppression de Consul à partir d’un système en cours d’exécution peut entraîner des problèmes de trafic entre vos services. Vérifiez que vous avez mis en place des dispositions pour que votre système continue de fonctionner correctement sans Consul avant de continuer.

### <a name="remove-consul-components-and-namespace"></a>Supprimer les composants et l’espace de noms Consul

Pour supprimer Consul de votre cluster AKS, utilisez les commandes suivantes. Les commandes `helm delete` suppriment le chart `consul`, et la commande `kubectl delete namespace` supprime l’espace de noms `consul`.

```console
helm delete --purge consul
kubectl delete namespace consul
```

## <a name="next-steps"></a>Étapes suivantes

Pour explorer davantage d’options de configuration et d’installation pour Consul, consultez les articles Consul officiels suivants :

- [Consul - Helm installation guide][consul-install-k8]
- [Consul - Helm installation options][consul-install-helm-options]

Pour suivre des scénarios supplémentaires, consultez :

- [Exemple d’application Consul][consul-app-example]
- [Architecture de référence Kubernetes Consul][consul-reference]
- [Passerelles de maillage Consul][consul-mesh-gateways]

<!-- LINKS - external -->
[Hashicorp]: https://hashicorp.com
[cosul-github]: https://github.com/hashicorp/consul
[helm]: https://helm.sh

[consul-docs-concepts]: https://www.consul.io/docs/platform/k8s/index.html
[consul-github]: https://github.com/hashicorp/consul
[consul-github-releases]: https://github.com/hashicorp/consul/releases
[consul-release-notes]: https://github.com/hashicorp/consul/blob/master/CHANGELOG.md
[consul-install-download]: https://www.consul.io/downloads.html
[consul-install-k8]: https://learn.hashicorp.com/consul/kubernetes/kubernetes-deployment-guide
[consul-install-helm-options]: https://www.consul.io/docs/platform/k8s/helm.html#configuration-values-
[consul-mesh-gateways]: https://learn.hashicorp.com/consul/kubernetes/mesh-gateways
[consul-reference]: https://learn.hashicorp.com/consul/kubernetes/kubernetes-reference
[consul-app-example]: https://learn.hashicorp.com/consul?track=gs-consul-service-mesh#gs-consul-service-mesh
[install-wsl]: /windows/wsl/install-win10

[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-port-forward]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#port-forward
[kubernetes-node-selectors]: ./concepts-clusters-workloads.md#node-selectors

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
[consul-scenario-mtls]: ./consul-mtls.md
[helm-install]: ./kubernetes-helm.md
