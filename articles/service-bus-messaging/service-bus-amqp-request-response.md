---
title: Opérations de demande/réponse AMQP 1.0 dans Azure Service Bus
description: Cet article énumère les opérations basées sur une requête-réponse AMQP dans le service Microsoft Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: b845f4086ee1ac4fe868571c1754caf6d29b9021
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88064412"
---
# <a name="amqp-10-in-microsoft-azure-service-bus-request-response-based-operations"></a>AMQP 1.0 dans Microsoft Azure Service Bus : opérations basées sur les requêtes-réponses

Cet article définit la liste des opérations basées sur les requêtes-réponses de Microsoft Azure Service Bus. Ces informations sont basées sur la première ébauche d’AMQP Management Version 1.0.  
  
Pour obtenir un guide détaillé sur le protocole AMQP 1.0 au niveau des câbles qui explique comment Service Bus implémente la spécification technique AMQP OASIS et s’appuie sur celle-ci, consultez le [Guide du protocole AMQP 1.0 dans Azure Service Bus et Event Hubs][Guide du protocole AMQP 1.0].  
  
## <a name="concepts"></a>Concepts  
  
### <a name="entity-description"></a>Description d’entité  

Une description d’entité fait référence à un objet Service Bus [classe QueueDescription](/dotnet/api/microsoft.servicebus.messaging.queuedescription), [classe TopicDescription](/dotnet/api/microsoft.servicebus.messaging.topicdescription) ou [classe SubscriptionDescription](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription).  
  
### <a name="brokered-message"></a>Message réparti  

Représente un message Service Bus mappé à un message AMQP. Le mappage est défini dans le [guide du protocole AMQP de Service Bus](service-bus-amqp-protocol-guide.md).  
  
## <a name="attach-to-entity-management-node"></a>Attacher à un nœud de gestion d’entité  

Toutes les opérations décrites dans ce document suivent un modèle demande/réponse sont limitées à une entité et nécessitent l’attachement à un nœud de gestion d’entité.  
  
### <a name="create-link-for-sending-requests"></a>Créer le lien pour l’envoi des demandes  

Crée un lien vers le nœud de gestion pour envoyer des demandes.  
  
```
requestLink = session.attach(
role: SENDER,
    target: { address: "<entity address>/$management" },
    source: { address: ""<my request link unique address>" }
)

```
  
### <a name="create-link-for-receiving-responses"></a>Créer le lien pour la réception des réponses  

Crée un lien pour la réception des réponses à partir du nœud de gestion.  
  
```
responseLink = session.attach(
role: RECEIVER,
    source: { address: "<entity address>/$management" }
    target: { address: "<my response link unique address>" }
)

```
  
### <a name="transfer-a-request-message"></a>Transférer un message de demande  

Transfère un message de demande.  
Un état de transaction peut être ajouté pour les opérations prenant en charge la transaction.

```
requestLink.sendTransfer(
        Message(
                properties: {
                        message-id: <request id>,
                        reply-to: "<my response link unique address>"
                },
                application-properties: {
                        "operation" -> "<operation>",
                }
        ),
        [Optional] State = transactional-state: {
                txn-id: <txn-id>
        }
)
```
  
### <a name="receive-a-response-message"></a>Recevoir un message de réponse  

Reçoit le message de réponse à partir du lien de réponse.  
  
```
responseMessage = responseLink.receiveTransfer()
```
  
La réponse présente le format suivant :
  
```
Message(
properties: {
        correlation-id: <request id>
    },
    application-properties: {
            "statusCode" -> <status code>,
            "statusDescription" -> <status description>,
           },
)

```
  
### <a name="service-bus-entity-address"></a>Adresse d’entité Service Bus  

Les entités Service Bus doivent être traitées comme suit :  
  
|Type d'entité|Adresse|Exemple|  
|-----------------|-------------|-------------|  
|queue|`<queue_name>`|`“myQueue”`<br /><br /> `“site1/myQueue”`|  
|topic|`<topic_name>`|`“myTopic”`<br /><br /> `“site2/page1/myQueue”`|  
|subscription|`<topic_name>/Subscriptions/<subscription_name>`|`“myTopic/Subscriptions/MySub”`|  
  
## <a name="message-operations"></a>Opérations de message  
  
### <a name="message-renew-lock"></a>Verrouillage du renouvellement du message  

Étend le verrouillage d’un message à la durée spécifiée dans la description d’entité.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:renew-lock`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
 Le corps du message de requête doit contenir une section amqp-value comprenant un mappage avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|`lock-tokens`|tableau d’uuid|Oui|Jetons de verrouillage de message à renouveler.|  

> [!NOTE]
> Les jetons de verrouillage sont la propriété `DeliveryTag` sur des messages reçus. Regardez l’exemple suivant dans le [Kit de développement logiciel (SDK) .NET SDK](https://github.com/Azure/azure-service-bus-dotnet/blob/6f144e91310dcc7bd37aba4e8aebd535d13fa31a/src/Microsoft.Azure.ServiceBus/Amqp/AmqpMessageConverter.cs#L336) qui les récupère. Le jeton peut également apparaître dans « DeliveryAnnotations » en tant que « x-opt-lock-token ». Toutefois, cela n’est pas garanti et il faut préférer `DeliveryTag`. 
> 
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec.|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section amqp-value comprenant un mappage avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|expirations|tableau d’horodatage|Oui|La nouvelle expiration du jeton de verrouillage de message correspond désormais aux jetons de verrouillage des demandes.|  
  
### <a name="peek-message"></a>Lire furtivement un message  

Lit furtivement les messages sans les verrouiller.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:peek-message`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|`from-sequence-number`|long|Oui|Numéro de séquence à partir duquel commencer la lecture furtive.|  
|`message-count`|int|Oui|Nombre maximal de messages à lire furtivement.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK : a plus de messages<br /><br /> 204 : aucun contenu – plus de messages|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|Cloud vers appareil|liste des mappages|Oui|Liste de messages dans laquelle chaque mappage représente un message.|  
  
Le mappage représentant un message doit contenir les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|message|tableau d’octets|Oui|Message codé en filaire AMPQ 1.0.|  
  
### <a name="schedule-message"></a>Message de planification  

Messages de planification. Cette opération prend en charge la transaction.
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:schedule-message`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|Cloud vers appareil|liste des mappages|Oui|Liste de messages dans laquelle chaque mappage représente un message.|  
  
Le mappage représentant un message doit contenir les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|message-id|string|Oui|`amqpMessage.Properties.MessageId` comme chaîne|  
|session-id|string|Non|`amqpMessage.Properties.GroupId as string`|  
|partition-key|string|Non|`amqpMessage.MessageAnnotations.”x-opt-partition-key"`|
|via-partition-key|string|Non|`amqpMessage.MessageAnnotations."x-opt-via-partition-key"`|
|message|tableau d’octets|Oui|Message codé en filaire AMPQ 1.0.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec.|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section **amqp-value** comprenant un mappage avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|sequence-numbers|tableau de type long|Oui|Numéro de séquence des messages planifiés. Le numéro de séquence est utilisé pour annuler.|  
  
### <a name="cancel-scheduled-message"></a>Annuler le message planifié  

Annule les messages planifiés.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:cancel-scheduled-message`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|sequence-numbers|tableau de type long|Oui|Numéros de séquence des messages planifiés pour annuler.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec.|  
|statusDescription|string|Non|Description de l’état.|   
  
## <a name="session-operations"></a>Opérations de session  
  
### <a name="session-renew-lock"></a>Verrouillage de renouvellement de session  

Étend le verrouillage d’un message à la durée spécifiée dans la description d’entité.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:renew-session-lock`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|session-id|string|Oui|ID de la session.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK : a plus de messages<br /><br /> 204 : aucun contenu – plus de messages|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section **amqp-value** comprenant un mappage avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|expiration|timestamp|Oui|Nouvelle expiration.|  
  
### <a name="peek-session-message"></a>Message de session de lecture furtive  

Lit furtivement les messages de session sans les verrouiller.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:peek-message`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|from-sequence-number|long|Oui|Numéro de séquence à partir duquel commencer la lecture furtive.|  
|message-count|int|Oui|Nombre maximal de messages à lire furtivement.|  
|session-id|string|Oui|ID de la session.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK : a plus de messages<br /><br /> 204 : aucun contenu – plus de messages|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section **amqp-value** comprenant un mappage avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|Cloud vers appareil|liste des mappages|Oui|Liste de messages dans laquelle chaque mappage représente un message.|  
  
 Le mappage représentant un message doit contenir les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|message|tableau d’octets|Oui|Message codé en filaire AMPQ 1.0.|  
  
### <a name="set-session-state"></a>Définir l’état d’une session  

Définit l’état d’une session.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:set-session-state`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|session-id|string|Oui|ID de la session.|  
|session-state|tableau d’octets|Oui|Données binaires opaques.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec|  
|statusDescription|string|Non|Description de l’état.|  
  
### <a name="get-session-state"></a>Obtenir l’état d’une session  

Obtient l’état d’une session.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:get-session-state`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|session-id|string|Oui|ID de la session.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|session-state|tableau d’octets|Oui|Données binaires opaques.|  
  
### <a name="enumerate-sessions"></a>Énumérer les sessions  

Énumère les sessions sur une entité de messagerie.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:get-message-sessions`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|last-updated-time|timestamp|Oui|Filtre permettant d’inclure uniquement les sessions mises à jour après une durée donnée.|  
|skip|int|Oui|Ignore un certain nombre de sessions.|  
|top|int|Oui|Nombre maximal de sessions.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK : a plus de messages<br /><br /> 204 : aucun contenu – plus de messages|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|skip|int|Oui|Nombre de sessions ignorées si le code d’état est 200.|  
|sessions-ids|tableau de chaînes|Oui|Tableau d’ID de session si le code d’état est 200.|  
  
## <a name="rule-operations"></a>Opérations relatives aux règles  
  
### <a name="add-rule"></a>Ajout de règle  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:add-rule`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|rule-name|string|Oui|Nom de règle, à l’exception des noms d’abonnement et de rubrique.|  
|rule-description|map|Oui|Description de la règle comme indiqué dans la section suivante.|  
  
Le mappage **rule-description** doit inclure les entrées suivantes, où **sql-filter** et **correlation-filter** s’excluent mutuellement :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|sql-filter|map|Oui|`sql-filter`, comme indiqué dans la section suivante.|  
|correlation-filter|map|Oui|`correlation-filter`, comme indiqué dans la section suivante.|  
|sql-rule-action|map|Oui|`sql-rule-action`, comme indiqué dans la section suivante.|  
  
Le mappage sql-filter doit inclure les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|expression|string|Oui|Expression de filtre Sql.|  
  
Le mappage **correlation-filter** doit inclure au moins l’une des entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|correlation-id|string|Non||  
|message-id|string|Non||  
|to|string|Non||  
|reply-to|string|Non||  
|label|string|Non||  
|session-id|string|Non||  
|reply-to-session-id|string|Non||  
|content-type|string|Non||  
|properties|map|Non|Mappage sur [BrokeredMessage.Properties](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) de Service Bus.|  
  
Le mappage **sql-rule-action** doit inclure les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|expression|string|Oui|Expression d’action SQL.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec|  
|statusDescription|string|Non|Description de l’état.|  
  
### <a name="remove-rule"></a>Supprimer la règle  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:remove-rule`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|rule-name|string|Oui|Nom de règle, à l’exception des noms d’abonnement et de rubrique.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec|  
|statusDescription|string|Non|Description de l’état.|  
  
### <a name="get-rules"></a>Obtenir les règles

#### <a name="request"></a>Requête

Le message de requête doit inclure les propriétés d’application suivantes :

|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:enumerate-rules`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  

Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|top|int|Oui|Le nombre de règles à extraire dans la page.|  
|skip|int|Oui|Le nombre de règles à ignorer. Définit l’index de départ (+1) dans la liste des règles. | 

#### <a name="response"></a>response

Le message de réponse inclut les propriétés suivantes :

|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec|  
|rules| tableau de mappage|Oui|Tableau de règles. Chaque règle est représentée par un mappage.|

Chaque entrée de mappage dans le tableau inclut les propriétés suivantes :

|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|rule-description|tableau d’objets décrits|Oui|`com.microsoft:rule-description:list` avec le code AMQP décrit 0x0000013700000004| 

`com.microsoft.rule-description:list` est un tableau d’objets décrits. Le tableau inclut les éléments suivants :

|Index|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
| 0 | tableau d’objets décrits | Oui | `filter` comme indiqué ci-dessous. |
| 1 | tableau d’objet décrit | Oui | `ruleAction` comme indiqué ci-dessous. |
| 2 | string | Oui | nom de la règle. |

`filter` peut être d’un des types suivants :

| Nom du descripteur | Code du descripteur | Valeur |
| --- | --- | ---|
| `com.microsoft:sql-filter:list` | 0x000001370000006 | Filtre SQL |
| `com.microsoft:correlation-filter:list` | 0x000001370000009 | Filtre de corrélation |
| `com.microsoft:true-filter:list` | 0x000001370000007 | Filtre True représentant 1=1 |
| `com.microsoft:false-filter:list` | 0x000001370000008 | Filtre False représentant 1=0 |

`com.microsoft:sql-filter:list` est un tableau décrit qui inclut :

|Index|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
| 0 | string | Oui | Expression de filtre SQL |

`com.microsoft:correlation-filter:list` est un tableau décrit qui inclut :

|Index (s’il existe)|Type de valeur|Contenu de la valeur|  
|---------|----------------|--------------|
| 0 | string | ID de corrélation : |
| 1 | string | ID de message |
| 2 | string | À |
| 3 | string | Adresse de réponse |
| 4 | string | Étiquette |
| 5 | string | ID de la session |
| 6 | string | ID de session de réponse|
| 7 | string | Type de contenu |
| 8 | Mappage | Mappage des propriétés définies de l’application |

`ruleAction` peut être d’un des types suivants :

| Nom du descripteur | Code du descripteur | Valeur |
| --- | --- | ---|
| `com.microsoft:empty-rule-action:list` | 0x0000013700000005 | Action de règle vide - aucune action de règle présente |
| `com.microsoft:sql-rule-action:list` | 0x0000013700000006 | Actions de règle SQL |

`com.microsoft:sql-rule-action:list` est un tableau d’objets décrits dont la première entrée est une chaîne qui contient l’expression de l’action de règle SQL.

## <a name="deferred-message-operations"></a>Opérations relatives aux messages différés  
  
### <a name="receive-by-sequence-number"></a>Recevoir par numéro de séquence  

Reçoit des messages différés par numéro de séquence.  
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:receive-by-sequence-number`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|sequence-numbers|tableau de type long|Oui|Numéros de séquence.|  
|receiver-settle-mode|ubyte|Oui|Mode de **réglage du récepteur** tel que spécifié dans AMQP core v1.0.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec|  
|statusDescription|string|Non|Description de l’état.|  
  
Le corps du message de réponse doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|Cloud vers appareil|liste des mappages|Oui|Liste de messages dans laquelle chaque mappage représente un message.|  
  
Le mappage représentant un message doit contenir les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|lock-token|uuid|Oui|Jeton de verrouillage si `receiver-settle-mode` est 1.|  
|message|tableau d’octets|Oui|Message codé en filaire AMPQ 1.0.|  
  
### <a name="update-disposition-status"></a>Mettre à jour le statut de disposition  

Met à jour le statut de disposition des messages différés. Cette opération prend en charge les transactions.
  
#### <a name="request"></a>Requête  

Le message de requête doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|opération|string|Oui|`com.microsoft:update-disposition`|  
|`com.microsoft:server-timeout`|uint|Non|Délai d’expiration du serveur de l’opération en millisecondes.|  
  
Le corps du message de requête doit contenir une section **amqp-value** comprenant un **mappage** avec les entrées suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|disposition-status|string|Oui|Terminé<br /><br /> abandonné<br /><br /> interrompu|  
|lock-tokens|tableau d’uuid|Oui|Jetons de verrouillage de message pour mettre à jour le statut de disposition.|  
|deadletter-reason|string|Non|Peut être défini si le statut de disposition est défini sur **interrompu**.|  
|deadletter-description|string|Non|Peut être défini si le statut de disposition est défini sur **interrompu**.|  
|properties-to-modify|map|Non|Liste des propriétés de message réparti Service Bus à modifier.|  
  
#### <a name="response"></a>response  

Le message de réponse doit inclure les propriétés d’application suivantes :  
  
|Clé|Type de valeur|Obligatoire|Contenu de la valeur|  
|---------|----------------|--------------|--------------------|  
|statusCode|int|Oui|Code de réponse HTTP [RFC2616]<br /><br /> 200 : OK-réussite, sinon échec|  
|statusDescription|string|Non|Description de l’état.|

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur AMQP et Service Bus, consultez les liens suivants :

* [Vue d’ensemble du protocole AMQP de Service Bus]
* [Guide du protocole AMQP 1.0]
* [AMQP in Service Bus for Windows Server (AMQP dans Service Bus pour Windows Server)]

[Vue d’ensemble du protocole AMQP de Service Bus]: service-bus-amqp-overview.md
[Guide du protocole AMQP 1.0]: service-bus-amqp-protocol-guide.md
[AMQP in Service Bus for Windows Server (AMQP dans Service Bus pour Windows Server)]: /previous-versions/service-bus-archive/dn282144(v=azure.100)