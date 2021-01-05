---
title: Créer un groupe de serveurs PostgreSQL Hyperscale à l’aide des outils Kubernetes
description: Créer un groupe de serveurs PostgreSQL Hyperscale à l’aide des outils Kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: f447c6028b1750aa96e531a97e7b0861f66a5749
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91761666"
---
# <a name="create-a-postgresql-hyperscale-server-group-using-kubernetes-tools"></a>Créer un groupe de serveurs PostgreSQL Hyperscale à l’aide des outils Kubernetes

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Prérequis

Vous devez avoir déjà créé un [contrôleur de données Azure Arc](./create-data-controller.md).

Pour créer un groupe de serveurs PostgreSQL Hyperscale à l’aide des outils Kubernetes, vous devez avoir installé les outils Kubernetes.  Les exemples de cet article utilisent `kubectl`, mais des approches similaires peuvent être utilisées avec d’autres outils Kubernetes tels que le tableau de bord Kubernetes, `oc`ou `helm` si vous êtes familiarisé avec ces outils et Kubernetes yaml/json.

[Installer l’outil kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## <a name="overview"></a>Vue d’ensemble

Pour créer un groupe de serveurs PostgreSQL Hyperscale, vous devez créer un secret Kubernetes pour stocker votre connexion d’administrateur postgres et votre mot de passe de manière sécurisée, ainsi qu’une ressource personnalisée de groupe de serveurs PostgreSQL Hyperscale basée sur les définitions de ressources personnalisées postgresql-12 ou postgresql-11.

## <a name="create-a-yaml-file"></a>Créer un fichier YAML

Vous pouvez utiliser le [modèle de fichier yaml](https://raw.githubusercontent.com/microsoft/azure_arc/master/arc_data_services/deploy/yaml/postsgresql.yaml) comme point de départ pour créer votre propre fichier yaml de groupe de serveurs PostgreSQL Hyperscale personnalisé.  Téléchargez ce fichier sur votre ordinateur local et ouvrez-le dans un éditeur de texte.  Il est utile d’utiliser un éditeur de texte tel que [VS Code](https://code.visualstudio.com/download) qui prend en charge la mise en surbrillance de la syntaxe et le linting pour les fichiers YAML.

Voici un exemple de fichier YAML :

```yaml
apiVersion: v1
data:
  password: <your base64 encoded password>
kind: Secret
metadata:
  name: example-login-secret
type: Opaque
---
apiVersion: arcdata.microsoft.com/v1alpha1
kind: postgresql-12
metadata:
  generation: 1
  name: example
spec:
  engine:
    extensions:
    - name: citus
  scale:
    shards: 3
  scheduling:
    default:
      resources:
        limits:
          cpu: "4"
          memory: 4Gi
        requests:
          cpu: "1"
          memory: 2Gi
  service:
    type: LoadBalancer
  storage:
    backups:
      className: default
      size: 5Gi
    data:
      className: default
      size: 5Gi
    logs:
      className: default
      size: 1Gi
```

### <a name="customizing-the-login-and-password"></a>Personnalisation de l’identifiant et du mot de passe.
Une clé secrète Kubernetes est stockée sous la forme d’une chaîne encodée en base64, une pour le nom d’utilisateur et l’autre pour le mot de passe.  Vous devez coder en base64 un identifiant et un mot de passe d’administrateur, et les placer à l’emplacement de l’espace réservé à `data.password` et `data.username`.  N’incluez pas les symboles `<` et `>` fournis dans le modèle.

Vous pouvez utiliser un outil en ligne pour coder en base64 le nom d’utilisateur et le mot de passe souhaités, ou vous pouvez utiliser des outils d’interface de ligne de commande intégrés en fonction de votre plateforme.

PowerShell

```console
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('<your string to encode here>'))

#Example
#[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('example'))

```

Linux/macOS

```console
echo '<your string to encode here>' | base64

#Example
# echo 'example' | base64
```

### <a name="customizing-the-name"></a>Personnalisation du nom

Le modèle a la valeur « example » pour l’attribut de nom.  Vous pouvez modifier cela, mais il doit s’agir de caractères qui respectent les normes DNS en matière d’attribution de noms.  Vous devez également modifier le nom du secret pour qu’il corresponde.  Par exemple, si vous modifiez le nom du groupe de serveurs PostgreSQL Hyperscale en « postgres1 », vous devez modifier le nom du secret de « example-login-secret » en « postgres1-login-secret ».

### <a name="customizing-the-engine-version"></a>Personnalisation de la version de moteur

Vous pouvez changer la version de moteur pour qu’elle soit postgresql-11 ou postgresql-12 en modifiant l’attribut `kind`.

### <a name="customizing-the-resource-requirements"></a>Personnalisation des besoins en ressources

Vous pouvez modifier les besoins en ressources (la RAM, les limites principales et les requêtes) le cas échéant.  

> [!NOTE]
> Vous pouvez en apprendre plus sur la [gouvernance des ressources Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes).

Conditions requises pour les requêtes et les limites de ressources :
- La valeur limite des cœurs est **obligatoire** à des fins de facturation.
- Les autres requêtes et limites de ressources sont facultatives.
- La limite et la requête de cœurs doivent être une valeur entière positive, le cas échéant.
- Un cœur au minimum est obligatoire pour la requête de cœurs, le cas échéant.
- Le format de la valeur de mémoire respecte la notation Kubernetes.  

### <a name="customizing-service-type"></a>Personnalisation du type de service

Le type de service peut être modifié en NodePort si vous le souhaitez.  Un numéro de port aléatoire sera attribué.

### <a name="customizing-storage"></a>Personnalisation du stockage

Vous pouvez personnaliser les classes de stockage en fonction de votre environnement.  Si vous n’êtes pas sûr des classes de stockage disponibles, vous pouvez exécuter la commande `kubectl get storageclass` pour les afficher.  Le modèle a une valeur par défaut « default ».  Cela signifie qu’il existe une classe de stockage _nommée_ « default », et non pas qu’une classe de stockage _est_ utilisée par défaut.  Vous pouvez également modifier la taille de votre stockage.  Vous pouvez en apprendre plus sur [la configuration du stockage](./storage-configuration.md).

## <a name="creating-the-postgresql-hyperscale-server-group"></a>Création du groupe de serveurs PostgreSQL Hyperscale

Maintenant que vous avez personnalisé le fichier yaml du groupe de serveurs PostgreSQL Hyperscale, vous pouvez créer le groupe de serveurs PostgreSQL Hyperscale en exécutant la commande suivante :

```console
kubectl create -n <your target namespace> -f <path to your yaml file>

#Example
#kubectl create -n arc -f C:\arc-data-services\postgres.yaml
```


## <a name="monitoring-the-creation-status"></a>Surveillance de l’état de la création

La création du groupe de serveurs PostgreSQL Hyperscale prendra quelques minutes. Vous pouvez surveiller la progression dans une autre fenêtre du terminal à l’aide des commandes suivantes :

> [!NOTE]
>  Les exemples de commandes ci-dessous partent du principe que vous avez créé un groupe de serveurs PostgreSQL Hyperscale nommé « postgres1 » et un espace de noms Kubernetes nommé « arc ».  Si vous avez utilisé un autre espace de noms/nom de groupe de serveurs PostgreSQL Hyperscale, vous pouvez remplacer « arc » et « postgres1 » par vos noms.

```console
kubectl get postgresql-12/postgres1 --namespace arc
```

```console
kubectl get pods --namespace arc
```

Vous pouvez également vérifier l’état de la création de n’importe quel pod en exécutant une commande comme celle qui figure ci-dessous.  C’est particulièrement utile pour résoudre les problèmes.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/postgres1-0 --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Résolution des problèmes de création

Si vous rencontrez des problèmes avec la création, consultez le [Guide de résolution des problèmes](troubleshoot-guide.md).