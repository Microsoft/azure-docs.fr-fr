---
title: Qu’est-ce qu’une notification Azure Service Health ?
description: Les notifications Service Health vous permettent de voir les messages relatifs à l’intégrité du service publiés par Microsoft Azure.
ms.topic: conceptual
ms.date: 4/12/2018
ms.openlocfilehash: 4f2b4a0779d775ed5be5bfa28831cccc68c33caf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86529010"
---
# <a name="use-the-azure-portal-to-view-service-health-notifications"></a>Utiliser le portail Azure pour afficher les notifications d’intégrité du service

Les notifications Service Health sont publiées par Azure et contiennent des informations sur les ressources de votre abonnement. Ces notifications constituent une sous-classe des événements du journal d’activité, et figurent également dans le journal d’activité. Selon leur classe, les notifications Service Health peuvent être fournies à titre d’information ou nécessiter une action de votre part.

Il existe plusieurs classes de notifications Service Health :  

- **Action nécessaire :** Azure peut remarquer un événement inhabituel dans votre compte et vous aider à résoudre le problème. Azure vous envoie une notification contenant des informations sur les actions à effectuer, ou les coordonnées des ingénieurs ou du support Azure.  
- **Incident :** un événement impactant le service affecte actuellement une ou plusieurs des ressources de votre abonnement.  
- **Maintenance :** une activité de maintenance planifiée peut impacter une ou plusieurs ressources de votre abonnement.  
- **Informations :** suggestions d’optimisations pouvant aider à améliorer l’utilisation de vos ressources. 
- **Sécurité :** informations de sécurité urgentes concernant vos solutions exécutées dans Azure.

Chaque notification d’intégrité de service contient des détails sur l’étendue et l’impact sur vos ressources. Les détails sont les suivants :

Nom de la propriété | Description
-------- | -----------
channels | L’une des valeurs suivantes : **Admin** ou **Opération**.
correlationId | Généralement un GUID au format chaîne. Les événements qui appartiennent à la même action partagent généralement le même correlationId.
eventDataId | L’identificateur unique d’un événement.
eventName | Titre de l’événement.
level | Niveau de l’événement
resourceProviderName | Nom du fournisseur de ressources de la ressource impactée.
resourceType| Le type de ressource de la ressource affectée.
subStatus | En général, le code d’état HTTP de l’appel REST correspondant. Peut également inclure d’autres chaînes décrivant un sous-état. Par exemple : OK (code d’état HTTP : 200), Créé (code d’état HTTP : 201), Accepté (code d’état HTTP : 202, Aucun contenu (code d’état HTTP : 204, Requête incorrecte (code d’état HTTP : 400, Introuvable (code d’état HTTP : 404), Conflit (code d’état HTTP : 409), Erreur interne du serveur (code d’état HTTP : 500), Service indisponible (code d’état HTTP : 503) et Dépassement de délai de la passerelle (code d’état HTTP : 504).
eventTimestamp | Horodatage de la génération de l’événement effectuée par le service Azure lors du traitement de la requête correspondant à l’événement.
submissionTimestamp | Horodatage lorsque l’événement est devenu disponible pour l’interrogation.
subscriptionId | L’abonnement Azure dans lequel l’événement est enregistré.
status | Chaîne décrivant l’état de l’opération. Voici plusieurs valeurs courantes : **Started**, **In Progress**, **Succeeded**, **Failed**, **Active** et **Resolved**.
operationName | Nom de l’opération.
catégorie | Cette propriété est toujours **ServiceHealth**.
resourceId | ID de ressource de la ressource impactée.
Properties.title | Le titre localisé pour cette communication. L’anglais est la langue par défaut.
Properties.communication | Les détails localisés de la communication avec le balisage HTML. L’anglais est la langue par défaut.
Properties.incidentType | L’une des valeurs suivantes : **ActionRequired**, **Information**, **Incident**, **Maintenance** ou **Security**.
Properties.trackingId | Incident associé à cet événement. Cela permet de mettre en corrélation les événements liés à un incident.
Properties.impactedServices | Objet blob JSON placé dans une séquence d’échappement, qui décrit les services et régions impactés par l’incident. La propriété inclut une liste de services, ayant chacun un **ServiceName** et une liste de régions impactées, ayant chacune un **RegionName**.
Properties.defaultLanguageTitle | Communication dans la langue par défaut.
Properties.defaultLanguageContent | Communication dans la langue par défaut, au format HTML ou texte brut.
Properties.stage | Les valeurs possibles pour **Incident** et **Security** sont **Active**, **Resolved** ou **RCA**. Pour **ActionRequired** ou **Informations**, **Active** est la seule valeur possible. Pour **Maintenance**, les valeurs possibles sont : **Active**, **Planned**, **InProgress**, **Canceled**, **Rescheduled**, **Resolved** et **Complete**.
Properties.communicationId | Communication associée à cet événement.

### <a name="details-on-service-health-level-information"></a>Détails sur les informations au niveau de l’intégrité du service

**Action Required** (properties.incidentType == ActionRequired)
- Information – Une action de l’administrateur est nécessaire pour éviter un impact sur les services existants.
    
**Maintenance** (properties.incidentType == Maintenance)
- Avertissement – Maintenance d’urgence
- Information – Maintenance planifiée standard

**Information** (properties.incidentType == Information)
- Information – Une intervention de l’administrateur peut être nécessaire pour éviter un impact sur les services existants.

**Sécurité** (properties.incidentType == Security)
- Avertissement – Avis de sécurité qui concerne les services existants et peut nécessiter une intervention de l’administrateur.
- Information – Avis de sécurité qui concerne les services existants.

**Problèmes liés aux services** (properties.incidentType == Incident)
- Erreur - Problèmes étendus sur plusieurs services dans plusieurs régions affectant un large éventail de clients.
- Avertissement - Problèmes sur des services et/ou des régions spécifiques affectant un sous-ensemble de clients.
- Information - Problèmes ayant un impact sur les opérations de gestion et/ou la latence, sans affecter la disponibilité du service.
