---
title: Résoudre les problèmes liés à Live Video Analytics sur IoT Edge - Azure
description: Cet article traite des étapes de dépannage pour les problèmes liés à Live Video Analytics sur IoT Edge.
author: IngridAtMicrosoft
ms.topic: how-to
ms.author: inhenkel
ms.date: 05/24/2020
ms.openlocfilehash: c297a189f3b13ca8e72daf4eef009bc28fac32bf
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91823199"
---
# <a name="troubleshoot-live-video-analytics-on-iot-edge"></a>Résoudre les problèmes liés à Live Video Analytics sur IoT Edge

Cet article traite des étapes de résolution des problèmes liés à Live Video Analytics (LVA) sur Azure IoT Edge.

## <a name="troubleshoot-deployment-issues"></a>Résoudre les problèmes de déploiement

### <a name="diagnostics"></a>Diagnostics

Dans le cadre de votre déploiement de Live Video Analytics, vous configurez des ressources Azure telles qu’IoT Hub et des appareils IoT Edge. Pour commencer à diagnostiquer les problèmes, assurez-vous toujours que l’appareil Edge est correctement configuré en suivant les instructions ci-dessous :

1. [Exécutez la commande `check`](../../iot-edge/troubleshoot.md#run-the-check-command).
1. [Vérifiez votre version d’IoT Edge](../../iot-edge/troubleshoot.md#check-your-iot-edge-version).
1. [Vérifiez l’état du gestionnaire de sécurité IoT Edge et de ses journaux](../../iot-edge/troubleshoot.md#check-the-status-of-the-iot-edge-security-manager-and-its-logs).
1. [Affichez les messages acheminés via le hub IoT Edge](../../iot-edge/troubleshoot.md#view-the-messages-going-through-the-iot-edge-hub).
1. [Redémarrez les conteneurs](../../iot-edge/troubleshoot.md#restart-containers).
1. [Vérifiez vos règles de configuration de pare-feu et de port](../../iot-edge/troubleshoot.md#check-your-firewall-and-port-configuration-rules).

### <a name="pre-deployment-issues"></a>Problèmes liés au prédéploiement

Si l’infrastructure Edge est correcte, vous pouvez rechercher des problèmes dans le fichier du manifeste de déploiement. Pour déployer le module Live Video Analytics sur IoT Edge sur l’appareil IoT Edge avec d’autres modules IoT, utilisez un manifeste de déploiement qui contient le hub Edge, l’agent IoT Edge et d’autres modules avec leurs propriétés. Si le code JSON n’est pas bien formé, vous pouvez recevoir l’erreur suivante : 

```
az iot edge set-modules --hub-name <iot-hub-name> --device-id lva-sample-device --content <path-to-deployment_manifest.json>
```

Impossible d’analyser JSON à partir du fichier : « <deployment manifest.json> » pour l’argument « content » avec l’exception : « Données supplémentaires : ligne 101 colonne 1 (char 5325) »

Si vous rencontrez cette erreur, nous vous recommandons de vérifier dans le fichier JSON s’il manque des crochets ou si la structure du fichier présente d’autres problèmes. Pour valider la structure des fichiers, vous pouvez utiliser un client tel que [Notepad++ avec le plug-in JSON Viewer](https://riptutorial.com/notepadplusplus/example/18201/json-viewer) ou un outil en ligne tel que [JSON Formatter & Validator](https://jsonformatter.curiousconcept.com/).

### <a name="during-deployment-diagnose-with-media-graph-direct-methods"></a>Pendant le déploiement : diagnostiquer à l’aide des méthodes directes du graphe multimédia 

Une fois que le module Live Video Analytics sur IoT Edge est correctement déployé sur l’appareil IoT Edge, vous pouvez créer et exécuter le graphe multimédia en appelant les [méthodes directes](direct-methods.md). Vous pouvez utiliser le portail Azure pour exécuter un diagnostic du graphe multimédia via les méthodes directes :

1. Dans le portail Azure, accédez au hub IoT connecté à votre appareil IoT Edge.

1. Recherchez **Gestion automatique des appareils**, puis sélectionnez **IoT Edge**.  

1. Dans la liste des appareils Edge, sélectionnez l’appareil que vous souhaitez diagnostiquer.  
         
    ![Capture d’écran du portail Azure affichant une liste d’appareils Edge](./media/troubleshoot-how-to/lva-sample-device.png)

1. Vérifiez si le code de réponse est *200-OK*. Les autres codes de réponse pour le [runtime IoT Edge](../../iot-edge/iot-edge-runtime.md) sont les suivants :
    * 400 - La configuration de déploiement a un format incorrect ou n’est pas valide.
    * 417 - Aucune configuration de déploiement n’est définie pour l’appareil.
    * 412 - La version de schéma dans la configuration de déploiement n’est pas valide.
    * 406 - L’appareil IoT Edge est hors connexion ou n’envoie pas de rapports d’état.
    * 500 - Une erreur s’est produite dans le runtime IoT Edge.

1. Si vous recevez le code d’état 501, vérifiez que le nom de la méthode directe est exact. Si le nom de la méthode et la charge utile de la requête sont exacts, les résultats devraient contenir le code de réussite = 200. Dans le cas où la charge utile de demande serait inexacte, vous obtiendrez l’état = 400 et la charge utile de réponse indiquera un code d’erreur et un message qui vous aideront à diagnostiquer le problème avec votre appel de méthode directe.
    * La vérification des propriétés rapportées et souhaitées peut vous aider à comprendre si les propriétés du module sont synchronisées avec le déploiement. Si ce n’est pas le cas, vous pouvez redémarrer votre appareil IoT Edge. 
    * Utilisez le guide [Méthodes directes](direct-methods.md) pour appeler un certain nombre de méthodes, notamment les plus simples comme GraphTopologyList. Ce guide précise également les charges utiles de demande et de réponse attendues ainsi que les codes d’erreur. Lorsque les méthodes directes simples ont réussi, vous pouvez être certain que le module IoT Edge Live Video Analytics fonctionne correctement.
        
       ![Capture d’écran du volet « Méthode directe » pour le module IoT Edge.](./media/troubleshoot-how-to/direct-method.png) 

1. Si les colonnes **Spécifié lors du déploiement** et **Signalé par l’appareil** indiquent *Oui*, alors vous pouvez appeler des méthodes directes sur le module Live Video Analytics sur IoT Edge. Sélectionnez le module pour accéder à une page dans laquelle vous pouvez vérifier les propriétés souhaitées et rapportées et appeler des méthodes directes. Gardez à l’esprit les points suivants : 

### <a name="post-deployment-diagnose-logs-for-issues-during-the-run"></a>Après le déploiement : diagnostiquer les journaux à la recherche des problèmes survenus lors de l’exécution 

Les journaux des conteneurs de votre module IoT Edge doivent contenir des informations de diagnostic pour vous aider à déboguer les problèmes survenus pendant l’exécution du module. Vous pouvez [vérifier si les journaux des conteneurs indiquent des problèmes](../../iot-edge/troubleshoot.md#check-container-logs-for-issues) et auto-diagnostiquer le problème. 

Si vous avez exécuté toutes les vérifications précédentes et que vous rencontrez toujours des problèmes, rassemblez les journaux à partir de l’appareil IoT Edge [avec la commande `support bundle`](../../iot-edge/troubleshoot.md#gather-debug-information-with-support-bundle-command) pour une analyse plus approfondie par l’équipe Azure. Vous pouvez [nous contacter](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) pour obtenir de l’aide et envoyer les journaux collectés.

## <a name="common-error-resolutions"></a>Résolutions des erreurs courantes

Live Video Analytics est déployé en tant que module IoT Edge sur l’appareil IoT Edge et fonctionne de manière collaborative avec les modules du hub et de l’agent IoT Edge. Certaines des erreurs courantes que vous pouvez rencontrer lors du déploiement de Live Video Analytics sont dues à des problèmes liés à l’infrastructure IoT sous-jacente. Les erreurs comprennent :

* [L’agent IoT Edge s’arrête après environ une minute](../../iot-edge/troubleshoot-common-errors.md#iot-edge-agent-stops-after-about-a-minute).
* [L’agent IoT Edge ne peut pas accéder à l’image d’un module (403)](../../iot-edge/troubleshoot-common-errors.md#iot-edge-agent-cant-access-a-modules-image-403).
* [Le module de l’agent IoT Edge rapporte le message « fichier config vide » et aucun module ne démarre sur l’appareil](../../iot-edge/troubleshoot-common-errors.md#edge-agent-module-reports-empty-config-file-and-no-modules-start-on-the-device).
* [Le hub IoT Edge ne parvient pas à démarrer](../../iot-edge/troubleshoot-common-errors.md#iot-edge-hub-fails-to-start).
* [Le démon de sécurité IoT Edge échoue en raison d’un nom d’hôte non valide](../../iot-edge/troubleshoot-common-errors.md#iot-edge-security-daemon-fails-with-an-invalid-hostname).
* [Le module Live Video Analytics ou tout autre module IoT Edge personnalisé ne parvient pas à envoyer un message au hub IoT Edge avec l’erreur 404](../../iot-edge/troubleshoot-common-errors.md#iot-edge-module-fails-to-send-a-message-to-edgehub-with-404-error).
* [Le module IoT Edge est correctement déployé, puis disparaît de l’appareil](../../iot-edge/troubleshoot-common-errors.md#iot-edge-module-deploys-successfully-then-disappears-from-device).

### <a name="edge-setup-script-issues"></a>Problèmes liés au script de configuration Edge

Dans le cadre de notre documentation, nous fournissons un [script de configuration](https://github.com/Azure/live-video-analytics/tree/master/edge/setup) pour déployer les ressources Edge et Cloud et bien prendre en main Live Video Analytics Edge. Cette section présente quelques erreurs de script que vous pourriez rencontrer, ainsi que des solutions pour les déboguer.

Problème : Le script s’exécute en créant partiellement quelques ressources, mais il échoue avec le message suivant :

```
registering device...

Unable to load extension 'eventgrid: unrecognized kwargs: ['min_profile']'. Use --debug for more information.
The command failed with an unexpected error. Here is the traceback:

No module named 'azure.mgmt.iothub.iot_hub_client'
Traceback (most recent call last):
File "/opt/az/lib/python3.6/site-packages/knack/cli.py", line 215, in invoke
  cmd_result = self.invocation.execute(args)
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 631, in execute
  raise ex
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 695, in _run_jobs_serially
  results.append(self._run_job(expanded_arg, cmd_copy))
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 688, in _run_job
  six.reraise(*sys.exc_info())
File "/opt/az/lib/python3.6/site-packages/six.py", line 693, in reraise
  raise value
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 665, in _run_job
  result = cmd_copy(params)
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/commands/__init__.py", line 324, in __call__
  return self.handler(*args, **kwargs)
File "/opt/az/lib/python3.6/site-packages/azure/cli/core/__init__.py", line 574, in default_command_handler
  return op(**command_args)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/operations/hub.py", line 75, in iot_device_list
  result = iot_query(cmd, query, hub_name, top, resource_group_name, login=login)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/operations/hub.py", line 45, in iot_query
  target = get_iot_hub_connection_string(cmd, hub_name, resource_group_name, login=login)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/common/_azure.py", line 112, in get_iot_hub_connection_string
  client = iot_hub_service_factory(cmd.cli_ctx)
File "/home/.azure/cliextensions/azure-cli-iot-ext/azext_iot/_factory.py", line 28, in iot_hub_service_factory
  from azure.mgmt.iothub.iot_hub_client import IotHubClient
ModuleNotFoundError: No module named 'azure.mgmt.iothub.iot_hub_client'
```
    
Pour résoudre ce problème :

1. Exécutez la commande suivante :

    ```
    az --version
    ```
1. Assurez-vous que les extensions suivantes sont installées. À compter de la publication de cet article, les extensions et leurs versions sont les suivantes :

    | Extension | Version |
    |---|---|
    |azure-cli   |      2.5.1*|
    |command-modules-nspkg         |   2.0.3|
    |core  |    2.5.1*|
    |nspkg    | 3.0.4|
    |télémétrie| 1.0.4|
    |storage-preview          |     0.2.10|
    |azure-cli-iot-ext          |    0.8.9|
    |eventgrid| 0.4.9|
    |azure-iot                       | 0.9.2|
1. Si vous disposez d’une extension installée dont la version est antérieure au numéro de version répertorié ici, mettez à jour l’extension à l’aide de la commande suivante :

    ```
    az extension update --name <Extension name>
    ```

    Par exemple, vous pouvez exécuter `az extension update --name azure-iot`.

### <a name="sample-app-issues"></a>Problèmes avec l’application exemple

Dans le cadre de notre publication, nous fournissons des exemples de code .NET pour aider notre communauté de développeurs à démarrer. Cette section présente quelques erreurs que vous pourriez rencontrer lors de l’exécution de l’exemple de code, ainsi que des solutions pour les déboguer.

Problème : Program.cs échoue avec l’erreur suivante sur l’appel de méthode directe :

```
Unhandled exception. Microsoft.Azure.Devices.Common.Exceptions.UnauthorizedException: {"Message":"{\"errorCode\":401002,\"trackingId\":\"b1da85801b2e4faf951a2291a2c467c3-G:32-TimeStamp:04/06/2020 17:15:11\",\"message\":\"Unauthorized\",\"timestampUtc\":\"2020-04-06T17:15:11.6990676Z\"}","ExceptionMessage":""}
    
        at Microsoft.Azure.Devices.HttpClientHelper.ExecuteAsync(HttpClient httpClient, HttpMethod httpMethod, Uri requestUri, Func`3 modifyRequestMessageAsync, Func`2 isMappedToException, Func`3 processResponseMessageAsync, IDictionary`2 errorMappingOverrides, CancellationToken cancellationToken)
    
        at Microsoft.Azure.Devices.HttpClientHelper.ExecuteAsync(HttpMethod httpMethod, Uri requestUri, Func`3 modifyRequestMessageAsync, Func`3 processResponseMessageAsync, IDictionary`2 errorMappingOverrides, CancellationToken cancellationToken)
        
        at Microsoft.Azure.Devices.HttpClientHelper.PostAsync[T,T2](Uri requestUri, T entity, TimeSpan operationTimeout, IDictionary`2 errorMappingOverrides, IDictionary`2 customHeaders, CancellationToken cancellationToken)…
```

1. Assurez-vous d’avoir installé [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) dans votre environnement Visual Studio Code et d’avoir configuré la connexion à votre hub IoT. Pour ce faire, sélectionnez Ctrl+Maj+P, puis choisissez **Sélectionner une méthode IoT Hub**.

1. Vérifiez si vous pouvez appeler une méthode directe sur le module IoT Edge via Visual Studio Code. Par exemple, appelez GraphTopologyList avec la charge utile suivante {&nbsp;"@apiVersion": "1.0"}. Vous devez recevoir la réponse suivante : 

    ```
    {
      "status": 200,
      "payload": {
        "values": [
          {…
    …}
          ]
        }
    }
    ```

    ![Capture d’écran de la réponse dans Visual Studio Code.](./media/troubleshoot-how-to/visual-studio-code1.png)
1. Si la solution précédente échoue, essayez ce qui suit :

    a. Accédez à l’invite de commandes sur votre appareil IoT Edge, puis exécutez la commande suivante :
    
      ```
      sudo systemctl restart iotedge
      ```

      Cette commande redémarre l’appareil IoT Edge et tous les modules. Patientez quelques minutes, puis, avant d’essayer à nouveau d’utiliser la méthode directe, vérifiez que les modules fonctionnent en exécutant la commande suivante :

      ```
      sudo iotedge list
      ```

    b. Si l’approche précédente échoue également, essayez de redémarrer votre machine virtuelle ou votre ordinateur.

    c. Si toutes les approches échouent, exécutez la commande suivante pour obtenir un fichier compressé avec tous les [journaux pertinents](../../iot-edge/troubleshoot.md#gather-debug-information-with-support-bundle-command) et joignez-le à un [ticket de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

    ```
    sudo iotedge support-bundle --since 2h
    ```

1. Si vous recevez le code de réponse d’erreur *400*, assurez-vous que la charge utile d’appel de votre méthode est bien formée, conformément au guide [Méthodes directes](direct-methods.md).
1. Si vous recevez le code d’état *200*, cela signifie que votre hub fonctionne correctement et que le déploiement de votre module est correct et réactif. 

1. Vérifiez si la configuration de l’application est correcte. La configuration de votre application se compose des champs suivants dans le fichier *appsettings.json*. Vérifiez soigneusement que deviceId et moduleId sont exacts. Un moyen facile de vérifier est d’accéder à la section de l’extension Azure IoT Hub dans Visual Studio Code. Les valeurs contenues dans le fichier *appsettings.json* et la section IoT Hub doivent correspondre.
    
    ```
    {
        "IoThubConnectionString" : 
        "deviceId" : 
        "moduleId" : 
    }
    ```

1. Dans le fichier *appsettings.json*, assurez-vous d’avoir fourni la chaîne de connexion IoT Hub et *non* la chaîne de connexion de l’appareil IoT Hub, car les [formats de chaîne de connexion sont différents](https://devblogs.microsoft.com/iotdev/understand-different-connection-strings-in-azure-iot-hub/).

### <a name="live-video-analytics-working-with-external-modules"></a>Live Video Analytics avec des modules externes

Live Video Analytics via le processeur d’extension HTTP peut étendre le graphe multimédia pour envoyer et recevoir des données à partir d’autres modules IoT Edge sur HTTP en utilisant REST. À titre d’[exemple spécifique](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/httpExtension), le graphe multimédia peut envoyer des trames vidéo en tant qu’images à un module d’inférence externe tel que Yolo v3 et recevoir des résultats d’analyse JSON. Dans ce type de topologie, la destination des événements est principalement le hub IoT. Dans les cas où vous ne voyez pas les événements d’inférence sur le hub, vérifiez les éléments suivants :

* Vérifiez si le hub sur lequel est publié le graphe multimédia est le même que celui que vous examinez. Lorsque vous créez plusieurs déploiements, vous pouvez vous retrouver avec plusieurs hubs et vous risquez d’inspecter le mauvais hub d’événement par erreur.
* Dans Visual Studio Code, vérifiez si le module externe est déployé et s’il fonctionne. Dans cette image d’exemple, rtspsim et cv sont des modules IoT Edge qui s’exécutent à l’extérieur du module lvaEdge.

    ![Capture d’écran qui affiche l’état de fonctionnement des modules dans Azure IoT Hub.](./media/troubleshoot-how-to/iot-hub.png)

* Vérifiez que vous envoyez des événements au point de terminaison d’URL approprié. Le conteneur d’IA externe expose une URL et un port par lesquels il reçoit et renvoie les données des requêtes POST. Cette URL est spécifiée en tant que propriété `endpoint: url` pour le processeur d’extension HTTP. Comme indiqué dans l’[URL de topologie](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/httpExtension/topology.json), le point de terminaison est défini sur le paramètre d’URL d’inférence. Assurez-vous que la valeur par défaut du paramètre ou la valeur transmise est exacte. Vous pouvez effectuer un test pour déterminer s’il fonctionne à l’aide de l’URL du client (cURL).  

    À titre d’exemple, voici un conteneur Yolo v3 qui s’exécute sur l’ordinateur local avec l’adresse IP 172.17.0.3. Utilisez l’inspection Docker pour trouver l’adresse IP.

    ```
    curl -X POST http://172.17.0.3/score -H "Content-Type: image/jpeg" --data-binary @<fullpath to jpg>
    ```

    Résultat retourné :

    ```
    {"inferences": [{"type": "entity", "entity": {"tag": {"value": "car", "confidence": 0.8668569922447205}, "box": {"l": 0.3853073438008626, "t": 0.6063712999658677, "w": 0.04174524943033854, "h": 0.02989496027381675}}}]}
    ```

* Si vous exécutez une ou plusieurs instances d’un graphe qui utilise le processeur d’extension HTTP, vous devez disposer d’un filtre de fréquence d’images avant chaque processeur d’extension HTTP pour gérer la fréquence d’images par seconde (FPS) du flux vidéo. 

   Dans certains cas, lorsque le processeur ou la mémoire de l’ordinateur Edge est fortement sollicité, vous pouvez perdre certains événements d’inférence. Pour résoudre ce problème, définissez une valeur faible pour la propriété maximumFps sur le filtre de fréquence d’images. Vous pouvez définir la valeur 0,5 (« FPS maximum » : 0.5) sur chaque instance du graphe, puis réexécuter l’instance pour rechercher les événements d’inférence sur le hub.

   Vous pouvez également obtenir un ordinateur Edge plus puissant avec un processeur et une mémoire plus élevés.
    
### <a name="multiple-direct-methods-in-parallel--timeout-failure"></a>Plusieurs méthodes directes en parallèle – échec au terme du délai d’expiration 

Live Video Analytics sur IoT Edge fournit un modèle de programmation basé sur une méthode directe qui vous permet de configurer plusieurs topologies et plusieurs instances de graphe. Dans le cadre de la configuration de la topologie et du graphe, vous allez procéder à plusieurs appels de méthode directe sur le module IoT Edge. Si vous invoquez ces multiples appels de méthode en parallèle, en particulier ceux qui démarrent et arrêtent les graphes,vous pourriez connaître un échec au terme du délai d’expiration, comme ci-dessous : 

La méthode d’Assembly Initialization Microsoft.Media.LiveVideoAnalytics.Test.Feature.Edge.AssemblyInitializer.InitializeAssemblyAsync a levé l’exception. Microsoft.Azure.Devices.Common.Exceptions.IotHubException : Microsoft.Azure.Devices.Common.Exceptions.IotHubException :<br/> `{"Message":"{\"errorCode\":504101,\"trackingId\":\"55b1d7845498428593c2738d94442607-G:32-TimeStamp:05/15/2020 20:43:10-G:10-TimeStamp:05/15/2020 20:43:10\",\"message\":\"Timed out waiting for the response from device.\",\"info\":{},\"timestampUtc\":\"2020-05-15T20:43:10.3899553Z\"}","ExceptionMessage":""}. Aborting test execution. `

Nous vous recommandons de *pas* appeler des méthodes directes en parallèle. Appelez-les de manière séquentielle (c’est-à-dire, effectuez un appel de méthode directe seulement après la fin du précédent).

### <a name="collect-logs-for-submitting-a-support-ticket"></a>Collecter les journaux pour l’envoi d’un ticket de support

Lorsque les étapes de résolution des problèmes auto-guidée ne résolvent pas votre problème, accédez au portail Azure et [ouvrez un ticket de support](../../azure-portal/supportability/how-to-create-azure-support-request.md).

> [!WARNING]
> Les journaux peuvent contenir des informations d’identification personnelle telles que votre adresse IP. Toutes les copies locales des journaux seront supprimées dès que nous aurons terminé leur examen et fermé le ticket de support.  

Pour réunir les journaux pertinents qui doivent être ajoutés au ticket, suivez les instructions des sections suivantes. Vous pouvez charger les fichiers journaux dans le volet **Détails** de la demande de support.

### <a name="use-the-support-bundle-command"></a>Utiliser la commande support-bundle

Lorsque vous devez récupérer des journaux d'activité depuis un périphérique IoT Edge, la façon la plus simple est d’utiliser la commande `support-bundle`. Cette commande récupère :

- Les journaux de module
- Les journaux du gestionnaire de sécurité IoT Edge et du moteur de conteneur
- Sortie Json de vérification Iotedge
- Des informations de débogage utiles

1. Exécutez la commande `support-bundle` avec l’indicateur *--since* pour spécifier la période que vos journaux doivent couvrir. Par exemple, « 2 h » vous permettra de récupérer les journaux des deux dernières heures. Vous pouvez modifier la valeur de cet indicateur pour inclure les journaux de différentes périodes.

    ```
    sudo iotedge support-bundle --since 2h
    ```

   Cette commande crée un fichier nommé *support_bundle.zip* dans le répertoire où vous avez exécuté la commande. 
   
1. Joignez le fichier *support_bundle.zip* au ticket de support.

### <a name="live-video-analytics-debug-logs"></a>Journaux de débogage de Live Video Analytics

Pour configurer le module Live Video Analytics sur IoT Edge afin de générer des journaux de débogage, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre hub IoT.
1. Dans le volet gauche, sélectionnez **IoT Edge**.
1. Dans la liste des appareils, sélectionnez l’ID de l’appareil cible.
1. En haut du volet, sélectionnez **Définir les modules**.

   ![Capture d’écran du bouton « Définir les modules » dans le portail Azure.](media/troubleshoot-how-to/set-modules.png)

1. Dans la section **Modules IoT Edge**, recherchez et sélectionnez **lvaEdge**.
1. Sélectionnez **Options de création de conteneur**.
1. Dans la section **Liens**, ajoutez la commande suivante  :

    `/var/local/mediaservices/logs:/var/lib/azuremediaservices/logs`

    > [!NOTE] 
    > Cette commande lie les dossiers de journaux entre l’appareil Edge et le conteneur. Si vous souhaitez collecter les journaux dans un emplacement différent, utilisez la commande suivante en remplaçant **$LOG_LOCATION_ON_EDGE_DEVICE** par l’emplacement que vous souhaitez utiliser : `/var/$LOG_LOCATION_ON_EDGE_DEVICE:/var/lib/azuremediaservices/logs`

1. Sélectionnez **Update**.
1. Sélectionnez **Vérifier + créer**. Un message de validation réussie s’affiche sous une bannière verte.
1. Sélectionnez **Create** (Créer).
1. Mettez à jour **Jumeau d’identité de module** pour pointer vers le paramètre DebugLogsDirectory, qui pointe vers le répertoire dans lequel les journaux sont collectés :

    a. Sous la table **Modules**, sélectionnez **lvaEdge**.  
    b. En haut du volet, sélectionnez **Jumeau d’identité de module**. Un volet modifiable s’ouvre.  
    c. Sous **Clé souhaitée**, ajoutez la paire clé-valeur suivante :  
    `"DebugLogsDirectory": "/var/lib/azuremediaservices/logs"`

    > [!NOTE] 
    > Cette commande lie les dossiers de journaux entre l’appareil Edge et le conteneur. Si vous souhaitez collecter les journaux à un autre emplacement sur l’appareil :
    > 1. Créez une liaison pour l’emplacement du journal de débogage dans la section **Liaisons**, en remplaçant **$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE** et **$DEBUG_LOG_LOCATION** par l’emplacement de votre choix : `/var/$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE:/var/$DEBUG_LOG_LOCATION`
    > 2. Utilisez la commande suivante, en remplaçant **$DEBUG_LOG_LOCATION** par l’emplacement utilisé à l’étape précédente :  
    > `"DebugLogsDirectory": "/var/$DEBUG_LOG_LOCATION"`  
    
    d. Sélectionnez **Enregistrer**.

1. Reproduisez le problème.
1. Connectez-vous à la machine virtuelle à partir de la page **IoT Hub** dans le portail.
1. Compressez tous les fichiers dans le dossier *debugLogs*.

   > [!NOTE]
   > Ces fichiers journaux ne sont pas destinés à l’auto-diagnostic. Ils permettent aux ingénieurs Azure d’analyser vos problèmes.

   a. Dans la commande suivante, veillez à remplacer **$DEBUG_LOG_LOCATION_ON_EDGE_DEVICE** par l’emplacement des journaux de débogage sur l’appareil Edge que vous avez configuré précédemment.  

   ```
   sudo apt install zip unzip  
   zip -r debugLogs.zip $DEBUG_LOG_LOCATION_ON_EDGE_DEVICE 
   ```

   b. Joignez le fichier *debugLogs.zip* au ticket de support.

1. Vous pouvez arrêter la collecte des journaux en définissant la valeur dans **Jumeau d’identité de module** sur *Null*. Retournez sur la page **Module Identity Twin** et mettez à jour le paramètre suivant comme suit :

    `"DebugLogsDirectory": ""`

## <a name="next-steps"></a>Étapes suivantes

[Tutoriel : Enregistrement vidéo basé sur les événements dans le Cloud et lecture à partir du Cloud](event-based-video-recording-tutorial.md)
