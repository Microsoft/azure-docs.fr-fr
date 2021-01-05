---
title: Diagnostics du proxy inverse Azure Service Fabric
description: Découvrez comment superviser et diagnostiquer le traitement des demandes au niveau du proxy inverse pour une application Azure Service Fabric.
author: kavyako
ms.topic: conceptual
ms.date: 08/08/2017
ms.author: kavyako
ms.openlocfilehash: bbc1fe5a76ecb5720bc49e0a082d5e9151b403d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75645461"
---
# <a name="monitor-and-diagnose-request-processing-at-the-reverse-proxy"></a>Surveiller et diagnostiquer le traitement de requêtes au niveau du proxy inverse

Depuis la mise en production 5.7 de Service Fabric, il est possible de collecter des événements liés au proxy inverse. Les événements sont disponibles dans deux canaux, l’un récupérant uniquement les événements d’erreur liés à l’échec du traitement de requêtes au niveau du proxy inverse, l’autre contenant les événements détaillés avec des entrées pour les requêtes qui ont réussi et celles qui ont échoué.

Reportez-vous à [Collecter les événements de proxy inverse](service-fabric-diagnostics-event-aggregation-wad.md#log-collection-configurations) pour activer la collecte d’événements à partir de ces canaux dans les clusters locaux et les clusters Azure Service Fabric.

## <a name="troubleshoot-using-diagnostics-logs"></a>Résoudre les problèmes à l’aide de journaux de diagnostic
Voici quelques exemples sur la façon d’interpréter les journaux d’activité d’échecs courants que vous pouvez rencontrer :

1. Le proxy inverse retourne un code d’état de réponse 504 (délai d’expiration).

    La raison vient peut-être du fait que le service n’est pas parvenu pas à répondre dans le délai d’attente imparti à la requête.
   Le premier événement ci-dessous consigne dans un journal les détails de la requête reçue par le proxy inverse. 
   Le second événement indique que la requête a échoué lors de la transmission au service, en raison d’une « erreur interne = ERROR_WINHTTP_TIMEOUT » 

    La charge utile comprend :

   * **traceId** : ce GUID peut servir à mettre en corrélation tous les événements correspondant à une requête unique. Dans les deux événements ci-dessous, traceId = **2f87b722-e254-4ac2-a802-fd315c1a0271**, ce qui implique qu’ils appartiennent à la même requête.
   * **requestUrl** : l’URL (URL du proxy inverse) à laquelle la requête a été envoyée.
   * **verb** : verbe HTTP.
   * **remoteAddress** : adresse du client qui envoie la requête.
   * **resolvedServiceUrl** : URL du point de terminaison de service sur laquelle la requête entrante a été résolue. 
   * **errorDetails** : informations complémentaires sur l’échec.

     ```
     {
     "Timestamp": "2017-07-20T15:57:59.9871163-07:00",
     "ProviderName": "Microsoft-ServiceFabric",
     "Id": 51477,
     "Message": "2f87b722-e254-4ac2-a802-fd315c1a0271 Request url = https://localhost:19081/LocationApp/LocationFEService?zipcode=98052, verb = GET, remote (client) address = ::1, resolved service url = Https://localhost:8491/LocationApp/?zipcode=98052, request processing start time =     15:58:00.074114 (745,608.196 MSec) ",
     "ProcessId": 57696,
     "Level": "Informational",
     "Keywords": "0x1000000000000021",
     "EventName": "ReverseProxy",
     "ActivityID": null,
     "RelatedActivityID": null,
     "Payload": {
      "traceId": "2f87b722-e254-4ac2-a802-fd315c1a0271",
      "requestUrl": "https://localhost:19081/LocationApp/LocationFEService?zipcode=98052",
      "verb": "GET",
      "remoteAddress": "::1",
      "resolvedServiceUrl": "Https://localhost:8491/LocationApp/?zipcode=98052",
      "requestStartTime": "2017-07-20T15:58:00.0741142-07:00"
     }
     }

     {
     "Timestamp": "2017-07-20T16:00:01.3173605-07:00",
     ...
     "Message": "2f87b722-e254-4ac2-a802-fd315c1a0271 Error while forwarding request to service: response status code = 504, description = Reverse proxy Timeout, phase = FinishSendRequest, internal error = ERROR_WINHTTP_TIMEOUT ",
     ...
     "Payload": {
      "traceId": "2f87b722-e254-4ac2-a802-fd315c1a0271",
      "statusCode": 504,
      "description": "Reverse Proxy Timeout",
      "sendRequestPhase": "FinishSendRequest",
      "errorDetails": "internal error = ERROR_WINHTTP_TIMEOUT"
     }
     }
     ```

2. Le proxy inverse retourne un code d’état de réponse 404 (Introuvable). 
    
    Voici un exemple d’événement où le proxy inverse retourne le code 404, car il n’est pas parvenu à trouver le point de terminaison de service correspondant.
    Les entrées de charge utile intéressantes ici sont :
   * **processRequestPhase** : indique la phase au cours du traitement de requêtes pendant laquelle la défaillance s’est produite, ***TryGetEndpoint***, c’est-à-dire pendant la tentative d’extraction du point de terminaison de service pour transférer. 
   * **errorDetails** : liste les critères de recherche du point de terminaison. Ici, vous pouvez voir que listenerName a spécifié = **FrontEndListener** alors que la liste de points de terminaison de réplica ne contient qu’un écouteur portant le nom **OldListener**.
    
     ```
     {
     ...
     "Message": "c1cca3b7-f85d-4fef-a162-88af23604343 Error while processing request, cannot forward to service: request url = https://localhost:19081/LocationApp/LocationFEService?ListenerName=FrontEndListener&zipcode=98052, verb = GET, remote (client) address = ::1, request processing start time = 16:43:02.686271 (3,448,220.353 MSec), error = FABRIC_E_ENDPOINT_NOT_FOUND, message = , phase = TryGetEndoint, SecureOnlyMode = false, gateway protocol = https, listenerName = FrontEndListener, replica endpoint = {\"Endpoints\":{\"\":\"Https:\/\/localhost:8491\/LocationApp\/\"}} ",
     "ProcessId": 57696,
     "Level": "Warning",
     "EventName": "ReverseProxy",
     "Payload": {
      "traceId": "c1cca3b7-f85d-4fef-a162-88af23604343",
      "requestUrl": "https://localhost:19081/LocationApp/LocationFEService?ListenerName=NewListener&zipcode=98052",
      ...
      "processRequestPhase": "TryGetEndoint",
      "errorDetails": "SecureOnlyMode = false, gateway protocol = https, listenerName = FrontEndListener, replica endpoint = {\"Endpoints\":{\"OldListener\":\"Https:\/\/localhost:8491\/LocationApp\/\"}}"
     }
     }
     ```
     Un autre exemple dans lequel le proxy inverse peut retourner l’erreur 404 Introuvable est le suivant : le paramètre de configuration ApplicationGateway\Http **SecureOnlyMode** est défini sur true avec le proxy inverse écoutant sur **HTTPS** ; toutefois, aucun point de terminaison de réplica n’est sécurisé (à l’écoute sur HTTP).
     Le proxy inverse retourne l’erreur 404, car il ne peut pas trouver de point de terminaison à l’écoute sur HTTPS pour transmettre la requête. L’analyse des paramètres dans la charge utile de l’événement permet d’identifier le problème :
    
     ```
      "errorDetails": "SecureOnlyMode = true, gateway protocol = https, listenerName = NewListener, replica endpoint = {\"Endpoints\":{\"OldListener\":\"Http:\/\/localhost:8491\/LocationApp\/\", \"NewListener\":\"Http:\/\/localhost:8492\/LocationApp\/\"}}"
     ```

3. La requête sur le proxy inverse échoue avec une erreur de délai d’expiration. 
    Les journaux des événements contiennent un événement avec les détails de la requête reçue (non illustré ici).
    L’événement suivant montre que le service a répondu avec un code d’état 404 et que le proxy inverse lance une nouvelle résolution. 

    ```
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Request to service returned: status code = 404, status description = , Reresolving ",
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "statusCode": 404,
        "statusDescription": ""
      }
    }
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Re-resolved service url = Https://localhost:8491/LocationApp/?zipcode=98052 ",
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "requestUrl": "Https://localhost:8491/LocationApp/?zipcode=98052"
      }
    }
    ```
    Lors de la collecte de tous les événements, vous constatez une suite d’événements indiquant chaque résolution et tentative de transfert.
    Le dernier événement de la série montre l’échec du traitement de la requête en raison d’un délai d’expiration, ainsi que le nombre de tentatives de résolution réussies.
    
    > [!NOTE]
    > Il est recommandé de conserver désactivée par défaut la collecte d’événements détaillés de canal, et de l’activer pour la résolution des problèmes si nécessaire.

    ```
    {
      ...
      "Message": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e Error while processing request: number of successful resolve attempts = 12, error = FABRIC_E_TIMEOUT, message = , phase = ResolveServicePartition,  ",
      "EventName": "ReverseProxy",
      ...
      "Payload": {
        "traceId": "7ac6212c-c8c4-4c98-9cf7-c187a94f141e",
        "resolveCount": 12,
        "errorval": -2147017729,
        "errorMessage": "",
        "processRequestPhase": "ResolveServicePartition",
        "errorDetails": ""
      }
    }
    ```
    
    Si la collecte est activée uniquement pour les événements critiques ou d’erreur, vous voyez un événement contenant des détails sur le délai d’expiration et le nombre de tentatives de résolution. 
    
    Les services susceptibles d’envoyer un code d’état 404 à l’utilisateur doivent ajouter un en-tête « X-ServiceFabric » à la réponse. Une fois l’en-tête ajouté à la réponse, le proxy inverse transfère le code d’état au client d’origine.  

4. Cas lorsque le client est déconnecté de la demande.

    L’événement suivant est enregistré quand le proxy inverse transfère la réponse au client, mais que le client se déconnecte :

    ```
    {
      ...
      "Message": "6e2571a3-14a8-4fc7-93bb-c202c23b50b8 Unable to send response to client: phase = SendResponseHeaders, error = -805306367, internal error = ERROR_SUCCESS ",
      "ProcessId": 57696,
      "Level": "Warning",
      ...
      "EventName": "ReverseProxy",
      "Payload": {
        "traceId": "6e2571a3-14a8-4fc7-93bb-c202c23b50b8",
        "sendResponsePhase": "SendResponseHeaders",
        "errorval": -805306367,
        "winHttpError": "ERROR_SUCCESS"
      }
    }
    ```
5. Le proxy inverse retourne l’erreur 404 FABRIC_E_SERVICE_DOES_NOT_EXIST.

    L’erreur FABRIC_E_SERVICE_DOES_NOT_EXIST est retournée si le schéma d’URI n’est pas spécifié pour le point de terminaison de service dans le manifeste de service.

    ```
    <Endpoint Name="ServiceEndpointHttp" Port="80" Protocol="http" Type="Input"/>
    ```

    Pour résoudre le problème, spécifiez le schéma d’URI dans le manifeste.
    ```
    <Endpoint Name="ServiceEndpointHttp" UriScheme="http" Port="80" Protocol="http" Type="Input"/>
    ```

> [!NOTE]
> Les événements liés au traitement de la requête websocket ne sont pas consignés pour le moment. Ils seront ajoutés dans la prochaine mise en production.

## <a name="next-steps"></a>Étapes suivantes
* [Agrégation et collecte d’événements à l’aide de Diagnostics Azure pour Windows](service-fabric-diagnostics-event-aggregation-wad.md) pour activer la collecte des journaux dans les clusters Azure.
* Pour afficher les événements Service Fabric dans Visual Studio, consultez [Surveiller et diagnostiquer localement](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md).
* Reportez-vous à [Configure reverse proxy to connect to secure services](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/Reverse-Proxy-Sample#configure-reverse-proxy-to-connect-to-secure-services) (Configurer le proxy inverse pour se connecter à des services sécurisés) pour obtenir des exemples de modèles Azure Resource Manager illustrant la configuration du proxy inverse sécurisé avec les différentes options de validation de certificat de service.
* Consultez [Proxy inverse Service Fabric](service-fabric-reverseproxy.md) pour en savoir plus.
