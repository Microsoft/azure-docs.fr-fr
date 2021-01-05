---
title: Remise d’événement, identité de service géré et liaison privée
description: Cet article explique comment activer une identité de service managé pour une rubrique Azure Event Grid. Utilisez-la pour transférer des événements vers des destinations prises en charge.
ms.topic: how-to
ms.date: 10/22/2020
ms.openlocfilehash: edb3e5ac8257a29ecd3835e1dfd4c116c3cc7164
ms.sourcegitcommit: 1bdcaca5978c3a4929cccbc8dc42fc0c93ca7b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/13/2020
ms.locfileid: "97368611"
---
# <a name="event-delivery-with-a-managed-identity"></a>Remise d’événement avec une identité managée
Cet article explique comment activer une [identité de service managé](../active-directory/managed-identities-azure-resources/overview.md) pour des rubriques ou des domaines Azure Event Grid. Utilisez-le pour transférer des événements vers des destinations prises en charge, telles que des files d’attente et rubriques Service Bus, des concentrateurs d’événements et des comptes de stockage.

Les étapes décrites en détail dans cet article sont les suivantes :
1. Créer une rubrique ou un domaine avec une identité affectée par le système, ou mettre à jour une rubrique ou un domaine existants pour activer une identité. 
1. Ajouter l’identité à un rôle approprié (par exemple, expéditeur de données Service Bus) sur la destination (par exemple, une file d’attente Service Bus).
1. Lors de la création d’abonnements à des événements, activer l’utilisation de l’identité pour remettre des événements à la destination. 

> [!NOTE]
> Actuellement, il n’est pas possible de remettre des événements à l’aide de [points de terminaison privés](../private-link/private-endpoint-overview.md). Pour plus d’informations, consultez la section [Points de terminaison privés](#private-endpoints) à la fin de cet article. 

## <a name="create-a-topic-or-domain-with-an-identity"></a>Créer une rubrique ou un domaine avec une identité
Tout d’abord, voyons comment créer une rubrique ou un domaine avec une identité managée par le système.

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure
Il est possible d’activer une identité affectée par le système pour une rubrique ou un domaine lors de sa création sur le Portail Azure. L’image suivante montre comment activer une identité managée par le système pour une rubrique. Fondamentalement, dans la page **Avancée** de l’Assistant Création de rubrique, vous sélectionnez l’option **Activer l’identité attribuée par le système**. Cette option apparaît également sur la page **Avancé** de l’Assistant Création de domaine. 

![Activer une identité lors de la création d’une rubrique](./media/managed-service-identity/create-topic-identity.png)

### <a name="use-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Microsoft Azure
Il est également possible d’utiliser Azure CLI pour créer une rubrique ou un domaine avec une identité affectée par le système. Utilisez la commande `az eventgrid topic create` avec le paramètre `--identity` défini sur `systemassigned`. Si vous ne spécifiez pas de valeur pour ce paramètre, la valeur par défaut `noidentity` est utilisée. 

```azurecli-interactive
# create a topic with a system-assigned identity
az eventgrid topic create -g <RESOURCE GROUP NAME> --name <TOPIC NAME> -l <LOCATION>  --identity systemassigned
```

Vous pouvez également utiliser la commande `az eventgrid domain create` pour créer un domaine avec une identité managée par le système.

## <a name="enable-an-identity-for-an-existing-topic-or-domain"></a>Activer une identité pour une rubrique ou un domaine existant
Dans la section précédente, vous avez appris à activer une identité managée par le système lors de la création d’une rubrique ou d’un domaine. Dans cette section, vous allez apprendre à activer une identité managée par le système pour une rubrique ou un domaine existant. 

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure
La procédure suivante vous montre comment activer une identité managée par le système pour une rubrique. Les étapes d’activation d’une identité pour un domaine sont similaires. 

1. Accédez au [portail Azure](https://portal.azure.com).
2. Dans la barre de recherche en haut, recherchez **Rubriques Event Grid**.
3. Sélectionnez la **rubrique** pour laquelle vous souhaitez activer l’identité managée. 
4. Basculez vers l’onglet **Identité**. 
5. **Activez** le commutateur pour activer l’identité. 
1. Sélectionnez **Enregistrer** dans la barre d’outils pour enregistrer le paramètre. 

    :::image type="content" source="./media/managed-service-identity/identity-existing-topic.png" alt-text="Page identité pour une rubrique"::: 

La procédure permettant d’activer une identité pour un domaine Event Grid est similaire.

### <a name="use-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Microsoft Azure
Utilisez la commande `az eventgrid topic update` avec `--identity` défini sur `systemassigned` pour activer une identité attribuée par le système pour une rubrique existante. Si vous souhaitez désactiver l’identité, spécifiez la valeur `noidentity`. 

```azurecli-interactive
# Update the topic to assign a system-assigned identity. 
az eventgrid topic update -g $rg --name $topicname --identity systemassigned --sku basic 
```

La commande de mise à jour d’un domaine existant est similaire (`az eventgrid domain update`).

## <a name="supported-destinations-and-azure-roles"></a>Destinations et rôles Azure pris en charge
Une fois que vous avez activé l’identité pour votre rubrique ou votre domaine Event Grid, Azure crée automatiquement une identité dans Azure Active Directory. Ajoutez cette identité aux rôles Azure appropriés afin que la rubrique ou le domaine puisse transférer des événements vers les destinations prises en charge. Par exemple, ajoutez l’identité au rôle **Expéditeur de données Azure Event Hubs** pour un espace de noms Azure Event Hubs pour que la rubrique Event Grid puisse transférer les événements aux hubs d’événements de cet espace de noms. 

Actuellement, Azure Event Grid prend en charge les rubriques et les domaines configurés avec une identité managée affectée par le système pour transférer les événements vers les destinations suivantes. Ce tableau indique également les rôles à associer à l’identité pour que la rubrique puisse transférer les événements.

| Destination | Rôle Azure | 
| ----------- | --------- | 
| Files d’attente et rubriques Service Bus | [Expéditeur de données Azure Service Bus](../service-bus-messaging/authenticate-application.md#azure-built-in-roles-for-azure-service-bus) |
| Hubs d'événements Azure | [Expéditeur de données Azure Event Hubs](../event-hubs/authorize-access-azure-active-directory.md#azure-built-in-roles-for-azure-event-hubs) | 
| Stockage Blob Azure | [Contributeur aux données Blob du stockage](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues) |
| Stockage File d’attente Azure |[Expéditeur de messages de données en file d’attente du stockage](../storage/common/storage-auth-aad-rbac-portal.md#azure-roles-for-blobs-and-queues) | 

## <a name="add-an-identity-to-azure-roles-on-destinations"></a>Ajouter une identité à des rôles Azure sur des destinations
Cette section décrit comment ajouter l’identité pour votre rubrique ou domaine à un rôle Azure. 

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure
Vous pouvez utiliser le Portail Azure pour affecter à l’identité de la rubrique ou du domaine un rôle approprié afin que la rubrique ou le domaine puisse transférer les événements vers la destination. 

L’exemple suivant ajoute une identité managée pour une rubrique Event Grid nommée **msitesttopic** au rôle **Expéditeur de données Azure Service Bus** pour un espace de noms Service Bus contenant une ressource de file d’attente ou de rubrique. Lorsque l’ajout au rôle est effectué au niveau de l’espace de noms, la rubrique peut transférer les événements à toutes les entités de l’espace de noms. 

1. Accédez à votre **espace de noms Service Bus** sur le [Portail Azure](https://portal.azure.com). 
1. Sélectionnez **Contrôle d’accès** dans le volet gauche. 
1. Dans la section **Ajouter une attribution de rôle**, sélectionnez **Ajouter**. 
1. Dans la page **Ajouter une attribution de rôle**, effectuez les étapes suivantes :
    1. Sélectionnez le rôle. Dans ce cas, il s’agit du rôle **Expéditeur de données Azure Service Bus**. 
    1. Sélectionnez l’**identité** pour votre rubrique ou domaine. 
    1. Sélectionnez **Enregistrer** pour enregistrer la configuration.

Les étapes sont similaires pour l’ajout d’une identité à d’autres rôles mentionnés dans le tableau. 

### <a name="use-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Microsoft Azure
L’exemple présenté dans cette section montre comment utiliser Azure CLI pour ajouter une identité à un rôle Azure. Les exemples de commandes sont destinés aux rubriques Event Grid. Les commandes pour les domaines Event Grid sont similaires. 

#### <a name="get-the-principal-id-for-the-topics-system-identity"></a>Obtenir l’ID du principal pour l’identité système de la rubrique 
Tout d’abord, récupérez l’ID du principal de l’identité managée par le système de la rubrique et attribuez l’identité aux rôles appropriés.

```azurecli-interactive
topic_pid=$(az ad sp list --display-name "$<TOPIC NAME>" --query [].objectId -o tsv)
```

#### <a name="create-a-role-assignment-for-event-hubs-at-various-scopes"></a>Créer une attribution de rôle pour Event hubs à différentes étendues 
L’exemple d’interface CLI suivant montre comment ajouter l’identité d’une rubrique au rôle **Expéditeur de données Azure Event Hubs** au niveau de l’espace de noms ou du concentrateur d’événements. Si l’attribution de rôle est créée au niveau de l’espace de noms, la rubrique peut transférer les événements à tous les hubs d’événements de cet espace de noms. Si l’attribution de rôle est créée au niveau du hub d’événements, la rubrique ne peut transférer les événements qu’à ce hub d’événements précis. 


```azurecli-interactive
role="Azure Event Hubs Data Sender" 
namespaceresourceid=$(az eventhubs namespace show -n $<EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 
eventhubresourceid=$(az eventhubs eventhub show -n <EVENT HUB NAME> --namespace-name <EVENT HUBS NAMESPACE NAME> -g <RESOURCE GROUP of EVENT HUB> --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one event hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$eventhubresourceid" 
```

#### <a name="create-a-role-assignment-for-a-service-bus-topic-at-various-scopes"></a>Créer une attribution de rôle pour une rubrique Service Bus dans différentes étendues 
L’exemple d’interface CLI suivant montre comment ajouter une identité de rubrique au rôle **Expéditeur de données Azure Service Bus** au niveau de l’espace de noms ou de la rubrique Service Bus. Si l’attribution de rôle est créée au niveau de l’espace de noms, la rubrique Event Grid peut transférer les événements à toutes les entités (files d’attente et rubriques Service Bus) de cet espace de noms. Si l’attribution de rôle est créée au niveau de la file d’attente ou de la rubrique Service Bus, la rubrique Event Grid ne peut transférer les événements qu’à cette file d’attente ou à cette rubrique Service Bus précise. 

```azurecli-interactive
role="Azure Service Bus Data Sender" 
namespaceresourceid=$(az servicebus namespace show -n $RG\SB -g "$RG" --query "{I:id}" -o tsv 
sbustopicresourceid=$(az servicebus topic show -n topic1 --namespace-name $RG\SB -g "$RG" --query "{I:id}" -o tsv) 

# create role assignment for the whole namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$namespaceresourceid" 

# create role assignment scoped to just one hub inside the namespace 
az role assignment create --role "$role" --assignee "$topic_pid" --scope "$sbustopicresourceid" 
```

## <a name="create-event-subscriptions-that-use-an-identity"></a>Créer des abonnements à des événements qui utilisent une identité
Maintenant que vous disposez d’une rubrique ou d’un domaine avec une identité affectée par le système et que vous avez ajouté l’identité au rôle approprié sur la destination, vous pouvez créer des abonnements qui utilisent l’identité. 

### <a name="use-the-azure-portal"></a>Utilisation du portail Azure
Lors de la création d’un abonnement à des événements, une option permet d’activer l’utilisation d’une identité affectée par le système pour un point de terminaison dans la section **DÉTAILS DES POINTS DE TERMINAISON**. 

![Activer une identité lors de la création d’un abonnement à des événements pour une file d’attente Service Bus](./media/managed-service-identity/service-bus-queue-subscription-identity.png)

Vous pouvez également activer l’utilisation d’une identité affectée par le système pour la mise en file d’attente de lettres mortes sous l’onglet **Fonctionnalités supplémentaires**. 

![Activer une identité attribuée par le système pour la mise en file d'attente de lettres mortes](./media/managed-service-identity/enable-deadletter-identity.png)

### <a name="use-the-azure-cli---service-bus-queue"></a>Utilisation d’Azure CLI – File d’attente Service Bus 
Cette section explique comment utiliser Azure CLI afin d’activer l’utilisation d’une identité affectée par le système pour remettre des événements à une file d’attente Service Bus. L’identité doit être un membre du rôle **Expéditeur de données Azure Service Bus**. Elle doit également être membre du rôle **Contributeur aux données Blob du stockage** sur le compte de stockage utilisé pour la mise en file d'attente de lettres mortes. 

#### <a name="define-variables"></a>Définir des variables
Tout d’abord, spécifiez des valeurs pour les variables suivantes à utiliser dans la commande CLI. 

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

# get the service bus queue resource id
queueid=$(az servicebus queue show --namespace-name <SERVICE BUS NAMESPACE NAME> --name <QUEUE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
sb_esname = "<Specify a name for the event subscription>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Créer un abonnement à des événements en utilisant une identité managée pour la remise 
Cet exemple de commande crée un abonnement à des événements pour une rubrique Event Grid dont le type de point de terminaison est défini sur **File d’attente Service Bus**. 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sb_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery-and-dead-lettering"></a>Créer un abonnement à des événements en utilisant une identité managée pour la remise et la mise en file d’attente de lettres mortes
Cet exemple de commande crée un abonnement à des événements pour une rubrique Event Grid dont le type de point de terminaison est défini sur **File d’attente Service Bus**. Il spécifie également que l’identité managée par le système doit être utilisée pour la mise en file d’attente de lettres mortes. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sb_esnameq 
```

### <a name="use-the-azure-cli---event-hubs"></a>Utilisation d’Azure CLI – Event Hubs 
Cette section explique comment utiliser Azure CLI afin d’activer l’utilisation d’une identité affectée par le système pour remettre des événements à un hub d’événements. L’identité doit être un membre du rôle **Expéditeur de données Azure Event Hubs**. Elle doit également être membre du rôle **Contributeur aux données Blob du stockage** sur le compte de stockage utilisé pour la mise en file d'attente de lettres mortes. 

#### <a name="define-variables"></a>Définir des variables
```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

hubid=$(az eventhubs eventhub show --name <EVENT HUB NAME> --namespace-name <NAMESPACE NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)
eh_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Créer un abonnement à des événements en utilisant une identité managée pour la remise 
Cet exemple de commande crée un abonnement à des événements pour une rubrique Event Grid dont le type de point de terminaison est défini sur **Event Hubs**. 

```azurecli-interactive
az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type eventhub 
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    -n $sbq_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Créer un abonnement à des événements en utilisant une identité managée pour la remise et la mise en file d’attente de lettres mortes 
Cet exemple de commande crée un abonnement à des événements pour une rubrique Event Grid dont le type de point de terminaison est défini sur **Event Hubs**. Il spécifie également que l’identité managée par le système doit être utilisée pour la mise en file d’attente de lettres mortes. 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type servicebusqueue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $hubid
    --deadletter-identity-endpoint $eh_deadletterendpoint
    --deadletter-identity systemassigned 
    -n $eh_esname 
```

### <a name="use-the-azure-cli---azure-storage-queue"></a>Utilisation d’Azure CLI – File d’attente de stockage Azure 
Cette section explique comment utiliser Azure CLI afin d’activer l’utilisation d’une identité affectée par le système pour remettre des événements à une file d’attente de Stockage Azure. L’identité doit être membre du rôle **Contributeur aux données Blob du stockage** sur le compte de stockage.

#### <a name="define-variables"></a>Définir des variables  

```azurecli-interactive
subid="<AZURE SUBSCRIPTION ID>"
rg = "<RESOURCE GROUP of EVENT GRID TOPIC>"
topicname = "<EVENT GRID TOPIC NAME>"

# get the storage account resource id
storageid=$(az storage account show --name <STORAGE ACCOUNT NAME> --resource-group <RESOURCE GROUP NAME> --query id --output tsv)

# build the resource id for the queue
queueid="$storageid/queueservices/default/queues/<QUEUE NAME>" 

sa_esname = "<SPECIFY EVENT SUBSCRIPTION NAME>" 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery"></a>Créer un abonnement à des événements en utilisant une identité managée pour la remise 

```azurecli-interactive
az eventgrid event-subscription create 
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    -n $sa_esname 
```

#### <a name="create-an-event-subscription-by-using-a-managed-identity-for-delivery--deadletter"></a>Créer un abonnement à des événements en utilisant une identité managée pour la remise et la mise en file d’attente de lettres mortes 

```azurecli-interactive
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)
deadletterendpoint="$storageid/blobServices/default/containers/<BLOB CONTAINER NAME>"

az eventgrid event-subscription create  
    --source-resource-id /subscriptions/$subid/resourceGroups/$rg/providers/Microsoft.EventGrid/topics/$topicname 
    --delivery-identity-endpoint-type storagequeue  
    --delivery-identity systemassigned 
    --delivery-identity-endpoint $queueid
    --deadletter-identity-endpoint $deadletterendpoint 
    --deadletter-identity systemassigned 
    -n $sa_esname 
```

## <a name="private-endpoints"></a>Instances Private Endpoint
Actuellement, il n’est pas possible de remettre des événements à l’aide de [points de terminaison privés](../private-link/private-endpoint-overview.md). Autrement dit, il n’y a pas de prise en charge si vous avez des exigences strictes en matière d’isolement réseau lorsque le trafic des événements remis ne doit pas sortir de l’espace IP privé. 

Toutefois, si vos exigences comprennent une méthode sécurisée pour envoyer des événements à l’aide d’un canal chiffré et une identité connue de l’expéditeur (dans ce cas, Event Grid) à l’aide d’un espace IP public, vous pouvez remettre des événements à Event Hubs, Service Bus ou au service Stockage Azure à l’aide d’une rubrique Azure Event Grid ou d’un domaine avec une identité gérée par le système comme présenté dans cet article. Ensuite, vous pouvez utiliser une liaison privée configurée dans Azure Functions ou votre webhook déployé sur votre réseau virtuel pour extraire des événements. Voir l’exemple : [Connectez-vous à des points de terminaison privés avec Azure Functions](/samples/azure-samples/azure-functions-private-endpoints/connect-to-private-endpoints-with-azure-functions/).

Notez que, dans cette configuration, le trafic passe par l’adresse IP publique/Internet d’Event Grid vers Event Hubs, Service Bus ou le stockage Azure, mais que le canal peut être chiffré et qu’une identité gérée Event Grid est utilisée. Si vous configurez votre Azure Functions ou webhook déployé sur votre réseau virtuel pour utiliser un stockage Event Hubs, Service Bus ou stockage Azure via une liaison privée, cette section du trafic restera évidemment dans Azure.


## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur les identités de service managé, consultez [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md) 
