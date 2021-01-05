---
title: Restaurer l’exemple de base de données AdventureWorks avec PostgreSQL Hyperscale activé par Azure Arc
description: Restaurer l’exemple de base de données AdventureWorks avec PostgreSQL Hyperscale activé par Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 2b51c5ca2295671a30fa6c0aee8d313c4c333900
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930152"
---
# <a name="restore-the-adventureworks-sample-database-to-azure-arc-enabled-postgresql-hyperscale"></a>Restaurer l’exemple de base de données AdventureWorks avec PostgreSQL Hyperscale activé par Azure Arc

[AdventureWorks](/sql/samples/adventureworks-install-configure) est un exemple de base de données contenant une base de données OLTP utilisée dans des tutoriels et pour des exemples. Il est fourni et géré par Microsoft dans le [Dépôt GitHub d’échantillons SQL Server](https://github.com/microsoft/sql-server-samples/tree/master/samples/databases).

Un projet open source a converti la base de données AdventureWorks pour qu’elle soit compatible avec PostgreSQL Hyperscale activé par Azure Arc.
- [Projet d’origine](https://github.com/lorint/AdventureWorks-for-Postgres)
- [Suivre sur le projet qui convertit les fichiers CSV afin qu’ils soient compatibles avec PostgreSQL](https://github.com/NorfolkDataSci/adventure-works-postgres)

Ce document décrit un processus simple pour récupérer l’exemple de base de données AdventureWorks dans votre groupe de serveurs PostgreSQL Hyperscale.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="download-the-adventureworks-backup-file"></a>Télécharger le fichier de sauvegarde AdventureWorks

Téléchargez le fichier .sql AdventureWorks dans votre conteneur du groupe de serveurs PostgreSQL Hyperscale. Dans cet exemple, nous allons utiliser la commande `kubectl exec` pour exécuter à distance une commande dans le conteneur du groupe de serveurs PostgreSQL Hyperscale afin de télécharger le fichier dans le conteneur. Vous pouvez télécharger ce fichier à partir de n’importe quel emplacement accessible par `curl`. Utilisez cette même méthode si vous avez d’autres fichiers de sauvegarde de base de données que vous souhaitez extraire dans le conteneur du groupe de serveurs PostgreSQL Hyperscale. Une fois que le fichier se trouve dans le conteneur du groupe de serveurs PostgreSQL Hyperscale, il est facile de créer la base de données et le schéma, ainsi que de renseigner les données.

Exécutez une commande comme celle-ci pour télécharger le fichier en remplaçant la valeur du nom de pod et du nom d’espace de noms avant de l’exécuter :

> [!NOTE]
>  Pour télécharger le fichier à partir de GitHub, votre conteneur doit disposer d’une connexion Internet sur le port 443.

> [!NOTE]
>  Utilisez le nom pod du nœud coordinateur du groupe de serveurs Postgres Hyperscale. Son nom est <server group name>-0.  Si vous n’êtes pas sûr du nom de pod, exécutez la commande `kubectl get pod`

```console
kubectl exec <PostgreSQL pod name> -n <namespace name> -c postgres  -- /bin/bash -c "cd /tmp && curl -k -O https://raw.githubusercontent.com/microsoft/azure_arc/master/azure_arc_data_jumpstart/aks/arm_template/postgres_hs/AdventureWorks.sql"

#Example:
#kubectl exec postgres02-0 -n arc -c postgres -- /bin/bash -c "cd /tmp && curl -k -O https://raw.githubusercontent.com/microsoft/azure_arc/master/azure_arc_data_jumpstart/aks/arm_template/postgres_hs/AdventureWorks.sql"
```

## <a name="step-2-restore-the-adventureworks-database"></a>Étape 2 : Restaurer la base de données AdventureWorks

Vous pouvez également exécuter une commande exec kubectl pour utiliser l’outil d’interface de ligne de commande psql qui est inclus dans les conteneurs de groupe de serveurs PostgreSQL Hyperscale afin de créer et charger la base de données.

Exécutez une commande comme celle-ci pour créer la base de données vide, en remplaçant la valeur du nom de pod et du nom d’espace de noms avant de l’exécuter.

```console
kubectl exec <PostgreSQL pod name> -n <namespace name> -c postgres -- psql --username postgres -c 'CREATE DATABASE "adventureworks";'

#Example
#kubectl exec postgres02-0 -n arc -c postgres -- psql --username postgres -c 'CREATE DATABASE "adventureworks";'
```

Exécutez ensuite une commande comme celle-ci pour restaurer la base de données en remplaçant la valeur du nom de pod et du nom d’espace de noms avant de l’exécuter.

```console
kubectl exec <PostgreSQL pod name> -n <namespace name> -c postgres -- psql --username postgres -d adventureworks -f /tmp/AdventureWorks.sql

#Example
#kubectl exec postgres02-0 -n arc -c postgres -- psql --username postgres -d adventureworks -f /tmp/AdventureWorks.sql
```


> **Remarque : Vous ne verrez pas beaucoup d’avantages en matière de performances résultant de l’exécution sur PostgreSQL Hyperscale activé par Azure Arc jusqu’à effectuer un scale-out et partitionner/distribuer les données/tables entre les nœuds Worker de votre groupe de serveurs PostgreSQL Hyperscale. Consultez [Étapes suivantes suggérées](#suggested-next-steps).**

## <a name="suggested-next-steps"></a>Étapes suivantes suggérées
- Lisez les concepts et les guides pratiques d’Azure Database pour PostgreSQL Hyperscale pour distribuer vos données sur plusieurs nœuds PostgreSQL Hyperscale et tirer parti de toute la puissance d’Azure Database pour PostgreSQL Hyperscale. :
    * [Nœuds et tables](../../postgresql/concepts-hyperscale-nodes.md)
    * [Déterminer le type d’application](../../postgresql/concepts-hyperscale-app-type.md)
    * [Choisir une colonne de distribution](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocalisation de table](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribuer et modifier des tables](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Concevoir une base de données multilocataire](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Concevoir un tableau de bord d’analytique en temps réel](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

   > \* Dans les documents ci-dessus, ignorez les sections **Se connecter au portail Azure** et **Créer un serveur Azure Database pour PostgreSQL - Hyperscale (Citus)** . Implémentez les étapes restantes dans votre déploiement Azure Arc. Ces sections sont spécifiques à Azure Database pour PostgreSQL Hyperscale (Citus) proposé en tant que service PaaS dans le cloud Azure, mais les autres parties des documents s’appliquent directement à votre PostgreSQL Hyperscale activé pour Azure Arc.

- [Effectuer un scale-out de votre groupe de serveurs Azure Database pour PostgreSQL Hyperscale](scale-out-postgresql-hyperscale-server-group.md)
