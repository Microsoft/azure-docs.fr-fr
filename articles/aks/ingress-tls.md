---
title: Créer une entrée avec certification TLS automatique
titleSuffix: Azure Kubernetes Service
description: Découvrez comment installer et configurer un contrôleur d’entrée NGINX qui utilise Let’s Encrypt pour la génération automatique de certificats TLS dans un cluster Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/17/2020
ms.openlocfilehash: 88e2bdc1b516e55fb630b2fd31ff6a2977d57bfe
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607906"
---
# <a name="create-an-https-ingress-controller-on-azure-kubernetes-service-aks"></a>Créer un contrôleur d’entrée HTTPS dans Azure Kubernetes Service (AKS)

Un contrôleur d’entrée est un logiciel qui fournit un proxy inversé, un routage du trafic configurable et un arrêt TLS pour les services Kubernetes. Des ressources d’entrée Kubernetes sont utilisées pour configurer les règles d’entrée et les itinéraires des services Kubernetes individuels. En utilisant un contrôleur d’entrée et des règles d’entrée, une seule adresse IP peut être utilisée pour router le trafic vers plusieurs services dans un cluster Kubernetes.

Cet article montre comment déployer le [contrôleur d’entrée NGINX][nginx-ingress] dans un cluster Azure Kubernetes Service (AKS). Le projet [cert-manager][cert-manager] est utilisé pour générer et configurer automatiquement les certificats [Let’s Encrypt][lets-encrypt]. Enfin, plusieurs applications sont exécutées dans le cluster AKS, chacune étant accessible via une seule adresse IP.

Vous pouvez également :

- [Créer un contrôleur d’entrée dans Azure Kubernetes Service (AKS)][aks-ingress-basic]
- [Routage d’applications HTTP][aks-http-app-routing]
- [Create an ingress controller to an internal virtual network in Azure Kubernetes Service (AKS)][aks-ingress-internal] (Créer un contrôleur d’entrée pour un réseau virtuel interne dans Azure Kubernetes Service (AKS))
- [Créer un contrôleur d’entrée qui utilise vos propres certificats TLS][aks-ingress-own-tls]
- [Créer un contrôleur d’entrée qui utilise Let’s Encrypt pour générer automatiquement des certificats TLS avec une adresse IP publique statique][aks-ingress-static-tls]

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Cet article suppose également que vous disposez d’[un domaine personnalisé][custom-domain] avec une [zone DNS][dns-zone] dans le même groupe de ressources que votre cluster AKS.

Cet article utilise [Helm 3][helm] pour installer le contrôleur d’entrée NGINX et cert-manager. Assurez-vous que vous utilisez la version la plus récente de Helm et que vous avez accès aux référentiels Helm *ingress-nginx* et *jetstack*. Pour des instructions de mise à niveau, consultez la [documentation d’installation de Helm][helm-install]. Pour plus d’informations sur la configuration et l’utilisation de Helm, consultez [Installer des applications avec Helm dans Azure Kubernetes Service (AKS)][use-helm].

Pour les besoins de cet article, vous devez également exécuter Azure CLI version 2.0.64 ou ultérieure. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][azure-cli-install].

## <a name="create-an-ingress-controller"></a>Créer un contrôleur d’entrée

Pour créer le contrôleur d’entrée, utilisez la commande `helm` pour installer *nginx-ingress*. Pour renforcer la redondance, deux réplicas des contrôleurs d’entrée NGINX sont déployés avec le paramètre `--set controller.replicaCount`. Pour tirer pleinement parti de l’exécution de réplicas des contrôleurs d’entrée, vérifiez que votre cluster AKS comprend plusieurs nœuds.

Le contrôleur d’entrée doit également être planifié sur un nœud Linux. Les nœuds Windows Server ne doivent pas exécuter le contrôleur d’entrée. Un sélecteur de nœud est spécifié en utilisant le paramètre `--set nodeSelector` pour que le planificateur Kubernetes exécute le contrôleur d’entrée NGINX sur un nœud Linux.

> [!TIP]
> L’exemple suivant crée un espace de noms Kubernetes pour les ressources d’entrée *ingress-basic*. Spécifiez un espace de noms de votre propre environnement, si besoin.

> [!TIP]
> Si vous souhaitez activer la [préservation de l’adresse IP source du client][client-source-ip] pour les requêtes aux conteneurs de votre cluster, ajoutez `--set controller.service.externalTrafficPolicy=Local` à la commande d’installation Helm. L’IP source du client est stockée dans l’en-tête de la requête sous *X-Forwarded-For*. Lors de l’utilisation d’un contrôleur d’entrée pour lequel la conservation de l’adresse IP source du client est activée, un transfert direct TLS ne fonctionne pas.

```console
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Use Helm to deploy an NGINX ingress controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```

Pendant l’installation, une adresse IP publique Azure est créée pour le contrôleur d’entrée. Cette adresse IP publique est statique pour la durée de vie du contrôleur d’entrée. Si vous supprimez le contrôleur d’entrée, l’attribution d’adresse IP publique est perdue. Si vous créez ensuite un contrôleur d’entrée supplémentaires, une nouvelle adresse IP publique est attribuée. Si vous souhaitez conserver l’utilisation de l’adresse IP publique, vous pouvez au lieu de cela [créer un contrôleur d’entrée avec une adresse IP publique statique][aks-ingress-static-tls].

Pour obtenir l’adresse IP publique, utilisez la commande `kubectl get service`. L’affectation de l’adresse IP au service peut prendre quelques minutes.

```
$ kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller

NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.74.133   EXTERNAL_IP     80:32486/TCP,443:30953/TCP   44s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

Aucune règle d’entrée n’a encore été créée. Si vous accédez à l’adresse IP publique, la page 404 par défaut du contrôleur d’entrée NGINX s’affiche.

## <a name="add-an-a-record-to-your-dns-zone"></a>Ajouter un enregistrement A à votre zone DNS

Ajoutez un enregistrement *A* à votre zone DNS avec l’adresse IP externe du service NGINX à l’aide de la commande [az network dns record-set a add-record][az-network-dns-record-set-a-add-record].

```console
az network dns record-set a add-record \
    --resource-group myResourceGroup \
    --zone-name MY_CUSTOM_DOMAIN \
    --record-set-name * \
    --ipv4-address MY_EXTERNAL_IP
```

> [!NOTE]
> Si vous le souhaitez, vous pouvez configurer un nom de domaine complet pour l’adresse IP du contrôleur d’entrée au lieu d’un domaine personnalisé. Notez que cet exemple s'applique à un interpréteur de commandes Bash.
> 
> ```azurecli-interactive
> # Public IP address of your ingress controller
> IP="MY_EXTERNAL_IP"
> 
> # Name to associate with public IP address
> DNSNAME="demo-aks-ingress"
> 
> # Get the resource-id of the public ip
> PUBLICIPID=$(az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '$IP')].[id]" --output tsv)
> 
> # Update public ip address with DNS name
> az network public-ip update --ids $PUBLICIPID --dns-name $DNSNAME
> 
> # Display the FQDN
> az network public-ip show --ids $PUBLICIPID --query "[dnsSettings.fqdn]" --output tsv
> ```

## <a name="install-cert-manager"></a>Installer cert-manager

Le contrôleur d’entrée NGINX prend en charge l’arrêt TLS. Il existe plusieurs façons de récupérer et de configurer des certificats pour le protocole HTTPS. Cet article montre comment utiliser [cert-manager][cert-manager], qui permet de générer et de gérer automatiquement des certificats [Let’s Encrypt][lets-encrypt].

Pour installer le contrôleur cert-manager :

```console
# Label the ingress-basic namespace to disable resource validation
kubectl label namespace ingress-basic cert-manager.io/disable-validation=true

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install \
  cert-manager \
  --namespace ingress-basic \
  --version v0.16.1 \
  --set installCRDs=true \
  --set nodeSelector."beta\.kubernetes\.io/os"=linux \
  jetstack/cert-manager
```

Pour plus d’informations sur la configuration cert-manager, voir le [projet cert-manager][cert-manager].

## <a name="create-a-ca-cluster-issuer"></a>Créer un émetteur de cluster d’autorité de certification

Cert-manager a besoin d’une ressource [Issuer][cert-manager-issuer] ou [ClusterIssuer][cert-manager-cluster-issuer] pour pouvoir émettre des certificats. Ces ressources Kubernetes sont identiques du point de vue des fonctionnalités, mais `Issuer` s’applique à un seul espace de noms, tandis que `ClusterIssuer` couvre tous les espaces de noms. Pour plus d’informations, voir la documentation de [l’émetteur cert-manager][cert-manager-issuer].

Créez un émetteur de cluster, tel que `cluster-issuer.yaml`, avec l’exemple de manifeste suivant. Remplacez l’adresse e-mail par une adresse valide de votre organisation :

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
          podTemplate:
            spec:
              nodeSelector:
                "kubernetes.io/os": linux
```

Pour créer l’émetteur, utilisez la commande `kubectl apply`.

```console
kubectl apply -f cluster-issuer.yaml
```

## <a name="run-demo-applications"></a>Exécuter des applications de démonstration

Un contrôleur d’entrée et une solution de gestion de certificats ont été configurés. Maintenant, nous allons exécuter deux applications de démonstration dans le cluster AKS. Pour cet exemple, Helm est utilisé pour déployer deux instances d’une simple application *Hello world*.

Pour voir le contrôleur d’entrée à l’œuvre, exécutons deux applications de démonstration dans votre cluster AKS. Pour cet exemple, vous utilisez `kubectl apply` pour déployer deux instances d’une simple application *Hello world*.

Créez un fichier *aks-helloworld-one.yaml* et copiez-le dans l’exemple de YAML suivant :

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-one
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-one
  template:
    metadata:
      labels:
        app: aks-helloworld-one
    spec:
      containers:
      - name: aks-helloworld-one
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
  name: aks-helloworld-one
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-one
```

Créez un fichier *aks-helloworld-two.yaml* et copiez-le dans l’exemple de YAML suivant :

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aks-helloworld-two
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aks-helloworld-two
  template:
    metadata:
      labels:
        app: aks-helloworld-two
    spec:
      containers:
      - name: aks-helloworld-two
        image: mcr.microsoft.com/azuredocs/aks-helloworld:v1
        ports:
        - containerPort: 80
        env:
        - name: TITLE
          value: "AKS Ingress Demo"
---
apiVersion: v1
kind: Service
metadata:
  name: aks-helloworld-two
spec:
  type: ClusterIP
  ports:
  - port: 80
  selector:
    app: aks-helloworld-two
```

Exécutez les deux applications de démonstration avec `kubectl apply` :

```console
kubectl apply -f aks-helloworld-one.yaml --namespace ingress-basic
kubectl apply -f aks-helloworld-two.yaml --namespace ingress-basic
```

## <a name="create-an-ingress-route"></a>Créer un itinéraire d’entrée

Les deux applications sont maintenant en cours d’exécution sur votre cluster Kubernetes. Toutefois, elles sont configurées avec un service de type `ClusterIP` et ne sont pas accessibles à partir d’Internet. Pour les rendre disponibles publiquement, créez une ressource d’entrée Kubernetes. La ressource d’entrée configure les règles qui acheminent le trafic vers l’une des deux applications.

Dans l’exemple suivant, le trafic vers l’adresse *hello-world-ingress.MY_CUSTOM_DOMAIN* est routé vers le service *aks-helloworld*. Le trafic vers l’adresse *hello-world-ingress.MY_CUSTOM_DOMAIN/hello-world-two* est routé vers le service *aks-helloworld-two*. Le trafic vers *hello-world-ingress.MY_CUSTOM_DOMAIN/static* est routé vers le service nommé *aks-helloworld* pour les ressources statiques.

> [!NOTE]
> Si vous avez configuré un nom de domaine complet pour l’adresse IP du contrôleur d’entrée au lieu d’un domaine personnalisé, utilisez le nom de domaine complet au lieu de *hello-world-ingress.MY_CUSTOM_DOMAIN*. Par exemple, si votre nom de domaine complet est *demo-aks-ingress.eastus.cloudapp.azure.com*, remplacez *hello-world-ingress.MY_CUSTOM_DOMAIN* par *demo-aks-ingress.eastus.cloudapp.azure.com* dans `hello-world-ingress.yaml`.

Créez un fichier nommé `hello-world-ingress.yaml` en utilisant l’exemple de code YAML ci-dessous. Mettez à jour les valeurs de *hosts* et de *host* avec le nom DNS que vous avez créé à l’étape précédente.

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/use-regex: "true"
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - hello-world-ingress.MY_CUSTOM_DOMAIN
    secretName: tls-secret
  rules:
  - host: hello-world-ingress.MY_CUSTOM_DOMAIN
    http:
      paths:
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /hello-world-one(/|$)(.*)
      - backend:
          serviceName: aks-helloworld-two
          servicePort: 80
        path: /hello-world-two(/|$)(.*)
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /(.*)
---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress-static
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /static/$2
    nginx.ingress.kubernetes.io/use-regex: "true"
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - hello-world-ingress.MY_CUSTOM_DOMAIN
    secretName: tls-secret
  rules:
  - host: hello-world-ingress.MY_CUSTOM_DOMAIN
    http:
      paths:
      - backend:
          serviceName: aks-helloworld-one
          servicePort: 80
        path: /static(/|$)(.*)
```

Créez la ressource d’entrée avec la commande `kubectl apply`.

```console
kubectl apply -f hello-world-ingress.yaml --namespace ingress-basic
```

## <a name="verify-a-certificate-object-has-been-created"></a>Vérifier qu’un objet de certificat a été créé

Il faut ensuite créer une ressource de certificat. Elle définit le certificat X.509 souhaité. Pour plus d’informations, consultez [Certificats cert-manager][cert-manager-certificates]. cert-manager a automatiquement créé un objet de certificat à l’aide d’ingress-shim ; le certificat est automatiquement déployé avec cert-manager depuis la version 0.2.2. Pour plus d’informations, consultez la [documentation d’ingress-shim][ingress-shim].

Pour vérifier que le certificat a été créé avec succès, utilisez la commande `kubectl get certificate --namespace ingress-basic` et vérifiez que *READY* a la valeur *True*, ce qui peut prendre plusieurs minutes.

```
$ kubectl get certificate --namespace ingress-basic

NAME         READY   SECRET       AGE
tls-secret   True    tls-secret   11m
```

## <a name="test-the-ingress-configuration"></a>Tester la configuration d’entrée

Ouvrez un navigateur web sur le domaine *hello-world-ingress.MY_CUSTOM_DOMAIN* de votre contrôleur d’entrée Kubernetes. Notez que vous êtes redirigé pour utiliser HTTPS, que le certificat est approuvé et que l’application de démonstration est affichée dans le navigateur web. Ajoutez le chemin */hello-world-two* et notez que la deuxième application de démonstration dont le titre est personnalisé est affichée.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Cet article vous a montré comment utiliser Helm pour installer les composants d’entrée, les certificats et les exemples d’applications. Quand vous déployez un graphique Helm, une série de ressources Kubernetes est créée. Ces ressources incluent des pods, des déploiements et des services. Pour nettoyer ces ressources, vous pouvez supprimer l’espace de noms exemple en entier ou des ressources individuelles.

### <a name="delete-the-sample-namespace-and-all-resources"></a>Supprimer l’espace de noms exemple et toutes les ressources

Pour supprimer l’espace de noms exemple en entier, utilisez la commande `kubectl delete` et spécifiez le nom de votre espace de noms. Toutes les ressources de l’espace de noms sont supprimées.

```console
kubectl delete namespace ingress-basic
```

### <a name="delete-resources-individually"></a>Supprimer les ressources individuellement

Sinon, une approche plus précise consiste à supprimer les ressources individuelles créées. Tout d’abord, supprimez les ressources de l’émetteur de cluster :

```console
kubectl delete -f cluster-issuer.yaml --namespace ingress-basic
```

Listez les versions de Helm avec la commande `helm list`. Recherchez les graphiques nommés *nginx* et *cert-manager*, comme illustré dans l’exemple de sortie suivant :

```
$ helm list --namespace ingress-basic

NAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
cert-manager            ingress-basic   1               2020-01-15 10:23:36.515514 -0600 CST    deployed        cert-manager-v0.13.0    v0.13.0    
nginx                   ingress-basic   1               2020-01-15 10:09:45.982693 -0600 CST    deployed        nginx-ingress-1.29.1    0.27.0  
```

Désinstallez les versions avec la commande `helm uninstall`. L’exemple suivant désinstalle les déploiements d’entrée et de gestionnaire de certificats NGINX.

```
$ helm uninstall cert-manager nginx --namespace ingress-basic

release "cert-manager" uninstalled
release "nginx" uninstalled
```

Ensuite, supprimez les deux exemples d’applications :

```console
kubectl delete -f aks-helloworld-one.yaml --namespace ingress-basic
kubectl delete -f aks-helloworld-two.yaml --namespace ingress-basic
```

Supprimez la route d’entrée qui a dirigé le trafic vers les exemples d’applications :

```console
kubectl delete -f hello-world-ingress.yaml --namespace ingress-basic
```

Pour finir, vous pouvez supprimer l’espace de noms lui-même. Utilisez la commande `kubectl delete` et spécifiez le nom de votre espace de noms :

```console
kubectl delete namespace ingress-basic
```

## <a name="next-steps"></a>Étapes suivantes

Cet article a mentionné certains composants qui n’appartiennent pas à AKS. Pour plus d’informations sur ces composants, consultez les pages projet suivantes :

- [Interface CLI Helm][helm-cli]
- [Contrôleur d’entrée NGINX][nginx-ingress]
- [cert-manager][cert-manager]

Vous pouvez également :

- [Créer un contrôleur d’entrée dans Azure Kubernetes Service (AKS)][aks-ingress-basic]
- [Routage d’applications HTTP][aks-http-app-routing]
- [Create an ingress controller to an internal virtual network in Azure Kubernetes Service (AKS)][aks-ingress-internal] (Créer un contrôleur d’entrée pour un réseau virtuel interne dans Azure Kubernetes Service (AKS))
- [Créer un contrôleur d’entrée qui utilise vos propres certificats TLS][aks-ingress-own-tls]
- [Créer un contrôleur d’entrée qui utilise Let’s Encrypt pour générer automatiquement des certificats TLS avec une adresse IP publique statique][aks-ingress-static-tls]

<!-- LINKS - external -->
[az-network-dns-record-set-a-add-record]: /cli/azure/network/dns/record-set/a?view=azure-cli-latest#az-network-dns-record-set-a-add-record
[custom-domain]: ../app-service/manage-custom-dns-buy-domain.md#buy-an-app-service-domain
[dns-zone]: ../dns/dns-getstarted-cli.md
[helm]: https://helm.sh/
[helm-cli]: ./kubernetes-helm.md
[cert-manager]: https://github.com/jetstack/cert-manager
[cert-manager-certificates]: https://cert-manager.readthedocs.io/en/latest/reference/certificates.html
[ingress-shim]: https://docs.cert-manager.io/en/latest/tasks/issuing-certificates/ingress-shim.html
[cert-manager-cluster-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/clusterissuers.html
[cert-manager-issuer]: https://cert-manager.readthedocs.io/en/latest/reference/issuers.html
[lets-encrypt]: https://letsencrypt.org/
[nginx-ingress]: https://github.com/kubernetes/ingress-nginx
[helm-install]: https://docs.helm.sh/using_helm/#installing-helm

<!-- LINKS - internal -->
[use-helm]: kubernetes-helm.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-network-public-ip-create]: /cli/azure/network/public-ip#az-network-public-ip-create
[aks-ingress-internal]: ingress-internal-ip.md
[aks-ingress-static-tls]: ingress-static-ip.md
[aks-ingress-basic]: ingress-basic.md
[aks-http-app-routing]: http-application-routing.md
[aks-ingress-own-tls]: ingress-own-tls.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[client-source-ip]: concepts-network.md#ingress-controllers
[install-azure-cli]: /cli/azure/install-azure-cli
