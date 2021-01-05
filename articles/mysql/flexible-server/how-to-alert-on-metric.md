---
title: Configurer des alertes de métriques - Portail Azure - Azure Database pour MySQL - Serveur flexible
description: Cet article explique comment configurer et consulter des alertes de métriques pour le serveur flexible Azure Database pour MySQL à partir du portail Azure.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 4a099a9850289a046435b4e1763d7f54a702c0d0
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545087"
---
# <a name="use-the-azure-portal-to-set-up-alerts-on-metrics-for-azure-database-for-mysql---flexible-server"></a>Utiliser le portail Azure pour configurer des alertes sur des métriques pour Azure Database pour MySQL - Serveur flexible 

> [!IMPORTANT] 
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Cet article explique comment configurer des alertes Azure Database pour MySQL à l’aide du portail Azure. Vous pouvez recevoir une alerte basée sur des métriques de surveillance pour vos services Azure.

L’alerte se déclenche quand la valeur d’une métrique spécifiée dépasse le seuil que vous avez défini. L’alerte se déclenche une première fois lorsque la condition est vérifiée, puis une deuxième fois quand elle cesse de l’être. Les alertes de métrique sont avec état ; autrement dit, elles envoient uniquement des notifications lorsque l’état change.

Vous pouvez configurer une alerte pour effectuer les actions suivantes lors de son déclenchement :
* Envoyer des notifications par e-mail à l’administrateur du service et aux coadministrateurs
* Envoyer un e-mail à d’autres adresses que vous spécifiez
* Appeler un webhook

Vous pouvez configurer et obtenir des informations sur les règles d’alerte à l’aide des ressources suivantes :
* [Azure portal](../../azure-monitor/platform/alerts-metric.md#create-with-azure-portal)
* [Azure CLI](../../azure-monitor/platform/alerts-metric.md#with-azure-cli)
* [API REST Azure Monitor](/rest/api/monitor/metricalerts)

## <a name="create-an-alert-rule-on-a-metric-from-the-azure-portal"></a>Créer une règle d’alerte sur une métrique à partir du portail Azure
1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez le serveur flexible Azure Database pour MySQL à superviser.
2. Sous la section **Supervision** de la barre latérale, sélectionnez **Alertes** .
3. Sélectionnez **+ Nouvelle règle d’alerte** .
4. La page **Créer une règle** s’ouvre. Entrez les informations obligatoires :
5. Dans la section **Condition** , choisissez **Sélectionner la condition** .
6. Vous voyez alors la liste des signaux pris en charge. Sélectionnez la métrique sur laquelle vous voulez créer une alerte. Dans cet exemple, sélectionnez « Pourcentage de stockage ».
7. Vous verrez un graphique de la métrique pendant les six dernières heures. Utilisez la liste déroulante **Période du graphique** pour voir un historique plus long de la métrique.
8. Sélectionnez le **type de seuil** (par exemple, « Statique » ou « Dynamique »), l’ **opérateur** (par exemple, « Supérieur à ») et le **type d’agrégation** (par exemple, moyenne). Cela permet de déterminer la logique que la règle d’alerte de métrique évaluera.
    - Si vous utilisez un seuil **Statique** , définissez une **Valeur du seuil** (par exemple, 85 pour cent). Le graphique de métrique peut aider à déterminer un seuil raisonnable.
    - Si vous utilisez un seuil **Dynamique** , définissez la **Sensibilité du seuil** . Le graphique de métriques affiche les seuils calculés en fonction des données récentes. [Découvrez-en plus sur le type de condition des seuils dynamiques et les options de sensibilité](../../azure-monitor/platform/alerts-dynamic-thresholds.md).
9. Affinez la condition en réglant l’intervalle **Précision d’agrégation (période)** sur lequel les points de données sont regroupés à l’aide de la fonction de type d’agrégation (par exemple, « 30 minutes ») et la **fréquence** (par exemple, « Toutes les 15 minutes »).
10. Cliquez sur **Done** .
11. Ajoutez un groupe d’actions. Un groupe d’actions est une collection de préférences de notification définies par le propriétaire d’un abonnement Azure. Dans la section **Groupes d’actions** , choisissez **Sélectionner un groupe d’actions** pour sélectionner un groupe d’actions existant à associer à la règle d’alerte.
12. Vous pouvez aussi créer un groupe d’actions pour recevoir des notifications sur l’alerte. Pour plus d’informations, consultez [Créer et gérer un groupe d’actions](../../azure-monitor/platform/action-groups.md).
13. Pour créer un groupe d’actions, choisissez **+ Créer un groupe d’actions** . Renseignez le formulaire « Créer un groupe d’actions » avec un **abonnement** , un **groupe de ressources** , un **nom de groupe d’actions** et un **nom d’affichage** .
14. Configurez des **notifications** pour le groupe d’actions.
    
    Dans **Type de notification** , choisissez « Envoyer un e-mail au rôle Azure Resource Manager » pour sélectionner les propriétaires de l’abonnement, les contributeurs et les lecteurs qui recevront les notifications. Choisissez le **rôle Azure Resource Manager** pour envoyer l’e-mail.
    Vous pouvez également choisir **E-mail/SMS/Push/Message vocal** pour envoyer des notifications à des destinataires spécifiques.

    Indiquez le **nom** du type de notification et sélectionnez **Vérifier + créer** quand vous avez terminé.

    <!--:::image type="content" source="./media/howto-alert-on-metric/10-action-group-type.png" alt-text="Action group":::-->
    
15. Renseignez les **détails de la règle d’alerte** comme le **nom de la règle d’alerte** , la **description** , l’option **Enregistrer la règle d’alerte dans le groupe de ressources** et la **gravité** .

    <!--:::image type="content" source="./media/howto-alert-on-metric/11-name-description-severity.png" alt-text="Action group":::-->

16. Sélectionnez **Créer une règle d’alerte** pour créer l’alerte.
    Après quelques minutes, l’alerte est active et se déclenche comme décrit précédemment.
## <a name="manage-your-alerts"></a>Gérez vos alertes
Une fois que vous avez créé une alerte, vous pouvez la sélectionner et exécuter les actions suivantes :

* Afficher un graphique indiquant le seuil de la métrique et les valeurs réelles du jour précédent concernant cette alerte.
* **Modifiez** ou **Supprimez** la règle d’alerte.
* **Désactivez** ou **Activez** l’alerte, selon que vous voulez arrêter temporairement ou reprendre la réception de notifications.


## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur la [définition d’une alerte sur des métriques](../../azure-monitor/platform/alerts-metric.md).
- En savoir plus sur les [métriques disponibles sur le serveur flexible Azure Database pour MySQL](./concepts-monitoring.md).
- [Comprendre le fonctionnement des alertes de métrique dans Azure Monitor](../../azure-monitor/platform/alerts-metric-overview.md)