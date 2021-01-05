---
title: Gérer un plan App Service
description: Découvrez comment effectuer différentes tâches pour gérer un plan App Service (p.ex., créer, déplacer, mettre à l'échelle et supprimer).
keywords: app service, azure app service, mise à l’échelle, plan app service, changer, créer, gérer, gestion
ms.assetid: 4859d0d5-3e3c-40cc-96eb-f318b2c51a3d
ms.topic: article
ms.date: 10/24/2019
ms.custom: seodec18
ms.openlocfilehash: bb0765a1d7934d60f787ed277dd3bd1f9bc1359b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88962942"
---
# <a name="manage-an-app-service-plan-in-azure"></a>Gérer un plan App Service dans Azure

Un [plan Azure App Service](overview-hosting-plans.md) fournit les ressources nécessaires à l’exécution d’une application App Service. Ce guide montre comment gérer un plan App Service.

## <a name="create-an-app-service-plan"></a>Créer un plan App Service

> [!TIP]
> Si vous avez un environnement App Service, consultez [Créer un plan App Service dans un environnement App Service](environment/app-service-web-how-to-create-a-web-app-in-an-ase.md#createplan).

Vous pouvez créer un plan App Service vide ou en créer un dans le cadre de la création d’une application.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource**.

   ![Créez une ressource dans le Portail Azure.][createResource] 

1. Sélectionnez **Nouveau** > **Application web** ou un autre type d’application App Service.

   ![Créer une application dans le portail Azure.][createWebApp] 

2. Configurez la section **Détails de l’instance** avant de configurer le plan App Service. Des paramètres comme **Publier** et **Systèmes d’exploitation** sont susceptibles de modifier les niveaux tarifaires disponibles de votre plan App Service. **Région** détermine l’emplacement de création du plan App Service. 
   
3. Dans la section **Plan App Service**, sélectionnez un plan ou créez-en un en sélectionnant **Créer**.

   ![Créer un plan App Service.][createASP] 

4. Lorsque vous créez un plan, vous pouvez sélectionner son niveau tarifaire. Dans **Référence SKU et taille**, sélectionnez **Modifier la taille** pour modifier le niveau tarifaire. 

<a name="move"></a>

## <a name="move-an-app-to-another-app-service-plan"></a>Déplacer une application vers un autre plan App Service

Vous pouvez déplacer une application vers un autre plan App Service tant que les plans source et cible se trouvent dans _le même groupe de ressources et la même région géographique_.

> [!NOTE]
> Azure déploie chaque nouveau plan App Service dans une unité de déploiement, désignée en interne sous le terme d’espace web. Chaque région peut avoir de nombreux espaces web, mais votre application peut uniquement être déplacée entre des plans créés dans le même espace web. Un environnement App Service est un espace web isolé. Il en découle que les applications peuvent être déplacées entre les différents plans du même environnement App Service, mais pas entre des plans créés dans d’autres environnements App Service.
>
> Quand vous créez un plan, vous ne pouvez pas choisir l’espace web à utiliser, mais vous pouvez spécifier que le plan soit créé dans le même espace web qu’un autre plan existant. En résumé, tous les plans créés avec la même combinaison groupe de ressources/région sont déployés au sein du même espace web. Par exemple, si vous avez créé un plan dans le groupe de ressources A et la région B, tous les plans que vous créerez par la suite dans ce groupe de ressources A et cette région B seront déployés dans le même espace web. Notez que vous ne pouvez pas déplacer les plans vers d’autres espaces web après leur création et que, pour cette raison, vous ne pouvez pas déplacer un plan dans « le même espace web » qu’un autre plan en le déplaçant vers un autre groupe de ressources.
> 

1. Dans le [Portail Azure](https://portal.azure.com), cherchez et sélectionnez **App Services**, puis sélectionnez l’application que vous souhaitez déplacer.

2. Dans le menu de gauche, sélectionnez **Modifier le plan App Service**.

3. Dans la liste déroulante **Plan App Service**, sélectionnez le plan vers lequel l’application sera déplacée. La liste déroulante affiche seulement les plans qui se trouvent dans le même groupe de ressources et la même région géographique que le plan App Service actuel. S’il n’existe pas de plan de ce type, elle permet de créer un plan par défaut. Vous pouvez également créer un plan manuellement en sélectionnant **Créer**.

4. Si vous créez un plan, vous pouvez sélectionner son niveau tarifaire. Dans **Niveau tarifaire**, sélectionnez le niveau existant pour le modifier. 
   
   > [!IMPORTANT]
   > Si vous déplacez une application d’un plan de niveau élevé vers un plan de niveau inférieur, par exemple de **D1** à **F1**, elle risque de perdre certaines fonctionnalités dans le nouveau plan. Par exemple, si votre application utilise des certificats TLS/SSL, ce message d’erreur peut s’afficher :
   >
   > `Cannot update the site with hostname '<app_name>' because its current SSL configuration 'SNI based SSL enabled' is not allowed in the target compute mode. Allowed SSL configuration is 'Disabled'.`

5. Lorsque vous avez terminé, sélectionnez **OK**.
   
   ![Sélecteur de plan App Service.][change] 

## <a name="move-an-app-to-a-different-region"></a>Déplacer une application vers une autre région

La région dans laquelle votre application s’exécute est la région où se trouve le plan App Service. Toutefois, vous ne pouvez pas changer la région d’un plan App Service. Si vous souhaitez exécuter votre application dans une autre région, vous pouvez également utiliser le clonage d’application. Le clonage crée une copie de votre application dans un plan App Service, nouveau ou existant, dans n’importe quelle région.

La commande **Cloner l’application** figure dans la section **Outils de développement** du menu.

> [!IMPORTANT]
> Le clonage présente des limitations. Pour en savoir plus sur celles-ci, consultez [Clonage de l’application Azure App Service](app-service-web-app-cloning.md).

## <a name="scale-an-app-service-plan"></a>Mettre à l’échelle un plan App Service

Pour faire évoluer le niveau tarifaire d’un plan App Service, consultez [Scalabilité verticale d’une application dans Azure](manage-scale-up.md).

Pour effectuer un scale-out du nombre d’instances d’une application, consultez [Mise à l’échelle manuelle ou automatique du nombre d’instances](../azure-monitor/platform/autoscale-get-started.md).

<a name="delete"></a>

## <a name="delete-an-app-service-plan"></a>Supprimer un plan App Service

Par défaut, pour éviter des frais inattendus, quand vous supprimez la dernière application d’un plan App Service, App Service supprime également le plan. Si vous choisissez de conserver le plan, faites-le passer au niveau **Gratuit** pour ne pas être facturé.

> [!IMPORTANT]
> Les plans App Service auxquels aucune application n’est associée engendrent encore des frais, car ils continuent à réserver les instances de machine virtuelle configurées.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Montée en puissance d’une application dans Azure](manage-scale-up.md)

[change]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/change-appserviceplan.png
[createASP]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-appserviceplan.png
[createWebApp]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-web-app.png
[createResource]: ./media/azure-web-sites-web-hosting-plans-in-depth-overview/create-a-resource.png