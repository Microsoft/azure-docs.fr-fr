---
title: Changer le port PostgreSQL
description: Changez le port sur lequel le groupe de serveurs PostgreSQL Hyperscale compatible avec Azure Arc écoute.
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 11/02/2020
ms.topic: how-to
ms.openlocfilehash: 45424408c790e4921be48464818c55fe74313fd3
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93328626"
---
# <a name="change-the-port-on-which-the-server-group-is-listening"></a>Changer le port sur lequel le groupe de serveurs écoute 

Le changement de port est une opération de modification standard du groupe de serveurs. Pour changer de port, exécutez la commande suivante :
```console
 azdata arc postgres server edit -n <server group name> --port <desired port number>
```

Par exemple, supposons que le nom de votre groupe de serveurs est _postgres01_ et que vous souhaitez qu’il écoute sur le port _866_. Exécutez ensuite la commande suivante :
```console
 azdata arc postgres server edit -n postgres01 --port 866
```

## <a name="verify-that-the-port-was-changed"></a>Vérifiez que le port a changé

Pour vérifier que le port a changé, exécutez la commande suivante pour afficher la configuration de votre groupe de serveurs :
```console
azdata arc postgres server show -n <server group name>
```

Dans la sortie de cette commande, examinez le numéro de port affiché pour l’élément « port » dans la section « service » des spécifications de votre groupe de serveurs.
Vous pouvez également vérifier dans l’externalEndpoint de l’élément de la section État des spécifications de votre groupe de serveurs que l’adresse IP est suivie par le numéro de port que vous avez configuré.

À titre d’illustration, si nous continuons l’exemple ci-dessus, vous devez exécuter la commande :
```console
azdata arc postgres server show -n postgres01
```

et vous verrez le port 866 désigné ici :

```console
"service": {
      "port": 866,
      "type": "LoadBalancer"
    },
```
et ici

```console
"status": {
    "externalEndpoint": "12.678.345.09:866",
    "logSearchDashboard": "https://12.345.678.90:30777/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:postgres01'))",
    "metricsDashboard": "https://12.345.678.90:30777/grafana/d/postgres-metrics?var-Namespace=arc&var-Name=postgres01",
    "readyPods": "3/3",
    "state": "Ready"
  }
```
## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur la [procédure de connexion à votre groupe de serveurs](get-connection-endpoints-and-connection-strings-postgres-hyperscale.md).
- En savoir plus sur la procédure de configuration d’autres aspects de votre groupe de serveurs dans la section Procédures\Gérer\Configurer et mettre à l'échelle de la documentation.
