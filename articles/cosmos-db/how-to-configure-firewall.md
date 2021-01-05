---
title: Configurer un pare-feu IP pour votre compte Azure Cosmos DB
description: Découvrez comment configurer les stratégies de contrôle d’accès IP pour la prise en charge du pare-feu dans les comptes Azure Cosmos.
author: markjbrown
ms.service: cosmos-db
ms.topic: how-to
ms.date: 12/15/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: b4e01375388f12b828d9adcb1e2ed8851061a0bf
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560727"
---
# <a name="configure-ip-firewall-in-azure-cosmos-db"></a>Configurer un pare-feu IP dans Azure Cosmos DB
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Pour sécuriser les données stockées dans votre compte, Azure Cosmos DB prend en charge un modèle d’autorisation basé sur un secret, qui utilise un code d’authentification de message basé sur un hachage (Hash-based Message Authentication Code, HMAC) fort. De plus, Azure Cosmos DB prend en charge les contrôles d’accès basés sur IP pour le pare-feu entrant. Ce modèle est semblable aux règles de pare-feu d’un système de base de données classique et renforce la sécurité de votre compte. Avec des pare-feu, vous pouvez configurer un compte Cosmos pour qu’il soit accessible uniquement à partir d’un ensemble d’ordinateurs et/ou de services cloud approuvés. L’accès aux données stockées dans votre base de données Azure Cosmos à partir de ces ensembles d’ordinateurs et de services approuvés nécessite toujours que l’appelant présente un jeton d’autorisation valide.

## <a name="ip-access-control"></a><a id="ip-access-control-overview"></a>Contrôle d’accès IP

Par défaut, votre compte Azure Cosmos est accessible depuis Internet, tant que la demande est accompagnée d’un jeton d’autorisation valide. Pour configurer le contrôle d’accès basé sur la stratégie IP, l’utilisateur doit fournir le jeu d’adresses IP ou des plages d’adresses IP au format CIDR (Classless Inter-Domain Routing) pour être ajouté à la liste d’adresses IP clientes autorisées pour un accès à un compte Azure Cosmos donné. Une fois cette configuration appliquée, toutes les demandes provenant d'ordinateurs qui ne figurent pas sur cette liste autorisée reçoivent une réponse 403 (Interdit). Lorsque vous utilisez un pare-feu IP, il est recommandé d’autoriser le portail Azure à accéder à votre compte. L’accès est requis pour autoriser l’utilisation de l’Explorateur de données ainsi que pour récupérer des métriques pour votre compte qui s’affichent sur le portail Azure. Lorsque vous utilisez l’explorateur de données, en plus de permettre au portail Azure d'accéder à votre compte, vous devez mettre à jour vos paramètres de pare-feu pour ajouter votre adresse IP actuelle aux règles de pare-feu. Notez que les modifications de pare-feu peuvent prendre jusqu’à 15 minutes pour se propager et que le pare-feu peut présenter un comportement incohérent pendant cette période.

Vous pouvez combiner des pare-feu IP avec le contrôle d’accès de réseau virtuel et de sous-réseau. En les combinant, vous pouvez limiter l’accès à n’importe quelle source qui a une adresse IP publique et/ou à partir d’un sous-réseau spécifique au sein du réseau virtuel. Pour en savoir plus sur l’utilisation du contrôle d’accès basé sur le sous-réseau et le réseau virtuel, consultez [Accéder aux ressources Azure Cosmos DB à partir de réseaux virtuels](./how-to-configure-vnet-service-endpoint.md).

Pour résumer, le jeton d’autorisation est toujours requis pour accéder à un compte Azure Cosmos. Si le pare-feu IP et la liste de contrôle d’accès (ACL) de réseau virtuel ne sont pas configurés, le compte Azure Cosmos est accessible avec le jeton d’autorisation. Une fois que le pare-feu IP ou la liste de contrôle d’accès (ACL) de réseau virtuel ou les deux sont configurés sur le compte Azure Cosmos, seules les demandes provenant des sources que vous avez spécifiées (et avec le jeton d’autorisation) obtiennent des réponses valides. 

Vous pouvez sécuriser les données stockées dans votre compte Azure Cosmos DB à l’aide de pare-feu IP. Azure Cosmos DB prend en charge les contrôles d’accès basés sur IP pour le pare-feu entrant. Vous pouvez définir un pare-feu IP sur le compte Azure Cosmos DB de l’une des manières suivantes :

* À partir du portail Azure
* De manière déclarative en utilisant un modèle Azure Resource Manager
* Par programmation avec Azure CLI ou Azure PowerShell en mettant à jour la propriété **ipRangeFilter**

## <a name="configure-an-ip-firewall-by-using-the-azure-portal"></a><a id="configure-ip-policy"></a> Configurer un pare-feu IP à l’aide du portail Azure

Pour définir la stratégie de contrôle d’accès IP dans le portail Azure, accédez à la page du compte Azure Cosmos DB, puis sélectionnez **Pare-feu et réseaux virtuels** dans le menu de navigation. Définissez la valeur **Autoriser l’accès depuis** sur **Réseaux sélectionnés**, puis choisissez **Enregistrer**.

:::image type="content" source="./media/how-to-configure-firewall/azure-portal-firewall.png" alt-text="Capture d’écran montrant comment ouvrir la page Pare-feu sur le portail Azure":::

Lorsque le contrôle d’accès IP est activé, le portail Azure offre la possibilité de spécifier des adresses IP, des plages d’adresses IP et des commutateurs. Les commutateurs permettent d’accéder à d’autres services Azure et au portail Azure. Les sections suivantes fournissent plus d’informations sur ces commutateurs.

> [!NOTE]
> Une fois la stratégie de contrôle d’accès IP activée pour votre compte Azure Cosmos DB, toutes les demandes envoyées à votre compte Azure Cosmos DB à partir d’ordinateurs ne figurant pas dans la liste autorisée de plages d’adresses IP sont refusées. L’exploration de ressources Azure Cosmos DB à partir du portail est également bloquée pour garantir l’intégrité du contrôle d’accès.

### <a name="allow-requests-from-the-azure-portal"></a>Autoriser les demandes à partir du portail Azure

Lorsque vous activez une stratégie de contrôle d’accès IP par programme, vous devez ajouter l’adresse IP du Portail Azure à la propriété **ipRangeFilter** pour conserver l’accès. Les adresses IP du portail sont :

|Région|Adresse IP|
|------|----------|
|Allemagne|51.4.229.218|
|Chine|139.217.8.252|
|Gouvernement des États-Unis|52.244.48.71|
|Toutes les autres régions|104.42.195.92,40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26|

Vous pouvez autoriser l’accès de demandes au portail Azure en sélectionnant l’option **Autoriser l’accès à partir du portail Azure**, comme illustré dans la capture d’écran suivante :

:::image type="content" source="./media/how-to-configure-firewall/enable-azure-portal.png" alt-text="Capture d’écran montrant comment activer l’accès au portail Azure":::

### <a name="allow-requests-from-global-azure-datacenters-or-other-sources-within-azure"></a>Autoriser les demandes à partir de centres de données Azure internationaux ou d’autres sources au sein d’Azure

Si vous accédez à votre compte Azure Cosmos DB à partir de services qui ne fournissent pas d’adresse IP statique (par exemple, Azure Stream Analytics et Azure Functions), vous pouvez toujours utiliser le pare-feu IP pour limiter l’accès. Vous pouvez autoriser l’accès à partir d’autres sources au sein du portail Azure en sélectionnant l’option **Accepter les connexions des centres de données Azure**, comme illustré dans la capture d’écran suivante :

:::image type="content" source="./media/how-to-configure-firewall/enable-azure-services.png" alt-text="Capture d'écran montrant comment accepter les connexions des centres de données Azure":::

Lorsque vous activez cette option, l’adresse IP `0.0.0.0` est ajoutée à la liste des adresses IP autorisées. L’adresse IP `0.0.0.0` limite les demandes envoyées à votre compte Azure Cosmos DB à partir de la plage d’adresses IP du centre de données Azure. Ce paramètre n’autorise aucune autre plage d’adresses IP à accéder à votre compte Azure Cosmos DB.

> [!NOTE]
> Cette option configure le pare-feu pour autoriser toutes les demandes d’Azure, notamment les demandes envoyées par les abonnements d’autres clients déployés dans Azure. Comme la liste des adresses IP autorisées par cette option est grande, elle limite l’efficacité d’une stratégie de pare-feu. Utilisez cette option uniquement si vos demandes ne proviennent pas d’adresses IP statiques ou de sous-réseaux de réseaux virtuels. Comme le portail est déployé dans Azure, quand vous choisissez cette option, vous autorisez automatiquement l’accès à partir du portail Azure.

### <a name="requests-from-your-current-ip"></a>Demandes à partir de votre adresse IP actuelle

Pour simplifier le développement, le portail Azure vous aide à identifier et à ajouter l’adresse IP de votre ordinateur client à la liste autorisée, afin que les applications qui exécutent votre machine puissent accéder au compte Azure Cosmos DB.

Le portail détecte automatiquement l’adresse IP du client. Il peut s’agir de l’adresse IP de votre ordinateur ou de votre passerelle réseau. N’oubliez pas de supprimer cette adresse IP avant de mettre vos charges de travail en production.

Pour ajouter votre adresse IP actuelle à la liste des adresses IP, sélectionnez **Ajouter mon adresse IP actuelle**. Ensuite, sélectionnez **Enregistrer**.

:::image type="content" source="./media/how-to-configure-firewall/enable-current-ip.png" alt-text="Capture d’écran montrant comment configurer les paramètres de pare-feu pour l’adresse IP actuelle":::

### <a name="requests-from-cloud-services"></a>Demandes à partir de services cloud

Dans Azure, les services cloud sont une méthode courante d’hébergement de la logique de service de couche intermédiaire à l’aide d’Azure Cosmos DB. Pour autoriser l’accès à votre compte Azure Cosmos DB à partir d’un service cloud, vous devez ajouter l’adresse IP publique de ce dernier à la liste autorisée d’adresses IP associées à votre compte Azure Cosmos DB en [configurant la stratégie de contrôle d’accès IP](#configure-ip-policy). De cette façon, toutes les instances de rôle des services cloud ont accès à votre compte Azure Cosmos DB.

Vous pouvez récupérer des adresses IP pour vos services cloud dans le portail Azure, comme illustré dans la capture d’écran suivante :

:::image type="content" source="./media/how-to-configure-firewall/public-ip-addresses.png" alt-text="Capture d’écran illustrant l’adresse IP publique pour un service cloud affichée dans le portail Azure":::

Quand vous effectuez un scale-out de votre service cloud en ajoutant des instances de rôle, ces nouvelles instances ont automatiquement accès au compte Azure Cosmos DB car elles font partie du même service cloud.

### <a name="requests-from-virtual-machines"></a>Demandes à partir de machines virtuelles

Vous pouvez également utiliser des [machines virtuelles](https://azure.microsoft.com/services/virtual-machines/) ou des [jeux de mise à l’échelle de machine virtuelle](../virtual-machine-scale-sets/overview.md) pour héberger les services de couche intermédiaire à l’aide d’Azure Cosmos DB. Pour configurer votre compte Cosmos DB de telle manière qu’il autorise l’accès à partir de machines virtuelles, vous devez configurer l’adresse IP publique de la machine virtuelle et/ou le groupe de machines virtuelles identiques comme étant l’une des adresses IP autorisées pour votre compte Azure Cosmos DB en [configurant la stratégie de contrôle d’accès IP](#configure-ip-policy).

Vous pouvez récupérer des adresses IP pour les machines virtuelles dans le portail Azure, comme illustré dans la capture d’écran suivante :

:::image type="content" source="./media/how-to-configure-firewall/public-ip-addresses-dns.png" alt-text="Capture d’écran illustrant une adresse IP publique pour une machine virtuelle affichée dans le portail Azure":::

Quand vous ajoutez des instances de machine virtuelle au groupe, elles ont automatiquement accès à votre compte Azure Cosmos DB.

### <a name="requests-from-the-internet"></a>Demandes à partir d’Internet

Quand vous accédez à votre compte Azure Cosmos DB à partir d’un ordinateur sur Internet, l’adresse IP ou la plage d’adresses IP de l’ordinateur doit être ajoutée à la liste d’adresses IP autorisées pour votre compte.

## <a name="configure-an-ip-firewall-by-using-a-resource-manager-template"></a><a id="configure-ip-firewall-arm"></a>Configurer un pare-feu IP à l’aide d’un modèle Resource Manager

Pour configurer le contrôle d'accès à votre compte Azure Cosmos DB, vérifiez que le modèle Resource Manager spécifie la propriété **ipRules** avec un tableau de plages d'adresses IP autorisées. Si vous configurez le pare-feu IP sur un compte Cosmos déjà déployé, vérifiez que le tableau `locations` correspond à ce qui est actuellement déployé. Vous ne pouvez pas modifier simultanément le tableau `locations` et d’autres propriétés. Pour plus d’informations et pour obtenir des exemples de modèles Azure Resource Manager pour Azure Cosmos DB, consultez [Modèles Azure Resource Manager pour Azure Cosmos DB](./templates-samples-sql.md).

> [!IMPORTANT]
> La propriété **ipRules** a été introduite avec la version 2020-04-01 de l'API. Les versions précédentes exposaient plutôt une propriété **ipRangeFilter**, qui correspond à une liste d'adresses IP séparées par des virgules.

L'exemple ci-dessous montre comment la propriété **ipRules** est exposée dans la version 2020-04-01 ou ultérieure de l'API :

```json
{
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "name": "[variables('accountName')]",
  "apiVersion": "2020-04-01",
  "location": "[parameters('location')]",
  "kind": "GlobalDocumentDB",
  "properties": {
    "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
    "locations": "[variables('locations')]",
    "databaseAccountOfferType": "Standard",
    "enableAutomaticFailover": "[parameters('automaticFailover')]",
    "ipRules": [
      {
        "ipAddressOrRange": "40.76.54.131"
      },
      {
        "ipAddressOrRange": "52.176.6.30"
      },
      {
        "ipAddressOrRange": "52.169.50.45"
      },
      {
        "ipAddressOrRange": "52.187.184.26"
      }
    ]
  }
}
```

Voici le même exemple pour toute version de l'API antérieure à la version 2020-04-01 :

```json
{
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "name": "[variables('accountName')]",
  "apiVersion": "2019-08-01",
  "location": "[parameters('location')]",
  "kind": "GlobalDocumentDB",
  "properties": {
    "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
    "locations": "[variables('locations')]",
    "databaseAccountOfferType": "Standard",
    "enableAutomaticFailover": "[parameters('automaticFailover')]",
    "ipRangeFilter":"40.76.54.131,52.176.6.30,52.169.50.45,52.187.184.26"
  }
}
```

## <a name="configure-an-ip-access-control-policy-by-using-the-azure-cli"></a><a id="configure-ip-firewall-cli"></a>Configurer une stratégie de contrôle d’accès IP à l’aide d’Azure CLI

La commande suivante montre comment créer un compte Azure Cosmos DB avec le contrôle d’accès IP :

```azurecli-interactive
# Create a Cosmos DB account with default values and IP Firewall enabled
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
ipRangeFilter='192.168.221.17,183.240.196.255,40.76.54.131'

# Make sure there are no spaces in the comma-delimited list of IP addresses or CIDR ranges.
az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --locations regionName='East US 2' failoverPriority=1 isZoneRedundant=False \
    --ip-range-filter $ipRangeFilter
```

## <a name="configure-an-ip-access-control-policy-by-using-powershell"></a><a id="configure-ip-firewall-ps"></a>Configurer une stratégie de contrôle d’accès IP à l’aide de PowerShell

Le script suivant montre comment créer un compte Azure Cosmos DB avec contrôle d’accès IP :

```azurepowershell-interactive
# Create a Cosmos DB account with default values and IP Firewall enabled
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$ipRules = @("192.168.221.17","183.240.196.255","40.76.54.131")

$locations = @(
    @{ "locationName"="West US 2"; "failoverPriority"=0; "isZoneRedundant"=False },
    @{ "locationName"="East US 2"; "failoverPriority"=1, "isZoneRedundant"=False }
)

# Make sure there are no spaces in the comma-delimited list of IP addresses or CIDR ranges.
$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$locations;
    "ipRules"=$ipRules
}

New-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2020-04-01" -ResourceGroupName $resourceGroupName `
    -Name $accountName -PropertyObject $CosmosDBProperties
```

## <a name="troubleshoot-issues-with-an-ip-access-control-policy"></a><a id="troubleshoot-ip-firewall"></a>Résoudre les problèmes de stratégie de contrôle d’accès IP

Vous pouvez résoudre les problèmes de stratégie de contrôle d’accès IP en utilisant les options suivantes :

### <a name="azure-portal"></a>Portail Azure

En activant une stratégie de contrôle d’accès IP pour votre compte Azure Cosmos DB, vous bloquez toutes les demandes envoyées à votre compte à partir d’ordinateurs ne figurant pas dans la liste autorisée de plages d’adresses IP. Pour autoriser des opérations de plan de données du portail comme la navigation dans les conteneurs et l’interrogation de documents, vous devez autoriser explicitement l’accès au portail Azure à l’aide du volet **Pare-feu** du portail.

### <a name="sdks"></a>Kits SDK

Lorsque vous accédez à des ressources Azure Cosmos DB à l’aide de kits SDK sur des ordinateurs qui ne sont pas dans la liste autorisée, une réponse générique **403 Interdit** est renvoyée sans plus de détails. Vérifiez la liste d’adresses IP autorisées pour votre compte et vérifiez que la configuration de stratégie appropriée est appliquée à votre compte Azure Cosmos DB.

### <a name="source-ips-in-blocked-requests"></a>Adresses IP sources dans les demandes bloquées

Activez la journalisation des diagnostics sur votre compte Azure Cosmos DB. Ces journaux d’activité affichent chaque demande et réponse. Les messages associés au pare-feu sont journalisés avec le code de retour 403. En filtrant ces messages, vous pouvez voir les adresses IP sources des demandes bloquées. Consultez [Journalisation des diagnostics Azure Cosmos DB](./monitor-cosmos-db.md).

### <a name="requests-from-a-subnet-with-a-service-endpoint-for-azure-cosmos-db-enabled"></a>Demandes à partir d’un sous-réseau avec un point de terminaison de service activé pour Azure Cosmos DB

Les demandes à partir d’un sous-réseau de réseau virtuel avec un point de terminaison de service activé pour Azure Cosmos DB envoient l’identité de réseau virtuel et de sous-réseau aux comptes Azure Cosmos DB. Comme ces demandes n’ont pas l’adresse IP publique de la source, les filtres IP les rejettent. Pour autoriser l’accès à partir de sous-réseaux spécifiques de réseaux virtuels, ajoutez une liste de contrôle d’accès, comme indiqué dans [Configurer l’accès à votre compte Azure Cosmos DB à partir d’un réseau virtuel et d’un sous-réseau](how-to-configure-vnet-service-endpoint.md). L’application de règles de pare-feu peut prendre jusqu’à 15 minutes et le pare-feu peut présenter un comportement incohérent pendant cette période.

### <a name="private-ip-addresses-in-list-of-allowed-addresses"></a>Adresses IP privées dans la liste des adresses autorisées

La création ou la mise à jour d'un compte Azure Cosmos avec une liste d'adresses autorisées contenant des adresses IP privées échouera. Assurez-vous qu'aucune adresse IP privée ne figure dans la liste.

## <a name="next-steps"></a>Étapes suivantes

Pour configurer un point de terminaison de service de réseau virtuel pour votre compte Azure Cosmos DB, consultez les articles suivants :

* [Contrôle d’accès à votre compte Azure Cosmos DB à partir d’un réseau virtuel et d’un sous-réseau](how-to-configure-vnet-service-endpoint.md)
* [Configurer l’accès à votre compte Azure Cosmos DB à partir d’un réseau virtuel et d’un sous-réseau](how-to-configure-vnet-service-endpoint.md)