---
title: Configurer l’accès à un compte Azure Cosmos à partir d’un réseau virtuel
description: Ce document décrit les étapes nécessaires pour configurer un point de terminaison de service de réseau virtuel pour Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.topic: how-to
ms.date: 10/13/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1d63d21f4c49e3c7aef035208477ac9fc79f2e51
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94637181"
---
# <a name="configure-access-to-azure-cosmos-db-from-virtual-networks-vnet"></a>Configurer l’accès à Azure Cosmos DB à partir de réseaux virtuels (VNet)
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Vous pouvez configurer le compte Azure Cosmos pour n'autoriser l'accès qu'à partir d'un sous-réseau spécifique de réseau virtuel (VNet). En activant l’option [Point de terminaison de service](../virtual-network/virtual-network-service-endpoints-overview.md) pour accéder à Azure Cosmos DB sur le sous-réseau au sein d’un réseau virtuel, le trafic à partir de ce sous-réseau est envoyé à Azure Cosmos DB avec l’identité du sous-réseau et du réseau virtuel. Une fois le point de terminaison de service Azure Cosmos DB activé, vous pouvez limiter l’accès au sous-réseau en l’ajoutant à votre compte Azure Cosmos.

Par défaut, un compte Azure Cosmos est accessible depuis n’importe quelle source tant que la demande est accompagnée d’un jeton d’autorisation valide. Lorsque vous ajoutez un ou plusieurs sous-réseaux au sein de réseaux virtuels, seules les requêtes provenant de ces sous-réseaux obtiendront une réponse valide. Les requêtes provenant de toute autre source recevront une réponse 403 (Interdit). 

Vous pouvez configurer les comptes Azure Cosmos DB pour autoriser l’accès uniquement à partir d’un sous-réseau spécifique de réseau virtuel Azure. Pour limiter l’accès à un compte Azure Cosmos DB avec des connexions issues d’un sous-réseau dans un réseau virtuel :

1. Autorisez le sous-réseau à envoyer l’identité du sous-réseau et du réseau virtuel à Azure Cosmos DB. Pour cela, activez un point de terminaison de service pour Azure Cosmos DB sur le sous-réseau concerné.

1. Ajoutez au compte Azure Cosmos DB une règle spécifiant le sous-réseau comme source permettant d’accéder au compte.

> [!NOTE]
> Une fois qu’un point de terminaison de service pour votre compte Azure Cosmos DB est activé sur un sous-réseau, la source du trafic qui atteint Azure Cosmos DB bascule d’une adresse IP publique vers un réseau virtuel et un sous-réseau. Le basculement du trafic s’applique à tous les comptes Azure Cosmos DB accessibles à partir de ce sous-réseau. Si vos comptes Azure Cosmos DB utilisent un pare-feu basé sur l’adresse IP pour autoriser ce sous-réseau, les requêtes issues du sous-réseau avec service ne correspondent plus aux règles de pare-feu IP et sont rejetées.
>
> Pour plus d’informations, voir les étapes décrites dans la section [Migrer à partir d’une règle de pare-feu IP à une liste de contrôle d’accès de réseau virtuel](#migrate-from-firewall-to-vnet) de cet article.

Les sections suivantes expliquent comment configurer un point de terminaison de service de réseau virtuel pour un compte Azure Cosmos DB.

## <a name="configure-a-service-endpoint-by-using-the-azure-portal"></a><a id="configure-using-portal"></a>Configurer un point de terminaison de service avec le portail Azure

### <a name="configure-a-service-endpoint-for-an-existing-azure-virtual-network-and-subnet"></a>Configurer un point de terminaison de service pour un réseau virtuel et un sous-réseau Azure existants

1. À partir du panneau **Toutes les ressources**, recherchez le compte Azure Cosmos DB que vous souhaitez sécuriser.

1. Sélectionnez **Pare-feu et réseaux virtuels** dans le menu Paramètres et choisissez d’autoriser l’accès à partir de **Réseaux sélectionnés**.

1. Pour accorder l’accès au sous-réseau d’un réseau virtuel existant, sélectionnez **Ajouter un réseau virtuel Azure existant** sous **Réseaux virtuels**.

1. Sélectionnez **l’abonnement** à partir duquel vous souhaitez ajouter un réseau virtuel Azure. Sélectionnez les **Réseaux virtuels** et **Sous-réseaux** Azure auxquels vous voulez fournir l’accès à votre compte Azure Cosmos DB. Ensuite, sélectionnez **Activer** pour activer les réseaux sélectionnés avec les points de terminaison de service pour « Microsoft.AzureCosmosDB ». Une fois terminé, sélectionnez **Ajouter**.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/choose-subnet-and-vnet.png" alt-text="Sélectionner un réseau virtuel et un sous-réseau":::

1. Lorsque le compte Azure Cosmos DB se voit autoriser l’accès à partir d’un réseau virtuel, il autorise uniquement le trafic provenant de ce sous-réseau. Le réseau virtuel et le sous-réseau que vous avez ajoutés devraient se présenter comme dans la capture d’écran suivante :

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/vnet-and-subnet-configured-successfully.png" alt-text="Réseau virtuel et sous-réseau configurés correctement":::

> [!NOTE]
> Les autorisations d’abonnement suivantes sont nécessaires pour activer les points de terminaison de service de réseau virtuel :
>   * Abonnement avec un réseau virtuel : Contributeur de réseau
>   * Abonnement avec un compte Azure Cosmos DB : Contributeur de compte DocumentDB
>   * Si votre réseau virtuel et votre compte Azure Cosmos DB se trouvent dans des abonnements différents, assurez-vous que l’abonnement contenant le réseau virtuel possède également le fournisseur de ressources `Microsoft.DocumentDB` inscrit. Pour inscrire un fournisseur de ressources, consultez l’article [Fournisseurs et types de ressources Azure](../azure-resource-manager/management/resource-providers-and-types.md).

Voici les instructions pour inscrire un abonnement avec le fournisseur de ressources.

### <a name="configure-a-service-endpoint-for-a-new-azure-virtual-network-and-subnet"></a>Configurer un point de terminaison de service pour un nouveau réseau virtuel et un nouveau sous-réseau Azure

1. À partir du panneau **Toutes les ressources**, recherchez le compte Azure Cosmos DB que vous souhaitez sécuriser.  

1. Sélectionnez **Pare-feu et réseaux virtuels Azure** dans le menu Paramètres et choisissez d’autoriser l’accès à partir de **Réseaux sélectionnés**.  

1. Pour accorder l’accès à un nouveau réseau virtuel Azure, sélectionnez **Ajouter un nouveau réseau virtuel** sous **Réseaux virtuels**.  

1. Fournissez les détails nécessaires pour créer un réseau virtuel, puis sélectionnez **Créer**. Le sous-réseau sera créé avec un point de terminaison de service pour « Microsoft.AzureCosmosDB » activé.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/choose-subnet-and-vnet-new-vnet.png" alt-text="Sélectionner un réseau virtuel et un sous-réseau pour un nouveau réseau virtuel":::

Si votre compte Azure Cosmos DB est utilisé par d’autres services Azure, comme la Recherche cognitive Azure, ou s’il est accessible à partir de Stream Analytics ou de Power BI, sélectionnez **Accepter les connexions provenant des centres de données Azure globaux** pour en autoriser l’accès.

Pour être sûr d’avoir accès aux métriques Azure Cosmos DB à partir du portail, vous devez activer l’option **Autoriser l’accès à partir du portail Azure**. Pour en savoir plus sur ces options, consultez l’article [Configurer un pare-feu IP](how-to-configure-firewall.md). Après avoir activé l’accès, sélectionnez **Enregistrer** pour enregistrer les paramètres.

## <a name="remove-a-virtual-network-or-subnet"></a><a id="remove-vnet-or-subnet"></a>Supprimer un réseau virtuel ou un sous-réseau

1. À partir du panneau **Toutes les ressources**, recherchez le compte Azure Cosmos DB pour lequel vous avez affecté des points de terminaison de service.  

1. Sélectionnez **Pare-feu et réseaux virtuels** dans le menu Paramètres.  

1. Pour supprimer une règle de réseau virtuel ou de sous-réseau, sélectionnez **...** en regard du réseau virtuel ou du sous-réseau et sélectionnez **Supprimer**.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/remove-a-vnet.png" alt-text="Supprimer un réseau virtuel":::

1. Sélectionnez **Enregistrer** pour enregistrer vos modifications.

## <a name="configure-a-service-endpoint-by-using-azure-powershell"></a><a id="configure-using-powershell"></a>Configurer un point de terminaison de service avec Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Effectuez les étapes suivantes afin de configurer un point de terminaison de service pour un compte Azure Cosmos DB à l’aide d’Azure PowerShell :  

1. Installez [Azure PowerShell](/powershell/azure/install-Az-ps) et [connectez-vous](/powershell/azure/authenticate-azureps).  

1. Activez le point de terminaison de service pour un sous-réseau existant d’un réseau virtuel.  

   ```powershell
   $resourceGroupName = "<Resource group name>"
   $vnetName = "<Virtual network name>"
   $subnetName = "<Subnet name>"
   $subnetPrefix = "<Subnet address range>"
   $serviceEndpoint = "Microsoft.AzureCosmosDB"

   Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName | Set-AzVirtualNetworkSubnetConfig `
      -Name $subnetName `
      -AddressPrefix $subnetPrefix `
      -ServiceEndpoint $serviceEndpoint | Set-AzVirtualNetwork
   ```

   > [!NOTE]
   > Lorsque vous utilisez PowerShell ou l’interface Azure CLI, n’oubliez pas de spécifier la liste complète des filtres IP et les ACL du réseau virtuel dans Paramètres, pas seulement ceux qui doivent être ajoutés.

1. Obtenez les informations relatives au réseau virtuel.

   ```powershell
   $vnet = Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName

   $subnetId = $vnet.Id + "/subnets/" + $subnetName
   ```

1. Préparer une règle de réseau virtuel Cosmos DB

   ```powershell
   $vnetRule = New-AzCosmosDBVirtualNetworkRule `
      -Id $subnetId
   ```

1. Mettez à jour les propriétés du compte Azure Cosmos DB avec la nouvelle configuration du point de terminaison du réseau virtuel : 

   ```powershell
   $accountName = "<Cosmos DB account name>"

   Update-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName `
      -EnableVirtualNetwork $true `
      -VirtualNetworkRuleObject @($vnetRule)
   ```

1. Exécutez la commande suivante pour vérifier que votre compte Azure Cosmos DB est mis à jour avec le point de terminaison de service de réseau virtuel que vous avez configuré à l’étape précédente :

   ```powershell
   $account = Get-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName

   $account.IsVirtualNetworkFilterEnabled
   $account.VirtualNetworkRules
   ```

## <a name="configure-a-service-endpoint-by-using-the-azure-cli"></a><a id="configure-using-cli"></a>Configurer un point de terminaison de service avec Azure CLI

Des comptes Azure Cosmos peuvent être configurés pour des points de terminaison de service créés ou mis à jour ultérieurement si le sous-réseau est déjà configuré pour ceux-ci. Des points de terminaison de service peuvent également être activés sur le compte Cosmos où le sous-réseau n’est pas encore configuré pour ceux-ci. Ils commenceront à fonctionner lorsque le sous-réseau sera configuré. Cette flexibilité permet aux administrateurs qui n’ont pas accès aux ressources du compte Cosmos et du réseau virtuel de rendre leurs configurations indépendantes les unes des autres.

### <a name="create-a-new-cosmos-account-and-connect-it-to-a-back-end-subnet-for-a-new-virtual-network"></a>Créer un compte Cosmos et le connecter à un sous-réseau backend pour un nouveau réseau virtuel

Dans cet exemple, le réseau virtuel et le sous-réseau sont créés avec des points de terminaison de service activés pour les deux lors de leur création.

```azurecli-interactive
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet

# Resource group and Cosmos account variables
resourceGroupName='MyResourceGroup'
location='West US 2'
accountName='mycosmosaccount'

# Variables for a new Virtual Network with two subnets
vnetName='myVnet'
frontEnd='FrontEnd'
backEnd='BackEnd'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a virtual network with a front-end subnet
az network vnet create \
   -n $vnetName \
   -g $resourceGroupName \
   --address-prefix 10.0.0.0/16 \
   --subnet-name $frontEnd \
   --subnet-prefix 10.0.1.0/24

# Create a back-end subnet with service endpoints enabled for Cosmos DB
az network vnet subnet create \
   -n $backEnd \
   -g $resourceGroupName \
   --address-prefix 10.0.2.0/24 \
   --vnet-name $vnetName \
   --service-endpoints Microsoft.AzureCosmosDB

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values and service endpoints
az cosmosdb create \
   -n $accountName \
   -g $resourceGroupName \
   --enable-virtual-network true \
   --virtual-network-rules $svcEndpoint
```

### <a name="connect-and-configure-a-cosmos-account-to-a-back-end-subnet-independently"></a>Connecter et configurer un compte Cosmos sur un sous-réseau backend de manière indépendante

Cet exemple vise à montrer comment connecter un compte Azure Cosmos à un nouveau réseau virtuel dont le sous-réseau n’est pas encore configuré pour des points de terminaison de service. Pour ce faire, utilisez le paramètre `--ignore-missing-vnet-service-endpoint`. Cela permet de configurer le compte Cosmos sans erreur avant que la configuration du sous-réseau du réseau virtuel soit complète. Une fois le sous-réseau configuré, le compte Cosmos est alors accessible via le sous-réseau configuré.

```azurecli-interactive
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet
# that is not yet enabled for service endpoints.

# Resource group and Cosmos account variables
resourceGroupName='MyResourceGroup'
location='West US 2'
accountName='mycosmosaccount'

# Variables for a new Virtual Network with two subnets
vnetName='myVnet'
frontEnd='FrontEnd'
backEnd='BackEnd'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a virtual network with a front-end subnet
az network vnet create \
   -n $vnetName \
   -g $resourceGroupName \
   --address-prefix 10.0.0.0/16 \
   --subnet-name $frontEnd \
   --subnet-prefix 10.0.1.0/24

# Create a back-end subnet but without configuring service endpoints (--service-endpoints Microsoft.AzureCosmosDB)
az network vnet subnet create \
   -n $backEnd \
   -g $resourceGroupName \
   --address-prefix 10.0.2.0/24 \
   --vnet-name $vnetName

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values
az cosmosdb create -n $accountName -g $resourceGroupName

# Add the virtual network rule but ignore the missing service endpoint on the subnet
az cosmosdb network-rule add \
   -n $accountName \
   -g $resourceGroupName \
   --virtual-network $vnetName \
   --subnet svcEndpoint \
   --ignore-missing-vnet-service-endpoint true

read -p'Press any key to now configure the subnet for service endpoints'

az network vnet subnet update \
   -n $backEnd \
   -g $resourceGroupName \
   --vnet-name $vnetName \
   --service-endpoints Microsoft.AzureCosmosDB
```

## <a name="port-range-when-using-direct-mode"></a>Plage de ports lors de l’utilisation du mode direct

Lorsque vous utilisez des points de terminaison de service avec un compte Azure Cosmos via une connexion en mode direct, vous devez vous assurer que la plage de ports TCP comprise entre 10000 et 20000 est ouverte.

## <a name="migrating-from-an-ip-firewall-rule-to-a-virtual-network-acl"></a><a id="migrate-from-firewall-to-vnet"></a>Migrer à partir d’une règle de pare-feu IP à une liste de contrôle d’accès de réseau virtuel

Pour migrer un compte Azure Cosmos DB utilisant des règles de pare-feu IP afin qu’il utilise des points de terminaison de service de réseau virtuel, procédez comme suit.

Après la configuration d’un compte Azure Cosmos DB pour un point de terminaison de service pour un sous-réseau, les demandes de ce sous-réseau sont envoyées à Azure Cosmos DB avec les informations sur le réseau virtuel et le sous-réseau source au lieu d’une adresse IP publique source. Ces demandes ne correspondent plus à un filtre IP configuré sur le compte Azure Cosmos DB, ce qui explique pourquoi les étapes suivantes sont nécessaires afin d’éviter les temps d’arrêt.

Avant de continuer, activez le point de terminaison de service Azure Cosmos DB sur le réseau virtuel et le sous-réseau à l’aide de l’étape indiquée ci-dessus dans « Activer le point de terminaison de service pour un sous-réseau existant d’un réseau virtuel ».

1. Obtenez les informations relatives au réseau virtuel et au sous-réseau :

   ```powershell
   $resourceGroupName = "myResourceGroup"
   $accountName = "mycosmosaccount"
   $vnetName = "myVnet"
   $subnetName = "mySubnet"

   $vnet = Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName

   $subnetId = $vnet.Id + "/subnets/" + $subnetName
   ```

1. Préparez un nouvel objet de règle Réseau virtuel pour le compte Azure Cosmos DB :

   ```powershell
   $vnetRule = New-AzCosmosDBVirtualNetworkRule `
      -Id $subnetId
   ```

1. Mettez à jour le compte Azure Cosmos DB pour activer l’accès au point de terminaison de service à partir du sous-réseau :

   ```powershell
   Update-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName `
      -EnableVirtualNetwork $true `
      -VirtualNetworkRuleObject @($vnetRule)
   ```

1. Répétez les étapes précédentes pour tous les comptes Azure Cosmos DB faisant l’objet d’un accès à partir du sous-réseau.

1. Dans les règles de pare-feu du compte Azure Cosmos DB, supprimez la règle de pare-feu IP pour le sous-réseau.

## <a name="frequently-asked-questions"></a>Forum aux questions

Voici quelques questions fréquemment posées sur la configuration de l’accès à partir de réseaux virtuels :

### <a name="are-notebooks-and-mongocassandra-shell-currently-compatible-with-virtual-network-enabled-accounts"></a>Les notebooks et l’interpréteur de commandes Mongo/Cassandra sont-ils compatibles avec les comptes activés pour les réseaux virtuels ?

L’intégration de l’[interpréteur de commandes Mongo](https://devblogs.microsoft.com/cosmosdb/preview-native-mongo-shell/) et de l’[interpréteur de commandes Cassandra](https://devblogs.microsoft.com/cosmosdb/announcing-native-cassandra-shell-preview/) à l’Explorateur de données Cosmos DB n’est pas prise en charge avec l’accès au réseau virtuel, de même que le [service Jupyter Notebooks](./cosmosdb-jupyter-notebooks.md). Il s’agit d’un développement en cours.

### <a name="can-i-specify-both-virtual-network-service-endpoint-and-ip-access-control-policy-on-an-azure-cosmos-account"></a>Puis-je spécifier à la fois le point de terminaison de service de réseau virtuel et la stratégie de contrôle d’accès IP sur un compte Azure Cosmos ? 

Vous pouvez activer à la fois le point de terminaison de service de réseau virtuel et une stratégie de contrôle d’accès IP (également appelé pare-feu) sur votre compte Azure Cosmos. Ces deux fonctionnalités sont complémentaires et garantissent collectivement l’isolation et la sécurité de votre compte Azure Cosmos. L’utilisation du pare-feu IP permet de s’assurer que les adresses IP statiques peuvent accéder à votre compte. 

### <a name="how-do-i-limit-access-to-subnet-within-a-virtual-network"></a>Comment limiter l’accès à un sous-réseau au sein d’un réseau virtuel ? 

Il existe deux étapes requises pour limiter l’accès au compte Azure Cosmos à partir d’un sous-réseau. Tout d’abord, vous autorisez le trafic provenant du sous-réseau afin de communiquer à Azure Cosmos DB l’identité de son sous-réseau et de son réseau virtuel. Pour cela, vous activez le point de terminaison de service pour Azure Cosmos DB sur le sous-réseau. Ensuite, vous ajoutez une règle au compte Azure Cosmos en spécifiant ce sous-réseau comme source à partir de laquelle le compte est accessible.

### <a name="will-virtual-network-acls-and-ip-firewall-reject-requests-or-connections"></a>Les listes de contrôle d'accès (ACL) du réseau virtuel et le pare-feu IP rejetteront-ils les requêtes ou les connexions ? 

Quand un pare-feu IP ou des règles d’accès à un réseau virtuel sont ajoutés, seules les requêtes provenant de sources autorisées obtiennent des réponses valides. Les autres requêtes sont rejetées avec une erreur 403 (Interdit). Il est important de distinguer le pare-feu du compte Azure Cosmos d’un pare-feu au niveau de la connexion. La source peut toujours se connecter au service et les connexions elles-mêmes ne sont pas rejetées.

### <a name="my-requests-started-getting-blocked-when-i-enabled-service-endpoint-to-azure-cosmos-db-on-the-subnet-what-happened"></a>Mes requêtes ont été bloquées lorsque j’ai activé le point de terminaison de service dans Azure Cosmos DB sur le sous-réseau. Que s’est-il passé ?

Une fois que le point de terminaison de service pour Azure Cosmos DB est activé sur un sous-réseau, la source du trafic qui atteint le compte bascule de l’adresse IP publique vers le réseau virtuel et le sous-réseau. Si votre compte Azure Cosmos est uniquement protégé par un pare-feu basé sur IP, le trafic provenant du sous-réseau avec service ne respecte plus les règles du pare-feu IP et, par conséquent, il sera rejeté. Passez en revue les étapes pour migrer en toute transparence d’un pare-feu basé sur IP à un contrôle d’accès basé sur un réseau virtuel.

### <a name="are-additional-azure-rbac-permissions-needed-for-azure-cosmos-accounts-with-vnet-service-endpoints"></a>Des privilèges Azure RBAC supplémentaires sont-ils nécessaires pour les comptes Azure Cosmos dotés de points de terminaison de service de réseau virtuel ?

Si vous souhaitez apporter des modifications aux paramètres de compte après avoir ajouté les points de terminaison de service de réseau virtuel à un compte Azure Cosmos, vous devez accéder à l’action `Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action` pour tous les réseaux virtuels configurés sur votre compte Azure Cosmos. Ce privilège est requis, car le processus d’autorisation valide l’accès aux ressources (telles que les ressources de base de données et de réseau virtuel) avant d’évaluer des propriétés.
 
L’autorisation valide le privilège pour l’action de ressource de réseau virtuel, même si l’utilisateur ne spécifie pas les ACL de réseau virtuel à l’aide d’Azure CLI. Actuellement, le plan de contrôle du compte Azure Cosmos prend en charge la configuration de l’état complet du compte Azure Cosmos. L’un des paramètres des appels du plan de contrôle est `virtualNetworkRules`. Si ce paramètre n’est pas spécifié, Azure CLI effectue un appel d’extraction de base de données pour récupérer `virtualNetworkRules` et utilise cette valeur dans l’appel de mise à jour.

### <a name="do-the-peered-virtual-networks-also-have-access-to-azure-cosmos-account"></a>Les réseaux virtuels homologués ont-ils également accès au compte Azure Cosmos ? 
Seuls le réseau virtuel et ses sous-réseaux ajoutés au compte Azure Cosmos y ont accès. Leurs réseaux virtuels homologués ne peuvent pas accéder au compte tant que les sous-réseaux des réseaux virtuels homologués ne sont pas ajoutés au compte.

### <a name="what-is-the-maximum-number-of-subnets-allowed-to-access-a-single-cosmos-account"></a>Quel est le nombre maximal de sous-réseaux autorisés à accéder à un seul compte Cosmos ? 
Actuellement, vous pouvez avoir au maximum 256 sous-réseaux autorisés pour un compte Azure Cosmos.

### <a name="can-i-enable-access-from-vpn-and-express-route"></a>Puis-je activer l’accès à partir d’un VPN et d’Express Route ? 
Pour accéder localement au compte Azure Cosmos via ExpressRoute, vous devez activer le Peering Microsoft. Lorsque vous activez un pare-feu IP ou définissez des règles d’accès à un réseau virtuel, vous pouvez ajouter les adresses IP publiques utilisées pour le peering Microsoft au pare-feu IP de votre compte Azure Cosmos afin d’autoriser l’accès des services locaux au compte Azure Cosmos. 

### <a name="do-i-need-to-update-the-network-security-groups-nsg-rules"></a>Dois-je mettre à jour les règles des groupes de sécurité réseau (NSG) ? 
Les règles des groupes de sécurité réseau permettent de limiter la connectivité vers et depuis un sous-réseau de réseau virtuel. Lorsque vous ajoutez un point de terminaison de service pour Azure Cosmos DB au sous-réseau, inutile d’ouvrir la connectivité sortante dans le groupe de sécurité réseau pour votre compte Azure Cosmos. 

### <a name="are-service-endpoints-available-for-all-vnets"></a>Existe-t-il des points de terminaison de service pour tous les réseaux virtuels ?
Non, seuls les réseaux virtuels Azure Resource Manager peuvent bénéficier d’un point de terminaison de service. Les réseaux virtuels classiques ne prennent pas en charge les points de terminaison de service.

### <a name="can-i-accept-connections-from-within-public-azure-datacenters-when-service-endpoint-access-is-enabled-for-azure-cosmos-db"></a>Puis-je « Accepter les connexions provenant des centres de données Azure publics » quand l’accès aux points de terminaison de service est activé pour Azure Cosmos DB ?  
Ceci est nécessaire uniquement lorsque vous souhaitez que votre compte Azure Cosmos DB soit accessible par d’autres services Azure internes tels qu’Azure Data Factory, Recherche cognitive Azure ou tout autre service déployé dans une région Azure donnée.

## <a name="next-steps"></a>Étapes suivantes

* Pour configurer un pare-feu pour Azure Cosmos DB, voir l’article [Prise en charge du pare-feu](how-to-configure-firewall.md).
