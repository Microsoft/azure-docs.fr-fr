---
title: Contrôler le trafic avec Traffic Manager
description: Découvrez les bonnes pratiques pour configurer Azure Traffic Manager au moment de l’intégrer à Azure App Service.
ms.assetid: dabda633-e72f-4dd4-bf1c-6e945da456fd
ms.topic: article
ms.date: 02/25/2016
ms.custom: seodec18
ms.openlocfilehash: 040f84288c66f4506919e775b9ea41324b617cfa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80437894"
---
# <a name="controlling-azure-app-service-traffic-with-azure-traffic-manager"></a>Contrôle du trafic Azure App Service avec Azure Traffic Manager
> [!NOTE]
> Cet article présente des informations résumées sur Microsoft Azure Traffic Manager et son rapport à Azure App Service. Vous trouverez d'autres informations sur Azure Traffic Manager en suivant les liens fournis à la fin de cet article.
> 
> 

## <a name="introduction"></a>Introduction
Vous pouvez utiliser Azure Traffic Manager pour contrôler la façon dont les requêtes des clients web sont distribuées aux applications dans Azure App Service. Lorsque des points de terminaison App Service sont ajoutés à un profil Azure Traffic Manager, ce dernier conserve une trace du statut de vos applications App Service (en cours d’exécution, arrêtées ou supprimées) pour pouvoir décider du point de terminaison qui doit recevoir le trafic.

## <a name="routing-methods"></a>Méthodes de routage
Azure Traffic Manager utilise quatre méthodes de routage. Ces méthodes sont décrites dans la liste suivante, car elles se rapportent à Azure App Service.

* **[Priorité](../traffic-manager/traffic-manager-routing-methods.md#priority-traffic-routing-method) :** utilisez une application principale pour tout le trafic, et fournissez des sauvegardes au cas où celle-ci ou les applications de sauvegarde ne seraient pas disponibles.
* **[Pondération](../traffic-manager/traffic-manager-routing-methods.md#weighted) :** répartissez le trafic sur un ensemble d’applications, uniformément ou en fonction du poids que vous définissez.
* **[Performances](../traffic-manager/traffic-manager-routing-methods.md#performance) :** lorsque vous avez des applications à différents emplacements géographiques, utilisez l’application « la plus proche » en termes de latence réseau faible.
* **[Géographique](../traffic-manager/traffic-manager-routing-methods.md#geographic) :** orientez les utilisateurs vers des applications spécifiques en fonction de l’emplacement géographique d’où provient leur requête DNS. 

Pour plus d’informations, voir [Méthodes de routage de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md).

## <a name="app-service-and-traffic-manager-profiles"></a>App Service et profils dans Traffic Manager
Pour configurer le contrôle du trafic des applications App Service, créez un profil dans Azure Traffic Manager, qui utilise l’une des quatre méthodes d’équilibrage de charge décrites, puis ajoutez au profil les points de terminaison (ici, App Service) dont vous souhaitez contrôler le trafic. Le statut de votre application (en cours d’exécution, arrêtée ou supprimée) est régulièrement communiqué au profil, ce qui permet à Azure Traffic Manager d’orienter le trafic en conséquence.

Lorsque vous utilisez Azure Traffic Manager avec Azure, tenez compte des points suivants :

* Pour les déploiements des applications dans une même région uniquement, App Service fournit déjà les fonctionnalités de basculement et de tourniquet (round-robin) et ce, quel que soit le mode des applications.
* Pour les déploiements dans une même région qui utilisent App Service conjointement avec un autre service cloud Azure, vous pouvez combiner les deux types de point de terminaison pour autoriser les scénarios hybrides.
* Vous pouvez spécifier un seul point de terminaison App Service par région dans un profil. Lorsque vous sélectionnez une application comme point de terminaison pour une région, les autres applications de cette région ne peuvent plus être sélectionnées pour ce profil.
* Le point de terminaison App Service, que vous spécifiez dans un profil Azure Traffic Manager, s’affiche à la section **Noms de domaine** de la page Configurer pour l’application dans le profil, mais il n’est pas configurable ici.
* Lorsque vous avez ajouté une application à un profil, l’**URL du site** dans le tableau de bord de l’application sur le portail affiche l’URL du domaine personnalisé de l’application (s’il est défini). Sinon, l’URL du profil Traffic Manager (par exemple, `contoso.trafficmanager.net`) s’affiche. Le nom de domaine direct de l’application et l’URL de Traffic Manager sont tous deux affichés dans la page Configurer de l’application, à la section **Noms de domaine**.
* Vos noms de domaines personnalisés fonctionnent comme prévu, mais en plus de les ajouter à vos applications, vous devez configurer votre carte DNS pour qu’elle pointe vers l’URL de Traffic Manager. Pour plus d’informations sur la configuration d’un domaine personnalisé pour une application App Service, consultez [Configurer un nom de domaine personnalisé dans Azure App Service avec l’intégration de Traffic Manager](configure-domain-traffic-manager.md).
* Vous ne pouvez ajouter que des applications en mode standard ou Premium à un profil Azure Traffic Manager.
* L’ajout d’une application à un profil Traffic Manager entraîne le redémarrage de l’application.

## <a name="next-steps"></a>Étapes suivantes
Pour une vue d'ensemble conceptuelle et technique d'Azure Traffic Manager, consultez la rubrique [Vue d'ensemble de Traffic Manager](../traffic-manager/traffic-manager-overview.md).


