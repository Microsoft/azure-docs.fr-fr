---
title: Configurer des paramètres de serveur du moteur Postgres pour votre groupe de serveurs PostgreSQL Hyperscale sur Azure Arc
titleSuffix: Azure Arc enabled data services
description: Configurer des paramètres de serveur du moteur Postgres pour votre groupe de serveurs PostgreSQL Hyperscale sur Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: cdbddfc84b3f71576cfd0299f2babec859b4ef1f
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92311058"
---
# <a name="set-the-database-engine-settings-for-azure-arc-enabled-postgresql-hyperscale"></a>Définir les paramètres du moteur de base de données pour PostgreSQL Hyperscale activé par Azure Arc

Ce document décrit les étapes permettant de définir les paramètres du moteur de base de données de votre groupe de serveurs PostgreSQL Hyperscale sur des valeurs personnalisées (autres que par défaut). Pour plus d’informations sur les paramètres du moteur de base de données qui peuvent être définis et leur valeur par défaut, consultez la documentation de PostgreSQL [ici](https://www.postgresql.org/docs/current/runtime-config.html).

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

> [!NOTE]
> La préversion ne prend pas en charge la définition des paramètres suivants : 
>
> - `archive_command`
> - `archive_timeout`
> - `log_directory`
> - `log_file_mode`
> - `log_filename`
> - `restore_command`
> - `shared_preload_libraries`
> - `synchronous_commit`
> - `ssl`
> - `wal_level`

## <a name="syntax"></a>Syntaxe

Le format général de la commande permettant de configurer les paramètres du moteur de base de données est le suivant :

```console
azdata arc postgres server edit -n <server group name>, [{--engine-settings, -e}] [{--replace-engine-settings, --re}] {'<parameter name>=<parameter value>, ...'}
```

## <a name="show-current-custom-values"></a>Afficher les valeurs personnalisées actuelles

### <a name="with-azure-data-cli-azdata-command"></a>Avec une commande [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

```console
azdata arc postgres server show -n <server group name>
```

Exemple :

```console
azdata arc postgres server show -n postgres01
```

cette commande retourne les spécifications du groupe de serveurs dans lequel vous pouvez voir les paramètres que vous avez définis. S’il n’existe pas de section engine\settings, cela signifie que tous les paramètres s’exécutent sur la base de leur valeur par défaut :

```console
 "
...
engine": {
      "settings": {
        "default": {
          "autovacuum_vacuum_threshold": "65"
        }
      }
    },
...
```

### <a name="with-kubectl-command"></a>Avec la commande kubectl

Procédez comme suit.

1. Récupérez le type de définition de ressource personnalisée pour votre groupe de serveurs

   Exécutez :

   ```console
   azdata arc postgres server show -n <server group name>
   ```

   Exemple :

   ```console
   azdata arc postgres server show -n postgres01
   ```

   cette commande retourne les spécifications du groupe de serveurs dans lequel vous pouvez voir les paramètres que vous avez définis. S’il n’existe pas de section engine\settings, cela signifie que tous les paramètres s’exécutent sur la base de leur valeur par défaut :

   ```output
   > {
     >"apiVersion": "arcdata.microsoft.com/v1alpha1",
     >"**kind**": "**postgresql-12**",
     >"metadata": {
       >"creationTimestamp": "2020-08-25T14:32:23Z",
       >"generation": 1,
       >"name": "postgres01",
       >"namespace": "arc",  
   ```

   Dans les résultats de sortie, recherchez le champ `kind` et réservez la valeur, par exemple : `postgresql-12`.

2. Décrivez la ressource personnalisée Kubernetes correspondant à votre groupe de serveurs 

   Le format général de la commande est le suivant :

   ```console
   kubectl describe <kind of the custom resource> <server group name> -n <namespace name>
   ```

   Exemple :

   ```console
   kubectl describe postgresql-12 postgres01
   ```

   Si des valeurs personnalisées sont définies pour les paramètres du moteur, celui-ci les retourne. Exemple :

   ```output
   Engine:
   ...
    Settings:
      Default:
        autovacuum_vacuum_threshold:  65
   ```

   Si vous n’avez pas défini de valeurs personnalisées pour l’un des paramètres du moteur, la section Paramètres du moteur du `resultset` est vide comme suit :

   ```output
   Engine:
   ...
       Settings:
         Default:
   ```

## <a name="set-custom-values-for-engine-settings"></a>Définir des valeurs personnalisées pour les paramètres du moteur

Les commandes ci-dessous définissent les paramètres du nœud coordinateur et des nœuds Worker de votre PostgresSQL Hyperscale sur les mêmes valeurs. Il n’est pas encore possible de définir des paramètres par rôle dans votre groupe de serveurs. Autrement dit, il n’est pas encore possible de configurer un paramètre donné sur une valeur spécifique sur le nœud coordinateur et sur une autre valeur sur les nœuds Worker.

### <a name="set-a-single-parameter"></a>Définir un paramètre unique

```console
azdata arc server edit -n <server group name> -e <parameter name>=<parameter value>
```

Exemple :

```console
azdata arc postgres server edit -n postgres01 -e shared_buffers=8MB
```

### <a name="set-multiple-parameters-with-a-single-command"></a>Définir plusieurs paramètres avec une seule commande

```console
azdata arc postgres server edit -n <server group name> -e '<parameter name>=<parameter value>, <parameter name>=<parameter value>,...'
```

Exemple :

```console
azdata arc postgres server edit -n postgres01 -e 'shared_buffers=8MB, max_connections=50'
```

### <a name="reset-a-parameter-to-its-default-value"></a>Rétablir la valeur par défaut d’un paramètre

Pour rétablir la valeur par défaut d’un paramètre, définissez-le sans indiquer de valeur. 

Exemple :

```console
azdata arc postgres server edit -n postgres01 -e shared_buffers=
```

### <a name="reset-all-parameters-to-their-default-values"></a>Rétablir les valeurs par défaut de tous les paramètres

```console
azdata arc postgres server edit -n <server group name> -e '' -re
```

Exemple :

```console
azdata arc postgres server edit -n postgres01 -e '' -re
```

## <a name="special-considerations"></a>Considérations spéciales

### <a name="set-a-parameter-which-value-contains-a-comma-space-or-special-character"></a>Définir un paramètre dont la valeur contient une virgule, une espace ou un caractère spécial

```console
azdata arc postgres server edit -n <server group name> -e '<parameter name>="<parameter value>"'
```

Exemple :

```console
azdata arc postgres server edit -n postgres01 -e 'custom_variable_classes = "plpgsql,plperl"'
```

### <a name="pass-an-environment-variable-in-a-parameter-value"></a>Passer une variable d’environnement dans une valeur de paramètre

Pour que la variable d’environnement ne soit pas résolue avant sa définition, elle doit être encapsulée dans "’’".

Par exemple : 

```console
azdata arc postgres server edit -n postgres01 -e 'search_path = "$user"'
```

## <a name="next-steps"></a>Étapes suivantes
- Découvrez comment [effectuer un scale-out (ajout de nœuds Worker)](scale-out-postgresql-hyperscale-server-group.md) de votre groupe de serveurs
- Découvrez comment [effectuer un scale-up ou un scale-down (augmentation/diminution de la mémoire/des vCores)](scale-up-down-postgresql-hyperscale-server-group-using-cli.md) de votre groupe de serveurs
