---
title: Configurer et utiliser des métriques et des journaux avec un hub Azure IoT
description: Découvrez comment configurer et utiliser des métriques et des journaux avec un hub Azure IoT. Cela vous permet de disposer de données à analyser pour faciliter le diagnostic des problèmes de votre hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 10/29/2020
ms.author: robinsh
ms.custom:
- mvc
- mqtt
- devx-track-azurecli
- devx-track-csharp
ms.openlocfilehash: a16dbeedc2362f4a263d59a673dbb4358f7ba034
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436485"
---
# <a name="tutorial-set-up-and-use-metrics-and-logs-with-an-iot-hub"></a>Tutoriel : Configurer et utiliser des métriques et des journaux avec un hub IoT

Vous pouvez utiliser Azure Monitor afin de collecter des métriques et des journaux pour votre hub IoT qui peuvent vous aider à superviser le fonctionnement de votre solution et à résoudre les problèmes lorsqu’ils se produisent. Dans cet article, vous allez découvrir comment créer des graphiques basés sur des métriques, créer des alertes qui déclenchent des métriques, envoyer des opérations et des erreurs IoT Hub vers des journaux Azure Monitor, et consulter les erreurs dans les journaux.

Ce tutoriel utilise l’exemple Azure du [guide de démarrage rapide d’envoi de données de télémétrie (.NET)](quickstart-send-telemetry-dotnet.md) pour envoyer des messages au hub IoT. Vous pouvez toujours utiliser un appareil ou un autre exemple pour envoyer des messages, mais vous devrez peut-être modifier certaines étapes en conséquence.

Nous vous conseillons de vous familiariser avec les concepts Azure Monitor avant de commencer ce tutoriel. Pour plus d’informations, consultez [Surveiller IoT Hub](monitor-iot-hub.md). Pour en savoir plus sur les métriques et les journaux de ressources émis par IoT Hub, consultez les [informations de référence sur la supervision des données](monitor-iot-hub-reference.md).

Dans ce didacticiel, vous allez effectuer les tâches suivantes :

> [!div class="checklist"]
>
> * Utiliser Azure CLI pour créer un hub IoT, inscrire un appareil simulé et créer un espace de travail Log Analytics.  
> * Envoyer des connexions et des journaux de ressources de télémétrie d’appareil IoT Hub vers les journaux Azure Monitor dans l’espace de travail Log Analytics
> * Utiliser l’explorateur de métriques pour créer un graphique basé sur les métriques sélectionnées, et l’épingler à votre tableau de bord
> * Créer des alertes de métriques pour être averti par e-mail lorsque des conditions importantes se produisent
> * Télécharger et exécuter une application qui simule l’envoi de messages par un appareil IoT au hub IoT
> * Afficher les alertes lorsque vos conditions se produisent
> * Afficher le graphique des métriques dans votre tableau de bord
> * Afficher les erreurs et les opérations IoT Hub dans les journaux Azure Monitor.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- Votre machine de développement doit disposer du Kit SDK .NET Core 2.1 ou version ultérieure. Vous pouvez télécharger le Kit SDK .NET Core pour plusieurs plateformes sur [.NET](https://www.microsoft.com/net/download/all).

  Vous pouvez vérifier la version actuelle de C# sur votre machine de développement à l’aide de la commande suivante :

  ```cmd/sh
  dotnet --version
  ```

- Un compte e-mail capable de recevoir des e-mails.

- Assurez-vous que le port 8883 est ouvert dans votre pare-feu. L'exemple d’appareil de ce tutoriel utilise le protocole MQTT qui communique sur le port 8883. Dans certains environnements réseau professionnels et scolaires, ce port peut être bloqué. Pour plus d’informations sur les façons de contourner ce problème, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="set-up-resources"></a>Configurer des ressources

Pour ce tutoriel, vous avez besoin d’un hub IoT, d’un espace de travail Log Analytics et d’un appareil IoT simulé. Ces ressources peuvent être créées avec Azure CLI ou Azure PowerShell. Utilisez le même groupe de ressources et le même emplacement pour toutes les ressources. Ensuite, lorsque vous aurez terminé ce tutoriel, vous pourrez tout supprimer en une seule étape en supprimant le groupe de ressources.

Voici les étapes à suivre.

1. Créez un [groupe de ressources](../azure-resource-manager/management/overview.md).

2. Créez un hub IoT.

3. Créez un espace de travail Log Analytics.

4. Inscrivez une identité d’appareil pour l’appareil simulé qui envoie des messages à votre hub IoT. Enregistrez la chaîne de connexion de l’appareil à utiliser pour configurer l’appareil simulé.

### <a name="set-up-resources-using-azure-cli"></a>Configurer des ressources à l’aide d’Azure CLI

Copiez et collez ce script dans Cloud Shell. En supposant que vous êtes déjà connecté, il exécute le script, ligne par ligne. L’exécution de certaines commandes peut prendre un certain temps. Les nouvelles ressources sont créées dans le groupe de ressources *ContosoResources*.

Le nom de certaines ressources doit être unique dans Azure. Le script génère une valeur aléatoire avec la fonction `$RANDOM`, et la stocke dans une variable. Pour ces ressources, le script ajoute cette valeur aléatoire à un nom de base pour la ressource, ce qui rend le nom de la ressource unique.

Un seul hub IoT gratuit est autorisé par abonnement. Si vous disposez déjà d’un hub IoT gratuit dans votre abonnement, supprimez-le avant d’exécuter le script ou modifiez le script de façon à utiliser votre hub IoT gratuit ou un hub IoT qui utilise le niveau standard ou de base.

Le script imprime le nom du hub IoT, le nom de l’espace de travail Log Analytics et la chaîne de connexion pour l’appareil qu’il inscrit. Notez-les car vous en aurez besoin plus loin dans cet article.

```azurecli-interactive

# This is the IOT Extension for Azure CLI.
# You only need to install this the first time.
# You need it to create the device identity.
az extension add --name azure-iot

# Set the values for the resource names that don't have to be globally unique.
# The resources that have to have unique names are named in the script below
#   with a random number concatenated to the name so you can probably just
#   run this script, and it will work with no conflicts.
location=westus
resourceGroup=ContosoResources
iotDeviceName=Contoso-Test-Device
randomValue=$RANDOM

# Create the resource group to be used
#   for all the resources for this tutorial.
az group create --name $resourceGroup \
    --location $location

# The IoT hub name must be globally unique, so add a random number to the end.
iotHubName=ContosoTestHub$randomValue
echo "IoT hub name = " $iotHubName

# Create the IoT hub in the Free tier. Partition count must be 2.
az iot hub create --name $iotHubName \
    --resource-group $resourceGroup \
    --partition-count 2 \
    --sku F1 --location $location

# The Log Analytics workspace name must be globally unique, so add a random number to the end.
workspaceName=contoso-la-workspace$randomValue
echo "Log Analytics workspace name = " $workspaceName


# Create the Log Analytics workspace
az monitor log-analytics workspace create --resource-group $resourceGroup \
    --workspace-name $workspaceName --location $location

# Create the IoT device identity to be used for testing.
az iot hub device-identity create --device-id $iotDeviceName \
    --hub-name $iotHubName

# Retrieve the primary connection string for the device identity, then copy it to
#   Notepad. You need this to run the device simulation during the testing phase.
az iot hub device-identity show-connection-string --device-id $iotDeviceName \
    --hub-name $iotHubName

```

>[!NOTE]
>Quand vous créez l’identité d’appareil, vous pouvez recevoir l’erreur suivante : *Aucune clé trouvée pour la stratégie iothubowner du hub IoT ContosoTestHub*. Pour corriger cette erreur, mettez à jour l’extension Azure CLI IoT et réexécutez les deux dernières commandes dans le script. 
>
>Voici la commande pour mettre à jour l’extension. Exécutez cette commande dans votre instance Cloud Shell.
>
>```cli
>az extension update --name azure-iot
>```

## <a name="collect-logs-for-connections-and-device-telemetry"></a>Collecter les journaux pour les connexions et la télémétrie des appareils

IoT Hub émet des journaux de ressources pour plusieurs catégories d’opérations. Toutefois, pour afficher ces journaux vous devez créer un paramètre de diagnostic afin de les envoyer vers une destination, telle que les journaux Azure Monitor, qui sont collectés dans un espace de travail Log Analytics. Les journaux de ressources IoT Hub sont regroupés dans différentes catégories. Vous pouvez sélectionner celles que vous souhaitez envoyer vers les journaux Azure Monitor dans le paramètre de diagnostic. Dans cet article, nous allons recueillir les journaux des opérations et des erreurs qui se produisent au niveau des connexions et de la télémétrie des appareils. Pour obtenir la liste complète des catégories prises en charge pour IoT Hub, consultez [Journaux de ressources IoT Hub](monitor-iot-hub-reference.md#resource-logs).

Pour créer un paramètre de diagnostic pour envoyer des journaux de ressources IoT Hub vers les journaux Azure Monitor, effectuez les étapes suivantes :

1. Pour commencer, si vous n’êtes pas déjà sur votre hub dans le portail, sélectionnez **Groupe de ressources**, puis le groupe de ressources ContosoResources. Sélectionnez votre hub IoT dans la liste des ressources affichées.

1. Recherchez la section **Supervision** dans le panneau IoT Hub. Sélectionnez **Paramètres de diagnostic**. Sélectionnez ensuite **Ajouter un paramètre de diagnostic**.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/open-diagnostic-settings.png" alt-text="Capture d’écran qui met en évidence Paramètres de diagnostic dans la section Supervision.":::

1. Dans le volet **Paramètre de diagnostic**, donnez à votre paramètre un nom descriptif, par exemple « Envoyer des connexions et des données de télémétrie vers les journaux ».

1. Sous **Détails de la catégorie**, sélectionnez **Connexions** et **Télémétrie d’appareil**.

1. Sous **Détails de la destination**, sélectionnez **Envoyer à Log Analytics**, puis utilisez le sélecteur d’espace de travail Log Analytics pour sélectionner l’espace de travail noté précédemment. Lorsque vous avez terminé, le paramètre de diagnostic doit ressembler à la capture d’écran suivante :

   :::image type="content" source="media/tutorial-use-metrics-and-diags/add-diagnostic-setting.png" alt-text="Capture d’écran montrant la configuration finale des journaux de diagnostic.":::

1. Sélectionnez **Enregistrer** pour enregistrer les paramètres. Fermez le volet **Paramètre de diagnostic**. Vous pouvez voir votre nouveau paramètre dans la liste des paramètres de diagnostic.

## <a name="set-up-metrics"></a>Configurer des métriques

À présent, nous allons utiliser l’explorateur de métriques pour créer un graphique qui affiche les métriques que vous souhaitez suivre. Vous allez épingler ce graphique à votre tableau de bord par défaut dans le portail Azure.

1. Dans le volet gauche de votre hub IoT, sélectionnez **Métriques** dans la section **Supervision**.

1. En haut de l’écran, sélectionnez **Dernières 24 heures (automatique)** . Dans la liste déroulante qui s’affiche, sélectionnez **4 dernières heures** comme **Intervalle de temps**, **1 minute** comme **Granularité temporelle** et **Local** pour **Afficher l’heure en tant que**. Sélectionnez **Appliquer** pour enregistrer ces paramètres. Le paramètre doit maintenant indiquer **Heure locale : 4 dernières heures (1 minute)** .

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-select-time-range.png" alt-text="Capture d’écran montrant les paramètres horaires des métriques.":::

1. Sur le graphique, un paramètre de métrique partiel est affiché, avec votre hub IoT comme étendue. Conservez les valeurs par défaut d’**Étendue** et d’**Espace de noms de métrique**. Sélectionnez le paramètre **Métrique** et tapez « Télémétrie », puis sélectionnez **Messages de télémétrie envoyés** dans la liste déroulante. L’**Agrégation** sera automatiquement définie sur **Sum**. Notez que le titre de votre graphique change également.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-telemetry-messages-sent.png" alt-text="Capture d’écran qui montre comment ajouter la métrique Messages de télémétrie envoyés au graphique.":::

1. Sélectionnez maintenant **Ajouter une métrique** pour ajouter une autre métrique au graphique. Sous **Métrique**, sélectionnez **Nombre total de messages utilisés**. L’**Agrégation** sera automatiquement définie sur **Avg**. Remarquez là encore que le titre du graphique a changé pour inclure cette métrique.

   Votre écran montre maintenant la métrique réduite pour *Messages de télémétrie envoyés* ainsi que la nouvelle métrique pour *Nombre total de messages utilisés*.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-total-number-of-messages-used.png" alt-text="Capture d’écran qui montre comment ajouter la métrique Nombre total de messages utilisés au graphique.":::

1. Dans le coin supérieur droit du graphique, sélectionnez **Épingler au tableau de bord**.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-total-number-of-messages-used-pin.png" alt-text="Capture d’écran qui met en évidence le bouton Épingler au tableau de bord.":::

1. Dans le volet **Épingler au tableau de bord**, sélectionnez l’onglet **Existant**. Sélectionnez **Privé**, puis **Tableau de bord** dans la liste déroulante Tableau de bord. Pour finir, sélectionnez **Épingler** pour épingler le graphique à votre tableau de bord par défaut dans le portail Azure. Si vous n’épinglez pas votre graphique à un tableau de bord, vos paramètres ne sont pas conservés lorsque vous quittez l’explorateur de métriques.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/pin-to-dashboard.png" alt-text="Capture d’écran qui montre les paramètres d’épinglage au tableau de bord.":::

## <a name="set-up-metric-alerts"></a>Configurer des alertes de métriques

À présent, nous allons configurer des alertes à déclencher sur les deux métriques *Messages de télémétrie envoyés* et *Nombre total de messages utilisés*.

*Messages de télémétrie envoyés* est une bonne métrique à superviser pour suivre le débit des messages et éviter toute limitation Pour un hub IoT au niveau gratuit, le seuil de limitation est de 100 messages/s. Avec un seul appareil, nous ne pourrons pas atteindre ce type de débit. Par conséquent, nous allons plutôt configurer l’alerte pour qu’elle se déclenche si le nombre de messages dépasse 1000 durant une période de cinq minutes. En production, vous pouvez définir le signal sur une valeur plus élevée, en fonction du niveau, de l’édition et du nombre d’unités de votre hub IoT.

*Nombre total de messages utilisés* effectue le suivi du nombre quotidien de messages utilisés. Cette métrique est réinitialisée tous les jours à 00:00 UTC. Si votre quota quotidien dépasse un certain seuil, votre hub IoT n’acceptera plus de messages. Pour un hub IoT au niveau gratuit, le quota de messages quotidien est de 8000. Nous allons configurer l’alerte de façon à ce qu’elle se déclenche si le nombre total de messages dépasse 4000, soit 50 % du quota. Dans la pratique, vous choisirez probablement un pourcentage supérieur. La valeur de quota quotidien dépend du niveau, de l’édition et du nombre d’unités de votre hub IoT.

Pour plus d’informations sur les limites de quota et les seuils de limitation avec IoT Hub, consultez [Quotas et limitation](iot-hub-devguide-quotas-throttling.md).

Pour configurer des alertes de métriques

1. Accédez à votre hub IoT dans le portail Azure.

1. Sous **Supervision**, sélectionnez **Alertes**. Sélectionnez **Nouvelle règle d’alerte**.  Le volet **Créer une règle d’alerte** s’ouvre.

    :::image type="content" source="media/tutorial-use-metrics-and-diags/create-alert-rule-pane.png" alt-text="Capture d’écran montrant le volet Créer une règle d’alerte.":::

    Le volet **Créer une règle d’alerte** comporte quatre sections :

    * L’**Étendue** est déjà définie sur votre hub IoT. Nous allons donc laisser cette section inchangée.
    * **Condition** définit le signal et les conditions qui déclencheront l’alerte.
    * **Actions** configure ce qui se produit lorsque l’alerte se déclenche.
    * **Détails de la règle d’alerte** vous permet de définir un nom et une description pour l’alerte.

1. Tout d’abord, configurez la condition sur laquelle l’alerte sera déclenchée.

    1. Sous **Condition**, sélectionnez **Sélectionner une condition**. Dans le volet **Configurer la logique du signal**, tapez « télémétrie » dans la zone de recherche, puis sélectionnez **Messages de télémétrie envoyés**.

       :::image type="content" source="media/tutorial-use-metrics-and-diags/configure-signal-logic-telemetry-messages-sent.png" alt-text="Capture d’écran montrant la sélection de la métrique.":::

    1. Dans le volet **Configurer la logique du signal**, définissez ou confirmez les champs suivants sous **Logique d’alerte** (vous pouvez ignorer le graphique) :

       **Seuil** :  *Statique*.

       **Opérateur** : *Supérieur à*.

       **Type d’agrégation** : *Total*.

       **Valeur de seuil** : 1000.

       **Précision d’agrégation (période)**  : *5 minutes*.

       **Fréquence d’évaluation** : *Toutes les minutes*

        :::image type="content" source="media/tutorial-use-metrics-and-diags/configure-signal-logic-set-conditions.png" alt-text="Capture d’écran montrant les paramètres des conditions d’alerte.":::

       Ces paramètres définissent le signal de façon à totaliser le nombre de messages sur une période de cinq minutes. Ce total sera évalué toutes les minutes et, si le total des cinq minutes précédentes dépasse 1000 messages, l’alerte sera déclenchée.

       Sélectionnez **Terminé** pour enregistrer la logique du signal.

1. À présent, configurez l’action pour l’alerte.

    1. De retour dans le volet **Créer une règle d’alerte**, sous **Actions**, sélectionnez **Sélectionner un groupe d’actions**. Dans le volet **Sélectionner un groupe d’actions à attacher à cette règle d’alerte**, sélectionnez **Créer un groupe d’actions**.

    1. Sous l’onglet **De base** du volet **Créer un groupe d’actions**, attribuez un nom et un nom d’affichage à votre groupe d’actions.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-action-group-basics.png" alt-text="Capture d’écran montrant l’onglet De base du volet Créer un groupe d’actions.":::

    1. Sélectionnez l’onglet **Notifications**. Comme **Type de notification**, sélectionnez **E-mail/SMS/Push/Voix** dans la liste déroulante. Le volet **E-mail/SMS/Push/Voix** s’ouvre.

    1. Dans le volet **E-mail/SMS/Push/Voix**, sélectionnez e-mail et entrez votre adresse e-mail, puis sélectionnez **OK**.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/set-email-address.png" alt-text="Capture d’écran montrant le paramètre d’adresse e-mail.":::

    1. De retour dans le volet **Notifications**, entrez un nom pour la notification.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-action-group-notification-complete.png" alt-text="Capture d’écran montrant le volet Notifications renseigné.":::

    1. (Facultatif) Si vous sélectionnez l’onglet **Actions**, puis la liste déroulante **Type d’action**, vous pouvez voir les types d’actions que vous pouvez déclencher avec une alerte. Dans le cadre de cet article, nous utiliserons uniquement des notifications. Vous pouvez donc ignorer les paramètres sous cet onglet.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/action-types.png" alt-text="Capture d’écran montrant les types d’action disponibles dans le volet Actions.":::

    1. Sélectionnez l’onglet **Vérifier + créer**, vérifiez vos paramètres, puis sélectionnez **Créer**.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-action-group-review-and-create.png" alt-text="Capture d’écran montrant le volet Vérifier + créer.":::

    1. De retour dans le volet **Créer une règle d’alerte**, vous remarquerez que votre nouveau groupe d’actions a été ajouté aux actions de l’alerte.  

1. Pour finir, configurez les détails de la règle d’alerte et enregistrez la règle d’alerte.

    1. Dans le volet **Créer une règle d’alerte**, sous Détails de la règle d’alerte, entrez un nom et une description pour votre alerte. Par exemple, « Alerte si plus de 1000 messages en cinq minutes ». Vérifiez que **Activer la règle d’alerte lors de la création** est activé. Votre règle d’alerte terminée doit ressembler à cette capture d’écran.

        :::image type="content" source="media/tutorial-use-metrics-and-diags/create-alert-rule-final.png" alt-text="Capture d’écran montrant le volet Créer une règle d’alerte renseigné.":::

    1. Sélectionnez **Créer une règle d’alerte** pour enregistrer votre nouvelle règle.

1. Maintenant, configurez une autre alerte pour le *Nombre total de messages utilisés*. Cette métrique est utile si vous souhaitez envoyer une alerte quand le nombre de messages utilisés approche le quota quotidien du hub IoT, stade auquel le hub commencera à rejeter les messages. Effectuez les mêmes étapes que précédemment, mais avec les différences suivantes.

    * Pour le signal dans le volet **Configurer la logique du signal**, sélectionnez **Nombre total de messages utilisés**.

    * Dans le volet **Configurer la logique du signal**, définissez ou confirmez les champs suivants (vous pouvez ignorer le graphique) :

       **Seuil** :  *Statique*.

       **Opérateur** : *Supérieur à*.

       **Type d’agrégation** : *Maximum*.

       **Valeur de seuil** : 4000.

       **Précision d’agrégation (période)**  : *1 minute*.

       **Fréquence d’évaluation** : *Toutes les minutes*

       Ces paramètres définissent le signal pour qu’il se déclenche lorsque le nombre de messages atteint 4000. La métrique est évaluée toutes les minutes.

    * Lorsque vous spécifiez l’action pour votre règle d’alerte, sélectionnez simplement le groupe d’actions que vous avez créé plus tôt.

    * Pour les détails de l’alerte, choisissez un nom et une description différents de ceux choisis plus tôt.

1. Sélectionnez **Alertes** sous **Supervision** dans le volet gauche de votre hub IoT. À présent, sélectionnez **Gérer les règles d’alerte** dans le menu en haut du volet **Alertes**. Le volet **Règles** s’ouvre. Vous devez maintenant voir vos deux alertes :

   :::image type="content" source="media/tutorial-use-metrics-and-diags/rules-management.png" alt-text="Capture d’écran montrant le volet Règles avec les nouvelles règles d’alerte.":::

1. Fermez le volet **Règles**.

Avec ces paramètres, une alerte est déclenchée et vous recevez une notification par e-mail lorsque plus de 1000 messages sont envoyés durant une période de cinq minutes et que le nombre total de messages utilisés dépasse 4000 (50 % du quota quotidien d’un hub IoT au niveau gratuit).

## <a name="run-the-simulated-device-app"></a>Exécution de l’application de périphérique simulé

Dans la section [Configurer des ressources](#set-up-resources), vous avez inscrit une identité d’appareil à utiliser pour simuler l’utilisation d’un appareil IoT. Dans cette section, vous allez télécharger une application console .NET qui simule un appareil envoyant des messages appareil-à-cloud à un hub IoT, la configurer pour envoyer ces messages à votre hub IoT, puis l’exécuter. 

> [!IMPORTANT]
>
> La configuration et l’activation des alertes par IoT Hub peut prendre jusqu’à 10 minutes. Attendez au moins 10 minutes entre le moment où vous configurez votre dernière alerte et l’exécution de l’application d’appareil simulé.

Téléchargez la solution pour la [simulation d’appareil IoT](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Ce lien permet de télécharger un dépôt contenant plusieurs applications. Celle que vous recherchez se trouve à l’emplacement iot-hub/Quickstarts/simulated-device/.

1. Dans une fenêtre de terminal local, accédez au dossier racine de la solution. Ensuite, accédez au dossier **iot-hub\Quickstarts\simulated-device**.

1. Utilisez un éditeur de texte pour ouvrir le fichier **SimulatedDevice.cs**.

    1. Remplacez la valeur de la variable `s_connectionString` par la chaîne de connexion de l’appareil que vous avez notée lorsque vous avez exécuté le script pour configurer des ressources.

    1. Dans la méthode `SendDeviceToCloudMessagesAsync`, remplacez la valeur 1000 de `Task.Delay` par 1, ce qui réduit l’intervalle de temps entre l’envoi de messages de 1 à 0,001 seconde. Le fait d’écourter ce délai augmente le nombre de messages envoyés. (Vous n’atteindrez probablement pas un taux de 100 messages par seconde.)

        ```csharp
        await Task.Delay(1);
        ```

    1. Enregistrez les changements apportés à **SimulatedDevice.cs**.

1. Dans la fenêtre de terminal local, exécutez la commande suivante pour installer les packages requis pour l’application d’appareil simulé :

    ```cmd/sh
    dotnet restore
    ```

1. Dans la fenêtre de terminal local, exécutez la commande suivante pour générer et exécuter l’application d’appareil simulé :

    ```cmd/sh
    dotnet run
    ```

    La capture d’écran suivante présente la sortie lorsque l’application d’appareil simulé envoie des données de télémétrie à votre IoT Hub :

    :::image type="content" source="media/tutorial-use-metrics-and-diags/simulated-device-output.png" alt-text="Capture d’écran montrant la sortie de l’appareil simulé.":::

Laissez l’application s’exécuter pendant au moins 10 à 15 minutes. Dans l’idéal, laissez-la s’exécuter jusqu’à ce qu’elle cesse d’envoyer des messages (environ 20 à 30 minutes). Cela se produira lorsque vous aurez dépassé le quota quotidien de messages pour votre hub IoT et qu’il aura cessé d’accepter des messages.

> [!NOTE]
> Si vous laissez l’application d’appareil s’exécuter pendant une période prolongée après l’arrêt de l’envoi des messages, vous risquez de recevoir une exception. Vous pouvez ignorer cette exception sans risque et fermer la fenêtre d’application.

## <a name="view-metrics-chart-on-your-dashboard"></a>Afficher le graphique des métriques dans votre tableau de bord

1. Dans le coin supérieur gauche du portail Azure, ouvrez le menu du portail, puis sélectionnez **Tableau de bord**.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/select-dashboard.png" alt-text="Capture d’écran de la sélection de votre tableau de bord.":::

1. Recherchez le graphique que vous avez épinglé et cliquez n’importe où sur la vignette en dehors des données du graphique pour le développer. Le graphique montre les messages de télémétrie envoyés et le nombre total de messages utilisés. Les nombres les plus récents apparaissent en bas du graphique. Vous pouvez déplacer le curseur dans le graphique pour afficher les valeurs de métrique pour des heures spécifiques. Vous pouvez également modifier la valeur de temps et la précision en haut du graphique pour affiner ou développer les données sur une période qui vous intéresse.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/metrics-on-dashboard-last-hour.png" alt-text="Capture d’écran montrant le graphique de métriques.":::

   Dans ce scénario, le débit des messages de l’appareil simulé n’est pas suffisamment élevé pour qu’IoT Hub limite ses messages. Dans un scénario qui implique réellement une limitation, vous constaterez peut-être que le nombre de messages de télémétrie envoyés dépasse le seuil de limitation de votre hub IoT pendant une période limitée. Cela permet de gérer le trafic en rafale. Pour plus d’informations, consultez [Régulation de flux](iot-hub-devguide-quotas-throttling.md#traffic-shaping).

## <a name="view-the-alerts"></a>Afficher les alertes

Lorsque le nombre de messages envoyés dépasse les limites que vous avez définies dans vos règles d’alerte, vous commencez à recevoir des alertes par e-mail.

Pour voir s’il existe des alertes actives, sélectionnez **Alertes** sous **Supervision** dans le volet gauche de votre hub IoT. Le volet **Alertes** montre le nombre d’alertes déclenchées, triées par gravité pour la période spécifiée.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/view-alerts.png" alt-text="Capture d’écran montrant la synthèse des alertes.":::

Sélectionnez la ligne correspondant à la gravité 3. Le volet **Toutes les alertes** s’ouvre et contient la liste des alertes de gravité 3 qui ont été déclenchées.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/view-all-alerts.png" alt-text="Capture d’écran montrant le volet Toutes les alertes.":::

Sélectionnez l’une des alertes pour en afficher les détails.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/view-individual-alert.png" alt-text="Capture d’écran montrant les détails de l’alerte.":::

Ouvrez votre boîte de réception pour consulter les e-mails de Microsoft Azure. La ligne d’objet décrit l’alerte qui a été déclenchée. Par exemple, *Azure : Gravité activée : 3 Alerte si plus de 1000 messages en cinq minutes*. Le corps ressemblera à l’image suivante :

   :::image type="content" source="media/tutorial-use-metrics-and-diags/alert-mail.png" alt-text="Capture d’écran de l’e-mail montrant les alertes déclenchées.":::

## <a name="view-azure-monitor-logs"></a>Afficher les journaux Azure Monitor

Dans la section [Collecter les journaux pour les connexions et la télémétrie des appareils](#collect-logs-for-connections-and-device-telemetry), vous avez créé un paramètre de diagnostic afin d’envoyer les journaux de ressources émis par votre hub IoT pour les opérations de télémétrie des connexions et des appareils vers les journaux à Azure Monitor. Dans cette section, vous allez exécuter une requête Kusto sur des journaux Azure Monitor pour observer les erreurs qui se sont produites.

1. Sous **Supervision** dans le volet gauche de votre hub IoT dans le portail Azure, sélectionnez **Journaux**. Fermez la fenêtre **Requêtes** initiale si elle s’ouvre.

1. Dans le volet Nouvelle requête, sélectionnez l’onglet **Requêtes**, puis développez **IoT Hub** pour afficher la liste des requêtes par défaut.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/new-query-pane.png" alt-text="Capture d’écran des requêtes par défaut IoT Hub.":::

1. Sélectionnez la requête *Récapitulatif de l’erreur*. La requête s’affiche dans le volet Éditeur de requête. Sélectionnez **Exécuter** dans le volet de l’éditeur et observez les résultats de la requête. Développez l’une des lignes pour afficher les détails.

   :::image type="content" source="media/tutorial-use-metrics-and-diags/logs-errors.png" alt-text="Capture d’écran des journaux retournés par la requête Récapitulatif de l’erreur.":::

   > [!NOTE]
   > Si vous ne voyez aucune erreur, essayez d’exécuter la requête *Appareils connectés récemment*. Elle doit retourner une ligne pour l’appareil simulé.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer toutes les ressources que vous avez créées dans ce tutoriel, supprimez le groupe de ressources. Cette opération supprime toutes les ressources contenues dans le groupe. En l’occurrence, elle supprime le hub IoT, l’espace de travail Log Analytics et le groupe de ressources proprement dit. Si vous avez épinglé des graphiques de métriques au tableau de bord, vous devez les supprimer manuellement en cliquant sur les trois points situés dans le coin supérieur droit de chaque métrique et en sélectionnant **Supprimer**. Veillez à enregistrer vos modifications après avoir supprimé les graphiques.

Pour supprimer le groupe de ressources, utilisez la commande [az group delete](/cli/azure/group#az-group-delete).

```azurecli-interactive
az group delete --name ContosoResources
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à utiliser des métriques et des journaux IoT Hub en effectuant les tâches suivantes :

> [!div class="checklist"]
>
> * Utiliser Azure CLI pour créer un hub IoT, inscrire un appareil simulé et créer un espace de travail Log Analytics  
> * Envoyer des connexions et des journaux de ressources de télémétrie d’appareil IoT Hub vers les journaux Azure Monitor dans l’espace de travail Log Analytics
> * Utiliser l’explorateur de métriques pour créer un graphique basé sur les métriques sélectionnées, et l’épingler à votre tableau de bord
> * Créer des alertes de métriques pour être averti par e-mail lorsque des conditions importantes se produisent
> * Télécharger et exécuter une application qui simule l’envoi de messages par un appareil IoT au hub IoT
> * Afficher les alertes lorsque vos conditions se produisent
> * Afficher le graphique des métriques dans votre tableau de bord
> * Afficher les erreurs et les opérations IoT Hub dans les journaux Azure Monitor

Passez au didacticiel suivant pour découvrir comment gérer l’état d’un appareil IoT. 

> [!div class="nextstepaction"]
> [Configurer vos appareils à partir d’un service back-end](tutorial-device-twins.md)