---
title: Utiliser Azure CLI pour créer des rubriques et des abonnements Service Bus
description: Dans ce guide de démarrage rapide, vous allez apprendre à créer une rubrique Service Bus et des abonnements à cette rubrique à l’aide d’Azure CLI.
ms.date: 06/23/2020
ms.topic: quickstart
author: spelluru
ms.author: spelluru
ms.openlocfilehash: 080b089efa276779420f6d9bc8e76272f1e65788
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90069694"
---
# <a name="use-azure-cli-to-create-a-service-bus-topic-and-subscriptions-to-the-topic"></a>Utiliser Azure CLI pour créer une rubrique Service Bus et des abonnements à cette rubrique
Dans ce guide de démarrage rapide, vous allez utiliser Azure CLI pour créer une rubrique Service Bus, puis des abonnements à cette rubrique. 

## <a name="what-are-service-bus-topics-and-subscriptions"></a>Présentation des rubriques et des abonnements Service Bus
Les rubriques et les abonnements Service Bus prennent en charge un modèle de communication de messagerie *de publication et d'abonnement* . Lors de l’utilisation de rubriques et d’abonnements, les composants d’une application distribuée ne communiquent pas directement entre eux ; ils échangent plutôt des messages via une rubrique, qui fait office d’intermédiaire.

![TopicConcepts](./media/service-bus-java-how-to-use-topics-subscriptions/sb-topics-01.png)

Contrairement aux files d’attente Service Bus, où chaque message est traité par un seul consommateur, les rubriques et les abonnements fournissent une forme de communication « un-à-plusieurs », à l’aide d’un modèle de publication et d’abonnement. Il est possible d’inscrire plusieurs abonnements à une rubrique. Lorsqu’un message est envoyé à une rubrique, il est alors mis à disposition de chaque abonnement pour être géré ou traité indépendamment. Un abonnement à une rubrique ressemble à une file d’attente virtuelle qui reçoit des copies des messages envoyés à la rubrique. Vous pouvez éventuellement inscrire des règles de filtre pour une rubrique par abonnement, ce qui vous permet de filtrer ou de restreindre les messages d’une rubrique reçus en fonction des abonnements à une rubrique.

Les rubriques et les abonnements Service Bus vous permettent de mettre votre infrastructure à l’échelle pour traiter de nombreux messages parmi un grand nombre d’utilisateurs et d’applications.

## <a name="prerequisites"></a>Prérequis
Si vous n'avez pas d'abonnement Azure, vous pouvez créer un [compte gratuit][free account] avant de commencer.

Dans ce guide de démarrage rapide, vous allez utiliser Azure Cloud Shell, que vous pouvez lancer après vous être connecté au portail Azure. Pour plus d’informations sur Azure Cloud Shell, consultez [Vue d’ensemble d’Azure Cloud Shell](../cloud-shell/overview.md). Vous pouvez également [installer](/cli/azure/install-azure-cli) et utiliser Azure PowerShell sur votre machine. 

## <a name="create-a-service-bus-topic-and-subscriptions"></a>Créer une rubrique Service Bus et des abonnements
Chaque [abonnement à une rubrique](service-bus-messaging-overview.md#topics) peut recevoir une copie de chaque message. Les rubriques sont entièrement compatibles au niveau de la sémantique et du protocole avec les files d’attente Service Bus. Les rubriques Service Bus prennent en charge un large éventail de règles de sélection avec des conditions de filtre et avec des actions facultatives qui permettent de définir ou de modifier les propriétés des messages. Chaque fois qu’une règle correspond, elle génère un message. Pour en savoir plus sur les règles, filtres et actions, consultez ce [lien](topic-filters.md).

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Lancez Azure Cloud Shell en sélectionnant l’icône illustrée dans l’image suivante. Basculez en mode **Bash** si Cloud Shell est en mode **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Lancer Cloud Shell":::
3. Exécutez la commande suivante pour créer un groupe de ressources Azure. Mettez à jour le nom et l’emplacement du groupe de ressources si vous le souhaitez. 

    ```azurecli-interactive
    az group create --name MyResourceGroup --location eastus
    ```
4. Exécutez la commande suivante pour créer un espace de noms de messagerie Service Bus. Mettez à jour le nom de l’espace de noms pour qu’il soit unique. 

    ```azurecli-interactive
    namespaceName=MyNameSpace$RANDOM
    az servicebus namespace create --resource-group MyResourceGroup --name $namespaceName --location eastus
    ```
5. Exécutez la commande suivante pour créer une rubrique dans l’espace de noms. 

    ```azurecli-interactive
    az servicebus topic create --resource-group MyResourceGroup   --namespace-name $namespaceName --name MyTopic
    ```
6. Créez le premier abonnement à la rubrique.
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S1    
    ```
6. Créez le deuxième abonnement à la rubrique.
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S2    
    ```
6. Créez le troisième abonnement à la rubrique.
    
    ```azurecli-interactive
    az servicebus topic subscription create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --name S3    
    ```
7. Créez un filtre sur le premier abonnement avec un filtre utilisant des propriétés personnalisées (`StoreId` est `Store1`, `Store2` ou `Store3`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --subscription-name S1 --name MyFilter --filter-sql-expression "StoreId IN ('Store1','Store2','Store3')"    
    ```
8. Créez un filtre sur le deuxième abonnement avec un filtre utilisant des propriétés du client (`StoreId = Store4`)

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name myTopic --subscription-name S2 --name MySecondFilter --filter-sql-expression "StoreId = 'Store4'"    
    ```
9. Créez un filtre sur le troisième abonnement avec un filtre utilisant des propriétés du client (`StoreId` pas dans `Store1`, `Store2`, `Store3` ou `Store4`).

    ```azurecli-interactive
    az servicebus topic subscription rule create --resource-group MyResourceGroup --namespace-name $namespaceName --topic-name MyTopic --subscription-name S3 --name MyThirdFilter --filter-sql-expression "StoreId NOT IN ('Store1','Store2','Store3', 'Store4')"     
    ```
10. Exécutez la commande suivante pour obtenir la chaîne de connexion principale de l’espace de noms. Vous utilisez cette chaîne de connexion pour vous connecter à la file d’attente et envoyer et recevoir des messages. 

    ```azurecli-interactive
    az servicebus namespace authorization-rule keys list --resource-group MyResourceGroup --namespace-name $namespaceName --name RootManageSharedAccessKey --query primaryConnectionString --output tsv    
    ```

    Notez la chaîne de connexion et le nom de la rubrique. Vous les utiliserez pour envoyer et recevoir des messages. 
    

## <a name="next-steps"></a>Étapes suivantes
Pour savoir comment envoyer des messages à une rubrique et recevoir ces messages par le biais d’un abonnement, consultez l’article correspondant au langage de programmation de votre choix dans la table des matières. 

> [!div class="nextstepaction"]
> [Publier et s’abonner à des messages](service-bus-dotnet-how-to-use-topics-subscriptions.md)


[free account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
[fully qualified domain name]: https://wikipedia.org/wiki/Fully_qualified_domain_name
[Install the Azure CLI]: /cli/azure/install-azure-cli
[az group create]: /cli/azure/group#az_group_create
