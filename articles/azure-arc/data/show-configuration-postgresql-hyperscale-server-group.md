---
title: Afficher la configuration d’un groupe de serveurs Arc enabled PostgreSQL Hyperscale
titleSuffix: Azure Arc enabled data services
description: Afficher la configuration d’un groupe de serveurs Arc enabled PostgreSQL Hyperscale
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: a268cd6b2fa3da6846554e3d1b170298abec7f18
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93279399"
---
# <a name="show-the-configuration-of-an-arc-enabled-postgresql-hyperscale-server-group"></a>Afficher la configuration d’un groupe de serveurs Arc enabled PostgreSQL Hyperscale

Cet article explique comment afficher la configuration de vos groupes de serveurs. Pour cela, il anticipe certaines questions que vous pourriez vous poser et y répond. Il peut parfois y avoir plusieurs réponses valides. Cet article présente les réponses les plus courantes ou les plus utiles. Il regroupe ces questions par thème :

- du point de vue de Kubernetes
- du point de vue des services de données activés pour Azure Arc

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="from-a-kubernetes-point-of-view"></a>Du point de vue de Kubernetes

### <a name="how-many-pods-are-used-by-azure-arc-enabled-postgresql-hyperscale"></a>Combien de pods sont utilisés par Azure Arc enabled PostgreSQL Hyperscale ?

Listez les ressources Kubernetes de type Postgres. Exécutez la commande suivante :

```console
kubectl get postgresqls [-n <namespace name>]
```

La sortie de cette commande montre la liste des groupes de serveurs créés. Pour chacun, il indique le nombre de pods. Exemple :

```output
NAME                                             STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgresql-12.arcdata.microsoft.com/postgres01   Ready   3/3          10.0.0.4:30499      6h34m
postgresql-12.arcdata.microsoft.com/postgres02   Ready   3/3          10.0.0.4:31066      6d7h
```

Cet exemple montre que 2 groupes de serveurs sont créés et que chacun s’exécute sur 3 pods (1 coordinateur + 2 workers). Cela signifie que les groupes de serveurs créés dans ce contrôleur de données Azure Arc utilisent 6 pods.

### <a name="what-pods-are-used-by-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Quels pods sont utilisés par Azure Arc enabled PostgreSQL Hyperscale ?

Exécutez :

```console
kubectl get pods [-n <namespace name>]
```

Ceci retourne la liste des pods. Vous voyez les pods utilisés par vos groupes de serveurs d’après les noms que vous avez donnés à ces groupes de serveurs. Exemple :

```console 
NAME                 READY   STATUS    RESTARTS   AGE
bootstrapper-vdltm   1/1     Running   0          6d8h
control-h6kc9        2/2     Running   0          6d8h
controldb-0          2/2     Running   0          6d8h
controlwd-96sbn      1/1     Running   0          6d8h
logsdb-0             1/1     Running   0          6d8h
logsui-7wkg2         1/1     Running   0          6d8h
metricsdb-0          1/1     Running   0          6d8h
metricsdc-28ffl      1/1     Running   0          6d8h
metricsui-k7qsh      1/1     Running   0          6d8h
mgmtproxy-gd84z      2/2     Running   0          6d8h
postgres01-0         3/3     Running   0          6h50m
postgres01-1         3/3     Running   0          6h50m
postgres01-2         3/3     Running   0          6h50m
postgres02-0         3/3     Running   0          22h
postgres02-1         3/3     Running   0          22h
postgres02-2         3/3     Running   0          22h
```

Dans cet exemple, les 6 pods qui hébergent les deux groupes de serveurs créés sont :
- `postgres01-0`
- `postgres01-1`
- `postgres01-2`
- `postgres02-0`
- `postgres02-1`
- `postgres02-2`  

### <a name="what-server-group-pod-is-used-for-what-role-the-server-group"></a>Quel pod du groupe de serveurs est utilisé pour quel rôle du groupe de serveurs ?

Les noms des pods avec le suffixe `-0` représentent un nœud coordinateur. Les noms des pods avec le suffixe `-x` et un nombre égal ou supérieur à 1 représentent un nœud worker. Dans l’exemple ci-dessus :
- Les coordinateurs sont : `postgres01-0`, `postgres02-0`
- Les workers sont : `postgres01-2`, `postgres01-2` `postgres02-1`, `postgres02-2`

### <a name="what-is-the-status-of-the-pods"></a>Quel est l’état des pods ?

Exécutez `kubectl get pods` et examinez la colonne `STATUS`.

### <a name="what-persistent-volume-claims-pvcs-are-being-used"></a>Quelles revendications de volume persistant sont utilisées ? 

Pour comprendre quelles revendications de volume persistant sont utilisées, et lesquelles sont utilisées spécifiquement pour les données, les journaux et les sauvegardes, exécutez : 

```console
kubectl get pvc [-n <namespace name>]
```

Par défaut, le préfixe du nom d’une revendication de volume persistant indique son utilisation :

- `backups-`... : est une revendication de volume persistant utilisée pour les sauvegardes
- `data-`... : est une revendication de volume persistant utilisée pour les fichiers de données
- `logs-`... : est une revendication de volume persistant utilisée pour les journaux de transactions/les fichiers WAL

Exemple :

```output
NAME                                            STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS    AGE
backups-few7hh0k4npx9phsiobdc3hq-postgres01-0   Bound    local-pv-485e37db   1938Gi     RWO            local-storage   6d6h
backups-few7hh0k4npx9phsiobdc3hq-postgres01-1   Bound    local-pv-9d3d4a15   1938Gi     RWO            local-storage   6d6h
backups-few7hh0k4npx9phsiobdc3hq-postgres01-2   Bound    local-pv-7b8dd819   1938Gi     RWO            local-storage   6d6h
...
data-few7hh0k4npx9phsiobdc3hq-postgres01-0      Bound    local-pv-3c1a8cc5   1938Gi     RWO            local-storage   6d6h
data-few7hh0k4npx9phsiobdc3hq-postgres01-1      Bound    local-pv-8303ab19   1938Gi     RWO            local-storage   6d6h
data-few7hh0k4npx9phsiobdc3hq-postgres01-2      Bound    local-pv-55572fe6   1938Gi     RWO            local-storage   6d6h
...
logs-few7hh0k4npx9phsiobdc3hq-postgres01-0      Bound    local-pv-5e852b76   1938Gi     RWO            local-storage   6d6h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-1      Bound    local-pv-55d309a7   1938Gi     RWO            local-storage   6d6h
logs-few7hh0k4npx9phsiobdc3hq-postgres01-2      Bound    local-pv-5ccd02e6   1938Gi     RWO            local-storage   6d6h
...
```


## <a name="from-an-azure-arc-enabled-data-services-point-of-view"></a>Du point de vue des services de données activés pour Azure Arc :

* Combien de groupes de serveurs sont créés dans un contrôleur de données Arc ?
* Quels sont leurs noms ?
* Combien de nœuds worker utilisent-ils ?
* Quelle version de Postgres exécutent-ils ?

Utilisez une des commandes suivantes.
- **Avec kubectl :**

   ```console
   kubectl get postgresqls [-n <namespace name>]
   ``` 

   Par exemple, elle produit la sortie ci-dessous, où chaque ligne est un `servergroup`. La structure du nom dans l’affichage ci-dessous est formée comme suit :

   `<Name-Of-Custom-Resource-Definition>`/`<Server-Group-Name>`

   ```output
   NAME                                             STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
   postgresql-12.arcdata.microsoft.com/postgres01   Ready   3/3          10.0.0.4:30499      7h15m
   postgresql-12.arcdata.microsoft.com/postgres02   Ready   3/3          10.0.0.4:31066      6d7h
   ```

   La sortie ci-dessus montre 2 groupes de serveurs qui utilisent Postgres version 12. Si la version de Postgres était 11, le nom du CRD serait alors postgresql-11.arcdata.microsoft.com.

   Chacun d’eux s’exécute sur 3 nœuds/pods : 1 coordinateur et 2 workers.

- **Avec azdata :**

Exécutez la commande suivante. La sortie affiche des informations similaires à ce que kubectl affiche :

   ```console
   azdata arc postgres server list

   `output
   Name        State    Workers
   ----------  -------  ---------
   postgres01  Ready    2
   postgres02  Ready    2
   ```


### <a name="how-much-memory-and-vcores-are-being-used-and-what-extensions-were-created-for-a-group"></a>Quelle quantité de mémoire et de vCores est utilisée, et quelles extensions ont été créées pour un groupe ?

Exécutez une des commandes suivantes.

**Avec Kubectl :**

Utilisez kubectl pour décrire une ressource Postgres. Pour cela, vous avez besoin de son type (nom de la ressource Kubernetes (CRD) pour la version de Postgres correspondante, comme indiqué ci-dessus) et du nom du groupe de serveurs.
Le format général de cette commande est :

```console
kubectl describe <CRD name>/<server group name> [-n <namespace name>]
```

Exemple :

```console
kubectl describe postgresql-12/postgres02
```

Cette commande montre la configuration du groupe de serveurs :

```output
Name:         postgres02
Namespace:    arc
Labels:       <none>
Annotations:  <none>
API Version:  arcdata.microsoft.com/v1alpha1
Kind:         postgresql-12
Metadata:
  Creation Timestamp:  2020-08-31T21:01:07Z
  Generation:          10
  Resource Version:    569516
  Self Link:           /apis/arcdata.microsoft.com/v1alpha1/namespaces/arc/postgresql-12s/postgres02
  UID:                 8a9cd118-361b-4a2e-8a9d-5f9257bf6abb
Spec:
  Engine:
    Extensions:
      Name:  citus
      Name:  pg_stat_statements
  Scale:
    Workers:  2
  Scheduling:
    Default:
      Resources:
        Limits:
          Cpu:     4
          Memory:  1024Mi
        Requests:
          Cpu:     1
          Memory:  512Mi
  Service:
    Type:  NodePort
  Storage:
    Data:
      Class Name:  local-storage
      Size:        5Gi
    Logs:
      Class Name:  local-storage
      Size:        5Gi
Status:
  External Endpoint:  10.0.0.4:31066
  Ready Pods:         3/3
  State:              Ready
Events:               <none>
```

>[!NOTE]
>Avant la version d’octobre 2020, `Workers` s’appelait `Shards` dans l’exemple précédent. Pour plus d’informations, consultez [Notes de publication – Services de données activés par Azure Arc (préversion)](release-notes.md).

Détaillons quelques points d’intérêt spécifiques dans la description du `servergroup` présentée ci-dessus. Que nous dit-elle de ce groupe de serveurs ?

- Il utilise la version 12 de Postgres : 
   > ```json
   > Kind:         `postgresql-12`
   > ```
- Il a été créé au cours du mois d’août 2020 :
   > ```json
   > Creation Timestamp:  `2020-08-31T21:01:07Z`
   > ```
- Deux extensions Postgres ont été créées dans ce groupe de serveurs : `citus` et `pg_stat_statements`
   > ```json
   > Engine:
   >    Extensions:
   >      Name:  `citus`
   >      Name:  `pg_stat_statements`
   > ```
- Il utilise deux nœuds worker.
   > ```json
   > Scale:
   >    Workers:  `2`
   > ```
- Il a la garantie d’utiliser 1 processeur/vCore et 512 Mo de RAM par nœud. Il va utiliser plus de 4 processeurs/vCores et 1 024 Mo de mémoire :
   > ```json
   > Scheduling:
   >    Default: 
   >      Resources:
   >        Limits:
   >          Cpu:     4
   >          Memory:  1024Mi
   >        Requests:
   >          Cpu:     1
   >          Memory:  512Mi
   > ```
 - Elle est disponible pour les requêtes et n’a pas de problème. Tous les nœuds sont opérationnels :
   > ```json
   > Status:
   >  ...
   >  Ready Pods:         3/3
   >  State:              Ready
   > ```

**Avec azdata :**

Le format général de la commande est :

```console
azdata arc postgres server show -n <server group name>
```

Exemple :

```console
azdata arc postgres server show -n postgres02
```

Retourne la sortie ci-dessous dans un format et avec un contenu très similaire à celui retourné par kubectl.

```console
{
  "apiVersion": "arcdata.microsoft.com/v1alpha1",
  "kind": "postgresql-12",
  "metadata": {
    "creationTimestamp": "2020-08-31T21:01:07Z",
    "generation": 10,
    "name": "postgres02",
    "namespace": "arc",
    "resourceVersion": "569516",
    "selfLink": "/apis/arcdata.microsoft.com/v1alpha1/namespaces/arc/postgresql-12s/postgres02",
    "uid": "8a9cd118-361b-4a2e-8a9d-5f9257bf6abb"
  },
  "spec": {
    "engine": {
      "extensions": [
        {
          "name": "citus"
        },
        {
          "name": "pg_stat_statements"
        }
      ]
    },
    "scale": {
      "workers": 2
    },
    "scheduling": {
      "default": {
        "resources": {
          "limits": {
            "cpu": "4",
            "memory": "1024Mi"
          },
          "requests": {
            "cpu": "1",
            "memory": "512Mi"
          }
        }
      }
    },
    "service": {
      "type": "NodePort"
    },
    "storage": {
      "data": {
        "className": "local-storage",
        "size": "5Gi"
      },
      "logs": {
        "className": "local-storage",
        "size": "5Gi"
      }
    }
  },
  "status": {
    "externalEndpoint": "10.0.0.4:31066",
    "readyPods": "3/3",
    "state": "Ready"
  }
}
```

## <a name="next-steps"></a>Étapes suivantes
- [Découvrir les concepts d’Azure Arc enabled PostgreSQL Hyperscale](concepts-distributed-postgres-hyperscale.md)
- [Découvrir le scale-out (ajout des nœuds worker) d’un groupe de serveurs](scale-out-postgresql-hyperscale-server-group.md)
- [Découvrir le scale-up/scale-down (augmentation ou réduction de la mémoire et/ou des vCores) d’un groupe de serveurs](scale-up-down-postgresql-hyperscale-server-group-using-cli.md)
- [Explorer la configuration du stockage](storage-configuration.md)
- [Lisez comment surveiller une instance de base de données](monitor-grafana-kibana.md)
- [Utiliser les extensions PostgreSQL dans votre groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale](using-extensions-in-postgresql-hyperscale-server-group.md)
- [Configurer la sécurité pour votre groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale](configure-security-postgres-hyperscale.md)
