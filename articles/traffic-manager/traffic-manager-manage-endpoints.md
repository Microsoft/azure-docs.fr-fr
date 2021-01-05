---
title: Gérer des points de terminaison dans Azure Traffic Manager | Microsoft Docs
description: Cet article vous aide à ajouter, supprimer, activer et désactiver des points de terminaison d’Azure Traffic Manager.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/08/2017
ms.author: duau
ms.openlocfilehash: fc6e8bcda17f9d788a430ed763bd9f8702eaf250
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003546"
---
# <a name="add-disable-enable-or-delete-endpoints"></a>Ajouter, désactiver, activer ou supprimer des points de terminaison

La fonction Web Apps d’Azure App Service fournit déjà des fonctionnalités de routage du trafic par basculement et tourniquet pour les sites web d’un centre de données, indépendamment du mode de site Web. Azure Traffic Manager vous permet de spécifier le routage du trafic par basculement et tourniquet pour des sites web et des services cloud de différents centres de données. La première étape nécessaire pour fournir cette fonctionnalité consiste à ajouter le point de terminaison de service cloud ou de site web à Traffic Manager.

Vous pouvez également désactiver des points de terminaison individuels qui font partie d'un profil Traffic Manager. Quand un point de terminaison est désactivé, il fait toujours partie du profil, mais celui-ci se comporte comme si ce n'était pas le cas. Cette action s’avère utile pour supprimer temporairement un point de terminaison en mode de maintenance ou redéployé. Une fois le point de terminaison à nouveau en fonction, il peut être activé.

> [!NOTE]
> La désactivation d'un point de terminaison n'a aucun lien avec son état de déploiement dans Azure. Un point de terminaison sain reste en fonction et peut recevoir du trafic même quand il est désactivé dans Traffic Manager. En outre, le fait de désactiver un point de terminaison dans un profil n'affecte pas son statut dans un autre profil.

## <a name="to-add-a-cloud-service-or-an-app-service-endpoint-to-a-traffic-manager-profile"></a>Pour ajouter un point de terminaison de service cloud ou App Service à un profil Traffic Manager

1. Dans un navigateur, connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous souhaitez modifier et cliquez sur le profil Traffic Manager dans les résultats affichés.
3. Dans le panneau **Profil Traffic Manager**, dans la section **Paramètres**, cliquez sur **Points de terminaison**.
4. Dans le panneau **Points de terminaison** qui s’affiche, cliquez sur **Ajouter**.
5. Dans le panneau **Ajouter le point de terminaison**, procédez comme suit :
    1. Sous **Type**, cliquez sur **Point de terminaison Azure**.
    2. Entrez un **Nom** pour ce point de terminaison.
    3. Pour **Type de ressource cible**, dans la liste déroulante, choisissez le type de ressource approprié.
    4. Pour **Ressource cible**, cliquez sur le sélecteur **Choisir...** pour dresser la liste des ressources au sein du même abonnement dans le **Panneau des ressources**. Dans le panneau **Ressources** qui s’affiche, choisissez le service que vous souhaitez ajouter en tant que premier point de terminaison.
    5. Pour **Priorité**, sélectionnez **1**. Ainsi, tout le trafic est dirigé vers ce point de terminaison, s’il est intègre.
    6. Vérifiez que la case **Ajouter comme désactivé** est désélectionnée.
    7. Cliquez sur **OK**
6.  Répétez les étapes 4 et 5 pour ajouter le prochain point de terminaison Azure. Veillez à ajouter à sa valeur de **Priorité** définie sur **2**.
7.  Lorsque l’ajout de deux points de terminaison est terminé, ceux-ci s’affichent dans le panneau du **profil Traffic Manager** ainsi que leur état de surveillance en tant que **En ligne**.

> [!NOTE]
> Après ajout ou suppression d’un point de terminaison dans un profil selon la méthode de routage du trafic par *Basculement*, la liste de priorité des basculements peut ne pas être ordonnée comme vous le souhaitez. Vous pouvez ajuster l’ordre de la liste de priorité des basculements sur la page Configuration. Pour plus d’informations, consultez la rubrique [Configurer le routage du trafic par basculement](traffic-manager-configure-failover-routing-method.md).

## <a name="to-disable-an-endpoint"></a>Pour désactiver un point de terminaison

1. Dans un navigateur, connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous souhaitez modifier, puis cliquez sur le profil Traffic Manager dans les résultats affichés.
3. Dans le panneau **Profil Traffic Manager**, dans la section **Paramètres**, cliquez sur **Points de terminaison**. 
4. Cliquez sur le point de terminaison que vous voulez désactiver.
5. Dans le panneau **Point de terminaison**, changez l’état du point de terminaison en **Désactivé**, puis cliquez sur **Enregistrer**.
6. Les clients continuent à envoyer le trafic vers le point de terminaison pendant la durée de vie (TTL). Vous pouvez modifier la durée de vie sur la page Configuration du profil Traffic Manager.

## <a name="to-enable-an-endpoint"></a>Pour activer un point de terminaison

1. Dans un navigateur, connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous souhaitez modifier, puis cliquez sur le profil Traffic Manager dans les résultats affichés.
3. Dans le panneau **Profil Traffic Manager**, dans la section **Paramètres**, cliquez sur **Points de terminaison**. 
4. Cliquez sur le point de terminaison que vous voulez activer.
5. Dans le panneau **Point de terminaison**, changez l’état du point de terminaison en **Activé**, puis cliquez sur **Enregistrer**.
6. Les clients continuent à envoyer le trafic vers le point de terminaison pendant la durée de vie (TTL). Vous pouvez modifier la durée de vie sur la page Configuration du profil Traffic Manager.

## <a name="to-delete-an-endpoint"></a>Pour supprimer un point de terminaison

1. Dans un navigateur, connectez-vous au [portail Azure](https://portal.azure.com).
2. Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous souhaitez modifier, puis cliquez sur le profil Traffic Manager dans les résultats affichés.
3. Dans le panneau **Profil Traffic Manager**, dans la section **Paramètres**, cliquez sur **Points de terminaison**. 
4. Cliquez sur le point de terminaison que vous voulez supprimer.
5. Dans le panneau **Point de terminaison**, cliquez sur **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes

* [Gérer des profils Traffic Manager](traffic-manager-manage-profiles.md)
* [Configurer des méthodes de routage](traffic-manager-configure-routing-method.md)
* [Résolution des problèmes liés à l’état Détérioré de Traffic Manager](traffic-manager-troubleshooting-degraded.md)
* [Considérations sur les performances de Traffic Manager](traffic-manager-performance-considerations.md)
* [Opérations sur Traffic Manager (Référence sur l’API REST)](https://go.microsoft.com/fwlink/p/?LinkID=313584)

