---
title: Configurer un point de terminaison public - Azure SQL Managed Instance
description: Découvrir comment configurer un point de terminaison public pour Azure SQL Managed Instance
services: sql-database
ms.service: sql-managed-instance
ms.subservice: security
ms.custom: sqldbrb=1
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: vanto, sstein
ms.date: 05/07/2019
ms.openlocfilehash: 73fa4d4988c7a036dc1d2eb7dc81c3c1c5d77026
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92788279"
---
# <a name="configure-public-endpoint-in-azure-sql-managed-instance"></a>Configurer un point de terminaison public dans Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Un point de terminaison public pour une [instance gérée](./sql-managed-instance-paas-overview.md) permet l’accès aux données de votre instance gérée depuis un emplacement extérieur au [réseau virtuel](../../virtual-network/virtual-networks-overview.md). Vous pouvez accéder à votre instance gérée à partir de services Azure multilocataires, tels que Power BI et Azure App Service, ou d’un réseau local. Comme vous utilisez le point de terminaison public sur une instance gérée, vous n’avez pas besoin de recourir à un VPN, ce qui peut aider à éviter les problèmes de débit VPN.

Dans cet article, vous allez apprendre à :

> [!div class="checklist"]
>
> - Activer un point de terminaison public pour votre instance gérée dans le Portail Microsoft Azure
> - Activer un point de terminaison public pour votre instance gérée à l’aide de PowerShell
> - Configurer votre groupe de sécurité réseau d’instance gérée pour autoriser le trafic vers le point de terminaison public de l’instance gérée
> - Obtenir la chaîne de connexion du point de terminaison public de l’instance gérée

## <a name="permissions"></a>Autorisations

En raison de la sensibilité des données qui se trouvent sur une instance gérée, la configuration permettant d’activer le point de terminaison public de ce type d’instance repose sur un processus en deux étapes. Cette mesure de sécurité est conforme à la séparation des tâches :

- L’activation d’un point de terminaison public sur une instance gérée doit être effectuée par l’administrateur de cette instance. Cet administrateur est indiqué sur la page **Vue d’ensemble** de votre ressource d’instance gérée.
- L’autorisation du trafic à l’aide d’un groupe de sécurité réseau doit être effectuée par un administrateur réseau. Pour en savoir plus, consultez la liste des [permissions du groupe de sécurité réseau](../../virtual-network/manage-network-security-group.md#permissions).

## <a name="enabling-public-endpoint-for-a-managed-instance-in-the-azure-portal"></a>Activation d’un point de terminaison public pour une instance gérée dans le Portail Microsoft Azure

1. Démarrez le Portail Microsoft Azure sur <https://portal.azure.com/.>
1. Ouvrez le groupe de ressources avec l’instance gérée, puis sélectionnez **l’instance gérée SQL** sur laquelle vous souhaitez configurer le point de terminaison.
1. Dans la page de paramètres **Sécurité** , sélectionnez l’onglet **Réseau virtuel** .
1. Dans la page de configuration du réseau virtuel, sélectionnez **Activer** , puis cliquez sur l’icône **Enregistrer** afin de mettre à jour la configuration.

![Capture d'écran représentant une page Réseau virtuel de SQL Managed Instance, sur laquelle l'option Point de terminaison public est activée.](./media/public-endpoint-configure/mi-vnet-config.png)

## <a name="enabling-public-endpoint-for-a-managed-instance-using-powershell"></a>Activation d’un point de terminaison public pour une instance gérée à l’aide de PowerShell

### <a name="enable-public-endpoint"></a>Activer un point de terminaison public

Exécutez les commandes PowerShell suivantes. Remplacez la valeur **subscription-id** par l’identifiant de votre abonnement. Remplacez également la valeur **rg-name** par le nom du groupe de ressources de votre instance gérée, puis remplacez **mi-name** par le nom de cette dernière.

```powershell
Install-Module -Name Az

Import-Module Az.Accounts
Import-Module Az.Sql

Connect-AzAccount

# Use your subscription ID in place of subscription-id below

Select-AzSubscription -SubscriptionId {subscription-id}

# Replace rg-name with the resource group for your managed instance, and replace mi-name with the name of your managed instance

$mi = Get-AzSqlInstance -ResourceGroupName {rg-name} -Name {mi-name}

$mi = $mi | Set-AzSqlInstance -PublicDataEndpointEnabled $true -force
```

### <a name="disable-public-endpoint"></a>Désactiver un point de terminaison public

Pour désactiver un point de terminaison public à l’aide de PowerShell, vous devez exécuter la commande suivante (sans oublier de fermer le groupe de sécurité réseau du port entrant 3342, s’il est configuré) :

```powershell
Set-AzSqlInstance -PublicDataEndpointEnabled $false -force
```

## <a name="allow-public-endpoint-traffic-on-the-network-security-group"></a>Autoriser le trafic du point de terminaison public sur le groupe de sécurité réseau

1. Si la page de configuration de l’instance gérée est toujours ouverte, accédez à l’onglet **Vue d’ensemble** . Dans le cas contraire, revenez à votre ressource **d’instance gérée SQL** . Sélectionnez le lien **Réseau/sous-réseau virtuel** , qui vous permet d’accéder à la page de configuration du réseau virtuel.

    ![Capture d'écran représentant la page de configuration du réseau virtuel dans laquelle vous pouvez trouver la valeur de votre réseau/sous-réseau virtuel.](./media/public-endpoint-configure/mi-overview.png)

1. Sélectionnez l’onglet **Sous-réseaux** figurant dans le volet de configuration du réseau virtuel, sur la gauche, et notez le **GROUPE DE SÉCURITÉ** de votre instance gérée.

    ![Capture d'écran représentant l'onglet Sous-réseau, dans lequel vous pouvez accéder au GROUPE DE SÉCURITÉ de votre instance gérée.](./media/public-endpoint-configure/mi-vnet-subnet.png)

1. Revenez au groupe de ressources contenant votre instance gérée. Vous devez voir le nom du **groupe de sécurité réseau** indiqué ci-dessus, que vous avez noté. Sélectionnez ce nom pour accéder à la page de configuration du groupe de sécurité réseau.

1. Sélectionnez l’onglet **Règles de sécurité de trafic entrant** , et **ajoutez** une règle ayant une priorité plus élevée que la règle **deny_all_inbound** avec les paramètres suivants : </br> </br>

    |Paramètre  |Valeur suggérée  |Description  |
    |---------|---------|---------|
    |**Source**     |N’importe quelle adresse IP ou balise de service         |<ul><li>Pour les services Azure tels que Power BI, sélectionnez la balise de service Azure Cloud</li> <li>Pour votre ordinateur ou machine virtuelle Azure, utilisez l’adresse IP NAT.</li></ul> |
    |**Plages de ports source**     |* |Laissez la valeur * (tout) pour cette option, car les ports sources sont généralement alloués de manière dynamique et, de ce fait, imprévisibles |
    |**Destination**     |Quelconque         |En conservant la valeur Tout pour le paramètre Destination, vous autorisez le trafic au sein du sous-réseau de l’instance gérée |
    |**Plages de ports de destination**     |3342         |Définissez la portée du port de destination sur 3342, qui est le point de terminaison TDS public de l’instance gérée |
    |**Protocole**     |TCP         |SQL Managed Instance utilise le protocole TCP pour TDS |
    |**Action**     |Allow         |Autorisez le trafic entrant vers une instance gérée par le biais du point de terminaison public |
    |**Priorité**     |1 300         |Assurez-vous que cette règle présente une priorité plus élevée que la règle **deny_all_inbound** |

    ![Capture d'écran représentant les Règles de sécurité de trafic entrant, qui contiennent la nouvelle règle public_endpoint_inbound et la règle deny_all_inbound.](./media/public-endpoint-configure/mi-nsg-rules.png)

    > [!NOTE]
    > Le port 3342 est utilisé pour les connexions du point de terminaison public à l’instance gérée ; il ne peut pas être changé à ce stade.

## <a name="obtaining-the-managed-instance-public-endpoint-connection-string"></a>Obtention de la chaîne de connexion du point de terminaison public de l’instance gérée

1. Accédez à la page de configuration de l’instance gérée qui a été activée pour le point de terminaison public. Sélectionnez l’onglet **Chaînes de connexion** sous la configuration **Paramètres** .
1. Notez que le nom d’hôte du point de terminaison public présente le format <nom_mi>. **public** . <zone_dns>. database.windows.net et que le port utilisé pour la connexion est 3342.

    ![Capture d'écran représentant les chaînes de connexion de vos points de terminaison publics et privés.](./media/public-endpoint-configure/mi-public-endpoint-conn-string.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [Azure SQL Managed Instance en toute sécurité avec un point de terminaison public](public-endpoint-overview.md).