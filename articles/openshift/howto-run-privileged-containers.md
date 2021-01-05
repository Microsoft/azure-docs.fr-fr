---
title: Gérer des conteneurs privilégiés dans un cluster Azure Red Hat OpenShift | Microsoft Docs
description: Exécutez des conteneurs privilégiés pour superviser la sécurité et la conformité.
author: makdaam
ms.author: b-lejaku
ms.service: container-service
ms.topic: conceptual
ms.date: 12/05/2019
keywords: aro, openshift, aquasec, twistlock, red hat
ms.openlocfilehash: 914b29410a0f30e5c3d3a893c2e278ecbb83b648
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92218865"
---
# <a name="run-privileged-containers-in-an-azure-red-hat-openshift-cluster"></a>Gérer des conteneurs privilégiés dans un cluster Azure Red Hat OpenShift

> [!IMPORTANT]
> Azure Red Hat OpenShift 3.11 sera mis hors service le 30 juin 2022. La prise en charge de la création de nouveaux clusters Azure Red Hat OpenShift 3.11 se poursuit jusqu’au 30 novembre 2020. Après la mise hors service, les clusters Azure Red Hat OpenShift 3.11 restants seront arrêtés pour éviter des failles de sécurité.
> 
> Suivez ce guide pour [créer un cluster Azure Red Hat OpenShift 4](tutorial-create-cluster.md).
> Si vous avez des questions spécifiques, n’hésitez pas à [nous contacter](mailto:arofeedback@microsoft.com).

Vous ne pouvez pas exécuter de conteneurs privilégiés sur les clusters Azure Red Hat OpenShift.
Deux solutions de supervision et de conformité de la sécurité sont autorisées à s’exécuter sur les clusters ARO.
Ce document décrit les différences par rapport à la documentation générique de déploiement OpenShift des fournisseurs de produits de sécurité.


Lisez ces instructions avant de suivre celles du fournisseur.
Les titres de section dans les étapes propres au produit ci-dessous font directement référence aux titres de section de la documentation des fournisseurs.

## <a name="before-you-begin"></a>Avant de commencer

La documentation de la plupart des produits de sécurité suppose que vous disposiez de privilèges d’administrateur de cluster.
Les administrateurs de clients n’ont pas tous les privilèges dans Azure Red Hat OpenShift. Les autorisations requises pour modifier les ressources au niveau du cluster sont limitées.

Tout d’abord, vérifiez que l’utilisateur est connecté au cluster en tant qu’administrateur du client, en exécutant `oc get scc`. Tous les utilisateurs qui sont membres du groupe d’administrateurs du client sont autorisés à afficher les contraintes de contexte de sécurité sur le cluster.

Ensuite, vérifiez que la version binaire `oc` est `3.11.154`.
```
oc version
oc v3.11.154
kubernetes v1.11.0+d4cacc0
features: Basic-Auth GSSAPI Kerberos SPNEGO

Server https://openshift.aqua-test.osadev.cloud:443
openshift v3.11.154
kubernetes v1.11.0+d4cacc0
```

## <a name="product-specific-steps-for-aqua-security"></a>Étapes propres au produit pour Aqua Security
Vous trouverez les instructions de base que nous allons modifier dans la [documentation sur le déploiement Aqua Security](https://docs.aquasec.com/docs/openshift-red-hat). Les étapes ci-dessous s’exécutent conjointement à la documentation sur le déploiement Aqua.

La première étape consiste à annoter les contraintes de contexte de sécurité requises qui seront mises à jour. Ces annotations empêchent le pod de synchronisation du cluster de restaurer les modifications apportées à ces contraintes.

```
oc annotate scc hostaccess openshift.io/reconcile-protect=true
oc annotate scc privileged openshift.io/reconcile-protect=true
```

### <a name="step-1-prepare-prerequisites"></a>Étape 1 : Préparer les prérequis
N’oubliez pas de vous connecter au cluster en tant qu’administrateur de client ARO, et non à l’aide du rôle administrateur de cluster.

Créez le projet et le compte de service.
```
oc new-project aqua-security
oc create serviceaccount aqua-account -n aqua-security
```

Au lieu d’attribuer le rôle cluster-reader, affectez le rôle customer-admin-cluster au compte Aqua à l’aide de la commande suivante.
```
oc adm policy add-cluster-role-to-user customer-admin-cluster system:serviceaccount:aqua-security:aqua-account
oc adm policy add-scc-to-user privileged system:serviceaccount:aqua-security:aqua-account
oc adm policy add-scc-to-user hostaccess system:serviceaccount:aqua-security:aqua-account
```

Continuez à suivre les instructions restantes de l’étape 1.  Ces instructions décrivent la configuration du secret pour le registre Aqua.

### <a name="step-2-deploy-the-aqua-server-database-and-gateway"></a>Étape 2 : Déployer le serveur, la base de données et la passerelle Aqua
Suivez les étapes indiquées dans la documentation Aqua pour installer le fichier aqua-console.yaml.

Modifiez le fichier `aqua-console.yaml` fourni.  Supprimez les deux premiers objets étiquetés `kind: ClusterRole` et `kind: ClusterRoleBinding`.  Ces ressources ne sont pas créées, car l’administrateur du client ne dispose pas de l’autorisation nécessaire pour modifier les objets `ClusterRole` et `ClusterRoleBinding`.

La deuxième modification porte sur la partie `kind: Route` du fichier `aqua-console.yaml`. Dans le fichier `aqua-console.yaml`, remplacez l’objet `kind: Route` par le code yaml suivant.
```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    app: aqua-web
  name: aqua-web
  namespace: aqua-security
spec:
  port:
    targetPort: aqua-web
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: edge
  to:
    kind: Service
    name: aqua-web
    weight: 100
  wildcardPolicy: None
```

Suivez les instructions restantes.

### <a name="step-3-login-to-the-aqua-server"></a>Étape 3 : Se connecter au serveur Aqua
Cette section n’est pas modifiée.  Suivez la documentation Aqua.

Utilisez la commande suivante pour récupérer l’adresse de la console Aqua.
```
oc get route aqua-web -n aqua-security
```

### <a name="step-4-deploy-aqua-enforcers"></a>Étape 4 : Déployer les Aqua Enforcers
Définissez les champs suivants lors du déploiement des enforcers :

| Champ          | Valeur         |
| -------------- | ------------- |
| Un orchestrateur   | OpenShift     |
| ServiceAccount | aqua-account  |
| Project        | aqua-security |

## <a name="product-specific-steps-for-prisma-cloud--twistlock"></a>Étapes propres au produit pour Prisma Cloud/Twistlock

Vous trouverez les instructions de base que nous allons modifier dans la [documentation sur le déploiement Prisma Cloud](https://docs.paloaltonetworks.com/prisma/prisma-cloud/19-11/prisma-cloud-compute-edition-admin/install/install_openshift.html).

Commencez par installer l’outil `twistcli` comme décrit dans les sections « Install Prisma Cloud » et « Download the Prisma Cloud software ».

Créer un projet OpenShift
```
oc new-project twistlock
```

Ignorez la section facultative « Push the Prisma Cloud images to a private registry ». Elle n’est pas applicable à Azure Red Hat OpenShift. Utilisez le registre en ligne à la place.

Vous pouvez suivre la documentation officielle tout en appliquant les corrections décrites ci-dessous.
Commencez par la section « Installer la console ».

### <a name="install-console"></a>Installer la console

Lors de l’exécution de la commande `oc create -f twistlock_console.yaml` à l’étape 2, vous obtenez une erreur liée à la création de l’espace de noms.
Vous pouvez l’ignorer en toute sécurité, l’espace de noms ayant déjà été créé avec la commande `oc new-project`.

Utilisez `azure-disk` en guise de type de stockage.

### <a name="create-an-external-route-to-console"></a>Créer une route externe vers la console

Vous pouvez suivre la documentation ou les instructions ci-dessous si vous préférez la commande oc.
Copiez la définition de route suivante dans un fichier appelé twistlock_route.yaml sur votre ordinateur.
```
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  labels:
    name: console
  name: twistlock-console
  namespace: twistlock
spec:
  port:
    targetPort: mgmt-http
  tls:
    insecureEdgeTerminationPolicy: Redirect
    termination: edge
  to:
    kind: Service
    name: twistlock-console
    weight: 100
  wildcardPolicy: None
```
Exécutez ensuite la commande suivante :
```
oc create -f twistlock_route.yaml
```

Vous pouvez récupérer l’URL affectée à la console Twistlock à l’aide de cette commande : `oc get route twistlock-console -n twistlock`

### <a name="configure-console"></a>Configurer la console

Suivez la documentation de Twistlock.

### <a name="install-defender"></a>Installer un défenseur

Lors de l’exécution de la commande `oc create -f defender.yaml` à l’étape 2, vous obtenez des erreurs liées à la création du rôle de cluster et de la liaison de rôle de cluster.
Vous pouvez les ignorer.

Les défenseurs sont déployés uniquement sur les nœuds de calcul. Vous n’êtes pas obligé de les limiter avec un sélecteur de nœud.
