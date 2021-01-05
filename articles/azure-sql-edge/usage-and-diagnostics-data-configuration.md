---
title: Configuration des données d’utilisation et de diagnostic Azure SQL Edge
description: Découvrez comment configurer les données d’utilisation et de diagnostic dans Azure SQL Edge.
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 08/04/2020
ms.openlocfilehash: 81b2aabbae148faec06c9ab420bdfecde475b4bb
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97604996"
---
# <a name="azure-sql-edge-usage-and-diagnostics-data-configuration"></a>Configuration des données d’utilisation et de diagnostic Azure SQL Edge

Par défaut, Azure SQL Edge collecte des informations sur la façon dont ses clients utilisent l’application. Plus précisément, Azure SQL Edge recueille des données sur l’expérience d’installation, l’utilisation et les performances. Elles aident Microsoft à améliorer le produit pour mieux répondre aux besoins des clients. Par exemple, Microsoft collecte des informations sur les types de codes d’erreur rencontrés par les utilisateurs afin que nous puissions corriger les bogues associés, améliorer notre documentation sur l’utilisation d’Azure SQL Server et déterminer s’il faudrait ajouter des fonctionnalités au produit pour mieux servir des clients.

En particulier, Microsoft n’envoie par ce mécanisme aucune information de ces types :

- Toute valeur des tables utilisateur.
- Tout identifiant d’ouverture de session ou autres informations d’authentification.
- Toute donnée personnelle ou cliente.

L’exemple de scénario suivant comprend des informations sur l’utilisation des fonctionnalités, qui permettent d’améliorer le produit.

Vous trouverez ci-dessous un exemple de requête à partir des requêtes utilisées pour la collecte des données d’utilisation et de diagnostic. La requête identifie le nombre et les types de sources de données de streaming différentes utilisées dans Azure SQL Edge. Ces données aident Microsoft à identifier les sources de données de diffusion en continu couramment utilisées, de telle sorte que Microsoft peut améliorer les performances et l’expérience utilisateur associées à ces sources de données. 

```sql
select 
count(*) as [count], sum(inputs) as inputs, sum(outputs) as outputs, sum(linked_to_job) 
as linked_to_job, data_source_type
from ( 
select isnull(value,'unknown') as data_source_type, inputs, outputs, linked_to_job
from 
    ( 
        select 
        convert(sysname, lower(substring(ds.location, 0, charindex('://', ds.location))), 1) as data_source_type, 
        isnull(inputs, 0) as inputs, isnull(outputs, 0) as outputs, isnull(js.stream_id/js.stream_id, 0) as linked_to_job 
        from sys.external_streams es
        join sys.external_data_sources ds 
             on es.data_source_id = ds.data_source_id
        left join 
            ( 
            select stream_id, max(cast(is_input as int)) inputs, max(cast(is_output as int)) outputs 
            from sys.external_job_streams group by stream_id 
            ) js
             on js.stream_id = es.object_id 
    ) ds
left join 
    (
        select value from string_split('edgehub,sqlserver,kafka', ',')) as known_ep on data_source_type = value 
    ) known_ds
group by data_source_type
```

## <a name="disable-usage-and-diagnostic-data-collection"></a>Désactiver la collecte des données d’utilisation et de diagnostic

La collecte des données d’utilisation et de diagnostic sur Azure SQL Edge peut être désactivée à l’aide de l’une des méthodes ci-dessous.

> [!NOTE]
> Les données d’utilisation et de diagnostic ne peuvent pas être désactivées pour la version développeur.

### <a name="disable-usage-and-diagnostics-using-environment-variables"></a>Désactiver l’utilisation et les diagnostics à l’aide de variables d’environnement

Pour désactiver la collecte des données d’utilisation et de diagnostics sur Azure SQL Edge, ajoutez la variable d’environnement suivante et affectez-lui la valeur `*False*`. Pour plus d’informations sur la configuration d’Azure SQL Edge à l’aide de variables d’environnement, consultez [Configurer à l’aide de variables d’environnement](configure.md#configure-by-using-environment-variables).

`MSSQL_TELEMETRY_ENABLED = TRUE | FALSE`

- TRUE - Active la collecte des données d’utilisation et de diagnostic. Il s’agit de la configuration par défaut.
- FALSE - Désactive la collecte des données d’utilisation et de diagnostic.

### <a name="disable-usage-and-diagnostics-using-mssqlconf-file"></a>Désactiver l’utilisation et les diagnostics à l’aide de dossiers mssql.conf

Pour désactiver la collecte des données d’utilisation et de diagnostics sur Azure SQL Edge, ajoutez les lignes suivantes dans le fichier mssql. conf sur le lecteur de stockage persistant qui est mappé au dossier /var/opt/mssql/ dans le module SQL Edge. Pour plus d’informations sur la configuration d’Azure SQL Edge à l’aide du fichier mssql.conf, consultez [Configurer à l’aide du fichier mssql.conf](configure.md#configure-by-using-an-mssqlconf-file).

```ini
[telemetry]
customerfeedback = false
```

## <a name="local-audit-of-usage-and-diagnostic-data-collection"></a>Audit local pour l’utilisation et collecte des données de diagnostic

Le composant d’audit local  de l’utilisation et de la collecte de données d’utilisation et de diagnostic de’Azure SQL Edge peut écrire les données collectées par le service vers un dossier désigné, représentant les données (journaux) qui peuvent être envoyées à Microsoft. L’objectif de l’audit local est d’autoriser les clients à visualiser toutes les données collectées par Microsoft avec cette fonctionnalité, pour des raisons de conformité, de réglementation ou de validation de la confidentialité.

### <a name="enable-local-audit-of-usage-and-diagnostics-data"></a>Activer l’audit local pour l’utilisation et collecte des données de diagnostic

Pour activer l’utilisation de l’audit local et les données de diagnostic sur Azure SQL Edge

1. Créez un répertoire cible pour le stockage des nouveaux journaux d’audit local. Ce répertoire cible peut être sur l’ordinateur hôte ou dans le conteneur. Dans l’exemple ci-dessous, le répertoire cible est créé dans le même volume de montage qui est mappé sur le chemin /var/opt/MSSQL/ sur SQL Edge.

   ```bash
   sudo mkdir <host mount path>/audit
   ```

2. Configurez l’audit des données d’utilisation et de diagnostics à l’aide de variables d’environnement ou d’un fichier mssql.conf.

   - À l’aide de variables d’environnement, ajoutez la variable d’environnement suivante à votre déploiement SQL Edge et spécifiez le répertoire cible pour les fichiers d’audit.
   
     `*MSSQL_TELEMETRY_DIR = <host mount path>/audit*`
   
   - À l’aide du fichier mssql.conf, ajoutez les lignes suivantes dans le fichier mssql.conf et spécifiez le répertoire cible pour les fichiers d’audit.
       ```ini
       [telemetry]
       userrequestedlocalauditdirectory  = <host mount path>/audit
       ```  

## <a name="next-steps"></a>Étapes suivantes

- [Se connecter à Azure SQL Edge](connect.md)
- [Générer une solution IoT de bout en bout avec SQL Edge](tutorial-deploy-azure-resources.md)
