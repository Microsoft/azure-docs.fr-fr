---
title: Fichier Include
description: Fichier Include
services: app-service
author: ggailey777
ms.service: app-service
ms.topic: include
ms.date: 06/26/2020
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: e208b52c67f173bd0d289715b63562df656b1ec9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88009756"
---
1. Dans **l’Explorateur de solutions**, cliquez avec le bouton droit sur le projet, puis sélectionnez **Publier**.

1. Dans la boîte de dialogue **Publier**, sélectionnez **Azure** dans **Cible**, puis sélectionnez **Suivant**. 

1. Sélectionnez **Azure WebJobs** pour **Cible spécifique**, puis sélectionnez **Suivant**.

1. Sélectionnez **Créer un nouveau WebJob Azure**.

   ![Choisir une cible de publication](./media/webjobs-publish-netcore/pick-publish-target.png)

1. Dans la boîte de dialogue **App Service (Windows)** , utilisez les paramètres d’hébergement spécifiés dans le tableau suivant.

    | Paramètre      | Valeur suggérée  | Description                                |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Nom** | Nom globalement unique | Nom qui identifie uniquement votre nouvelle application de fonction. |
    | **Abonnement** | Choisir votre abonnement | Sélectionnez l’abonnement Azure à utiliser. |
    | **[Groupe de ressources](../articles/azure-resource-manager/management/overview.md)** | myResourceGroup |  Nom du groupe de ressources où créer votre application de fonction. Choisissez **Nouveau** pour créer un groupe de ressources.|
    | **[Plan d’hébergement](../articles/app-service/overview-hosting-plans.md)** | Plan App Service | Un [plan App Service](../articles/app-service/overview-hosting-plans.md) spécifie l’emplacement, la taille et les fonctionnalités de la batterie de serveurs web qui héberge votre application. Vous pouvez économiser de l’argent lors de l’hébergement de plusieurs applications en configurant les applications web pour partager un plan App Service. Les plans App Service définissent la région, la taille de l’instance, le compte de l’échelle et la référence SKU (gratuite, partagée, de base, standard et premium). Choisissez **Nouveau** pour créer un plan App Service. |

    ![Boîte de dialogue Créer App Service](./media/webjobs-publish-netcore/app-service-dialog.png)

1. Sélectionnez **Créer** pour créer un WebJob et les ressources associées dans Azure avec ces paramètres et déployer votre code de projet.