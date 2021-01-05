---
title: Présentation des méthodes directes Azure IoT Hub | Microsoft Docs
description: Guide de développeur - Utiliser des méthodes directes pour appeler du code sur vos appareils à partir d’une application de service.
author: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 07/17/2018
ms.author: philmea
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: b75e859fc1237bc88bee464cef423b7289810fa8
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92147789"
---
# <a name="understand-and-invoke-direct-methods-from-iot-hub"></a>Comprendre et appeler des méthodes directes à partir d’IoT Hub

IoT Hub vous donne la possibilité d’appeler des méthodes directes sur des appareils à partir du cloud. Les méthodes directes représentent une interaction de demande-réponse avec un appareil, similaire à un appel HTTP, dans la mesure où elles réussissent ou échouent immédiatement (après un délai d’attente spécifié par l’utilisateur). Cette approche est utile pour les scénarios où le plan d’action immédiate est différent selon que l’appareil a été en mesure ou non de répondre.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-whole.md)]

Chaque méthode d’appareil cible à un seul appareil. L’article [Planifier des travaux sur plusieurs appareils](iot-hub-devguide-jobs.md) offre un moyen d’appeler des méthodes directes sur plusieurs appareils et de planifier un appel de méthode pour des appareils déconnectés.

Toute personne disposant d’autorisations **Connexion de service** sur IoT Hub peut appeler une méthode sur un appareil.

Les méthodes directes suivent un modèle de requête-réponse et sont destinées aux communications qui nécessitent une confirmation immédiate de leur résultat. Par exemple, le contrôle interactif de l’appareil, tel que l’activation d’un ventilateur.

Reportez-vous à [l’aide sur la communication cloud-à-appareil](iot-hub-devguide-c2d-guidance.md) en cas de doute entre l’utilisation des propriétés de votre choix, des méthodes directes ou des messages cloud-à-appareil.

## <a name="method-lifecycle"></a>Cycle de vie de méthode

Les méthodes directes sont implémentées sur l’appareil et peuvent nécessiter de zéro à plusieurs entrées dans la charge utile pour s’instancier correctement. Vous appelez une méthode directe via un URI côté service (`{iot hub}/twins/{device id}/methods/`). Un appareil reçoit des méthodes directes via une rubrique MQTT spécifique de l’appareil (`$iothub/methods/POST/{method name}/`) ou via des liens AMQP (les propriétés d’application `IoThub-methodname` et `IoThub-status`).

> [!NOTE]
> Lorsque vous appelez une méthode directe sur un appareil, les noms et les valeurs de propriété peuvent contenir uniquement des caractères alphanumériques US-ASCII imprimables, à l’exception des caractères suivants : ``{'$', '(', ')', '<', '>', '@', ',', ';', ':', '\', '"', '/', '[', ']', '?', '=', '{', '}', SP, HT}``.
> 

Les méthodes directes sont synchrones et réussissent ou échouent à l’issue du délai d’expiration (par défaut : 30 secondes, définissable entre 5 et 300 secondes). Les méthodes directes sont utiles dans des scénarios interactifs où vous souhaitez qu’un appareil agisse si et seulement s’il est en ligne et reçoit des commandes. Par exemple, allumer une lumière à partir d’un téléphone. Dans ces scénarios, vous souhaitez constater immédiatement la réussite ou l’échec de la commande, de façon à ce que le service cloud puisse agir sur le résultat dès que possible. L’appareil peut renvoyer un corps de message résultant de la méthode, mais il n’est pas obligatoire que la méthode procède de la sorte. Il existe ni garantie de classement, ni sémantique de concurrence sur les appels de méthode.

Les méthodes directes sont exclusivement HTTPS côté cloud, et MQTT, AMQP, MQTT sur WebSockets ou AMQP sur WebSockets côté appareil.

La charge utile des requêtes et des réponses de méthodes correspond à un document JSON d’une taille pouvant aller jusqu’à 128 Ko.

## <a name="invoke-a-direct-method-from-a-back-end-app"></a>Appeler une méthode directe à partir d’une application principale

Maintenant, appelez une méthode directe à partir d’une application back-end.

### <a name="method-invocation"></a>Appel de méthode

Les appels de méthode directe sur un appareil sont des appels HTTPS qui comprennent les éléments suivants :

* l’*URI de demande* spécifique à l’appareil et la [version d’API](/rest/api/iothub/service/devices/invokemethod) :

    ```http
    https://fully-qualified-iothubname.azure-devices.net/twins/{deviceId}/methods?api-version=2018-06-30
    ```

* la *méthode* POST ;

* des *en-têtes* contenant l’autorisation, l’ID de demande, le type de contenu et l’encodage du contenu ;

* un *corps* JSON transparent au format suivant :

    ```json
    {
        "methodName": "reboot",
        "responseTimeoutInSeconds": 200,
        "payload": {
            "input1": "someInput",
            "input2": "anotherInput"
        }
    }
    ```

La valeur fournie en tant que `responseTimeoutInSeconds` dans la requête correspond à la durée pendant laquelle le service IoT Hub doit attendre la fin de l’exécution d’une méthode directe sur un appareil. Définissez ce délai d’expiration de sorte qu’il soit au moins aussi long que la durée d’exécution prévue d’une méthode directe par un appareil. Si le délai d’expiration n’est pas spécifié, la valeur par défaut de 30 secondes est utilisée. Les valeurs minimales et maximales pour `responseTimeoutInSeconds` sont respectivement 5 et 300 secondes.

La valeur fournie en tant que `connectTimeoutInSeconds` dans la requête correspond à la durée pendant laquelle le service IoT Hub doit attendre pour qu’un appareil déconnecté soit mis en ligne après l’appel d’une méthode directe. La valeur par défaut est 0, ce qui signifie que les appareils doivent déjà être en ligne lors de l’appel d’une méthode directe. La valeur maximale pour `connectTimeoutInSeconds` est 300 secondes.

#### <a name="example"></a>Exemple

Cet exemple vous permettra de lancer en toute sécurité une demande d’appel d’une méthode directe sur un appareil IoT inscrit auprès d’un IoT Hub Azure.

Pour commencer, utilisez l’[extension Microsoft Azure IoT pour Azure CLI](https://github.com/Azure/azure-iot-cli-extension) afin de créer une SharedAccessSignature.

```bash
az iot hub generate-sas-token -n <iothubName> -du <duration>
```

Ensuite, remplacez l’en-tête Authorization par la SharedAccessSignature que vous venez de générer, puis modifiez les paramètres `iothubName`, `deviceId`, `methodName` et `payload` pour qu’ils correspondent à votre implémentation dans l’exemple de commande `curl` ci-dessous.  

```bash
curl -X POST \
  https://<iothubName>.azure-devices.net/twins/<deviceId>/methods?api-version=2018-06-30 \
  -H 'Authorization: SharedAccessSignature sr=iothubname.azure-devices.net&sig=x&se=x&skn=iothubowner' \
  -H 'Content-Type: application/json' \
  -d '{
    "methodName": "reboot",
    "responseTimeoutInSeconds": 200,
    "payload": {
        "input1": "someInput",
        "input2": "anotherInput"
    }
}'
```

Exécutez la commande modifiée pour appeler la méthode directe spécifiée. Les demandes réussies retournent un code d’état HTTP 200.

> [!NOTE]
> L’exemple ci-dessus illustre l’appel d’une méthode directe sur un appareil.  Si vous souhaitez appeler une méthode directe dans un module IoT Edge, vous devez modifier la demande d’URL comme indiqué ci-dessous :

```bash
https://<iothubName>.azure-devices.net/twins/<deviceId>/modules/<moduleName>/methods?api-version=2018-06-30
```
### <a name="response"></a>response

L’application back-end reçoit une réponse constituée des éléments suivants :

* *Code d’état HTTP* :
  * 200 indique que l’exécution de la méthode directe est réussie ;
  * 404 indique que l’ID de périphérique n’est pas valide ou que l’appareil n’était pas en ligne lors de l’appel d’une méthode directe et pendant la durée `connectTimeoutInSeconds` par la suite (utilisez le message d’erreur accompagné pour comprendre la cause racine) ;
  * 504 indique l’expiration du délai de la passerelle provoquée par l’appareil qui ne répond pas à un appel de méthode directe en un temps inférieur ou égal à `responseTimeoutInSeconds`.

* des *en-têtes* contenant l’ETag, l’ID de demande, le type de contenu et l’encodage du contenu ;

* un *corps* JSON au format suivant :

    ```json
    {
        "status" : 201,
        "payload" : {...}
    }
    ```

    Les propriétés `status` et `body` sont fournies par l’appareil et permettent de répondre avec le code d’état et/ou la description spécifiques de l’appareil.

### <a name="method-invocation-for-iot-edge-modules"></a>Appel de méthode pour les modules IoT Edge

L’appel de méthodes directes avec un ID de module est pris en charge dans le [SDK client C# du service IoT](https://www.nuget.org/packages/Microsoft.Azure.Devices/).

Pour ce faire, utilisez la méthode `ServiceClient.InvokeDeviceMethodAsync()`, et passez `deviceId` et `moduleId` comme paramètres.

## <a name="handle-a-direct-method-on-a-device"></a>Gérer une méthode directe sur un appareil

Voyons comment gérer une méthode directe sur un appareil IoT.

### <a name="mqtt"></a>MQTT

La section suivante s’applique au protocole MQTT.

#### <a name="method-invocation"></a>Appel de méthode

Les appareils reçoivent des requêtes de méthode directe sur la rubrique MQTT : `$iothub/methods/POST/{method name}/?$rid={request id}`. Le nombre d’abonnements par appareil est limité à 5. Il est donc recommandé de ne pas s’abonner individuellement à chaque méthode directe. Il est préférable de vous abonner à `$iothub/methods/POST/#`, puis de filtrer les messages remis en fonction du nom des méthodes souhaitées.

Le corps que l’appareil reçoit est au format suivant :

```json
{
    "input1": "someInput",
    "input2": "anotherInput"
}
```

Les demandes de méthode sont QoS 0.

#### <a name="response"></a>response

L’appareil envoie des réponses à `$iothub/methods/res/{status}/?$rid={request id}`, où :

* La propriété `status` est l’état d’exécution de la méthode fourni par l’appareil.

* La propriété `$rid`est l’ID de demande de l’appel de méthode reçu d’IoT Hub.

Le corps est défini par l’appareil et peut être n’importe quel état.

### <a name="amqp"></a>AMQP

La section suivante s’applique au protocole AMQP.

#### <a name="method-invocation"></a>Appel de méthode

L’appareil reçoit des requêtes de méthode directe en créant un lien de réception à l’adresse `amqps://{hostname}:5671/devices/{deviceId}/methods/deviceBound`.

Le message AMQP est transmis au lien de réception qui représente la demande de méthode. Il contient les sections suivantes :

* La propriété d’ID de corrélation, qui contient un ID de requête qui doit être transmis avec la réponse de la méthode correspondante.

* Une propriété d’application nommée `IoThub-methodname`, qui contient le nom de la méthode appelée.

* Le corps du message AMQP, qui contient la charge utile de la méthode en tant que JSON.

#### <a name="response"></a>response

L’appareil crée un lien d’envoi pour retourner la réponse de la méthode à l’adresse `amqps://{hostname}:5671/devices/{deviceId}/methods/deviceBound`.

La réponse de la méthode est retournée via le lien d’envoi et est structurée comme suit :

* La propriété d’ID de corrélation, qui contient l’ID de requête transmis dans le message de requête de la méthode.

* Une propriété d’application nommée `IoThub-status`, qui contient l’état de la méthode fournie par l’utilisateur.

* Le corps du message AMQP, qui contient la réponse de la méthode en tant que JSON.

## <a name="additional-reference-material"></a>Matériel de référence supplémentaire

Les autres rubriques de référence dans le Guide du développeur IoT Hub comprennent :

* La rubrique [Points de terminaison IoT Hub](iot-hub-devguide-endpoints.md), qui décrit les différents points de terminaison que chaque hub IoT expose pour les opérations d’exécution et de gestion.

* La rubrique [Limitation et quotas](iot-hub-devguide-quotas-throttling.md), qui décrit les quotas appliqués et le comportement de limitation auquel s’attendre quand vous utilisez IoT Hub.

* La rubrique [SDK des services et appareils Azure IoT](iot-hub-devguide-sdks.md), qui répertorie les SDK en différents langages que vous pouvez utiliser pour le développement d’applications d’appareil et de service qui interagissent avec IoT Hub.

* La rubrique [Langage de requête d’IoT Hub pour les jumeaux d’appareil, les travaux et le routage des messages](iot-hub-devguide-query-language.md), qui décrit le langage de requête d’IoT Hub permettant de récupérer, à partir d’IoT Hub, des informations relatives à vos jumeaux d’appareil et à vos travaux.

* La rubrique [Prise en charge de MQTT au niveau d’IoT Hub](iot-hub-mqtt-support.md), qui fournit des informations supplémentaires sur la prise en charge du protocole MQTT par IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous savez comment utiliser les méthodes directes, vous serez peut-être intéressé par l’article suivant du Guide du développeur IoT Hub :

* [Planifier des travaux sur plusieurs appareils](iot-hub-devguide-jobs.md)

Si vous souhaitez tenter de mettre en pratique certains des concepts décrits dans cet article, vous serez peut-être intéressé par le didacticiel IoT Hub suivant :

* [Utiliser des méthodes directes](quickstart-control-device-node.md)
* [Gestion des appareils avec Azure IoT Tools pour VS Code](iot-hub-device-management-iot-toolkit.md)