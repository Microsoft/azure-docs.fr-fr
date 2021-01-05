---
title: Déployer un modèle sur Azure Analysis Services à l’aide de Visual Studio | Microsoft Docs
description: Découvrez comment déployer un modèle tabulaire sur un serveur Azure Analysis Services à l’aide de Visual Studio.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 9df10760164dcd0d207663c14107f72c46b76d25
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501241"
---
# <a name="deploy-a-model-from-visual-studio"></a>Déployer un modèle à partir de Visual Studio

Une fois que vous avez créé un serveur dans votre abonnement Azure, vous êtes prêt à déployer une base de données de modèle tabulaire sur celui-ci. Vous pouvez utiliser Visual Studio avec des projets de Analysis Services pour générer et déployer un projet de modèle tabulaire sur lequel vous travaillez. 

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous avez besoin des éléments suivants :

* **Serveur Analysis Services** dans Azure. Pour plus d’informations, consultez l’article [Création d’un serveur Azure Analysis Services dans le portail Azure](analysis-services-create-server.md).
* **Projet de modèle tabulaire** dans Visual Studio ou modèle tabulaire existant au niveau de compatibilité 1200 ou supérieur. Vous ne l’avez jamais fait ? Essayez le [Didacticiel de modélisation tabulaire des ventes Internet Adventure Works](/analysis-services/tutorial-tabular-1400/as-adventure-works-tutorial).
* **Passerelle locale** : si une ou plusieurs sources de données sont locales dans le réseau de votre entreprise, vous devez installer une [Passerelle de données locale](analysis-services-gateway.md). La passerelle est nécessaire pour que votre serveur se connecte dans le cloud à vos sources de données locales pour traiter et actualiser les données du modèle.

> [!TIP]
> Avant de déployer, assurez-vous que vous pouvez traiter les données de vos tables. Dans Visual Studio, cliquez sur **Modèle** > **Traiter** > **Traiter tout**. Si le traitement échoue, vous ne pourrez pas procéder au déploiement.
> 
> 

## <a name="get-the-server-name"></a>Obtenir le nom du serveur

Dans **Portail Azure** > Serveur > **Présentation** > **Nom du serveur**, copiez le nom du serveur.
   
![Obtenir le nom du serveur dans Azure](./media/analysis-services-deploy/aas-deploy-get-server-name.png)

## <a name="to-deploy-from-visual-studio"></a>Pour déployer depuis Visual Studio

1. Dans Visual Studio > **Explorateur de solutions**, cliquez avec le bouton droit sur le projet > **Propriétés**. Ensuite, dans **Déploiement** > **Serveur**, collez le nom du serveur.   
   
    ![Coller le nom du serveur dans la propriété du serveur de déploiement](./media/analysis-services-deploy/aas-deploy-deployment-server-property.png)
2. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur **Propriétés**, puis cliquez sur **Déployer**. Vous serez peut-être invité à vous connecter à Azure.
   
    ![Déployer sur un serveur](./media/analysis-services-deploy/aas-deploy-deploy.png)
   
    L’état du déploiement s’affiche dans la fenêtre Sortie et dans Déployer.
   
    ![état du déploiement](./media/analysis-services-deploy/aas-deploy-status.png)

C’est tout !


## <a name="troubleshooting"></a>Dépannage

Si le déploiement échoue lors du déploiement de métadonnées, il est probable que Visual Studio n’ait pas pu se connecter à votre serveur. Vérifiez que vous pouvez vous connecter à votre serveur en utilisant SQL Server Management Studio (SSMS). Vérifiez ensuite que la propriété Serveur de déploiement du projet est correcte.

Si le déploiement échoue sur une table, il est probable que votre serveur n’ait pas pu se connecter à une source de données. Si votre source de données est locale dans le réseau de votre entreprise, veillez à installer une [Passerelle de données locale](analysis-services-gateway.md).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que votre modèle tabulaire est déployé sur votre serveur, vous êtes prêt à vous connecter à celui-ci. Vous pouvez vous [connecter avec SQL Server Management Studio (SSMS)](analysis-services-manage.md) pour le gérer. Vous pouvez également vous [connecter à celui-ci à l’aide d’un outil client](analysis-services-connect.md) tel que Power BI, Power BI Desktop ou Excel, et commencer à créer des rapports.   

Pour en savoir plus sur les méthodes du déploiement avancé, consultez [Déploiement d’une solution de modèle tabulaire](/analysis-services/deployment/tabular-model-solution-deployment?view=azure-analysis-services-current&preserve-view=true).