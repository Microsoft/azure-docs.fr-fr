---
title: Configurer des points de terminaison privés pour les rubriques ou les domaines Azure Event Grid
description: Cet article explique comment configurer des points de terminaison privés pour les rubriques ou les domaines Azure Event Grid.
ms.topic: how-to
ms.date: 11/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: f8e0cfc0a850ae15ea6d03ff6ca8b90003adbfc9
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94916934"
---
# <a name="configure-private-endpoints-for-azure-event-grid-topics-or-domains"></a>Configurer des points de terminaison privés pour les rubriques ou les domaines Azure Event Grid
Vous pouvez utiliser des [points de terminaison privés](../private-link/private-endpoint-overview.md) pour autoriser l’entrée sécurisée d’événements directement à partir de votre réseau virtuel vers vos rubriques et domaines via une [liaison privée](../private-link/private-link-overview.md), sans passer par le réseau Internet public. Le point de terminaison privé utilise une adresse IP de l’espace d’adressage du réseau virtuel pour votre rubrique ou domaine. Pour plus d’informations conceptuelles, consultez [Sécurité du réseau](network-security.md).

Cet article explique comment configurer des points de terminaison privés pour les rubriques ou les domaines.

## <a name="use-azure-portal"></a>Utiliser le portail Azure 
Cette section vous montre comment utiliser le Portail Azure pour créer un point de terminaison privé pour une rubrique ou un domaine.

> [!NOTE]
> Les étapes présentées dans cette section concernent principalement les rubriques. Vous pouvez utiliser des étapes similaires pour créer des points de terminaison privés pour des **domaines**. 

1. Connectez-vous au [Portail Azure](https://portal.azure.com) et accédez à votre rubrique ou domaine.
2. Basculez vers l’onglet **Mise en réseau** de votre page de rubrique. Sélectionnez **+ Point de terminaison privé** dans la barre d’outils.

    ![Ajouter un point de terminaison privé](./media/configure-private-endpoints/add-button.png)
2. Dans la page **Informations de base**, suivez ces étapes : 
    1. Sélectionnez un **abonnement Azure** où créer le point de terminaison privé. 
    2. Sélectionnez un **groupe de ressources Azure** pour le point de terminaison privé. 
    3. Entrez un **nom** pour le point de terminaison. 
    4. Sélectionnez la **région** pour le point de terminaison. La région de votre point de terminaison privé doit être la même que celle de votre réseau virtuel, mais elle peut être différente de celle de la ressource de lien privé (en l’occurrence, une rubrique Event Grid). 
    5. Ensuite, sélectionnez **Suivant : Bouton Ressource >** en bas de la page. 

      ![Point de terminaison privé - page de base](./media/configure-private-endpoints/basics-page.png)
3. Dans la page **Ressource**, suivez ces étapes : 
    1. Pour la méthode de connexion, si vous sélectionnez **Se connecter à une ressource Azure dans mon répertoire**, suivez ces étapes. Cet exemple montre comment se connecter à une ressource Azure dans votre annuaire. 
        1. Sélectionnez l’**abonnement Azure** où se trouve votre **rubrique/domaine**. 
        1. Pour **Type de ressource**, sélectionnez **Microsoft.EventGrid/topics** ou **Microsoft.EventGrid/domains** pour le **Type de ressource**.
        2. Pour **Ressource**, sélectionnez une rubrique/un domaine dans la liste déroulante. 
        3. Vérifiez que la **Sous-ressource cible** est définie sur **rubrique** ou **domaine** (selon le type de ressource que vous avez sélectionné).    
        4. Sélectionnez **Suivant : Bouton Configuration >** en bas de la page. 

            ![Capture d’écran montrant la page « Créer un point de terminaison privé - Ressource »](./media/configure-private-endpoints/resource-page.png)
    2. Si vous sélectionnez **Se connecter à une ressource par alias ou ID de ressource**, suivez ces étapes :
        1. Entrez l’ID de la ressource. Par exemple : `/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>`.  
        2. Pour **Ressource**, entrez **rubrique** ou **domaine**. 
        3. (facultatif) Ajoutez un message de requête. 
        4. Sélectionnez **Suivant : Bouton Configuration >** en bas de la page. 

            ![Point de terminaison privé - page des ressources](./media/configure-private-endpoints/connect-azure-resource-id.png)
4. Dans la page **Configuration**, vous sélectionnez le sous-réseau dans un réseau virtuel sur lequel vous voulez déployer le point de terminaison privé. 
    1. Sélectionnez un **réseau virtuel**. Seuls les réseaux virtuels dans l’abonnement et l’emplacement sélectionnés sont présents dans la liste déroulante. 
    2. Sélectionnez un **sous-réseau** dans le réseau virtuel que vous avez sélectionné. 
    3. Sélectionnez **Suivant : Étiquettes >** au bas de la page. 

    ![Point de terminaison privé – page de configuration](./media/configure-private-endpoints/configuration-page.png)
5. Dans la page **Étiquettes**, créez les étiquettes (noms et valeurs) à associer à la ressource de point de terminaison privé. Ensuite, sélectionnez le bouton **Vérifier + créer** en bas de la page. 
6. Dans la page **Vérifier + créer**, examinez tous les paramètres et sélectionnez **Créer** pour créer le point de terminaison privé. 

    ![Point de terminaison privé - page vérifier et créer](./media/configure-private-endpoints/review-create-page.png)
    

### <a name="manage-private-link-connection"></a>Gérer une connexion à liaison privée

Quand vous créez un point de terminaison privé, la connexion doit être approuvée. Si la ressource pour laquelle vous créez un point de terminaison privé se trouve dans votre répertoire, vous pouvez approuver la requête de connexion à condition d’avoir les autorisations nécessaires. Si vous vous connectez à une ressource Azure dans un autre répertoire, vous devez attendre que le propriétaire de cette ressource approuve votre requête de connexion.

Il existe quatre états de provisionnement :

| Action de service | État du point de terminaison privé de l’utilisateur du service | Description |
|--|--|--|
| None | Pending | La connexion est créée manuellement et est en attente d’approbation du propriétaire de la ressource Private Link. |
| Approbation | Approved | La connexion a été approuvée automatiquement ou manuellement et est prête à être utilisée. |
| Rejeter | Rejeté | La connexion a été rejetée par le propriétaire de la ressource Private Link. |
| Supprimer | Déconnecté | La connexion a été supprimée par le propriétaire de la ressource Private Link, le point de terminaison privé devient donc informatif et doit être supprimé dans le cadre d’un nettoyage. |
 
###  <a name="how-to-manage-a-private-endpoint-connection"></a>Comment gérer une connexion de point de terminaison privé
Les sections suivantes vous montrent comment approuver ou rejeter une connexion de point de terminaison privée. 

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans la barre de recherche, tapez **Rubriques Event Grid** ou **Domaines Event Grid**.
1. Sélectionnez la **rubrique** ou le **domaine** que vous souhaitez gérer.
1. Sélectionnez l’onglet **Mise en réseau**.
1. Si une connexion est en attente, celle-ci présente l’état d’approvisionnement **En attente**. 

### <a name="to-approve-a-private-endpoint"></a>Pour approuver un point de terminaison privé
Vous pouvez approuver un point de terminaison privé qui se trouve dans l’état en attente. Pour approuver, procédez comme suit : 

> [!NOTE]
> Les étapes présentées dans cette section concernent principalement les rubriques. Vous pouvez utiliser des étapes similaires pour approuver des points de terminaison privés pour des **domaines**. 

1. Sélectionnez le **point de terminaison privé** que vous souhaitez approuver, puis sélectionnez **Approuver** dans la barre d’outils.

    ![Point de terminaison privé - état en attente](./media/configure-private-endpoints/pending.png)
1. Dans la boîte de dialogue **Approuver la connexion**, ajoutez un commentaire (facultatif) et sélectionnez **Oui**. 

    ![Point de terminaison privé – approuver](./media/configure-private-endpoints/approve.png)
1. Vérifiez que vous voyez l’état du point de terminaison en tant que **Approuvé**. 

    ![Point de terminaison privé – état approuvé](./media/configure-private-endpoints/approved-status.png)

### <a name="to-reject-a-private-endpoint"></a>Pour rejeter un point de terminaison privé
Vous pouvez rejeter un point de terminaison privé qui se trouve dans l’état en attente ou approuvé. Pour rejeter, procédez comme suit : 

> [!NOTE]
> Les étapes présentées dans cette section concernent les rubriques. Vous pouvez utiliser des étapes similaires pour rejeter des points de terminaison privés pour des **domaines**. 

1. Sélectionnez le **point de terminaison privé** que vous souhaitez rejeter, puis sélectionnez **Rejeter** dans la barre d’outils.

    ![Capture d'écran montrant « Réseau - Connexions des points de terminaison privés » avec l'option « Rejeter » sélectionnée](./media/configure-private-endpoints/reject-button.png)
1. Dans la boîte de dialogue **Rejeter la connexion**, ajoutez un commentaire (facultatif) et sélectionnez **Oui**. 

    ![Point de terminaison privé – rejeter](./media/configure-private-endpoints/reject.png)
1. Vérifiez que vous voyez l’état du point de terminaison en tant que **Rejeté**. 

    ![Point de terminaison privé – état rejeté](./media/configure-private-endpoints/rejected-status.png)

    > [!NOTE]
    > Vous ne pouvez pas approuver un point de terminaison privé dans le Portail Azure une fois qu’il est rejeté. 


## <a name="use-azure-cli"></a>Utiliser l’interface de ligne de commande Microsoft Azure
Pour créer un point de terminaison privé, utilisez la méthode [az network private-endpoint create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create), comme indiqué dans l’exemple suivant :

```azurecli-interactive
az network private-endpoint create \
    --resource-group <RESOURECE GROUP NAME> \
    --name <PRIVATE ENDPOINT NAME> \
    --vnet-name <VIRTUAL NETWORK NAME> \
    --subnet <SUBNET NAME> \
    --private-connection-resource-id "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME> \
    --connection-name <PRIVATE LINK SERVICE CONNECTION NAME> \
    --location <LOCATION> \
    --group-ids topic
```

Pour obtenir une description des paramètres utilisés dans l’exemple, consultez la documentation relative à [az network private-endpoint create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create). Voici quelques points à noter dans cet exemple : 

- Pour `private-connection-resource-id`, spécifiez l’ID de ressource de la **rubrique** ou du **domaine**. L’exemple précédent utilise le type : rubrique.
- pour `group-ids`, spécifiez `topic` ou `domain`. Dans l’exemple précédent, `topic` est utilisé. 

Pour supprimer un point de terminaison privé, utilisez la méthode [az network private-endpoint create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-delete), comme indiqué dans l’exemple suivant :

```azurecli-interactive
az network private-endpoint delete --resource-group <RESOURECE GROUP NAME> --name <PRIVATE ENDPOINT NAME>
```

> [!NOTE]
> Les étapes présentées dans cette section concernent les rubriques. Vous pouvez utiliser des étapes similaires pour créer des points de terminaison privés pour des **domaines**. 



### <a name="prerequisites"></a>Prérequis
Mettez à jour l’extension Azure Event Grid pour Azure CLI en exécutant la commande suivante : 

```azurecli-interactive
az extension update -n eventgrid
```

Si l’extension n’est pas installée, exécutez la commande suivante pour l’installer : 

```azurecli-interactive
az extension add -n eventgrid
```

### <a name="create-a-private-endpoint"></a>Créer un Private Endpoint
Pour créer un point de terminaison privé, utilisez la méthode [az network private-endpoint create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create), comme indiqué dans l’exemple suivant :

```azurecli-interactive
az network private-endpoint create \
    --resource-group <RESOURECE GROUP NAME> \
    --name <PRIVATE ENDPOINT NAME> \
    --vnet-name <VIRTUAL NETWORK NAME> \
    --subnet <SUBNET NAME> \
    --private-connection-resource-id "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME> \
    --connection-name <PRIVATE LINK SERVICE CONNECTION NAME> \
    --location <LOCATION> \
    --group-ids topic
```

Pour obtenir une description des paramètres utilisés dans l’exemple, consultez la documentation relative à [az network private-endpoint create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-create). Voici quelques points à noter dans cet exemple : 

- Pour `private-connection-resource-id`, spécifiez l’ID de ressource de la **rubrique** ou du **domaine**. L’exemple précédent utilise le type : rubrique.
- pour `group-ids`, spécifiez `topic` ou `domain`. Dans l’exemple précédent, `topic` est utilisé. 

Pour supprimer un point de terminaison privé, utilisez la méthode [az network private-endpoint create](/cli/azure/network/private-endpoint?#az-network-private-endpoint-delete), comme indiqué dans l’exemple suivant :

```azurecli-interactive
az network private-endpoint delete --resource-group <RESOURECE GROUP NAME> --name <PRIVATE ENDPOINT NAME>
```

> [!NOTE]
> Les étapes présentées dans cette section concernent les rubriques. Vous pouvez utiliser des étapes similaires pour créer des points de terminaison privés pour des **domaines**. 

#### <a name="sample-script"></a>Exemple de script
Voici un exemple de script qui crée les ressources Azure suivantes :

- Resource group
- Réseau virtuel
- Sous-réseau dans le réseau virtuel
- Rubrique Azure Event Grid
- Point de terminaison privé pour la rubrique

> [!NOTE]
> Les étapes présentées dans cette section concernent les rubriques. Vous pouvez utiliser des étapes similaires pour créer des points de terminaison privés pour des domaines.

```azurecli-interactive
subscriptionID="<AZURE SUBSCRIPTION ID>"
resourceGroupName="<RESOURCE GROUP NAME>"
location="<LOCATION>"
vNetName="<VIRTUAL NETWORK NAME>"
subNetName="<SUBNET NAME>"
topicName = "<TOPIC NAME>"
connectionName="<ENDPOINT CONNECTION NAME>"
endpointName=<ENDPOINT NAME>

# resource ID of the topic. replace <SUBSCRIPTION ID>, <RESOURCE GROUP NAME>, and <TOPIC NAME>
topicResourceID="/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME>"

# select subscription
az account set --subscription $subscriptionID

# create resource group
az group create --name $resourceGroupName --location $location

# create vnet 
az network vnet create \
    --resource-group $resourceGroupName \
    --name $vNetName \
    --address-prefix 10.0.0.0/16

# create subnet
az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $vNetName \
    --name $subNetName \
    --address-prefixes 10.0.0.0/24

# disable private endpoint network policies for the subnet
az network vnet subnet update \
    --resource-group $resourceGroupName \
    --vnet-name $vNetName \
    --name $subNetName \
    --disable-private-endpoint-network-policies true

# create event grid topic. update <LOCATION>
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location

# verify that the topic was created.
az eventgrid topic show \
    --resource-group $resourceGroupName \
    --name $topicName

# create private endpoint for the topic you created
az network private-endpoint create 
    --resource-group $resourceGroupName \
    --name $endpointName \
    --vnet-name $vNetName \
    --subnet $subNetName \
    --private-connection-resource-id $topicResourceID \
    --connection-name $connectionName \
    --location $location \
    --group-ids topic

# get topic 
az eventgrid topic show \
    --resource-group $resourceGroupName \
    --name $topicName

```

### <a name="approve-a-private-endpoint"></a>Approuver un point de terminaison privé
L’exemple d’extrait de code CLI suivant vous montre comment approuver une connexion de point de terminaison privé. 

```azurecli-interactive
az eventgrid topic private-endpoint-connection approve \
    --resource-group $resourceGroupName \
    --topic-name $topicName \
    --name  $endpointName \
    --description "connection approved"
```


### <a name="reject-a-private-endpoint"></a>Rejeter un point de terminaison privé
L’exemple d’extrait de code CLI suivant vous montre comment rejeter une connexion de point de terminaison privé. 

```azurecli-interactive
az eventgrid topic private-endpoint-connection reject \
    --resource-group $resourceGroupName \
    --topic-name $topicName \
    --name $endpointName \
    --description "Connection rejected"
```

### <a name="disable-public-network-access"></a>Désactiver l’accès au réseau public
Par défaut, l’accès au réseau public est activé pour une rubrique ou un domaine Event Grid. Pour autoriser l’accès via des points de terminaison privés uniquement, désactivez l’accès au réseau public en exécutant la commande suivante :  

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access disabled
```


## <a name="use-powershell"></a>Utiliser PowerShell
Cette section vous montre comment créer un point de terminaison privé pour une rubrique ou un domaine à l’aide de PowerShell. 

### <a name="prerequisite"></a>Configuration requise
Suivez les instructions de [Procédure : utiliser le portail pour créer une application Azure AD et un principal de service pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md) pour créer une application Azure Active Directory et notez les valeurs suivantes pour **ID de l'annuaire (locataire)** , **ID d’application (client)** et **Secret d’application (client)** . 

### <a name="prepare-token-and-headers-for-rest-api-calls"></a>Préparer un jeton et des en-têtes pour des appels d’API REST 
Exécutez les commandes préalables suivantes pour recevoir un jeton d’authentification à utiliser avec les appels d’API REST, ainsi que les informations d’en-tête et d’autorisation. 

```azurepowershell-interactive
$body = "grant_type=client_credentials&client_id=<CLIENT ID>&client_secret=<CLIENT SECRET>&resource=https://management.core.windows.net"

# get authentication token
$Token = Invoke-RestMethod -Method Post `
    -Uri https://login.microsoftonline.com/<TENANT ID>/oauth2/token  `
    -Body $body  `
    -ContentType 'application/x-www-form-urlencoded' 

# set authorization and content-type headers
$Headers = @{}
$Headers.Add("Authorization","$($Token.token_type) "+ " " + "$($Token.access_token)")
```

### <a name="create-a-subnet-with-endpoint-network-policies-disabled"></a>Créer un sous-réseau avec des stratégies de réseau de point de terminaison désactivées

```azurepowershell-interactive

# create resource group
New-AzResourceGroup -ResourceGroupName <RESOURCE GROUP NAME>  -Location <LOCATION>

# create virtual network
$virtualNetwork = New-AzVirtualNetwork `
                    -ResourceGroupName <RESOURCE GROUP NAME> `
                    -Location <LOCATION> `
                    -Name <VIRTUAL NETWORK NAME> `
                    -AddressPrefix 10.0.0.0/16

# create subnet with endpoint network policy disabled
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
                    -Name example-privatelinksubnet `
                    -AddressPrefix 10.0.0.0/24 `
                    -PrivateEndpointNetworkPoliciesFlag "Disabled" `
                    -VirtualNetwork $virtualNetwork

# update virtual network
$virtualNetwork | Set-AzVirtualNetwork
```

### <a name="create-an-event-grid-topic-with-a-private-endpoint"></a>Créer une rubrique Event Grid avec un point de terminaison privé

> [!NOTE]
> Les étapes présentées dans cette section concernent les rubriques. Vous pouvez utiliser des étapes similaires pour créer des points de terminaison privés pour des **domaines**. 


```azurepowershell-interactive
$body = @{"location"="<LOCATION>"; "properties"=@{"publicNetworkAccess"="disabled"}} | ConvertTo-Json

# create topic
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01"  `
    -Headers $Headers  `
    -Body $body

# verify that the topic was created
$topic=Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01"   `
    -Headers $Headers  

# create private link service connection
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
                                -Name "<PRIVATE LINK SERVICE CONNECTION NAME>" `
                                -PrivateLinkServiceId $topic.Id `
                                -GroupId "topic"

# get virtual network info 
$virtualNetwork = Get-AzVirtualNetwork  `
                    -ResourceGroupName  <RESOURCE GROUP NAME> `
                    -Name <VIRTUAL NETWORK NAME>

# get subnet info
$subnet = $virtualNetwork | Select -ExpandProperty subnets `
                             | Where-Object  {$_.Name -eq <SUBNET NAME>}  

# now, you are ready to create a private endpoint 
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName <RESOURCE GROUP NAME>  `
                                        -Name <PRIVATE ENDPOINT NAME>   `
                                        -Location <LOCATION> `
                                        -Subnet  $subnet   `
                                        -PrivateLinkServiceConnection $privateEndpointConnection

# verify that the endpoint was created
Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections?api-version=2020-06-01"   `
    -Headers $Headers   `
    | ConvertTo-Json -Depth 5

```

Lorsque vous vérifiez que le point de terminaison a été créé, le résultat doit ressembler à ce qui suit :

```json

{
  "value": [
    {
      "properties": {
        "privateEndpoint": {
          "id": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.Network/privateEndpoints/<PRIVATE ENDPOINT NAME>"
        },
        "groupIds": [
          "topic"
        ],
        "privateLinkServiceConnectionState": {
          "status": "Approved",
          "description": "Auto-approved",
          "actionsRequired": "None"
        },
        "provisioningState": "Succeeded"
      },
      "id": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>",
      "name": "myConnection",
      "type": "Microsoft.EventGrid/topics/privateEndpointConnections"
    }
  ]
}
```

### <a name="approve-a-private-endpoint-connection"></a>Approuver une connexion de point de terminaison privé
L’exemple d’extrait de code PowerShell suivant vous montre comment approuver un point de terminaison privé. 

> [!NOTE]
> Les étapes présentées dans cette section concernent les rubriques. Vous pouvez utiliser des étapes similaires pour approuver des points de terminaison privés pour des **domaines**. 

```azurepowershell-interactive
$approvedBody = @{"properties"=@{"privateLinkServiceConnectionState"=@{"status"="approved";"description"="connection approved";"actionsRequired"="none"}}} | ConvertTo-Json

# approve endpoint connection
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AzuRE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01"  `
    -Headers $Headers  `
    -Body $approvedBody

# confirm that the endpoint connection was approved
Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01"  `
    -Headers $Headers

```

### <a name="reject-a-private-endpoint-connection"></a>Rejeter une connexion de point de terminaison privé
L’exemple suivant montre comment rejeter un point de terminaison privé à l’aide de PowerShell. Vous pouvez obtenir le GUID du point de terminaison privé à partir du résultat de la commande GET précédente. 

> [!NOTE]
> Les étapes présentées dans cette section concernent les rubriques. Vous pouvez utiliser des étapes similaires pour rejeter des points de terminaison privés pour des **domaines**. 


```azurepowershell-interactive
$rejectedBody = @{"properties"=@{"privateLinkServiceConnectionState"=@{"status"="rejected";"description"="connection rejected";"actionsRequired"="none"}}} | ConvertTo-Json

# reject private endpoint
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01"  `
    -Headers $Headers  `
    -Body $rejectedBody

# confirm that endpoint was rejected
Invoke-RestMethod -Method 'Get' 
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-06-01" ` 
    -Headers $Headers
```

Vous pouvez approuver la connexion même après qu’elle a été rejetée via l’API. Si vous utilisez le Portail Azure, vous ne pouvez pas approuver un point de terminaison qui a été rejeté. 

## <a name="next-steps"></a>Étapes suivantes
* Pour en savoir plus sur la configuration des paramètres de pare-feu IP, consultez [Configurer le pare-feu IP pour les rubriques ou les domaines Azure Event Grid](configure-firewall.md).
* Pour résoudre les problèmes de connectivité réseau, consultez [Résoudre les problèmes de connectivité réseau](troubleshoot-network-connectivity.md).
