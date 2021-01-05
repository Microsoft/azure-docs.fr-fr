---
title: Démarrage rapide - Utiliser Azure CLI pour créer une file d’attente Service Bus | Microsoft Docs
description: Dans ce guide de démarrage rapide, vous allez apprendre à utiliser Azure CLI pour créer un espace de noms Service Bus puis une file d'attente dans celui-ci.
author: spelluru
ms.topic: quickstart
ms.date: 08/12/2020
ms.author: spelluru
ms.openlocfilehash: 14892b114f3bb8a0bb6c07f7cdd01a174b931cf9
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95810630"
---
# <a name="use-the-azure-cli-to-create-a-service-bus-namespace-and-a-queue"></a>Utiliser Azure CLI pour créer un espace de noms Service Bus ainsi qu’une file d'attente
Ce guide de démarrage rapide vous montre comment créer un espace de noms Service Bus et une file d’attente à l’aide d’Azure CLI. Il vous montre également comment obtenir les informations d’autorisation qu’une application cliente peut utiliser pour envoyer/recevoir des messages à/de la file d'attente. 

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]

## <a name="prerequisites"></a>Prérequis
Si vous n'avez pas d'abonnement Azure, vous pouvez créer un [compte gratuit][free account] avant de commencer.

Dans ce guide de démarrage rapide, vous allez utiliser Azure Cloud Shell, que vous pouvez lancer après vous être connecté au portail Azure. Pour en savoir plus sur Azure Cloud Shell, consultez [Vue d’ensemble d’Azure Cloud Shell](../cloud-shell/overview.md). Vous pouvez également [installer](/cli/azure/install-azure-cli) et utiliser Azure PowerShell sur votre machine. 

## <a name="provision-resources"></a>Provisionner les ressources
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Lancez Azure Cloud Shell en sélectionnant l’icône illustrée dans l’image suivante. Basculez en mode **Bash** si Cloud Shell est en mode **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Lancer Cloud Shell":::
3. Exécutez la commande suivante pour créer un groupe de ressources Azure. Mettez à jour le nom et l’emplacement du groupe de ressources si vous le souhaitez. 

    ```azurecli-interactive
    az group create --name ContosoRG --location eastus
    ```
4. Exécutez la commande suivante pour créer un espace de noms de messagerie Service Bus.

    ```azurecli-interactive
    az servicebus namespace create --resource-group ContosoRG --name ContosoSBusNS --location eastus
    ```
5. Exécutez la commande suivante pour créer une file d’attente dans l’espace de noms que vous avez créé à l’étape précédente. Dans cet exemple, `ContosoRG` est le groupe de ressources que vous avez créé à l’étape précédente. `ContosoSBusNS` est le nom de l’espace de noms Service Bus créé dans ce groupe de ressources. 

    ```azurecli-interactive
    az servicebus queue create --resource-group ContosoRG --namespace-name ContosoSBusNS --name ContosoOrdersQueue
    ```
6. Exécutez la commande suivante pour obtenir la chaîne de connexion principale de l’espace de noms. Vous utilisez cette chaîne de connexion pour vous connecter à la file d’attente, envoyer et recevoir des messages. 

    ```azurecli-interactive
    az servicebus namespace authorization-rule keys list --resource-group ContosoRG --namespace-name ContosoSBusNS --name RootManageSharedAccessKey --query primaryConnectionString --output tsv    
    ```

    Notez la chaîne de connexion et le nom de la file d'attente. Vous les utiliserez pour envoyer et recevoir des messages. 


## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez créé un espace de noms Service Bus ainsi qu’une file d'attente dans celui-ci. Pour apprendre à envoyer/recevoir des messages à/de la file d’attente, consultez l’un des guides de démarrage rapide suivants dans la section **Envoyer et recevoir des messages**. 

- [.NET](service-bus-dotnet-get-started-with-queues.md)
- [Java](service-bus-java-how-to-use-queues.md)
- [JavaScript](service-bus-nodejs-how-to-use-queues.md)
- [Python](service-bus-python-how-to-use-queues.md)
- [PHP](service-bus-php-how-to-use-queues.md)
- [Ruby](service-bus-ruby-how-to-use-queues.md)

[free account]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio

