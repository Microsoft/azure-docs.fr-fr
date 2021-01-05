---
title: Créer des alertes pour Azure Cosmos DB à l’aide d’Azure Monitor
description: Découvrez comment configurer des alertes pour Azure Cosmos DB à l’aide d’Azure Monitor.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.devlang: dotnet
ms.topic: how-to
ms.date: 07/16/2020
ms.openlocfilehash: e29db7e31438bc7f6ac609384d0d9b92c275e813
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339544"
---
# <a name="create-alerts-for-azure-cosmos-db-using-azure-monitor"></a>Créer des alertes pour Azure Cosmos DB à l’aide d’Azure Monitor
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Les alertes sont utilisées pour configurer des tests périodiques afin de surveiller la disponibilité et la réactivité de vos ressources de Azure Cosmos DB. Les alertes peuvent vous envoyer une notification sous la forme d’un message électronique ou exécuter une fonction Azure quand l’une de vos métriques atteint le seuil ou si un événement spécifique est consigné dans le journal d’activité.

Vous pouvez recevoir une alerte en fonction des métriques ou des événements du journal d’activité sur votre compte Azure Cosmos :

* **Métriques** - L’alerte se déclenche quand la valeur d’une métrique spécifiée dépasse le seuil que vous avez défini. Par exemple, lorsque le nombre total d’unités de requête consommées dépasse 1000 RU/s. Cette alerte se déclenche une première fois lorsque la condition est vérifiée, puis une deuxième fois quand elle cesse de l’être. Pour connaître les différentes mesures disponibles dans Azure Cosmos DB, consultez l’article de [référence sur la surveillance des données](monitor-cosmos-db-reference.md#metrics).

* **Événements du journal d'activité** : cette alerte se déclenche lorsqu’un événement particulier se produit. Par exemple, lors de l’accès ou de l’actualisation des clés de votre compte Azure Cosmos.

Vous pouvez configurer des alertes à partir du volet Azure Cosmos DB ou du service Azure Monitor dans le portail Azure. Les deux interfaces offrent les mêmes options. Cet article vous montre comment configurer des alertes pour Azure Cosmos DB à l’aide d’Azure Monitor.

## <a name="create-an-alert-rule"></a>Création d'une règle d'alerte

Cette section explique comment créer une alerte lorsque vous recevez un code d’état HTTP 429, qui est reçu quand les requêtes ont une fréquence limitée. Par exemple, vous souhaiterez peut-être recevoir une alerte quand il y a 100 requêtes limitées ou plus. Cet article explique comment configurer une alerte pour un tel scénario à l’aide du code d’état HTTP. Vous pouvez utiliser les mêmes étapes pour configurer d’autres types d’alertes ; il vous suffit de choisir une condition différente en fonction de vos besoins.

1. Connectez-vous au [portail Azure.](https://portal.azure.com/)

1. Sélectionnez **Surveillance** dans la barre de navigation gauche, puis sélectionnez **Alertes**.

1. Sélectionnez le bouton Nouvelle règle d’alerte pour ouvrir la page Créer une règle d’alerte.  

1. Renseignez la section **Étendue** :

   * Ouvrez le volet **Sélectionner une ressource** et configurez les éléments suivants :

   * Choisir votre nom d’ **abonnement**.

   * Sélectionnez **Comptes Azure Cosmos DB** pour le **type de ressource**.

   * L’ **emplacement** de votre compte Azure Cosmos.

   * Une fois les détails renseignés, une liste de comptes Azure Cosmos dans l’étendue sélectionnée s’affiche. Choisissez celui pour lequel vous souhaitez configurer des alertes et sélectionnez **Terminé**.

1. Remplissez la section **Condition**  :

   * Ouvrez le volet **Sélectionner une condition** pour ouvrir la page **Configurer la logique du signal** , et configurez les éléments suivants :

   * Sélectionnez un signal. Le **Type de signal** peut être **Métrique** ou **Journal d’activité**. Choisissez **Métrique** pour ce scénario. Vous souhaitez recevoir une alerte en cas de limitation du débit sur la métrique d’unités de requête totales.

   * Sélectionnez **Tout** pour **Service de surveillance**

   * Choisissez un **Nom de signal**. Pour obtenir une alerte pour les codes d’état HTTP, choisissez le signal **Unités de requête totales**.

   * Dans l’onglet suivant, vous pouvez définir la logique de déclenchement d’une alerte et utiliser le graphique pour afficher les tendances de votre compte Azure Cosmos. La métrique **Unités de requête totales** prend en charge les dimensions. Ces dimensions vous permettent de filtrer sur la métrique. Si vous ne sélectionnez aucune dimension, cette valeur est ignorée.

   * Choisissez **StatusCode** comme **Nom de dimension**. Sélectionnez **Ajouter une valeur personnalisée** et définissez le code d’état sur 429.

   * Dans le **Logique d'alerte** , définissez le **Seuil** sur **Statique**. Le seuil statique utilise une valeur de seuil définie par l’utilisateur pour évaluer la règle, tandis que les seuils dynamiques utilisent des algorithmes de Machine Learning intégrés pour apprendre continuellement le modèle de comportement de la métrique et calculer les seuils automatiquement.

   * Définissez l’ **opérateur** sur **Supérieur à** , le **Type d’agrégation** sur **Total** et la **Valeur de seuil** sur **100**. Avec cette logique, si votre client voit plus de 100 demandes avec un code d’état 429, l’alerte est déclenchée. Vous pouvez également configurer le type d’agrégation, la granularité de l’agrégation et la fréquence d’évaluation en fonction de vos besoins.

   * Après avoir rempli le formulaire, sélectionnez **Terminé**. La capture d’écran suivante montre les détails de la logique d’alerte :

     :::image type="content" source="./media/create-alerts/configure-alert-logic.png" alt-text="Configurer la logique de réception des alertes pour les requêtes limitées en fréquence/429":::

1. Renseignez la section **Groupe d’actions** :

   * Sur le volet **Créer une règle** , sélectionnez un groupe d'actions existant, ou créez un nouveau groupe d’actions. Un groupe d’actions vous permet de définir l’action à exécuter lorsqu’une condition d’alerte se produit. Pour cet exemple, créez un nouveau groupe d’actions pour recevoir une notification par courrier électronique lorsque l’alerte est déclenchée. Ouvrez le volet **Ajouter un groupe d’actions** et renseignez les informations suivantes :

   * **Nom du groupe d’actions**  : le nom du groupe d’actions doit être unique au sein d’un groupe de ressources.

   * **Nom court**  : le nom abrégé du groupe d’actions ; cette valeur est incluse dans les notifications par courrier électronique et SMS pour identifier le groupe d’actions qui était la source de la notification.

   * Choisissez l’abonnement et le groupe de ressources dans lequel ce groupe d’actions sera créé.  

   * Donnez un nom à votre action et sélectionnez **Message électronique/SMS/Push/Message vocal** comme **Type d’action**. La capture d’écran suivante montre les détails du type d’action :

     :::image type="content" source="./media/create-alerts/configure-alert-action-type.png" alt-text="Configurer le type d’action, par exemple notification par courrier électronique, pour recevoir l’alerte":::

1. Renseignez la section **Détails de la règle d’alerte**  :

   * Définissez le nom de la règle, fournissez une description facultative, choisissez un niveau de gravité pour la règle, indiquez si vous souhaitez activer la règle lors de la création de la règle, puis sélectionnez **Créer une alerte de règle** pour créer l’alerte de règle de la métrique.

Une fois l’alerte créée, elle est active dans les 10 minutes.

## <a name="common-alerting-scenarios"></a>Scénarios d’alerte courants

Voici quelques scénarios dans lesquels vous pouvez utiliser les alertes :

* Lorsque les clés d’un compte Azure Cosmos sont mises à jour.
* Lorsque l’utilisation de données ou d’index d’un conteneur, d’une base de données ou d’une région dépasse un certain nombre d’octets.
* Lorsque la consommation de RU/s normalisée est supérieure à un certain pourcentage. La mesure de consommation en RU normalisée donne l’utilisation maximale du débit au sein d’un jeu de réplicas. Pour en savoir plus, consultez l’article [Comment surveiller les RU/s normalisées](monitor-normalized-request-units.md).  
* Quand une région est ajoutée, supprimée ou passe hors connexion.
* Lorsqu’une base de données ou un conteneur est créé, supprimé ou mis à jour.
* Lorsque le débit de votre base de données ou conteneur est modifié.

## <a name="next-steps"></a>Étapes suivantes

* Comment [surveiller la métrique RU/s normalisées](monitor-normalized-request-units.md) dans un conteneur Azure Cosmos.
* Comment [surveiller le débit ou l’utilisation des unités de requête d’une opération](monitor-request-unit-usage.md) dans Azure Cosmos DB.