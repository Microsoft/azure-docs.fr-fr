---
author: baanders
description: Fichier Include pour le processus de publication d’une fonction Azure à partir de Visual Studio
ms.service: digital-twins
ms.topic: include
ms.date: 1/21/2021
ms.author: baanders
ms.openlocfilehash: 49f57f39ba59575417f3336d395b1cfa6cf97eb6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 05/25/2021
ms.locfileid: "110451115"
---
Pour publier le projet dans une application de fonction située dans Azure, commencez dans l’Explorateur de solutions. Cliquez avec le bouton droit sur le projet, puis choisissez **Publier**.

> [!IMPORTANT] 
> La publication d’une application de fonction dans Azure entraîne des frais supplémentaires dans votre abonnement, indépendamment d’Azure Digital Twins.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-1.png" alt-text="Capture d’écran de Visual Studio montrant le menu contextuel de la solution. L’option Publier est mise en évidence.":::

Dans la page **Publier** qui s’ouvre, laissez **Azure** comme cible par défaut. Sélectionnez ensuite **Suivant**. 

Pour une cible spécifique, choisissez **Application Azure Function (Windows)** , puis sélectionnez **Suivant**.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-2.png" alt-text="Capture d’écran de Visual Studio montrant la boîte de dialogue Publier une fonction Azure. Dans la page Cible spécifique, l’option Application Azure Function (Windows) est sélectionnée.":::

Sous l’onglet **Instance de Functions**, choisissez votre abonnement. Sélectionnez ensuite l’icône + pour créer une fonction.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-3.png" alt-text="Capture d’écran de Visual Studio montrant la boîte de dialogue Publier une fonction Azure. L’icône + est mise en évidence.":::

Dans la fenêtre **Application Azure Function - Créer**, renseignez les champs suivants :
* Le **Nom** est le nom du plan de consommation qu’Azure utilisera pour héberger votre application Azure Functions. Ce nom s’applique également à l’application de fonction qui contient votre fonction. Vous pouvez choisir une valeur ou conserver celle suggérée par défaut.
* Dans **Abonnement**, vérifiez que l’abonnement correspond bien à celui que vous souhaitez utiliser. 
* Dans **Groupe de ressources**, vérifiez que le groupe de ressources correspond bien à celui que vous souhaitez utiliser.
* Gardez **Consommation** comme **Type de plan**.
* Sélectionnez l’**Emplacement** de votre groupe de ressources.
* Créez une ressource **Stockage Azure** en sélectionnant le lien **Nouveau**. Définissez l’emplacement pour qu’il corresponde à votre groupe de ressources, utilisez les autres valeurs par défaut, puis sélectionnez **OK**.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-4.png" alt-text="Capture d’écran de Visual Studio montrant la page de la boîte de dialogue Publier une fonction Azure, dans laquelle les détails d’une nouvelle application de fonction sont remplis.":::

Sélectionnez ensuite **Créer**.

Une fois le service d’application créé, l’onglet **Instance de Functions** s’ouvre. Votre nouvelle application de fonction apparaît dans la zone **Applications de fonction** sous votre groupe de ressources. Sélectionnez **Terminer**.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-5.png" alt-text="Capture d’écran de Visual Studio montrant la boîte de dialogue Publier une fonction Azure, dans laquelle l’onglet Instances de Functions est sélectionné.":::

Dans le volet **Publier** qui s’ouvre dans la fenêtre principale de Visual Studio, vérifiez que toutes les informations sont correctes. Sélectionnez ensuite **Publier**.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-6.png" alt-text="Capture d’écran de Visual Studio montrant le volet Publier. Le bouton Publier est mis en évidence.":::

> [!NOTE]
> Si une fenêtre contextuelle semblable à celle de l’exemple suivant s’affiche, sélectionnez **Tentative de récupération des informations d’identification à partir d’Azure**, puis sélectionnez **Enregistrer**.
> :::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-7.png" alt-text="Capture d’écran d’une fenêtre contextuelle de Visual Studio appelée Informations d’identification de publication." border="false":::
>
> Si vous voyez l’un des avertissements suivants, suivez les invites pour effectuer une mise à niveau vers la dernière version du runtime Azure Functions :
> * « Mettre à niveau la version de Functions sur Azure »
> * « Votre version du runtime Functions ne correspond pas à la version exécutée dans Azure »
>
> Ces avertissements peuvent s’afficher si vous utilisez une ancienne version de Visual Studio.

Votre application de fonction est maintenant publiée sur Azure.
