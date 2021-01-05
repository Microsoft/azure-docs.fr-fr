---
title: Résolution des problèmes avec Application Gateway Ingress Controller
description: Cet article fournit de la documentation sur la résolution des problèmes courants et/ou des problèmes liés à Application Gateway Ingress Controller.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 06/18/2020
ms.author: caya
ms.openlocfilehash: f2b9f79f0914e645c736f8a577c46baa42587332
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874608"
---
# <a name="troubleshoot-common-questions-or-issues-with-ingress-controller"></a>Résoudre les problèmes courants liés à Ingress Controller

[Azure Cloud Shell](https://shell.azure.com/) est le moyen le plus pratique pour résoudre les problèmes liés à votre installation d’AKS et d’AGIC. Lancez votre interpréteur de commandes à partir de [shell.azure.com](https://shell.azure.com/) ou en cliquant sur le lien suivant :

[![Lancement de l’incorporation](https://shell.azure.com/images/launchcloudshell.png "Lancement d’Azure Cloud Shell")](https://shell.azure.com)


## <a name="test-with-a-simple-kubernetes-app"></a>Tester avec une simple application Kubernetes

Les étapes suivantes partent des suppositions ci-après :
  - Vous disposez d’un cluster AKS avec mise en réseau avancée activée.
  - AGIC a été installé sur le cluster AKS.
  - Vous avez déjà une passerelle Application Gateway sur un réseau virtuel partagé avec votre cluster AKS

Pour vérifier que l’installation Application Gateway + AKS + AGIC est correctement configurée, déployez l’application la plus simple possible :

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: test-agic-app-pod
  labels:
    app: test-agic-app
spec:
  containers:
  - image: "mcr.microsoft.com/dotnet/core/samples:aspnetapp"
    name: aspnetapp-image
    ports:
    - containerPort: 80
      protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: test-agic-app-service
spec:
  selector:
    app: test-agic-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-agic-app-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
    - host: test.agic.contoso.com
      http:
        paths:
          - path: /
            backend:
              serviceName: test-agic-app-service
              servicePort: 80
EOF
```

Copiez et collez toutes les lignes ensemble à partir du script ci-dessus dans une instance [Azure Cloud Shell](https://shell.azure.com/). Vérifiez que la commande entière est copiée, en commençant par `cat` et en incluant le dernier `EOF`.

![apply](./media/application-gateway-ingress-controller-troubleshooting/tsg--apply-config.png)

Une fois le déploiement réussi de l’application ci-dessus, votre nouveau cluster AKS a un nouveau pod, un service et une entrée.

Obtenez la liste des pods avec [Cloud Shell](https://shell.azure.com/) : `kubectl get pods -o wide`.
Nous nous attendons à la création d’un pod nommé « test-agic-app-pod ». Il a une adresse IP. Cette adresse doit se trouver dans le réseau virtuel de la passerelle Application Gateway, qui est utilisée avec AKS.

![Capture d’écran de la fenêtre Bash dans Azure Cloud Shell montrant une liste de pods incluant test-agic-app-pod dans la liste.](./media/application-gateway-ingress-controller-troubleshooting/tsg--get-pods.png)

Obtenez la liste des services : `kubectl get services -o wide`. Nous nous attendons à voir un service nommé « test-agic-app-service ».

![Capture d’écran de la fenêtre Bash dans Azure Cloud Shell montrant une liste de services incluant test-agic-app-pod dans la liste.](./media/application-gateway-ingress-controller-troubleshooting/tsg--get-services.png)

Obtenez la liste des entrées : `kubectl get ingress`. Nous nous attendons à ce qu’une ressource d’entrée nommée « test-agic-app-ingres » soit créée. La ressource a le nom d’hôte « test.agic.contoso.com ».

![Capture d’écran de la fenêtre Bash dans Azure Cloud Shell montrant une liste d’entrées incluant test-agic-app-ingress dans la liste.](./media/application-gateway-ingress-controller-troubleshooting/tsg--get-ingress.png)

L’un des pods est AGIC. `kubectl get pods` affiche une liste de pods, dont l’un commence par « ingres-azure ». Récupérez tous les journaux de ce pod avec `kubectl logs <name-of-ingress-controller-pod>` pour vérifier que le déploiement a réussi. Un déploiement réussi ajoute les lignes suivantes au journal :
```
I0927 22:34:51.281437       1 process.go:156] Applied Application Gateway config in 20.461335266s
I0927 22:34:51.281585       1 process.go:165] cache: Updated with latest applied config.
I0927 22:34:51.282342       1 process.go:171] END AppGateway deployment
```

Vous pouvez également, à partir de [Cloud Shell](https://shell.azure.com/), récupérer uniquement les lignes indiquant la réussite de la configuration d’Application Gateway avec `kubectl logs <ingress-azure-....> | grep 'Applied App Gateway config in'`, où `<ingress-azure....>` doit être le nom exact du pod AGIC.

La configuration suivante est appliquée à Application Gateway :

- Écouteur : ![écouteur](./media/application-gateway-ingress-controller-troubleshooting/tsg--listeners.png)

- Règle d’acheminement : ![routing_rule](./media/application-gateway-ingress-controller-troubleshooting/tsg--rule.png)

- Pool backend :
  - il y a une adresse IP dans le pool d’adresses backend et elle correspond à l’adresse IP du pod que nous avons observée précédemment avec `kubectl get pods -o wide`
![backend_pool](./media/application-gateway-ingress-controller-troubleshooting/tsg--backendpools.png)


Enfin, nous pouvons utiliser la commande `cURL` dans [Cloud Shell](https://shell.azure.com/) pour établir une connexion HTTP à l’application nouvellement déployée :

1. Utilisez `kubectl get ingress` pour récupérer l’adresse IP publique d’Application Gateway.
2. Utilisez `curl -I -H 'test.agic.contoso.com' <publitc-ip-address-from-previous-command>`.

![Capture d’écran de la fenêtre Bash dans Azure Cloud Shell montrant une commande cURL établissant avec succès une connexion HTTP à l’application de test.](./media/application-gateway-ingress-controller-troubleshooting/tsg--curl.png)

Un résultat de `HTTP/1.1 200 OK` indique que le système Application Gateway + AKS + AGIC fonctionne comme prévu.


## <a name="inspect-kubernetes-installation"></a>Inspecter l’installation de Kubernetes

### <a name="pods-services-ingress"></a>Pods, services, entrée
Application Gateway Ingress Controller (AGIC) surveille en continu les ressources Kubernetes suivantes : [Déploiement](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment) ou [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/#what-is-a-pod), [Service](https://kubernetes.io/docs/concepts/services-networking/service/), [Entrée](https://kubernetes.io/docs/concepts/services-networking/ingress/)


Les éléments suivants doivent être en place pour que AGIC fonctionne comme prévu :
  1. AKS doit avoir un ou plusieurs **pods** sains.
     Vérifiez cela à partir de [Cloud Shell](https://shell.azure.com/) avec `kubectl get pods -o wide --show-labels`. Si vous avez un pod avec `apsnetapp`, votre sortie peut se présenter comme suit :
     ```bash
     delyan@Azure:~$ kubectl get pods -o wide --show-labels

     NAME                   READY   STATUS    RESTARTS   AGE   IP          NODE                       NOMINATED NODE   READINESS GATES   LABELS
     aspnetapp              1/1     Running   0          17h   10.0.0.6    aks-agentpool-35064155-1   <none>           <none>            app=aspnetapp
     ```

  2. Un ou plusieurs **services**, qui font référence aux pods ci-dessus via les étiquettes `selector` correspondantes.
     Vérifiez cela à partir de [Cloud Shell](https://shell.azure.com/) avec `kubectl get services -o wide`.
     ```bash
     delyan@Azure:~$ kubectl get services -o wide --show-labels

     NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE   SELECTOR        LABELS
     aspnetapp           ClusterIP   10.2.63.254    <none>        80/TCP    17h   app=aspnetapp   <none>     
     ```

  3. **Entrées**, annotées avec `kubernetes.io/ingress.class: azure/application-gateway`, référençant le service ci-dessus. Vérifiez cela dans [Cloud Shell](https://shell.azure.com/) avec `kubectl get ingress -o wide --show-labels`.
     ```bash
     delyan@Azure:~$ kubectl get ingress -o wide --show-labels

     NAME        HOSTS   ADDRESS   PORTS   AGE   LABELS
     aspnetapp   *                 80      17h   <none>
     ```

  4. Affichez les annotations de l’entrée ci-dessus : `kubectl get ingress aspnetapp -o yaml` (remplacez `aspnetapp` par le nom de votre entrée).
     ```bash
     delyan@Azure:~$ kubectl get ingress aspnetapp -o yaml

     apiVersion: extensions/v1beta1
     kind: Ingress
     metadata:
       annotations:
         kubernetes.io/ingress.class: azure/application-gateway
       name: aspnetapp
     spec:
       backend:
         serviceName: aspnetapp
         servicePort: 80
     ```

     La ressource d’entrée doit être annotée avec `kubernetes.io/ingress.class: azure/application-gateway`.
 

### <a name="verify-observed-namespace"></a>Vérifier l’espace de noms observé

* Récupérez les espaces de noms existants dans le cluster Kubernetes. Dans quel espace de noms votre application s’exécute-t-elle ? AGIC surveille-t-il cet espace de noms ? Reportez-vous à la documentation [Prise en charge de plusieurs espaces de noms](./ingress-controller-multiple-namespace-support.md#enable-multiple-namespace-support) pour savoir comment configurer correctement les espaces de noms observés.

    ```bash
    # What namespaces exist on your cluster
    kubectl get namespaces
    
    # What pods are currently running
    kubectl get pods --all-namespaces -o wide
    ```


* Le pod AGIC doit se trouver dans l’espace de noms `default` (consultez la colonne `NAMESPACE`). Un pod sain a `Running` dans la colonne `STATUS`. Il doit y avoir au moins un pod AGIC.

    ```bash
    # Get a list of the Application Gateway Ingress Controller pods
    kubectl get pods --all-namespaces --selector app=ingress-azure
    ```


* Si le pod AGIC n’est pas sain (la colonne `STATUS` de la commande ci-dessus n’est pas `Running`) :
  - Consultez les journaux pour comprendre pourquoi : `kubectl logs <pod-name>`.
  - Pour l’instance précédente du pod : `kubectl logs <pod-name> --previous`.
  - Décrivez le pod pour obtenir davantage de contexte : `kubectl describe pod <pod-name>`.


* Disposez-vous d’un [service](https://kubernetes.io/docs/concepts/services-networking/service/) Kubernetes et de ressources d’[entrée](https://kubernetes.io/docs/concepts/services-networking/ingress/) ?
    
    ```bash
    # Get all services across all namespaces
    kubectl get service --all-namespaces -o wide
    
    # Get all ingress resources across all namespaces
    kubectl get ingress --all-namespaces -o wide
    ```


* Votre [entrée](https://kubernetes.io/docs/concepts/services-networking/ingress/) est-elle annotée avec `kubernetes.io/ingress.class: azure/application-gateway` ? AGIC surveille uniquement les ressources d’entrée Kubernetes qui ont cette annotation.
    
    ```bash
    # Get the YAML definition of a particular ingress resource
    kubectl get ingress --namespace  <which-namespace?>  <which-ingress?>  -o yaml
    ```


* AGIC émet des événements Kubernetes pour certaines erreurs critiques. Vous pouvez les afficher :
  - Dans votre terminal via `kubectl get events --sort-by=.metadata.creationTimestamp`.
  - Dans votre navigateur à l’aide de l’[interface utilisateur web de Kubernetes (tableau de bord)](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).


## <a name="logging-levels"></a>Niveaux de journalisation

AGIC a 3 niveaux de journalisation. Le niveau 1 est celui par défaut et affiche le nombre minimal de lignes de journal.
Le niveau 5 affiche, en revanche, tous les journaux, notamment le contenu assaini de la configuration appliquée à ARM.

La communauté Kubernetes a établi 9 niveaux de journalisation pour l’outil [kubectl](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-output-verbosity-and-debugging). Dans ce dépôt, nous utilisons 3 de ces niveaux, avec une sémantique similaire :


| Commentaires | Description |
|-----------|-------------|
|  1        | Niveau de journalisation par défaut ; affiche les détails, les avertissements et les erreurs de démarrage |
|  3        | Informations étendues sur les événements et les modifications ; listes d’objets créés |
|  5        | Journalise les objets marshalés ; affiche la configuration JSON expurgée appliquée à ARM |


Les niveaux de détail sont ajustables via la variable `verbosityLevel` dans le fichier [helm-config.yaml](#sample-helm-config-file). Augmentez le niveau de détail à `5` pour que la configuration JSON soit envoyée à [ARM](../azure-resource-manager/management/overview.md) :
  - Ajoutez `verbosityLevel: 5` sur une ligne spécifique dans le fichier [helm-config.yaml](#sample-helm-config-file) et effectuez une réinstallation.
  - Récupérez les journaux avec `kubectl logs <pod-name>`.

### <a name="sample-helm-config-file"></a>Exemple de fichier config Helm
```yaml
# This file contains the essential configs for the ingress controller helm chart

# Verbosity level of the App Gateway Ingress Controller
verbosityLevel: 3

################################################################################
# Specify which application gateway the ingress controller will manage
#
appgw:
    subscriptionId: <subscriptionId>
    resourceGroup: <resourceGroupName>
    name: <applicationGatewayName>

    # Setting appgw.shared to "true" will create an AzureIngressProhibitedTarget CRD.
    # This prohibits AGIC from applying config for any host/path.
    # Use "kubectl get AzureIngressProhibitedTargets" to view and change this.
    shared: false

################################################################################
# Specify which kubernetes namespace the ingress controller will watch
# Default value is "default"
# Leaving this variable out or setting it to blank or empty string would
# result in Ingress Controller observing all acessible namespaces.
#
# kubernetes:
#   watchNamespace: <namespace>

################################################################################
# Specify the authentication with Azure Resource Manager
#
# Two authentication methods are available:
# - Option 1: AAD-Pod-Identity (https://github.com/Azure/aad-pod-identity)
armAuth:
    type: aadPodIdentity
    identityResourceID: <identityResourceId>
    identityClientID:  <identityClientId>

## Alternatively you can use Service Principal credentials
# armAuth:
#    type: servicePrincipal
#    secretJSON: <<Generate this value with: "az ad sp create-for-rbac --subscription <subscription-uuid> --sdk-auth | base64 -w0" >>

################################################################################
# Specify if the cluster is Kubernetes RBAC enabled or not
rbac:
    enabled: false # true/false

# Specify aks cluster related information. THIS IS BEING DEPRECATED.
aksClusterConfiguration:
    apiServerAddress: <aks-api-server-address>
```