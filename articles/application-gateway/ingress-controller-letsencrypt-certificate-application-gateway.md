---
title: Utiliser des certificats LetsEncrypt.org avec Application Gateway
description: Cet article fournit des informations sur la façon d’obtenir un certificat de LetsEncrypt.org et de l’utiliser sur votre Application Gateway pour des clusters AKS.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: df8722e8160538daa1535711092790dbb2405097
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84807031"
---
# <a name="use-certificates-with-letsencryptorg-on-application-gateway-for-aks-clusters"></a>Utiliser des certificats avec LetsEncrypt.org sur Application Gateway pour des clusters AKS

Cette section configure votre AKS pour tirer parti de [LetsEncrypt.org](https://letsencrypt.org/) et obtenir automatiquement un certificat TLS/SSL pour votre domaine. Le certificat sera installé sur Application Gateway, ce qui entraînera l’arrêt du protocole SSL/TLS pour votre cluster AKS. La configuration décrite décrit ici utilise le module complémentaire Kubernetes [ cert-manager](https://github.com/jetstack/cert-manager), qui automatise la création et la gestion des certificats.

Suivez les étapes ci-dessous pour installer [cert-manager](https://docs.cert-manager.io) sur votre cluster AKS existant.

1. Graphique Helm

    Exécutez le script suivant pour installer le graphique Helm `cert-manager`. Avec cette opération, vous pouvez :

    - créer un espace de noms `cert-manager` sur votre AKS
    - créer les CRD suivants : Certificate, Challenge, ClusterIssuer, Issuer, Order
    - installer le graphique cert-manager (à partir de [docs.cert-manager.io](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html#steps))

    ```bash
    #!/bin/bash

    # Install the CustomResourceDefinition resources separately
    kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.8/deploy/manifests/00-crds.yaml

    # Create the namespace for cert-manager
    kubectl create namespace cert-manager

    # Label the cert-manager namespace to disable resource validation
    kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

    # Add the Jetstack Helm repository
    helm repo add jetstack https://charts.jetstack.io

    # Update your local Helm chart repository cache
    helm repo update

    # Install the cert-manager Helm chart
    helm install \
      --name cert-manager \
      --namespace cert-manager \
      --version v0.8.0 \
      jetstack/cert-manager
    ```

2. Ressource ClusterIssuer

    Créez une ressource `ClusterIssuer`. Elle est requise par `cert-manager` pour représenter l’autorité de certification `Lets Encrypt` auprès de laquelle les certificats signés seront obtenus.

    À l’aide de la ressource `ClusterIssuer` sans espace de noms, cert-manager émet des certificats qui peuvent être utilisés à partir de plusieurs espaces de noms. `Let’s Encrypt` utilise le protocole ACME pour vérifier que vous contrôlez un nom de domaine donné et vous délivrer un certificat. Plus d’informations sur la configuration des propriétés `ClusterIssuer` sont accessibles [ici](https://docs.cert-manager.io/en/latest/tasks/issuers/index.html). `ClusterIssuer` demande à `cert-manager` d’émettre des certificats à l’aide de l’environnement intermédiaire `Lets Encrypt` utilisé pour les tests (le certificat racine n’est pas présent dans les magasins d’approbations de navigateur/client).

    Le type de stimulation par défaut dans le YAML ci-dessous est `http01`. D’autres stimulations sont documentées sur [letsencrypt.org – Types de stimulation](https://letsencrypt.org/docs/challenge-types/)

    > [!IMPORTANT] 
    > Mettez à jour `<YOUR.EMAIL@ADDRESS>` dans le YAML ci-dessous

    ```bash
    #!/bin/bash
    kubectl apply -f - <<EOF
    apiVersion: certmanager.k8s.io/v1alpha1
    kind: ClusterIssuer
    metadata:
    name: letsencrypt-staging
    spec:
    acme:
        # You must replace this email address with your own.
        # Let's Encrypt will use this to contact you about expiring
        # certificates, and issues related to your account.
        email: <YOUR.EMAIL@ADDRESS>
        # ACME server URL for Let’s Encrypt’s staging environment.
        # The staging environment will not issue trusted certificates but is
        # used to ensure that the verification process is working properly
        # before moving to production
        server: https://acme-staging-v02.api.letsencrypt.org/directory
        privateKeySecretRef:
        # Secret resource used to store the account's private key.
        name: example-issuer-account-key
        # Enable the HTTP-01 challenge provider
        # you prove ownership of a domain by ensuring that a particular
        # file is present at the domain
        http01: {}
    EOF
    ```

3. Déployer une application

    Créez une ressource d’entrée pour exposer l’application `guestbook` à l’aide d’Application Gateway avec le certificat Let’s Encrypt.

    Assurez-vous qu’Application Gateway dispose d’une configuration IP frontale publique avec un nom DNS (soit en utilisant le domaine `azure.com` par défaut, soit en fournissant un service `Azure DNS Zone` et en attribuant votre propre domaine personnalisé).
    Notez l’annotation `certmanager.k8s.io/cluster-issuer: letsencrypt-staging`, qui indique à cert-manager de traiter la ressource d’entrée étiquetée.

    > [!IMPORTANT] 
    > Mettez à jour `<PLACEHOLDERS.COM>` dans le YAML ci-dessous avec votre propre domaine (ou celui d’Application Gateway, par exemple « kh-aks-ingress.westeurope.cloudapp.azure.com »)

    ```bash
    kubectl apply -f - <<EOF
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
    name: guestbook-letsencrypt-staging
    annotations:
        kubernetes.io/ingress.class: azure/application-gateway
        certmanager.k8s.io/cluster-issuer: letsencrypt-staging
    spec:
    tls:
    - hosts:
        - <PLACEHOLDERS.COM>
        secretName: guestbook-secret-name
    rules:
    - host: <PLACEHOLDERS.COM>
        http:
        paths:
        - backend:
            serviceName: frontend
            servicePort: 80
    EOF
    ```

    Après quelques secondes, vous pouvez accéder au service `guestbook` par le biais de l’URL HTTPS d’Application Gateway à l’aide du certificat **intermédiaire** `Lets Encrypt` émis automatiquement.
    Votre navigateur peut vous avertir qu’une autorité de certification est non valide. Le certificat intermédiaire est émis par `CN=Fake LE Intermediate X1`. Cela indique que le système a fonctionné comme prévu et que vous êtes prêt pour votre certificat de production.

4. Certificat de production

    Une fois que votre certificat de préproduction est correctement configuré, vous pouvez basculer vers un serveur ACME de production :
    1. Remplacez l’annotation intermédiaire de votre ressource d’entrée par : `certmanager.k8s.io/cluster-issuer: letsencrypt-prod`
    1. Supprimez la ressource `ClusterIssuer` intermédiaire existante que vous avez créée à l’étape précédente et créez-en une nouvelle en remplaçant le serveur ACME du YAML ClusterIssuer ci-dessus par `https://acme-v02.api.letsencrypt.org/directory`

5. Expiration et renouvellement du certificat

    Avant l’expiration du certificat `Lets Encrypt`, `cert-manager` met automatiquement à jour le certificat dans le magasin des secrets Kubernetes. À ce stade, le contrôleur d’entrée Application Gateway appliquera le secret mis à jour référencé dans les ressources d’entrée qu’il utilise pour configurer l’Application Gateway.
