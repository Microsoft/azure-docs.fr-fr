---
title: 'Tutoriel : Partager des ancres entre plusieurs sessions et appareils'
description: Dans ce tutoriel, vous allez apprendre à partager des identificateurs Azure Spatial Anchor entre des appareils Android/iOS dans Unity avec un service back-end.
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: tutorial
ms.service: azure-spatial-anchors
ms.openlocfilehash: 3048eb1ca7f9312a43bc1ab5885bf19d4c2e7931
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185393"
---
# <a name="tutorial-share-spatial-anchors-across-sessions-and-devices"></a>Tutoriel : Partager des ancres spatiales entre plusieurs sessions et appareils

Azure Spatial Anchors est un service de développement multiplateforme avec lequel vous pouvez créer des expériences de réalité mixte en utilisant des objets qui conservent leur emplacement sur les appareils dans le temps. 

Dans ce tutoriel, vous allez utiliser [Azure Spatial Anchors](../overview.md) pour créer des ancres au cours d’une session et les localiser ensuite sur le même appareil ou sur un autre. Les mêmes ancres peuvent aussi être localisées par plusieurs appareils à un même emplacement et à un même moment.

![Animation montrant des ancres spatiales créées avec un appareil mobile et utilisées avec un appareil différent au fil des jours.](./media/persistence.gif)


Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Déployer une application web ASP.NET Core dans Azure permettant de partager des ancres et de les stocker en mémoire pendant une période spécifiée.
> * Configurer la scène AzureSpatialAnchorsLocalSharedDemo dans l’exemple Unity de nos guides de démarrage rapide pour tirer parti de l’application web de partage d’ancres.
> * Déployer et exécuter les ancres sur un ou plusieurs appareils.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [Share Anchors Sample Prerequisites](../../../includes/spatial-anchors-share-sample-prereqs.md)]

> [!NOTE]
> Vous allez utiliser Unity et une application web ASP.NET Core dans ce tutoriel, mais l’idée ici est seulement de donner un exemple de la manière dont sont partagés les identificateurs Azure Spatial Anchors avec d’autres appareils. Vous pouvez utiliser d’autres langages et technologies back-end pour atteindre le même objectif.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="download-the-sample-project"></a>Téléchargement de l’exemple de projet

[!INCLUDE [Clone Sample Repo](../../../includes/spatial-anchors-clone-sample-repository.md)]

## <a name="deploy-the-sharing-anchors-service"></a>Déployer le service de partage d’ancres

## <a name="visual-studio"></a>[Visual Studio](#tab/VS)

Après avoir ouvert Visual Studio, ouvrez le projet dans le dossier *Sharing\SharingServiceSample*.

[!INCLUDE [Publish Azure](../../../includes/spatial-anchors-publish-azure.md)]

## <a name="visual-studio-code"></a>[Visual Studio Code](#tab/VSC)

Vous devez créer un groupe de ressources et un plan App Service avant de déployer le service dans Visual Studio Code.

### <a name="sign-in-to-azure"></a>Connexion à Azure

Accédez au <a href="https://portal.azure.com/" target="_blank">portail Azure</a>, puis connectez-vous à votre abonnement Azure.

### <a name="create-a-resource-group"></a>Créer un groupe de ressources

[!INCLUDE [resource group intro text](../../../includes/resource-group.md)]

En regard de **Groupe de ressources**, sélectionnez **Nouveau**.

Nommez le groupe de ressources **myResourceGroup**, puis sélectionnez **OK**.

### <a name="create-an-app-service-plan"></a>Créer un plan App Service

[!INCLUDE [app-service-plan](../../../includes/app-service-plan.md)]

En regard de **Plan d’hébergement**, sélectionnez **Nouveau**.

Dans le volet **Configurer un plan d’hébergement**, utilisez ces paramètres :

| Paramètre | Valeur suggérée | Description |
|-|-|-|
|Plan App Service| MySharingServicePlan | Nom du plan App Service |
| Emplacement | USA Ouest | Centre de données où l’application web est hébergée |
| Taille | Gratuit | [Niveau tarifaire](https://azure.microsoft.com/pricing/details/app-service/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) déterminant les fonctionnalités d’hébergement |

Sélectionnez **OK**.

Après avoir ouvert Visual Studio Code, ouvrez le projet dans le dossier *Sharing\SharingServiceSample*. 

Pour déployer le service de partage via Visual Studio Code, suivez les instructions dans <a href="/aspnet/core/tutorials/publish-to-azure-webapp-using-vscode?view=aspnetcore-2.2#open-it-with-visual-studio-code" target="_blank">Publier une application ASP.NET Core sur Azure avec Visual Studio Code</a>. Commencez à la section « Ouvrir le projet avec Visual Studio Code ». Ne créez pas un autre projet ASP.NET comme expliqué à l’étape précédente, car vous avez déjà un projet à déployer et à publier : SharingServiceSample.

---

## <a name="deploy-the-sample-app"></a>Déployer l’exemple d’application

[!INCLUDE [Run Share Anchors Sample](../../../includes/spatial-anchors-run-share-sample.md)]

[!INCLUDE [Clean-up section](../../../includes/clean-up-section-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez déployé une application web ASP.NET Core dans Azure, puis configuré et déployé une application Unity. Vous avez créé des ancres spatiales avec l’application et les avez partagées avec d’autres appareils en utilisant votre application web ASP.NET Core.

Vous pouvez améliorer votre application web ASP.NET Core afin qu’elle utilise Azure Cosmos DB pour rendre le stockage des identificateurs de vos ancres spatiales partagées permanent. En ajoutant la prise en charge d’Azure Cosmos DB, vous pouvez dès aujourd’hui créer une ancre avec votre application web ASP.NET Core. Ensuite, à partir de l’identificateur d’ancre stocké dans votre application web, vous pouvez faire en sorte que l’application relocalise l’ancre quelques jours plus tard.

> [!div class="nextstepaction"]
> [Utiliser Azure Cosmos DB pour stocker des ancres](./tutorial-use-cosmos-db-to-store-anchors.md)