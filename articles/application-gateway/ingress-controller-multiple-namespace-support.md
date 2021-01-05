---
title: Activer les prises en charge de plusieurs espaces de noms pour le contrôleur d’entrée d’Application Gateway
description: Cet article fournit des informations sur l’activation de la prise en charge de plusieurs espaces de noms dans un cluster Kubernetes avec un contrôleur d’entrée Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: c13c4410852d97f0bf4548578f40a5cc560804d7
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874591"
---
# <a name="enable-multiple-namespace-support-in-an-aks-cluster-with-application-gateway-ingress-controller"></a>Activer la prise en charge de plusieurs espaces de noms dans un cluster AKS avec un contrôleur d’entrée Application Gateway

## <a name="motivation"></a>Motivation
Les [espaces de noms](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) Kubernetes permettent de partitionner un cluster Kubernetes et de l’allouer à des sous-groupes d’une équipe plus importante. Ces sous-équipes peuvent ensuite déployer et gérer l’infrastructure avec un contrôle affiné des ressources, de la sécurité, de la configuration, etc. Kubernetes permet de définir indépendamment une ou plusieurs ressources d’entrée au sein de chaque espace de noms.

À partir de la version 0.7, [Azure Application Gateway Kubernetes IngressController](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/master/README.md) (AGIC) peut observer plusieurs espaces de noms et en ingérer des événements. Si l’administrateur AKS décide d’utiliser [App Gateway](https://azure.microsoft.com/services/application-gateway/) comme entrée, tous les espaces de noms utiliseront la même instance d’Application Gateway. Une seule installation d’AGIC surveillera les espaces de noms accessibles et configurera l’Application Gateway à laquelle il est associé.

La version 0.7 d’AGIC continuera à observer exclusivement l’espace de noms `default`, à moins que celui-ci ne soit explicitement modifié en un ou plusieurs espaces de noms différents dans la configuration Helm (voir la section ci-dessous).

## <a name="enable-multiple-namespace-support"></a>Activer la prise en charge de plusieurs espaces de noms
Pour activer la prise en charge de plusieurs espaces de noms :
1. modifiez le fichier [helm-config.yaml](#sample-helm-config-file) de l’une des manières suivantes :
   - supprimez entièrement la clé `watchNamespace` de [helm-config.yaml](#sample-helm-config-file) ; AGIC observera tous les espaces de noms
   - définissez `watchNamespace` sur une chaîne vide ; AGIC observera tous les espaces de noms
   - ajoutez plusieurs espaces de noms séparés par une virgule (`watchNamespace: default,secondNamespace`) ; AGIC observera ces espaces de noms exclusivement
2. appliquez les modifications du modèle Helm avec : `helm install -f helm-config.yaml application-gateway-kubernetes-ingress/ingress-azure`

Une fois déployé avec la possibilité d’observer plusieurs espaces de noms, AGIC réalisera les actions suivantes :
  - répertorier les ressources d’entrée de tous les espaces de noms accessibles
  - filtrer jusqu’aux ressources d’entrée annotées avec `kubernetes.io/ingress.class: azure/application-gateway`
  - composer une [configuration Application Gateway](https://github.com/Azure/azure-sdk-for-go/blob/37f3f4162dfce955ef5225ead57216cf8c1b2c70/services/network/mgmt/2016-06-01/network/models.go#L1710-L1744) combinée
  - appliquer la configuration à l’Application Gateway associée via [ARM](../azure-resource-manager/management/overview.md)

## <a name="conflicting-configurations"></a>Configurations conflictuelles
Des [ressources d’entrée](https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource) à plusieurs espaces de noms peuvent demander à AGIC de créer des configurations conflictuelles pour une seule Application Gateway. (Deux entrées revendiquant le même domaine, par exemple.)

En haut de la hiérarchie : des **écouteurs**  (adresse IP, port et hôte) et des **règles d’acheminement** (écouteur de liaison, pool principal et paramètres HTTP) peuvent être créés et partagés par plusieurs espaces de noms/entrées.

En bas de la hiérarchie : des chemins d’accès, des pools principaux, des paramètres HTTP et des certificats TLS peuvent être créés par un seul espace de noms et les doublons sont supprimés.

Par exemple, considérez que les ressources d’entrée en double suivantes définissent les espaces de noms `staging` et `production` pour `www.contoso.com` :

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: websocket-ingress
  namespace: staging
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
    - host: www.contoso.com
      http:
        paths:
          - backend:
              serviceName: web-service
              servicePort: 80
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: websocket-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
    - host: www.contoso.com
      http:
        paths:
          - backend:
              serviceName: web-service
              servicePort: 80
```

Bien que les deux ressources d’entrée exigent que le trafic pour `www.contoso.com` soit acheminé vers les espaces de noms Kubernetes respectifs, un seul serveur principal peut fournir le trafic. AGIC crée une configuration selon le principe du « premier arrivé, premier servi » pour l’une des ressources. Si deux ressources d’entrée sont créées en même temps, la priorité est accordée à la ressource apparaissant en premier selon l’ordre alphabétique. Dans l’exemple ci-dessus, nous pourrons uniquement créer des paramètres pour l’entrée `production`. Application Gateway sera configuré avec les ressources suivantes :

  - Écouteur : `fl-www.contoso.com-80`
  - Règle d’acheminement : `rr-www.contoso.com-80`
  - Pool principal : `pool-production-contoso-web-service-80-bp-80`
  - Paramètres HTTP : `bp-production-contoso-web-service-80-80-websocket-ingress`
  - Sonde d’intégrité : `pb-production-contoso-web-service-80-websocket-ingress`

Notez qu’à l’exception de l’*écouteur* et de la *règle d’acheminement*, les ressources Application Gateway créées incluent le nom de l’espace de noms (`production`) pour lequel elles ont été créées.

Si les deux ressources d’entrée sont introduites dans le cluster AKS à des moments différents, AGIC risque vraisemblablement d’aboutir à un scénario où il reconfigure Application Gateway et réachemine le trafic de `namespace-B` vers `namespace-A`.

Par exemple, si vous avez ajouté `staging` en premier, AGIC configurera Application Gateway de sorte qu’il achemine le trafic vers le pool principal intermédiaire. À un stade ultérieur, l’introduction de l’entrée `production` amènera AGIC à reprogrammer Application Gateway, qui commencera à acheminer le trafic vers le pool principal `production`.

## <a name="restrict-access-to-namespaces"></a>Limiter l’accès aux espaces de noms
Par défaut, AGIC configure Application Gateway en fonction de l’entrée annotée dans n’importe quel espace de noms. Si vous souhaitez limiter ce comportement, vous disposez des options suivantes :
  - limitez les espaces de noms en définissant explicitement ceux qu’AGIC doit observer par le biais de la clé YAML `watchNamespace` dans [helm-config.yaml](#sample-helm-config-file)
  - utilisez [Role/RoleBinding](../aks/azure-ad-rbac.md) pour limiter AGIC à des espaces de noms spécifiques

## <a name="sample-helm-config-file"></a>Exemple de fichier config Helm

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