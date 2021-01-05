---
title: Annotations du contrôleur d’entrée Application Gateway
description: Cet article fournit de la documentation sur les annotations spécifiques au contrôleur d’entrée Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: article
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 4f570f9f18f9c9d484a9bc9c1a5c64d42dbdc714
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397448"
---
# <a name="annotations-for-application-gateway-ingress-controller"></a>Annotations pour le contrôleur d’entrée Application Gateway 

## <a name="introductions"></a>Introduction

La ressource d’entrée Kubernetes peut être annotée avec des paires clé/valeur arbitraires. AGIC s’appuie sur les annotations pour programmer des fonctionnalités d’Application Gateway, qui ne sont pas configurables par le biais du YAML d’entrée. Les annotations d’entrée sont appliquées à tous les paramètres HTTP, pools principaux et écouteurs dérivés d’une ressource d’entrée.

## <a name="list-of-supported-annotations"></a>Liste des annotations prises en charge

Pour qu’une ressource d’entrée soit observée par AGIC, elle **doit être annotée** avec `kubernetes.io/ingress.class: azure/application-gateway`. Ce n’est qu’à ce moment-là qu’AGIC fonctionnera avec la ressource d’entrée en question.

| Clé d’annotation | Type de valeur | Valeur par défaut | Valeurs autorisées
| -- | -- | -- | -- |
| [appgw.ingress.kubernetes.io/backend-path-prefix](#backend-path-prefix) | `string` | `nil` | |
| [appgw.ingress.kubernetes.io/ssl-redirect](#tls-redirect) | `bool` | `false` | |
| [appgw.ingress.kubernetes.io/connection-draining](#connection-draining) | `bool` | `false` | |
| [appgw.ingress.kubernetes.io/connection-draining-timeout](#connection-draining) | `int32` (secondes) | `30` | |
| [appgw.ingress.kubernetes.io/cookie-based-affinity](#cookie-based-affinity) | `bool` | `false` | |
| [appgw.ingress.kubernetes.io/request-timeout](#request-timeout) | `int32` (secondes) | `30` | |
| [appgw.ingress.kubernetes.io/use-private-ip](#use-private-ip) | `bool` | `false` | |
| [appgw.ingress.kubernetes.io/backend-protocol](#backend-protocol) | `string` | `http` | `http`, `https` |

## <a name="backend-path-prefix"></a>Préfixe du chemin d’accès principal

Cette annotation permet de réécrire le chemin d’accès principal spécifié dans une ressource d’entrée avec le préfixe spécifié dans cette annotation. Cela permet aux utilisateurs d’exposer des services dont les points de terminaison sont différents des noms de point de terminaison utilisés pour exposer un service dans une ressource d’entrée.

### <a name="usage"></a>Usage

```yaml
appgw.ingress.kubernetes.io/backend-path-prefix: <path prefix>
```

### <a name="example"></a>Exemple

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-server-ingress-bkprefix
  namespace: test-ag
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/backend-path-prefix: "/test/"
spec:
  rules:
  - http:
      paths:
      - path: /hello/
        backend:
          serviceName: go-server-service
          servicePort: 80
```
Dans l’exemple ci-dessus, nous avons défini une ressource d’entrée nommée `go-server-ingress-bkprefix` avec une annotation `appgw.ingress.kubernetes.io/backend-path-prefix: "/test/"`. L’annotation indique à la passerelle de l’application de créer un paramètre HTTP, dont le préfixe du chemin d’accès `/hello` sera substitué par `/test/`.

> [!NOTE] 
> Dans l’exemple ci-dessus, une seule règle est définie. Toutefois, les annotations sont applicables à l’ensemble de la ressource d’entrée. Par conséquent, si un utilisateur a défini plusieurs règles, le préfixe du chemin d’accès principal sera configuré pour chacun des chemins d’accès spécifiés. Ainsi, si un utilisateur souhaite des règles différentes avec différents préfixes de chemin d’accès (même pour le même service), il devra définir des ressources d’entrée différentes.

## <a name="tls-redirect"></a>Redirection TLS

Application Gateway [peut être configurée](./redirect-overview.md) pour rediriger automatiquement les URL HTTP vers leur équivalent HTTPS. Lorsque cette annotation est présente et que le protocole TLS est correctement configuré, le contrôleur d’entrée Kubernetes créera une [règle d’acheminement avec une configuration de redirection](./redirect-http-to-https-portal.md#add-a-routing-rule-with-a-redirection-configuration) et appliquera les modifications à votre Application Gateway. La redirection créée sera HTTP `301 Moved Permanently`.

### <a name="usage"></a>Usage

```yaml
appgw.ingress.kubernetes.io/ssl-redirect: "true"
```

### <a name="example"></a>Exemple

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-server-ingress-redirect
  namespace: test-ag
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
   - hosts:
     - www.contoso.com
     secretName: testsecret-tls
  rules:
  - host: www.contoso.com
    http:
      paths:
      - backend:
          serviceName: websocket-repeater
          servicePort: 80
```

## <a name="connection-draining"></a>Drainage de la connexion

`connection-draining`: Cette annotation permet aux utilisateurs de spécifier s’il faut activer le drainage de la connexion.
`connection-draining-timeout`: Cette annotation permet aux utilisateurs de spécifier un délai d’expiration après lequel Application Gateway mettra fin aux demandes adressées au point de terminaison principal de drainage.

### <a name="usage"></a>Usage

```yaml
appgw.ingress.kubernetes.io/connection-draining: "true"
appgw.ingress.kubernetes.io/connection-draining-timeout: "60"
```

### <a name="example"></a>Exemple

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-server-ingress-drain
  namespace: test-ag
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/connection-draining: "true"
    appgw.ingress.kubernetes.io/connection-draining-timeout: "60"
spec:
  rules:
  - http:
      paths:
      - path: /hello/
        backend:
          serviceName: go-server-service
          servicePort: 80
```

## <a name="cookie-based-affinity"></a>Affinité basée sur les cookies

Cette annotation permet de spécifier s’il faut activer l’affinité basée sur les cookies.

### <a name="usage"></a>Usage

```yaml
appgw.ingress.kubernetes.io/cookie-based-affinity: "true"
```

### <a name="example"></a>Exemple

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-server-ingress-affinity
  namespace: test-ag
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/cookie-based-affinity: "true"
spec:
  rules:
  - http:
      paths:
      - path: /hello/
        backend:
          serviceName: go-server-service
          servicePort: 80
```

## <a name="request-timeout"></a>Délai d’expiration de la demande

Cette annotation permet de spécifier le délai d’expiration de la requête en secondes après lequel Application Gateway fera échouer la requête si la réponse n’est pas reçue.

### <a name="usage"></a>Usage

```yaml
appgw.ingress.kubernetes.io/request-timeout: "20"
```

### <a name="example"></a>Exemple

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-server-ingress-timeout
  namespace: test-ag
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/request-timeout: "20"
spec:
  rules:
  - http:
      paths:
      - path: /hello/
        backend:
          serviceName: go-server-service
          servicePort: 80
```

## <a name="use-private-ip"></a>Utiliser une adresse IP privée

Cette annotation nous permet de spécifier s’il faut exposer ce point de terminaison sur le protocole Internet (IP) privé d’Application Gateway.

> [!NOTE]
> * Application Gateway ne prend pas en charge plusieurs IP sur le même port (par exemple : 80/443). Une entrée avec l’annotation `appgw.ingress.kubernetes.io/use-private-ip: "false"` et une autre avec `appgw.ingress.kubernetes.io/use-private-ip: "true"` sur `HTTP` entraînera l’échec d’AGIC à mettre à jour Application Gateway.
> * Pour Application Gateway sans IP privé, les entrées avec `appgw.ingress.kubernetes.io/use-private-ip: "true"` seront ignorées. Cela se reflétera dans les journaux du contrôleur et les événements d’entrée pour les entrées avec l’avertissement `NoPrivateIP`.


### <a name="usage"></a>Usage
```yaml
appgw.ingress.kubernetes.io/use-private-ip: "true"
```

### <a name="example"></a>Exemple
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-server-ingress-timeout
  namespace: test-ag
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/use-private-ip: "true"
spec:
  rules:
  - http:
      paths:
      - path: /hello/
        backend:
          serviceName: go-server-service
          servicePort: 80
```

## <a name="backend-protocol"></a>Protocole principal

Cette annotation nous permet de spécifier le protocole qu’Application Gateway doit utiliser lors de la communication avec les pods. Protocoles pris en charge : `http`, `https`

> [!NOTE]
> * Alors que les certificats auto-signés sont pris en charge sur Application Gateway, AGIC prend actuellement en charge uniquement `https` lorsque les pods utilisent un certificat signé par une autorité de certification reconnue.
> * Veillez à ne pas utiliser le port 80 avec HTTPs et le port 443 avec HTTP sur les pods.

### <a name="usage"></a>Usage
```yaml
appgw.ingress.kubernetes.io/backend-protocol: "https"
```

### <a name="example"></a>Exemple
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: go-server-ingress-timeout
  namespace: test-ag
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
    appgw.ingress.kubernetes.io/backend-protocol: "https"
spec:
  rules:
  - http:
      paths:
      - path: /hello/
        backend:
          serviceName: go-server-service
          servicePort: 443
```