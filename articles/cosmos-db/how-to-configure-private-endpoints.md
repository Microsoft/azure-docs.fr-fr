---
title: Configurer Azure Private Link pour un compte Azure Cosmos
description: Découvrez comment configurer Azure Private Link pour accéder à un compte Azure Cosmos à l’aide d’une adresse IP privée dans un réseau virtuel.
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/18/2020
ms.author: thweiss
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4ba4e5f462a3cc88de5b23b32a5e749f9363e93f
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93081890"
---
# <a name="configure-azure-private-link-for-an-azure-cosmos-account"></a>Configurer Azure Private Link pour un compte Azure Cosmos
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

À l’aide d’Azure Private Link, vous pouvez vous connecter à un compte Azure Cosmos via un point de terminaison privé. Le point de terminaison privé est un ensemble d’adresses IP privées dans un sous-réseau au sein de votre réseau virtuel. Vous pouvez alors limiter l’accès à un compte Azure Cosmos via des adresses IP privées. Lorsque Private Link est combiné à des stratégies NSG limitées, il permet de réduire le risque d’exfiltration de données. Pour plus d’informations sur les points de terminaison privés, consultez l’article [Azure Private Link](../private-link/private-link-overview.md).

> [!NOTE]
> Azure Private Link n’empêche pas la résolution de vos points de terminaison Azure Cosmos par le DNS public. Le filtrage des requêtes entrantes se produit au niveau de l’application, et non au niveau du transport ou du réseau.

Private Link permet aux utilisateurs d’accéder à un compte Azure Cosmos à partir du réseau virtuel ou de tout réseau virtuel appairé. Les ressources mappées à Private Link sont également accessibles localement via un Peering privé par le biais d’un VPN ou d’Azure ExpressRoute. 

Vous pouvez vous connecter à un compte Azure Cosmos configuré avec Private Link en utilisant les méthodes d’approbation automatique ou manuelle. Pour plus d’informations, consultez la section [Flux de travail d’approbation](../private-link/private-endpoint-overview.md#access-to-a-private-link-resource-using-approval-workflow) de la documentation Private Link. 

Cet article décrit pas à pas la création d’un point de terminaison privé. Il suppose que vous utilisez la méthode d’approbation automatique.

## <a name="create-a-private-endpoint-by-using-the-azure-portal"></a>Créer un point de terminaison privé au moyen du Portail Azure

Afin de créer un point de terminaison privé pour un compte Azure Cosmos existant à l’aide du Portail Azure, procédez comme suit :

1. Dans le volet **Toutes les ressources** , choisissez un compte Azure Cosmos.

1. Sélectionnez **Connexions au point de terminaison privé** dans la liste de paramètres, puis **Point de terminaison privé**  :

   :::image type="content" source="./media/how-to-configure-private-endpoints/create-private-endpoint-portal.png" alt-text="Sélections pour créer un point de terminaison privé dans le Portail Azure":::

1. Dans le volet **Créer un point de terminaison privé – Concepts de base** , entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez un groupe de ressources.|
    | **Détails de l’instance** |  |
    | Nom | Entrez un nom pour votre point de terminaison privé. Si ce nom est utilisé, créez-en un unique. |
    |Région| Sélectionnez la région dans laquelle vous souhaitez déployer Private Link. Créez le point de terminaison privé au même emplacement que votre réseau virtuel.|
    |||
1. Sélectionnez **Suivant : Ressource**.
1. Dans **Créer un point de terminaison privé - Ressource** , entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    |Méthode de connexion  | Sélectionnez **Se connecter à une ressource Azure dans mon répertoire**. <br/><br/> Vous pouvez ensuite choisir l’une de vos ressources pour configurer Private Link. Ou vous pouvez vous connecter à la ressource d’un autre utilisateur à l’aide d’un ID ou d’un alias de ressource qu’il a partagé avec vous.|
    | Abonnement| Sélectionnez votre abonnement. |
    | Type de ressource | Sélectionnez **Microsoft.AzureCosmosDB/databaseAccounts**. |
    | Ressource |Sélectionnez votre compte Azure Cosmos. |
    |Sous-ressource cible |Sélectionnez le type d’API Azure Cosmos DB que vous souhaitez mapper. Par défaut, il n’y a qu’un seul choix pour les API SQL, MongoDB et Cassandra. Pour les API Gremlin et Table, vous pouvez également choisir **Sql** , car ces API sont interopérables avec l’API SQL. |
    |||

1. Sélectionnez **Suivant : Configuration**.
1. Dans **Créer un point de terminaison privé – Configuration** , entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    |**Mise en réseau**| |
    | Réseau virtuel| Sélectionnez votre réseau virtuel. |
    | Subnet | Sélectionnez votre sous-réseau. |
    |**Intégration à un DNS privé**||
    |Intégrer à une zone DNS privée |Sélectionnez **Oui**. <br><br/> Pour vous connecter en privé à votre point de terminaison privé, vous avez besoin d’un enregistrement DNS. Nous vous recommandons d’intégrer votre point de terminaison privé à une zone DNS privée. Vous pouvez également utiliser vos propres serveurs DNS ou créer des enregistrements DNS à l’aide des fichiers hôtes sur vos machines virtuelles. |
    |Zone DNS privée |Sélectionnez **privatelink.documents.azure.com**. <br><br/> La zone DNS privée est déterminée automatiquement. Vous ne pouvez pas la modifier à l’aide du Portail Azure.|
    |||

1. Sélectionnez **Revoir + créer**. Sur la page **Revoir + créer** , Azure valide votre configuration.
1. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

Si vous avez approuvé Private Link pour un compte Azure Cosmos, dans le Portail Azure, l’option **Tous les réseaux** dans le volet **Pare-feu et réseaux virtuels** est désactivée.

Le tableau suivant montre le mappage entre différents types d’API de compte Azure Cosmos, les sous-ressources prises en charge et les noms de zones privées correspondants. Vous pouvez également accéder aux comptes API Gremlin et Table via l’API SQL : il y a donc deux entrées pour ces API.

|Type d’API de compte Azure Cosmos  |Sous-ressources prises en charge (ou ID de groupe) |Nom de zone privée  |
|---------|---------|---------|
|SQL    |   SQL      | privatelink.documents.azure.com   |
|Cassandra    | Cassandra        |  privatelink.cassandra.cosmos.azure.com    |
|Mongo   |  MongoDB       |  privatelink.mongo.cosmos.azure.com    |
|Gremlin     | Gremlin        |  privatelink.gremlin.cosmos.azure.com   |
|Gremlin     |  SQL       |  privatelink.documents.azure.com    |
|Table de charge de travail    |    Table de charge de travail     |   privatelink.table.cosmos.azure.com    |
|Table de charge de travail     |   SQL      |  privatelink.documents.azure.com    |

### <a name="fetch-the-private-ip-addresses"></a>Récupérer les adresses IP privées

Une fois le point de terminaison privé provisionné, vous pouvez interroger les adresses IP. Pour afficher les adresses IP à partir du Portail Azure :

1. Sélectionnez **Toutes les ressources**.
1. Recherchez le point de terminaison privé que vous avez créé précédemment. Dans ce cas, il s’agit de **cdbPrivateEndpoint3**.
1. Sélectionnez l’onglet **Vue d’ensemble** pour afficher les paramètres DNS et les adresses IP.

:::image type="content" source="./media/how-to-configure-private-endpoints/private-ip-addresses-portal.png" alt-text="Adresses IP privées dans le Portail Azure":::

Plusieurs adresses IP sont créées par point de terminaison privé :

* Une pour le point de terminaison global (indépendant de la région) du compte Azure Cosmos
* Une pour chaque région dans laquelle le compte Azure Cosmos est déployé

## <a name="create-a-private-endpoint-by-using-azure-powershell"></a>Créer un point de terminaison privé à l’aide d’Azure PowerShell

Exécutez le script PowerShell suivant afin de créer un point de terminaison privé nommé « MyPrivateEndpoint » pour un compte Azure Cosmos existant. Remplacez les valeurs des variables par les informations de votre environnement.

```azurepowershell-interactive
$SubscriptionId = "<your Azure subscription ID>"
# Resource group where the Azure Cosmos account and virtual network resources are located
$ResourceGroupName = "myResourceGroup"
# Name of the Azure Cosmos account
$CosmosDbAccountName = "mycosmosaccount"

# API type of the Azure Cosmos account: Sql, MongoDB, Cassandra, Gremlin, or Table
$CosmosDbApiType = "Sql"
# Name of the existing virtual network
$VNetName = "myVnet"
# Name of the target subnet in the virtual network
$SubnetName = "mySubnet"
# Name of the private endpoint to create
$PrivateEndpointName = "MyPrivateEndpoint"
# Location where the private endpoint can be created. The private endpoint should be created in the same location where your subnet or the virtual network exists
$Location = "westcentralus"

$cosmosDbResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.DocumentDB/databaseAccounts/$($CosmosDbAccountName)"

$privateEndpointConnection = New-AzPrivateLinkServiceConnection -Name "myConnectionPS" -PrivateLinkServiceId $cosmosDbResourceId -GroupId $CosmosDbApiType
 
$virtualNetwork = Get-AzVirtualNetwork -ResourceGroupName  $ResourceGroupName -Name $VNetName  
 
$subnet = $virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq $SubnetName}  
 
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $ResourceGroupName -Name $PrivateEndpointName -Location "westcentralus" -Subnet  $subnet -PrivateLinkServiceConnection $privateEndpointConnection
```

### <a name="integrate-the-private-endpoint-with-a-private-dns-zone"></a>Intégrer le point de terminaison privé à une zone DNS privée

Après avoir créé le point de terminaison privé, vous pouvez l’intégrer à une zone DNS privée à l’aide du script PowerShell suivant :

```azurepowershell-interactive
Import-Module Az.PrivateDns
$zoneName = "privatelink.documents.azure.com"
$zone = New-AzPrivateDnsZone -ResourceGroupName $ResourceGroupName `
  -Name $zoneName

$link  = New-AzPrivateDnsVirtualNetworkLink -ResourceGroupName $ResourceGroupName `
  -ZoneName $zoneName `
  -Name "myzonelink" `
  -VirtualNetworkId $virtualNetwork.Id  
 
$pe = Get-AzPrivateEndpoint -Name $PrivateEndpointName `
  -ResourceGroupName $ResourceGroupName

$networkInterface = Get-AzResource -ResourceId $pe.NetworkInterfaces[0].Id `
  -ApiVersion "2019-04-01"
 
foreach ($ipconfig in $networkInterface.properties.ipConfigurations) { 
foreach ($fqdn in $ipconfig.properties.privateLinkConnectionProperties.fqdns) { 
Write-Host "$($ipconfig.properties.privateIPAddress) $($fqdn)"  
$recordName = $fqdn.split('.',2)[0] 
$dnsZone = $fqdn.split('.',2)[1] 
New-AzPrivateDnsRecordSet -Name $recordName `
  -RecordType A -ZoneName $zoneName  `
  -ResourceGroupName $ResourceGroupName -Ttl 600 `
  -PrivateDnsRecords (New-AzPrivateDnsRecordConfig `
  -IPv4Address $ipconfig.properties.privateIPAddress)  
}
}
```

### <a name="fetch-the-private-ip-addresses"></a>Récupérer les adresses IP privées

Une fois le point de terminaison privé configuré, vous pouvez interroger les adresses IP et le mappage FQDNS à l’aide du script PowerShell suivant :

```azurepowershell-interactive
$pe = Get-AzPrivateEndpoint -Name MyPrivateEndpoint -ResourceGroupName myResourceGroup
$networkInterface = Get-AzNetworkInterface -ResourceId $pe.NetworkInterfaces[0].Id
foreach ($IPConfiguration in $networkInterface.IpConfigurations)
{
    Write-Host $IPConfiguration.PrivateIpAddress ":" $IPConfiguration.PrivateLinkConnectionProperties.Fqdns
}
```

## <a name="create-a-private-endpoint-by-using-azure-cli"></a>Créer un point de terminaison privé à l’aide d’Azure CLI

Exécutez le script Azure CLI suivant afin de créer un point de terminaison privé nommé « myPrivateEndpoint » pour un compte Azure Cosmos existant. Remplacez les valeurs des variables par les informations de votre environnement.

```azurecli-interactive
# Resource group where the Azure Cosmos account and virtual network resources are located
ResourceGroupName="myResourceGroup"

# Subscription ID where the Azure Cosmos account and virtual network resources are located
SubscriptionId="<your Azure subscription ID>"

# Name of the existing Azure Cosmos account
CosmosDbAccountName="mycosmosaccount"

# API type of your Azure Cosmos account: Sql, MongoDB, Cassandra, Gremlin, or Table
CosmosDbApiType="Sql"

# Name of the virtual network to create
VNetName="myVnet"

# Name of the subnet to create
SubnetName="mySubnet"

# Name of the private endpoint to create
PrivateEndpointName="myPrivateEndpoint"

# Name of the private endpoint connection to create
PrivateConnectionName="myConnection"

az network vnet create \
 --name $VNetName \
 --resource-group $ResourceGroupName \
 --subnet-name $SubnetName

az network vnet subnet update \
 --name $SubnetName \
 --resource-group $ResourceGroupName \
 --vnet-name $VNetName \
 --disable-private-endpoint-network-policies true

az network private-endpoint create \
    --name $PrivateEndpointName \
    --resource-group $ResourceGroupName \
    --vnet-name $VNetName  \
    --subnet $SubnetName \
    --private-connection-resource-id "/subscriptions/$SubscriptionId/resourceGroups/$ResourceGroupName/providers/Microsoft.DocumentDB/databaseAccounts/$CosmosDbAccountName" \
    --group-ids $CosmosDbApiType \
    --connection-name $PrivateConnectionName
```

### <a name="integrate-the-private-endpoint-with-a-private-dns-zone"></a>Intégrer le point de terminaison privé à une zone DNS privée

Après avoir créé le point de terminaison privé, vous pouvez l’intégrer à une zone DNS privée à l’aide du script Azure CLI suivant :

```azurecli-interactive
zoneName="privatelink.documents.azure.com"

az network private-dns zone create --resource-group $ResourceGroupName \
   --name  $zoneName

az network private-dns link vnet create --resource-group $ResourceGroupName \
   --zone-name  $zoneName\
   --name myzonelink \
   --virtual-network $VNetName \
   --registration-enabled false 

#Query for the network interface ID  
networkInterfaceId=$(az network private-endpoint show --name $PrivateEndpointName --resource-group $ResourceGroupName --query 'networkInterfaces[0].id' -o tsv)
 
# Copy the content for privateIPAddress and FQDN matching the Azure Cosmos account 
az resource show --ids $networkInterfaceId --api-version 2019-04-01 -o json 
 
#Create DNS records 
az network private-dns record-set a create --name recordSet1 --zone-name privatelink.documents.azure.com --resource-group $ResourceGroupName
az network private-dns record-set a add-record --record-set-name recordSet2 --zone-name privatelink.documents.azure.com --resource-group $ResourceGroupName -a <Private IP Address>
```

## <a name="create-a-private-endpoint-by-using-a-resource-manager-template"></a>Créer un point de terminaison privé à l’aide d’un modèle Resource Manager

Vous pouvez configurer une Liaison privée (Private Link) en créant un point de terminaison privé dans un sous-réseau de réseau virtuel. Pour ce faire, utilisez un modèle Azure Resource Manager.

Utilisez le code suivant pour créer un modèle Resource Manager nommé « PrivateEndpoint_template.json ». Ce modèle crée un point de terminaison privé pour un compte d’API SQL Azure Cosmos existant dans un réseau virtuel existant.

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "location": {
          "type": "string",
          "defaultValue": "[resourceGroup().location]",
          "metadata": {
            "description": "Location for all resources."
          }
        },
        "privateEndpointName": {
            "type": "string"
        },
        "resourceId": {
            "type": "string"
        },
        "groupId": {
            "type": "string"
        },
        "subnetId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('privateEndpointName')]",
            "type": "Microsoft.Network/privateEndpoints",
            "apiVersion": "2019-04-01",
            "location": "[parameters('location')]",
            "properties": {
                "subnet": {
                    "id": "[parameters('subnetId')]"
                },
                "privateLinkServiceConnections": [
                    {
                        "name": "MyConnection",
                        "properties": {
                            "privateLinkServiceId": "[parameters('resourceId')]",
                            "groupIds": [ "[parameters('groupId')]" ],
                            "requestMessage": ""
                        }
                    }
                ]
            }
        }
    ],
    "outputs": {
        "privateEndpointNetworkInterface": {
          "type": "string",
          "value": "[reference(concat('Microsoft.Network/privateEndpoints/', parameters('privateEndpointName'))).networkInterfaces[0].id]"
        }
    }
}
```

**Définir le fichier de paramètres pour le modèle**

Créez un fichier de paramètres pour le modèle, puis nommez-le « PrivateEndpoint_parameters.json ». Ajoutez le code suivant au fichier de paramètres :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "privateEndpointName": {
            "value": ""
        },
        "resourceId": {
            "value": ""
        },
        "groupId": {
            "value": ""
        },
        "subnetId": {
            "value": ""
        }
    }
}
```

**Déployer le modèle à l’aide d’un script PowerShell**

Créez un script PowerShell en utilisant le code suivant. Avant d’exécuter le script, remplacez l’ID d’abonnement, le nom du groupe de ressources et d’autres valeurs de variables par les informations de votre environnement.

```azurepowershell-interactive
### This script creates a private endpoint for an existing Azure Cosmos account in an existing virtual network

## Step 1: Fill in these details. Replace the variable values with the details for your environment.
$SubscriptionId = "<your Azure subscription ID>"
# Resource group where the Azure Cosmos account and virtual network resources are located
$ResourceGroupName = "myResourceGroup"
# Name of the Azure Cosmos account
$CosmosDbAccountName = "mycosmosaccount"
# API type of the Azure Cosmos account. It can be one of the following: "Sql", "MongoDB", "Cassandra", "Gremlin", "Table"
$CosmosDbApiType = "Sql"
# Name of the existing virtual network
$VNetName = "myVnet"
# Name of the target subnet in the virtual network
$SubnetName = "mySubnet"
# Name of the private endpoint to create
$PrivateEndpointName = "myPrivateEndpoint"

$cosmosDbResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.DocumentDB/databaseAccounts/$($CosmosDbAccountName)"
$VNetResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.Network/virtualNetworks/$($VNetName)"
$SubnetResourceId = "$($VNetResourceId)/subnets/$($SubnetName)"
$PrivateEndpointTemplateFilePath = "PrivateEndpoint_template.json"
$PrivateEndpointParametersFilePath = "PrivateEndpoint_parameters.json"

## Step 2: Sign in to your Azure account and select the target subscription.
Login-AzAccount
Select-AzSubscription -SubscriptionId $subscriptionId

## Step 3: Make sure private endpoint network policies are disabled in the subnet.
$VirtualNetwork= Get-AzVirtualNetwork -Name "$VNetName" -ResourceGroupName "$ResourceGroupName"
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq "$SubnetName"} ).PrivateEndpointNetworkPolicies = "Disabled"
$virtualNetwork | Set-AzVirtualNetwork

## Step 4: Create the private endpoint.
Write-Output "Deploying private endpoint on $($resourceGroupName)"
$deploymentOutput = New-AzResourceGroupDeployment -Name "PrivateCosmosDbEndpointDeployment" `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $PrivateEndpointTemplateFilePath `
    -TemplateParameterFile $PrivateEndpointParametersFilePath `
    -SubnetId $SubnetResourceId `
    -ResourceId $CosmosDbResourceId `
    -GroupId $CosmosDbApiType `
    -PrivateEndpointName $PrivateEndpointName

$deploymentOutput
```

Dans le script PowerShell, la variable `GroupId` ne peut contenir qu’une seule valeur. Cette valeur est le type d’API du compte. Les valeurs autorisées sont : `Sql`, `MongoDB`, `Cassandra`, `Gremlin` et `Table`. Certains types de comptes Azure Cosmos sont accessibles par le biais de plusieurs API. Par exemple :

* Un compte d’API Gremlin est accessible à partir de comptes d’API Gremlin et SQL.
* Un compte d’API Table est accessible à partir de comptes d’API Table et SQL.

Pour ces comptes, vous devez créer un point de terminaison privé pour chaque type d’API. Le type d’API correspondant est spécifié dans le tableau `GroupId`.

Une fois que le modèle a bien été déployé, vous pouvez voir une sortie similaire à ce que montre l’image suivante. La valeur `provisioningState` est `Succeeded` si les points de terminaison privés sont configurés correctement.

:::image type="content" source="./media/how-to-configure-private-endpoints/resource-manager-template-deployment-output.png" alt-text="Sortie du déploiement pour le modèle Resource Manager":::

Une fois le modèle déployé, les adresses IP privées sont réservées dans le sous-réseau. La règle de pare-feu du compte Azure Cosmos est configurée pour accepter uniquement les connexions à partir du point de terminaison privé.

### <a name="integrate-the-private-endpoint-with-a-private-dns-zone"></a>Intégrer le point de terminaison privé à une zone DNS privée

Utilisez le code suivant pour créer un modèle Resource Manager nommé « PrivateZone_template.json ». Ce modèle crée une zone DNS privée pour un compte d’API SQL Azure Cosmos existant dans un réseau virtuel existant.

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "privateZoneName": {
            "type": "string"
        },
        "VNetId": {
            "type": "string"
        }        
    },
    "resources": [
        {
            "name": "[parameters('privateZoneName')]",
            "type": "Microsoft.Network/privateDnsZones",
            "apiVersion": "2018-09-01",
            "location": "global",
            "properties": {                
            }
        },
        {
            "type": "Microsoft.Network/privateDnsZones/virtualNetworkLinks",
            "apiVersion": "2018-09-01",
            "name": "[concat(parameters('privateZoneName'), '/myvnetlink')]",
            "location": "global",
            "dependsOn": [
                "[resourceId('Microsoft.Network/privateDnsZones', parameters('privateZoneName'))]"
            ],
            "properties": {
                "registrationEnabled": false,
                "virtualNetwork": {
                    "id": "[parameters('VNetId')]"
                }
            }
        }        
    ]
}
```

Utilisez le code suivant pour créer un modèle Resource Manager nommé « PrivateZoneRecords_template.json ».

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "DNSRecordName": {
            "type": "string"
        },
        "IPAddress": {
            "type":"string"
        }        
    },
    "resources": [
         {
            "type": "Microsoft.Network/privateDnsZones/A",
            "apiVersion": "2018-09-01",
            "name": "[parameters('DNSRecordName')]",
            "properties": {
                "ttl": 300,
                "aRecords": [
                    {
                        "ipv4Address": "[parameters('IPAddress')]"
                    }
                ]
            }
        }    
    ]
}
```

**Définir le fichier de paramètres pour le modèle**

Créez les deux fichiers de paramètres suivants pour le modèle. Créez le modèle « PrivateZone_parameters.json ». par le code suivant :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "privateZoneName": {
            "value": ""
        },
        "VNetId": {
            "value": ""
        }
    }
}
```

Créez le modèle « PrivateZoneRecords_parameters.json ». par le code suivant :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "DNSRecordName": {
            "value": ""
        },
        "IPAddress": {
            "type":"object"
        }
    }
}
```

**Déployer le modèle à l’aide d’un script PowerShell**

Créez un script PowerShell en utilisant le code suivant. Avant d’exécuter le script, remplacez l’ID d’abonnement, le nom du groupe de ressources et d’autres valeurs de variables par les informations de votre environnement.

```azurepowershell-interactive
### This script:
### - creates a private zone
### - creates a private endpoint for an existing Cosmos DB account in an existing VNet
### - maps the private endpoint to the private zone

## Step 1: Fill in these details. Replace the variable values with the details for your environment.
$SubscriptionId = "<your Azure subscription ID>"
# Resource group where the Azure Cosmos account and virtual network resources are located
$ResourceGroupName = "myResourceGroup"
# Name of the Azure Cosmos account
$CosmosDbAccountName = "mycosmosaccount"
# API type of the Azure Cosmos account. It can be one of the following: "Sql", "MongoDB", "Cassandra", "Gremlin", "Table"
$CosmosDbApiType = "Sql"
# Name of the existing virtual network
$VNetName = "myVnet"
# Name of the target subnet in the virtual network
$SubnetName = "mySubnet"
# Name of the private zone to create
$PrivateZoneName = "myPrivateZone.documents.azure.com"
# Name of the private endpoint to create
$PrivateEndpointName = "myPrivateEndpoint"

$cosmosDbResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.DocumentDB/databaseAccounts/$($CosmosDbAccountName)"
$VNetResourceId = "/subscriptions/$($SubscriptionId)/resourceGroups/$($ResourceGroupName)/providers/Microsoft.Network/virtualNetworks/$($VNetName)"
$SubnetResourceId = "$($VNetResourceId)/subnets/$($SubnetName)"
$PrivateZoneTemplateFilePath = "PrivateZone_template.json"
$PrivateZoneParametersFilePath = "PrivateZone_parameters.json"
$PrivateZoneRecordsTemplateFilePath = "PrivateZoneRecords_template.json"
$PrivateZoneRecordsParametersFilePath = "PrivateZoneRecords_parameters.json"
$PrivateEndpointTemplateFilePath = "PrivateEndpoint_template.json"
$PrivateEndpointParametersFilePath = "PrivateEndpoint_parameters.json"

## Step 2: Login your Azure account and select the target subscription
Login-AzAccount 
Select-AzSubscription -SubscriptionId $subscriptionId

## Step 3: Make sure private endpoint network policies are disabled in the subnet
$VirtualNetwork= Get-AzVirtualNetwork -Name "$VNetName" -ResourceGroupName "$ResourceGroupName"
($virtualNetwork | Select -ExpandProperty subnets | Where-Object  {$_.Name -eq "$SubnetName"} ).PrivateEndpointNetworkPolicies = "Disabled"
$virtualNetwork | Set-AzVirtualNetwork

## Step 4: Create the private zone
New-AzResourceGroupDeployment -Name "PrivateZoneDeployment" `
    -ResourceGroupName $ResourceGroupName `
    -TemplateFile $PrivateZoneTemplateFilePath `
    -TemplateParameterFile $PrivateZoneParametersFilePath `
    -PrivateZoneName $PrivateZoneName `
    -VNetId $VNetResourceId

## Step 5: Create the private endpoint
Write-Output "Deploying private endpoint on $($resourceGroupName)"
$deploymentOutput = New-AzResourceGroupDeployment -Name "PrivateCosmosDbEndpointDeployment" `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $PrivateEndpointTemplateFilePath `
    -TemplateParameterFile $PrivateEndpointParametersFilePath `
    -SubnetId $SubnetResourceId `
    -ResourceId $CosmosDbResourceId `
    -GroupId $CosmosDbApiType `
    -PrivateEndpointName $PrivateEndpointName
$deploymentOutput

## Step 6: Map the private endpoint to the private zone
$networkInterface = Get-AzResource -ResourceId $deploymentOutput.Outputs.privateEndpointNetworkInterface.Value -ApiVersion "2019-04-01"
foreach ($ipconfig in $networkInterface.properties.ipConfigurations) {
    foreach ($fqdn in $ipconfig.properties.privateLinkConnectionProperties.fqdns) {
        $recordName = $fqdn.split('.',2)[0]
        $dnsZone = $fqdn.split('.',2)[1]
        Write-Output "Deploying PrivateEndpoint DNS Record $($PrivateZoneName)/$($recordName) Template on $($resourceGroupName)"
        New-AzResourceGroupDeployment -Name "PrivateEndpointDNSDeployment" `
            -ResourceGroupName $ResourceGroupName `
            -TemplateFile $PrivateZoneRecordsTemplateFilePath `
            -TemplateParameterFile $PrivateZoneRecordsParametersFilePath `
            -DNSRecordName "$($PrivateZoneName)/$($RecordName)" `
            -IPAddress $ipconfig.properties.privateIPAddress
    }
}
```

## <a name="configure-custom-dns"></a>Configurer un système DNS personnalisé

Vous devez utiliser une zone DNS privée dans le sous-réseau où vous avez créé le point de terminaison privé. Configurez les points de terminaison de façon à ce que chaque adresse IP privée soit mappée à une entrée DNS. (Consultez la propriété `fqdns` dans la réponse indiquée plus haut.)

Lorsque vous créez le point de terminaison privé, vous pouvez intégrer celui-ci à une zone DNS privée dans Azure. Si vous choisissez plutôt d’utiliser une zone DNS personnalisée, vous devez la configurer afin d’ajouter des enregistrements DNS pour toutes les adresses IP privées réservées au point de terminaison privé.

## <a name="private-link-combined-with-firewall-rules"></a>Private Link combiné à des règles de pare-feu

Les situations et résultats suivants sont possibles lorsque vous utilisez Private Link en association avec des règles de pare-feu :

* Si vous ne configurez pas de règles de pare-feu, par défaut, tout le trafic peut accéder à un compte Azure Cosmos.

* Si vous configurez un trafic public ou un point de terminaison de service et que vous créez des points de terminaison privés, différents types de trafic entrant sont alors autorisés par le type de règle de pare-feu correspondant. Si un point de terminaison privé est configuré dans un sous-réseau où le point de terminaison de service est également configuré :
  * le trafic vers le compte de base de données mappé par le point de terminaison privé est routé via un point de terminaison privé,
  * le trafic vers d’autres comptes de bases de données à partir du sous-réseau est routé via le point de terminaison de service.

* Si vous ne configurez aucun trafic public ni point de terminaison de service et que vous créez des points de terminaison privés, le compte Azure Cosmos est alors uniquement accessible via les points de terminaison privés. Si vous ne configurez aucun trafic public ni point de terminaison de service, après le rejet ou la suppression de tous les points de terminaison privés approuvés, le compte est ouvert sur l’ensemble du réseau à moins que PublicNetworkAccess ne soit défini sur Désactivé (consultez la section ci-dessous).

## <a name="blocking-public-network-access-during-account-creation"></a>Blocage de l’accès au réseau public pendant la création du compte

Comme décrit dans la section précédente, et à moins que des règles de pare-feu spécifiques n’aient été définies, l’ajout d’un point de terminaison privé rend votre compte Azure Cosmos accessible via des points de terminaison privés uniquement. Cela signifie que le compte Azure Cosmos peut être atteint à partir du trafic public, après sa création et avant l’ajout d’un point de terminaison privé. Pour vous assurer que l’accès au réseau public est désactivé avant même la création de points de terminaison privés, vous pouvez définir l’indicateur `publicNetworkAccess` sur `Disabled` pendant la création du compte. Notez que cet indicateur est prioritaire sur toute règle de réseau virtuel ou d’adresse IP ; tout le trafic public et de réseau virtuel est bloqué lorsque l’indicateur est défini sur `Disabled`, même si l’adresse IP source ou le réseau virtuel est autorisé dans la configuration du pare-feu.

Pour voir un exemple d’utilisation de cet indicateur, consultez [ce modèle Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-cosmosdb-private-endpoint/).

## <a name="adding-private-endpoints-to-an-existing-cosmos-account-with-no-downtime"></a>Ajout de points de terminaison privés à un compte Cosmos existant sans temps d’arrêt

Par défaut, l’ajout d’un point de terminaison privé à un compte existant entraîne un court temps d’arrêt d’environ cinq minutes. Suivez les instructions ci-dessous pour éviter ce temps d’arrêt :

1. Ajoutez des règles de réseau virtuel ou d’adresse IP à votre configuration de pare-feu pour autoriser explicitement vos connexions client.
1. Attendez dix minutes pour vous assurer que la mise à jour de la configuration est appliquée.
1. Configurez votre nouveau point de terminaison privé.
1. Supprimez les règles de pare-feu définies à l’étape 1.

## <a name="port-range-when-using-direct-mode"></a>Plage de ports lors de l’utilisation du mode direct

Lorsque vous utilisez Private Link avec un compte Azure Cosmos via une connexion en mode direct, vous devez vous assurer que la plage de ports TCP complète (0 à 65535) est ouverte.

## <a name="update-a-private-endpoint-when-you-add-or-remove-a-region"></a>Mettre à jour un point de terminaison privé lors de l’ajout ou de la suppression d’une région

À moins que vous n’utilisiez un groupe de zones DNS privées, l’ajout ou la suppression de régions dans un compte Azure Cosmos vous oblige à ajouter ou supprimer des entrées DNS pour ce compte. Après l’ajout ou la suppression des régions, vous pouvez mettre à jour la zone DNS privée du sous-réseau pour refléter les entrées DNS ajoutées ou supprimées et leurs adresses IP privées correspondantes.

Par exemple, imaginez que vous déployez un compte Azure Cosmos dans trois régions : « USA Ouest », « USA Centre » et « Europe Ouest ». Lorsque vous créez un point de terminaison privé pour votre compte, quatre adresses IP privées sont réservées dans le sous-réseau. Il existe une adresse IP pour chacune des trois régions et une adresse IP pour le point de terminaison global/indépendant de la région.

Plus tard, vous pouvez ajouter une nouvelle région (par exemple « USA Est ») au compte Azure Cosmos. Après avoir ajouté la nouvelle région, vous devez ajouter un enregistrement DNS correspondant à votre zone DNS privée ou à votre DNS personnalisé.

Vous pouvez utiliser les mêmes étapes lorsque vous supprimez une région. Après avoir supprimé la région, vous devez supprimer l’enregistrement DNS correspondant de votre zone DNS privée ou de votre DNS personnalisé.

## <a name="current-limitations"></a>Limites actuelles

Les limitations suivantes s’appliquent lorsque vous utilisez Private Link avec un compte Azure Cosmos :

* Vous ne pouvez pas avoir plus de 200 points de terminaison privés sur un seul compte Azure Cosmos.

* Lorsque vous utilisez Private Links avec un compte Azure Cosmos à travers une connexion en mode direct, vous ne pouvez utiliser que le protocole TCP. Le protocole HTTP n’est pas pris en charge actuellement.

* Lorsque vous utilisez l’API Azure Cosmos DB pour les comptes MongoDB, un point de terminaison privé est pris en charge pour les comptes sur le serveur version 3.6 uniquement (c’est-à-dire les comptes utilisant le point de terminaison au format `*.mongo.cosmos.azure.com`). Private Link n’est pas pris en charge pour les comptes sur le serveur version 3.2 (c’est-à-dire les comptes utilisant le point de terminaison au format `*.documents.azure.com`). Pour utiliser Private Link, vous devez migrer les anciens comptes vers la nouvelle version.

* Lorsque vous utilisez une API d’Azure Cosmos DB pour le compte MongoDB qui possède un lien privé, il se peut que des outils ou bibliothèques ne fonctionnent parce qu’il écartent automatiquement le paramètre `appName` de la chaîne de connexion. Ce paramètre est obligatoire pour se connecter au compte sur un point de terminaison privé. Certains outils, tel Visual Studio Code, ne suppriment pas ce paramètre de la chaîne de connexion. Ils sont donc compatibles.

* Un administrateur réseau doit disposer au moins de l’autorisation `Microsoft.DocumentDB/databaseAccounts/PrivateEndpointConnectionsApproval/action` dans l’étendue du compte Azure Cosmos pour créer des points de terminaison privés approuvés automatiquement.

### <a name="limitations-to-private-dns-zone-integration"></a>Limitations de l’intégration à une zone DNS privée

À moins que vous n’utilisiez un groupe de zones DNS privées, les enregistrements DNS dans la zone DNS privée ne sont pas supprimés automatiquement lorsque vous supprimez un point de terminaison privé ou une région du compte Azure Cosmos. Vous devez supprimer manuellement les enregistrements DNS avant d’effectuer les actions suivantes :

* Ajout d’un nouveau point de terminaison privé lié à cette zone DNS privée.
* Ajout d’une nouvelle région à un compte de base de données qui dispose de points de terminaison privés liés à cette zone DNS privée.

Si vous ne nettoyez pas les enregistrements DNS, des problèmes de plan de données inattendus peuvent se produire. Ces problèmes incluent l’indisponibilité des données dans les régions ajoutées après la suppression du point de terminaison privé ou de la région.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les fonctionnalités de sécurité d’Azure Cosmos DB, consultez l’article suivant :

* Pour configurer un pare-feu pour Azure Cosmos DB, consultez [Prise en charge du pare-feu](how-to-configure-firewall.md).

* Pour découvrir comment configurer un point de terminaison de service de réseau virtuel pour votre compte Azure Cosmos, consultez [Configurer l’accès à partir de réseaux virtuels](how-to-configure-vnet-service-endpoint.md).

* Pour en savoir plus sur Liaison privée (Private Link), consultez la documentation [Azure Private Link](../private-link/private-link-overview.md).
