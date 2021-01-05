---
title: Utiliser un contrôleur d’entrée NGINX personnalisé et configurer HTTPS
services: azure-dev-spaces
ms.date: 12/10/2019
ms.topic: conceptual
description: Découvrez comment configurer Azure Dev Spaces pour utiliser un contrôleur d’entrée NGINX personnalisé et configurer HTTPS à l’aide de ce contrôleur d’entrée.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs, Helm, service Mesh, routage du service Mesh, kubectl, k8s
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 873057e88809c1aaa8047ac02129d83ca8e9a478
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96608552"
---
# <a name="use-a-custom-nginx-ingress-controller-and-configure-https"></a>Utiliser un contrôleur d’entrée NGINX personnalisé et configurer HTTPS

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Cet article explique comment configurer Azure Dev Spaces pour utiliser un contrôleur d’entrée NGINX personnalisé. Cet article vous montre également comment configurer ce contrôleur d’entrée personnalisé pour utiliser le protocole HTTPS.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’en avez pas, vous pouvez créer un [compte gratuit][azure-account-create].
* [Azure CLI][az-cli].
* Cluster AKS (Azure Kubernetes Service) avec Azure Dev Spaces activé.
* [kubectl][kubectl] installé.
* [Helm 3 installé][helm-installed].
* [Un domaine personnalisé][custom-domain] avec une [zone DNS][dns-zone].  Cet article suppose que le domaine personnalisé et la zone DNS se trouvent dans le même groupe de ressources que votre cluster AKS, mais il est possible d’utiliser un domaine personnalisé et une zone DNS dans un groupe de ressources différent.

## <a name="configure-a-custom-nginx-ingress-controller"></a>Configurer un contrôleur d’entrée NGINX personnalisé

Connectez-vous à votre cluster à l’aide de [kubectl][kubectl], le client de ligne de commande Kubernetes. Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials][az-aks-get-credentials]. Cette commande télécharge les informations d’identification et configure l’interface CLI Kubernetes pour les utiliser.

```azurecli
az aks get-credentials --resource-group myResourceGroup --name myAKS
```

Pour vérifier la connexion à votre cluster, utilisez la commande [kubectl get][kubectl-get] pour retourner une liste des nœuds du cluster.

```console
kubectl get nodes
NAME                                STATUS   ROLES   AGE    VERSION
aks-nodepool1-12345678-vmssfedcba   Ready    agent   13m    v1.14.1
```

Ajoutez le [référentiel Helm stable officiel][helm-stable-repo], qui contient le graphique Helm du contrôleur d’entrée NGINX.

```console
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

Créez un espace de noms Kubernetes pour le contrôleur d’entrée NGINX et installez-le à l’aide de `helm`.

```console
kubectl create ns nginx
helm install nginx stable/nginx-ingress --namespace nginx --version 1.27.0
```

> [!NOTE]
> L’exemple ci-dessus crée un point de terminaison public pour votre contrôleur d’entrée. Si vous devez plutôt utiliser un point de terminaison privé pour votre contrôleur d’entrée, ajoutez le paramètre *--set controller.service.annotations."service\\.beta\\.kubernetes\\.io/azure-load-balancer-internal"=true* à la commande *helm install*. Par exemple :
> ```console
> helm install nginx stable/nginx-ingress --namespace nginx --set controller.service.annotations."service\.beta\.kubernetes\.io/azure-load-balancer-internal"=true --version 1.27.0
> ```
> Ce point de terminaison privé est exposé au sein du réseau virtuel dans lequel votre cluster AKS est déployé.

Récupérez l’adresse IP du service de contrôleur d’entrée NGINX à l’aide de [kubectl get][kubectl-get].

```console
kubectl get svc -n nginx --watch
```

L’exemple de sortie montre les adresses IP de tous les services de l’espace de noms *Nginx*.

```console
NAME                                  TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
nginx-nginx-ingress-controller        LoadBalancer   10.0.19.39     <pending>        80:31314/TCP,443:30521/TCP   10s
nginx-nginx-ingress-default-backend   ClusterIP      10.0.210.231   <none>           80/TCP                       10s
...
nginx-nginx-ingress-controller        LoadBalancer   10.0.19.39     MY_EXTERNAL_IP   80:31314/TCP,443:30521/TCP   26s
```

Ajoutez un enregistrement *A* à votre zone DNS avec l’adresse IP externe du service NGINX à l’aide de la commande [az network dns record-set a add-record][az-network-dns-record-set-a-add-record].

```azurecli
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name *.nginx \
    --ipv4-address MY_EXTERNAL_IP
```

L’exemple ci-dessus ajoute un enregistrement *A* à la zone *MY_CUSTOM_DOMAIN*.

Dans cet article, vous utilisez l’[exemple d’application de partage de vélos Azure Dev Spaces](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp) pour illustrer l’utilisation d’Azure Dev Spaces. Clonez l’application à partir de GitHub, puis accédez à son répertoire :

```cmd
git clone https://github.com/Azure/dev-spaces
cd dev-spaces/samples/BikeSharingApp/charts
```

Ouvrez [values.yaml][values-yaml] et effectuez les mises à jour suivantes :
* Remplacez toutes les instances de *<REPLACE_ME_WITH_HOST_SUFFIX>* par *nginx.MY_CUSTOM_DOMAIN* en utilisant votre domaine pour *MY_CUSTOM_DOMAIN*. 
* Remplacez *kubernetes.io/ingress.class: traefik-azds  # Dev Spaces-specific* par *kubernetes.io/ingress.class: nginx  # Custom Ingress*. 

Voici un exemple de fichier `values.yaml` mis à jour :

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
    hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
    hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
```

Enregistrer vos modifications et fermez le fichier.

Créez l’espace *dev* avec votre exemple d’application à l’aide de `azds space select`.

```console
azds space select -n dev -y
```

Déployez l’exemple d’application avec `helm install`.

```console
helm install bikesharingsampleapp . --dependency-update --namespace dev --atomic
```

L’exemple ci-dessus déploie l’exemple d'application dans l’espace de noms *dev*.

Affichez les URL pour accéder à l’exemple d’application à l’aide de `azds list-uris`.

```console
azds list-uris
```

La sortie ci-dessous montre les exemples d’URL de `azds list-uris`.

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/  Available
http://dev.gateway.nginx.MY_CUSTOM_DOMAIN/         Available
```

Accédez au service *bikesharingweb* en ouvrant l’URL publique à partir de la commande `azds list-uris`. Dans l’exemple ci-dessus, l’URL publique pour le service *bikesharingweb* est `http://dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/`.

> [!NOTE]
> Si vous voyez une page d’erreur au lieu du service *bikesharingweb*, vérifiez que vous avez mis à jour **tant** l’annotation *kubernetes.io/ingress.class* que l’hôte dans le fichier *values.yaml*.

Utilisez la commande `azds space select` pour créer un espace enfant sous *dev* et répertoriez les URL pour accéder à l’espace de développement enfant.

```console
azds space select -n dev/azureuser1 -y
azds list-uris
```

La sortie ci-dessous montre les exemples d’URL de `azds list-uris` pour accéder à l'exemple d’application dans l’espace de développement enfant *azureuser1*.

```console
Uri                                                  Status
---------------------------------------------------  ---------
http://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/  Available
http://azureuser1.s.dev.gateway.nginx.MY_CUSTOM_DOMAIN/         Available
```

Accédez au service *bikesharingweb* pour l’espace de développement enfant *azureuser1* en ouvrant l’URL publique à partir de la commande `azds list-uris`. Dans l’exemple ci-dessus, l’URL publique pour le service *bikesharingweb* dans l’espace de développement enfant *azureuser1* est `http://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/`.

## <a name="configure-the-nginx-ingress-controller-to-use-https"></a>Configurer le contrôleur d’entrée NGINX pour utiliser HTTPS

Utilisez [cert-manager][cert-manager] pour automatiser la gestion du certificat TLS lors de la configuration de votre contrôleur d’entrée NGINX pour utiliser le protocole HTTPS. Utilisez `helm` pour installer le graphique *certmanager*.

```console
kubectl apply --validate=false -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.12/deploy/manifests/00-crds.yaml --namespace nginx
kubectl label namespace nginx certmanager.k8s.io/disable-validation=true
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager --namespace nginx --version v0.12.0 jetstack/cert-manager --set ingressShim.defaultIssuerName=letsencrypt --set ingressShim.defaultIssuerKind=ClusterIssuer
```

Créez un fichier `letsencrypt-clusterissuer.yaml` et mettez à jour le champ de l’e-mail avec votre adresse e-mail.

```yaml
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: MY_EMAIL_ADDRESS
    privateKeySecretRef:
      name: letsencrypt
    solvers:
      - http01:
          ingress:
            class: nginx
```

> [!NOTE]
> À des fins de test, il existe également un [serveur intermédiaire][letsencrypt-staging-issuer] vous pouvez utiliser pour votre *ClusterIssuer*.

Utilisez `kubectl` pour appliquer `letsencrypt-clusterissuer.yaml`.

```console
kubectl apply -f letsencrypt-clusterissuer.yaml --namespace nginx
```

Mettez à jour [values.yaml][values-yaml] pour inclure les détails relatifs à l’utilisation de *cert-manager* et HTTPS. Voici un exemple de fichier `values.yaml` mis à jour :

```yaml
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

bikesharingweb:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN
      secretName: dev-bikesharingweb-secret

gateway:
  ingress:
    annotations:
      kubernetes.io/ingress.class: nginx  # Custom Ingress
      cert-manager.io/cluster-issuer: letsencrypt
    hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN  # Assumes deployment to the 'dev' space
    tls:
    - hosts:
      - dev.gateway.nginx.MY_CUSTOM_DOMAIN
      secretName: dev-gateway-secret
```

Mettez à niveau l’exemple d’application avec `helm` :

```console
helm upgrade bikesharingsampleapp . --namespace dev --atomic
```

Accédez à l’exemple d’application dans l’espace enfant *dev/azureuser1* et remarquez que vous êtes redirigé pour utiliser le protocole HTTPS. Notez également que la page se charge, mais que le navigateur affiche des erreurs. L’ouverture de la console du navigateur affiche l’erreur liée à une page HTTPS tentant de charger des ressources HTTP. Par exemple :

```console
Mixed Content: The page at 'https://azureuser1.s.dev.bikesharingweb.nginx.MY_CUSTOM_DOMAIN/devsignin' was loaded over HTTPS, but requested an insecure resource 'http://azureuser1.s.dev.gateway.nginx.MY_CUSTOM_DOMAIN/api/user/allUsers'. This request has been blocked; the content must be served over HTTPS.
```

Pour corriger cette erreur, mettez à jour [BikeSharingWeb/azds.yaml][azds-yaml] similaire à ce qui suit :

```yaml
...
    ingress:
      annotations:
        kubernetes.io/ingress.class: nginx
        cert-manager.io/cluster-issuer: letsencrypt
      hosts:
      # This expands to [space.s.][rootSpace.]bikesharingweb.<random suffix>.<region>.azds.io
      - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.nginx.MY_CUSTOM_DOMAIN
      tls:
      - hosts:
        - $(spacePrefix)$(rootSpacePrefix)bikesharingweb.nginx.MY_CUSTOM_DOMAIN
        secretName: dev-bikesharingweb-secret
...
```

Mettez à jour [BikeSharingWeb/package.json][package-json] avec une dépendance pour le package *url*.

```json
{
...
    "react-responsive": "^6.0.1",
    "universal-cookie": "^3.0.7",
    "url": "0.11.0"
  },
...
```

Mettez à jour la méthode *getApiHostAsync* dans [BikeSharingWeb/lib/helpers.js][helpers-js] pour utiliser HTTPS :

```javascript
...
    getApiHostAsync: async function() {
        const apiRequest = await fetch('/api/host');
        const data = await apiRequest.json();
        
        var urlapi = require('url');
        var url = urlapi.parse(data.apiHost);

        console.log('apiHost: ' + "https://"+url.host);
        return "https://"+url.host;
    },
...
```

Accédez au répertoire `BikeSharingWeb` et utilisez `azds up` pour exécuter votre service *BikeSharingWeb* mis à jour.

```console
cd ../BikeSharingWeb/
azds up
```

Accédez à l’exemple d’application dans l’espace enfant *dev/azureuser1* et remarquez que vous êtes redirigé pour utiliser le protocole HTTPS sans erreur.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage plus sur le fonctionnement d’Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Fonctionnement d’Azure Dev Spaces](../how-dev-spaces-works.md)


[az-cli]: /cli/azure/install-azure-cli?view=azure-cli-latest
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[az-network-dns-record-set-a-add-record]: /cli/azure/network/dns/record-set/a?view=azure-cli-latest#az-network-dns-record-set-a-add-record
[custom-domain]: ../../app-service/manage-custom-dns-buy-domain.md#buy-an-app-service-domain
[dns-zone]: ../../dns/dns-getstarted-cli.md
[azds-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/azds.yaml
[azure-account-create]: https://azure.microsoft.com/free
[cert-manager]: https://cert-manager.io/
[helm-installed]: https://helm.sh/docs/intro/install/
[helm-stable-repo]: https://helm.sh/docs/intro/quickstart/#initialize-a-helm-chart-repository
[helpers-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/lib/helpers.js#L7
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[letsencrypt-staging-issuer]: https://cert-manager.io/docs/configuration/acme/#creating-a-basic-acme-issuer
[package-json]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/BikeSharingWeb/package.json
[values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml
