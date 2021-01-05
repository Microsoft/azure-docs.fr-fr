---
title: Intégrer Azure Relay au service Azure Private Link
description: Découvrir comment intégrer Azure Relay au service Azure Private Link
ms.date: 09/24/2020
ms.topic: article
ms.openlocfilehash: 10d82fe8e272ed18dcc339830dfef0f71d4b2ddb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91263846"
---
# <a name="integrate-azure-relay-with-azure-private-link-preview"></a>Intégrer Azure Relay à Azure Private Link (préversion)
Le service **Azure Private Link** vous permet d’accéder aux services Azure (par exemple, Azure Relay, Azure Service Bus, Azure Event Hubs, Stockage Azure et Azure Cosmos DB) ainsi qu’aux services de partenaires/clients hébergés par Azure sur un point de terminaison privé de votre réseau virtuel. Pour plus d’informations, consultez [Qu’est-ce qu’Azure Private Link (préversion) ?](../private-link/private-link-overview.md)

Un **point de terminaison privé** est une interface réseau qui permet à vos charges de travail exécutées dans un réseau virtuel de se connecter en privé et en toute sécurité à un service qui dispose d’une **ressource de liaison privée** (par exemple, un espace de noms Azure Relay). Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ni d’IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez fournir un niveau de granularité dans le contrôle d’accès en autorisant des connexions à des espaces de noms Azure Relay spécifiques. 


## <a name="add-a-private-endpoint-using-azure-portal"></a>Ajouter un point de terminaison privé avec le portail Azure

### <a name="prerequisites"></a>Prérequis
Pour intégrer un espace de noms Azure Relay à Azure Private Link (préversion), vous avez besoin des entités ou autorisations suivantes :

- Un espace de noms Azure Relay.
- Un réseau virtuel Azure.
- Un sous-réseau dans le réseau virtuel.
- Des autorisations de propriétaire ou de contributeur sur le réseau virtuel.

Votre point de terminaison privé et votre réseau virtuel doivent se trouver dans la même région. Au moment de sélectionner la région du point de terminaison privé sur le portail, les réseaux virtuels qui se trouvent dans cette région sont filtrés automatiquement. Votre espace de noms peut se trouver dans une autre région.

Votre point de terminaison privé utilise une adresse IP privée de votre réseau virtuel.

### <a name="steps"></a>Étapes
Pour obtenir des instructions pas à pas sur la création d’un espace de noms Azure Relay et des entités qu’il contient, consultez [Créer un espace de noms Azure Relay à l’aide du portail Azure](relay-create-namespace-portal.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Dans la barre de recherche, saisissez **Relay**.
3. Dans la liste, sélectionnez l’**espace de noms** auquel vous voulez ajouter un point de terminaison privé.
4. Sélectionnez l’onglet **Réseau** situé sous **Paramètres**.
5. Sélectionnez l’onglet **Connexions de point de terminaison privé (préversion)** en haut de la page.
6. Sélectionnez le bouton **+ Point de terminaison privé** en haut de la page.

    ![Bouton Ajouter un point de terminaison privé](./media/private-link-service/add-private-endpoint-button.png)
7. Dans la page **Informations de base**, suivez ces étapes : 
    1. Sélectionnez l’**abonnement Azure** où créer le point de terminaison privé. 
    2. Sélectionnez le **groupe de ressources** pour la ressource de point de terminaison privé.
    3. Entrez un **nom** pour le point de terminaison privé. 
    5. Sélectionnez une **région** pour le point de terminaison privé. La région de votre point de terminaison privé doit être la même que celle de votre réseau virtuel, mais elle peut être différente de celle de l’espace de noms Azure Relay auquel vous vous connectez. 
    6. Sélectionnez **Suivant : Bouton Ressource >** en bas de la page.

        ![Créer un point de terminaison privé - page Informations de base](./media/private-link-service/create-private-endpoint-basics-page.png)
8. Dans la page **Ressource**, suivez ces étapes :
    1. Pour la méthode de connexion, si vous sélectionnez **Se connecter à une ressource Azure dans mon répertoire**, que vous avez un accès propriétaire ou contributeur à l’espace de noms et que cet espace de noms se trouve dans le même répertoire que le point de terminaison privé, procédez comme suit : 
        1. Sélectionnez l’**abonnement Azure** où se trouve votre **espace de noms Azure Relay**. 
        2. Pour **Type de ressource**, sélectionnez **Microsoft.Relay/namespaces** pour le **type de ressource**.
        3. Pour **Ressource**, sélectionnez un espace de noms Azure Relay dans la liste déroulante. 
        4. Confirmez que la **Sous-ressource cible** est définie sur **espace de noms**.
        5. Sélectionnez **Suivant : Bouton Configuration >** en bas de la page. 
        
            ![Créer un point de terminaison privé - page Ressource](./media/private-link-service/create-private-endpoint-resource-page.png)    
    2. Si vous sélectionnez **Se connecter à une ressource Azure par ID de ressource ou alias** parce que l’espace de noms n’est pas sous le même répertoire que celui du point de terminaison privé, procédez comme suit :
        1. Entrez l’**ID de ressource** ou l’**alias**. Il peut s’agir de l’ID de ressource ou de l’alias que quelqu’un a partagé avec vous. Le moyen le plus simple d’obtenir l’ID de la ressource est d’accéder à l’espace de noms Azure Relay dans le portail Azure et de copier la partie de l’URI à partir de `/subscriptions/`. Voici un exemple : `/subscriptions/000000000-0000-0000-0000-000000000000000/resourceGroups/myresourcegroup/providers/Microsoft.Relay/namespaces/myrelaynamespace.` 
        2. Pour **Sous-ressource cible**, entrez **espace de noms**. Il s’agit du type de la sous-ressource à laquelle votre point de terminaison privé peut accéder.
        3. (facultatif) Entrez un **message de demande**. Le propriétaire de la ressource voit ce message quand il gère la connexion de point de terminaison privé.
        4. Ensuite, sélectionnez **Suivant : Bouton Configuration >** en bas de la page.

            ![Créer un point de terminaison privé - Se connecter à l’aide d’un ID de ressource](./media/private-link-service/connect-resource-id.png)
9. Dans la page **Configuration**, vous sélectionnez le sous-réseau dans un réseau virtuel sur lequel vous voulez déployer le point de terminaison privé. 
    1. Sélectionnez un **réseau virtuel**. Seuls les réseaux virtuels dans l’abonnement et l’emplacement sélectionnés sont présents dans la liste déroulante. 
    2. Sélectionnez un **sous-réseau** dans le réseau virtuel que vous avez sélectionné. 
    3. Activez **Intégrer à une zone DNS privée** si vous souhaitez intégrer votre point de terminaison privé à une zone DNS privée. 
    
        Pour vous connecter en privé à votre point de terminaison privé, vous avez besoin d’un enregistrement DNS. Nous vous recommandons d’intégrer votre point de terminaison privé à une **zone DNS privée**. Vous pouvez également utiliser vos propres serveurs DNS, ou créer des enregistrements DNS à l’aide des fichiers hôtes sur vos machines virtuelles. Pour plus d’informations, consultez [Configuration DNS des points de terminaison privés Azure](../private-link/private-endpoint-dns.md). Dans cet exemple, l’option **Intégrer à une zone DNS privée** est sélectionnée et une zone DNS privée est créée pour vous. 
    3. Sélectionnez **Suivant : Bouton Étiquettes >** en bas de la page. 

        ![Créer un point de terminaison privé - page Configuration](./media/private-link-service/create-private-endpoint-configuration-page.png)
10. Sur la page **Étiquettes**, créez les étiquettes (noms et valeurs) à associer au point de terminaison privé et à la zone DNS privée (si vous avez activé l’option). Ensuite, sélectionnez le bouton **Vérifier + créer** en bas de la page. 
11. Dans la page **Vérifier + créer**, examinez tous les paramètres et sélectionnez **Créer** pour créer le point de terminaison privé.
    
    ![Créer un point de terminaison privé - page Vérifier et créer](./media/private-link-service/create-private-endpoint-review-create-page.png)
12. Sur la page **Point de terminaison privé**, vous pouvez voir l’état de la connexion de point de terminaison privé. Si vous êtes le propriétaire de l’espace de noms Azure Relay ou si vous en avez la gestion et que vous avez sélectionné l’option **Se connecter à une ressource Azure dans mon répertoire** pour la **méthode de connexion**, la connexion au point de terminaison doit être **approuvée automatiquement**. Si elle est à l’état **En attente**, consultez la section [Gérer des points de terminaison privés avec le Portail Azure](#manage-private-endpoints-using-azure-portal).

    ![Page Point de terminaison privé](./media/private-link-service/private-endpoint-page.png)
13. Revenez à la page **Mise en réseau** de l’**espace de noms** et basculez sur l’onglet **Connexions de points de terminaison privés (préversion)** . Vous devez voir le point de terminaison privé que vous avez créé. 

    ![Point de terminaison privé créé](./media/private-link-service/private-endpoint-created.png)

## <a name="add-a-private-endpoint-using-powershell"></a>Ajouter un point de terminaison privé avec PowerShell
L’exemple suivant montre comment utiliser Azure PowerShell pour créer une connexion de point de terminaison privé à un espace de noms Azure Relay.

Votre point de terminaison privé et votre réseau virtuel doivent se trouver dans la même région. Votre espace de noms Azure Relay peut se trouver dans une autre région. Et votre point de terminaison privé utilise une adresse IP privée dans votre réseau virtuel.

```azurepowershell-interactive

$rgName = "<RESOURCE GROUP NAME>"
$vnetlocation = "<VNET LOCATION>"
$vnetName = "<VIRTUAL NETWORK NAME>"
$subnetName = "<SUBNET NAME>"
$namespaceLocation = "<NAMESPACE LOCATION>"
$namespaceName = "<NAMESPACE NAME>"
$peConnectionName = "<PRIVATE ENDPOINT CONNECTION NAME>"

# create resource group
az group create -l $vnetLocation -n $rgName

# create virtual network
$virtualNetwork = New-AzVirtualNetwork `
                    -ResourceGroupName $rgName `
                    -Location $vnetlocation `
                    -Name $vnetName `
                    -AddressPrefix 10.0.0.0/16

# create subnet with endpoint network policy disabled
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
                    -Name $subnetName `
                    -AddressPrefix 10.0.0.0/24 `
                    -PrivateEndpointNetworkPoliciesFlag "Disabled" `
                    -VirtualNetwork $virtualNetwork

# update virtual network
$virtualNetwork | Set-AzVirtualNetwork

# create a relay namespace
$namespaceResource = New-AzResource -Location $namespaceLocation -ResourceName $namespaceName -ResourceGroupName $rgName -Properties @{} -ResourceType "Microsoft.Relay/namespaces" 

# create a private link service connection
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
                                -Name $peConnectionName `
                                -PrivateLinkServiceId $namespaceResource.ResourceId `
                                -GroupId "namespace"

# get subnet object that you will use in the next step                                
$virtualNetwork = Get-AzVirtualNetwork -ResourceGroupName  $rgName -Name $vnetName
$subnet = $virtualNetwork | Select -ExpandProperty subnets `
                                | Where-Object  {$_.Name -eq $subnetName}  
   
# now, create private endpoint   
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $rgName  `
                                -Name $vnetName   `
                                -Location $vnetlocation `
                                -Subnet  $subnet   `
                                -PrivateLinkServiceConnection $privateEndpointConnection

(Get-AzResource -ResourceId $namespaceResource.ResourceId -ExpandProperties).Properties


```

## <a name="manage-private-endpoints-using-azure-portal"></a>Gérer des points de terminaison privés avec le portail Azure

Quand vous créez un point de terminaison privé, la connexion doit être approuvée. Si la ressource (espace de noms Azure Relay) pour laquelle vous créez un point de terminaison privé se trouve dans votre répertoire, vous pouvez approuver la requête de connexion à condition d’avoir les privilèges de gestion sur l’espace de noms Azure Relay. Si vous vous connectez à un espace de noms Azure Relay dont vous n’avez pas la gestion, vous devez attendre que le propriétaire de cette ressource approuve votre requête de connexion.

Il existe quatre états de provisionnement :

| Action de service | État du point de terminaison privé de l’utilisateur du service | Description |
|--|--|--|
| None | Pending | La connexion est créée manuellement et est en attente de l’approbation du propriétaire de l’espace de noms Azure Relay. |
| Approbation | Approved | La connexion a été approuvée automatiquement ou manuellement et est prête à être utilisée. |
| Rejeter | Rejeté | La connexion a été rejetée par le propriétaire de l’espace de noms Azure Relay. |
| Supprimer | Déconnecté | La connexion a été supprimée par le propriétaire de l’espace de noms Azure Relay, le point de terminaison privé devient donc informatif et doit être supprimé dans le cadre d’un nettoyage. |
 
###  <a name="approve-reject-or-remove-a-private-endpoint-connection"></a>Approuver, rejeter ou supprimer une connexion de point de terminaison privé

1. Connectez-vous au portail Azure.
1. Dans la barre de recherche, saisissez **Relay**.
1. Sélectionnez l’**espace de noms** que vous voulez gérer.
1. Sélectionnez l’onglet **Réseau**.
5. Accédez à la section appropriée ci-dessous en fonction de l’opération souhaitée : approuver, rejeter ou supprimer. 

### <a name="approve-a-private-endpoint-connection"></a>Approuver une connexion de point de terminaison privé

1. Si une connexion est en attente, elle est listée avec l’état de provisionnement **En attente**. 
2. Sélectionnez le **point de terminaison privé** à approuver
3. Sélectionnez le bouton **Approuver**.

    ![Approuver le point de terminaison privé](./media/private-link-service/private-endpoint-approve.png)
4. Dans la page **Approuver la connexion**, entrez éventuellement un **commentaire** et sélectionnez **Oui**. Si vous sélectionnez **Non**, il ne se passe rien. 

    ![Page Approuver la connexion](./media/private-link-service/approve-connection-page.png)
5. Vous devez voir que l’état de la connexion dans la liste est remplacé par **Approuvé**.

### <a name="reject-a-private-endpoint-connection"></a>Rejeter une connexion de point de terminaison privé

1. Si vous voulez rejeter une connexion de point de terminaison privé, qu’il s’agisse d’une demande en attente ou d’une connexion existante approuvée précédemment, sélectionnez la connexion de point de terminaison privé et cliquez sur le bouton **Rejeter**.

    ![Bouton Rejeter](./media/private-link-service/private-endpoint-reject.png)
2. Dans la page **Rejeter la connexion**, entrez éventuellement un commentaire et sélectionnez **Oui**. Si vous sélectionnez **Non**, il ne se passe rien. 

    ![Page Rejeter la connexion](./media/private-link-service/reject-connection-page.png)
3. Vous devez voir que l’état de la connexion dans la liste est remplacé par **Rejeté**.


### <a name="remove-a-private-endpoint-connection"></a>Supprimer une connexion de point de terminaison privé

1. Pour supprimer une connexion de point de terminaison privé, sélectionnez-la dans la liste et sélectionnez **Supprimer** dans la barre d’outils. 

    ![Bouton Supprimer](./media/private-link-service/remove-endpoint.png)
2. Dans la page **Supprimer la connexion**, sélectionnez **Oui** pour confirmer la suppression du point de terminaison privé. Si vous sélectionnez **Non**, il ne se passe rien. 

    ![Page Supprimer la connexion](./media/private-link-service/delete-connection-page.png)
3. Vous devez voir que l’état a été remplacé par **Déconnecté**. Vous voyez ensuite que le point de terminaison n’est plus dans la liste. 

## <a name="validate-that-the-private-link-connection-works"></a>Vérifier le fonctionnement de la connexion à liaison privée
Vous devez vérifier que les ressources qui se trouvent sur le réseau virtuel du point de terminaison privé se connectent à votre espace de noms Azure Relay par le biais de son adresse IP privée.

Pour ce test, créez une machine virtuelle en suivant les étapes décrites dans [Créer une machine virtuelle Windows dans le portail Azure](../virtual-machines/windows/quick-create-portal.md).

Sous l’onglet **Réseau** : 

1. Spécifiez un **réseau virtuel** et un **sous-réseau**. Vous devez sélectionner le réseau virtuel sur lequel vous avez déployé le point de terminaison privé.
2. Spécifiez une ressource d’**adresse IP publique**.
3. Pour **Groupe de sécurité réseau de la carte réseau**, sélectionnez **Aucun**.
4. Pour **Équilibrage de charge**, sélectionnez **Non**.

Connectez-vous à la machine virtuelle, ouvrez la ligne de commande et exécutez la commande suivante :

```console
nslookup <your-relay-namespace-name>.servicebus.windows.net
```

Vous devez obtenir un résultat similaire à la sortie suivante. 

```console
Non-authoritative answer:
Name:    <namespace-name>.privatelink.servicebus.windows.net
Address:  10.0.0.4 (private IP address associated with the private endpoint)
Aliases:  <namespace-name>.servicebus.windows.net
```

## <a name="limitations-and-design-considerations"></a>Limitations et remarques sur la conception

### <a name="design-considerations"></a>Remarques relatives à la conception
- Le point de terminaison privé pour Azure Relay est en **préversion publique**. 
- Pour obtenir des informations sur les prix, consultez [Tarifs Azure Private Link (préversion)](https://azure.microsoft.com/pricing/details/private-link/).

### <a name="limitations"></a>Limites 
- Nombre maximal de points de terminaison privés par espace de noms Azure Relay : 64.
- Nombre maximal d’espaces de noms Azure Relay avec points de terminaison privés par abonnement : 64.
- Les règles de groupe de sécurité réseau et les routes définies par l’utilisateur ne s’appliquent pas au point de terminaison privé. Pour plus d’informations, consultez [Service Liaison privée : Limitations](../private-link/private-link-service-overview.md#limitations)

## <a name="next-steps"></a>Étapes suivantes

- Découvrir plus en détail [Azure Private Link (préversion)](../private-link/private-link-service-overview.md)
- En savoir plus sur [Azure Relay](relay-what-is-it.md)
