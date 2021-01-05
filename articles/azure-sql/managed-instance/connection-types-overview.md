---
title: Types de connexion
titleSuffix: Azure SQL Managed Instance
description: Découvrez les types de connexion Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: vanto
ms.date: 10/07/2019
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: f1c4fe8268d24026609f55d76a102a5c9a4e8295
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91356297"
---
# <a name="azure-sql-managed-instance-connection-types"></a>Types de connexions Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Cet article explique comment les clients se connectent à Azure SQL Managed Instance en fonction du type de connexion. Des exemples de script permettant de changer les types de connexion sont fournis ci-dessous, ainsi que des considérations sur le changement des paramètres de connectivité par défaut.

## <a name="connection-types"></a>Types de connexion

Azure SQL Managed Instance prend en charge les deux types de connexion suivants :

- **Redirection (recommandée) :** les clients établissent les connexions directement sur le nœud qui héberge la base de données. Pour activer la connectivité à l’aide de la redirection, vous devez ouvrir les pare-feu et les groupes de sécurité réseau (NSG) pour autoriser l’accès sur les ports 1433 et 11000-11999. Les paquets étant directement dirigés vers la base de données, l’utilisation d’une redirection apporte des améliorations en matière de débit et de latence, par rapport à l’utilisation d’un proxy.
- **Proxy (valeur par défaut) :** dans ce mode, toutes les connexions utilisent un composant passerelle proxy. Pour activer la connectivité, seuls le port 1433 pour les réseaux privés et le port 3342 pour la connexion publique doivent être ouverts. Ce mode peut entraîner une latence plus élevée et un débit inférieur, en fonction de la nature de la charge de travail. Nous vous recommandons la stratégie de connexion redirigée plutôt que la stratégie de connexion proxy pour bénéficier d’une plus faible latence et d’un débit plus élevé.

## <a name="redirect-connection-type"></a>Type de connexion redirigée

Avec le type de connexion redirigée, une fois la session TCP au moteur SQL établie, la session cliente obtient de l’équilibreur de charge l’adresse IP virtuelle de destination du nœud de cluster virtuel à partir. Les paquets suivants sont directement acheminés vers le nœud de cluster virtuel, en ignorant la passerelle. Le schéma suivant illustre ce flux de trafic :

![Diagramme représentant un réseau local où redirect-find-db est connectée à une passerelle dans un réseau virtuel Azure et où redirect-query est connectée à un nœud principal de base de données dans le réseau virtuel.](./media/connection-types-overview/redirect.png)

> [!IMPORTANT]
> Actuellement, le type de connexion redirigée ne fonctionne que pour un point de terminaison privé. Quel que soit le paramètre de type de connexion, les connexions entrantes par le biais du point de terminaison public s’effectuent via un proxy.

## <a name="proxy-connection-type"></a>Type de connexion Proxy

Avec le type de connexion proxy, la session TCP est établie à l’aide de la passerelle et que tous les paquets suivants transitent par elle. Le schéma suivant illustre ce flux de trafic :

![Diagramme représentant un réseau local où un proxy est connecté à une passerelle dans un réseau virtuel Azure, avec connexion en regard d'un nœud principal de base de données dans le réseau virtuel.](./media/connection-types-overview/proxy.png)

## <a name="changing-connection-type"></a>Modification du type de connexion

- **Utilisation du portail :** pour modifier le type de connexion via le portail Azure, ouvrez la page Réseau virtuel et utilisez le paramètre **Type de connexion** pour modifier le type de connexion et enregistrer les modifications.

- **Script pour modifier les paramètres de type de connexion à l’aide de PowerShell :**

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Le script PowerShell suivant montre comment modifier le type de connexion SQL à `Redirect` pour une instance managée.

```powershell
Install-Module -Name Az
Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount
# Get your SubscriptionId from the Get-AzSubscription command
Get-AzSubscription
# Use your SubscriptionId in place of {subscription-id} below
Select-AzSubscription -SubscriptionId {subscription-id}
# Replace {rg-name} with the resource group for your managed instance, and replace {mi-name} with the name of your managed instance
$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}
$mi = $mi | Set-AzSqlInstance -ProxyOverride "Redirect" -force
```

## <a name="next-steps"></a>Étapes suivantes

- [Restaurer une base de données sur SQL Managed Instance](restore-sample-database-quickstart.md)
- Découvrez comment [configurer un point de terminaison public sur une instance managée SQL](public-endpoint-configure.md)
- Apprenez-en davantage sur l’[architecture de connectivité SQL Managed Instance](connectivity-architecture-overview.md)
