---
title: Supprimer un service SQL Managed Instance activé par Azure Arc
description: Supprimer un service SQL Managed Instance activé par Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: e531349e8f404380d9f0601caa3b66557c297062
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930785"
---
# <a name="delete-azure-arc-enabled-sql-managed-instance"></a>Supprimer un service SQL Managed Instance activé par Azure Arc
Cet article explique comment supprimer un service SQL Managed Instance activé par Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="view-existing-azure-arc-enabled-sql-managed-instances"></a>Afficher les instances gérées SQL activées par Azure Arc
Pour afficher les instances SQL Managed Instance, exécutez la commande suivante :

```console
azdata arc sql mi list
```

Le résultat suivant doit ressembler à ce qui suit :

```console
Name    Replicas    ServerEndpoint    State
------  ----------  ----------------  -------
demo-mi 1/1         10.240.0.4:32023  Ready
```

## <a name="delete-a-azure-arc-enabled-sql-managed-instance"></a>Supprimer un service SQL Managed Instance activé par Azure Arc
Pour supprimer un service SQL Managed Instance, exécutez la commande suivante :

```console
azdata arc sql mi delete -n <NAME_OF_INSTANCE>
```

Le résultat suivant doit ressembler à ce qui suit :

```console
# azdata arc sql mi delete -n demo-mi
Deleted demo-mi from namespace arc
```

## <a name="reclaim-the-kubernetes-persistent-volume-claims-pvcs"></a>Récupérer les revendications de volume persistant (PVC) Kubernetes

La suppression d’un service SQL Managed Instance n’a pas pour effet de supprimer ses [PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) associés. C'est la procédure normale. L’objectif est d’aider l’utilisateur à accéder aux fichiers de base de données en cas de suppression accidentelle de l’instance. La suppression des PVC n’est pas obligatoire. Elle est cependant recommandée. Si vous ne récupérez pas ces PVC, vous finirai par obtenir des erreurs, car votre cluster Kubernetes estimera que l’espace disque est insuffisant. Pour récupérer les PVC, procédez comme suit :

### <a name="1-list-the-pvcs-for-the-server-group-you-deleted"></a>1. Répertoriez les PVC du groupe de serveurs que vous avez supprimés
Pour répertorier les PVC, exécutez la commande suivante :
```console
kubectl get pvc
```

Dans l’exemple ci-dessous, notez les PVC pour les instances gérées SQL que vous avez supprimées.
```console
# kubectl get pvc -n arc

NAME                    STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
data-demo-mi-0        Bound     pvc-1030df34-4b0d-4148-8986-4e4c20660cc4   5Gi        RWO            managed-premium   13h
logs-demo-mi-0        Bound     pvc-11836e5e-63e5-4620-a6ba-d74f7a916db4   5Gi        RWO            managed-premium   13h
```

### <a name="2-delete-each-of-the-pvcs"></a>2. Supprimez chacun des PVC
Supprimez les PVC de données et de journal pour chacune des instances gérées SQL que vous avez supprimées.
Le format général de cette commande est le suivant : 
```console
kubectl delete pvc <name of pvc>
```

Exemple :
```console
kubectl delete pvc data-demo-mi-0 -n arc
kubectl delete pvc logs-demo-mi-0 -n arc
```

Chacune de ces commandes kubectl confirme la réussite de la suppression du PVC. Exemple :
```console
persistentvolumeclaim "data-demo-mi-0" deleted
persistentvolumeclaim "logs-demo-mi-0" deleted
```
  

> [!NOTE]
> Comme indiqué, le fait de ne pas supprimer les PVC peut finalement conduire votre cluster Kubernetes à une situation où il lèvera des erreurs. Certaines de ces erreurs peuvent inclure l’incapacité de se connecter à votre cluster Kubernetes avec azdata, car les pods peuvent en être exclus en raison de ce problème de stockage (comportement Kubernetes normal).
>
> Par exemple, vous pouvez voir des messages dans les journaux, similaires à ce qui suit :  
> - Annotations:    microsoft.com/ignore-pod-health: true  
> - État :         Échec  
> - Motif :         Supprimé  
> - Message :        La nœud manquait de ressource : éphémère-storage. Le contrôleur de conteneurs utilisait 16372 Ki, ce qui dépasse sa demande de 0.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les [Fonctionnalités et capacités de SQL Managed Instance activé par Azure Arc](managed-instance-features.md)

[Commencer en créant un contrôleur de données](create-data-controller.md)

Vous avez déjà créé un contrôleur de données ? [Créer un service SQL Managed Instance activé par Azure Arc](create-sql-managed-instance.md)