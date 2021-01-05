---
title: Débogage local de fonctions Azure Event Grid
description: Apprendre à déboguer localement des fonctions Azure Functions déclenchées par un événement Event Grid
author: craigshoemaker
ms.topic: conceptual
ms.date: 10/18/2018
ms.author: cshoe
ms.openlocfilehash: be05d237d2799404c3fd8b5733464e23eeb49aa3
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94833059"
---
# <a name="azure-function-event-grid-trigger-local-debugging"></a>Débogage local lors du déclenchement de fonctions Azure Event Grid

Cet article montre comment déboguer une fonction locale qui gère un événement Azure Event Grid déclenché par un compte de stockage. 

## <a name="prerequisites"></a>Prérequis

- Créer ou utiliser une application de fonction existante
- Créer ou utiliser un compte de stockage existant
- Télécharger [ngrok](https://ngrok.com/) pour permettre à Azure d’appeler une fonction locale

## <a name="create-a-new-function"></a>Créer une fonction

Ouvrez votre application de fonction dans Visual Studio, cliquez avec le bouton droit sur le nom du projet dans l’Explorateur de solutions, puis cliquez sur **Ajouter > Nouvelle fonction Azure**.

Dans la fenêtre *Nouvelle fonction Azure*, sélectionnez **Déclencheur Event Grid**, puis cliquez sur **OK**.

![Créer une fonction](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-add-function.png)

Une fois la fonction créée, ouvrez le fichier de code et copiez l’URL commentée en haut du fichier. Cet emplacement est utilisé lorsque vous configurez le déclencheur Event Grid.

![Emplacement de copie](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-copy-location.png)

Définissez ensuite un point d’arrêt sur la ligne commençant par `log.LogInformation`.

![Définir le point d’arrêt](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-set-breakpoint.png)


**Appuyez sur F5** pour démarrer une session de débogage.

## <a name="allow-azure-to-call-your-local-function"></a>Autoriser Azure à appeler votre fonction locale

Pour arrêter une fonction en cours de débogage sur votre ordinateur, vous devez permettre à Azure de communiquer avec votre fonction locale à partir du cloud.

L’utilitaire [ngrok](https://ngrok.com/) permet à Azure d’appeler la fonction en cours d’exécution sur votre ordinateur. Démarrez *ngrok* en utilisant la commande suivante :

```bash
ngrok http -host-header=localhost 7071
```
Pendant la configuration de l’utilitaire, la fenêtre de commande doit ressembler à la capture d’écran suivante :

![Capture d’écran de l’invite de commandes après le lancement de l’utilitaire « ngrok »](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-ngrok.png)

Copiez l’URL **HTTPS** générée lorsque *ngrok* est exécuté. Cette valeur est utilisée lors de la configuration du point de terminaison de l’événement Event Grid.

## <a name="add-a-storage-event"></a>Ajouter une clé de stockage

Ouvrez le portail Azure, accédez à un compte de stockage, puis cliquez sur l’option **Événements**.

![Ajouter un événement de compte de stockage](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-add-event.png)

Dans la fenêtre *Événements*, cliquez sur le bouton **Abonnement à un événement**. Dans la fenêtre *Abonnement à un événement*, cliquez sur la liste déroulante *Type de point de terminaison* et sélectionnez **Webhook**.

![Sélectionner un type d’abonnement](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-event-subscription-type.png)

Une fois le type de point de terminaison configuré, cliquez sur **Sélectionner un point de terminaison** pour configurer la valeur du point de terminaison.

![Sélectionner un type de point de terminaison](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-event-subscription-endpoint.png)

La valeur *Point de terminaison de l’abonné* se compose de trois valeurs distinctes. Le préfixe correspond à l’adresse URL HTTPS générée par *ngrok*. Le reste de l’URL est fourni par l’URL qui se trouve dans le fichier de code de fonction, avec le nom de la fonction ajouté à la fin. En commençant par l’URL provenant du fichier de code de fonction, l’URL *ngrok* remplace `http://localhost:7071` et le nom de la fonction remplace `{functionname}`.

L’URL finale doit ressembler à la capture d’écran suivante :

![Sélection de point de terminaison](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-event-subscription-endpoint-selection.png)

Une fois que vous avez entré la valeur appropriée, cliquez sur **Confirmer la sélection**.

> [!IMPORTANT]
> Chaque fois que vous démarrez *ngrok*, l’URL HTTPS est régénérée et la valeur change. Par conséquent, vous devez créer un abonnement d’événement chaque fois que vous exposez votre fonction à Azure via *ngrok*.

## <a name="upload-a-file"></a>Charger un fichier

Vous pouvez maintenant charger un fichier sur votre compte de stockage afin de déclencher un événement Event Grid pour votre fonction locale à gérer. 

Ouvrez l’[Explorateur Stockage](https://azure.microsoft.com/features/storage-explorer/) et connectez-vous à votre compte de stockage. 

- Développer **Conteneurs d'objets blob** 
- Cliquez avec le bouton droit et sélectionnez **Conteneurs d’objets blob**.
- Nommez le conteneur **test**
- Sélectionnez le conteneur *test*
- Cliquez sur le bouton **Télécharger**
- Cliquez sur **Charger des fichiers**
- Sélectionnez un fichier et chargez-le dans le conteneur d’objets blob

## <a name="debug-the-function"></a>Déboguer la fonction

Quand Event Grid reconnaît qu'un nouveau fichier est chargé dans le conteneur de stockage, le point d’arrêt est atteint dans votre fonction locale.

![Démarrer ngrok](./media/functions-debug-event-grid-trigger-local/functions-debug-event-grid-trigger-local-breakpoint.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour nettoyer les ressources créées dans cet article, supprimez le conteneur **test** de votre compte de stockage.

## <a name="next-steps"></a>Étapes suivantes

- [Automatiser le redimensionnement des images chargées à l’aide d’Event Grid](../event-grid/resize-images-on-storage-blob-upload-event.md)
- [Déclencheur Event Grid pour Azure Functions](./functions-bindings-event-grid.md)
