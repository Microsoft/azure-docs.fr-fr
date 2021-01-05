---
title: Journaux de ressources pour Azure SignalR Service
description: Découvrez comment configurer les journaux de ressources pour Azure SignalR Service et utiliser ceux-ci pour résoudre les problèmes automatiquement.
author: wanlwanl
ms.service: signalr
ms.topic: conceptual
ms.date: 12/17/2019
ms.author: wanl
ms.openlocfilehash: 5ad40ca051677ced0c6d8b5c35e8563272ff598f
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183972"
---
# <a name="resource-logs-for-azure-signalr-service"></a>Journaux de ressources pour Azure SignalR Service

Ce didacticiel décrit les journaux des ressources pour Azure SignalR Service, ainsi que la façon de les configurer et de résoudre les problèmes qui y sont liés. 

## <a name="prerequisites"></a>Prérequis
Pour activer les journaux de ressources, vous avez besoin d’un emplacement pour stocker vos données de journal. Ce tutoriel utilise Stockage Azure et Log Analytics.

* [Stockage Azure](../azure-monitor/platform/resource-logs.md#send-to-azure-storage) – Conserve les journaux de ressources pour l’audit de stratégie, l’analyse statique ou la sauvegarde.
* [Log Analytics](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace) – Outil analytique et de recherche dans les journaux flexible qui permet d’analyser les journaux bruts générés par une ressource Azure.

## <a name="set-up-resource-logs-for-an-azure-signalr-service"></a>Configurer des journaux de ressources pour Azure SignalR Service

Vous pouvez afficher les journaux des ressources pour Azure SignalR Service. Ces journaux offrent une vue plus complète de la connectivité à votre instance Azure SignalR Service. Les journaux de ressources fournissent des informations détaillées sur chaque connexion. Par exemple, des informations de base (ID utilisateur, ID de connexion; type de transport, etc.) et des informations sur l’événement (connexion, déconnexion, abandon, etc.) de la connexion. Des journaux de ressources peuvent être utilisés pour l’identification des problèmes, ainsi que le suivi et l’analyse des connexions.

### <a name="enable-resource-logs"></a>Activer les journaux de ressources

Les journaux de ressources sont désactivés par défaut. Pour activer les journaux de ressources, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), sous **Analyse**, cliquez sur **Paramètres de diagnostic**.

    ![Navigation dans le volet vers les journaux de diagnostic](./media/signalr-tutorial-diagnostic-logs/diagnostic-settings-menu-item.png)

1. Cliquez ensuite sur **Ajouter un paramètre de diagnostic**.

    ![Ajouter des journaux de ressources](./media/signalr-tutorial-diagnostic-logs/add-diagnostic-setting.png)

1. Définissez la cible d’archivage de votre choix. Les options actuellement disponibles sont **Archiver dans un compte de stockage** et **Envoyer à Log Analytics**.

1. Sélectionnez les journaux à archiver.

    ![Volet Paramètres de diagnostic](./media/signalr-tutorial-diagnostic-logs/diagnostics-settings-pane.png)


1. Enregistrez les nouveaux paramètres de diagnostic.

Les nouveaux paramètres prennent effet au bout de 10 minutes environ. Après cela, les journaux d’activité apparaissent dans la cible d’archivage configurée, dans le volet **Journaux de diagnostic**.

Pour plus d’informations sur la configuration des diagnostics, consultez la [vue d’ensemble des journaux de ressources Azure](../azure-monitor/platform/platform-logs-overview.md).

### <a name="resource-logs-categories"></a>Catégories de journaux de ressources

Azure SignalR Service capture les journaux de ressources dans une catégorie :

* **Tous les journaux** : Suivez les connexions à Azure SignalR Service. Les journaux fournissent des informations sur la connexion, la déconnexion, l’authentification et la limitation. Pour plus d'informations, consultez la section suivante.

### <a name="archive-to-a-storage-account"></a>Archiver dans un compte de stockage

Les journaux sont stockés dans le compte de stockage configuré dans le volet **Journaux de diagnostic**. Un conteneur nommé `insights-logs-alllogs` est automatiquement créé pour stocker les journaux de ressources. Dans le conteneur, les journaux sont stockés dans le fichier `resourceId=/SUBSCRIPTIONS/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/RESOURCEGROUPS/XXXX/PROVIDERS/MICROSOFT.SIGNALRSERVICE/SIGNALR/XXX/y=YYYY/m=MM/d=DD/h=HH/m=00/PT1H.json`. Fondamentalement, le chemin d’accès est combiné par `resource ID` et `Date Time`. Les fichiers journaux sont fractionnés par `hour`. Par conséquent, les minutes sont toujours `m=00`.

Tous les journaux d’activité sont stockés au format JSON (JavaScript Object Notation). Chaque entrée comporte des champs de type chaîne au format décrit dans les sections suivantes.

Les chaînes JSON du journal d’archivage incluent les éléments répertoriés dans les tableaux suivants :

**Format**

Nom | Description
------- | -------
time | Heure de l’événement de journal
level | Niveau de l’événement de journal
resourceId | ID de ressource de votre Azure SignalR Service
location | Emplacement de votre Azure SignalR Service
catégorie | Catégorie de l’événement de journal
operationName | Nom d’opération de l’événement
callerIpAddress | Adresse IP de votre serveur/client
properties | Propriétés détaillées relatives à cet événement de journal. Pour plus d’informations, voir le tableau des propriétés ci-dessous

**Tableau des propriétés**

Nom | Description
------- | -------
type | Type de l’événement de journal. Actuellement, nous fournissons des informations sur la connectivité à Azure SignalR Service. Seul le type de `ConnectivityLogs` est disponible
collection | Collection de l’événement de journal. Les valeurs autorisées sont : `Connection`, `Authorization` et `Throttling`
connectionId | Identité de la connexion
transportType | Type de transport de la connexion. Les valeurs autorisées sont : `Websockets` \| `ServerSentEvents` \| `LongPolling`
connectionType | Type de la connexion. Les valeurs autorisées sont : `Server` \| `Client`. `Server` : connexion du côté serveur ; `Client` : connexion du côté client
userId | Identité de l’utilisateur
message | Message détaillé de l’événement de journal

Le code suivant est un exemple de chaîne JSON de journal d’archivage :

```json
{
    "properties": {
        "message": "Entered Serverless mode.",
        "type": "ConnectivityLogs",
        "collection": "Connection",
        "connectionId": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
        "userId": "User",
        "transportType": "WebSockets",
        "connectionType": "Client"
    },
    "operationName": "ServerlessModeEntered",
    "category": "AllLogs",
    "level": "Informational",
    "callerIpAddress": "xxx.xxx.xxx.xxx",
    "time": "2019-01-01T00:00:00Z",
    "resourceId": "/SUBSCRIPTIONS/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/RESOURCEGROUPS/XXXX/PROVIDERS/MICROSOFT.SIGNALRSERVICE/SIGNALR/XXX",
    "location": "xxxx"
}
```

### <a name="archive-logs-schema-for-log-analytics"></a>Schéma des journaux d’archivage pour Log Analytics

Pour afficher les journaux de ressources, procédez comme suit :

1. Cliquez sur `Logs` dans votre Log Analytics cible.

    ![Élément de menu Log Analytics](./media/signalr-tutorial-diagnostic-logs/log-analytics-menu-item.png)

2. Entrez `SignalRServiceDiagnosticLogs`, puis sélectionnez l’intervalle de temps pour interroger les journaux de ressources. Pour une requête avancée, voir [Bien démarrer avec Log Analytics dans Azure Monitor](../azure-monitor/log-query/log-analytics-tutorial.md)

    ![Journal des requêtes dans Log Analytics](./media/signalr-tutorial-diagnostic-logs/query-log-in-log-analytics.png)

Les colonnes du journal d’archivage incluent les éléments répertoriés dans le tableau suivant :

Nom | Description
------- | ------- 
TimeGenerated | Heure de l’événement de journal
Collection | Collection de l’événement de journal. Les valeurs autorisées sont : `Connection`, `Authorization` et `Throttling`
NomOpération | Nom d’opération de l’événement
Emplacement | Emplacement de votre Azure SignalR Service
Level | Niveau de l’événement de journal
callerIpAddress | Adresse IP de votre serveur/client
Message | Message détaillé de l’événement de journal
UserId | Identité de l’utilisateur
ConnectionId | Identité de la connexion
ConnectionType | Type de la connexion. Les valeurs autorisées sont : `Server` \| `Client`. `Server` : connexion du côté serveur ; `Client` : connexion du côté client
TransportType | Type de transport de la connexion. Les valeurs autorisées sont : `Websockets` \| `ServerSentEvents` \| `LongPolling`

### <a name="troubleshooting-with-resource-logs"></a>Résolution des problèmes à l’aide des journaux de ressources

Pour résoudre des problèmes liés à Azure SignalR Service, vous pouvez activer les journaux côté serveur/client pour capturer les défaillances. Actuellement, Azure SignalR Service expose les journaux de ressources. Vous pouvez également activer les journaux pour le côté service.

En cas de problème de croissance ou de perte inattendues de la connexion, vous pouvez tirer parti des journaux de ressources pour résoudre les problèmes.

Les problèmes courants sont souvent liés à des modifications inattendues de la quantité des connexions, où les connexions atteignent les limites de connexion et entraînent un échec d’autorisation. Consultez les sections suivantes sur la résolution des problèmes.

#### <a name="unexpected-connection-number-changes"></a>Modifications inattendues du nombre de connexions

##### <a name="unexpected-connection-dropping"></a>Abandon de connexion inattendu

Si vous rencontrez un abandon de connexion inattendu, commencez par activer les journaux des côtés service, serveur et client.

Si une connexion est perdue, les journaux de ressources enregistrent cet événement, et vous voyez `ConnectionAborted` ou `ConnectionEnded` dans `operationName`.

La différence entre `ConnectionAborted` et `ConnectionEnded` est que `ConnectionEnded` est une déconnexion attendue déclenchée par le côté client ou serveur. Tandis que `ConnectionAborted` est généralement un événement d’abandon de connexion inattendu dont le motif est fourni dans `message`.

Les motifs d’abandon sont répertoriés dans le tableau suivant :

Motif | Description
------- | ------- 
Le nombre de connexions atteint la limite | Le nombre de connexions atteint la limite de votre niveau tarifaire actuel. Envisagez une montée en puissance de l’unité de service
Le serveur d’applications a fermé la connexion | Le serveur d’applications déclenche l’abandon. Celui-ci peut être considéré comme un abandon attendu
Expiration de test ping de connexion | En général, cela est dû à un problème de réseau. Songez à vérifier la disponibilité de votre serveur d’applications à partir d’Internet
Rechargement du service, reconnexion | Azure SignalR Service est en cours de rechargement. Azure Signalr prend en charge la reconnexion automatique. Vous pouvez attendre la reconnexion ou vous reconnecter manuellement à Azure SignalR Service
Erreur temporaire de serveur interne | Une erreur temporaire se produit dans Azure SignalR Service, qui doit être récupérée automatiquement
Connexion au serveur abandonnée | La connexion au serveur est abandonnée en raison d’une erreur inconnue. Envisagez de résoudre automatiquement le problème avec le journal côté service/serveur/client. Essayez d’exclure les problèmes de base (par exemple, problème de réseau, problème côté serveur d’applications, etc.). Si le problème n’est pas résolu, contactez-nous pour obtenir une aide supplémentaire. Pour plus d’informations, voir la section [Obtenir de l’aide](#get-help). 

##### <a name="unexpected-connection-growing"></a>Croissance inattendue des connexions

Pour résoudre les problèmes de croissance inattendue des connexions, la première chose à faire est de filtrer les connexions supplémentaires. Vous pouvez ajouter un ID d’utilisateur de test unique à votre connexion de client de test. Ensuite, vérifiez les journaux de ressources. Si vous constatez que plusieurs connexions de client ont le même identifiant utilisateur de test ou la même adresse IP, il est probable que le côté client crée et établisse plus de connexions que prévu. Vérifiez votre côté client.

#### <a name="authorization-failure"></a>Échec de l’autorisation

Si l’erreur 401 Non autorisé est retournée pour des demandes de clients, vérifiez vos journaux de ressources. Si vous rencontrez l’erreur `Failed to validate audience. Expected Audiences: <valid audience>. Actual Audiences: <actual audience>`, cela signifie qu’aucune des audiences dans votre jeton d’accès n’est valide. Essayez d’utiliser les audiences valides suggérées dans le journal.


#### <a name="throttling"></a>Limitation

Si vous constatez que vous ne pouvez pas établir de connexions de client SignalR à Azure SignalR Service, vérifiez vos journaux de ressources. Si vous rencontrez `Connection count reaches limit` dans le journal de ressources, cela signifie que vous établissez un trop grand nombre de connexions à Azure SignalR Service, et que la limite du nombre de connexions est atteinte. Envisagez une montée en puissance de votre Azure SignalR Service. Si vous rencontrez `Message count reaches limit` dans le journal de ressources, cela signifie que vous utilisez le niveau gratuit et épuisez le quota de messages. Si vous souhaitez envoyer davantage de messages, envisagez de remplacer votre Azure SignalR Service par le niveau standard pour envoyer des messages supplémentaires. Pour plus d’informations, voir [Tarification d’Azure SignalR Service](https://azure.microsoft.com/pricing/details/signalr-service/).

### <a name="get-help"></a>Obtenir de l’aide

Nous vous recommandons de commencer par tenter de résoudre les problèmes par vous-même. La plupart des problèmes sont dus à des problèmes de serveur d’applications ou de réseau. Suivez le [guide de résolution des problèmes avec le journal de ressources](#troubleshooting-with-resource-logs) et le [guide de résolution des problèmes de base](https://github.com/Azure/azure-signalr/blob/dev/docs/tsg.md) pour trouver la cause racine.
Si vous ne parvenez toujours pas à résoudre le problème, envisagez d’ouvrir un problème dans GitHub ou de créer un ticket dans le portail Azure.
Procédez de la façon suivante :
1. Intervalle de temps d’environ 30 minutes lorsque le problème se produit
2. ID de ressource d’Azure SignalR Service
3. Détails du problème, aussi précis que possible : Par exemple, AppServer n’envoie pas de messages, la connexion du client est abandonnée, etc.
4. Journaux collectés du côté serveur/client et autres éléments pouvant être utiles
5. [Facultatif] Code de reproduction

> [!NOTE]
> Si vous ouvrez un problème dans GitHub, gardez vos informations sensibles (par exemple, ID de ressource, journaux de serveur/client) confidentielles. Ne les envoyez à des membres de l’organisation Microsoft qu’en privé.