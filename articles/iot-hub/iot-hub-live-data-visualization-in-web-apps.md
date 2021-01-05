---
title: Visualisation en temps réel de données IoT Hub dans une application web
description: Utilisez une application web pour voir les données de température et d’humidité qui sont collectées par un capteur et envoyées à votre hub IoT.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 05/31/2019
ms.author: robinsh
ms.custom:
- 'Role: Cloud Development'
- 'Role: Data Analytics'
- devx-track-azurecli
ms.openlocfilehash: 3218df4601ef7a5e1b1e04c20c89eefc367b720b
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96746452"
---
# <a name="visualize-real-time-sensor-data-from-your-azure-iot-hub-in-a-web-application"></a>Visualiser les données de capteur en temps réel depuis votre hub Azure IoT dans une application web

![Diagramme de bout en bout](./media/iot-hub-live-data-visualization-in-web-apps/1_iot-hub-end-to-end-diagram.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

## <a name="what-you-learn"></a>Contenu

Ce tutoriel explique comment visualiser en temps réel les données de capteur que votre hub IoT reçoit au moyen d’une application web node.js s’exécutant sur votre ordinateur local. Après avoir exécuté l’application web localement, vous pouvez le cas échéant suivre les étapes permettant d’héberger l’application web dans Azure App Service. Si vous souhaitez essayer de visualiser les données dans votre IoT Hub avec Power BI, voir [Visualiser des données de capteur en temps réel depuis Azure IoT Hub à l’aide de Power BI](iot-hub-live-data-visualization-in-power-bi.md).

## <a name="what-you-do"></a>Procédure

* Ajouter un groupe de consommateurs à votre hub IoT que l’application web utilisera pour lire les données de capteur
* Télécharger le code de l’application web à partir de GitHub
* Examiner le code de l’application web
* Configurer les variables d’environnement pour stocker les artefacts IoT Hub nécessaires à votre application web
* Exécuter l’application web sur votre machine de développement
* Ouvrir une page web pour voir les données de température et d’humidité en temps réel à partir de votre hub IoT
* (Facultatif) Utiliser Azure CLI pour héberger votre application web dans Azure App Service

## <a name="what-you-need"></a>Ce dont vous avez besoin

* Suivre le tutoriel [Simulateur en ligne Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md) ou un des tutoriels de l’appareil, par exemple [Raspberry Pi avec node.js](iot-hub-raspberry-pi-kit-node-get-started.md). Ceux-ci couvrent les exigences suivantes :

  * Un abonnement Azure actif
  * Un IoT Hub associé à votre abonnement
  * Une application cliente qui envoie des messages à votre IoT Hub

* [Télécharger Git](https://www.git-scm.com/downloads)

* Les étapes de cet article supposent une machine de développement Windows. Vous pouvez, toutefois, effectuer ces étapes facilement sur un système Linux, dans l’interpréteur de commandes de votre choix.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="add-a-consumer-group-to-your-iot-hub"></a>Ajouter un groupe de consommateurs à votre instance IoT Hub

Les [Groupes de consommateurs](../event-hubs/event-hubs-features.md#event-consumers) fournissent des vues indépendantes dans le flux d’événements qui permettent aux applications et services Azure de consommer indépendamment des données à partir du même point de terminaison Event Hub. Dans cette section, vous ajoutez un groupe de consommateurs au point de terminaison intégré de votre hub IoT, point à partir duquel l’application web lira les données.

Exécutez la commande suivante pour ajouter un groupe de consommateurs au point de terminaison intégré de votre hub IoT :

```azurecli-interactive
az iot hub consumer-group create --hub-name YourIoTHubName --name YourConsumerGroupName
```

Notez le nom que vous choisissez, vous en aurez besoin plus loin dans ce tutoriel.

## <a name="get-a-service-connection-string-for-your-iot-hub"></a>Obtenir une chaîne de connexion de service pour votre hub IoT

Les hubs IoT sont créés avec plusieurs stratégies d’accès par défaut. L’une d’elles, la stratégie **service**, fournit des autorisations suffisantes pour qu’un service lise et écrive les points de terminaison du hub IoT. Exécutez la commande suivante afin d’obtenir une chaîne de connexion pour votre hub IoT qui respecte la stratégie de service :

```azurecli-interactive
az iot hub show-connection-string --hub-name YourIotHub --policy-name service
```

La chaîne de connexion doit ressembler à l’extrait suivant :

```javascript
"HostName={YourIotHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"
```

Notez la chaîne de connexion de service, vous en aurez besoin plus loin dans ce tutoriel.

## <a name="download-the-web-app-from-github"></a>Télécharger l’application web à partir de GitHub

Ouvrez une fenêtre de commande et entrez les commandes suivantes pour télécharger l’exemple à partir de GitHub et changer le répertoire de l’exemple :

```cmd
git clone https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization.git
cd web-apps-node-iot-hub-data-visualization
```

## <a name="examine-the-web-app-code"></a>Examiner le code de l’application web

À partir du répertoire web-apps-node-iot-hub-data-visualization, ouvrez l’application web dans votre éditeur favori. Ce qui suit montre la structure de fichiers affichée dans VS Code :

![Structure de fichiers de l’application web](./media/iot-hub-live-data-visualization-in-web-apps/web-app-files.png)

Prenez un moment pour examiner les fichiers suivants :

* **Server.js** est un script côté service qui initialise le socket web et la classe wrapper Event Hub. Il fournit un rappel à la classe wrapper Event Hub que la classe utilise pour diffuser des messages entrants vers le socket web.

* **Event-hub-reader.js** est un script côté service qui se connecte au point de terminaison intégré du hub IoT à l’aide du groupe de consommateurs et de la chaîne de connexion spécifiés. Il extrait DeviceId et EnqueuedTimeUtc depuis les métadonnées sur les messages entrants, puis relaie le message par le biais de la méthode de rappel inscrite par server.js.

* **Chart-device-data.js** est un script côté client qui écoute sur le socket web, effectue le suivi de chaque DeviceId et stocke les 50 derniers points de données entrantes pour chaque appareil. Il lie ensuite les données de l’appareil sélectionné à l’objet graphique.

* **Index.html** gère la disposition de l’interface utilisateur pour la page web, et référence les scripts nécessaires pour la logique côté client.

## <a name="configure-environment-variables-for-the-web-app"></a>Configurer des variables d’environnement pour l’application web

Pour lire les données à partir de votre hub IoT, l’application web a besoin de la chaîne de connexion de votre hub IoT, et du nom du groupe de consommateurs par l’intermédiaire duquel il doit lire. Elle obtient ces chaînes à partir de l’environnement du processus, dans les lignes suivantes de server.js :

```javascript
const iotHubConnectionString = process.env.IotHubConnectionString;
const eventHubConsumerGroup = process.env.EventHubConsumerGroup;
```

Définissez les variables d’environnement dans votre fenêtre de commande au moyen des commandes suivantes. Remplacez les valeurs d’espace réservé par la chaîne de connexion de service de votre hub IoT, et par le nom du groupe de consommateurs que vous avez créé précédemment. Ne mettez pas les chaînes entre guillemets.

```cmd
set IotHubConnectionString=YourIoTHubConnectionString
set EventHubConsumerGroup=YourConsumerGroupName
```

## <a name="run-the-web-app"></a>Exécuter l’application web

1. Assurez-vous que votre appareil est en cours d’exécution et qu’il envoie des données.

2. Dans la fenêtre de commande, exécutez les lignes suivantes pour télécharger et installer les packages référencés, puis démarrer le site web :

   ```cmd
   npm install
   npm start
   ```

3. Vous devez voir la sortie dans la console qui indique que l’application web s’est connectée correctement à votre hub IoT et qu’elle est à l’écoute sur le port 3000 :

   ![Application Web démarrée sur la console](./media/iot-hub-live-data-visualization-in-web-apps/web-app-console-start.png)

## <a name="open-a-web-page-to-see-data-from-your-iot-hub"></a>Ouvrir une page web pour afficher les données à partir de votre hub IoT

Ouvrez un navigateur et accédez à `http://localhost:3000`.

Dans la liste **Sélectionner un appareil**, sélectionnez votre appareil pour voir le tracé en cours des 50 derniers relevés de données de température et d’humidité envoyés par l’appareil à votre hub IoT.

![Page d’application web affichant l’humidité et la température en temps réel](./media/iot-hub-live-data-visualization-in-web-apps/web-page-output.png)

Vous devez également voir la sortie dans la console qui affiche les messages que votre application web diffuse au client du navigateur :  

![Sortie de la diffusion de l’application web sur la console](./media/iot-hub-live-data-visualization-in-web-apps/web-app-console-broadcast.png)

## <a name="host-the-web-app-in-app-service"></a>Héberger l’application web dans App Service

La [fonctionnalité Web Apps d’Azure App Service](../app-service/overview.md) fournit une plateforme en tant que service (PAAS) pour l’hébergement d’applications web. Les applications web hébergées dans Azure App Service peuvent bénéficier de puissantes fonctionnalités Azure, telles qu’une sécurité supplémentaire, l’équilibrage de charge et la scalabilité, ainsi que des solutions DevOps proposées par Azure et les partenaires, comme le déploiement continu, la gestion des packages, etc. Azure App Service prend en charge les applications web développées dans de nombreux langages connus, et déployées sur une infrastructure Windows ou Linux.

Dans cette section, vous provisionnez une application web dans App Service et y déployez votre code à l’aide de commandes Azure CLI. Les détails des commandes utilisées se trouvent dans la documentation [az webapp](/cli/azure/webapp?view=azure-cli-latest). Avant de commencer, assurez-vous que vous avez suivi les étapes pour [ajouter un groupe de ressources à votre hub IoT](#add-a-consumer-group-to-your-iot-hub), [obtenir une chaîne de connexion de service pour votre hub IoT](#get-a-service-connection-string-for-your-iot-hub) et [télécharger l’application web à partir de GitHub](#download-the-web-app-from-github).

1. Un [plan App Service](../app-service/overview-hosting-plans.md) définit un ensemble de ressources de calcul nécessaires à l’exécution d’une application hébergée dans App Service. Dans ce tutoriel, nous utilisons le niveau Développeur/Gratuit pour héberger l’application web. Avec le niveau Gratuit, votre application web s’exécute sur des ressources Windows partagées avec d’autres applications App Service, y compris les applications d’autres clients. Azure propose également des plans App Service pour déployer des applications web sur des ressources de calcul Linux. Vous pouvez ignorer cette étape si vous disposez déjà d’un plan App Service que vous souhaitez utiliser.

   Pour créer un plan App Service par l’intermédiaire du niveau gratuit de Windows, exécutez la commande suivante. Utilisez le même groupe de ressources que celui dans lequel se trouve votre hub IoT. Le nom de votre plan de service peut contenir des lettres majuscules et minuscules, des chiffres et des traits d’union.

   ```azurecli-interactive
   az appservice plan create --name <app service plan name> --resource-group <your resource group name> --sku FREE
   ```

2. À présent, provisionnez une application web dans votre plan App Service. Le paramètre `--deployment-local-git` active le code d’application web qui doit être chargé et déployé à partir d’un dépôt Git sur votre machine locale. Le nom de votre application web doit être globalement unique, il peut contenir des lettres majuscules et minuscules, des chiffres et des traits d’union. Veillez à spécifier la version 10.6 du nœud ou une version ultérieure pour le paramètre `--runtime`, selon la version du runtime Node.js que vous utilisez. Vous pouvez utiliser la commande `az webapp list-runtimes` pour obtenir une liste des runtimes pris en charge.

   ```azurecli-interactive
   az webapp create -n <your web app name> -g <your resource group name> -p <your app service plan name> --runtime "node|10.6" --deployment-local-git
   ```

3. Ajoutez maintenant les Paramètres de l’application pour les variables d’environnement qui spécifient la chaîne de connexion du hub IoT et le groupe de consommateurs du hub d’événements. Les paramètres individuels sont délimités par des espaces dans le paramètre `-settings`. Utilisez la chaîne de connexion de service de votre hub IoT, et le groupe de consommateurs que vous avez créé précédemment dans ce tutoriel. Ne mettez pas les valeurs entre guillemets.

   ```azurecli-interactive
   az webapp config appsettings set -n <your web app name> -g <your resource group name> --settings EventHubConsumerGroup=<your consumer group> IotHubConnectionString="<your IoT hub connection string>"
   ```

4. Activez le protocole Web Sockets pour l’application web et configurez l’application web pour recevoir des requêtes HTTPS uniquement (les requêtes HTTP sont redirigées vers HTTPS).

   ```azurecli-interactive
   az webapp config set -n <your web app name> -g <your resource group name> --web-sockets-enabled true
   az webapp update -n <your web app name> -g <your resource group name> --https-only true
   ```

5. Pour déployer le code sur App Service, vous allez utiliser vos [informations d’identification de déploiement au niveau utilisateur](../app-service/deploy-configure-credentials.md). Vos informations d’identification de déploiement au niveau de l’utilisateur sont différentes de vos informations d’identification Azure ; elles sont utilisées pour les déploiements FTP et Git local vers une application web. Une fois définies, elles sont valides sur l’ensemble de vos applications App Service, dans tous les abonnements de votre compte Azure. Si vous avez précédemment configuré des informations d’identification de déploiement au niveau utilisateur, vous pouvez les utiliser.

   Si vous ne l’avez pas fait, ou si vous avez oublié votre mot de passe, exécutez la commande suivante. Le nom d’utilisateur de déploiement doit être unique dans Azure, et il ne doit pas contenir le symbole « \@ » pour les push Git locaux. Lorsque vous y êtes invité, entrez et confirmez votre nouveau mot de passe. Le mot de passe doit comporter au moins huit caractères et inclure deux des trois éléments suivants : lettres, chiffres et symboles.

   ```azurecli-interactive
   az webapp deployment user set --user-name <your deployment user name>
   ```

6. Obtenez l’URL Git à utiliser pour envoyer votre code jusqu'à App Service.

   ```azurecli-interactive
   az webapp deployment source config-local-git -n <your web app name> -g <your resource group name>
   ```

7. Ajoutez une instance distante à votre clone qui référence le dépôt Git pour l’application web dans App Service. Pour \<Git clone URL\>, utilisez l’URL retournée à l’étape précédente. Exécutez la commande suivante dans votre fenêtre de commande.

   ```cmd
   git remote add webapp <Git clone URL>
   ```

8. Pour déployer le code sur App Service, entrez la commande suivante dans votre fenêtre de commande. Si vous êtes invité à entrer des informations d’identification, indiquez les informations d’identification de déploiement au niveau utilisateur que vous avez créées à l’étape 5. Vérifiez que vous envoyez à la branche principale de l’instance distante d’App Service.

    ```cmd
    git push webapp master:master
    ```

9. La progression du déploiement se met à jour dans votre fenêtre de commande. Un déploiement réussi se termine par des lignes ressemblant à la sortie suivante :

    ```cmd
    remote:
    remote: Finished successfully.
    remote: Running post deployment command(s)...
    remote: Deployment successful.
    To https://contoso-web-app-3.scm.azurewebsites.net/contoso-web-app-3.git
    6b132dd..7cbc994  master -> master
    ```

10. Exécutez la commande suivante pour interroger l’état de votre application web et vous assurer qu’elle est en cours d’exécution :

    ```azurecli-interactive
    az webapp show -n <your web app name> -g <your resource group name> --query state
    ```

11. Dans un navigateur, accédez à `https://<your web app name>.azurewebsites.net`. Une page web s’affiche, similaire à celle que vous avez vue lorsque vous avez exécuté l’application web localement. Si votre appareil est en cours d’exécution et qu’il envoie des données, vous devez voir s’afficher un tracé des 50 derniers relevés de température et d’humidité envoyés par l’appareil.

## <a name="troubleshooting"></a>Dépannage

Si vous rencontrez des problèmes avec cet exemple, consultez les étapes décrites dans les sections suivantes. Si vous rencontrez toujours des problèmes, faites-nous part de vos commentaires au bas de cette rubrique.

### <a name="client-issues"></a>Problèmes de clients

* Si un appareil n’apparaît pas dans la liste, ou si aucun graph n’est dessiné, vérifiez que le code de l’appareil est en cours d’exécution sur votre appareil.

* Dans le navigateur, ouvrez les outils de développement (dans de nombreux navigateurs, la touche F12 déclenche leur ouverture) et recherchez la console. Recherchez les éventuels avertissements ou erreurs imprimés dedans.

* Vous pouvez déboguer un script côté client dans /js/chat-device-data.js.

### <a name="local-website-issues"></a>Problèmes de site web local

* Regardez la sortie dans la fenêtre où vous avez lancé le nœud pour la sortie de console.

* Déboguez le code du serveur, en particulier server.js et /scripts/event-hub-reader.js.

### <a name="azure-app-service-issues"></a>Problèmes Azure App Service

* Dans le portail Azure, accédez à votre application web. Dans le volet de gauche, sous **Surveillance**, sélectionnez **Journaux App Service**. Activez **Journal des applications (Système de fichiers)** , définissez **Niveau** sur Erreur, puis sélectionnez **Enregistrer**. Ouvrez ensuite **Flux de journaux** (sous **Surveillance**).

* À partir de votre application web dans le portail Azure, sous **Outils de développement**, sélectionnez **Console** et validez des versions node et npm avec `node -v` et `npm -v`.

* Si vous voyez une erreur au sujet d’un package introuvable, il est possible que vous ayez exécuté les étapes dans le désordre. Lorsque le site est déployé (avec `git push`), le service d’application exécute `npm install`, qui s’exécute en fonction de la version actuelle de node qu’il a configurée. Si cette configuration est modifiée ultérieurement, vous devrez apporter une modification mineure au code, et l’envoyer (push) de nouveau.

## <a name="next-steps"></a>Étapes suivantes

Vous avez correctement utilisé votre application web pour visualiser en temps réel des données de capteur à partir de votre IoT Hub.

Pour découvrir une autre façon de visualiser des données depuis Azure IoT Hub, voir [Utiliser Power BI pour afficher les données de capteur en temps réel depuis votre hub IoT](iot-hub-live-data-visualization-in-power-bi.md).

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
