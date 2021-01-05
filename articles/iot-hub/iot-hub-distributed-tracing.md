---
title: Ajouter des ID de corrélation aux messages IoT avec le traçage distribué (préversion)
description: Découvrez comment utiliser la fonctionnalité de traçage distribué pour suivre les messages IoT dans tous les services Azure utilisés par votre solution.
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/06/2019
ms.author: jlian
ms.custom:
- amqp
- mqtt
- fasttrack-edit
- iot
ms.openlocfilehash: f8d37cf8f23de1d0535c7a9ff4a95ac217eddf74
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96452384"
---
# <a name="trace-azure-iot-device-to-cloud-messages-with-distributed-tracing-preview"></a>Suivre les messages appareil-à-cloud Azure IoT avec le traçage distribué (préversion)

Le traçage distribué est une [fonctionnalité en préversion](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) de Microsoft Azure IoT Hub.

IoT Hub est l’un des premiers services Azure à prendre en charge le traçage distribué. Lorsque le traçage distribué sera pris en charge par davantage de services Azure, vous pourrez suivre les messages IoT dans tous les services Azure qui sont impliqués dans votre solution. Pour obtenir des informations sur le traçage distribué, consultez [Traçage distribué](../azure-monitor/app/distributed-tracing.md).

L’activation du traçage distribué pour IoT Hub vous permet de :

- Superviser précisément le flux de chaque message via IoT Hub à l’aide du [contexte de trace](https://github.com/w3c/trace-context). Ce contexte de trace inclut des ID de corrélation qui vous permettent de corréler les événements d’un composant avec les événements d’un autre composant. Il peut être appliqué à un sous-ensemble de messages ou à tous les messages des appareils IoT à l’aide d’un [jumeau d’appareil](iot-hub-devguide-device-twins.md).
- Journaliser automatiquement le contexte de trace dans des [journaux Azure Monitor](monitor-iot-hub.md).
- Mesurer et comprendre le flux et la latence des messages entre les appareils, IoT Hub et les points de terminaison de routage.
- Commencer à réfléchir à la manière dont vous souhaitez implémenter le traçage distribué pour les services non Azure dans votre solution IoT.

Dans cet article, vous allez utiliser [Azure IoT device SDK pour le langage C](iot-hub-device-sdk-c-intro.md) avec le traçage distribué. La prise en charge du traçage distribué par les autres SDK est en cours.

## <a name="prerequisites"></a>Prérequis

- Pour le moment, la préversion du traçage distribué est uniquement prise en charge par les hubs IoT créés dans les régions suivantes :

  - **Europe Nord**
  - **Asie Sud-Est**
  - **USA Ouest 2**

- Cet article suppose que vous savez envoyer des messages de télémétrie aux hubs IoT. Vous devez avoir suivi le [guide de démarrage rapide sur l’envoi de données de télémétrie en C](quickstart-send-telemetry-c.md).

- Inscrivez un appareil auprès de votre hub IoT (les étapes sont indiquées dans tous les guides de démarrage rapide), puis notez la chaîne de connexion.

- Installez la dernière version de [Git](https://git-scm.com/download/).

## <a name="configure-iot-hub"></a>Configurer IoT Hub

Dans cette section, vous allez configurer un hub IoT pour qu’il journalise les attributs de traçage distribué (ID de corrélation et horodatages).

1. Accédez à votre hub IoT dans le [portail Azure](https://portal.azure.com/).

1. Dans le volet gauche de votre hub IoT, faites défiler jusqu’à la section **Supervision**, puis cliquez sur **Paramètres de diagnostic**.

1. Cliquez sur **Ajouter le paramètre de diagnostic**.

1. Dans le champ **Nom**, entrez un nom pour le nouveau paramètre de diagnostic. Par exemple, **DistributedTracingSettings**.

1. Choisissez une ou plusieurs des options suivantes qui déterminent où les journaux doivent être envoyés :

    - **Archiver dans un compte de stockage** : configurez le compte de stockage qui doit contenir les informations de journalisation.
    - **Diffuser vers un hub d’événements** : configurez le hub d’événements qui doit contenir les informations de journalisation.
    - **Envoyer à Log Analytics** : configurez l’espace de travail Log Analytics qui doit contenir les informations de journalisation.

1. Dans la section **Journal**, sélectionnez les opérations au sujet desquelles vous souhaitez obtenir des informations de journalisation.

    Veillez à inclure **DistributedTracing**, puis configurez une durée de **Conservation** pour les journaux. La conservation des journaux n’a pas d’impact sur les coûts de stockage.

    ![Capture d’écran montrant où se trouve la catégorie DistributedTracing pour les paramètres de diagnostic IoT](./media/iot-hub-distributed-tracing/diag-logs.png)

1. Cliquez sur **Enregistrer** pour enregistrer le nouveau paramètre.

1. (Facultatif) Pour voir le flux des messages à différents emplacements, configurez des [règles de routage pour au moins deux points de terminaison différents](iot-hub-devguide-messages-d2c.md).

Une fois que la journalisation est activée, IoT Hub enregistre un journal lorsqu’un message contenant des propriétés de trace valides est trouvé dans l’une des situations suivantes :

- Les messages arrivent à la passerelle du hub IoT.
- Le message est traité par le hub IoT.
- Le message est routé vers des points de terminaison personnalisés. Le routage doit être activé.

Pour plus d’informations sur ces journaux d’activité et ces schémas, consultez [Surveiller IoT Hub](monitor-iot-hub.md) et [Traçage distribué dans les journaux de ressources IoT Hub](monitor-iot-hub-reference.md#distributed-tracing-preview).

## <a name="set-up-device"></a>Configurer l’appareil

Dans cette section, vous allez préparer un environnement de développement à utiliser avec le [SDK Azure IoT pour le langage C](https://github.com/Azure/azure-iot-sdk-c). Ensuite, vous allez modifier un des exemples pour activer le traçage distribué sur les messages de télémétrie de votre appareil.

Ces instructions concernent la création de l’exemple sur un système Windows. Pour les autres environnements, consultez [Compile the C SDK](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#compile) ou [Prepackaged C SDK for Platform Specific Development](https://github.com/Azure/azure-iot-sdk-c/blob/master/iothub_client/readme.md#prepackaged-c-sdk-for-platform-specific-development).

### <a name="clone-the-source-code-and-initialize"></a>Cloner le code source et initialiser

1. Installez la charge de travail [« Développement Desktop en C++ »](/cpp/build/vscpp-step-0-installation?view=vs-2019) pour Visual Studio 2019. Visual Studio 2017 et 2015 sont également pris en charge.

1. Installez [CMake](https://cmake.org/). Vérifiez qu’il se trouve dans votre chemin (`PATH`) en tapant `cmake -version` dans une invite de commandes.

1. Ouvrez une invite de commandes ou l’interpréteur de commandes Git Bash. Exécutez les commandes suivantes pour cloner la dernière version du dépôt GitHub du [kit de développement logiciel (SDK) C Azure IoT](https://github.com/Azure/azure-iot-sdk-c) :

    ```cmd
    git clone -b public-preview https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Attendez-vous à ce que cette opération prenne plusieurs minutes.

1. Créez un sous-répertoire `cmake` dans le répertoire racine du référentiel Git et accédez à ce dossier. Exécutez les commandes suivantes à partir du répertoire `azure-iot-sdk-c` :

    ```cmd
    mkdir cmake
    cd cmake
    cmake ..
    ```

    Si `cmake` ne trouve pas votre compilateur C++, vous obtiendrez peut-être des erreurs de build lors de l’exécution de la commande ci-dessus. Si cela se produit, essayez d’exécuter cette commande dans [l’invite de commandes de Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs). 

    Une fois la génération terminée, les dernières lignes de sortie doivent ressembler à la sortie suivante :

    ```cmd
    $ cmake ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

### <a name="edit-the-send-telemetry-sample-to-enable-distributed-tracing"></a>Modifier l’exemple d’envoi de données de télémétrie pour activer le traçage distribué

> [!div class="button"]
> <a href="https://github.com/Azure-Samples/azure-iot-distributed-tracing-sample/blob/master/iothub_ll_telemetry_sample-c/iothub_ll_telemetry_sample.c" target="_blank">Obtenir l’exemple sur GitHub</a>

1. Utilisez un éditeur pour ouvrir le fichier source `azure-iot-sdk-c/iothub_client/samples/iothub_ll_telemetry_sample/iothub_ll_telemetry_sample.c`.

1. Recherchez la déclaration de la constante `connectionString` :

    [!code-c[](~/samples-iot-distributed-tracing/iothub_ll_telemetry_sample-c/iothub_ll_telemetry_sample.c?name=snippet_config&highlight=2)]

    Remplacez la valeur de la constante `connectionString` par la chaîne de connexion d’appareil que vous avez notée dans la section [Inscrire un appareil](./quickstart-send-telemetry-c.md#register-a-device) du [Guide de démarrage rapide sur l’envoi de données de télémétrie en C](./quickstart-send-telemetry-c.md).

1. Définissez `MESSAGE_COUNT` sur `5000` :

    [!code-c[](~/samples-iot-distributed-tracing/iothub_ll_telemetry_sample-c/iothub_ll_telemetry_sample.c?name=snippet_config&highlight=3)]

1. Recherchez la ligne de code qui appelle `IoTHubDeviceClient_LL_SetConnectionStatusCallback` pour inscrire une fonction de rappel d’état de connexion avant la boucle d’envoi de messages. Ajoutez le code sous cette ligne, comme indiqué ci-dessous, pour appeler `IoTHubDeviceClient_LL_EnablePolicyConfiguration` et activer le traçage distribué sur l’appareil :

    [!code-c[](~/samples-iot-distributed-tracing/iothub_ll_telemetry_sample-c/iothub_ll_telemetry_sample.c?name=snippet_tracing&highlight=5)]

    La fonction `IoTHubDeviceClient_LL_EnablePolicyConfiguration` active des stratégies pour certaines fonctionnalités IoT Hub qui sont configurées via les [jumeaux d’appareil](./iot-hub-devguide-device-twins.md). Une fois que `POLICY_CONFIGURATION_DISTRIBUTED_TRACING` est activé avec la ligne de code ci-dessus, le comportement de traçage de l’appareil reflète les modifications de traçage distribué qui ont été apportées au jumeau d’appareil.

1. Pour continuer d’exécuter l’exemple d’application sans épuiser votre quota, ajoutez un délai d’une seconde à la fin de la boucle d’envoi de messages :

    [!code-c[](~/samples-iot-distributed-tracing/iothub_ll_telemetry_sample-c/iothub_ll_telemetry_sample.c?name=snippet_sleep&highlight=8)]

### <a name="compile-and-run"></a>Compilation et exécution

1. Accédez au répertoire du projet *iothub_ll_telemetry_sample* à partir du répertoire CMake (`azure-iot-sdk-c/cmake`) créé précédemment, puis compilez l’exemple :

    ```cmd
    cd iothub_client/samples/iothub_ll_telemetry_sample
    cmake --build . --target iothub_ll_telemetry_sample --config Debug
    ```

1. Exécutez l'application. L’appareil envoie des données de télémétrie qui prennent en charge le traçage distribué.

    ```cmd
    Debug/iothub_ll_telemetry_sample.exe
    ```

1. Gardez l’application active. Si vous le souhaitez, vous pouvez examiner le message envoyé à IoT Hub via la fenêtre de console.

<!-- For a client app that can receive sampling decisions from the cloud, check out [this sample](https://aka.ms/iottracingCsample).  -->

### <a name="workaround-for-third-party-clients"></a>Solution de contournement pour les clients tiers

Il **n’est pas simple** d’afficher un aperçu de la fonctionnalité de traçage distribué sans utiliser le Kit de développement logiciel (SDK) C. Par conséquent, cette approche n’est pas recommandée.

Tout d’abord, vous devez implémenter toutes les primitives de protocole IoT Hub dans vos messages en suivant le guide de développement [Créer et lire des messages IoT Hub](iot-hub-devguide-messages-construct.md). Ensuite, modifiez les propriétés de protocole dans les messages MQTT/AMQP pour ajouter `tracestate` en tant que **propriété système**. Plus précisément :

* Pour MQTT, ajoutez `%24.tracestate=timestamp%3d1539243209` à la rubrique du message, où `1539243209` doit être remplacé par l’heure de création du message dans le format d’horodatage unix. Pour consulter un exemple, reportez-vous à l’implémentation [dans le Kit de développement logiciel (SDK) C](https://github.com/Azure/azure-iot-sdk-c/blob/6633c5b18710febf1af7713cf1a336fd38f623ed/iothub_client/src/iothubtransport_mqtt_common.c#L761)
* Pour AMQP, ajoutez `key("tracestate")` et `value("timestamp=1539243209")` en tant qu’annotation de message. Pour consulter une implémentation de référence, reportez-vous [ici](https://github.com/Azure/azure-iot-sdk-c/blob/6633c5b18710febf1af7713cf1a336fd38f623ed/iothub_client/src/uamqp_messaging.c#L527).

Pour contrôler le pourcentage de messages contenant cette propriété, implémentez une logique qui écoute les événements initiés par le cloud, tels que les mises à jour des jumeaux.

## <a name="update-sampling-options"></a>Modifier les options d’échantillonnage 

Pour modifier le pourcentage de messages devant être suivis à partir du cloud, vous devez mettre à jour le jumeau d’appareil. Pour cela, vous pouvez procéder de plusieurs façons, notamment en utilisant l’éditeur JSON dans le portail ou le SDK du service IoT Hub. Les sous-sections suivantes fournissent des exemples.

### <a name="update-using-the-portal"></a>Mettre à jour à l’aide du portail

1. Accédez à votre hub IoT dans le [portail Azure](https://portal.azure.com/), puis cliquez sur **Appareils IoT**.

1. Cliquez sur votre appareil.

1. Recherchez **Activer le traçage distribué (préversion)** , puis sélectionnez **Activer**.

    ![Activer le traçage distribué dans le portail Azure](./media/iot-hub-distributed-tracing/azure-portal.png)

1. Choisissez un **taux d’échantillonnage** compris entre 0 et 100 %.

1. Cliquez sur **Enregistrer**.

1. Attendez quelques secondes, puis appuyez sur **Actualiser**. En guise de confirmation, une icône de synchronisation symbolisée par une coche s’affiche.

1. Revenez à la fenêtre de console pour afficher l’application de messages de télémétrie. Vous voyez que `tracestate` est compris dans les propriétés d’application des messages envoyés.

    ![État du traçage](./media/iot-hub-distributed-tracing/MicrosoftTeams-image.png)

1. (Facultatif) Modifiez la valeur du taux d’échantillonnage, puis observez que la fréquence à laquelle `tracestate` est inclus dans les propriétés d’application n’est plus la même.

### <a name="update-using-azure-iot-hub-for-vs-code"></a>Mettre à jour à l’aide d’Azure IoT Hub pour VS Code

1. Installez VS Code, puis [cliquez ici](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) pour installer la dernière version d’Azure IoT Hub pour VS Code.

1. Ouvrez VS Code, puis [configurez la chaîne de connexion IoT Hub](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-toolkit#user-content-prerequisites).

1. Développez l’appareil puis recherchez **Distributed Tracing Setting (Preview)** (Paramètres de traçage distribué (préversion)). Dans cette section, cliquez sur le sous-noeud **Update Distributed Tracing Setting (Preview)** (Mettre à jour les paramètres de traçage distribué (préversion)).

    ![Activer le traçage distribué dans l’extension Azure IoT Hub](./media/iot-hub-distributed-tracing/update-distributed-tracing-setting-1.png)

1. Dans la fenêtre contextuelle, sélectionnez **Activer**, puis appuyez sur Entrée pour confirmer le taux d’échantillonnage de 100.

    ![Modifier le mode d’échantillonnage](./media/iot-hub-distributed-tracing/update-distributed-tracing-setting-2.png)

    ![Modifier le taux d’échantillonnage](./media/iot-hub-distributed-tracing/update-distributed-tracing-setting-3.png)

### <a name="bulk-update-for-multiple-devices"></a>Mettre à jour plusieurs appareils en bloc

Pour mettre à jour la configuration d’échantillonnage du traçage distribué pour plusieurs appareils, utilisez la [configuration automatique des appareils](./iot-hub-automatic-device-management.md). Vous devez suivre ce schéma pour les jumeaux :

```json
{
    "properties": {
        "desired": {
            "azureiot*com^dtracing^1": {
                "sampling_mode": 1,
                "sampling_rate": 100
            }
        }
    }
}
```

| Nom de l'élément | Obligatoire | Type | Description |
|-----------------|----------|---------|-----------------------------------------------------|
| `sampling_mode` | Oui | Integer | Deux valeurs de mode sont prises en charge pour activer et désactiver l’échantillonnage. `1` signifie Activé et `2` signifie Désactivé. |
| `sampling_rate` | Oui | Integer | Cette valeur est un pourcentage. Seules les valeurs de `0` à `100` (inclus) sont autorisées.  |

## <a name="query-and-visualize"></a>Interroger et visualiser

Pour voir toutes les traces journalisées par un hub IoT, interrogez le magasin de journaux que vous avez sélectionné dans les paramètres de diagnostic. Dans cette section, deux options vous sont expliquées.

### <a name="query-using-log-analytics"></a>Interroger avec Log Analytics

Si vous avez configuré [Log Analytics avec des journaux de ressources](../azure-monitor/platform/resource-logs.md#send-to-azure-storage), exécutez une requête pour rechercher les journaux d’activité appartenant à la catégorie `DistributedTracing`. Par exemple, la requête suivante montre toutes les traces journalisées :

```Kusto
// All distributed traces 
AzureDiagnostics 
| where Category == "DistributedTracing" 
| project TimeGenerated, Category, OperationName, Level, CorrelationId, DurationMs, properties_s 
| order by TimeGenerated asc  
```

Voici des exemple de journaux d’activité tels qu’ils apparaissent dans Log Analytics :

| TimeGenerated | NomOpération | Category | Level | CorrelationId | DurationMs | Propriétés |
|--------------------------|---------------|--------------------|---------------|---------------------------------------------------------|------------|------------------------------------------------------------------------------------------------------------------------------------------|
| 2018-02-22T03:28:28.633Z | DiagnosticIoTHubD2C | DistributedTracing | Informationnel | 00-8cd869a412459a25f5b4f31311223344-0144d2590aacd909-01 |  | {"deviceId":"AZ3166","messageSize":"96","callerLocalTimeUtc":"2018-02-22T03:27:28.633Z","calleeLocalTimeUtc":"2018-02-22T03:27:28.687Z"} |
| 2018-02-22T03:28:38.633Z | DiagnosticIoTHubIngress | DistributedTracing | Informationnel | 00-8cd869a412459a25f5b4f31311223344-349810a9bbd28730-01 | 20 | {"isRoutingEnabled":"false","parentSpanId":"0144d2590aacd909"} |
| 2018-02-22T03:28:48.633Z | DiagnosticIoTHubEgress | DistributedTracing | Informationnel | 00-8cd869a412459a25f5b4f31311223344-349810a9bbd28730-01 | 23 | {"endpointType":"EventHub","endpointName":"myEventHub", "parentSpanId":"0144d2590aacd909"} |

Pour connaître les différents types de journaux d’activité, consultez [Journaux de traçage distribués Azure IoT Hub](monitor-iot-hub-reference.md#distributed-tracing-preview).

### <a name="application-map"></a>Mise en correspondance d'applications

Pour visualiser le flux de messages IoT, configurez l’exemple d’application Application Map. L’exemple d’application envoie les journaux d’activité de traçage distribué à [Application Map](../azure-monitor/app/app-map.md) à l’aide d’une fonction Azure et d’un hub d’événements.

> [!div class="button"]
> <a href="https://github.com/Azure-Samples/e2e-diagnostic-provision-cli" target="_blank">Obtenir l’exemple sur GitHub</a>

L’image ci-dessous montre un traçage distribué dans Application Map, avec trois points de terminaison de routage :

![Traçage distribué IoT dans Application Map](./media/iot-hub-distributed-tracing/app-map.png)

## <a name="understand-azure-iot-distributed-tracing"></a>Comprendre le traçage distribué Azure IoT

### <a name="context"></a>Context

De nombreuses solutions IoT, y compris notre propre [architecture de référence](/azure/architecture/reference-architectures/iot) (en anglais uniquement), suivent généralement une variante de l’[architecture des microservices](/azure/architecture/microservices/). Lorsque votre solution IoT gagne en complexité, vous vous retrouvez à utiliser une dizaine de microservices, voire davantage. Ces microservices peuvent ou non provenir d’Azure. Il peut être difficile de déterminer à quel endroit les messages IoT ralentissent. Par exemple, vous disposez d’une solution IoT qui utilise cinq services Azure différents et 1 500 appareils actifs. Chaque appareil envoie 10 messages appareil-à-cloud par seconde (pour un total de 15 000 messages/seconde), mais vous remarquez que votre application web ne voit que 10 000 messages par seconde. Où se situe le problème ? Comment trouver le responsable ?

### <a name="distributed-tracing-pattern-in-microservice-architecture"></a>Modèle de traçage distribué dans l’architecture de microservice

Pour recréer le flux d’un message IoT dans différents services, chaque service doit propager un *ID de corrélation* qui identifie le message de façon unique. Une fois collectés dans un système centralisé, les ID de corrélation permettent de voir le flux des messages. Cette méthode est appelée [modèle de traçage distribué](/azure/architecture/microservices/logging-monitoring#distributed-tracing).

En vue de permettre une plus large adoption du traçage distribué, Microsoft contribue à une [proposition de norme W3C pour le traçage distribué](https://w3c.github.io/trace-context/).

### <a name="iot-hub-support"></a>Prise en charge d’IoT Hub

Une fois activée, la prise en charge du traçage distribué IoT Hub va suivre ce flux :

1. Un message est généré sur l’appareil IoT.
1. L’appareil IoT décide (avec l’aide du cloud) que ce message doit se voir attribuer un contexte de trace.
1. Le Kit de développement logiciel (SDK) ajoute un `tracestate` à la propriété du message, comprenant l’horodatage de la création du message.
1. L’appareil IoT envoie le message à IoT Hub.
1. Le message arrive à la passerelle du hub IoT.
1. IoT Hub recherche `tracestate` dans les propriétés du message, puis vérifie qu’il est au bon format.
1. Si c’est le cas, IoT Hub génère un `trace-id` global unique pour le message et un `span-id` pour le « tronçon », puis il les journalise dans des [journaux de traçage distribués IoT Hub](monitor-iot-hub-reference.md#distributed-tracing-preview) sous l’opération `DiagnosticIoTHubD2C`.
1. Quand le traitement des messages est terminé, IoT Hub génère un autre `span-id`, puis le journalise avec le `trace-id` existant, sous l’opération `DiagnosticIoTHubIngress`.
1. Si le routage est activé pour le message, IoT Hub écrit des données dans le point de terminaison personnalisé, puis journalise un autre `span-id` avec le même `trace-id` sous la catégorie `DiagnosticIoTHubEgress`.
1. Les étapes ci-dessus sont répétées pour chaque message généré.

## <a name="public-preview-limits-and-considerations"></a>Considérations et limites concernant la préversion publique

- La proposition d’une norme W3C pour le contexte de trace est encore au stade d’ébauche.
- Pour le moment, le seul langage de développement pris en charge par le SDK client est le langage C.
- La fonctionnalité de jumeau cloud-à-appareil n’est pas disponible pour le [niveau de base d’IoT Hub](iot-hub-scaling.md#basic-and-standard-tiers). Toutefois, IoT Hub continuera de journaliser dans Azure Monitor s’il voit un en-tête de contexte de trace correctement composé.
- Pour garantir un fonctionnement efficace, IoT Hub impose une limitation au niveau du taux de journalisation effectuée dans le cadre du traçage distribué.

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur le modèle général de traçage distribué dans les microservices, consultez [Microservice architecture pattern: distributed tracing](https://microservices.io/patterns/observability/distributed-tracing.html).
- Pour définir une configuration permettant d’appliquer des paramètres de traçage distribué à un grand nombre d’appareils, consultez [Configurer et surveiller des appareils IoT à grande échelle à l’aide du portail Azure](./iot-hub-automatic-device-management.md).
- Pour plus d’informations sur Azure Monitor, consultez [Qu’est-ce qu’Azure Monitor ?](../azure-monitor/overview.md).
- Pour en savoir plus sur d’Azure Monitor avec IoT Hub, consultez [Superviser avec IoT Hub](monitor-iot-hub.md)