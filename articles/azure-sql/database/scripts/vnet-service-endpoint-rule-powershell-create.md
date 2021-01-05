---
title: PowerShell pour les règles et points de terminaison de réseau virtuel pour les bases de données uniques et mises en pool
description: Fournit des scripts PowerShell pour créer et gérer des points de terminaison de service virtuel pour Azure SQL Database et Azure Synapse.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.devlang: PowerShell
ms.topic: conceptual
author: rohitnayakmsft
ms.author: rohitna
ms.reviewer: vanto
ms.date: 04/17/2019
ms.custom: sqldbrb=1
tags: azure-synapse
ms.openlocfilehash: f32599c9d289c8fc5e86eb8c7b0574d9703a6dd4
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792665"
---
# <a name="powershell-create-a-virtual-service-endpoint-and-vnet-rule-for-azure-sql-database"></a>PowerShell : Créer un point de terminaison de service virtuel et une règle de réseau virtuel pour Azure SQL Database
[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Les *règles de réseau virtuel* constituent une fonctionnalité de sécurité du pare-feu. Elles permettent de déterminer si le [serveur SQL logique](../logical-servers.md) de vos bases de données [Azure SQL Database](../sql-database-paas-overview.md), de vos pools élastiques ou de vos bases de données [Azure Synapse](../../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) doit accepter les communications provenant de sous-réseaux spécifiques de réseaux virtuels.

> [!IMPORTANT]
> Cet article s’applique à Azure SQL Database ainsi qu’à Azure Synapse (anciennement SQL DW). Par souci de simplicité, le terme Azure SQL Database dans cet article s’applique aux bases de données appartenant à Azure SQL Database ou Azure Synapse. Cet article ne s’applique *pas* à Azure SQL Managed Instance, car aucun point de terminaison de service ne lui est associé.

Cet article présente un script PowerShell qui effectue les actions suivantes :

1. Crée un *point de terminaison de service virtuel* Microsoft Azure sur votre sous-réseau.
2. Ajoute le point de terminaison au pare-feu de votre serveur afin de créer une *règle de réseau virtuel* .

Pour plus d’informations, consultez [Points de terminaison de service virtuel pour Azure SQL Database][sql-db-vnet-service-endpoint-rule-overview-735r].

> [!TIP]
> Si vous devez uniquement évaluer ou ajouter le point de terminaison de service virtuel *nom de type* pour Azure SQL Database à votre sous-réseau, vous pouvez passer directement à notre [script PowerShell plus direct](#a-verify-subnet-is-endpoint-ps-100).

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés aux [cmdlets `Az.Sql`](/powershell/module/az.sql). Pour obtenir l’ancien module, consultez [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

## <a name="major-cmdlets"></a>Principales applets de commande

Cet article se concentre sur la [cmdlet **New-AzSqlServerVirtualNetworkRule**](/powershell/module/az.sql/new-azsqlservervirtualnetworkrule) qui ajoute le point de terminaison de sous-réseau à la liste de contrôle d’accès (ACL, access-control list) de votre serveur, créant ainsi une règle.

La liste suivante montre la séquence des autres *principales* applets de commande que vous devez exécuter pour préparer votre appel à **New-AzSqlServerVirtualNetworkRule** . Dans cet article, ces appels se produisent dans le [script 3 « Règle de réseau virtuel »](#a-script-30) :

1. [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) : crée un objet sous-réseau.
2. [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) : crée votre réseau virtuel et lui attribue le sous-réseau.
3. [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Set-azVirtualNetworkSubnetConfig) : assigne un point de terminaison de service virtuel à votre sous-réseau.
4. [Set-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork) : permet de conserver les mises à jour apportées à votre réseau virtuel.
5. [New-AzSqlServerVirtualNetworkRule](/powershell/module/az.sql/new-azsqlservervirtualnetworkrule) : lorsque votre sous-réseau devient un point de terminaison, il est ajouté en tant que règle de réseau virtuel à l’ACL de votre serveur.
   - Cette cmdlet propose le paramètre **-IgnoreMissingVNetServiceEndpoint** , à compter du module AzureRM PowerShell version 5.1.1.

## <a name="prerequisites-for-running-powershell"></a>Prérequis pour l’exécution de PowerShell

- Vous pouvez déjà vous connecter à Azure, par exemple par le biais du [Portail Azure][http-azure-portal-link-ref-477t].
- Vous pouvez déjà exécuter des scripts PowerShell.

> [!NOTE]
> Assurez-vous que les points de terminaison de service sont activés pour le réseau virtuel/sous-réseau que vous voulez ajouter à votre serveur, sans quoi la création de la règle de pare-feu de réseau virtuel échoue.

## <a name="one-script-divided-into-four-chunks"></a>Un script est divisé en quatre parties

Notre script PowerShell de démonstration est divisé en une séquence de scripts plus petits. La division facilite l’apprentissage et offre plus de souplesse. Les scripts doivent être exécutés dans la séquence indiquée. Si vous n’avez pas le temps d’exécuter les scripts maintenant, nos résultats de test réels sont affichés après le script 4.

<a name="a-script-10"></a>

### <a name="script-1-variables"></a>Script 1 : Variables

Ce premier script PowerShell attribue des valeurs aux variables. Les scripts suivants dépendent de ces variables.

> [!IMPORTANT]
> Avant d’exécuter ce script, vous pouvez modifier les valeurs, si vous le souhaitez. Par exemple, si vous disposez déjà d’un groupe de ressources, vous pouvez remplacer le nom de votre groupe de ressources par la valeur attribuée.
>
> Le nom de votre abonnement doit être modifié dans le script.

### <a name="powershell-script-1-source-code"></a>Code source du script PowerShell 1

```powershell
######### Script 1 ########################################
##   LOG into to your Azure account.                     ##
##   (Needed only one time per powershell.exe session.)  ##
###########################################################

$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]';
if ('yes' -eq $yesno) { Connect-AzAccount; }

###########################################################
##  Assignments to variables used by the later scripts.  ##
###########################################################

# You can edit these values, if necessary.
$SubscriptionName = 'yourSubscriptionName';
Select-AzSubscription -SubscriptionName $SubscriptionName;

$ResourceGroupName = 'RG-YourNameHere';
$Region            = 'westcentralus';

$VNetName            = 'myVNet';
$SubnetName          = 'mySubnet';
$VNetAddressPrefix   = '10.1.0.0/16';
$SubnetAddressPrefix = '10.1.1.0/24';
$VNetRuleName        = 'myFirstVNetRule-ForAcl';

$SqlDbServerName         = 'mysqldbserver-forvnet';
$SqlDbAdminLoginName     = 'ServerAdmin';
$SqlDbAdminLoginPassword = 'ChangeYourAdminPassword1';

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql';  # Official type name.

Write-Host 'Completed script 1, the "Variables".';
```

<a name="a-script-20"></a>

### <a name="script-2-prerequisites"></a>Script 2 : Prérequis

Ce script prépare le script suivant, où se déroule l’action du point de terminaison. Ce script crée pour vous les éléments suivants, mais uniquement s’ils n’existent pas déjà. Vous pouvez ignorer le script 2 si vous êtes sûr que ces éléments existent déjà :

- Groupe de ressources Azure
- Serveur SQL logique

### <a name="powershell-script-2-source-code"></a>Code source du script PowerShell 2

```powershell
######### Script 2 ########################################
##   Ensure your Resource Group already exists.          ##
###########################################################

Write-Host "Check whether your Resource Group already exists.";

$gottenResourceGroup = $null;

$gottenResourceGroup = Get-AzResourceGroup `
  -Name        $ResourceGroupName `
  -ErrorAction SilentlyContinue;

if ($null -eq $gottenResourceGroup)
{
    Write-Host "Creating your missing Resource Group - $ResourceGroupName.";

    $gottenResourceGroup = New-AzResourceGroup `
      -Name $ResourceGroupName `
      -Location $Region;

    $gottenResourceGroup;
}
else { Write-Host "Good, your Resource Group already exists - $ResourceGroupName."; }

$gottenResourceGroup = $null;

###########################################################
## Ensure your server already exists. ##
###########################################################

Write-Host "Check whether your server already exists.";

$sqlDbServer = $null;

$sqlDbServer = Get-AzSqlServer `
  -ResourceGroupName $ResourceGroupName `
  -ServerName        $SqlDbServerName `
  -ErrorAction       SilentlyContinue;

if ($null -eq $sqlDbServer) {
    Write-Host "Creating the missing server - $SqlDbServerName.";
    Write-Host "Gather the credentials necessary to next create a server.";

    $sqlAdministratorCredentials = New-Object `
      -TypeName     System.Management.Automation.PSCredential `
      -ArgumentList `
        $SqlDbAdminLoginName, `
        $(ConvertTo-SecureString `
            -String      $SqlDbAdminLoginPassword `
            -AsPlainText `
            -Force `
         );

    if ($null -eq $sqlAdministratorCredentials) {
        Write-Host "ERROR, unable to create SQL administrator credentials.  Now ending.";
        return;
    }

    Write-Host "Create your server.";

    $sqlDbServer = New-AzSqlServer `
      -ResourceGroupName $ResourceGroupName `
      -ServerName        $SqlDbServerName `
      -Location          $Region `
      -SqlAdministratorCredentials $sqlAdministratorCredentials;

    $sqlDbServer;
}
else {
    Write-Host "Good, your server already exists - $SqlDbServerName.";
}

$sqlAdministratorCredentials = $null;
$sqlDbServer                 = $null;

Write-Host 'Completed script 2, the "Prerequisites".';
```

<a name="a-script-30"></a>

## <a name="script-3-create-an-endpoint-and-a-rule"></a>Script 3 : Créer un point de terminaison et une règle

Ce script crée un réseau virtuel avec un sous-réseau. Le script attribue ensuite le type de point de terminaison **Microsoft.Sql** à votre sous-réseau. Enfin, le script ajoute votre sous-réseau à l’ACL, créant ainsi une règle.

### <a name="powershell-script-3-source-code"></a>Code source du script PowerShell 3

```powershell
######### Script 3 ########################################
##   Create your virtual network, and give it a subnet.  ##
###########################################################

Write-Host "Define a subnet '$SubnetName', to be given soon to a virtual network.";

$subnet = New-AzVirtualNetworkSubnetConfig `
  -Name            $SubnetName `
  -AddressPrefix   $SubnetAddressPrefix `
  -ServiceEndpoint $ServiceEndpointTypeName_SqlDb;

Write-Host "Create a virtual network '$VNetName'." `
  "  Give the subnet to the virtual network that we created.";

$vnet = New-AzVirtualNetwork `
  -Name              $VNetName `
  -AddressPrefix     $VNetAddressPrefix `
  -Subnet            $subnet `
  -ResourceGroupName $ResourceGroupName `
  -Location          $Region;

###########################################################
##   Create a Virtual Service endpoint on the subnet.    ##
###########################################################

Write-Host "Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.";

$vnet = Set-AzVirtualNetworkSubnetConfig `
  -Name            $SubnetName `
  -AddressPrefix   $SubnetAddressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint $ServiceEndpointTypeName_SqlDb;

Write-Host "Persist the updates made to the virtual network > subnet.";

$vnet = Set-AzVirtualNetwork `
  -VirtualNetwork $vnet;

$vnet.Subnets[0].ServiceEndpoints;  # Display the first endpoint.

###########################################################
##   Add the Virtual Service endpoint Id as a rule,      ##
##   into SQL Database ACLs.                             ##
###########################################################

Write-Host "Get the subnet object.";

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Name              $VNetName;

$subnet = Get-AzVirtualNetworkSubnetConfig `
  -Name           $SubnetName `
  -VirtualNetwork $vnet;

Write-Host "Add the subnet .Id as a rule, into the ACLs for your server.";

$vnetRuleObject1 = New-AzSqlServerVirtualNetworkRule `
  -ResourceGroupName      $ResourceGroupName `
  -ServerName             $SqlDbServerName `
  -VirtualNetworkRuleName $VNetRuleName `
  -VirtualNetworkSubnetId $subnet.Id;

$vnetRuleObject1;

Write-Host "Verify that the rule is in the SQL Database ACL.";

$vnetRuleObject2 = Get-AzSqlServerVirtualNetworkRule `
  -ResourceGroupName      $ResourceGroupName `
  -ServerName             $SqlDbServerName `
  -VirtualNetworkRuleName $VNetRuleName;

$vnetRuleObject2;

Write-Host 'Completed script 3, the "Virtual-Network-Rule".';
```

<a name="a-script-40"></a>

## <a name="script-4-clean-up"></a>Script 4 : Nettoyage

Ce dernier script supprime les ressources que les scripts précédents ont créées pour la démonstration. Toutefois, le script vous demande de confirmer la suppression des éléments suivants :

- Serveur SQL logique
- Groupe de ressources Azure

Une fois le script 1 terminé, vous pouvez exécuter le script 4 à tout moment.

### <a name="powershell-script-4-source-code"></a>Code source du script PowerShell 4

```powershell
######### Script 4 ########################################
##   Clean-up phase A:  Unconditional deletes.           ##
##                                                       ##
##   1. The test rule is deleted from SQL Database ACL.        ##
##   2. The test endpoint is deleted from the subnet.    ##
##   3. The test virtual network is deleted.             ##
###########################################################

Write-Host "Delete the rule from the SQL Database ACL.";

Remove-AzSqlServerVirtualNetworkRule `
  -ResourceGroupName      $ResourceGroupName `
  -ServerName             $SqlDbServerName `
  -VirtualNetworkRuleName $VNetRuleName `
  -ErrorAction            SilentlyContinue;

Write-Host "Delete the endpoint from the subnet.";

$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Name              $VNetName;

Remove-AzVirtualNetworkSubnetConfig `
  -Name $SubnetName `
  -VirtualNetwork $vnet;

Write-Host "Delete the virtual network (thus also deletes the subnet).";

Remove-AzVirtualNetwork `
  -Name              $VNetName `
  -ResourceGroupName $ResourceGroupName `
  -ErrorAction       SilentlyContinue;

###########################################################
##   Clean-up phase B:  Conditional deletes.             ##
##                                                       ##
##   These might have already existed, so user might     ##
##   want to keep.                                       ##
##                                                       ##
##   1. Logical SQL server                        ##
##   2. Azure resource group                             ##
###########################################################

$yesno = Read-Host 'CAUTION !: Do you want to DELETE your server AND your resource group?  [yes/no]';
if ('yes' -eq $yesno) {
    Write-Host "Remove the server.";

    Remove-AzSqlServer `
      -ServerName        $SqlDbServerName `
      -ResourceGroupName $ResourceGroupName `
      -ErrorAction       SilentlyContinue;

    Write-Host "Remove the Azure Resource Group.";

    Remove-AzResourceGroup `
      -Name        $ResourceGroupName `
      -ErrorAction SilentlyContinue;
}
else {
    Write-Host "Skipped over the DELETE of SQL Database and resource group.";
}

Write-Host 'Completed script 4, the "Clean-Up".';
```

<a name="a-actual-output"></a>

<a name="a-verify-subnet-is-endpoint-ps-100"></a>

## <a name="verify-your-subnet-is-an-endpoint"></a>Vérifiez que votre sous-réseau est un point de terminaison

Vous disposez peut-être d’un sous-réseau auquel le nom de type **Microsoft.Sql** a déjà été attribué, ce qui signifie qu’il est déjà un point de terminaison de service virtuel. Vous pouvez utiliser le [Portail Azure][http-azure-portal-link-ref-477t] pour créer une règle de réseau virtuel à partir du point de terminaison.

Ou, vous n’êtes peut-être pas certain que votre sous-réseau comporte un nom de type **Microsoft.Sql** . Vous pouvez exécuter le script PowerShell suivant pour effectuer ces actions :

1. Assurez-vous que votre sous-réseau comporte le nom de type **Microsoft.Sql** .
2. Attribuez éventuellement le nom de type s’il est absent.
    - Le script vous invite à *confirmer* que le nom de type est absent.

### <a name="phases-of-the-script"></a>Phases du script

Voici les phases du script PowerShell :

1. Connectez-vous à votre compte Azure, nécessaire une seule fois par session PS.  Attribuez les variables.
2. Recherchez votre réseau virtuel, puis votre sous-réseau.
3. Votre sous-réseau est-il étiqueté comme type de serveur de point de terminaison  **Microsoft.Sql** ?
4. Ajoutez un point de terminaison de service virtuel avec le nom de type **Microsoft.Sql** sur votre sous-réseau.

> [!IMPORTANT]
> Avant d’exécuter ce script, vous devez modifier les valeurs affectées aux variables $, au début du script.

### <a name="direct-powershell-source-code"></a>Code de source PowerShell direct

Ce script PowerShell ne met rien à jour, sauf si vous répondez Oui lorsque PowerShell vous demande confirmation. Le script peut ajouter le nom de type **Microsoft.Sql** à votre sous-réseau. Mais le script essaie d’ajouter ce nom de type uniquement si votre sous-réseau n’en dispose pas.

```powershell
### 1. LOG into to your Azure account, needed only once per PS session.  Assign variables.
$yesno = Read-Host 'Do you need to log into Azure (only one time per powershell.exe session)?  [yes/no]';
if ('yes' -eq $yesno) { Connect-AzAccount; }

# Assignments to variables used by the later scripts.
# You can EDIT these values, if necessary.

$SubscriptionName  = 'yourSubscriptionName';
Select-AzSubscription -SubscriptionName "$SubscriptionName";

$ResourceGroupName   = 'yourRGName';
$VNetName            = 'yourVNetName';
$SubnetName          = 'yourSubnetName';
$SubnetAddressPrefix = 'Obtain this value from the Azure portal.'; # Looks roughly like: '10.0.0.0/24'

$ServiceEndpointTypeName_SqlDb = 'Microsoft.Sql';  # Do NOT edit. Is official value.

### 2. Search for your virtual network, and then for your subnet.
# Search for the virtual network.
$vnet = $null;
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName $ResourceGroupName `
  -Name              $VNetName;

if ($vnet -eq $null) {
    Write-Host "Caution: No virtual network found by the name '$VNetName'.";
    Return;
}

$subnet = $null;
for ($nn=0; $nn -lt $vnet.Subnets.Count; $nn++) {
    $subnet = $vnet.Subnets[$nn];
    if ($subnet.Name -eq $SubnetName)
    { break; }
    $subnet = $null;
}

if ($subnet -eq $null) {
    Write-Host "Caution: No subnet found by the name '$SubnetName'";
    Return;
}

### 3. Is your subnet tagged as 'Microsoft.Sql' endpoint server type?
$endpointMsSql = $null;
for ($nn=0; $nn -lt $subnet.ServiceEndpoints.Count; $nn++) {
    $endpointMsSql = $subnet.ServiceEndpoints[$nn];
    if ($endpointMsSql.Service -eq $ServiceEndpointTypeName_SqlDb) {
        $endpointMsSql;
        break;
    }
    $endpointMsSql = $null;
}

if ($endpointMsSql -ne $null) {
    Write-Host "Good: Subnet found, and is already tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'.";
    Return;
}
else {
    Write-Host "Caution: Subnet found, but not yet tagged as an endpoint of type '$ServiceEndpointTypeName_SqlDb'.";

    # Ask the user for confirmation.
    $yesno = Read-Host 'Do you want the PS script to apply the endpoint type name to your subnet?  [yes/no]';
    if ('no' -eq $yesno) { Return; }
}

### 4. Add a Virtual Service endpoint of type name 'Microsoft.Sql', on your subnet.
$vnet = Set-AzVirtualNetworkSubnetConfig `
  -Name            $SubnetName `
  -AddressPrefix   $SubnetAddressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint $ServiceEndpointTypeName_SqlDb;

# Persist the subnet update.
$vnet = Set-AzVirtualNetwork `
  -VirtualNetwork $vnet;

for ($nn=0; $nn -lt $vnet.Subnets.Count; $nn++) {
    $vnet.Subnets[0].ServiceEndpoints; }  # Display.
```

<!-- Link references: -->
[sql-db-vnet-service-endpoint-rule-overview-735r]:../vnet-service-endpoint-rule-overview.md
[http-azure-portal-link-ref-477t]: https://portal.azure.com/