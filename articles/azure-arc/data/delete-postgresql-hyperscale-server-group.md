---
title: Créer un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc
description: Créer un groupe de serveurs Postgres Hyperscale activé par Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: ac620909996b03a97a311e5f06c31d6dab8f1a60
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92218644"
---
# <a name="delete-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Créer un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc

Ce document décrit les étapes permettant de supprimer un groupe de serveurs de votre installation Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="delete-the-server-group"></a>Supprimer le groupe de serveurs

À titre d’exemple, considérons que nous voulons supprimer l’instance _postgres01_ de l’installation ci-dessous :

```console
azdata arc postgres server list
Name        State    Workers
----------  -------  ---------
postgres01  Ready    3
```

Le format général de la commande delete est le suivant :
```console
azdata arc postgres server delete -n <server group name>
```
Pour plus d’informations sur la commande delete, exécutez la commande suivante :
```console
azdata arc postgres server delete --help
```

### <a name="delete-the-server-group-used-in-this-example"></a>Supprimer le groupe de serveurs utilisé dans cet exemple

```console
azdata arc postgres server delete -n postgres01
```

## <a name="reclaim-the-kubernetes-persistent-volume-claims-pvcs"></a>Récupérer les revendications de volume persistant (PVC) Kubernetes

La suppression d’un groupe de serveurs ne supprime pas les [PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) associés. C'est la procédure normale. L’objectif est d’aider l’utilisateur à accéder aux fichiers de base de données en cas de suppression accidentelle de l’instance. La suppression des PVC n’est pas obligatoire. Toutefois, elle est recommandé. Si vous ne récupérez pas ces PVC, vous obtiendrez finalement des erreurs, car votre cluster Kubernetes estimera que l’espace disque est insuffisant. Pour récupérer les PVC, procédez comme suit :

### <a name="1-list-the-pvcs-for-the-server-group-you-deleted"></a>1. Répertoriez les PVC du groupe de serveurs que vous avez supprimés

Pour répertorier les PVC, exécutez la commande suivante :

```console
kubectl get pvc [-n <namespace name>]
```

Celle-ci retourne la liste des PVC, en particulier les PVC du groupe de serveurs que vous avez supprimés. Exemple :

```output
kubectl get pvc
NAME                                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-few7hh0k4npx9phsiobdc3hq-postgres01-0   Bound    pvc-72ccc225-dad0-4dee-8eae-ed352be847aa   5Gi        RWO            default        2d18h
data-few7hh0k4npx9phsiobdc3hq-postgres01-1   Bound    pvc-ce6f0c51-faed-45ae-9472-8cdf390deb0d   5Gi        RWO            default        2d18h
data-few7hh0k4npx9phsiobdc3hq-postgres01-2   Bound    pvc-5a863ab9-522a-45f3-889b-8084c48c32f8   5Gi        RWO            default        2d18h
data-few7hh0k4npx9phsiobdc3hq-postgres01-3   Bound    pvc-00e1ace3-1452-434f-8445-767ec39c23f2   5Gi        RWO            default        2d15h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-0   Bound    pvc-8b810f4c-d72a-474a-a5d7-64ec26fa32de   5Gi        RWO            default        2d18h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-1   Bound    pvc-51d1e91b-08a9-4b6b-858d-38e8e06e60f9   5Gi        RWO            default        2d18h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-2   Bound    pvc-8e5ad55e-300d-4353-92d8-2e383b3fe96e   5Gi        RWO            default        2d18h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-3   Bound    pvc-f9e4cb98-c943-45b0-aa07-dd5cff7ea585   5Gi        RWO            default        2d15h
```
Il y a 8 PVC pour ce groupe de serveurs.

### <a name="2-delete-each-of-the-pvcs"></a>2. Supprimez chacun des PVC

Supprimez les données et consignez les PVC pour chacun des nœuds PostgreSQL Hyperscale (coordinateur et Workers) du groupe de serveurs, que vous avez supprimés.

Le format général de cette commande est le suivant : 

```console
kubectl delete pvc <name of pvc>  [-n <namespace name>]
```

Exemple :

```console
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-0
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-1
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-2
kubectl delete pvc data-few7hh0k4npx9phsiobdc3hq-postgres01-3
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-0
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-1
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-2
kubectl delete pvc logs-few7hh0k4npx9phsiobdc3hq-postgres01-3
```

Chacune de ces commandes kubectl confirme la réussite de la suppression du PVC. Exemple :

```output
persistentvolumeclaim "data-postgres01-0" deleted
```
  

>[!NOTE]
> Comme indiqué, le fait de ne pas supprimer les PVC peut finalement conduire votre cluster Kubernetes à une situation où il lèvera des erreurs. Certaines de ces erreurs peuvent inclure l’incapacité de se connecter à votre cluster Kubernetes avec azdata, car les pods peuvent en être exclus en raison de ce problème de stockage (comportement Kubernetes normal).
>
> Par exemple, vous pouvez voir des messages dans les journaux, similaires à ce qui suit :  
> ```output
> Annotations:    microsoft.com/ignore-pod-health: true  
> Status:         Failed  
> Reason:         Evicted  
> Message:        The node was low on resource: ephemeral-storage. Container controller was using 16372Ki, which exceeds its request of 0.
> ```
    
## <a name="next-step"></a>Étape suivante
Créer [PostgreSQL Hyperscale avec Azure Arc](create-postgresql-hyperscale-server-group.md)
