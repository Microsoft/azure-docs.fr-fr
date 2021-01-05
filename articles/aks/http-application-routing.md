---
title: Module complémentaire de routage des applications HTTP Azure Kubernetes Service (AKS)
description: Utilisez le module complémentaire de routage des applications HTTP pour accéder aux applications déployées sur Azure Kubernetes Service (AKS).
services: container-service
author: lachie83
ms.topic: article
ms.date: 07/20/2020
ms.author: laevenso
ms.openlocfilehash: bbedb20d9e5c75fd49c08950bbf5d459130206ce
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93125867"
---
# <a name="http-application-routing"></a>Routage d’applications HTTP

La solution de routage des applications HTTP permet d’accéder facilement aux applications déployées sur votre cluster AKS (Azure Kubernetes Service). Quand elle est activée, la solution configure un [contrôleur d’entrée](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) sur votre cluster AKS. À mesure que les applications sont déployées, la solution crée également des noms DNS accessibles publiquement pour les points de terminaison d’application.

Lorsque ce module complémentaire est activé, il crée une zone DNS dans votre abonnement. Pour plus d’informations sur les coûts DNS, consultez [Tarification de DNS][dns-pricing].

> [!CAUTION]
> Le module complémentaire de routage des applications HTTP est conçu pour vous permettre de créer rapidement un contrôleur d’entrée et d’accéder à vos applications. Ce module complémentaire n’est pas actuellement conçu pour une utilisation dans un environnement de production et n’est pas recommandé pour une utilisation en production. Pour les déploiements d’entrée prêts pour la production qui incluent plusieurs réplicas et la prise en charge de TLS, consultez [Créer un contrôleur d’entrée HTTPS](./ingress-tls.md).

## <a name="http-routing-solution-overview"></a>Vue d’ensemble de la solution de routage HTTP

Le module complémentaire déploie deux composants : un [contrôleur d’entrée Kubernetes][ingress] et un contrôleur [DNS externe][external-dns].

- **Contrôleur d’entrée**  : le contrôleur d’entrée est exposé sur Internet à l’aide d’un service Kubernetes de type LoadBalancer. Le contrôleur d’entrée surveille et implémente des [ressources d’entrée Kubernetes][ingress-resource], ce qui crée des routes es points de terminaison d’application.
- **Contrôleur DNS externe**  : surveille les ressources d’entrée Kubernetes et crée des enregistrements DNS A dans la zone DNS spécifique au cluster.

## <a name="deploy-http-routing-cli"></a>Déployer le routage HTTP : Interface de ligne de commande

Le module complémentaire de routage des applications HTTP peut être activé via le l’interface CLI Azure lors du déploiement d’un cluster AKS. Pour ce faire, utilisez la commande [az aks create][az-aks-create] avec l’argument `--enable-addons`.

```azurecli
az aks create --resource-group myResourceGroup --name myAKSCluster --enable-addons http_application_routing
```

> [!TIP]
> Si vous souhaitez activer plusieurs modules complémentaires, fournissez-les sous forme de liste séparée par des virgules. Par exemple, pour activer le routage et la surveillance d’applications HTTP, utilisez le format `--enable-addons http_application_routing,monitoring`.

Vous pouvez également activer le routage HTTP sur un cluster AKS existant en utilisant la commande [az aks enable-addons][az-aks-enable-addons]. Pour activer le routage HTTP sur un cluster existant, ajoutez le paramètre `--addons` et spécifiez *http_application_routing* comme indiqué dans l’exemple suivant :

```azurecli
az aks enable-addons --resource-group myResourceGroup --name myAKSCluster --addons http_application_routing
```

Une fois le cluster déployé ou mis à jour, utilisez la commande [az aks show][az-aks-show] pour récupérer le nom de la zone DNS.

```azurecli
az aks show --resource-group myResourceGroup --name myAKSCluster --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName -o table
```

Ce nom est nécessaire pour déployer des applications sur le cluster AKS et est illustré dans l’exemple de sortie suivant :

```console
9f9c1fe7-21a1-416d-99cd-3543bb92e4c3.eastus.aksapp.io
```

## <a name="deploy-http-routing-portal"></a>Déployer le routage HTTP : Portail

Le module complémentaire de routage des applications HTTP peut être activé via le portail Azure lors du déploiement d’un cluster AKS.

![Activer la fonctionnalité de routage HTTP](media/http-routing/create.png)

Une fois le cluster déployé, accédez au groupe de ressources AKS créé automatiquement, puis sélectionnez la zone DNS. Notez le nom de la zone DNS. Ce nom est nécessaire pour déployer des applications sur le cluster AKS.

![Obtenir le nom de zone DNS](media/http-routing/dns.png)

## <a name="connect-to-your-aks-cluster"></a>Se connecter à votre cluster AKS

Pour vous connecter au cluster Kubernetes à partir de votre ordinateur local, utilisez [kubectl][kubectl], le client de ligne de commande Kubernetes.

Si vous utilisez Azure Cloud Shell, `kubectl` est déjà installé. Vous pouvez également l’installer en local à l’aide de la commande [az aks install-cli][] :

```azurecli
az aks install-cli
```

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials][]. L’exemple suivant obtient les informations d’identification du cluster AKS nommé *MyAKSCluster* dans le groupe de ressources *MyResourceGroup*  :

```azurecli
az aks get-credentials --resource-group MyResourceGroup --name MyAKSCluster
```

## <a name="use-http-routing"></a>Utiliser le routage HTTP

La solution de routage des applications HTTP peut uniquement être déclenchée sur des ressources d’entrée annotées de la façon suivante :

```yaml
annotations:
  kubernetes.io/ingress.class: addon-http-application-routing
```

Créez un fichier nommé **samples-http-application-routing.yaml** , puis copiez-y le code YAML suivant. À la ligne 43, mettez à jour `<CLUSTER_SPECIFIC_DNS_ZONE>` avec le nom de zone DNS collecté à la précédente étape de cet article.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld
  template:
    metadata:
      labels:
        app: aks-helloworld
    spec:
      containers:
      - name: aks-helloworld
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "Welcome to Azure Kubernetes Service (AKS)"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld  
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: aks-helloworld
  annotations:
    kubernetes.io/ingress.class: addon-http-application-routing
spec:
  rules:
  - host: aks-helloworld.<CLUSTER_SPECIFIC_DNS_ZONE>
    http:
      paths:
      - backend:
          serviceName: aks-helloworld
          servicePort: 80
        path: /
```

Utilisez la commande [kubectl apply][kubectl-apply] pour créer les ressources.

```bash
kubectl apply -f samples-http-application-routing.yaml
```

L’exemple suivant illustre les ressources créées :

```bash
$ kubectl apply -f samples-http-application-routing.yaml

deployment.apps/aks-helloworld created
service/aks-helloworld created
ingress.networking.k8s.io/aks-helloworld created
```

Ouvrez un navigateur web pour *aks-helloworld.\<CLUSTER_SPECIFIC_DNS_ZONE\>* , par exemple *aks-helloworld.9f9c1fe7-21a1-416d-99cd-3543bb92e4c3.eastus.aksapp.io* , et vérifiez que l’application de démonstration s’affiche. L’application peut apparaître au bout de quelques minutes seulement.

## <a name="remove-http-routing"></a>Supprimer le routage HTTP

La solution de routage HTTP peut être supprimée à l’aide d’Azure CLI. Pour ce faire, exécutez la commande suivante, en substituant votre cluster AKS et le nom du groupe de ressources.

```azurecli
az aks disable-addons --addons http_application_routing --name myAKSCluster --resource-group myResourceGroup --no-wait
```

Quand le module complémentaire de routage d’application HTTP est désactivé, certaines ressources Kubernetes peuvent rester dans le cluster. Ces ressources incluent *configMaps* et *secrets* , et sont créées dans l’espace de noms *kube-system*. Pour maintenir un cluster propre, vous voulez peut-être supprimer ces ressources.

Recherchez les ressources *addon-http-application-routing* à l’aide des commandes [kubectl get][kubectl-get] suivantes :

```console
kubectl get deployments --namespace kube-system
kubectl get services --namespace kube-system
kubectl get configmaps --namespace kube-system
kubectl get secrets --namespace kube-system
```

L’exemple de sortie suivant montre configMaps qui doit être supprimée :

```
$ kubectl get configmaps --namespace kube-system

NAMESPACE     NAME                                                       DATA   AGE
kube-system   addon-http-application-routing-nginx-configuration         0      9m7s
kube-system   addon-http-application-routing-tcp-services                0      9m7s
kube-system   addon-http-application-routing-udp-services                0      9m7s
```

Pour supprimer des ressources, utilisez la commande [kubectl delete][kubectl-delete]. Spécifiez le type de ressource, le nom de la ressource et l’espace de noms. L’exemple suivant supprime l’une des ressources configMaps précédentes :

```console
kubectl delete configmaps addon-http-application-routing-nginx-configuration --namespace kube-system
```

Répétez l’étape `kubectl delete` précédente pour toutes les ressources *addon-http-application-routing* qui sont restées dans votre cluster.

## <a name="troubleshoot"></a>Dépanner

Utilisez la commande [kubectl logs][kubectl-logs] pour afficher les journaux d’application de l’application DNS externe. Les journaux d’activité doivent confirmer que des enregistrements DNS A et TXT ont bien été créés.

```
$ kubectl logs -f deploy/addon-http-application-routing-external-dns -n kube-system

time="2018-04-26T20:36:19Z" level=info msg="Updating A record named 'aks-helloworld' to '52.242.28.189' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
time="2018-04-26T20:36:21Z" level=info msg="Updating TXT record named 'aks-helloworld' to '"heritage=external-dns,external-dns/owner=default"' for Azure DNS zone '471756a6-e744-4aa0-aa01-89c4d162a7a7.canadaeast.aksapp.io'."
```

Ces enregistrements peuvent également être consultés sur la ressource de zone DNS dans le portail Azure.

![Obtenir les enregistrements DNS](media/http-routing/clippy.png)

Utilisez la commande [kubectl logs][kubectl-logs] pour afficher les journaux d’application du contrôleur d’entrée Nginx. Les journaux d’activité doivent confirmer la création (`CREATE`) d’une ressource d’entrée et le rechargement du contrôleur. Toutes les activités HTTP sont journalisées.

```bash
$ kubectl logs -f deploy/addon-http-application-routing-nginx-ingress-controller -n kube-system

-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.13.0
  Build:      git-4bc943a
  Repository: https://github.com/kubernetes/ingress-nginx
-------------------------------------------------------------------------------

I0426 20:30:12.212936       9 flags.go:162] Watching for ingress class: addon-http-application-routing
W0426 20:30:12.213041       9 flags.go:165] only Ingress with class "addon-http-application-routing" will be processed by this ingress controller
W0426 20:30:12.213505       9 client_config.go:533] Neither --kubeconfig nor --master was specified.  Using the inClusterConfig.  This might not work.
I0426 20:30:12.213752       9 main.go:181] Creating API client for https://10.0.0.1:443
I0426 20:30:12.287928       9 main.go:225] Running in Kubernetes Cluster version v1.8 (v1.8.11) - git (clean) commit 1df6a8381669a6c753f79cb31ca2e3d57ee7c8a3 - platform linux/amd64
I0426 20:30:12.290988       9 main.go:84] validated kube-system/addon-http-application-routing-default-http-backend as the default backend
I0426 20:30:12.294314       9 main.go:105] service kube-system/addon-http-application-routing-nginx-ingress validated as source of Ingress status
I0426 20:30:12.426443       9 stat_collector.go:77] starting new nginx stats collector for Ingress controller running in namespace  (class addon-http-application-routing)
I0426 20:30:12.426509       9 stat_collector.go:78] collector extracting information from port 18080
I0426 20:30:12.448779       9 nginx.go:281] starting Ingress controller
I0426 20:30:12.463585       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-nginx-configuration", UID:"2588536c-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"559", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-nginx-configuration
I0426 20:30:12.466945       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-tcp-services", UID:"258ca065-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"561", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-tcp-services
I0426 20:30:12.467053       9 event.go:218] Event(v1.ObjectReference{Kind:"ConfigMap", Namespace:"kube-system", Name:"addon-http-application-routing-udp-services", UID:"259023bc-4990-11e8-a5e1-0a58ac1f0ef2", APIVersion:"v1", ResourceVersion:"562", FieldPath:""}): type: 'Normal' reason: 'CREATE' ConfigMap kube-system/addon-http-application-routing-udp-services
I0426 20:30:13.649195       9 nginx.go:302] starting NGINX process...
I0426 20:30:13.649347       9 leaderelection.go:175] attempting to acquire leader lease  kube-system/ingress-controller-leader-addon-http-application-routing...
I0426 20:30:13.649776       9 controller.go:170] backend reload required
I0426 20:30:13.649800       9 stat_collector.go:34] changing prometheus collector from  to default
I0426 20:30:13.662191       9 leaderelection.go:184] successfully acquired lease kube-system/ingress-controller-leader-addon-http-application-routing
I0426 20:30:13.662292       9 status.go:196] new leader elected: addon-http-application-routing-nginx-ingress-controller-5cxntd6
I0426 20:30:13.763362       9 controller.go:179] ingress backend successfully reloaded...
I0426 21:51:55.249327       9 event.go:218] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"default", Name:"aks-helloworld", UID:"092c9599-499c-11e8-a5e1-0a58ac1f0ef2", APIVersion:"extensions", ResourceVersion:"7346", FieldPath:""}): type: 'Normal' reason: 'CREATE' Ingress default/aks-helloworld
W0426 21:51:57.908771       9 controller.go:775] service default/aks-helloworld does not have any active endpoints
I0426 21:51:57.908951       9 controller.go:170] backend reload required
I0426 21:51:58.042932       9 controller.go:179] ingress backend successfully reloaded...
167.220.24.46 - [167.220.24.46] - - [26/Apr/2018:21:53:20 +0000] "GET / HTTP/1.1" 200 234 "" "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0)" 197 0.001 [default-aks-helloworld-80] 10.244.0.13:8080 234 0.004 200
```

## <a name="clean-up"></a>Nettoyer

Supprimez les objets Kubernetes associés que vous avez créés dans cet article avec `kubectl delete`.

```bash
kubectl delete -f samples-http-application-routing.yaml
```

L’exemple de sortie montre que les objets Kubernetes ont été supprimés.

```bash
$ kubectl delete -f samples-http-application-routing.yaml

deployment "aks-helloworld" deleted
service "aks-helloworld" deleted
ingress "aks-helloworld" deleted
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’installation d’un contrôleur d’entrée sécurisé HTTPS dans AKS, consultez [Entrée HTTPS sur Azure Kubernetes Service (AKS)][ingress-https].

<!-- LINKS - internal -->
[az-aks-create]: /cli/azure/aks?view=azure-cli-latest#az-aks-create
[az-aks-show]: /cli/azure/aks?view=azure-cli-latest#az-aks-show
[ingress-https]: ./ingress-tls.md
[az-aks-enable-addons]: /cli/azure/aks#az-aks-enable-addons
[az aks install-cli]: /cli/azure/aks#az-aks-install-cli
[az aks get-credentials]: /cli/azure/aks#az-aks-get-credentials

<!-- LINKS - external -->
[dns-pricing]: https://azure.microsoft.com/pricing/details/dns/
[external-dns]: https://github.com/kubernetes-incubator/external-dns
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[ingress]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[ingress-resource]: https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource
