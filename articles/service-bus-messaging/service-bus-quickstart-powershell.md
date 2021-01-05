---
title: Utiliser Azure PowerShell pour créer une file d’attente Service Bus
description: Dans ce guide de démarrage rapide, vous allez apprendre à créer un espace de noms Service Bus puis une file d’attente dans un espace de noms à l’aide d’Azure PowerShell.
author: spelluru
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 08/12/2020
ms.author: spelluru
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8cf024735c66e6bae9d334e3d8ce8d0a0eed1426
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95799105"
---
# <a name="use-azure-powershell-to-create-a-service-bus-namespace-and-a-queue"></a>Utiliser Azure PowerShell pour créer un espace de noms Service Bus ainsi qu’une file d'attente
Ce guide de démarrage rapide vous montre comment créer un espace de noms Service Bus et une file d’attente à l’aide d’Azure PowerShell. Il vous montre également comment obtenir les informations d’autorisation qu’une application cliente peut utiliser pour envoyer/recevoir des messages à/de la file d'attente. 

[!INCLUDE [howto-service-bus-queues](../../includes/howto-service-bus-queues.md)]


## <a name="prerequisites"></a>Prérequis

Pour suivre ce démarrage rapide, vérifiez que vous disposez d’un abonnement Azure. Si vous n'avez pas d'abonnement Azure, vous pouvez créer un [compte gratuit][] avant de commencer. 

Dans ce guide de démarrage rapide, vous allez utiliser Azure Cloud Shell, que vous pouvez lancer après vous être connecté au portail Azure. Pour en savoir plus sur Azure Cloud Shell, consultez [Vue d’ensemble d’Azure Cloud Shell](../cloud-shell/overview.md). Vous pouvez également [installer](/powershell/azure/install-Az-ps) et utiliser Azure PowerShell sur votre machine. 


## <a name="provision-resources"></a>Provisionner les ressources
1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Lancez Azure Cloud Shell en sélectionnant l’icône illustrée dans l’image suivante : 

    :::image type="content" source="./media/service-bus-quickstart-powershell/launch-cloud-shell.png" alt-text="Lancer Cloud Shell":::
3. Dans la fenêtre Cloud Shell située en bas, passez du mode **Bash** au mode **PowerShell**. 

    :::image type="content" source="./media/service-bus-quickstart-powershell/cloud-power-shell.png" alt-text="Passer au mode PowerShell":::    
4. Exécutez la commande suivante pour créer un groupe de ressources Azure. Mettez à jour le nom et l’emplacement du groupe de ressources si vous le souhaitez. 

    ```azurepowershell-interactive
    New-AzResourceGroup –Name ContosoRG –Location eastus
    ```
5. Exécutez la commande suivante pour créer un espace de noms de messagerie Service Bus. Dans cet exemple, `ContosoRG` est le groupe de ressources que vous avez créé à l’étape précédente. `ContosoSBusNS` est le nom de l’espace de noms Service Bus créé dans ce groupe de ressources. 

    ```azurepowershell-interactive
    New-AzServiceBusNamespace -ResourceGroupName ContosoRG -Name ContosoSBusNS -Location eastus
    ```
6. Exécutez ce qui suit pour créer une file d’attente dans l’espace de noms que vous avez créé à l’étape précédente. 

    ```azurepowershell-interactive
    New-AzServiceBusQueue -ResourceGroupName ContosoRG -NamespaceName ContosoSBusNS -Name ContosoOrdersQueue 
    ```
7. Obtenez la première chaîne de connexion pour l’espace de noms. Vous utilisez cette chaîne de connexion pour vous connecter à la file d’attente, envoyer et recevoir des messages. 

    ```azurepowershell-interactive    
    Get-AzServiceBusKey -ResourceGroupName ContosoRG -Namespace ContosoSBusNS -Name RootManageSharedAccessKey
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

[compte gratuit]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio

