---
title: Exposer un serveur WebSocket à Application Gateway
description: Cet article fournit des informations sur la façon d’exposer un serveur WebSocket à Application Gateway avec le contrôleur d’entrée pour les clusters AKS.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 68d4ff7e4617136e4c58ce672f34de56e46f0229
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85207785"
---
# <a name="expose-a-websocket-server-to-application-gateway"></a>Exposer un serveur WebSocket à Application Gateway

Comme indiqué dans la documentation d’Application Gateway v2 - [Application Gateway prend en charge les protocoles WebSocket et HTTP/2 de manière native](features.md#websocket-and-http2-traffic). Notez que, pour Application Gateway et l’entrée Kubernetes, Il n’existe aucun paramètre configurable par l’utilisateur permettant d’activer ou de désactiver de manière sélective la prise en charge de WebSocket.

Le YAML de déploiement Kubernetes ci-dessous montre la configuration minimale utilisée pour déployer un serveur WebSocket, ce qui revient à déployer un serveur Web normal :
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: websocket-server
spec:
  selector:
    matchLabels:
      app: ws-app
  replicas: 2
  template:
    metadata:
      labels:
        app: ws-app
    spec:
      containers:
        - name: websocket-app
          imagePullPolicy: Always
          image: your-container-repo.azurecr.io/websockets-app
          ports:
            - containerPort: 8888
      imagePullSecrets:
        - name: azure-container-registry-credentials

---

apiVersion: v1
kind: Service
metadata:
  name: websocket-app-service
spec:
  selector:
    app: ws-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8888

---

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: websocket-repeater
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
    - host: ws.contoso.com
      http:
        paths:
          - backend:
              serviceName: websocket-app-service
              servicePort: 80
```

Étant donné que toutes les conditions préalables sont remplies et que vous disposez d’un Application Gateway contrôlé par une entrée Kubernetes dans votre AKS, le déploiement ci-dessus se traduirait par un serveur WebSockets exposé sur le port 80 de l’adresse IP publique de votre Application Gateway et le domaine `ws.contoso.com`.

La commande cURL suivante testerait le déploiement du serveur WebSocket :
```shell
curl -i -N -H "Connection: Upgrade" \
        -H "Upgrade: websocket" \
        -H "Origin: http://localhost" \
        -H "Host: ws.contoso.com" \
        -H "Sec-Websocket-Version: 13" \
        -H "Sec-WebSocket-Key: 123" \
        http://1.2.3.4:80/ws
```

## <a name="websocket-health-probes"></a>Sondes d’intégrité WebSocket

Si votre déploiement ne définit pas explicitement les sondes d’intégrité, Application Gateway tenterait un HTTP GET sur votre point de terminaison de serveur WebSocket.
En fonction de l’implémentation du serveur ([voici une que nous aimons](https://github.com/gorilla/websocket/blob/master/examples/chat/main.go)) des en-têtes spécifiques à WebSocket peuvent être nécessaires (`Sec-Websocket-Version` par exemple).
Étant donné qu’Application Gateway n’ajoute pas d’en-têtes WebSocket, la réponse de la sonde d’intégrité d’Application Gateway de votre serveur WebSocket sera probablement `400 Bad Request`.
Par conséquent Application Gateway marquera vos pods comme étant défectueux, ce qui aboutira à une `502 Bad Gateway` pour les consommateurs du serveur WebSocket.
Pour éviter cela, vous devrez peut-être ajouter un gestionnaire HTTP GET pour un contrôle d’intégrité de votre serveur (`/health` par exemple, qui retourne `200 OK`).
