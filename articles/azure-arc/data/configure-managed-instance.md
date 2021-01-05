---
title: Configurer une instance gérée SQL activée par Azure Arc
description: Configurer une instance gérée SQL activée par Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 6e2443014f6788504a11784945078187a5a72de4
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92311122"
---
# <a name="configure-azure-arc-enabled-sql-managed-instance"></a>Configurer une instance gérée SQL activée par Azure Arc

Cet article explique comment configurer une instance gérée SQL activée par Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="configure-resources"></a>Configuration des ressources

### <a name="configure-using-azure-data-cli-azdata"></a>Configuration avec [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

Vous pouvez modifier la configuration des instances SQL Managed Instance activées par Azure Arc avec l’interface [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]. Exécutez la commande suivante pour afficher les options de configuration. 

```
azdata arc sql mi edit --help
```

L’exemple suivant définit les demandes et les limites de mémoire et de cœur de processeur.

```
azdata arc sql mi edit --cores-limit 4 --cores-request 2 --memory-limit 4Gi --memory-request 2Gi -n <NAME_OF_SQL_MI>
```

Pour voir les modifications apportées à l’instance gérée SQL, vous pouvez utiliser les commandes suivantes pour afficher le fichier yaml de configuration :

```
azdata arc sql mi show -n <NAME_OF_SQL_MI>
```

## <a name="configure-server-options"></a>Configurer les options de serveur

Vous pouvez définir les paramètres de configuration du serveur pour une instance gérée SQL activée par Azure Arc après la création. Cet article explique comment configurer des paramètres tels que l’activation ou la désactivation de l’agent mssql, et activer des indicateurs de trace spécifiques pour les scénarios de résolution des problèmes.

Pour modifier l’un de ces paramètres, procédez comme suit :

1. Créez un fichier `mssql-custom.conf` personnalisé qui comprend les paramètres ciblés. L’exemple suivant active SQL Agent et l’indicateur de trace 1204 :

   ```
   [sqlagent]
   enabled=true
   
   [traceflag]
   traceflag0 = 1204
   ```

1. Copiez le fichier `mssql-custom.conf` dans `/var/opt/mssql` dans le conteneur `mssql-miaa` du pod `master-0`. Remplacez `<namespaceName>` par le nom du cluster Big Data.

   ```bash
   kubectl cp mssql-custom.conf master-0:/var/opt/mssql/mssql-custom.conf -c mssql-server -n <namespaceName>
   ```

1. Redémarrez l’instance SQL Server.  Remplacez `<namespaceName>` par le nom du cluster Big Data.

   ```bash
   kubectl exec -it master-0  -c mssql-server -n <namespaceName> -- /bin/bash
   supervisorctl restart mssql-server
   exit
   ```


**Limitations connues**
- Les étapes ci-dessus nécessitent des autorisations d’administrateur de cluster Kubernetes
- Cela est susceptible de changer tout au long de la période de préversion
