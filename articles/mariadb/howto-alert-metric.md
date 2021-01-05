---
title: Configurer des alertes de métriques - Portail Azure - Azure Database for MariaDB
description: Cet article décrit comment configurer et consulter des alertes Métrique pour Azure Database for MariaDB à partir du portail Azure.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 3/18/2020
ms.openlocfilehash: e108ca3aa346af20b31ff446130014e1250f49c0
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541077"
---
# <a name="use-the-azure-portal-to-set-up-alerts-on-metrics-for-azure-database-for-mariadb"></a>Utiliser le portail Azure pour configurer des alertes sur des métriques pour Azure Database for MariaDB

Cet article explique comment configurer des alertes Azure Database pour MariaDB à l’aide du portail Azure. Vous pouvez recevoir une alerte basée sur des métriques de surveillance pour vos services Azure.

L’alerte se déclenche quand la valeur d’une métrique spécifiée dépasse le seuil que vous avez défini. L’alerte se déclenche une première fois lorsque la condition est vérifiée, puis une deuxième fois quand elle cesse de l’être.

Vous pouvez configurer une alerte pour effectuer les actions suivantes lors de son déclenchement :
* Envoyer des notifications par e-mail à l’administrateur du service et aux coadministrateurs
* Envoyer un e-mail à d’autres adresses que vous spécifiez
* Appeler un webhook

Vous pouvez configurer et obtenir des informations sur les règles d’alerte à l’aide des ressources suivantes :
* [Azure portal](../azure-monitor/platform/alerts-metric.md#create-with-azure-portal)
* [Azure CLI](../azure-monitor/platform/alerts-metric.md#with-azure-cli)
* [API REST Azure Monitor](/rest/api/monitor/metricalerts)

## <a name="create-an-alert-rule-on-a-metric"></a>Créer une règle d'alerte d'une métrique
1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez le serveur Azure Database pour MariaDB à surveiller.

2. Sous la section **Surveillance** de la barre latérale, sélectionnez **Alertes**, comme illustré :

   ![Sélectionner des règles d’alerte](./media/howto-alert-metric/2-alert-rules.png)

3. Sélectionnez **Ajouter une alerte Métrique** (icône +).

4. La page **Créer une règle** s’ouvre, comme illustré ci-dessous. Entrez les informations obligatoires :

   ![Formulaire Ajouter une alerte Métrique](./media/howto-alert-metric/4-add-rule-form.png)

5. Dans la section **Condition**, sélectionnez **Ajouter une condition**.

6. Sélectionnez une métrique dans la liste des signaux d'alerte. Dans cet exemple, sélectionnez « Storage percent ».
   
   ![Sélectionner la métrique](./media/howto-alert-metric/6-configure-signal-logic.png)

7. Configurez la logique d’alerte, notamment les éléments **Condition** (par exemple, « Supérieur à »), **Seuil** (par exemple, 85 %), **Agrégation de temps**, **Période** de temps pendant laquelle la règle de métrique doit être satisfaite pour que l’alerte se déclenche (par exemple, « Au cours des 30 dernières minutes ») et **Frequency**.
   
   Sélectionnez **Terminé** lorsque vous avez terminé.

   ![Sélectionner la métrique 2](./media/howto-alert-metric/7-set-threshold-time.png)

8. Dans la section **Groupes d’actions**, sélectionnez **Créer un nouveau** pour créer un nouveau groupe afin de recevoir des notifications sur l’alerte.

9. Renseignez le formulaire « Ajouter un groupe d'actions » avec un nom, un nom court, un abonnement et un groupe de ressources.

10. Configurez un type d'action **E-mail/SMS/Push/Voix**.
    
    Choisissez « Envoyer un e-mail au rôle Azure Resource Manager » pour sélectionner les propriétaires de l'abonnement, les contributeurs et les lecteurs qui recevront les notifications.
   
    Vous pouvez aussi fournir un URI valide dans le champ **Webhook** si vous souhaitez qu’il soit appelé lorsque l’alerte se déclenche.

    Sélectionnez **OK** lorsque vous avez terminé.

    ![Groupe d’actions](./media/howto-alert-metric/10-action-group-type.png)

11. Spécifiez un nom de règle d’alerte, une description et une gravité.

    ![Groupe d’actions 2](./media/howto-alert-metric/11-name-description-severity.png) 

12. Sélectionnez **Créer une règle d’alerte** pour créer l’alerte.

    Après quelques minutes, l’alerte est active et se déclenche comme décrit précédemment.

## <a name="manage-your-alerts"></a>Gérez vos alertes
Une fois que vous avez créé une alerte, vous pouvez la sélectionner et exécuter les actions suivantes :

* Afficher un graphique indiquant le seuil de la métrique et les valeurs réelles du jour précédent concernant cette alerte.
* **Modifiez** ou **Supprimez** la règle d’alerte.
* **Désactivez** ou **Activez** l’alerte, selon que vous voulez arrêter temporairement ou reprendre la réception de notifications.


## <a name="next-steps"></a>Étapes suivantes
* Découvrez plus en détail la [configuration des webhooks dans les alertes](../azure-monitor/platform/alerts-webhooks.md).
* Consultez une [vue d’ensemble de la collecte des métriques](../azure-monitor/platform/data-platform.md) pour vous assurer que votre service est disponible et réactif.