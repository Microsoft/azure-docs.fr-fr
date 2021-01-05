---
title: Gérer Azure Cache pour Redis à l’aide de l’interface de ligne de commande classique Azure
description: Découvrez comment installer l’interface de ligne de commande classique Azure sur une plateforme quelconque, comment l’utiliser pour vous connecter à votre compte Azure et comment créer et gérer un cache Azure pour Redis à partir de cette interface.
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 01/23/2017
ms.author: yegu
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7643f882d5ac330046c169e0a3f2fa4920331d4e
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537692"
---
# <a name="how-to-create-and-manage-azure-cache-for-redis-using-the-azure-classic-cli"></a>Création et gestion du Cache Azure pour Redis à l’aide de l’interface de ligne de commande classique Azure
> [!div class="op_single_selector"]
> * [PowerShell](cache-how-to-manage-redis-cache-powershell.md)
> * [Azure Classic CLI](cache-manage-cli.md)
>

L'interface de ligne de commande classique Azure est un excellent moyen de gérer votre infrastructure Azure à partir de n'importe quelle plateforme. Cet article montre comment créer et gérer vos instances de Cache Azure pour Redis à l’aide de l’interface de ligne de commande classique Azure.

[!INCLUDE [outdated-cli-content](../../includes/contains-classic-cli-content.md)]
> [!NOTE]
> Pour obtenir les exemples de scripts d’interface de ligne de commande Azure les plus récents, consultez [Exemples de Cache Azure pour Redis d’interface de ligne de commande Azure](cli-samples.md).

## <a name="prerequisites"></a>Prérequis
Pour créer et gérer des instances de Cache Azure pour Redis à l’aide de l’interface de ligne de commande classique Azure, vous devez procéder comme suit.

* Vous devez disposer d’un compte Azure. Si vous n’en avez pas, vous pouvez créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/) en quelques minutes.
* [Installez l’interface Azure Classic CLI](/cli/azure/install-classic-cli).
* Connectez votre installation d’interface de ligne de commande Azure à un compte Azure personnel ou à un compte Azure professionnel ou scolaire, puis connectez-vous à partir de l’interface de ligne de commande classique à l’aide de la commande `azure login`.
* Avant d'exécuter les commandes suivantes, faites basculer l'interface de ligne de commande classique en mode Gestionnaire des ressources en exécutant la commande `azure config mode arm`. Pour plus d’informations, consultez [Utiliser l’interface de ligne de commande classique Azure pour gérer les ressources et les groupes de ressources Azure](../azure-resource-manager/management/manage-resources-cli.md).

## <a name="azure-cache-for-redis-properties"></a>Propriétés du cache Azure pour Redis
Les propriétés suivantes sont utilisées lors de la création et de la mise à jour des instances de Cache Azure pour Redis.

| Propriété | Commutateur | Description |
| --- | --- | --- |
| name |-n, --name |Nom du Cache Azure pour Redis. |
| resource group |-g, --resource-group |Nom du groupe de ressources. |
| location |-l, --location |Emplacement où créer le cache. |
| taille |-z, --size |Taille du Cache Azure pour Redis. Valeurs valides : [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4] |
| sku |-x, --sku |SKU Redis. Doit être une des valeurs : [De Base, Standard, Premium] |
| EnableNonSslPort |-e, --enable-non-ssl-port |Propriété EnableNonSslPort du Cache Azure pour Redis. Ajoutez cet indicateur si vous souhaitez activer le port non TLS/SSL pour votre cache |
| Configuration de Redis |-c, --redis-configuration |Configuration de Redis. Entrez ici une chaîne au format JSON des clés et des valeurs de configuration. Format :"{"":"","":""}" |
| Configuration de Redis |-f, --redis-configuration-file |Configuration de Redis. Entrez ici le chemin d’un fichier contenant les clés et les valeurs de configuration. Format pour l’entrée du fichier : {"":"","":""} |
| Nombre de partitions |-r, --shard-count |Nombre de partitions à créer sur un cache de cluster Premium avec clustering. |
| Réseau virtuel |-v, --virtual-network |Quand vous hébergez votre cache dans un réseau virtuel, spécifie l’ID de la ressource ARM exacte du réseau virtuel où déployer le Cache Azure pour Redis. Exemple de format : /subscriptions/{ID_abonnement}/resourceGroups/{nom_groupe_ressources}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| key type |-t, --key-type |Type de clé à renouveler. Valeurs valides : [Primary, Secondary] |
| StaticIP |-p, --static-ip \<static-ip\> |Lorsque vous hébergez votre cache dans un réseau virtuel, spécifie une adresse IP unique dans le sous-réseau pour le cache. Si elle est omise, une adresse IP est choisie pour vous dans le sous-réseau. |
| Subnet |t, --subnet \<subnet\> |Lorsque vous hébergez votre cache dans un réseau virtuel, spécifie le nom du sous-réseau dans lequel déployer le cache. |
| VirtualNetwork |-v, --virtual-network \<virtual-network\> |Quand vous hébergez votre cache dans un réseau virtuel, spécifie l’ID de la ressource ARM exacte du réseau virtuel où déployer le Cache Azure pour Redis. Exemple de format : /subscriptions/{ID_abonnement}/resourceGroups/{nom_groupe_ressources}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 |
| Abonnement |-s, --subscription |Identificateur de l’abonnement. |

## <a name="see-all-azure-cache-for-redis-commands"></a>Voir toutes les commandes de Cache Azure pour Redis
Pour afficher toutes les commandes de Cache Azure pour Redis et leurs paramètres, utilisez la commande `azure rediscache -h`.

```azurecli
C:\>azure rediscache -h
help:    Commands to manage your Azure Cache for Redis(s)
help:
help:    Create an Azure Cache for Redis
help:      rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
help:
help:    Delete an existing Azure Cache for Redis
help:      rediscache delete [--name <name> --resource-group <resource-group> ]
help:
help:    List all Azure Cache for Redis within your Subscription or Resource Group
help:      rediscache list [options]
help:
help:    Show properties of an existing Azure Cache for Redis
help:      rediscache show [--name <name> --resource-group <resource-group>]
help:
help:    Change settings of an existing Azure Cache for Redis
help:      rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
help:
help:    Renew the authentication key for an existing Azure Cache for Redis
help:      rediscache renew-key [--name <name> --resource-group <resource-group> ]
help:
help:    Lists Primary and Secondary key of an existing Azure Cache for Redis
help:      rediscache list-keys [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help  output usage information
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="create-an-azure-cache-for-redis"></a>Créer un Cache Azure pour Redis
Pour créer un Cache Azure pour Redis, utilisez la commande suivante :

```azurecli
    azure rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
```

Pour plus d’informations sur cette commande, exécutez la commande `azure rediscache create -h`.

```azurecli
C:\>azure rediscache create -h
help:    Create an Azure Cache for Redis
help:
help:    Usage: rediscache create [--name <name> --resource-group <resource-group> --location <location> [options]]
help:
help:    Options:
help:      -h, --help                                               output usage information
help:      -v, --verbose                                            use verbose output
help:      -vv                                                      more verbose with debug output
help:      --json                                                   use json output
help:      -n, --name <name>                                        Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>                    Name of the Resource Group
help:      -l, --location <location>                                Location to create cache.
help:      -z, --size <size>                                        Size of the Azure Cache for Redis. Valid values: [C0, C1, C2, C3, C4, C5, C6, P1, P2, P3, P4]
help:      -x, --sku <sku>                                          Redis SKU. Should be one of : [Basic, Standard, Premium]
help:      -e, --enable-non-ssl-port                                EnableNonSslPort property of the Azure Cache for Redis. Add this flag if you want to enable the non-TLS/SSL Port for your cache
help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here. Format:"{"<key1>":"<value1>","<key2>":"<value2>"}"
help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here. Format for the file entry: {"<key1>":"<value1>","<key2>":"<value2>"}
help:      -r, --shard-count <shard-count>                          Number of Shards to create on a Premium Cluster Cache
help:      -v, --virtual-network <virtual-network>                  The exact ARM resource ID of the virtual network to deploy the Azure Cache for Redis in. Example format: /subscriptions/{subid}/resourceGroups/{resourceGroupName}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1
help:      -t, --subnet <subnet>                                    Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network
help:      -p, --static-ip <static-ip>                              Required when deploying an Azure Cache for Redis inside an existing Azure Virtual Network
help:      -s, --subscription <id>                                  the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="delete-an-existing-azure-cache-for-redis"></a>Supprimer un Cache Azure pour Redis existant
Pour supprimer un Cache Azure pour Redis, utilisez la commande suivante :

```azurecli
    azure rediscache delete [--name <name> --resource-group <resource-group> ]
```

Pour plus d’informations sur cette commande, exécutez la commande `azure rediscache delete -h`.

```azurecli
C:\>azure rediscache delete -h
help:    Delete an existing Azure Cache for Redis
help:
help:    Usage: rediscache delete [--name <name> --resource-group <resource-group> ]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which the cache exists
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="list-all-azure-cache-for-redis-within-your-subscription-or-resource-group"></a>Répertorier tous les caches Azure pour Redis dans votre abonnement ou groupe de ressources
Pour répertorier tous les caches Azure pour Redis dans votre abonnement ou groupe de ressources, utilisez la commande suivante :

```azurecli
    azure rediscache list [options]
```

Pour plus d’informations sur cette commande, exécutez la commande `azure rediscache list -h`.

```azurecli
C:\>azure rediscache list -h
help:    List all Azure Cache for Redis within your Subscription or Resource Group
help:
help:    Usage: rediscache list [options]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -g, --resource-group <resource-group>  Name of the Resource Group
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="show-properties-of-an-existing-azure-cache-for-redis"></a>Afficher les propriétés d’un Cache Azure pour Redis existant
Pour afficher les propriétés d’un Cache Azure pour Redis existant, utilisez la commande suivante :

```azurecli
    azure rediscache show [--name <name> --resource-group <resource-group>]
```

Pour plus d’informations sur cette commande, exécutez la commande `azure rediscache show -h`.

```azurecli
C:\>azure rediscache show -h
help:    Show properties of an existing Azure Cache for Redis
help:
help:    Usage: rediscache show [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

<a name="scale"></a>

## <a name="change-settings-of-an-existing-azure-cache-for-redis"></a>Modifier les paramètres d’un Cache Azure pour Redis existant
Pour modifier les propriétés d’un Cache Azure pour Redis existant, utilisez la commande suivante :

```azurecli
    azure rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
```

Pour plus d’informations sur cette commande, exécutez la commande `azure rediscache set -h`.

```azurecli
C:\>azure rediscache set -h
help:    Change settings of an existing Azure Cache for Redis
help:
help:    Usage: rediscache set [--name <name> --resource-group <resource-group> --redis-configuration <redis-configuration>/--redis-configuration-file <redisConfigurationFile>]
help:
help:    Options:
help:      -h, --help                                               output usage information
help:      -v, --verbose                                            use verbose output
help:      -vv                                                      more verbose with debug output
help:      --json                                                   use json output
help:      -n, --name <name>                                        Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>                    Name of the Resource Group
help:      -c, --redis-configuration <redis-configuration>          Redis Configuration. Enter a JSON formatted string of configuration keys and values here.
help:      -f, --redis-configuration-file <redisConfigurationFile>  Redis Configuration. Enter the path of a file containing configuration keys and values here.
help:      -s, --subscription <subscription>                        the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="renew-the-authentication-key-for-an-existing-azure-cache-for-redis"></a>Renouveler la clé d’authentification pour un Cache Azure pour Redis existant
Pour renouveler la clé d’authentification pour un Cache Azure pour Redis, utilisez la commande suivante :

```azurecli
    azure rediscache renew-key [--name <name> --resource-group <resource-group> --key-type <key-type>]
```

Spécifiez `Primary` ou `Secondary` pour `key-type`.

Pour plus d’informations sur cette commande, exécutez la commande `azure rediscache renew-key -h`.

```azurecli
C:\>azure rediscache renew-key -h
help:    Renew the authentication key for an existing Azure Cache for Redis
help:
help:    Usage: rediscache renew-key [--name <name> --resource-group <resource-group> ]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which cache exists
help:      -t, --key-type <key-type>              type of key to renew. Valid values are: 'Primary', 'Secondary'.
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```

## <a name="list-primary-and-secondary-keys-of-an-existing-azure-cache-for-redis"></a>Répertorier les clés primaire et secondaire d’un Cache Azure pour Redis
Pour répertorier les clés primaire et secondaire d’un Cache Azure pour Redis, utilisez la commande suivante :

```azurecli
    azure rediscache list-keys [--name <name> --resource-group <resource-group>]
```

Pour plus d’informations sur cette commande, exécutez la commande `azure rediscache list-keys -h`.

```azurecli
C:\>azure rediscache list-keys -h
help:    Lists Primary and Secondary key of an existing Azure Cache for Redis
help:
help:    Usage: rediscache list-keys [--name <name> --resource-group <resource-group>]
help:
help:    Options:
help:      -h, --help                             output usage information
help:      -v, --verbose                          use verbose output
help:      -vv                                    more verbose with debug output
help:      --json                                 use json output
help:      -n, --name <name>                      Name of the Azure Cache for Redis.
help:      -g, --resource-group <resource-group>  Name of the Resource Group under which Cache exists
help:      -s, --subscription <subscription>      the subscription identifier
help:
help:    Current Mode: arm (Azure Resource Management)
```