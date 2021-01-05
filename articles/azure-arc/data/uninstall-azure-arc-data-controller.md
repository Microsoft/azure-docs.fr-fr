---
title: Supprimer le contrôleur de données Azure Arc
description: Supprimer le contrôleur de données Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: a040200c5746defcaee84a951521d5919c0c4d28
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91660674"
---
# <a name="delete-azure-arc-data-controller"></a>Supprimer le contrôleur de données Azure Arc

L’article suivant explique comment supprimer un contrôleur de données Azure Arc.

Avant de continuer, vérifiez que tous les services de données qui ont été créés sur le contrôleur de données sont supprimés comme suit :

## <a name="log-in-to-the-data-controller"></a>Se connecter au contrôleur de données

Connectez-vous au contrôleur de données que vous souhaitez supprimer :

```
azdata login
```

## <a name="list--delete-existing-data-services"></a>Répertorier et supprimer des services de données existants

Exécutez la commande suivante pour vérifier si des instances gérées SQL sont créées :

```
azdata arc sql mi list
```

Pour chaque instance gérée SQL figurant dans la liste ci-dessus, exécutez la commande delete comme suit :

```
azdata arc sql mi delete -n <name>
# for example: azdata arc sql mi delete -n sqlinstance1
```

De même, pour vérifier les instances PostgreSQL Hyperscale, exécutez la commande suivante :

```
azdata arc postgres server list
```

Et, pour chaque instance PostgreSQL Hyperscale, exécutez la commande delete comme suit :
```
azdata arc postgres server delete -n <name>
# for example: azdata arc postgres server delete -n pg1
```

## <a name="delete-controller"></a>Supprimer le contrôleur

Une fois la totalité des instances gérées SQL et instances PostgreSQL Hyperscale supprimées, vous pouvez supprimer le contrôleur de données comme suit :

```
azdata arc dc delete -n <name> -ns <namespace>
# for example: azdata arc dc delete -ns arc -n arcdc
```

### <a name="remove-sccs-red-hat-openshift-only"></a>Supprimer des contraintes de contexte de sécurité (Red Hat OpenShift uniquement)

```console
oc adm policy remove-scc-from-user privileged -z default -n arc
oc adm policy remove-scc-from-user anyuid     -z default -n arc
```

### <a name="delete-cluster-level-objects"></a>Supprimer les objets de niveau cluster

Outre les objets délimités par l’espace de noms, si vous souhaitez également supprimer les objets de niveau cluster tels que les CRD `clusterroles` et `clusterrolebindings`, exécutez les commandes suivantes :

```console
# Cleanup azure arc data service artifacts
#Delete CRDs
kubectl delete crd datacontrollers.arcdata.microsoft.com 
kubectl delete crd sqlmanagedinstances.sql.arcdata.microsoft.com 
kubectl delete crd postgresql-11s.arcdata.microsoft.com 
kubectl delete crd postgresql-12s.arcdata.microsoft.com
#Delete clusterrole
kubectl delete clusterrole <namespace>:cr-arc-metricsdc-reader
#For example: kubectl delete clusterrole arc:cr-arc-metricsdc-reader
#Delete rolebinding
kubectl delete clusterrolebinding <namespace>:crb-arc-metricsdc-reader
#For example: kubectl delete clusterrolebinding arc:crb-arc-metricsdc-reader
```

### <a name="optionally-delete-the-azure-arc-data-controller-namespace"></a>Si vous le souhaitez, supprimez l’espace de noms du contrôleur de données Azure Arc


```console
kubectl delete ns <nameSpecifiedDuringCreation>
# for example: kubectl delete ns arc
```

## <a name="next-steps"></a>Étapes suivantes

[Que sont les services de données activés pour Azure Arc ?](overview.md)
