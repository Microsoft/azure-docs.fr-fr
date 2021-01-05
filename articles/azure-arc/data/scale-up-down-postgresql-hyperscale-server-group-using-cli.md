---
title: Mettre à l’échelle un groupe de serveurs Azure Database pour PostgreSQL Hyperscale à l’aide de l’interface de ligne de commande (azdata ou kubectl)
description: Mettre à l’échelle un groupe de serveurs Azure Database pour PostgreSQL Hyperscale à l’aide de l’interface de ligne de commande (azdata ou kubectl)
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 303a919cc0afc9b5db49918233f3e5718a896646
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148053"
---
# <a name="scale-up-and-down-an-azure-database-for-postgresql-hyperscale-server-group-using-cli-azdata-or-kubectl"></a>Mettre à l’échelle un groupe de serveurs Azure Database pour PostgreSQL Hyperscale à l’aide de l’interface de ligne de commande (azdata ou kubectl)



Il est parfois nécessaire de modifier les caractéristiques ou la définition d’un groupe de serveurs. Exemple :

- Mettre à l’échelle le nombre de vCores que chaque coordinateur ou nœud Worker utilise
- Mettre à l’échelle la mémoire que chaque nœud coordinateur ou Worker utilise

Ce guide explique comment mettre à l’échelle vCore et/ou la mémoire.

La mise à l’échelle des paramètres de vCore ou de mémoire de votre groupe de serveurs signifie que vous avez la possibilité de définir une valeur minimale et/ou maximale pour chacun des paramètres de vCore et de mémoire. Si vous souhaitez configurer votre groupe de serveurs pour utiliser un nombre spécifique de vCores ou une quantité spécifique de mémoire, vous devez définir la même valeur pour les paramètres min et les paramètres max.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="show-the-current-definition-of-the-server-group"></a>Afficher la définition actuelle du groupe de serveurs

Pour afficher la définition actuelle de votre groupe de serveurs et voir les paramètres de vCore et de mémoire actuels, exécutez l’une des commandes suivantes :

### <a name="cli-with-azdata"></a>Interface de ligne de commande avec azdata

```console
azdata arc postgres server show -n <server group name>
```
### <a name="cli-with-kubectl"></a>Interface de ligne de commande avec kubectl

```console
kubectl describe postgresql-12/<server group name> [-n <namespace name>]
```
> [!NOTE]
> Si vous avez créé un groupe de serveurs de PostgreSQL version 11, exécutez plutôt la commande `kubectl describe postgresql-11/<server group name>`.

Celle-ci retourne la configuration de votre groupe de serveurs. Si vous avez créé le groupe de serveurs avec les paramètres par défaut, vous devez voir la définition suivante :

```console
"scheduling": {
      "default": {
        "resources": {
          "requests": {
            "memory": "256Mi"
          }
        }
      }
    },
```

## <a name="interpret-the-definition-of-the-server-group"></a>Interpréter la définition du groupe de serveurs

Dans la définition d’un groupe de serveurs, la section contenant les paramètres de nombre min/max de vCores par nœud et de mémoire min/max par nœud est la section **« planification »** . Dans cette section, les paramètres max sont conservés dans une sous-section appelée **« Limites »** , et les paramètres min dans la sous-section intitulée **« demandes »** .

Si vous définissez des paramètres min différents des paramètres max, la configuration garantit que les ressources demandées sont allouées au groupe de serveurs si nécessaire. Les limites que vous définissez ne seront pas dépassées.

Les ressources (vCores et mémoire) qui seront réellement utilisées par votre groupe de serveurs sont ne dépassent pas les paramètres max et dépendent des charges de travail et des ressources disponibles sur le cluster. Si ne limitez pas les paramètres à une valeur max, votre groupe de serveurs peut utiliser toutes les ressources que le cluster Kubernetes alloue aux nœuds Kubernetes sur lesquels votre groupe de serveurs est planifié.

Les paramètres de vCore et de mémoire s’appliquent à chacun des nœuds PostgreSQL Hyperscale (nœud coordinateur et nœuds Worker). Il n’est pas encore possible de définir le nœud coordinateur et les nœuds Worker séparément.

Dans une configuration par défaut, seule la mémoire minimale est définie sur 256Mi, car il s’agit de la quantité minimale de mémoire recommandée pour exécuter PostgreSQL Hyperscale.

> [!NOTE]
> Le fait de définir un minimum ne signifie pas que le groupe de serveurs utilisera nécessairement ce minimum. Cela signifie que si le groupe de serveurs en a besoin, il est garanti qu’au moins ce minimum est alloué. Supposons, par exemple, que nous définissions `--minCpu 2`. Cela ne signifie pas que le groupe de serveurs utilisera au moins 2 vCores à tout moment. Au lieu de cela, le groupe de serveurs peut commencer à utiliser moins de 2 vCores s’il n’en a pas besoin, avec la garantie de se voir alloué au moins 2 vCores s’il en a besoin plus tard. Cela implique que le cluster Kubernetes alloue des ressources à d’autres charges de travail de façon à pouvoir allouer 2 vCores au groupe de serveurs s’il en a besoin.

>[!NOTE]
>Avant de modifier la configuration de votre système, veillez à vous familiariser avec le modèle de ressource Kubernetes [ici](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

## <a name="scale-up-the-server-group"></a>Effectuer un scale-up du groupe de serveurs

Les paramètres que vous allez définir doivent être pris en compte dans la configuration que vous définissez pour votre cluster Kubernetes. Veillez à ne pas définir de valeurs que votre cluster Kubernetes ne sera pas en mesure de satisfaire. Cela peut conduire à des erreurs ou à un comportement imprévisible. Par exemple, si l’état de votre groupe de serveurs reste défini sur _mise à jour_ pendant un temps prolongé après la modification de la configuration, cela peut indiquer que vous définissez les paramètres ci-dessous sur des valeurs que votre cluster Kubernetes ne peut pas satisfaire. Si tel est le cas, annulez la modification ou lisez _troubleshooting_section.

Par exemple, supposons que vous souhaitiez effectuer un scale-up de la définition de votre groupe de serveurs vers :

- vCore min = 2
- vCore max = 4
- Mémoire min = 512 Mo
- Mémoire max = 1 Go

Vous devez utiliser l’une des approches suivantes :

### <a name="cli-with-azdata"></a>Interface de ligne de commande avec azdata

```console
azdata arc postgres server edit -n <name of your server group> --cores-request <# core-request>  --cores-limit <# core-limit>  --memory-request <# memory-request>Mi  --memory-limit <# memory-limit>Mi
```

> [!CAUTION]
> Voici un exemple fourni pour illustrer l’utilisation de la commande. Avant d’exécuter une commande edit, veillez à définir les paramètres sur des valeurs que le cluster Kubernetes peut honorer.

```console
azdata arc postgres server edit -n <name of your server group> --cores-request 2  --cores-limit 4  --memory-request 512Mi  --memory-limit 1024Mi
```

La commande s’exécute correctement lorsqu’elle affiche :

```console
<name of your server group> is Ready
```

> [!NOTE]
> Pour plus d’informations sur ces paramètres, exécutez `azdata arc postgres server edit --help`.

### <a name="cli-with-kubectl"></a>Interface de ligne de commande avec kubectl

```console
kubectl edit postgresql-12/<server group name> [-n <namespace name>]
```

Vous accédez à l’éditeur vi dans lequel vous pouvez naviguer et modifier la configuration. Utilisez la commande suivante pour mapper le paramètre souhaité au nom du champ dans la spécification :

> [!CAUTION]
> Voici un exemple fourni pour illustrer la modification de la configuration. Avant de mettre à jour la configuration, veillez à définir les paramètres sur des valeurs que le cluster Kubernetes peut honorer.

Par exemple :
- vCore min = 2 -> planification\par défaut\ressources\demandes\uc
- vCore max = 4-> planification\par défaut\ressources\limites\uc
- Mémoire min = 512 Mo -> planification\par défaut\ressources\demandes\uc
- Mémoire max = 1 Go ->  planification\par défaut\ressources\limites\uc

Si vous n’êtes pas familiarisé avec l’éditeur vi, consultez la description des commandes dont vous pouvez avoir besoin [ici](https://www.computerhope.com/unix/uvi.htm) :
- Mode d’édition : `i`
- se déplacer avec des flèches
- _cesser de modifier : `esc`
- _quitter sans enregistrer : `:qa!`
- _quitter après enregistrement : `:qw!`


## <a name="show-the-scaled-up-definition-of-the-server-group"></a>Afficher la définition de scale up du groupe de serveurs

Exécutez de nouveau la commande pour afficher la définition du groupe de serveurs et vérifier qu’elle est telle que vous le souhaitez :

### <a name="cli-with-azdata"></a>Interface de ligne de commande avec azdata

```console
azdata arc postgres server show -n <the name of your server group>
```
### <a name="cli-with-kubectl"></a>Interface de ligne de commande avec kubectl

```console
kubectl describe postgresql-12/<server group name>  [-n <namespace name>]
```
> [!NOTE]
> Si vous avez créé un groupe de serveurs de PostgreSQL version 11, exécutez plutôt la commande `kubectl describe postgresql-11/<server group name>`.


Celle-ci affiche la nouvelle définition du groupe de serveurs :

```console
"scheduling": {
      "default": {
        "resources": {
          "limits": {
            "cpu": "4",
            "memory": "1024Mi"
          },
          "requests": {
            "cpu": "2",
            "memory": "512Mi"
          }
        }
      }
    },
```

## <a name="scale-down-the-server-group"></a>effectuer un scale-down du groupe de serveurs

Pour effectuer un scale-down du groupe de serveurs, vous exécutez la même commande, mais vous définissez des valeurs inférieures pour les paramètres que vous souhaitez modifier. Pour supprimer les demandes et/ou les limites, spécifiez sa valeur en tant que chaîne vide.

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer un scale-out de votre groupe de serveurs Azure Database pour PostgreSQL Hyperscale](scale-out-postgresql-hyperscale-server-group.md)
- [Configuration de stockage et concepts de stockage Kubernetes](storage-configuration.md)
- [Modèle de ressources Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)
