---
title: Déterminer la taille de sous-réseau requise
titleSuffix: Azure SQL Managed Instance
description: Cette rubrique explique comment calculer la taille du sous-réseau dans lequel les instances Azure SQL Managed Instance seront déployées.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 02/22/2019
ms.openlocfilehash: 156a4c74eea24b20c28df88be85cb32c0ebe2981
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012446"
---
# <a name="determine-required-subnet-size--range-for-azure-sql-managed-instance"></a>Déterminer la taille de sous-réseau d’un réseau virtuel pour Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance doit être déployé au sein d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md) Azure.

Le nombre d’instances managées pouvant être déployées dans le sous-réseau du réseau virtuel dépend de la taille du sous-réseau (plage de sous-réseau).

Quand vous créez une option Managed Instance, Azure alloue plusieurs machines virtuelles en fonction du niveau sélectionné pendant l’approvisionnement. Étant donné que ces machines virtuelles sont associées à votre sous-réseau, elles nécessitent des adresses IP. Pour garantir la haute disponibilité pendant les opérations normales et la maintenance du service, Azure peut allouer des machines virtuelles supplémentaires. Par conséquent, le nombre d’adresses IP nécessaires dans un sous-réseau est supérieur au nombre d’options Managed Instance dans ce sous-réseau.

Par défaut, une option Managed Instance a besoin d’un minimum de 32 adresses IP dans un sous-réseau. Ainsi, vous pouvez utiliser un masque de sous-réseau minimal de /27 lors de la définition de vos plages d’adresses IP de sous-réseau. Une planification minutieuse de la taille des sous-réseaux pour vos déploiements d’instances managées est recommandée. Les entrées qui doivent être prises en considération lors de la planification sont les suivantes :

- Nombre d’instances managées, y compris les paramètres d’instance suivants :
  - Niveau de service
  - Génération du matériel
  - Nombre de vCores
- Plans de scale-up/scale-down ou de modification du niveau de service

> [!IMPORTANT]
> Une taille de sous-réseau avec 16 adresses IP (masque de sous-réseau /28) autorise le déploiement d’une instance managée, mais elle doit être utilisée uniquement pour le déploiement d’une instance unique utilisée à des fins d’évaluation ou dans des scénarios de développement/test dans lesquels aucune opération de mise à l’échelle n’est effectuée.

## <a name="determine-subnet-size"></a>Déterminer la taille du sous-réseau

Dimensionnez votre sous-réseau en fonction des besoins futurs de déploiement et de mise à l’échelle de l’instance. Les paramètres suivants peuvent vous aider à établir un calcul :

- Azure utilise cinq adresses IP dans le sous-réseau pour ses besoins propres.
- Chaque cluster virtuel alloue un nombre supplémentaire d’adresses 
- Chaque instance gérée utilise un nombre d’adresses dépendant du niveau tarifaire et de la génération de matériel

> [!IMPORTANT]
> Il n’est pas possible de modifier la plage d’adresses de sous-réseau si une ressource existe dans le sous-réseau. Il n’est pas non plus possible de déplacer des instances gérées d’un sous-réseau vers un autre. Dans la mesure du possible, envisagez d’utiliser des sous-réseaux plus volumineux plutôt que plus petits pour éviter les problèmes à l’avenir.

GP = usage général ; BC = critique pour l’entreprise ; VC = cluster virtuel

| **Génération de matériel** | **Niveau tarifaire** | **Utilisation d’Azure** | **Utilisation de VC** | **Utilisation de l’instance** | **Total** _ |
| --- | --- | --- | --- | --- | --- |
| Gen4 | GP | 5 | 1 | 5 | 11 |
| Gen4 | BC | 5 | 1 | 5 | 11 |
| Gen5 | GP | 5 | 6 | 3 | 14 |
| Gen5 | BC | 5 | 6 | 5 | 16 |

  \_ Le total de la colonne affiche le nombre d’adresses qui seraient prises lorsqu’une instance est déployée dans le sous-réseau. Chaque instance supplémentaire dans le sous-réseau ajoute le nombre d’adresses représentées par la colonne d’utilisation de l’instance. Les adresses représentées par la colonne d’utilisation d’Azure sont partagées entre plusieurs clusters virtuels, tandis que les adresses représentées par la colonne d’utilisation de VC sont partagées entre les instances placées dans ce cluster virtuel.

L’opération de mise à jour nécessite généralement un redimensionnement de cluster virtuel. Dans certains cas, l’opération de mise à jour nécessite la création d’un cluster virtuel (pour plus d’informations, consultez [l’article sur les opérations de gestion](sql-managed-instance-paas-overview.md#management-operations)). En cas de création d’un cluster virtuel, le nombre d’adresses supplémentaires requises est égal au nombre d’adresses représentées par la colonne d’utilisation de VC auquel vous ajoutez les adresses requises pour les instances placées dans ce cluster virtuel (colonne d’utilisation de l’instance).

**Exemple 1** : Vous envisagez de disposer d’une instance gérée à usage général (Gen4) et d’une instance gérée stratégique (matériel Gen5). Cela signifie que vous avez besoin d’un minimum de 5 + 1 + 1 * 5 + 6 + 1 * 5 = 22 adresses IP pour pouvoir effectuer le déploiement. Étant donné que les plages d’adresses IP sont définies à la puissance 2, votre sous-réseau requiert une plage d’adresses IP minimale de 32 (2 ^ 5) pour ce déploiement.<br><br>
Comme indiqué ci-dessus, dans certains cas, l’opération de mise à jour nécessite la création d’un cluster virtuel. Cela signifie que, en guise d’exemple, dans le cas d’une mise à jour de l’instance gérée critique de l’entreprise Gen5 qui nécessite la création d’un cluster virtuel, vous devrez disposer de 6 + 5 adresses = 11 IP supplémentaires. Étant donné que vous utilisez déjà 22 adresses sur 32, il n’y a pas assez d’adresses disponibles pour cette opération. Ainsi, vous devez réserver le sous-réseau avec un masque de sous-réseau de /26 (64 adresses).

**Exemple 2 :** Vous envisagez de disposer de trois instances à usage général (matériel Gen5) et de deux instances critiques (matériel Gen5) placées dans le même sous-réseau. Cela signifie que vous avez besoin de5 + 6 + 3 * 3 + 2 * 5 = 30 adresses IP. Ainsi, vous devez réserver le sous-réseau avec un masque de sous-réseau de /26. Si vous sélectionnez un masque de sous-réseau de /27, le nombre d’adresses restantes sera égal à 2 (32 - 30). Cela empêchera les opérations de mise à jour pour toutes les instances, car des adresses supplémentaires sont requises dans le sous-réseau pour effectuer la mise à l’échelle des instances.

**Exemple 3** : Vous envisagez de disposer d’une instance gérée à usage général (matériel Gen5) et d’effectuer une opération de mise à jour des vCores de temps en temps. Cela signifie que vous avez besoin de 5 + 6 + 1 * 3 + 3 = 17 adresses IP. Comme les plages d’adresses IP sont définies par puissance de 2, vous avez besoin d’une plage de 32 (2 ^ 5) adresses IP. Ainsi, vous devez réserver le sous-réseau avec un masque de sous-réseau de /27.

### <a name="address-requirements-for-update-scenarios"></a>Adresses requises pour les scénarios de mise à jour

Pendant l’opération de mise à l’échelle, les instances nécessitent temporairement une capacité d’adresse IP supplémentaire qui dépend du niveau tarifaire et de la génération de matériel

| **Génération de matériel** | **Niveau tarifaire** | **Scénario** | **Adresses supplémentaires** _ |
| --- | --- | --- | --- |
| Gen4 | GP ou BC | Mise à l’échelle de vCores | 5 |
| Gen4 | GP ou BC | Mise à l’échelle du stockage | 5 |
| Gen4 | GP ou BC | Passage de GP à BC ou BC à GP | 5 |
| Gen4 | GP | Passage à Gen5_ | 9 |
| Gen4 | BC | Passage à Gen5* | 11 |
| Gen5 | GP | Mise à l’échelle de vCores | 3 |
| Gen5 | GP | Mise à l’échelle du stockage | 0 |
| Gen5 | GP | Passage à BC | 5 |
| Gen5 | BC | Mise à l’échelle de vCores | 5 |
| Gen5 | BC | Mise à l’échelle du stockage | 5 |
| Gen5 | BC | Passage à GP | 3 |

  \* Le matériel Gen4 est en cours de retrait et n’est plus disponible pour les nouveaux déploiements. Mettez à jour la génération de matériel de Gen4 vers Gen5 pour tirer parti des fonctionnalités spécifiques à la génération de matériel Gen5.

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble, consultez [Présentation d’Azure SQL Managed Instance](sql-managed-instance-paas-overview.md).
- Découvrez plus en détail l’[architecture de connectivité pour SQL Managed Instance](connectivity-architecture-overview.md).
- Découvrez comment [créer le réseau virtuel dans lequel seront déployées les instances SQL Managed Instance](virtual-network-subnet-create-arm-template.md).
- Pour les problèmes liés au DNS, consultez [Configurer un DNS personnalisé](custom-dns-configure.md).
