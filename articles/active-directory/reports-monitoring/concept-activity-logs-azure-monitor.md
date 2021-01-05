---
title: Journaux d’activité Azure Active Directory dans Azure Monitor | Microsoft Docs
description: Introduction aux journaux d’activité Azure Active Directory dans Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4b18127b-d1d0-4bdc-8f9c-6a4c991c5f75
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/09/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8dbf14b1816d367f864cc3a2977497b34111df3f
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97586191"
---
# <a name="azure-ad-activity-logs-in-azure-monitor"></a>Journaux d’activité Azure Active Directory dans Azure Monitor

Vous pouvez acheminer les journaux d’activité Azure Active Directory (Azure AD) vers plusieurs points de terminaison pour une rétention à long terme et l’analyse des données. Cette fonctionnalité vous permet de :

* Archiver les journaux d’activité Azure AD vers un compte de stockage Azure afin conserver les données pendant longtemps.
* Diffuser les journaux d’activité Azure AD vers un Event Hub Azure pour analyser des données à l’aide des outils populaires de Security Information and Event Management (SIEM), comme Splunk et QRadar.
* Intégrer vos journaux d’activité Azure AD à vos propres solutions de journaux d’activité personnalisés en les diffusant en continu vers un Event Hub.
* Envoyer vos journaux d’activité Azure AD aux journaux d’activité Azure Monitor pour permettre les visualisations enrichies, la supervision et les alertes sur les données connectées.

> [!VIDEO https://www.youtube.com/embed/syT-9KNfug8]

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="supported-reports"></a>Rapports pris en charge

Vous pouvez acheminer les journaux d’audit Azure AD et authentifier des journaux d’activité vers votre compte de stockage Azure, un Event Hub, des journaux d’activité Azure Monitor ou une solution personnalisée à l’aide de cette fonctionnalité. 

* **Journaux d’audit**: Le [rapport d’activité des journaux d’audit](concept-audit-logs.md) vous permet de consulter l’historique de toutes les tâches effectuées dans votre locataire.
* **Journaux d’activité de connexion** : Le [rapport d’activité de connexion](concept-sign-ins.md) vous permet d’identifier qui a effectué les tâches consignées dans les journaux d’audit.

> [!NOTE]
> Les journaux d’activité d’audit et de connexions associés à B2C ne sont pas pris en charge pour le moment.
>

## <a name="prerequisites"></a>Prérequis

Pour utiliser cette fonctionnalité, vous avez besoin des éléments suivants :

* Un abonnement Azure. Si vous n’en avez pas, vous pouvez demander un [essai gratuit](https://azure.microsoft.com/free/).
* Une [licence](https://azure.microsoft.com/pricing/details/active-directory/) Azure AD Free, Basic, Premium 1 ou Premium 2 permet d’accéder aux journaux d’audit Azure AD dans le portail Azure. 
* Un locataire Azure AD.
* Utilisateur considéré comme **administrateur général** ou **administrateur de la sécurité** pour un locataire Azure AD.
* Une [licence](https://azure.microsoft.com/pricing/details/active-directory/) Azure AD Premium 1 ou Premium 2 permet d’accéder aux journaux d’activité de connexion Azure AD dans le portail Azure. 

Selon l’endroit où vous souhaitez acheminer les données du journal d’audit, vous devez disposer de l’un des éléments suivants :

* Un compte de stockage Azure doté des autorisations *ListKeys* activées. Nous vous recommandons d’utiliser un compte de stockage général et pas un compte de stockage blob. Pour plus d’informations sur la tarification du stockage, utilisez la [Calculatrice de prix pour le stockage Azure](https://azure.microsoft.com/pricing/calculator/?service=storage). 
* Un espace de noms Azure Event Hubs pour intégrer avec des solutions tierces.
* Un espace de travail Azure Log Analytics pour l’envoi de journaux d’activité aux journaux d’activité Azure Monitor.

## <a name="cost-considerations"></a>Considérations relatives au coût

Si vous avez déjà une licence Azure AD, vous avez besoin d’un abonnement Azure pour configurer le compte de stockage et l’Event Hub. L’abonnement Azure est fourni gratuitement, mais vous devez payer pour utiliser des ressources Azure, y compris le compte de stockage que vous utilisez pour l’archivage, et l’Event Hub que vous utilisez pour la diffusion en continu. La quantité de données, et donc le coût, varient considérablement selon la taille du locataire. 

### <a name="storage-size-for-activity-logs"></a>Taille de stockage des journaux d’activité

Chaque événement du journal d’audit utilise environ 2 Ko de stockage de données. Les journaux des événements de connexions représentent 4 Ko de stockage de données. Par conséquent, pour un locataire possédant 100 000 utilisateurs, et qui impliquerait environ 1,5 million d’événements par jour, vous auriez besoin d’environ 3 Go de stockage de données par jour. Dans la mesure où les écritures sont réalisées par lots de cinq minutes environ, vous pouvez anticiper environ 9000 opérations d’écriture par mois. 


Le tableau suivant contient une estimation des coûts, selon la taille du locataire. Ces coûts sont calculés pour un compte de stockage à usage général v2 dans la région USA Ouest, pendant au moins un an de rétention. Utilisez la [Calculatrice de prix de stockage Azure](https://azure.microsoft.com/pricing/details/storage/blobs/) pour obtenir une estimation plus précise du volume de données que vous prévoyez pour votre application.


| Catégorie de journal | Nombre d’utilisateurs | Événements par jour | Volume de données par mois (estimation) | Coût par mois (estimation) | Coût par an (estimation) |
|--------------|-----------------|----------------------|--------------------------------------|----------------------------|---------------------------|
| Audit | 100 000 | 1,5&nbsp;million | 90 Go | 1,93 $ | 23,12 $ |
| Audit | 1 000 | 15,000 | 900 Mo | 0,02 $ | 0,24 $ |
| Connexions | 1 000 | 34800 | 4 Go | 0,13 $ | 1,56 $ |
| Connexions | 100 000 | 15&nbsp;millions | 1,7 To | 35,41 $ | 424,92 $ |
 









### <a name="event-hub-messages-for-activity-logs"></a>Messages Event Hub pour les journaux d’activité

Les événements sont regroupés dans des intervalles d’environ cinq minutes et sont envoyés sous forme de message unique contenant tous les événements apparus au cours de cette période de temps. La taille maximale d’un message dans le Event Hub est de 256 Ko. Si la taille de tous les messages pendant cette période de temps dépasse ce volume, plusieurs messages sont envoyés. 

Par exemple, on compte normalement environ 18 événements par seconde pour un locataire important de plus de 100 000 utilisateurs, un taux qui équivaut à 5 400 événements toutes les cinq minutes. Étant donné que les journaux d’audit font environ 2 Ko par événement, cela équivaut à 10.8 Mo de données. Par conséquent, 43 sont envoyés à l’Event Hub dans cet intervalle de cinq minutes. 

Le tableau suivant contient les coûts estimés par mois pour un Event Hub de base aux USA Ouest en fonction du volume de données d’événement, qui peut varier d’un client à l’autre selon de nombreux facteurs tels que le comportement de connexion de l’utilisateur, etc. Utilisez la [Calculatrice de prix Event Hub](https://azure.microsoft.com/pricing/details/event-hubs/) pour obtenir une estimation plus précise du volume de données que vous prévoyez pour votre application.

| Catégorie de journal | Nombre d’utilisateurs | Événements par seconde | Événements par intervalle de cinq minutes | Volume par intervalle | Messages par intervalle | Messages par mois | Coût par mois (estimation) |
|--------------|-----------------|-------------------------|----------------------------------------|---------------------|---------------------------------|------------------------------|----------------------------|
| Audit | 100 000 | 18 | 5400 | 10,8 Mo | 43 | 371,520 | 10,83 $ |
| Audit | 1 000 | 0.1 | 52 | 104 Ko | 1 | 8640 | 10,80 $ |
| Connexions | 100 000 | 18000 | 5 400 000 | 10,.8 Go | 42188 | 364 504 320 | 23,9 $ |  
| Connexions | 1 000 | 178 | 53400 | 106,8&nbsp;MO | 418 | 3 611 520 | 11,06 $ |  

### <a name="azure-monitor-logs-cost-considerations"></a>Considérations relatives au coût des journaux d’activité Azure Monitor



| Catégorie de journal | Nombre d’utilisateurs | Événements par jour | Événements par mois (30 jours) | Coût par mois en dollars américains (estimation) |
|:-|--|--|--|-:|
| Audit et connexions | 100 000 | 16 500 000 | 495 000 000 | 1093,00 $ |
| Audit | 100 000 | 1 500 000 | 45 000 000 | 246,66 $ |
| Connexions | 100 000 | 15 000 000 | 450 000 000 | 847,28 $ |










Pour connaître les coûts liés à la gestion des journaux d’activité Azure Monitor, consultez [Gérer les coûts en contrôlant le volume et la rétention dans les journaux d’activité Azure Monitor](../../azure-monitor/platform/manage-cost-storage.md).

## <a name="frequently-asked-questions"></a>Forum aux questions

Cette section répond aux questions fréquemment posées et traitent des problèmes rencontrés avec les journaux d’activité Azure Active Directory dans Azure Monitor.

**Q : Quels sont les journaux d’activité inclus ?**

**R** : Les journaux d’audit et d’activité de connexion sont disponibles pour le routage via cette fonctionnalité. Néanmoins, les événements d’audit associés à B2C ne sont pas inclus à l’heure actuelle. Consultez le [Schéma des journaux d’audit](reference-azure-monitor-audit-log-schema.md) et le [Schéma des journaux de connexion](reference-azure-monitor-sign-ins-log-schema.md) pour savoir quels types de journaux et quelles fonctionnalités de ces journaux sont actuellement pris en charge. 

---

**Q : Combien de temps après une action les journaux d’activité correspondants s’affichent-ils dans mon Event Hub ?**

**R** : Les journaux d’activité apparaissent normalement dans votre Event Hub dans les deux à cinq minutes après que l’action est effectuée. Pour plus d’informations sur Event Hubs, consultez la section [Qu’est-ce qu’Event Hubs ?](../../event-hubs/event-hubs-about.md).

---

**Q : Combien de temps après une action les journaux d’activité correspondants s’affichent-ils dans mon compte de stockage ?**

**R** : Pour les comptes de stockage Azure, la latence est d’environ 5 à 15 minutes après l’exécution de l’action.

---

**Q : Que se passe-t-il si un administrateur modifie la période de rétention d’un paramètre de diagnostic ?**

**R** : La nouvelle stratégie de rétention sera appliquée aux journaux d’activité collectés après la modification. Les journaux d’activité collectés avant le changement de stratégie ne seront pas affectés.

---

**Q : Combien me coûtera le stockage de mes données ?**

**R** : Les coûts du stockage dépendent de la taille de vos journaux d’activité, ainsi que de la période de rétention que vous choisissez. Pour obtenir la liste de l’estimation des coûts pour les clients, qui dépendent du volume de journaux d’activité générés, consultez la section [Taille de stockage des journaux d’activité](#storage-size-for-activity-logs).

---

**Q : Quel est le coût de la diffusion en continu de mes données vers un Event Hub ?**

**R** : Le coût de la diffusion en continu dépend du nombre de messages reçus par minute. Cet article explique comment sont calculés les coûts et répertorie les estimations de coûts, qui sont basées sur le nombre de messages. 

---

**Q : Comment puis-je intégrer les journaux d’activité Azure AD à mon système SIEM ?**

**R** : Il existe deux méthodes pour le faire :

- Utilisez Azure Monitor avec Event Hubs pour diffuser les journaux dans votre système SIEM. Tout d’abord, [diffusez les journaux vers un Event Hub](tutorial-azure-monitor-stream-logs-to-event-hub.md), puis [configurez votre outil SIEM](tutorial-azure-monitor-stream-logs-to-event-hub.md#access-data-from-your-event-hub) auprès du Event Hub configuré. 

- Utilisez l’[API Graph de création de rapports](concept-reporting-api.md) pour accéder aux données, puis les intégrer à votre système SIEM avec vos propres scripts.

---

**Q : Quels sont les outils SIEM actuellement pris en charge ?** 

**R** : **R** : Actuellement, Azure Monitor est pris en charge par [Splunk](./howto-integrate-activity-logs-with-splunk.md), IBM QRadar, [Sumo Logic](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory), [ArcSight](./howto-integrate-activity-logs-with-arcsight.md), LogRhythm et Logz.io. Pour en savoir plus sur le fonctionnement des connecteurs, consultez la section [Diffuser des données de surveillance Azure vers un Event Hub pour les utiliser dans un outil externe](../../azure-monitor/platform/stream-monitoring-data-event-hubs.md).

---

**Q : Comment puis-je intégrer les journaux d’activité Azure AD à mon instance Splunk ?**

**R** : Tout d’abord, [acheminez les journaux d’activité Azure AD vers un Event Hub](./tutorial-azure-monitor-stream-logs-to-event-hub.md), puis suivez les étapes pour [intégrer des journaux d’activité avec Splunk](./howto-integrate-activity-logs-with-splunk.md).

---

**Q : Comment puis-je intégrer les journaux d’activité Azure AD avec Sumo Logic ?** 

**R** : Tout d’abord, [acheminez les journaux d’activité Azure AD à un Event Hub](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Collect_Logs_for_Azure_Active_Directory), puis suivez les étapes pour [installer l’application Azure AD et afficher les tableaux de bord dans SumoLogic](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards).

---

**Q : Puis-je accéder aux données d’un Event Hub sans avoir recours à un outil SIEM externe ?** 

**R** : Oui. Vous pouvez utiliser l’[API Event Hubs](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md) pour accéder aux journaux d’activité à partir de votre application personnalisée. 

---


## <a name="next-steps"></a>Étapes suivantes

* [Archiver les journaux d’activité dans un compte de stockage](quickstart-azure-monitor-route-logs-to-storage-account.md)
* [Acheminer les journaux d’activité vers un Event Hub](./tutorial-azure-monitor-stream-logs-to-event-hub.md)
* [Intégrer les journaux d’activité à Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md)