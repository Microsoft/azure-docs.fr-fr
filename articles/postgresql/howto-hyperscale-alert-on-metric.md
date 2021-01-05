---
title: Configurer des alertes – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Cet article décrit comment configurer des alertes de métriques pour Azure Database pour PostgreSQL – Hyperscale (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: how-to
ms.date: 3/16/2020
ms.openlocfilehash: 73705434aef3ee438c02fbfd6502d30e7620b695
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026452"
---
# <a name="use-the-azure-portal-to-set-up-alerts-on-metrics-for-azure-database-for-postgresql---hyperscale-citus"></a>Utiliser le portail Azure pour configurer des alertes de métriques pour Azure Database pour PostgreSQL – Hyperscale (Citus)

Cet article vous explique comment définir des alertes Azure Database pour PostgreSQL à l’aide du portail Azure. Vous pouvez recevoir une alerte basée sur des [métriques de supervision](concepts-hyperscale-monitoring.md) pour vos services Azure.

Nous allons configurer une alerte qui se déclenche quand la valeur d’une métrique spécifiée dépasse un seuil. L’alerte se déclenche quand la condition est remplie pour la première fois et continue à se déclencher par la suite.

Vous pouvez configurer une alerte pour effectuer les actions suivantes lors de son déclenchement :
* Envoyer des notifications par e-mail à l’administrateur de service et aux coadministrateurs.
* Envoyer un e-mail à d’autres adresses que vous spécifiez
* Appeler un webhook.

Vous pouvez configurer et obtenir des informations sur les règles d’alerte à l’aide des ressources suivantes :
* [Azure portal](../azure-monitor/platform/alerts-metric.md#create-with-azure-portal)
* [Azure CLI](../azure-monitor/platform/alerts-metric.md#with-azure-cli)
* [API REST Azure Monitor](/rest/api/monitor/metricalerts)

## <a name="create-an-alert-rule-on-a-metric-from-the-azure-portal"></a>Créer une règle d’alerte sur une métrique à partir du portail Azure
1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez le serveur Azure Database pour PostgreSQL à surveiller.

2. Sous la section **Surveillance** de la barre latérale, sélectionnez **Alertes**, comme illustré :

   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/2-alert-rules.png" alt-text="Sélectionner des règles d’alerte":::

3. Sélectionnez **Nouvelle règle d’alerte** (icône +).

4. La page **Créer une règle** s’ouvre, comme illustré ci-dessous. Entrez les informations obligatoires :

   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/4-add-rule-form.png" alt-text="Formulaire Ajouter une alerte Métrique":::

5. Dans la section **Condition**, sélectionnez **Ajouter**.

6. Sélectionnez une métrique dans la liste des signaux d'alerte. Dans cet exemple, sélectionnez « Storage percent ».
   
   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/6-configure-signal-logic.png" alt-text="Capture d’écran de la page Configurer la logique du signal sur laquelle apparaissent plusieurs signaux":::

7. Configurez la logique d’alerte :

    * **Opérateur** (par exemple, « Supérieur à »)
    * **Valeur de seuil** (par exemple, 85 pour cent)
    * **Précision d’agrégation**, soit le temps pendant lequel la règle de métrique doit être satisfaite pour que l’alerte se déclenche (par exemple, « Au cours des 30 dernières minutes »)
    * et **Fréquence d’évaluation** (par exemple, « 1 minute »)
   
   Sélectionnez **Terminé** lorsque vous avez terminé.

   :::image type="content" source="./media/howto-hyperscale-alert-on-metric/7-set-threshold-time.png" alt-text="Capture d’écran du volet permettant de configurer la logique des alertes":::

8. Dans la section **Groupes d’actions**, sélectionnez **Créer un nouveau** pour créer un nouveau groupe afin de recevoir des notifications sur l’alerte.

9. Renseignez le formulaire « Ajouter un groupe d'actions » avec un nom, un nom court, un abonnement et un groupe de ressources.

    :::image type="content" source="./media/howto-hyperscale-alert-on-metric/9-add-action-group.png" alt-text="Capture d’écran du formulaire Ajouter un groupe d’actions permettant d’entrer les valeurs décrites":::

10. Configurez un type d'action **E-mail/SMS/Push/Voix**.
    
    Choisissez « Envoyer un e-mail au rôle Azure Resource Manager » pour envoyer des notifications aux propriétaires, aux contributeurs et aux lecteurs de l’abonnement.
   
    Sélectionnez **OK** lorsque vous avez terminé.

    :::image type="content" source="./media/howto-hyperscale-alert-on-metric/10-action-group-type.png" alt-text="Capture d’écran du volet E-mail/SMS/Push/Voix":::

11. Spécifiez un nom de règle d’alerte, une description et une gravité.

    :::image type="content" source="./media/howto-hyperscale-alert-on-metric/11-name-description-severity.png" alt-text="Capture d’écran du volet Détails de l’alerte"::: 

12. Sélectionnez **Créer une règle d’alerte** pour créer l’alerte.

    Après quelques minutes, l’alerte est active et se déclenche comme décrit précédemment.

### <a name="managing-alerts"></a>Gestion des alertes

Après avoir créé une alerte, vous pouvez la sélectionner et exécuter les actions suivantes :

* Afficher un graphique indiquant le seuil de la métrique et les valeurs réelles du jour précédent concernant cette alerte.
* **Modifiez** ou **Supprimez** la règle d’alerte.
* **Désactivez** ou **Activez** l’alerte, selon que vous voulez arrêter temporairement ou reprendre la réception de notifications.

## <a name="suggested-alerts"></a>Alertes suggérées

### <a name="disk-space"></a>Espace disque

La supervision et les alertes sont importantes pour chaque groupe de serveurs de production Hyperscale (Citus). La base de données PostgreSQL sous-jacente nécessite un espace disque libre pour fonctionner correctement. Si le disque est saturé, le nœud du serveur de base de données est mis hors connexion et refuse de démarrer jusqu’à ce que de l’espace soit disponible. À ce stade, une demande de support Microsoft est requise pour résoudre le problème.

Nous vous recommandons de définir des alertes d’espace disque sur chaque nœud de chaque groupe de serveurs, même pour une utilisation hors production. Les alertes d’utilisation de l’espace disque fournissent l’avertissement préalable nécessaire à une intervention et au maintien de l’intégrité des nœuds. Pour de meilleurs résultats, essayez une série d’alertes à 75 %, 85 % et 95 % d’utilisation. Les pourcentages à choisir dépendent de la vitesse d’ingestion des données, étant donné qu’une ingestion des données rapide remplit le disque plus rapidement.

À mesure que le disque approche de sa limite d’espace, essayez ces techniques pour obtenir davantage d’espace libre :

* Examinez la stratégie de conservation des données. Déplacez les données les plus anciennes vers le stockage froid si possible.
* Envisagez d’[ajouter des nœuds](howto-hyperscale-scale-grow.md#add-worker-nodes) au groupe de serveurs et de rééquilibrer les partitions. Le rééquilibrage répartit les données sur d’autres ordinateurs.
* Envisagez d’[augmenter la capacité](howto-hyperscale-scale-grow.md#increase-or-decrease-vcores-on-nodes) des nœuds worker. Chaque worker peut contenir jusqu’à 2 Tio de stockage. Toutefois, l’ajout de nœuds doit être tenté avant le redimensionnement des nœuds, car l’ajout de nœuds s’effectue plus rapidement.

### <a name="cpu-usage"></a>Utilisation de l’UC

La supervision de l’utilisation du processeur est utile pour établir une base de référence pour les performances. Par exemple, vous remarquerez peut-être que l’utilisation du processeur est généralement d’environ 40 à 60 %. Si l’utilisation du processeur commence soudainement à tourner autour de 95 %, vous pouvez identifier une anomalie. L’utilisation du processeur peut refléter une croissance naturelle, mais elle peut également révéler une requête isolée. Lors de la création d’une alerte de processeur, définissez une précision d’agrégation longue pour intercepter les augmentations prolongées et ignorer les pics momentanés.

## <a name="next-steps"></a>Étapes suivantes
* Découvrez plus en détail la [configuration des webhooks dans les alertes](../azure-monitor/platform/alerts-webhooks.md).
* Consultez une [vue d’ensemble de la collecte des métriques](../azure-monitor/platform/data-platform.md) pour vous assurer que votre service est disponible et réactif.
