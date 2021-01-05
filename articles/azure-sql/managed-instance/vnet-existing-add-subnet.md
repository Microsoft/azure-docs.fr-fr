---
title: Configurer un réseau virtuel existant
titleSuffix: Azure SQL Managed Instance
description: Cet article décrit comment configurer un réseau virtuel et un sous-réseau existants sur lesquels vous pouvez déployer Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 03/17/2020
ms.openlocfilehash: 461acc07ee2217a38f7bb59805d4c7e0de4a1e22
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91617653"
---
# <a name="configure-an-existing-virtual-network-for-azure-sql-managed-instance"></a>Configurer un réseau virtuel existant pour Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance doit être déployé au sein d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md) Azure et du sous-réseau dédié exclusivement aux instances managées. Vous pouvez utiliser le réseau virtuel et le sous-réseau existants s’ils sont configurés conformément aux [exigences des réseaux virtuels SQL Managed Instance](connectivity-architecture-overview.md#network-requirements).

Si l’un des cas suivants vous concerne, vous pouvez valider et modifier votre réseau en utilisant le script expliqué dans cet article :

- Vous avez un sous-réseau qui n’est pas encore configuré.
- Vous ne savez pas si le sous-réseau est conforme aux [exigences](connectivity-architecture-overview.md#network-requirements).
- Vous souhaitez vérifier que le sous-réseau est toujours conforme aux [exigences du réseau](connectivity-architecture-overview.md#network-requirements) après des changements.

> [!Note]
> Vous pouvez créer une instance managée uniquement sur les réseaux virtuels créés par le biais du modèle de déploiement Azure Resource Manager. Les réseaux virtuels Azure créés par le biais du modèle de déploiement classique ne sont pas pris en charge. Calculez la taille du sous-réseau en suivant les instructions de l’article [Déterminer la taille de sous-réseau pour SQL Managed Instance](vnet-subnet-determine-size.md). Vous ne pouvez pas redimensionner le sous-réseau après avoir déployé les ressources à l’intérieur.
>
> Une fois l’instance managée créée, le déplacement de l’instance ou du réseau virtuel vers un autre groupe de ressources ou vers un autre abonnement n’est pas pris en charge.

## <a name="validate-and-modify-an-existing-virtual-network"></a>Valider et modifier un réseau virtuel existant

Si vous souhaitez créer une instance managée à l’intérieur d’un sous-réseau existant, nous vous recommandons de préparer ce dernier à l’aide du script PowerShell suivant :

```powershell
$scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/delegate-subnet'

$parameters = @{
    subscriptionId = '<subscriptionId>'
    resourceGroupName = '<resourceGroupName>'
    virtualNetworkName = '<virtualNetworkName>'
    subnetName = '<subnetName>'
    }

Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/delegateSubnet.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters
```

Le script prépare le sous-réseau en trois étapes :

1. Validation : valide le réseau virtuel et le sous-réseau sélectionnés par rapport aux exigences de mise en réseau de SQL Managed Instance.
2. Confirmation : présente à l’utilisateur un ensemble de changements à apporter au sous-réseau en vue de le préparer au déploiement de SQL Managed Instance. Invite également l’utilisateur à donner son consentement.
3. Préparation : configure correctement le réseau virtuel et le sous-réseau.

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble, consultez [Présentation de SQL Managed Instance](sql-managed-instance-paas-overview.md).
- Pour accéder à un tutoriel montrant comment créer un réseau virtuel, créer une instance managée et restaurer une base de données à partir d’une sauvegarde, consultez [Créer une instance managée](instance-create-quickstart.md).
- Pour les problèmes liés au DNS, consultez [Configuration d’un DNS personnalisé](custom-dns-configure.md).
