---
title: Journaux de diagnostics Azure Service Bus | Microsoft Docs
description: Cet article fournit une vue d’ensemble de tous les journaux de diagnostics et d’opérations disponibles pour Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: f227f5a988ccd51425b4f43e87b4ed4d9af74e1f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88064432"
---
# <a name="enable-diagnostics-logs-for-service-bus"></a>Activer les journaux de diagnostics pour Service Bus

Quand vous commencez à utiliser votre espace de noms Azure Service Bus, vous souhaiterez peut-être superviser le moment et le déroulement de la création, de la suppression ou de l’accès à votre espace de noms. Cet article fournit une vue d’ensemble de tous les journaux de diagnostics et d’opérations disponibles.

Azure Service Bus prend actuellement en charge les journaux d’activité et d’opérations qui capturent les *opérations de gestion* effectuées sur l’espace de noms Azure Service Bus. Plus précisément, ces journaux d’activité capturent le type d’opération, notamment la création de la file d’attente, les ressources utilisées et l’état de l’opération.

## <a name="operational-logs-schema"></a>Schéma des journaux d’activité des opérations

Tous les journaux sont stockés au format JSON (JavaScript Object Notation) dans les deux emplacements suivants :

- **AzureActivity** : Affiche les journaux des opérations et actions qui sont effectuées en relation avec votre espace de noms dans le Portail Azure ou via des déploiements de modèles Azure Resource Manager.
- **AzureDiagnostics** : Affiche les journaux des opérations et actions qui sont effectuées en relation avec votre espace de noms à l’aide de l’API ou via les clients de gestion sur le kit SDK du langage.

Les chaînes JSON du journal des opérations incluent les éléments répertoriés dans le tableau suivant :

| Nom | Description |
| ------- | ------- |
| ActivityId | ID interne, utilisé pour identifier l’activité spécifiée |
| EventName | Nom d’opération |
| ResourceId | ID de ressource Azure Resource Manager |
| SubscriptionId | Identifiant d’abonnement |
| EventTimeString | Durée de l’opération |
| EventProperties | Propriétés de l’opération |
| Statut | État de l’opération |
| Appelant | Appelant de l’opération (le Portail Azure ou le client de gestion) |
| Category | OperationalLogs |

Voici un exemple de chaîne JSON du journal des opérations :

```json
{
  "ActivityId": "6aa994ac-b56e-4292-8448-0767a5657cc7",
  "EventName": "Create Queue",
  "resourceId": "/SUBSCRIPTIONS/1A2109E3-9DA0-455B-B937-E35E36C1163C/RESOURCEGROUPS/DEFAULT-SERVICEBUS-CENTRALUS/PROVIDERS/MICROSOFT.SERVICEBUS/NAMESPACES/SHOEBOXEHNS-CY4001",
  "SubscriptionId": "1a2109e3-9da0-455b-b937-e35e36c1163c",
  "EventTimeString": "9/28/2016 8:40:06 PM +00:00",
  "EventProperties": "{\"SubscriptionId\":\"1a2109e3-9da0-455b-b937-e35e36c1163c\",\"Namespace\":\"shoeboxehns-cy4001\",\"Via\":\"https://shoeboxehns-cy4001.servicebus.windows.net/f8096791adb448579ee83d30e006a13e/?api-version=2016-07\",\"TrackingId\":\"5ee74c9e-72b5-4e98-97c4-08a62e56e221_G1\"}",
  "Status": "Succeeded",
  "Caller": "ServiceBus Client",
  "category": "OperationalLogs"
}
```

## <a name="events-and-operations-captured-in-operational-logs"></a>Événements et opérations capturés dans les journaux des opérations

Les journaux des opérations capturent toutes les opérations de gestion qui sont effectuées sur l’espace de noms Azure Service Bus. Les opérations de données ne sont pas capturées en raison du volume élevé d’opérations de données effectuées sur Azure Service Bus.

> [!NOTE]
> Pour vous aider à améliorer le suivi des opérations de données, nous vous recommandons d’utiliser un suivi côté client.

Les opérations de gestion suivantes sont capturées dans les journaux des opérations : 

| Étendue | Opération|
|-------| -------- |
| Espace de noms | <ul> <li> Créer l'espace de noms</li> <li> Mettre à jour un espace de noms </li> <li> Supprimer un espace de noms </li> <li> Mettre à jour la stratégie SharedAccess d’espace de noms </li> </ul> | 
| File d'attente | <ul> <li> Créer une file d’attente</li> <li> Mettre à jour une file d’attente</li> <li> Supprimer une file d’attente </li> <li> Supprimer automatiquement une file d’attente </li> </ul> | 
| Rubrique | <ul> <li> Créer une rubrique </li> <li> Mettre à jour une rubrique </li> <li> Supprimer une rubrique </li> <li> Supprimer automatiquement une rubrique </li> </ul> |
| Abonnement | <ul> <li> Créer un abonnement </li> <li> Mettre à jour un abonnement </li> <li> Supprimer un abonnement </li> <li> Supprimer automatiquement un abonnement </li> </ul> |

> [!NOTE]
> Actuellement, les opérations de *lecture* ne font pas l’objet d’un suivi dans les journaux des opérations.

## <a name="enable-operational-logs"></a>Activer les journaux des opérations

Les journaux des opérations sont désactivés par défaut. Pour activer les journaux de diagnostics, procédez comme suit :

1. Dans le [Portail Azure](https://portal.azure.com), accédez à votre espace de noms Azure Service Bus, puis sous **Supervision**, sélectionnez **Paramètres de diagnostics**.

   ![Le lien « Paramètres de diagnostics »](./media/service-bus-diagnostic-logs/image1.png)

1. Dans le volet **Paramètres de diagnostic**, sélectionnez **Ajouter un paramètre de diagnostic**.  

   ![Le lien « Ajouter le paramètre de diagnostic »](./media/service-bus-diagnostic-logs/image2.png)

1. Configurez les paramètres de diagnostics en procédant comme suit :

   a. Dans la zone **Nom**, entrez un nom pour les paramètres de diagnostics.  

   b. Sélectionnez l’une des trois destinations suivantes pour vos journaux de diagnostics :  
   - Si vous sélectionnez **Archiver dans un compte de stockage**, vous devez configurer le compte de stockage dans lequel les journaux de diagnostics seront stockés.  
   - Si vous sélectionnez **Diffuser vers Event Hub**, vous devez configurer Event Hub vers lequel vous souhaitez diffuser les journaux de diagnostics.
   - Si vous sélectionnez **Envoyer à Log Analytics**, vous devez spécifier l’instance de Log Analytics vers laquelle les diagnostics seront envoyés.  

   c. Activez la case à cocher **OperationalLogs**.

    ![Le volet « Paramètres de diagnostics »](./media/service-bus-diagnostic-logs/image3.png)

1. Sélectionnez **Enregistrer**.

Les nouveaux paramètres prennent effet au bout de 10 minutes environ. Les journaux d’activité s’affichent dans la cible d’archivage configurée, dans le volet **Journaux de diagnostics**.

Pour plus d’informations sur la configuration des paramètres de diagnostics, consultez [Vue d’ensemble des journaux de diagnostics Azure](../azure-monitor/platform/platform-logs-overview.md).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Service Bus, consultez :

* [Introduction à Service Bus](service-bus-messaging-overview.md)
* [Bien démarrer avec Service Bus](service-bus-dotnet-get-started-with-queues.md)