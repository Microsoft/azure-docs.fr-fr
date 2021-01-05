---
title: Exposer un service AKS via HTTP ou HTTPS à l’aide d’Application Gateway
description: Cet article fournit des informations sur la façon d’exposer un service AKS via HTTP ou HTTPS en utilisant Application Gateway.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 3b816ddc0eccf8c406cfed37d6bfc594e27d3629
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85850364"
---
# <a name="expose-an-aks-service-over-http-or-https-using-application-gateway"></a>Exposer un service AKS via HTTP ou HTTPS à l’aide d’Application Gateway 

Ces tutoriels aident à illustrer l’utilisation des [ressources d’entrée de Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/) pour exposer un exemple de service Kubernetes par l’intermédiaire d’[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) via HTTP ou HTTPS.

## <a name="prerequisites"></a>Prérequis

- Installer le graphique Helm `ingress-azure`.
  - [**Déploiement « greenfield »** ](ingress-controller-install-new.md) : Si vous partez de zéro, reportez-vous à ces instructions d’installation, qui décrivent les étapes à suivre pour déployer un cluster AKS à l’aide d’Application Gateway et y installer le contrôleur d’entrée de la passerelle d’application.
  - [**Déploiement « brownfield »** ](ingress-controller-install-existing.md) : Si vous disposez déjà d’un cluster AKS et d’Application Gateway, reportez-vous à ces instructions pour installer le contrôleur d’entrée de passerelle d’application sur le cluster AKS.
- Si vous souhaitez utiliser le protocole HTTPS sur cette application, vous aurez besoin d’un certificat x509 et de sa clé privée.

## <a name="deploy-guestbook-application"></a>Déployer l’application `guestbook`

L’application de livre d’or est une application Kubernetes canonique qui se compose d’un serveur frontal pour l’interface utilisateur Web, d’un serveur principal et d’une base de données Redis. Par défaut, `guestbook` expose son application par le biais d’un service nommé `frontend` sur le port `80`. Sans une ressource d’entrée Kubernetes, le service n’est pas accessible depuis l’extérieur du cluster AKS. Nous utiliserons les ressources d’entrée de l’application et du programme d’installation pour accéder à l’application via HTTP et HTTPS.

Pour déployer l’application de livre d’or, suivez les instructions ci-dessous.

1. Téléchargez `guestbook-all-in-one.yaml`[ici](https://raw.githubusercontent.com/kubernetes/examples/master/guestbook/all-in-one/guestbook-all-in-one.yaml)
1. Déployez `guestbook-all-in-one.yaml` dans votre cluster AKS en l’exécutant

  ```bash
  kubectl apply -f guestbook-all-in-one.yaml
  ```

L’application `guestbook` est à présent déployée.

## <a name="expose-services-over-http"></a>Exposer des services via HTTP

Pour exposer l’application de livre d’or, nous utiliserons la ressource d’entrée suivante :

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: guestbook
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: frontend
          servicePort: 80
```

Cette entrée exposera le service `frontend` du déploiement `guestbook-all-in-one` comme serveur principal par défaut d’Application Gateway.

Enregistrez la ressource d’entrée ci-dessus sous `ing-guestbook.yaml`.

1. Déployez `ing-guestbook.yaml` en l’exécutant :

    ```bash
    kubectl apply -f ing-guestbook.yaml
    ```

1. Vérifiez l’état du déploiement dans le journal du contrôleur d’entrée.

L’application `guestbook` devrait maintenant être disponible. Vous pouvez le vérifier en vous rendant sur l’adresse publique d’Application Gateway.

## <a name="expose-services-over-https"></a>Exposer des services via HTTPS

### <a name="without-specified-hostname"></a>Sans nom d’hôte spécifié

Si vous ne spécifiez pas de nom d’hôte, le service de livre d’or sera disponible sur tous les noms d’hôtes pointant vers la passerelle de l’application.

1. Avant de déployer l’entrée, vous devez créer un secret Kubernetes pour héberger le certificat et la clé privée. Vous pouvez créer un secret Kubernetes en l’exécutant

    ```bash
    kubectl create secret tls <guestbook-secret-name> --key <path-to-key> --cert <path-to-cert>
    ```

1. Définissez l’entrée suivante. Dans l’entrée, indiquez le nom du secret dans la section `secretName`.

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: guestbook
      annotations:
        kubernetes.io/ingress.class: azure/application-gateway
    spec:
      tls:
        - secretName: <guestbook-secret-name>
      rules:
      - http:
          paths:
          - backend:
              serviceName: frontend
              servicePort: 80
    ```

    > [!NOTE] 
    > Remplacez `<guestbook-secret-name>` dans la ressource d’entrée ci-dessus par le nom de votre clé secrète. Stockez la ressource d’entrée ci-dessus dans un nom de fichier `ing-guestbook-tls.yaml`.

1. Déployez ing-guestbook-tls.yam en l’exécutant

    ```bash
    kubectl apply -f ing-guestbook-tls.yaml
    ```

1. Vérifiez l’état du déploiement dans le journal du contrôleur d’entrée.

L’application `guestbook` est désormais disponible sur HTTP et HTTPS.

### <a name="with-specified-hostname"></a>Avec le nom d’hôte spécifié

Vous pouvez également spécifier le nom d’hôte sur l’entrée afin de multiplexer les configurations et les services TLS.
En spécifiant le nom d’hôte, le service de livre d’or ne sera disponible que sur l’hôte spécifié.

1. Définissez l’entrée suivante.
    Dans l’entrée, spécifiez le nom du secret dans la section `secretName` et remplacez le nom d’hôte dans la section `hosts` en conséquence.

    ```yaml
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: guestbook
      annotations:
        kubernetes.io/ingress.class: azure/application-gateway
    spec:
      tls:
        - hosts:
          - <guestbook.contoso.com>
          secretName: <guestbook-secret-name>
      rules:
      - host: <guestbook.contoso.com>
        http:
          paths:
          - backend:
              serviceName: frontend
              servicePort: 80
    ```

1. Déployez `ing-guestbook-tls-sni.yaml` en l’exécutant

    ```bash
    kubectl apply -f ing-guestbook-tls-sni.yaml
    ```

1. Vérifiez l’état du déploiement dans le journal du contrôleur d’entrée.

L’application `guestbook` est désormais disponible sur HTTP et HTTPS uniquement sur l’hôte spécifié (`<guestbook.contoso.com>` dans cet exemple).

## <a name="integrate-with-other-services"></a>Intégrer aux autres services

L’entrée suivante vous permettra d’ajouter des chemins supplémentaires dans cette entrée et de rediriger ces chemins vers d’autres services :

```yaml
apiVersion: extensions/v1beta1
  kind: Ingress
  metadata:
    name: guestbook
    annotations:
      kubernetes.io/ingress.class: azure/application-gateway
  spec:
    rules:
    - http:
        paths:
        - path: </other/*>
          backend:
            serviceName: <other-service>
            servicePort: 80
        - backend:
            serviceName: frontend
            servicePort: 80
```
