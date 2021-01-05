---
title: Créer un tableau de bord de triage des données médicales avec Azure IoT Central | Microsoft Docs
description: Apprenez à créer un tableau de bord de triage des données médicales à l'aide des modèles d'application Azure IoT Central.
author: philmea
ms.author: philmea
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: eliotgra
ms.openlocfilehash: 5fa74ce99c0ee00970b1f18d366f0b24c3fb498d
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590135"
---
# <a name="tutorial-build-a-power-bi-provider-dashboard"></a>Tutoriel : Créer un tableau de bord des fournisseurs Power BI

Lors de l’élaboration de votre solution de surveillance continue des patients, vous pouvez également créer un tableau de bord permettant à une équipe soignante de visualiser les données des patients. Dans ce tutoriel, vous allez apprendre à créer un tableau de bord de streaming en temps réel Power BI à partir de votre modèle d’application de surveillance continue des patients IoT Central. Si votre cas d’utilisation ne nécessite pas l’accès aux données en temps réel, vous pouvez utiliser le [tableau de bord Power BI IoT Central](../core/howto-connect-powerbi.md), qui présente un processus de déploiement simplifié. 

:::image type="content" source="media/dashboard-gif-3.gif" alt-text="GIF du tableau de bord":::

L'architecture de base suivra cette structure :

:::image type="content" source="media/dashboard-architecture.png" alt-text="Tableau de bord de triage des fournisseurs":::

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Exporter des données d'Azure IoT Central vers Azure Event Hubs
> * Configurer un jeu de données de streaming Power BI
> * Connecter votre application logique à Azure Event Hubs
> * Diffuser des données vers Power BI à partir de votre application logique
> * Créer un tableau de bord en temps réel pour les données vitales des patients


## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Un modèle d'application de surveillance continue des patients Azure IoT Central. Si vous n'en avez pas encore, vous pouvez suivre les étapes de [Déploiement d'un modèle d'application](overview-iot-central-healthcare.md).

* Un [espace de noms Azure Event Hubs et un hub d'événements](../../event-hubs/event-hubs-create.md).

* L'application logique à partir de laquelle vous souhaitez accéder à votre hub d'événements. Pour démarrer votre application logique avec un déclencheur Azure Event Hubs, vous devez disposer d'une [application logique vierge](../../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Un compte de service Power BI. Si vous n'en avez pas encore, vous pouvez [créer un compte d'essai gratuit pour le service Power BI](https://app.powerbi.com/). Si vous n'avez encore jamais utilisé Power BI, n'hésitez pas à consulter [Prise en main de Power BI](/power-bi/service-get-started).


## <a name="set-up-a-continuous-data-export-to-azure-event-hubs"></a>Configurer une exportation continue des données vers Azure Event Hubs
Vous devez d'abord configurer une exportation continue des données de votre modèle d'application Azure IoT Central vers Azure Event Hubs dans le cadre de votre abonnement. Pour ce faire, suivez les étapes d'[exportation vers Event Hubs](../core/howto-export-data.md) décrites dans ce didacticiel Azure IoT Central. Vous aurez uniquement besoin d'exporter les données de télémétrie aux fins de ce didacticiel.


## <a name="create-a-power-bi-streaming-dataset"></a>Créer un jeu de données de streaming Power BI

1. Connectez-vous à votre compte Power BI.

1. Dans votre espace de travail, créez un jeu de données de streaming en sélectionnant le bouton **+ Créer** en haut à droite de la barre d'outils. Vous devrez créer un jeu de données distinct pour chaque patient que vous souhaitez inclure dans votre tableau de bord.

   :::image type="content" source="media/create-streaming-dataset.png" alt-text="Créer un jeu de données de streaming":::


1. Choisissez **API** comme source de votre jeu de données.

1. Entrez un **nom** (par exemple, le nom d'un patient) pour votre jeu de données, puis renseignez les valeurs de votre flux. Vous trouverez ci-dessous un exemple basé sur les valeurs provenant des appareils simulés du modèle d'application de surveillance continue des patients. L'exemple repose sur deux patients :

   * Teddy Silvers, pour les données provenant de la genouillère intelligente Smart Knee Brace.
   * Yesenia Sanford, pour les données provenant du dispositif Smart Vitals Patch.

   :::image type="content" source="media/enter-dataset-values.png" alt-text="Entrez les valeurs du jeu de données":::

Pour en savoir plus sur les jeux de données de streaming dans Power BI, vous pouvez lire ce document consacré à la [diffusion en temps réel dans Power BI](/power-bi/service-real-time-streaming).


## <a name="connect-your-logic-app-to-azure-event-hubs"></a>Connecter votre application logique à Azure Event Hubs

Pour connecter votre application logique à Azure Event Hubs, vous pouvez suivre les instructions fournies dans ce document sur l'[Envoi d'événements avec Azure Event Hubs et Azure Logic Apps](../../connectors/connectors-create-api-azure-event-hubs.md#add-event-hubs-action). Voici quelques suggestions de paramètres :

|Paramètre|Valeur|
|---|---|
|Type de contenu|application/json|
|Intervalle|3|
|Fréquence|Seconde|

À la fin de cette étape, votre Concepteur d'applications logiques doit se présenter comme suit :

>[!div class="mx-imgBorder"] 
>![Logic Apps se connecte à Event Hubs](media/eh-logic-app.png)

:::image type="content" source="media/enter-dataset-values.png" alt-text="Entrez les valeurs du jeu de données":::


## <a name="stream-data-to-power-bi-from-your-logic-app"></a>Diffuser des données vers Power BI à partir de votre application logique

L'étape suivante consistera à analyser les données provenant de votre hub d'événements pour les diffuser dans les jeux de données Power BI que vous avez créés précédemment.

Avant cela, vous devez comprendre la charge utile JSON envoyée de votre appareil vers votre hub d'événements. Pour ce faire, vous pouvez consulter cet [exemple de schéma](../core/howto-export-data.md#telemetry-format) et le modifier afin qu'il corresponde à votre schéma, ou utiliser l'[explorateur Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer) pour inspecter les messages. Si vous utilisez les applications de surveillance continue des patients, vos messages se présenteront comme suit :

**Données de télémétrie du dispositif Smart Vitals Patch**

```json
{
  "HeartRate": 80,
  "RespiratoryRate": 12,
  "HeartRateVariability": 64,
  "BodyTemperature": 99.08839032397609,
  "BloodPressure": {
    "Systolic": 23,
    "Diastolic": 34
  },
  "Activity": "walking"
}
```

**Données de télémétrie de la genouillère intelligente Smart Knee Brace**

```json
{
  "Acceleration": {
    "x": 72.73510947763711,
    "y": 72.73510947763711,
    "z": 72.73510947763711
  },
  "RangeOfMotion": 123,
  "KneeBend": 3
}
```

**Propriétés**

```json
{
  "iothub-connection-device-id": "1qsi9p8t5l2",
  "iothub-connection-auth-method": "{\"scope\":\"device\",\"type\":\"sas\",  \"issuer\":\"iothub\",\"acceptingIpFilterRule\":null}",
  "iothub-connection-auth-generation-id": "637063718586331040",
  "iothub-enqueuedtime": 1571681440990,
  "iothub-message-source": "Telemetry",
  "iothub-interface-name": "Patient_health_data_3bk",
  "x-opt-sequence-number": 7,
  "x-opt-offset": "3672",
  "x-opt-enqueued-time": 1571681441317
}
```

1. Maintenant que vous avez inspecté vos charges utiles JSON, revenez à votre Concepteur d'applications logiques et sélectionnez **+ Nouvelle étape**. Recherchez et ajoutez **Initialiser la variable**, puis entrez les paramètres suivants :

   |Paramètre|Valeur|
   |---|---|
   |Nom|Nom de l'interface|
   |Type|String|

   Sélectionnez **Enregistrer**. 

1. Ajoutez une autre variable appelée **Corps** accompagnée du Type **Chaîne**. Ces actions seront ajoutées à votre application logique :

   :::image type="content" source="media/initialize-string-variables.png" alt-text="Initialiser les variables":::
    
1. Sélectionnez **+ Nouvelle étape** et ajoutez une action **Analyser JSON**. Remplacez son nom par **Propriétés d'analyse**. Pour le contenu, choisissez les **Propriétés** provenant du hub d'événements. Sélectionnez **Utiliser l'exemple de charge utile pour générer le schéma** en bas, puis collez l'exemple de charge utile dans la section Propriétés au-dessus.

1. Choisissez ensuite l'action **Définir la variable** et mettez à jour votre variable **Nom de l'interface** en la remplaçant par le nom **iothub-interface-name** provenant des propriétés JSON analysées.

1. Ajoutez un contrôle **Fractionner** en guise d'action suivante, et choisissez la variable **Nom de l'interface** comme paramètre Activé. Vous l'utiliserez pour acheminer les données vers le jeu de données qui convient.

1. Dans votre application Azure IoT Central, recherchez le Nom de l'interface correspondant aux données médicales du dispositif Smart Vitals Patch et de la genouillère Smart Knee Brace dans la vue **Modèles d'appareil**. Créez deux cas différents pour le contrôle **Basculer** de chaque Nom d'interface, et renommez le contrôle en conséquence. Vous pouvez définir le cas par défaut pour utiliser le contrôle **Terminer** et choisir l'état à afficher.

   :::image type="content" source="media/split-by-interface.png" alt-text="Contrôle Fractionner":::

1. Dans le cas du dispositif **Smart Vitals Patch**, ajoutez une action **Analyser JSON**. Pour le contenu, choisissez le **Contenu** provenant du hub d'événements. Copiez et collez les exemples de charge utile du dispositif Smart Vitals Patch précédent pour générer le schéma.

1. Ajoutez une action **Définir la variable** et mettez à jour la variable **Corps** avec la variable **Corps** du JSON analysé à l'étape 7.

1. Ajoutez un contrôle **Condition** en guise d'action suivante, et définissez la condition sur **Corps**, **contient**, **RythmeCardiaque**. Vous saurez ainsi que vous disposez du bon jeu de données du dispositif Smart Vitals Patch avant de renseigner le jeu de données Power BI. Les étapes 7 à 9 se présentent comme suit :

   :::image type="content" source="media/smart-vitals-pbi.png" alt-text="Condition d'ajout de Smart Vitals":::

1. Pour le cas **Vrai** de la condition, ajoutez une action qui appelle la fonctionnalité Power BI **Ajouter des lignes à un jeu de données**. Pour ce faire, vous devez vous connecter à Power BI. Votre cas **Faux** peut à nouveau utiliser le contrôle **Terminer**.

1. Choisissez l'**espace de travail**, le **jeu de données** et la **table** appropriés. Mappez les paramètres que vous avez spécifiés lors de la création de votre jeu de données de streaming dans Power BI avec les valeurs JSON analysées provenant de votre hub d'événements. Vos actions complétées doivent se présenter comme suit :
 
   :::image type="content" source="media/add-rows-yesenia.png" alt-text="Ajouter des lignes à Power BI":::

1. Pour le cas du commutateur **Smart Knee Brace**, ajoutez une action **Analyser JSON** pour analyser le contenu, comme à l'étape 7. Ensuite, **ajoutez des lignes à un jeu de données** pour mettre à jour votre jeu de données Teddy Silvers dans Power BI.

   :::image type="content" source="media/knee-brace-pbi.png" alt-text="Capture d’écran qui montre comment ajouter des lignes à un jeu de données":::

1. Appuyez sur **Enregistrer**, puis exécutez votre application logique.


## <a name="build-a-real-time-dashboard-for-patient-vitals"></a>Créer un tableau de bord en temps réel pour les données vitales des patients

Revenez maintenant à Power BI et sélectionnez **+ Créer** pour créer un nouveau **Tableau de bord**. Nommez votre tableau de bord et appuyez sur **Créer**.

Sélectionnez les trois points dans la barre de navigation supérieure, puis sélectionnez **+ Ajouter une vignette**.

:::image type="content" source="media/add-tile.png" alt-text="Ajouter une vignette au tableau de bord":::

Choisissez le type de vignette que vous souhaitez ajouter et personnalisez votre application comme vous le souhaitez.


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne pensez pas continuer à utiliser cette application, supprimez vos ressources en procédant comme suit :

1. Sur le portail Azure, vous pouvez supprimer les ressources Event Hub et Logic Apps que vous avez créées.

1. Pour votre application IoT Central, accédez à l'onglet Administration et sélectionnez **Supprimer**.


## <a name="next-steps"></a>Étapes suivantes

* Consultez le [Guide relatif à l'architecture de surveillance continue des patients](concept-continuous-patient-monitoring-architecture.md).
