---
title: Migrer des données d’une base de données PostgreSQL vers un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc
titleSuffix: Azure Arc enabled database services
description: Migrer des données d’une base de données PostgreSQL vers un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 521fd61f18d6673e21c23dbca4cfc12d2ee4bf0b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930284"
---
# <a name="migrate-postgresql-database-to-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Migrer une base de données PostgreSQL vers un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc

Ce document décrit les étapes à suivre pour faire de votre base de données PostgreSQL existante (qui n’est pas hébergée dans des Data Services compatibles Azure Arc) dans votre groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="considerations"></a>Considérations

Un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc est la version de communauté de PostgreSQL et s’exécute avec l’extension CitusData activée. Par conséquent, tout outil qui fonctionne sur PostgreSQL en dehors d’Azure Arc doit fonctionner avec un groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc.


Par conséquent, avec l’ensemble d’outils que vous utilisez aujourd’hui pour Postgres, vous devez être en mesure d’effectuer les opérations suivantes :
1. sauvegarder votre base de données Postgres à partir de votre instance hébergée en dehors d’Azure Arc ;
2. la restaurer dans votre groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc.

Voici ce qu’il vous reste à faire :
- réinitialiser les paramètres du serveur ;
- réinitialiser les contextes de sécurité (recréer des utilisateurs, des rôles et réinitialiser des autorisations...).

Pour effectuer cette opération de sauvegarde/restauration, vous pouvez utiliser n’importe quel outil capable d’effectuer des sauvegardes/restaurations pour Postgres. Exemple :
- Azure Data Studio et son extension Postgres
- `pgcli`
- `pgAdmin`
- `pg_dump`
- `pg_restore`
- `psql`
- ...

## <a name="example"></a>Exemple
Illustrons ces étapes à l’aide de l’outil `pgAdmin`.
Prenons la configuration suivante :
- **Source :**  
    un serveur Postgres s’exécutant localement sur un serveur nu et nommé JEANYDSRV. Il est de la version 12 et héberge une base de données nommée MyOnPremPostgresDB contenant une table T1 comportant 1 ligne :::image type="content" source="media/postgres-hyperscale/migrate-pg-source.jpg" alt-text="Migrate-source":::

- **Destination :**  
    un serveur Postgres s’exécutant dans un environnement Azure Arc et nommé postgres01. Il est de la version 12. Il n’a pas de base de données à l’exception de la base de données Postgres standard.  
    :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination.jpg" alt-text="Migrate-source":::


### <a name="take-a-backup-of-the-source-database-on-premises"></a>Effectuez une sauvegarde de la base de données localement

:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup.jpg" alt-text="Migrate-source":::

Configurez-la :
1. Donnez-lui un nom de fichier : **MySourceBackup**
2. Définissez le format sur **Personnalisé**
:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup2.jpg" alt-text="Migrate-source":::

La sauvegarde se termine correctement :  
:::image type="content" source="media/postgres-hyperscale/Migrate-PG-Source-Backup3.jpg" alt-text="Migrate-source":::

### <a name="create-an-empty-database-on-the-destination-system-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Créez une base de données vide sur le système de destination dans votre groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc

> [!NOTE]
> Pour inscrire une instance Postgres dans l’outil `pgAdmin`, vous devez utiliser une adresse IP publique de votre instance dans votre cluster Kubernetes et définir le port et le contexte de sécurité de manière appropriée. Vous trouverez ces détails sur la ligne du point de terminaison `psql` après avoir exécuté la commande suivante :

```console
azdata arc postgres endpoint list -n postgres01
```
Une sortie semblable à la suivante est renvoyée :
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

Donnez un nom à la base de données de destination **RESTORED_MyOnPremPostgresDB**  
:::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbcreate.jpg" alt-text="Migrate-destination-db-create"lightbox="media/postgres-hyperscale/migrate-pg-destination-dbcreate.jpg":::

### <a name="restore-the-database-in-your-arc-setup"></a>Restaurez la base de données dans votre configuration Arc
:::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore.jpg" alt-text="Migrate-source":::

Configurez la restauration :
1. Pointez sur le fichier qui contient la sauvegarde à restaurer : **MySourceBackup**
2. Conservez le format défini sur **Personnalisé ou tar**
   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore2.jpg" alt-text="Migrate-source":::

3. Cliquez sur **Restaurer**.  

   La restauration a réussi.  
   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestore3.jpg" alt-text="Migrate-source":::

### <a name="verify-that-the-database-was-successfully-restored-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Vérifiez que la base de données a été correctement restaurée dans votre groupe de serveurs PostgreSQL Hyperscale activé par Azure Arc

Utilisez l'une des méthodes suivantes :

**À partir de `pgAdmin`:**  

Développez l’instance Postgres hébergée dans votre installation Azure Arc. Vous voyez la table dans la base de données que vous avez restaurée et, lorsque vous sélectionnez les données, elle affiche la même ligne que dans l’instance locale :

   :::image type="content" source="media/postgres-hyperscale/migrate-pg-destination-dbrestoreverif.jpg" alt-text="Migrate-source"
     },
     {
       "Description": "Log Search Dashboard",
       "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
     },
     {
       "Description": "Metrics Dashboard",
       "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
     }
   ]
   ```

1. À partir de votre chaîne de connexion `psql`, utilisez le paramètre `-d` pour indiquer le nom de la base de données. Avec la commande ci-dessous, vous êtes invité à entrer le mot de passe :

   ```console
   psql -d RESTORED_MyOnPremPostgresDB -U postgres -h 10.0.0.4 -p 32639
   ```

   Connexions `psql`.

   ```output
   Password for user postgres:
   psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1), server 12.3 (Debian 12.3-1.pgdg100+1))
   WARNING: psql major version 10, server major version 12.
         Some psql features might not work.
   SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
   Type "help" for help.

   RESTORED_MyOnPremPostgresDB=#   
   ```

1. Sélectionnez la table pour voir les données que vous avez restaurées à partir de l’instance Postgres locale :

   ```console
   RESTORED_MyOnPremPostgresDB=# select * from t1;
   ```

   ```output
    col1 |    col2
   ------+-------------
       1 | BobbyIsADog
   (1 row)
   ```

> [!NOTE]
> - Vous ne verrez pas beaucoup d’avantages en matière de performances résultant de l’exécution sur PostgreSQL Hyperscale activé par Azure Arc jusqu’à effectuer un scale-out et partitionner/distribuer les données entre les nœuds Worker de votre groupe de serveurs PostgreSQL Hyperscale. Consultez les [Étapes suivantes](#next-steps).
>
> - Il n’est pas possible aujourd’hui d’« intégrer dans Azure Arc » une instance Postgres existante qui s’exécuterait localement ou dans un autre cloud. En d’autres termes, il n’est pas possible d’installer une sorte d’« agent Azure Arc » sur votre instance Postgres existante pour en faire une configuration de Postgres activé par Azure Arc. Au lieu de cela, vous devez créer une nouvelle instance Postgres et y transférer des données. Vous pouvez utiliser la technique décrite ci-dessus pour effectuer cette opération, ou vous utiliser n’importe quel outil ETL de votre choix.

## <a name="next-steps"></a>Étapes suivantes

- Lisez les concepts et les guides pratiques d'Azure Database pour PostgreSQL Hyperscale afin de distribuer vos données sur différents nœuds PostgreSQL Hyperscale et de tirer parti de toute la puissance d'Azure Database pour PostgreSQL Hyperscale :
    * [Nœuds et tables](../../postgresql/concepts-hyperscale-nodes.md)
    * [Déterminer le type d’application](../../postgresql/concepts-hyperscale-app-type.md)
    * [Choisir une colonne de distribution](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocalisation de table](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribuer et modifier des tables](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Concevoir une base de données multilocataire](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Concevoir un tableau de bord d’analytique en temps réel](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

> *Dans ces documents, ignorez les sections **Connectez-vous au portail Azure** et **Créer un serveur Azure Database pour Postgres – Hyperscale (Citus)** . Implémentez les étapes restantes dans votre déploiement Azure Arc. Ces sections sont spécifiques à Azure Database pour PostgreSQL Hyperscale (Citus) proposé en tant que service PaaS dans le cloud Azure, mais les autres parties des documents s’appliquent directement à votre PostgreSQL Hyperscale activé pour Azure Arc.

- [Effectuer un scale-out de votre groupe de serveurs Azure Database pour PostgreSQL Hyperscale](scale-out-postgresql-hyperscale-server-group.md)
