---
title: Stratégies de lettres mortes et de nouvelles tentatives - Azure Event Grid
description: Explique comment personnaliser les options de remise des événements pour Event Grid. Définissez une destination de lettres mortes, et spécifiez la durée des nouvelles tentatives de remise.
ms.topic: conceptual
ms.date: 07/20/2020
ms.openlocfilehash: 7d8cd74ccfb77bcec45d06071a4f46fb2a640cf8
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92460935"
---
# <a name="set-dead-letter-location-and-retry-policy"></a>Définir un emplacement de lettres mortes et une stratégie de nouvelles tentatives

Quand vous créez un abonnement aux événements, vous pouvez personnaliser les paramètres de remise des événements. Cet article vous montre comment configurer un emplacement de lettres mortes et personnaliser les paramètres de nouvelle tentative. Pour plus d’informations sur ces fonctionnalités, voir [Remise et nouvelle tentative de remise de messages Event Grid](delivery-and-retry.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

> [!NOTE]
> Pour en savoir plus sur la transmission des messages, les nouvelles tentatives et les lettres mortes, consultez l’article conceptuel : [Distribution et nouvelle tentative de distribution de messages avec Azure Event Grid](delivery-and-retry.md).

## <a name="set-dead-letter-location"></a>Définir l’emplacement des lettres mortes

Pour définir un emplacement de lettres mortes, vous avez besoin d’un compte de stockage. Ce dernier sert à contenir les événements qui ne peuvent pas être remis à un point de terminaison. Les exemples reçoivent l'ID de ressource d'un compte de stockage existant. Ils créent un abonnement aux événements qui utilise un conteneur de ce compte de stockage pour le point de terminaison des lettres mortes.

> [!NOTE]
> - Créez un compte de stockage et un conteneur d’objets blob dans le stockage avant d’exécuter les commandes de cet article.
> - Le service Event Grid crée des objets blob dans ce conteneur. Les noms des objets blob comportent le nom de l’abonnement Event Grid avec toutes les lettres en majuscules. Par exemple, si le nom de l’abonnement est My-Blob-Subscription, les noms des objets BLOB de lettres mortes comportent MY-BLOB-SUBSCRIPTION (myblobcontainer/MY-BLOB-SUBSCRIPTION/2019/8/8/5/111111111-1111-1111-1111-111111111111.json). Ce comportement assure une protection contre les différences de gestion de la casse entre les services Azure.


### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
containername=testcontainer

topicid=$(az eventgrid topic show --name demoTopic -g gridResourceGroup --query id --output tsv)
storageid=$(az storage account show --name demoStorage --resource-group gridResourceGroup --query id --output tsv)

az eventgrid event-subscription create \
  --source-resource-id $topicid \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --deadletter-endpoint $storageid/blobServices/default/containers/$containername
```

Pour désactiver les lettres mortes, réexécutez la commande pour créer l’abonnement aux événements, mais ne fournissez pas de valeur pour `deadletter-endpoint`. Vous n’avez pas besoin de supprimer l’abonnement aux événements.

> [!NOTE]
> Si vous utilisez Azure CLI sur votre ordinateur local, utilisez Azure CLI version 2.0.56 ou supérieure. Pour des instructions sur l’installation de la version d’Azure CLI la plus récente, consultez [Installer l’interface Azure CLI](/cli/azure/install-azure-cli).

### <a name="powershell"></a>PowerShell

```azurepowershell-interactive
$containername = "testcontainer"

$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name demoTopic).Id
$storageid = (Get-AzStorageAccount -ResourceGroupName gridResourceGroup -Name demostorage).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -DeadLetterEndpoint "$storageid/blobServices/default/containers/$containername"
```

Pour désactiver les lettres mortes, réexécutez la commande pour créer l’abonnement aux événements, mais ne fournissez pas de valeur pour `DeadLetterEndpoint`. Vous n’avez pas besoin de supprimer l’abonnement aux événements.

> [!NOTE]
> Si vous utilisez Azure PowerShell sur votre ordinateur local, utilisez Azure PowerShell version 1.1.0 ou supérieure. Téléchargez et installez la version la plus récente d’Azure PowerShell à partir des [téléchargements Azure](https://azure.microsoft.com/downloads/).

## <a name="set-retry-policy"></a>Définir une stratégie de nouvelle tentative

Au moment de créer un abonnement Event Grid, vous pouvez définir des valeurs fixant la durée pendant laquelle Event Grid doit tenter de remettre l’événement. Par défaut, Event Grid essaie pendant 24 heures (1 440 minutes) ou 30 fois. Vous pouvez définir chacune des ces valeurs pour votre abonnement Event Grid. La valeur de durée de vie d’un événement doit être un entier de 1 à 1440. La valeur du nombre maximum de nouvelles tentatives doit être un entier compris entre 1 et 30.

Vous ne pouvez pas configurer de [nouvelle tentative de planification](delivery-and-retry.md#retry-schedule-and-duration).

### <a name="azure-cli"></a>Azure CLI

Pour attribuer aux événements une durée de vie différente de 1 440 minutes, utilisez :

```azurecli-interactive
az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --event-ttl 720
```

Pour définir un nombre maximum de nouvelles tentatives différent de 30, utilisez :

```azurecli-interactive
az eventgrid event-subscription create \
  -g gridResourceGroup \
  --topic-name <topic_name> \
  --name <event_subscription_name> \
  --endpoint <endpoint_URL> \
  --max-delivery-attempts 18
```

> [!NOTE]
> Si vous définissez `event-ttl` et `max-deliver-attempts`, Event Grid utilise la date de la première expiration pour déterminer quand arrêter la remise des événements. Par exemple, si vous définissez une durée de vie (TTL) de 30 minutes et 10 tentatives de remise maximum. Lorsqu’un événement n’est pas remis après 30 minutes (ou) n’est pas remis après 10 tentatives, selon la première éventualité, l’événement devient lettres mortes.  

### <a name="powershell"></a>PowerShell

Pour attribuer aux événements une durée de vie différente de 1 440 minutes, utilisez :

```azurepowershell-interactive
$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name demoTopic).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -EventTtl 720
```

Pour définir un nombre maximum de nouvelles tentatives différent de 30, utilisez :

```azurepowershell-interactive
$topicid = (Get-AzEventGridTopic -ResourceGroupName gridResourceGroup -Name demoTopic).Id

New-AzEventGridSubscription `
  -ResourceId $topicid `
  -EventSubscriptionName <event_subscription_name> `
  -Endpoint <endpoint_URL> `
  -MaxDeliveryAttempt 18
```

> [!NOTE]
> Si vous définissez `event-ttl` et `max-deliver-attempts`, Event Grid utilise la date de la première expiration pour déterminer quand arrêter la remise des événements. Par exemple, si vous définissez une durée de vie (TTL) de 30 minutes et 10 tentatives de remise maximum. Lorsqu’un événement n’est pas remis après 30 minutes (ou) n’est pas remis après 10 tentatives, selon la première éventualité, l’événement devient lettres mortes.  

## <a name="next-steps"></a>Étapes suivantes

* Pour un exemple d’application utilisant une application de Azure Function pour traiter des événements de lettres mortes, voir [Exemples de lettres mortes Azure Event Grid pour .NET](https://azure.microsoft.com/resources/samples/event-grid-dotnet-handle-deadlettered-events/).
* Pour plus d’informations sur la remise d’événements et sur les nouvelles tentatives, consultez [Remise et nouvelle tentative de remise de messages avec Azure Grid](delivery-and-retry.md).
* Pour une présentation d’Event Grid, consultez [À propos d’Event Grid](overview.md).
* Pour une prise en main rapide d’Event Grid, consultez [Créer et acheminer des événements personnalisés avec Azure Event Grid](custom-event-quickstart.md).
