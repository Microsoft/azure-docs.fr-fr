---
title: 'Tutoriel : Connecter une solution de bout en bout'
titleSuffix: Azure Digital Twins
description: Tutoriel montrant comment créer une solution Azure Digital Twins de bout en bout basée sur des données d’appareil.
author: baanders
ms.author: baanders
ms.date: 4/15/2020
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: f788c9e78790e6872870869e2bc153e1b1451e51
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566535"
---
# <a name="tutorial-build-out-an-end-to-end-solution"></a>Tutoriel : Créer une solution de bout en bout

Pour configurer une solution de bout en bout complète basée sur les données réelles de votre environnement, vous pouvez connecter votre instance Azure Digital Twins à d’autres services Azure pour la gestion des appareils et des données.

Ce tutoriel présente les procédures suivantes :
> [!div class="checklist"]
> * Configurer une instance Azure Digital Twins
> * Découvrir l’exemple de scénario d’un bâtiment et instancier les composants pré-écrits
> * Utiliser une application [Azure Functions](../azure-functions/functions-overview.md) pour router les données de télémétrie simulées depuis un appareil [IoT Hub](../iot-hub/about-iot-hub.md) vers les propriétés de jumeaux numériques
> * Propager les modifications par le biais du **graphe de jumeaux**, en traitant les notifications de jumeaux numériques avec Azure Functions, les points de terminaison et les routes

[!INCLUDE [Azure Digital Twins tutorial: sample prerequisites](../../includes/digital-twins-tutorial-sample-prereqs.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="set-up-cloud-shell-session"></a>Configurer une session Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

[!INCLUDE [Azure Digital Twins tutorial: configure the sample project](../../includes/digital-twins-tutorial-sample-configure.md)]

## <a name="get-started-with-the-building-scenario"></a>Démarrer avec le scénario d’un bâtiment

L’exemple de projet utilisé dans ce tutoriel représente un **scénario de bâtiment** concret, comportant un étage, une pièce et un thermostat. Ces composants sont représentés numériquement dans une instance Azure Digital Twins, qui est ensuite connectée à [IoT Hub](../iot-hub/about-iot-hub.md), [Event Grid](../event-grid/overview.md) et à deux [fonctions Azure](../azure-functions/functions-overview.md) pour faciliter le déplacement des données.

Voici un diagramme représentant le scénario complet. 

Vous allez successivement créer l’instance Azure Digital Twins (**section A** dans le diagramme), configurer le flux de données de télémétrie dans les jumeaux numériques (**flèche B**) et configurer la propagation des données par le biais du graphe de jumeaux (**flèche C**).

:::image type="content" source="media/tutorial-end-to-end/building-scenario.png" alt-text="Graphique du scénario de bâtiment complet. Décrit le flux de données depuis un appareil vers successivement IoT Hub, une fonction Azure (flèche B), une instance Azure Digital Twins (section A), Event Grid et une autre fonction Azure en vue du traitement (flèche C)":::

Pour parcourir le scénario, vous interagissez avec les composants de l’exemple d’application pré-écrite que vous avez téléchargé.

Voici les composants implémentés par l’exemple d’application *AdtSampleApp* du scénario d’un bâtiment :
* Authentification des appareils 
* Exemples d’utilisation du [SDK .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client?view=azure-dotnet&preserve-view=true) (disponibles dans *CommandLoop.cs*)
* Interface de la console pour appeler l’API Azure Digital Twins
* *SampleClientApp* : exemple de solution Azure Digital Twins
* *SampleFunctionsApp* : application Azure Functions qui met à jour votre graphe Azure Digital Twins à partir des données de télémétrie issues des événements IoT Hub et Azure Digital Twins

L’exemple de projet contient également un composant d’autorisation interactif. Chaque fois que vous démarrez le projet, une fenêtre de navigateur s’ouvre, vous invitant à vous connecter avec votre compte Azure.

### <a name="instantiate-the-pre-created-twin-graph"></a>Instancier le graphe de jumeaux pré-créé

Tout d’abord, vous allez utiliser la solution *AdtSampleApp* à partir de l’exemple de projet pour créer la partie Azure Digital Twins du scénario de bout en bout (**section A**) :

:::image type="content" source="media/tutorial-end-to-end/building-scenario-a.png" alt-text="Partie du graphique du scénario de bâtiment complet mettant en évidence la section A (instance Azure Digital Twins)":::

Dans la fenêtre Visual Studio où le projet _**AdtE2ESample**_ est ouvert, exécutez le projet avec le bouton de la barre d’outils mis en évidence ci-dessous :

:::image type="content" source="media/tutorial-end-to-end/start-button-sample.png" alt-text="Bouton de démarrage dans Visual Studio (projet SampleClientApp)":::

Une fenêtre de console s’ouvre, exécute l’authentification et attend une commande. Dans cette console, exécutez la commande suivante pour instancier l’exemple de solution Azure Digital Twins.

> [!IMPORTANT]
> Si vous avez déjà des jumeaux numériques et des relations dans votre instance Azure Digital Twins, l’exécution de cette commande les supprime et les remplace par les jumeaux et les relations de l’exemple de scénario.

```cmd/sh
SetupBuildingScenario
```

Cette commande génère une série de messages confirmant la création et la connexion de trois [**jumeaux numériques**](concepts-twins-graph.md) dans votre instance Azure Digital Twins : un étage nommé *floor1*, une pièce nommée *Room21* et un capteur de température nommé *thermostat67*. Ces jumeaux numériques représentent les entités qui existeraient dans un environnement réel.

Elles sont connectées par le biais de relations dans le [**graphe de jumeaux**](concepts-twins-graph.md). Le graphe de jumeaux représente l’environnement dans son ensemble, y compris les interactions et les liens entre les entités.

:::image type="content" source="media/tutorial-end-to-end/building-scenario-graph.png" alt-text="Graphe indiquant que floor1 contient room21 et que room21 contient thermostat67" border="false":::

Vous pouvez vérifier les jumeaux qui ont été créées en exécutant la commande suivante, qui interroge l’instance Azure Digital Twins connectée afin de déterminer tous les jumeaux numériques qu’elle contient :

```cmd/sh
Query
```

>[!TIP]
> Cette méthode simplifiée est fournie dans le cadre du projet _**AdtE2ESample**_. En dehors du contexte de cet exemple de code, vous pouvez à tout moment interroger tous les jumeaux dans votre instance à l’aide des [API de requête](/rest/api/digital-twins/dataplane/query) ou des [commandes CLI](how-to-use-cli.md).
>
> Voici le corps complet de la requête pour obtenir tous les jumeaux numériques dans votre instance :
> 
> ```sql
> SELECT *
> FROM DIGITALTWINS
> ``` 

Ensuite, vous pouvez arrêter l’exécution du projet. Gardez toutefois la solution ouverte dans Visual Studio, car vous l’utiliserez tout au long du tutoriel.

## <a name="set-up-the-sample-function-app"></a>Configurer l’exemple d’application de fonction

L’étape suivante consiste à configurer une [application Azure Functions](../azure-functions/functions-overview.md) qui sera utilisée tout au long de ce tutoriel pour traiter les données. L’application de fonction, *SampleFunctionsApp*, contient deux fonctions :
* *ProcessHubToDTEvents* : traite les données IoT Hub entrantes et met à jour Azure Digital Twins en conséquence
* *ProcessDTRoutedData* : traite les données provenant des jumeaux numériques et met à jour les jumeaux parents dans Azure Digital Twins en conséquence

Dans cette section, vous allez publier l’application de fonction préécrite et lui attribuer une identité Azure Active Directory (Azure AD) pour qu’elle puisse accéder à Azure Digital Twins. Une fois ces étapes effectuées, vous pourrez utiliser les fonctions de l’application de fonction dans le reste du tutoriel. 

De retour dans la fenêtre Visual Studio dans laquelle le projet _**AdtE2ESample**_ est ouvert, l’application de fonction se trouve dans le fichier de projet _**SampleFunctionsApp**_. Vous pouvez l’afficher dans le volet *Explorateur de solutions*.

### <a name="update-dependencies"></a>Mettre à jour les dépendances

Avant de publier l’application, il est judicieux de vérifier que vos dépendances sont à jour pour être certain que vous disposez de la dernière version de tous les packages inclus.

Dans le volet *Explorateur de solutions*, développez *SampleFunctionsApp > Dépendances*. Cliquez avec le bouton droit sur *Packages*, puis sélectionnez *Gérer les packages NuGet...* .

:::image type="content" source="media/tutorial-end-to-end/update-dependencies-1.png" alt-text="Visual Studio : Gérer les packages NuGet pour le projet SampleFunctionsApp" border="false":::

Cette action a pour effet d’ouvrir le Gestionnaire de package NuGet. Sélectionnez l’onglet *Mises à jour* et, s’il y a des packages à mettre à jour, activez la case à cocher *Sélectionner tous les packages*. Ensuite, appuyez sur *Mettre à jour*.

:::image type="content" source="media/tutorial-end-to-end/update-dependencies-2.png" alt-text="Visual Studio : Sélection de la mise à jour de tous les packages dans le Gestionnaire de package NuGet":::

### <a name="publish-the-app"></a>Publier l’application

Dans la fenêtre Visual Studio où le projet _**AdtE2ESample**_ est ouvert, dans le volet *Explorateur de solutions*, cliquez avec le bouton droit sur le fichier de projet _**SampleFunctionsApp**_, puis cliquez sur **Publier**.

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-1.png" alt-text="Visual Studio : publier le projet":::

Dans la page *Publier* qui suit, conservez **Azure** comme sélection de la cible par défaut et cliquez sur *Suivant*. 

Pour une cible spécifique, choisissez **Application de fonction Azure (Windows)** et cliquez sur *Suivant*.

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-2.png" alt-text="Publier la fonction Azure dans Visual Studio : cible spécifique":::

Dans la page *Instance de Functions*, choisissez votre abonnement. Cette opération doit remplir une zone avec les *groupes de ressources* de votre abonnement.

Sélectionnez le groupe de ressources de votre instance et cliquez sur *+ Créer une fonction Azure...* .

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-3.png" alt-text="Publier la fonction Azure dans Visual Studio : Instance de Functions (avant la création de l’application de fonction)":::

Dans la fenêtre *Application de fonction (Windows) - Créer* , renseignez les champs comme suit :
* Le **Nom** est le nom du plan de consommation qu’Azure utilisera pour héberger votre application Azure Functions. Il s’agit également du nom de l’application de fonction qui contient votre fonction réelle. Vous pouvez choisir votre propre valeur unique ou conserver la suggestion par défaut.
* Assurez-vous que l’**Abonnement** correspond à l’abonnement que vous souhaitez utiliser. 
* Assurez-vous que le **Groupe de ressources** correspond au groupe de ressources que vous souhaitez utiliser.
* Conservez *Consommation* comme **Type de plan**.
* Sélectionnez la **Localisation** qui correspond à la localisation de votre groupe de ressources.
* Créez une ressource de **Stockage Azure** à l’aide du lien *Nouveau...* . Définissez la localisation sur votre groupe de ressources, utilisez les autres valeurs par défaut et cliquez sur « OK ».

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-4.png" alt-text="Publier la fonction Azure dans Visual Studio : application de fonction (Windows) - Créer":::

Sélectionnez ensuite **Create** (Créer).

Vous revenez ainsi à la page *Instance de Functions*, où votre nouvelle application de fonction est désormais visible sous votre groupe de ressources. Cliquez sur *Terminer*.

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-5.png" alt-text="Publier la fonction Azure dans Visual Studio : Instance Functions (après la création de l’application de fonction)":::

Dans le volet *Publier* qui s’ouvre de nouveau dans la fenêtre principale de Visual Studio, vérifiez que toutes les informations semblent correctes et sélectionnez **Publier**.

:::image type="content" source="media/tutorial-end-to-end/publish-azure-function-6.png" alt-text="Publier la fonction Azure dans Visual Studio : publier":::

> [!NOTE]
> Si une fenêtre contextuelle similaire à celle-ci s’affiche : :::image type="content" source="media/tutorial-end-to-end/publish-azure-function-7.png" alt-text="Publier la fonction Azure dans Visual Studio : informations d’identification de publication" border="false":::
> Sélectionnez **Essayer de récupérer les informations d’identification d’Azure**, puis **Enregistrer**.
>
> Si vous voyez s’afficher l’avertissement *Mettre à niveau la version de Functions sur Azure* ou *Votre version du runtime Functions ne correspond pas à la version en cours d’exécution dans Azure* :
>
> Suivez les invites pour effectuer une mise à niveau vers la dernière version du runtime Azure Functions. Ce problème peut se produire si vous utilisez une version de Visual Studio antérieure à celle recommandée dans la section *Prérequis* du début de ce tutoriel.

### <a name="assign-permissions-to-the-function-app"></a>Attribuer des autorisations à l’application de fonction

Pour permettre à l’application de fonction d’accéder à Azure Digital Twins, l’étape suivante consiste à configurer un paramètre d’application, à attribuer à l’application une identité Azure AD managée par le système et à accorder à cette identité le rôle *Propriétaire des données Azure Digital Twins* dans l’instance Azure Digital Twins. Ce rôle est requis pour tout utilisateur ou fonction souhaitant effectuer de nombreuses activités de plan de données sur l’instance. Pour en savoir plus sur la sécurité et les attributions de rôles, consultez [*Concepts : Sécurité pour les solutions Azure Digital Twins*](concepts-security.md).

[!INCLUDE [digital-twins-role-rename-note.md](../../includes/digital-twins-role-rename-note.md)]

Dans Azure Cloud Shell, utilisez la commande suivante pour définir un paramètre d’application que votre application de fonction utilisera pour référencer votre instance Azure Digital Twins.

```azurecli-interactive
az functionapp config appsettings set -g <your-resource-group> -n <your-App-Service-(function-app)-name> --settings "ADT_SERVICE_URL=<your-Azure-Digital-Twins-instance-URL>"
```

Utilisez la commande suivante pour créer l’identité gérée par le système. Prenez note du champ *principalId* dans la sortie.

```azurecli-interactive
az functionapp identity assign -g <your-resource-group> -n <your-App-Service-(function-app)-name>
```

Utilisez la valeur *principalId* de la sortie dans la commande suivante afin d’attribuer l’identité de l’application de fonction au rôle *Propriétaire des données Azure Digital Twins* pour votre instance Azure Digital Twins :

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<principal-ID>" --role "Azure Digital Twins Data Owner"
```

Les informations générées par cette commande décrivent l’attribution de rôle que vous avez créée. L’application de fonction dispose maintenant des autorisations nécessaires pour accéder à votre instance Azure Digital Twins.

## <a name="process-simulated-telemetry-from-an-iot-hub-device"></a>Traiter les données de télémétrie simulées provenant d’un appareil IoT Hub

Un graphe Azure Digital Twins repose sur les données de télémétrie provenant d’appareils réels. 

Au cours de cette étape, vous allez connecter un thermostat simulé inscrit dans [IoT Hub](../iot-hub/about-iot-hub.md) au jumeau numérique qui le représente dans Azure Digital Twins. À mesure que l’appareil simulé émet des données de télémétrie, celles-ci sont acheminées via la fonction Azure *ProcessHubToDTEvents* qui déclenche une mise à jour correspondante dans le jumeau numérique. De cette façon, le jumeau numérique reste à jour avec les données de l’appareil réel. Dans Azure Digital Twins, le processus de redirection des données d’événements est appelé [**événements de routage**](concepts-route-events.md).

Ce processus se produit dans cette partie du scénario de bout en bout (**flèche B**) :

:::image type="content" source="media/tutorial-end-to-end/building-scenario-b.png" alt-text="Partie du graphique du scénario de bâtiment complet mettant en évidence la flèche B ainsi que les éléments situés avant Azure Digital Twins : l’appareil, IoT Hub et la première fonction Azure":::

Voici les actions que vous allez effectuer pour configurer la connexion de l’appareil :
1. Créer un hub IoT qui gérera l’appareil simulé
2. Connecter le hub IoT à la fonction Azure appropriée en configurant un abonnement aux événements
3. Inscrire l’appareil simulé dans le hub IoT
4. Exécuter l’appareil simulé et générer les données de télémétrie
5. Interroger Azure Digital Twins pour voir les résultats réels

### <a name="create-an-iot-hub-instance"></a>Créer une instance IoT Hub

Azure Digital Twins est conçu pour fonctionner avec [IoT Hub](../iot-hub/about-iot-hub.md), service Azure de gestion des appareils et de leurs données. Dans cette étape, vous allez configurer un hub IoT qui gérera l’exemple d’appareil dans ce tutoriel.

Dans Azure Cloud Shell, utilisez cette commande pour créer un hub IoT :

```azurecli-interactive
az iot hub create --name <name-for-your-IoT-hub> -g <your-resource-group> --sku S1
```

Les informations générées par cette commande décrivent le hub IoT qui a été créé.

Enregistrez le nom que vous avez donné à votre hub IoT. Vous le réutiliserez ultérieurement.

### <a name="connect-the-iot-hub-to-the-azure-function"></a>Connecter le hub IoT à la fonction Azure

Connectez ensuite votre hub IoT à la fonction Azure *ProcessHubToDTEvents* dans l’application de fonction que vous avez publiée, afin que les données puissent passer de l’appareil dans IoT Hub à la fonction, qui met à jour Azure Digital Twins.

Pour ce faire, vous allez créer un **abonnement aux événements** sur votre hub IoT, avec la fonction Azure comme point de terminaison. Cette opération « abonne » la fonction aux événements qui se produisent dans IoT Hub.

Dans le [portail Azure](https://portal.azure.com/), accédez au hub IoT tout juste créé en recherchant son nom dans la barre de recherche supérieure. Sélectionnez *Événements* dans le menu du hub, puis sélectionnez *+ Abonnement aux événements*.

:::image type="content" source="media/tutorial-end-to-end/event-subscription-1.png" alt-text="Portail Azure : abonnement aux événements IoT Hub":::

La page *créer un abonnement aux événements* s’affiche.

:::image type="content" source="media/tutorial-end-to-end/event-subscription-2.png" alt-text="Portail Azure : créer un abonnement aux événements":::

Renseignez les champs comme suit (les champs remplis par défaut ne sont pas mentionnés) :
* *DÉTAILS DE L’ABONNEMENT AUX ÉVÉNEMENTS* > **Nom** : Donnez un nom à votre abonnement aux événements.
* *DÉTAILS DE LA RUBRIQUE* > **Nom de la rubrique système** : spécifiez un nom à utiliser pour la rubrique système. 
* *TYPES D’ÉVÉNEMENTS* > **Filtrer les types d’événements** : Sélectionnez *Télémétrie d’appareil* dans les options de menu.
* *DÉTAILS DU POINT DE TERMINAISON* > **Type de point de terminaison** : Sélectionnez *Fonction Azure* dans les options de menu.
* *DÉTAILS DU POINT DE TERMINAISON* > **Point de terminaison** : Cliquez sur le lien *Sélectionner un point de terminaison*. Une fenêtre *Sélectionner une fonction Azure* s’ouvre : :::image type="content" source="media/tutorial-end-to-end/event-subscription-3.png" alt-text="Abonnement aux événements dans le portail Azure : sélectionner une fonction Azure" border="false":::
    - Renseignez vos **Abonnement**, **Groupe de ressources**, **Application de fonction** et **Fonction** (*ProcessHubToDTEvents*). Il est possible que certaines de ces informations soient automatiquement renseignées une fois que vous avez sélectionné l’abonnement.
    - Cliquez sur **Confirmer la sélection**.

Dans la page *Créer un abonnement aux événements*, cliquez sur **Créer**.

### <a name="register-the-simulated-device-with-iot-hub"></a>Inscrire l’appareil simulé auprès d’IoT Hub 

Dans cette section, vous allez créer une représentation d’appareil dans IoT Hub avec l’ID *thermostat67*. L’appareil simulé se connectera à cette dernière, ce qui permettra aux événements de télémétrie émis par l’appareil d’atteindre IoT Hub, où la fonction Azure souscrite à l’étape précédente est à l’écoute, prête à récupérer les événements et à continuer le traitement.

Dans Azure Cloud Shell, créez un appareil dans IoT Hub à l’aide de la commande suivante :

```azurecli-interactive
az iot hub device-identity create --device-id thermostat67 --hub-name <your-IoT-hub-name> -g <your-resource-group>
```

Les informations générées décrivent l’appareil qui a été créé.

### <a name="configure-and-run-the-simulation"></a>Configurer et démarrer la simulation

Ensuite, configurez le simulateur d’appareil pour envoyer des données à votre instance IoT Hub.

Commencez par obtenir la *chaîne de connexion IoT Hub* à l’aide de cette commande :

```azurecli-interactive
az iot hub connection-string show -n <your-IoT-hub-name>
```

Ensuite, récupérez la *chaîne de connexion de l’appareil* à l’aide de la commande suivante :

```azurecli-interactive
az iot hub device-identity connection-string show --device-id thermostat67 --hub-name <your-IoT-hub-name>
```

Vous intégrerez ces valeurs au code du simulateur d’appareil de votre projet local pour connecter le simulateur à ce hub IoT et à l’appareil IoT Hub.

Dans une nouvelle fenêtre Visual Studio, ouvrez (à partir du dossier de solution téléchargé) _Device Simulator > **DeviceSimulator.sln**_.

>[!NOTE]
> Vous devez maintenant avoir deux fenêtres Visual Studio : une avec _**DeviceSimulator.sln**_ et une autre antérieure avec _**AdtE2ESample.sln**_.

Dans le volet *Explorateur de solutions* de cette nouvelle fenêtre Visual Studio, sélectionnez _DeviceSimulator/**AzureIoTHub.cs**_ pour l’ouvrir dans la fenêtre d’édition. Remplacez les valeurs de chaîne de connexion suivantes par les valeurs que vous venez de collecter :

```csharp
iotHubConnectionString = <your-hub-connection-string>
deviceConnectionString = <your-device-connection-string>
```

Enregistrez le fichier .

Maintenant, pour voir les résultats de la simulation de données que vous avez configurée, exécutez le projet **DeviceSimulator** avec le bouton de la barre d’outils mis en évidence ci-dessous :

:::image type="content" source="media/tutorial-end-to-end/start-button-simulator.png" alt-text="Bouton de démarrage dans Visual Studio (projet DeviceSimulator)":::

Une fenêtre de console s’ouvre, affichant les messages de télémétrie de température simulée. Ils sont envoyés à IoT Hub, avant d’être récupérés et traités par la fonction Azure.

:::image type="content" source="media/tutorial-end-to-end/console-simulator-telemetry.png" alt-text="Sortie de la console du simulateur d’appareil montrant les données de télémétrie de température envoyées":::

Vous n’avez rien d’autre à faire dans cette console, mais laissez-la s’exécuter pendant que vous effectuez les étapes suivantes.

### <a name="see-the-results-in-azure-digital-twins"></a>Voir les résultats dans Azure Digital Twins

La fonction *ProcessHubToDTEvents* que vous avez publiée écoute les données IoT Hub et appelle une API Azure Digital Twins pour mettre à jour la propriété *Température* sur le jumeau *thermostat67*.

Pour voir les données du côté d’Azure Digital Twins, accédez à la fenêtre Visual Studio dans laquelle le projet _**AdtE2ESample**_ est ouvert et exécutez-le.

Dans la fenêtre de console du projet qui s’ouvre, exécutez la commande suivante pour récupérer les températures signalées par le jumeau numérique *thermostat67* :

```cmd
ObserveProperties thermostat67 Temperature
```

Les températures mises à jour réelles *issues de votre instance Azure Digital Twins* doivent normalement être journalisées dans la console toutes les 10 secondes.

:::image type="content" source="media/tutorial-end-to-end/console-digital-twins-telemetry.png" alt-text="Sortie de la console montrant le journal des messages de température issus du jumeau numérique thermostat67":::

Une fois que vous avez vérifié que la journalisation fonctionne correctement, vous pouvez arrêter l’exécution des deux projets. Laissez les fenêtres Visual Studio ouvertes, car vous continuerez à les utiliser dans le reste du tutoriel.

## <a name="propagate-azure-digital-twins-events-through-the-graph"></a>Propager des événements Azure Digital Twins à travers le graphe

Jusqu’à ce stade du tutoriel, vous avez vu comment Azure Digital Twins peut être mis à jour à partir de données d’appareil externes. Vous allez à présent voir comment les modifications apportées à un seul jumeau numérique peuvent se propager dans le graphe Azure Digital Twins, en d’autres termes, comment mettre à jour les jumeaux à partir de données internes au service.

Pour ce faire, vous allez utiliser la fonction Azure *ProcessDTRoutedData* afin de mettre à jour un jumeau *Room* quand le jumeau *Thermostat* connecté est mis à jour. Ce processus se produit dans cette partie du scénario de bout en bout (**flèche C**) :

:::image type="content" source="media/tutorial-end-to-end/building-scenario-c.png" alt-text="Partie du graphique du scénario de bâtiment complet mettant en évidence la flèche C ainsi que les éléments situés après Azure Digital Twins : Event Grid et la seconde fonction Azure":::

Voici les actions que vous allez effectuer pour configurer ce flux de données :
1. Créer un point de terminaison Event Grid dans Azure Digital Twins qui connecte l’instance à Event Grid
2. Configurer une route dans Azure Digital Twins pour envoyer les événements de modification de propriété de jumeau au point de terminaison
3. Déployer une application Azure Functions qui écoute (via [Event Grid](../event-grid/overview.md)) le point de terminaison et met à jour les autres jumeaux en conséquence
4. Exécuter l’appareil simulé et interroger Azure Digital Twins pour voir les résultats réels

### <a name="set-up-endpoint"></a>Configurer un point de terminaison

[Event Grid](../event-grid/overview.md) est un service Azure qui vous aide à router et à remettre des événements provenant de services Azure à d’autres points dans Azure. Vous pouvez créer une [rubrique Event Grid](../event-grid/concepts.md) pour collecter certains événements d’une source ; les abonnés peuvent ensuite écouter la rubrique pour recevoir les événements au fur et à mesure de leur arrivée.

Dans cette section, vous allez créer une rubrique Event Grid, puis créer un point de terminaison dans Azure Digital Twins qui pointe (envoie des événements) vers cette rubrique. 

Dans Azure Cloud Shell, exécutez la commande suivante pour créer une rubrique Event Grid :

```azurecli-interactive
az eventgrid topic create -g <your-resource-group> --name <name-for-your-event-grid-topic> -l <region>
```

> [!TIP]
> Pour obtenir la liste des noms de régions Azure qui peuvent être transmis à des commandes dans Azure CLI, exécutez la commande suivante :
> ```azurecli-interactive
> az account list-locations -o table
> ```

Les informations générées par cette commande décrivent la rubrique Event Grid que vous avez créée.

Ensuite, créez un point de terminaison Event Grid dans Azure Digital Twins, qui connectera votre instance à votre rubrique Event Grid. Utilisez la commande ci-dessous, en remplissant les champs d’espace réservé en fonction de vos besoins :

```azurecli-interactive
az dt endpoint create eventgrid --dt-name <your-Azure-Digital-Twins-instance> --eventgrid-resource-group <your-resource-group> --eventgrid-topic <your-event-grid-topic> --endpoint-name <name-for-your-Azure-Digital-Twins-endpoint>
```

Les informations générées par cette commande décrivent le point de terminaison que vous avez créé.

Vous pouvez également vérifier que la création du point de terminaison a réussi en exécutant la commande suivante pour le récupérer de votre instance Azure Digital Twins :

```azurecli-interactive
az dt endpoint show --dt-name <your-Azure-Digital-Twins-instance> --endpoint-name <your-Azure-Digital-Twins-endpoint> 
```

Recherchez le champ `provisioningState` dans la sortie et vérifiez que la valeur est « Succeeded ». Il peut également indiquer « Approvisionnement », ce qui signifie que le point de terminaison est toujours en cours de création. Dans ce cas, attendez quelques secondes, puis réexécutez la commande pour vérifier que la création du point de terminaison a réussi.

:::image type="content" source="media/tutorial-end-to-end/output-endpoints.png" alt-text="Résultat de la requête de point de terminaison, indiquant que l’état provisioningState du point de terminaison est Succeeded":::

Enregistrez les noms que vous avez donnés à votre rubrique Event Grid et à votre point de terminaison Event Grid dans Azure Digital Twins. Vous les utiliserez plus tard.

### <a name="set-up-route"></a>Configurer la route

Vous allez à présent créer une route Azure Digital Twins qui envoie les événements au point de terminaison Event Grid que vous venez de créer.

```azurecli-interactive
az dt route create --dt-name <your-Azure-Digital-Twins-instance> --endpoint-name <your-Azure-Digital-Twins-endpoint> --route-name <name-for-your-Azure-Digital-Twins-route>
```

Les informations générées par cette commande décrivent la route que vous avez créé.

>[!NOTE]
>L’approvisionnement des points de terminaison (de l’étape précédente) doit être terminé pour que vous puissiez configurer un itinéraire des événements qui les utilise. Si la création d’itinéraires échoue parce que les points de terminaison ne sont pas prêts, patientez quelques minutes, puis réessayez.

#### <a name="connect-the-function-to-event-grid"></a>Connecter la fonction à Event Grid

Vous allez à présent abonner la fonction Azure *ProcessDTRoutedData* à la rubrique Event Grid que vous avez créée, afin que les données de télémétrie puissent aller du jumeau *thermostat67*, via la rubrique Event Grid, à la fonction, qui, une fois dans Azure Digital Twins, met à jour le jumeau *Room21*.

Pour ce faire, vous allez créer un **abonnement Event Grid** à partir de votre rubrique Event Grid vers votre fonction Azure *ProcessDTRoutedData* en tant que point de terminaison.

Dans le [portail Azure](https://portal.azure.com/), accédez à votre rubrique Event Grid en recherchant son nom dans la barre de recherche supérieure. Sélectionnez *+ Abonnement aux événements*.

:::image type="content" source="media/tutorial-end-to-end/event-subscription-1b.png" alt-text="Portail Azure : abonnement aux événements Event Grid":::

Les étapes de création de cet abonnement aux événements sont similaires à celles que vous avez suivies pour abonner la première fonction Azure à IoT Hub plus haut dans ce tutoriel. Cette fois, vous n’avez pas besoin de spécifier *Télémétrie d’appareil* comme type d’événement à écouter et vous allez vous connecter à une autre fonction Azure.

Dans la page *Créer un abonnement aux événements*, renseignez les champs comme suit (les champs remplis par défaut ne sont pas mentionnés) :
* *DÉTAILS DE L’ABONNEMENT AUX ÉVÉNEMENTS* > **Nom** : Donnez un nom à votre abonnement aux événements.
* *DÉTAILS DU POINT DE TERMINAISON* > **Type de point de terminaison** : Sélectionnez *Fonction Azure* dans les options de menu.
* *DÉTAILS DU POINT DE TERMINAISON* > **Point de terminaison** : Cliquez sur le lien *Sélectionner un point de terminaison*. Une fenêtre *Sélectionner une fonction Azure* s’ouvre :
    - Renseignez vos **Abonnement**, **Groupe de ressources**, **Application de fonction** et **Fonction** (*ProcessDTRoutedData*). Il est possible que certaines de ces informations soient automatiquement renseignées une fois que vous avez sélectionné l’abonnement.
    - Cliquez sur **Confirmer la sélection**.

Dans la page *Créer un abonnement aux événements*, cliquez sur **Créer**.

### <a name="run-the-simulation-and-see-the-results"></a>Exécuter la simulation et afficher les résultats

Vous pouvez maintenant exécuter le simulateur d’appareil pour lancer le nouveau flux d’événements que vous avez configuré. Accédez à la fenêtre Visual Studio dans laquelle le projet _**DeviceSimulator**_ est ouvert, puis exécutez le projet.

Comme quand vous avez exécuté le simulateur d’appareil plus haut, une fenêtre de console s’ouvre, affichant les messages de télémétrie de température simulée. Ces événements empruntent le flux que vous avez configuré plus haut pour mettre à jour le jumeau *thermostat67*, puis le flux que vous avez configuré récemment pour mettre à jour le jumeau *Room21* en conséquence.

:::image type="content" source="media/tutorial-end-to-end/console-simulator-telemetry.png" alt-text="Sortie de la console du simulateur d’appareil montrant les données de télémétrie de température envoyées":::

Vous n’avez rien d’autre à faire dans cette console, mais laissez-la s’exécuter pendant que vous effectuez les étapes suivantes.

Pour voir les données du côté d’Azure Digital Twins, accédez à la fenêtre Visual Studio dans laquelle le projet _**AdtE2ESample**_ est ouvert et exécutez-le.

Dans la fenêtre de console du projet qui s’ouvre, exécutez la commande suivante pour récupérer les températures signalées **à la fois** par le jumeau numérique *thermostat67* et par le jumeau numérique *room21*.

```cmd
ObserveProperties thermostat67 Temperature room21 Temperature
```

Les températures mises à jour réelles *issues de votre instance Azure Digital Twins* doivent normalement être journalisées dans la console toutes les 10 secondes. Notez que la température de *Room21* est mise à jour pour correspondre aux mises à jour apportées à *thermostat67*.

:::image type="content" source="media/tutorial-end-to-end/console-digital-twins-telemetry-b.png" alt-text="Sortie de la console montrant le journal des messages de température entre un thermostat et une pièce":::

Une fois que vous avez vérifié que la journalisation fonctionne correctement, vous pouvez arrêter l’exécution des deux projets. Vous pouvez également fermer les fenêtres de Visual Studio, car le tutoriel est maintenant terminé.

## <a name="review"></a>Révision

Voici une révision du scénario que vous avez créé au cours de ce tutoriel.

1. Une instance Azure Digital Twins représente numériquement un étage, une pièce et un thermostat (représentés par la **section A** du diagramme ci-dessous)
2. La télémétrie des appareils simulés est envoyée à IoT Hub, où la fonction Azure *ProcessHubToDTEvents* écoute les événements de télémétrie. La fonction Azure *ProcessHubToDTEvents* utilise les informations contenues dans ces événements pour définir la propriété *Temperature* sur *thermostat67* (**flèche B** du diagramme).
3. Les événements de modification de propriété dans Azure Digital Twins sont routés vers une rubrique Event Grid, où la fonction Azure *ProcessDTRoutedData* écoute les événements. La fonction Azure *ProcessDTRoutedData* utilise les informations contenues dans ces événements pour définir la propriété *Temperature* sur *room21* (**flèche C** du diagramme).

:::image type="content" source="media/tutorial-end-to-end/building-scenario.png" alt-text="Graphique du scénario de bâtiment complet. Décrit le flux de données depuis un appareil vers successivement IoT Hub, une fonction Azure (flèche B), une instance Azure Digital Twins (section A), Event Grid et une autre fonction Azure en vue du traitement (flèche C)":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin des ressources créées dans ce tutoriel, effectuez les étapes suivantes pour les supprimer. 

Dans [Azure Cloud Shell](https://shell.azure.com), vous pouvez supprimer toutes les ressources Azure d’un groupe de ressources avec la commande [az group delete](/cli/azure/group?preserve-view=true&view=azure-cli-latest#az-group-delete). Sont ainsi supprimés le groupe de ressources ; l’instance Azure Digital Twins ; le hub IoT et l’inscription de l’appareil hub ; la rubrique Event Grid et les abonnements associés ; l’application Azure Functions, y compris les deux fonctions et les ressources associées (par exemple, le stockage).

> [!IMPORTANT]
> La suppression d’un groupe de ressources est irréversible. Le groupe de ressources et toutes les ressources qu’il contient sont supprimés définitivement. Veillez à ne pas supprimer accidentellement des ressources ou un groupe de ressources incorrects. 

```azurecli-interactive
az group delete --name <your-resource-group>
```

Pour finir, supprimez le dossier d’exemple de projet que vous avez téléchargé sur votre ordinateur local.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé un scénario de bout en bout qui montre comment exploiter Azure Digital Twins à l’aide de données d’appareil réelles.

Vous pouvez à présent commencer à consulter la documentation de concept pour en savoir plus sur les éléments avec lesquels vous avez travaillé dans le tutoriel :

> [!div class="nextstepaction"]
> [*Concepts : Modèles personnalisés*](concepts-models.md)