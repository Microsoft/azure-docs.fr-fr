---
title: Activer la configuration de sous-réseau assistée par le service pour Azure SQL Managed Instance
description: Activer la configuration de sous-réseau assistée par le service pour Azure SQL Managed Instance
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.date: 03/12/2020
ms.openlocfilehash: 67b398194d9094cd99fccaa85ed0df3be362ce2b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91618010"
---
# <a name="enabling-service-aided-subnet-configuration-for-azure-sql-managed-instance"></a>Activer la configuration de sous-réseau assistée par le service pour Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

La configuration de sous-réseau assistée par le service fournit une gestion automatique de la configuration de réseau pour les sous-réseaux hébergeant des instances gérées. Avec la configuration de sous-réseau assistée par service, l’utilisateur continue de contrôler totalement l'accès au trafic de données (TDS), tandis que l'instance gérée assume la responsabilité de garantir un flux ininterrompu du trafic de gestion afin de respecter le contrat de niveau de service.

Les groupes de sécurité réseau et les règles de table de routage configurés automatiquement sont visibles par le client et annotés avec le préfixe _Microsoft.Sql-managedInstances_UseOnly__.

La configuration assistée par le service est automatiquement activée après activation de la [délégation de sous-réseau](../../virtual-network/subnet-delegation-overview.md) pour le fournisseur de ressources `Microsoft.Sql/managedInstances`.

> [!IMPORTANT] 
> Une fois la délégation de sous-réseau activée, vous ne pouvez pas la désactiver avant d'avoir supprimé le dernier cluster virtuel du sous-réseau. Pour plus d’informations sur la suppression d’un cluster virtuel, consultez l’[article](virtual-cluster-delete.md#delete-a-virtual-cluster-from-the-azure-portal) suivant.

> [!NOTE] 
> La configuration de sous-réseau assistée par le service étant indispensable à la gestion des contrats SLA, à compter du 1er mai 2020, il ne sera plus possible de déployer d'instances gérées dans des sous-réseaux non délégués au fournisseur de ressources d'instance gérée. Le 1er juillet 2020, tous les sous-réseaux contenant des instances gérées seront automatiquement délégués au fournisseur de ressources d’instance gérée. 

## <a name="enabling-subnet-delegation-for-new-deployments"></a>Activer la délégation de sous-réseau pour les nouveaux déploiements
Pour déployer une instance gérée dans un sous-réseau vide, vous devez la déléguer au `Microsoft.Sql/managedInstances`fournisseur de ressources, comme décrit dans l’[article](../../virtual-network/manage-subnet-delegation.md) suivant. _Notez que l’article référencé utilise le fournisseur de ressources `Microsoft.DBforPostgreSQL/serversv2` à titre d'exemple. En ce qui vous concerne, vous devrez utiliser le `Microsoft.Sql/managedInstances` fournisseur de ressources_ .

## <a name="enabling-subnet-delegation-for-existing-deployments"></a>Activer la délégation de sous-réseau pour les déploiements existants

Pour activer la délégation de sous-réseau pour votre déploiement d’instance gérée existant, vous devez rechercher le sous-réseau de réseau virtuel où il se trouve. 

Pour ce faire, consultez `Virtual network/subnet` dans le panneau du portail `Overview` de votre instance gérée.

Vous pouvez également exécuter les commandes PowerShell suivantes. Remplacez la valeur **subscription-id** par l’identifiant de votre abonnement. Remplacez également la valeur **rg-name** par le nom du groupe de ressources de votre instance gérée, puis remplacez **mi-name** par le nom de cette dernière.

```powershell
Install-Module -Name Az

Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount

# Use your subscription ID in place of subscription-id below

Select-AzSubscription -SubscriptionId {subscription-id}

# Replace rg-name with the resource group for your managed instance, and replace mi-name with the name of your managed instance

$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}

$mi.SubnetId
```

Une fois le sous-réseau d’instance gérée trouvé, vous devez le déléguer au fournisseur de ressources `Microsoft.Sql/managedInstances` comme décrit dans l’[article](../../virtual-network/manage-subnet-delegation.md) suivant. _Notez que l’article référencé utilise le fournisseur de ressources `Microsoft.DBforPostgreSQL/serversv2` à titre d'exemple. En ce qui vous concerne, vous devrez utiliser le `Microsoft.Sql/managedInstances` fournisseur de ressources_ .


> [!IMPORTANT]
> L’activation de la configuration assistée par le service n’entraîne pas de basculement ou d’interruption de la connectivité pour les instances gérées déjà présentes dans le sous-réseau.
