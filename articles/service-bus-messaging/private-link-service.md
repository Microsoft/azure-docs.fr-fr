---
title: Intégrer Azure Service Bus au service Azure Private Link
description: Découvrir comment intégrer Azure Service Bus au service Azure Private Link
author: spelluru
ms.author: spelluru
ms.date: 10/07/2020
ms.topic: article
ms.openlocfilehash: 66de9a4ff65c73264257cb6f7f215fc15820c95f
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94427145"
---
# <a name="allow-access-to-azure-service-bus-namespaces-via-private-endpoints"></a>Autoriser l’accès aux espaces de noms Azure Service Bus via des points de terminaison privés
Le service Azure Private Link vous permet d’accéder aux services Azure (par exemple, Azure Service Bus, le Stockage Azure et Azure Cosmos DB) ainsi qu’aux services de partenaires/clients hébergés par Azure sur un **point de terminaison privé** de votre réseau virtuel.

> [!IMPORTANT]
> Cette fonctionnalité est prise en charge avec le niveau **Premium** d’Azure Service Bus. Pour plus d’informations sur le niveau Premium, consultez l’article [Couches de messagerie Service Bus Premium et Standard](service-bus-premium-messaging.md).

Un point de terminaison privé est une interface réseau qui vous permet de vous connecter de façon privée et sécurisée à un service basé sur Azure Private Link. Le point de terminaison privé utilise une adresse IP privée de votre réseau virtuel, plaçant de fait le service dans votre réseau virtuel. Sachant que l’ensemble du trafic à destination du service peut être routé via le point de terminaison privé, il n’y a aucun besoin de passerelles, d’appareils NAT, de connexions ExpressRoute ou VPN ou d’adresses IP publiques. Le trafic entre votre réseau virtuel et le service transite par le réseau principal de Microsoft, éliminant ainsi toute exposition à l’Internet public. Vous pouvez vous connecter à une instance d’une ressource Azure, ce qui vous donne le plus haut niveau de granularité en matière de contrôle d’accès.

Pour plus d’informations, consultez [Qu’est-ce qu’Azure Private Link ?](../private-link/private-link-overview.md)

>[!WARNING]
> L’implémentation de points de terminaison privés peut empêcher d’autres services Azure d’interagir avec Service Bus. En guise d’exception, vous pouvez autoriser l’accès aux ressources Service Bus à partir de certains services approuvés, même lorsque les points de terminaison privés sont activés. Pour obtenir la liste des services approuvés, consultez [Services approuvés](#trusted-microsoft-services).
>
> Les services Microsoft suivants doivent se trouver sur un réseau virtuel
> - Azure App Service
> - Azure Functions



## <a name="add-a-private-endpoint-using-azure-portal"></a>Ajouter un point de terminaison privé avec le portail Azure

### <a name="prerequisites"></a>Prérequis

Pour intégrer un espace de noms Service Bus à Azure Private Link, vous avez besoin des entités ou autorisations suivantes :

- Un espace de noms Service Bus.
- Un réseau virtuel Azure.
- Un sous-réseau dans le réseau virtuel. Vous pouvez utiliser le sous-réseau **par défaut**. 
- Des autorisations de propriétaire ou de contributeur à la fois pour l’espace de noms Service Bus et le réseau virtuel.

Votre point de terminaison privé et votre réseau virtuel doivent se trouver dans la même région. Au moment de sélectionner la région du point de terminaison privé sur le portail, les réseaux virtuels qui se trouvent dans cette région sont filtrés automatiquement. Votre espace de noms Service Bus peut se trouver dans une autre région. Et votre point de terminaison privé utilise une adresse IP privée dans votre réseau virtuel.

### <a name="steps"></a>steps

Si vous avez déjà un espace de noms, vous pouvez créer un point de terminaison privé en suivant ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
2. Dans la barre de recherche, saisissez **Service Bus**.
3. Dans la liste, sélectionnez l’**espace de noms** auquel vous voulez ajouter un point de terminaison privé.
2. Dans le menu de gauche, sélectionnez l’option **Réseaux** sous **Paramètres**. 

    > [!NOTE]
    > L’onglet **Réseau** s’affiche uniquement pour les espaces de noms **Premium**.  
    
    Par défaut, l’option **Réseaux sélectionnés** est sélectionnée. Si vous n’ajoutez pas au moins une règle de pare-feu IP ou un réseau virtuel sur cette page, l’espace de noms est accessible via l’Internet public (à l’aide de la clé d’accès).

    :::image type="content" source="./media/service-bus-ip-filtering/default-networking-page.png" alt-text="Page Mise en réseau - Par défaut" lightbox="./media/service-bus-ip-filtering/default-networking-page.png":::
    
    Si vous sélectionnez l’option **Tous les réseaux**, votre espace de noms Service Bus accepte les connexions depuis n’importe quelle adresse IP (à l’aide de la clé d’accès). Ce paramètre par défaut est équivalent à une règle qui accepte la plage d’adresses IP 0.0.0.0/0. 

    ![Option Pare-feu - Tous les réseaux sélectionnée](./media/service-bus-ip-filtering/firewall-all-networks-selected.png)
5. Pour autoriser l’accès à l’espace de noms via des points de terminaison privés, sélectionnez l’onglet **Connexions des points de terminaison privés** en haut de la page
6. Sélectionnez le bouton **+ Point de terminaison privé** en haut de la page.

    ![Bouton Ajouter un point de terminaison privé](./media/private-link-service/private-link-service-3.png)
7. Dans la page **Informations de base**, suivez ces étapes : 
    1. Sélectionnez l’**abonnement Azure** où créer le point de terminaison privé. 
    2. Sélectionnez le **groupe de ressources** pour la ressource de point de terminaison privé.
    3. Entrez un **nom** pour le point de terminaison privé. 
    5. Sélectionnez une **région** pour le point de terminaison privé. La région de votre point de terminaison privé doit être la même que celle de votre réseau virtuel, mais elle peut être différente de celle de la ressource de lien privé à laquelle vous vous connectez. 
    6. Sélectionnez **Suivant : Bouton Ressource >** en bas de la page.

        ![Créer un point de terminaison privé - page Informations de base](./media/private-link-service/create-private-endpoint-basics-page.png)
8. Dans la page **Ressource**, suivez ces étapes :
    1. Pour la méthode de connexion, si vous sélectionnez **Se connecter à une ressource Azure dans mon répertoire**, suivez ces étapes :   
        1. Sélectionnez l’**abonnement Azure** où se trouve votre **espace de noms Service Bus**. 
        2. Pour **Type de ressource**, sélectionnez **Microsoft.ServiceBus/namespaces**.
        3. Pour **Ressource**, sélectionnez un espace de noms Service Bus dans la liste déroulante. 
        4. Confirmez que la **Sous-ressource cible** est définie sur **espace de noms**.
        5. Sélectionnez **Suivant : Bouton Configuration >** en bas de la page. 
        
            ![Créer un point de terminaison privé - page Ressource](./media/private-link-service/create-private-endpoint-resource-page.png)
    2. Si vous sélectionnez **Se connecter à une ressource Azure par alias ou ID de ressource**, suivez ces étapes :
        1. Entrez l’**ID de ressource** ou l’**alias**. Il peut s’agir de l’ID de ressource ou de l’alias que quelqu’un a partagé avec vous. Le moyen le plus simple d’obtenir l’ID de ressource est d’accéder à l’espace de noms Service Bus dans le portail Azure et de copier la partie de l’URI à partir de `/subscriptions/`. L’image suivante montre un exemple. 
        2. Pour **Sous-ressource cible**, entrez **espace de noms**. Il s’agit du type de la sous-ressource à laquelle votre point de terminaison privé peut accéder. 
        3. (facultatif) Entrez un **message de demande**. Le propriétaire de la ressource voit ce message quand il gère la connexion de point de terminaison privé. 
        4. Ensuite, sélectionnez **Suivant : Bouton Configuration >** en bas de la page. 

            ![Créer un point de terminaison privé - se connecter à l’aide d’un ID de ressource](./media/private-link-service/connect-resource-id.png)
9. Dans la page **Configuration**, vous sélectionnez le sous-réseau dans un réseau virtuel sur lequel vous voulez déployer le point de terminaison privé. 
    1. Sélectionnez un **réseau virtuel**. Seuls les réseaux virtuels dans l’abonnement et l’emplacement sélectionnés sont présents dans la liste déroulante. 
    2. Sélectionnez un **sous-réseau** dans le réseau virtuel que vous avez sélectionné. 
    3. Sélectionnez **Suivant : Bouton Étiquettes >** en bas de la page. 

        ![Créer un point de terminaison privé - page Configuration](./media/private-link-service/create-private-endpoint-configuration-page.png)
10. Dans la page **Étiquettes**, créez les étiquettes (noms et valeurs) à associer à la ressource de point de terminaison privé. Ensuite, sélectionnez le bouton **Vérifier + créer** en bas de la page. 
11. Dans la page **Vérifier + créer**, examinez tous les paramètres et sélectionnez **Créer** pour créer le point de terminaison privé.
    
    ![Créer un point de terminaison privé - page Vérifier et créer](./media/private-link-service/create-private-endpoint-review-create-page.png)
12. Confirmez que le point de terminaison privé est créé. Si vous êtes le propriétaire de la ressource et que vous avez sélectionné l’option **Se connecter à une ressource Azure dans mon répertoire** pour la **méthode de connexion**, la connexion au point de terminaison doit être **approuvée automatiquement**. Si elle est à l’état **En attente**, consultez la section [Gérer des points de terminaison privés avec le Portail Azure](#manage-private-endpoints-using-azure-portal).

    ![Point de terminaison privé créé](./media/private-link-service/private-endpoint-created.png)

[!INCLUDE [service-bus-trusted-services](../../includes/service-bus-trusted-services.md)]

## <a name="add-a-private-endpoint-using-powershell"></a>Ajouter un point de terminaison privé avec PowerShell
L’exemple suivant montre comment utiliser Azure PowerShell pour créer une connexion de point de terminaison privé à un espace de noms Service Bus.

Votre point de terminaison privé et votre réseau virtuel doivent se trouver dans la même région. Votre espace de noms Service Bus peut se trouver dans une autre région. Et votre point de terminaison privé utilise une adresse IP privée dans votre réseau virtuel.

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

# create premium service bus namespace
$namespaceResource = New-AzResource -Location $namespaceLocation -ResourceName $namespaceName -ResourceGroupName $rgName -Sku @{name = "Premium"; capacity = 1} -Properties @{} -ResourceType "Microsoft.ServiceBus/namespaces" -

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

Quand vous créez un point de terminaison privé, la connexion doit être approuvée. Si la ressource pour laquelle vous créez un point de terminaison privé se trouve dans votre répertoire, vous pouvez approuver la requête de connexion à condition d’avoir les autorisations nécessaires. Si vous vous connectez à une ressource Azure dans un autre répertoire, vous devez attendre que le propriétaire de cette ressource approuve votre requête de connexion.

Il existe quatre états de provisionnement :

| Action de service | État du point de terminaison privé de l’utilisateur du service | Description |
|--|--|--|
| None | Pending | La connexion est créée manuellement et est en attente d’approbation du propriétaire de la ressource Private Link. |
| Approbation | Approved | La connexion a été approuvée automatiquement ou manuellement et est prête à être utilisée. |
| Rejeter | Rejeté | La connexion a été rejetée par le propriétaire de la ressource Private Link. |
| Supprimer | Déconnecté | La connexion a été supprimée par le propriétaire de la ressource Private Link, le point de terminaison privé devient donc informatif et doit être supprimé dans le cadre d’un nettoyage. |
 
###  <a name="approve-reject-or-remove-a-private-endpoint-connection"></a>Approuver, rejeter ou supprimer une connexion de point de terminaison privé

1. Connectez-vous au portail Azure.
1. Dans la barre de recherche, saisissez **Service Bus**.
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

    ![État de la connexion - approuvé](./media/private-link-service/connection-status-approved.png)

### <a name="reject-a-private-endpoint-connection"></a>Rejeter une connexion de point de terminaison privé

1. Si vous voulez rejeter une connexion de point de terminaison privé, qu’il s’agisse d’une demande en attente ou d’une connexion existante approuvée précédemment, sélectionnez la connexion de point de terminaison privé et cliquez sur le bouton **Rejeter**.

    ![Bouton Rejeter](./media/private-link-service/private-endpoint-reject.png)
2. Dans la page **Rejeter la connexion**, entrez éventuellement un commentaire et sélectionnez **Oui**. Si vous sélectionnez **Non**, il ne se passe rien. 

    ![Page Rejeter la connexion](./media/private-link-service/reject-connection-page.png)
3. Vous devez voir que l’état de la connexion dans la liste est remplacé par **Rejeté**. 

    ![Point de terminaison rejeté](./media/private-link-service/endpoint-rejected.png)


### <a name="remove-a-private-endpoint-connection"></a>Supprimer une connexion de point de terminaison privé

1. Pour supprimer une connexion de point de terminaison privé, sélectionnez-la dans la liste et sélectionnez **Supprimer** dans la barre d’outils. 

    ![Bouton Supprimer](./media/private-link-service/remove-endpoint.png)
2. Dans la page **Supprimer la connexion**, sélectionnez **Oui** pour confirmer la suppression du point de terminaison privé. Si vous sélectionnez **Non**, il ne se passe rien. 

    ![Page Supprimer la connexion](./media/private-link-service/delete-connection-page.png)
3. Vous devez voir que l’état a été remplacé par **Déconnecté**. Vous voyez ensuite que le point de terminaison n’est plus dans la liste. 

## <a name="validate-that-the-private-link-connection-works"></a>Vérifier le fonctionnement de la connexion à liaison privée

Vous devez vérifier que les ressources contenues dans le réseau virtuel du point de terminaison privé se connectent à votre espace de noms Service Bus via une adresse IP privée, et qu’elles sont intégrées à la zone DNS privée appropriée.

Commencez par créer une machine virtuelle en suivant les étapes décrites dans [Créer une machine virtuelle Windows sur le portail Azure](../virtual-machines/windows/quick-create-portal.md).

Sous l’onglet **Réseau** : 

1. Spécifiez un **réseau virtuel** et un **sous-réseau**. Vous devez sélectionner le réseau virtuel sur lequel vous avez déployé le point de terminaison privé.
2. Spécifiez une ressource d’**adresse IP publique**.
3. Pour **Groupe de sécurité réseau de la carte réseau**, sélectionnez **Aucun**.
4. Pour **Équilibrage de charge**, sélectionnez **Non**.

Connectez-vous à la machine virtuelle, ouvrez la ligne de commande et exécutez la commande suivante :

```console
nslookup <service-bus-namespace-name>.servicebus.windows.net
```

Vous devez obtenir un résultat similaire à la sortie suivante. 

```console
Non-authoritative answer:
Name:    <service-bus-namespace-name>.privatelink.servicebus.windows.net
Address:  10.0.0.4 (private IP address associated with the private endpoint)
Aliases:  <service-bus-namespace-name>.servicebus.windows.net
```

## <a name="limitations-and-design-considerations"></a>Limitations et remarques sur la conception

**Prix** : Pour plus d’informations sur les prix, consultez [Prix d’Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/).

**Limitations** :  Cette fonctionnalité est disponible dans toutes les régions publiques Azure.

**Nombre maximal de points de terminaison privés par espace de noms Service Bus** : 120.

Pour plus d’informations, consultez [Service Azure Private Link : Limitations](../private-link/private-link-service-overview.md#limitations)

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [Azure Private Link](../private-link/private-link-service-overview.md)
- En savoir plus sur [Azure Service Bus](service-bus-messaging-overview.md).
