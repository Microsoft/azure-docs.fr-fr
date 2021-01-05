---
title: Journaux de diagnostic pour les connexions hybrides
description: Cet article fournit une vue d’ensemble des journaux d’activités et de diagnostics disponibles pour Azure Relay.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 980f2f7a737d3f2460c17a84c472cbf56f5eb90f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87533000"
---
# <a name="enable-diagnostics-logs-for-azure-relay-hybrid-connections"></a>Activer les journaux de diagnostic pour les connexions hybrides Azure Relay
Lorsque vous commencez à utiliser vos connexions hybrides Azure Relay, vous pouvez surveiller le mode et le moment d’ouverture et de fermeture de vos écouteurs et expéditeurs, ainsi que la façon dont vos connexions hybrides sont créées et les messages envoyés. Cet article fournit une vue d’ensemble des journaux d’activités et de diagnostics fournis par le service Azure Relay. 

Vous pouvez afficher deux types de journaux pour Azure Relay :

- [Journaux d’activité](../azure-monitor/platform/platform-logs-overview.md) : Ces journaux contiennent des informations sur les opérations effectuées sur votre espace de noms via le portail Azure ou un modèle Azure Resource Manager. Ces journaux d’activité sont toujours activés. Par exemple : « Créer ou mettre à jour un espace de noms », « Créer ou mettre à jour une connexion hybride ». 
- [Journaux de diagnostic](../azure-monitor/platform/platform-logs-overview.md) : Vous pouvez configurer des journaux de diagnostic pour obtenir une vue plus détaillée de l’ensemble des opérations et actions effectuées sur votre espace de noms à l’aide de l’API ou du Kit de développement logiciel (SDK) pour le langage concerné.

## <a name="view-activity-logs"></a>Afficher les journaux d’activité
Pour afficher les journaux d’activité de votre espace de noms Azure Relay, basculez vers la page **Journal d’activité** dans le portail Azure.

![Journal d’activité Azure Relay](./media/diagnostic-logs/activity-log.png)

## <a name="enable-diagnostic-logs"></a>Activer les journaux de diagnostic

> [!NOTE]
> Les journaux de diagnostic sont disponibles uniquement pour des connexions hybrides, pas pour un relais Windows Communication Foundation (WCF).

Pour activer les journaux de diagnostics, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre espace de noms Azure Relay, puis, sous **Surveillance**, sélectionnez **Paramètres de diagnostic**.
1. Dans la page **Paramètres de diagnostic**, sélectionnez **Ajouter un paramètre de diagnostic**.  

   ![Le lien « Ajouter le paramètre de diagnostic »](./media/diagnostic-logs/add-diagnostic-setting.png)

1. Configurez les paramètres de diagnostic en procédant comme suit :
    1. Dans la zone **Nom**, entrez un nom pour les paramètres de diagnostics.  
    2. Pour le type de journal, sélectionnez **HybridConnectionsEvent**. 
    3. Pour vos journaux de diagnostics, sélectionnez l’une des trois **destinations** suivantes :  
        1. Si vous sélectionnez **Archiver dans un compte de stockage**, configurez le compte de stockage dans lequel les journaux de diagnostics seront stockés.  
        2. Si vous sélectionnez **Diffuser vers un hub d’événements**, configurez le hub d’événements vers lequel vous souhaitez diffuser les journaux de diagnostics.
        3. Si vous sélectionnez **Envoyer à Log Analytics**, spécifiez l’instance de Log Analytics à laquelle envoyer les diagnostics.  

        ![Exemple de paramètres de diagnostic](./media/diagnostic-logs/sample-diagnostic-settings.png)
1. Sélectionnez **Enregistrer** dans la barre d’outils pour enregistrer les paramètres.

Les nouveaux paramètres prennent effet au bout de 10 minutes environ. Les journaux d’activité s’affichent dans la cible d’archivage configurée, dans le volet **Journaux de diagnostics**. Pour plus d’informations sur la configuration des paramètres de diagnostics, consultez [Vue d’ensemble des journaux de diagnostics Azure](../azure-monitor/platform/platform-logs-overview.md).


## <a name="schema-for-hybrid-connections-events"></a>Schéma pour les événements de connexions hybrides
Les chaînes JSON du journal des événements de connexions hybrides incluent les éléments répertoriés dans le tableau suivant :

| Nom | Description |
| ------- | ------- |
| ResourceId | ID de ressource Azure Resource Manager |
| ActivityId | ID interne, utilisé pour identifier l’opération spécifiée. Peut également être nommé « TrackingId ». |
| Point de terminaison | Adresse de la ressource Relay. |
| NomOpération | Type de l’opération de connexion hybride enregistrée. |
| EventTimeString | Horodatage UTC de l’enregistrement de journal. |
| Message | Message détaillé de l’événement. |
| Category | Catégorie de l’événement. Actuellement, la seule catégorie disponible est `HybridConnectionsEvents`. 


## <a name="sample-hybrid-connections-event"></a>Exemple d’événement de connexion hybride
Voici un exemple d’événement de connexion hybride au format JSON. 

```json
{
    "resourceId": "/SUBSCRIPTIONS/0000000000-0000-0000-0000-0000000000000/RESOURCEGROUPS/MyResourceGroup/PROVIDERS/MICROSOFT.RELAY/NAMESPACES/MyRelayNamespace",
    "ActivityId": "7006a0db-27eb-445c-939b-ce86133014cc",
    "endpoint": "sb://myrelaynamespace.servicebus.windows.net/myhybridconnection/7006a0db-27eb-445c-939b-ce86133014cc_G5",
    "operationName": "Microsoft.Relay/HybridConnections/NewSenderRegistering",
    "EventTimeString": "2020-04-27T20:27:57.3842810Z",
    "message": "A new sender is registering.",
    "category": "HybridConnectionsEvent"
}
```

## <a name="events-and-operations-captured-in-diagnostic-logs"></a>Événements et opérations capturés dans les journaux de diagnostic

| Opération | Description | 
| --------- | ----------- | 
| AuthorizationFailed | Échec de l’autorisation.|
| InvalidSasToken | Jeton SAP non valide. | 
| ListenerAcceptingConnection | L’écouteur accepte la connexion. |
| ListenerAcceptingConnectionTimeout | L’écouteur qui accepte la connexion a dépassé le délai d’attente. |
| ListenerAcceptingHttpRequestFailed | L’écouteur qui accepte la requête HTTP a échoué en raison d’une exception. |
| ListenerAcceptingRequestTimeout | L’écouteur qui accepte la requête a dépassé le délai d’attente. |  
| ListenerClosingFromExpiredToken | L’écouteur se ferme parce que le jeton de sécurité a expiré. | 
| ListenerRejectedConnection | L’écouteur a rejeté la connexion. |
| ListenerReturningHttpResponse | L’écouteur retourne une réponse HTTP. |  
| ListenerReturningHttpResponseFailed | L’écouteur retourne une réponse HTTP avec un code d’échec. | 
 ListenerSentHttpResponse | Le service de relais a reçu une réponse HTTP de l’écouteur. | 
| ListenerUnregistered | L’écouteur n’est pas inscrit. | 
| ListenerUnresponsive | L’écouteur ne réagit pas lors du retour d’une réponse. | 
| MessageSendingToListener | Un message est en cours d’envoi à l’écouteur. |
| MessageSentToListener | Un message a été envoyé à l’écouteur. | 
| NewListenerRegistered | Nouvel écouteur inscrit. |
| NewSenderRegistering | Nouvel expéditeur en cours d’inscription. | 
| ProcessingRequestFailed | Échec du traitement d’une opération de connexion hybride. | 
| SenderConnectionClosed | La connexion de l’expéditeur est fermée. |
| SenderListenerConnectionEstablished | L’expéditeur et l’écouteur ont correctement établi la connexion. |
| SenderSentHttpRequest | L’expéditeur a envoyé une requête HTTP. | 


## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure Relay, consultez :

* [Présentation d’Azure Relay](relay-what-is-it.md)
* [Prise en main des connexions hybrides Relay](relay-hybrid-connections-dotnet-get-started.md)
