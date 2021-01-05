---
title: Utiliser Azure Time Series Insights pour stocker et analyser la télémétrie des appareils Azure IoT Plug-and-Play
description: Configurez un environnement Time Series Insights et connectez votre hub IoT pour afficher et analyser la télémétrie de vos appareils IoT Plug-and-Play.
author: lyrana
ms.author: lyhughes
ms.date: 10/14/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 5491df61a1198e8eee4ba4701ccfc56154ec75eb
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905081"
---
# <a name="preview-tutorial-create-and-configure-a-time-series-insights-gen2-environment"></a>Tutoriel de la préversion : Créer et configurer un environnement Time Series Insights Gen2

Dans ce tutoriel, vous apprenez à créer et à configurer un environnement [Azure Time Series Insights Gen2](../time-series-insights/overview-what-is-tsi.md) en vue de son intégration à votre solution IoT Plug-and-Play. Utilisez Time Series Insights pour collecter, traiter, stocker, interroger et visualiser des données de série chronologique à l’échelle IoT (Internet des objets).

Tout d’abord, vous provisionnez un environnement Time Series Insights et vous connectez votre hub IoT en tant que source d’événements de streaming. Vous utilisez ensuite la synchronisation de modèle pour créer votre [modèle de série chronologique](../time-series-insights/concepts-model-overview.md). Vous reprenez les exemples de fichiers de modèle [DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl) que vous avez utilisés pour les appareils à thermostat et de contrôle de température.

> [!NOTE]
> Cette intégration entre Time Series Insights et IoT Plug-and-Play est en préversion. La façon dont les modèles d’appareil DTDL sont mappés au modèle de série chronologique Time Series Insights est susceptible de changer. 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [iot-pnp-prerequisites](../../includes/iot-pnp-prerequisites.md)]

À ce stade, vous disposez des éléments suivants :

* Un IoT Hub Azure.
* Une instance DPS (Device Provisioning Service) liée à votre hub IoT. L’instance DPS doit avoir une inscription d’appareil individuel pour votre appareil IoT Plug-and-Play.
* Une connexion à votre hub IoT à partir d’un appareil à un seul composant ou à plusieurs composants, qui transmet en streaming les données simulées.

Pour éviter d’avoir à installer Azure CLI localement, vous pouvez configurer les services cloud à l’aide d’Azure Cloud Shell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prepare-your-event-source"></a>Préparer votre source d’événements

Le hub IoT que vous avez créé fera office de [source d’événements](../time-series-insights/concepts-streaming-ingestion-event-sources.md) dans votre environnement Time Series Insights.

> [!IMPORTANT]
> Désactivez les routes IoT Hub existants. Il existe un problème connu quand un hub IoT est utilisé avec le [routage](../iot-hub/iot-hub-devguide-messages-d2c.md#routing-endpoints) configuré. Désactivez temporairement tous les points de terminaison de routage. Quand votre hub IoT est connecté à Time Series Insights, vous pouvez les réactiver.

Sur votre hub IoT, créez un groupe de consommateurs unique qui fournira les informations à Time Series Insights. Dans l’exemple suivant, remplacez `my-pnp-hub` par le nom du hub IoT que vous avez utilisé précédemment.

```azurecli-interactive
az iot hub consumer-group create --hub-name my-pnp-hub --name tsi-consumer-group
```

## <a name="choose-a-time-series-id"></a>Choisir un ID Time Series

Quand vous provisionnez votre environnement Time Series Insights, vous devez sélectionner un *ID de série chronologique*. Il est important de sélectionner l’ID de série chronologique correct. Cette propriété est immuable et ne peut plus être modifiée une fois définie. Un ID de série chronologique est semblable à une clé de partition de base de données. L’ID de série chronologique fait office de clé primaire pour votre modèle de série chronologique. Pour plus d’informations, consultez [Meilleures pratiques pour le choix d’un ID Time Series](../time-series-insights/how-to-select-tsid.md).

En tant qu’utilisateur IoT Plug-and-Play, pour votre ID de série chronologique, spécifiez une _clé composite_ constituée des éléments `iothub-connection-device-id` et `dt-subject`. Le hub IoT ajoute ces propriétés système, qui contiennent respectivement votre ID d’appareil IoT Plug-and-Play et les noms de vos composants d’appareil.

Même si vos modèles d’appareil IoT Plug-and-Play n’utilisent pas encore de composants, vous devez inclure `dt-subject` dans une clé composite afin de pouvoir utiliser les composants à l’avenir. Votre ID de série chronologique étant immuable, Microsoft recommande d’activer cette option au cas où vous en auriez besoin.

> [!NOTE]
> Les exemples de cet article concernent l’appareil `TemperatureController` à plusieurs composants. Les concepts sont les mêmes pour l’appareil `Thermostat` sans composant.

## <a name="provision-your-time-series-insights-environment"></a>Provisionner votre environnement Time Series Insights

Cette section explique comment provisionner votre environnement Azure Time Series Insights Gen2.

Exécutez la commande suivante pour :

* Créer un compte de stockage Azure pour le [magasin de stockage froid](../time-series-insights/concepts-storage.md#cold-store) de votre environnement. Ce compte sert à la conservation à long terme et à l’analyse des données historiques.
  * Dans votre code, remplacez `mytsicoldstore` par un nom unique pour votre compte de stockage froid.
* Créer un environnement Azure Time Series Insights Gen2. L’environnement sera créé avec un stockage chaud ayant une période de conservation de sept jours. Le compte de stockage froid sera attaché pour une conservation infinie.
  * Dans votre code, remplacez `my-tsi-env` par un nom unique pour votre environnement Time Series Insights.
  * Dans votre code, remplacez `my-pnp-resourcegroup` par le nom du groupe de ressources que vous avez utilisé lors de la configuration.
  * La propriété de votre ID de série chronologique est `iothub-connection-device-id, dt-subject`.

```azurecli-interactive
storage=mytsicoldstore
rg=my-pnp-resourcegroup
az storage account create -g $rg -n $storage --https-only
key=$(az storage account keys list -g $rg -n $storage --query [0].value --output tsv)
az timeseriesinsights environment longterm create --name my-tsi-env --resource-group $rg --time-series-id-properties iothub-connection-device-id, dt-subject --sku-name L1 --sku-capacity 1 --data-retention 7 --storage-account-name $storage --storage-management-key $key --location eastus2
```

Connectez votre source d’événements IoT Hub. Remplacez `my-pnp-resourcegroup`, `my-pnp-hub` et `my-tsi-env` par les valeurs que vous avez choisies. La commande suivante référence le groupe de consommateurs pour Time Series Insights que vous avez créé :

```azurecli-interactive
rg=my-pnp-resourcegroup
iothub=my-pnp-hub
env=my-tsi-env
es_resource_id=$(az iot hub create -g $rg -n $iothub --query id --output tsv)
shared_access_key=$(az iot hub policy list -g $rg --hub-name $iothub --query "[?keyName=='service'].primaryKey" --output tsv)
az timeseriesinsights event-source iothub create -g $rg --environment-name $env -n iot-hub-event-source --consumer-group-name tsi-consumer-group  --key-name iothubowner --shared-access-key $shared_access_key --event-source-resource-id $es_resource_id
```

Dans le [portail Azure](https://portal.azure.com), accédez à votre groupe de ressources, puis sélectionnez votre nouvel environnement Time Series Insights. Accédez à l’**URL de l’Explorateur Time Series Insights** affichée dans la vue d’ensemble de l’instance :

![Capture d’écran de la page de vue d’ensemble du portail.](./media/tutorial-configure-tsi/view-environment.png)

Dans l’Explorateur, vous voyez trois instances :

* &lt;Votre ID d’appareil pnp&gt;, thermostat1
* &lt;Votre ID d’appareil pnp&gt;, thermostat2
* &lt;Votre ID d’appareil pnp&gt;, `null`

> [!NOTE]
> La troisième étiquette représente la télémétrie de `TemperatureController`, telle que la plage de travail de la mémoire de l’appareil. Étant donné qu’il s’agit d’une propriété de niveau supérieur, la valeur du nom du composant est null. Dans une étape ultérieure, vous rendrez ce nom plus convivial.

![Capture d’écran montrant trois instances dans l’Explorateur.](./media/tutorial-configure-tsi/tsi-env-first-view.png)

## <a name="configure-model-translation"></a>Configurer la traduction du modèle

Ensuite, vous traduisez votre modèle d’appareil DTDL en modèle de ressource dans Azure Time Series Insights. Dans Time Series Insights, le modèle Time Series est un outil de modélisation sémantique servant à la contextualisation des données. Le modèle comprend trois composants de base :

* Les [instances de modèle de série chronologique](../time-series-insights/concepts-model-overview.md#time-series-model-instances) sont des représentations virtuelles de la série chronologique elle-même. Les instances sont identifiées de manière unique par votre ID de série chronologique.
* Les [hiérarchies de modèle de série chronologique](../time-series-insights/concepts-model-overview.md#time-series-model-hierarchies) organisent les instances en spécifiant les noms des propriétés et leurs relations.
* Les [types de modèle de série chronologique](../time-series-insights/concepts-model-overview.md#time-series-model-types) vous aident à définir des [variables](../time-series-insights/concepts-variables.md) ou des formules pour des calculs. Les types sont associés à une instance spécifique.

### <a name="define-your-types"></a>Définir vos types

Vous pouvez commencer à ingérer des données dans Azure Time Series Insights Gen2 sans avoir prédéfini de modèle. Quand il reçoit les données de télémétrie, Time Series Insights tente de résoudre automatiquement les instances de série chronologique en fonction des valeurs des propriétés de votre ID de série chronologique. Le *type par défaut* est attribué à toutes les instances. Vous devez créer manuellement un type pour classer correctement vos instances. 

Les détails suivants indiquent la méthode la plus simple pour synchroniser vos modèles DTDL d’appareil avec vos types de modèles de série chronologique :

* Votre identificateur DTMI de modèle de jumeau numérique devient votre ID de type.
* Le nom du type peut être le nom du modèle ou le nom d’affichage.
* La description du modèle devient la description du type.
* Au moins une variable de type est créée pour chaque donnée de télémétrie ayant un schéma numérique.
  * Seuls les types de données numériques peuvent être utilisés comme variables, mais si une valeur est envoyée sous la forme d’un autre type pouvant être converti, par exemple `"0"`, vous pouvez utiliser une fonction de [conversion](/rest/api/time-series-insights/reference-time-series-expression-syntax#conversion-functions) telle que `toDouble`.
* Le nom de la variable peut être le nom de la télémétrie ou le nom d’affichage.
* Quand vous définissez la variable Time Series Expression, référencez le nom et le type de la donnée de télémétrie transmise.

| JSON DTDL | JSON du type de modèle de série chronologique | Valeur d'exemple |
|-----------|------------------|-------------|
| `@id` | `id` | `dtmi:com:example:TemperatureController;1` |
| `displayName`    | `name`   |   `Temperature Controller`  |
| `description`  |  `description`  |  `Device with two thermostats and remote reboot.` |  
|`contents` (tableau)| `variables` (objet)  | Consultez l’exemple qui suit.

![Capture d’écran montrant le type de modèle de série chronologique à DTDL.](./media/tutorial-configure-tsi/DTDL-to-TSM-Type.png)

> [!NOTE]
> Cet exemple montre trois variables, mais chaque type peut avoir jusqu’à 100 variables. Différentes variables peuvent référencer la même valeur de télémétrie pour faire différents calculs en fonction des besoins. Pour obtenir la liste complète des filtres, agrégats et fonctions scalaires, consultez la [syntaxe Time Series Expression dans Time Series Insights Gen2](/rest/api/time-series-insights/reference-time-series-expression-syntax).

Ouvrez un éditeur de texte et enregistrez le code JSON suivant sur votre disque local.

```JSON
{
  "put": [
    {
      "id": "dtmi:com:example:TemperatureController;1",
      "name": "Temperature Controller",
      "description": "Device with two thermostats and remote reboot.",
      "variables": {
        "workingSet": {
          "kind": "numeric",
          "value": {
            "tsx": "coalesce($event.workingSet.Long, toLong($event.workingSet.Double))"
          }, 
          "aggregation": {
            "tsx": "avg($value)"
          }
        },
        "temperature": {
          "kind": "numeric",
          "value": {
            "tsx": "coalesce($event.temperature.Long, toLong($event.temperature.Double))"
          },
          "aggregation": {
            "tsx": "avg($value)"
          }
        },
        "eventCount": {
          "kind": "aggregate",
          "aggregation": {
            "tsx": "count()"
          }
        }
      }
    }
  ]
}
```

Dans l’Explorateur Time Series Insights, sélectionnez l’icône de modèle sur la gauche pour ouvrir l’onglet **Modèle**. Sélectionnez **Types**, puis sélectionnez **Charger un fichier JSON** :

![Capture d’écran montrant comment charger JSON.](./media/tutorial-configure-tsi/upload-type.png)

Sélectionnez **Choisir un fichier**, sélectionnez le fichier JSON que vous avez enregistré, puis sélectionnez **Charger**.

Vous voyez le type **Temperature Controller** (Contrôleur de température) nouvellement défini.

### <a name="create-a-hierarchy"></a>Créer une hiérarchie

Créez une hiérarchie pour organiser les étiquettes sous leur parent `TemperatureController`. L’exemple simple suivant présente un seul niveau, mais les solutions IoT ont généralement de nombreux niveaux d’imbrication pour contextualiser les étiquettes dans leur position physique et sémantique au sein d’une organisation.

Sélectionnez **Hiérarchies**, puis sélectionnez **Ajouter une hiérarchie**. Pour le nom, entrez *Device Fleet*. Créez un niveau appelé *Device Name*. Ensuite, sélectionnez **Enregistrer**.

![Capture d’écran montrant comment ajouter une hiérarchie.](./media/tutorial-configure-tsi/add-hierarchy.png)

### <a name="assign-your-instances-to-the-correct-type"></a>Affecter vos instances au type correct

Ensuite, vous changez le type de vos instances et les placez dans la hiérarchie.

Sélectionnez l’onglet **Instances**. Recherchez l’instance qui représente la plage de travail de l’appareil, puis sélectionnez l’icône **Modifier** tout à droite.

![Capture d’écran montrant comment modifier une instance.](./media/tutorial-configure-tsi/edit-instance.png)

Ouvrez le menu déroulant **Type**, puis sélectionnez **Temperature Controller**. Entrez *defaultComponent, <your device name>* pour mettre à jour le nom de l’instance qui représente toutes les étiquettes de niveau supérieur associées à votre appareil.

![Capture d’écran montrant comment changer un type d’instance.](./media/tutorial-configure-tsi/change-type.png)

Avant de sélectionner **Enregistrer**, sélectionnez l’onglet **Champs d’instance**, puis sélectionnez **Device Fleet**. Pour regrouper les données de télémétrie, entrez *\<your device name> - Temp Controller*. Ensuite, sélectionnez **Enregistrer**.

![Capture d’écran montrant comment affecter une instance à une hiérarchie](./media/tutorial-configure-tsi/assign-to-hierarchy.png)

Répétez les étapes précédentes pour affecter à vos étiquettes de thermostat les type et hiérarchie appropriés.

## <a name="view-your-data"></a>Affichage de vos données

Revenez au volet de graphique et développez **Device Fleet > votre appareil**. Sélectionnez **thermostat1**, sélectionnez la variable **Temperature**, puis sélectionnez **Ajouter** pour représenter la valeur sous forme graphique. Procédez de la même façon pour **thermostat2** et la valeur **workingSet** **defaultComponent**.

![Capture d’écran montrant comment changer le type d’instance pour thermostat2.](./media/tutorial-configure-tsi/charting-values.png)

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les différentes options de graphique, notamment la taille des intervalles et les contrôles de l’axe Y, consultez [Explorateur Azure Time Series Insights](../time-series-insights/concepts-ux-panels.md).

* Pour obtenir une présentation détaillée du modèle de série chronologique de votre environnement, consultez [Modèle de série chronologique dans Azure Time Series Insights Gen2](../time-series-insights/concepts-model-overview.md).

* Pour explorer les API de requête et la syntaxe des expressions de série chronologique, consultez [API de requête Azure Time Series Insights Gen2](/rest/api/time-series-insights/reference-query-apis).
