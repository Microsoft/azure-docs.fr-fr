---
title: Visualiser la télémétrie simulée avec Time Series Insights - Azure | Microsoft Docs
description: Découvrez comment configurer votre environnement Time Series Insights pour explorer et analyser la télémétrie générée par l’accélérateur de solution Simulation d’appareil.
author: dominicbetts
manager: timlt
ms.author: dobett
ms.date: 08/20/2018
ms.topic: conceptual
ms.service: iot-accelerators
services: iot-accelerators
ms.openlocfilehash: e1409b43f0ce1fc0d8c622dda79e857ac6abdd33
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854561"
---
# <a name="use-time-series-insights-to-visualize-telemetry-sent-from-the-device-simulation-solution-accelerator"></a>Time Series Insights permet de visualiser la télémétrie envoyée par l’accélérateur de solution Simulation d’appareil

L’accélérateur de solution Simulation d’appareil vous permet de générer une télémétrie à partir d’appareils simulés pour tester vos solutions IoT. Ce guide vous montre comment visualiser et analyser la télémétrie simulée à l’aide d’un environnement Time Series Insights.

## <a name="prerequisites"></a>Conditions préalables requises

Pour suivre les étapes décrites dans ce guide, vous avez besoin d’un abonnement Azure actif. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Les étapes décrites dans ce guide partent du principe que vous avez déployé l’accélérateur de solution Simulation d’appareil dans votre abonnement Azure. Si vous n’avez pas encore déployé la simulation d’appareil, consultez [Déploiement de la simulation d’appareil](https://github.com/Azure/device-simulation-dotnet/blob/master/README.md) sur GitHub.

Cet article part du principe que le nom de votre accélérateur de solution est **contoso-simulation**. Dans les étapes suivantes, remplacez **contoso-simulation** par le nom de votre accélérateur de solution.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-consumer-group"></a>Créer un groupe de consommateurs

Vous devez créer un groupe de consommateurs dédié dans votre hub IoT pour permettre la transmission des données de télémétrie à Time Series Insights. Une source d’événement dans Time Series Insights doit avoir l’usage exclusif d’un groupe de consommateurs IoT Hub.

Les étapes suivantes utilisent l’interface Azure CLI dans Azure Cloud Shell pour créer le groupe de consommateurs :

1. Le hub IoT est l’une des ressources qui ont été créées quand vous avez déployé l’accélérateur de solution Simulation d’appareil. Exécutez la commande suivante pour obtenir le nom de votre hub IoT. N’oubliez pas de spécifier le nom de votre propre accélérateur de solution :

    ```azurecli-interactive
    az resource list --resource-group contoso-simulation -o table
    ```

    Le hub IoT est la ressource de type **Microsoft.Devices/IotHubs**.

1. Ajoutez au hub un groupe de consommateurs appelé **devicesimulationtsi**. Dans la commande suivante, utilisez le nom de votre hub et de votre accélérateur de solution :

    ```azurecli-interactive
    az iot hub consumer-group create --hub-name contoso-simulation7d894 --name devicesimulationtsi --resource-group contoso-simulation
    ```

    Vous pouvez maintenant fermer Azure Cloud Shell.

## <a name="create-a-new-time-series-insights-environment"></a>Créer un environnement Time Series Insights

[Azure Time Series Insights](../../articles/time-series-insights/time-series-insights-overview.md) est un service entièrement managé d’analytique, de stockage et de visualisation pour la gestion des données chronologiques IoT dans le cloud. Pour créer un environnement Time Series Insights :

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

1. Sélectionnez **Créer une ressource** > **Internet des objets** > **Time Series Insights** :

    ![Nouvel environnement Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/new-time-series-insights.png)

1. Pour créer votre environnement Time Series Insights dans le même groupe de ressources que votre accélérateur de solution, utilisez les valeurs du tableau ci-dessous :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom de l’environnement | La capture d’écran suivante utilise le nom **Contoso-TSI**. Choisissez votre propre nom unique quand vous effectuez cette étape. |
    | Subscription | Sélectionnez votre abonnement Azure dans la liste déroulante. |
    | Resource group | **contoso-simulation**. Utilisez le nom de votre accélérateur de solution. |
    | Location | Cet exemple utilise la région **USA Est**. Créez votre environnement dans la même région que votre accélérateur de solution Simulation d’appareil. |
    | Sku |**S1** |
    | Capacité | **1** |

    ![Créer un environnement Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/new-time-series-insights-create.png)

    > [!NOTE]
    > Si vous ajoutez l’environnement Time Series Insights au même groupe de ressources que l’accélérateur de solution, l’environnement est automatiquement supprimé en même temps que l’accélérateur de solution.

1. Cliquez sur **Créer**. La création de l’environnement peut prendre plusieurs minutes.

## <a name="create-event-source"></a>Créez la source d’événement

Créez une source d’événement à connecter à votre hub IoT. Utilisez le groupe de consommateurs que vous avez créé aux étapes précédentes. Une source d’événement Time Series Insights nécessite un groupe de consommateurs dédié qui ne soit pas déjà utilisé par un autre service.

1. Dans le portail Azure, accédez à votre nouvel environnement Time Series.

1. Sur la gauche, cliquez sur **Sources d’événements** :

    ![Afficher les sources d’événements](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-event-sources.png)

1. Cliquez sur **Ajouter** :

    ![Ajouter une source d’événement](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-event-sources-add.png)

1. Pour configurer votre hub IoT comme nouvelle source d’événement, utilisez les valeurs dans le tableau suivant :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom de la source d’événement | La capture d’écran suivante utilise le nom **contoso-iot-hub**. Utilisez votre propre nom unique quand vous effectuez cette étape. |
    | Source | **IoT Hub** |
    | Option d’importation | **Utiliser Iot Hub à partir des abonnements disponibles** |
    | Identifiant d’abonnement | Sélectionnez votre abonnement Azure dans la liste déroulante. |
    | Nom du hub IoT | **contoso-simulation7d894**. Utilisez le nom de votre hub IoT spécifié dans votre accélérateur de solution Simulation d’appareil. |
    | Nom de la stratégie du hub IoT | **iothubowner** |
    | Clé de stratégie du hub IoT | Ce champ est rempli automatiquement. |
    | Groupe de consommateurs du hub IoT | **devicesimulationtsi** |
    | Format de sérialisation de l’événement | **JSON** |
    | Nom de la propriété d’horodatage | Laisser vide |

    ![Créer une source d’événement](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-event-source-create.png)

1. Cliquez sur **Créer**.

> [!NOTE]
> Vous pouvez [accorder l’accès à des utilisateurs supplémentaires](../time-series-insights/concepts-access-policies.md#grant-data-access) à l’Explorateur Time Series Insights.

## <a name="start-a-simulation"></a>Démarrer une simulation

Avant d’utiliser l’Explorateur Time Series Insights, configurez l’accélérateur de solution Simulation d’appareil pour générer une télémétrie. La capture d’écran suivante présente une simulation en cours avec 10 appareils de type réfrigérateur :

![Exécution de simulation d’appareil](./media/iot-accelerators-device-simulation-time-series-insights/running-simulation.png)

## <a name="time-series-insights-explorer"></a>Explorateur Time Series Insights

L’Explorateur Time Series Insights est une application web que vous pouvez utiliser pour visualiser vos données de télémétrie.

1. Dans le portail Azure, sélectionnez l’onglet **Vue d’ensemble** de Time Series Insights.

1. Pour ouvrir l’application web Explorateur Time Series Insights, cliquez sur **Accéder à l’environnement** :

    ![Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-environment.png)

1. Dans le panneau de sélection des durées, dans le menu des durées courtes, sélectionnez **30 dernières minutes**, puis cliquez sur **Rechercher** :

    ![Recherche dans l’Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-search-time.png)

1. Dans le panneau des conditions à gauche, sélectionnez **Température** comme **Mesure** et **iothub-connection-device-id** comme valeur pour **Divisé par** :

    ![Capture d’écran montrant le panneau « conditions » de Time Series Insights, avec les valeurs « Mesure » et « Divisé par » mises en évidence.](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-query1.png)

1. Cliquez avec le bouton droit sur le graphique, puis sélectionnez **Explorer les événements**:

    ![Événements de l’Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-explore-events.png)

1. Les données d’événement s’affichent dans une grille :

    ![Tableau de l’Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-table.png)

1. Cliquez sur le bouton du mode perspective :

    ![Perspective de l’Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-explorer-perspective.png)

1. Cliquez sur **+** pour ajouter une nouvelle requête à la perspective :

    ![Ajouter une requête dans l’Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-new-query.png)

1. Sélectionnez **30 dernières minutes** comme intervalle de temps, **Humidité** comme **Mesure** et **iothub-connection-device-id** comme valeur pour **Divisé par** :

    ![Requête de l’Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-query2.png)

1. Cliquez sur le bouton du mode perspective pour voir le tableau de bord des données de télémétrie de votre appareil :

    ![Tableau de bord de l’Explorateur Time Series Insights](./media/iot-accelerators-device-simulation-time-series-insights/time-series-insights-dashboard.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de l’explorer davantage, laissez l’accélérateur de solution déployé.

Si vous n’avez plus besoin de l’accélérateur de solution, supprimez-le de la page [Solutions approvisionnées](https://www.azureiotsolutions.com/Accelerators#dashboard) en le sélectionnant et en cliquant sur **Supprimer la solution**.

Si vous avez ajouté l’environnement Time Series Insights à un groupe de ressources de l’accélérateur de solution, l’environnement est automatiquement supprimé en même temps que l’accélérateur de solution. Sinon, vous devez supprimer manuellement l’environnement Time Series Insights dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la façon d’explorer et d’interroger des données dans l’Explorateur Time Series Insights, voir [Explorateur Azure Time Series Insights](../time-series-insights/time-series-insights-explorer.md).