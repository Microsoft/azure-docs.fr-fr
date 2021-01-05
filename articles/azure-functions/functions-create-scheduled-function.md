---
title: Créer une fonction qui s’exécute selon une planification dans Azure
description: Apprenez à créer une fonction dans Azure, qui s’exécute selon une planification que vous définissez.
ms.assetid: ba50ee47-58e0-4972-b67b-828f2dc48701
ms.topic: how-to
ms.date: 04/16/2020
ms.custom: mvc, cc996988-fb4f-47
ms.openlocfilehash: 9e542c493c02174364072f91d092f05ad9ec69cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90973099"
---
# <a name="create-a-function-in-azure-that-is-triggered-by-a-timer"></a>Créez une fonction dans Azure, qui est déclenchée par un minuteur

Apprenez à utiliser Azure Functions pour créer une fonction [serverless](https://azure.microsoft.com/solutions/serverless/) qui s’exécute selon une planification que vous définissez.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

+ Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-an-azure-function-app"></a>Création d’une application Azure Function

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

Votre nouvelle application de fonction est prête à être utilisée. Créez ensuite une fonction dans la nouvelle application de fonction.

:::image type="content" source="./media/functions-create-scheduled-function/function-app-create-success.png" alt-text="Application de fonction créée avec succès." border="true":::

<a name="create-function"></a>

## <a name="create-a-timer-triggered-function"></a>Créer une fonction déclenchée par un minuteur

1. Dans votre application de fonction, sélectionnez **Fonctions**, puis **+ Ajouter**. 

   :::image type="content" source="./media/functions-create-scheduled-function/function-add-function.png" alt-text="Application de fonction créée avec succès." border="true":::

1. Sélectionnez le modèle **Déclencheur de minuteur**. 

    :::image type="content" source="./media/functions-create-scheduled-function/function-select-timer-trigger.png" alt-text="Application de fonction créée avec succès." border="true":::

1. Configurez le nouveau déclencheur avec les paramètres spécifié dans le tableau situé sous l’image, puis sélectionnez **Créer une fonction**.

    :::image type="content" source="./media/functions-create-scheduled-function/function-configure-timer-trigger.png" alt-text="Application de fonction créée avec succès." border="true":::
    
    | Paramètre | Valeur suggérée | Description |
    |---|---|---|
    | **Nom** | Default | Définit le nom de votre fonction déclenchée par minuteur. |
    | **Planification** | 0 \*/1 \* \* \* \* | Un champ de six [expressions CRON](functions-bindings-timer.md#ncrontab-expressions) qui planifie l’exécution de votre fonction chaque minute. |

## <a name="test-the-function"></a>Tester la fonction

1. Dans votre fonction, sélectionnez **Code + test**, puis développez les journaux.

    :::image type="content" source="./media/functions-create-scheduled-function/function-test-timer-trigger.png" alt-text="Application de fonction créée avec succès." border="true":::

1. Vérifiez l’exécution en consultant les informations écrites dans les journaux.

    :::image type="content" source="./media/functions-create-scheduled-function/function-view-timer-logs.png" alt-text="Application de fonction créée avec succès." border="true":::

À présent, vous pouvez modifier la planification de la fonction afin qu’elle s’exécute une fois par heure plutôt que toutes les minutes.

## <a name="update-the-timer-schedule"></a>Mise à jour de la planification du minuteur

1. Dans votre fonction, sélectionnez **Intégration**. Il s’agit de l’endroit où vous définissez les liaisons d’entrée et de sortie pour votre fonction, et où vous configurez la planification. 

1. Sélectionnez **Minuteur (myTimer)** .

    :::image type="content" source="./media/functions-create-scheduled-function/function-update-timer-schedule.png" alt-text="Application de fonction créée avec succès." border="true":::

1. Mettez à jour la valeur **Planification** sur `0 0 */1 * * *`, puis sélectionnez **Enregistrer**.  

    :::image type="content" source="./media/functions-create-scheduled-function/function-edit-timer-schedule.png" alt-text="Application de fonction créée avec succès." border="true":::

Vous disposez maintenant d’une fonction qui s’exécute toutes les heures, à l’heure pile.

## <a name="clean-up-resources"></a>Nettoyer les ressources

[!INCLUDE [Next steps note](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé une fonction qui s’exécute selon une planification. Pour plus d’informations sur les déclencheurs de minuteur, consultez [Planifier l’exécution de code avec Azure Functions](functions-bindings-timer.md).

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]
