---
title: Journaux Communication Services
titleSuffix: An Azure Communication Services concept document
description: Découvrez la journalisation dans Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 10/15/2020
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: c4c9808813de80beea55e083c5bd80667ae2861f
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033117"
---
# <a name="communication-services-logs"></a>Journaux Communication Services

Azure Communication Services offre des fonctionnalités de journalisation que vous pouvez utiliser pour superviser et déboguer votre solution Communication Services. Ces fonctionnalités peuvent être configurées à l’aide du portail Azure.

## <a name="enable-diagnostic-logs-in-your-resource"></a>Activer les journaux de diagnostic dans votre ressource

La journalisation est désactivée par défaut lors de la création d’une ressource. Pour activer la journalisation, accédez au panneau **Paramètres de diagnostic** dans le menu de ressources sous la section **Supervision**. Cliquez ensuite sur **Ajouter le paramètre de diagnostic**.

Ensuite, sélectionnez la cible d’archivage de votre choix. Nous prenons en charge les comptes de stockage et Log Analytics comme cibles d’archivage. Après avoir sélectionné les types de journaux que vous souhaitez capturer, enregistrez les paramètres de diagnostic.
 
Les nouveaux paramètres prennent effet au bout de dix minutes environ. Les journaux commenceront à apparaître dans la cible d’archivage configurée, dans le volet Journaux de votre ressource Communication Services.

:::image type="content" source="./media/diagnostic-settings.png" alt-text="Options Paramètres de diagnostic d’ACS":::

Pour plus d’informations sur la configuration des diagnostics, consultez la vue d’ensemble des [journaux de ressources Azure](../../azure-monitor/platform/platform-logs-overview.md).

## <a name="resource-log-categories"></a>Catégories de journaux de ressources

Communication Services propose trois types de journaux que vous pouvez activer :

* **Journaux d’utilisation** : fournit les données d’utilisation associées à chaque offre de service facturée
* **Journaux des opérations de la conversation** : fournit les informations de base relatives au service de conversation
* **Journaux des opérations du service de SMS** : fournit les informations de base relatives au service de SMS
* **Journaux des opérations du service d’authentification** : fournit les informations de base relatives au service d’authentification

### <a name="usage-logs-schema"></a>Schéma des journaux d’utilisation

| Propriété | Description |
| -------- | ---------------|
| Timestamp | Horodatage (UTC) de la génération du journal. |
| Nom d’opération | Opération associée à l’enregistrement du journal. |
| Version de l’opération | `api-version` associée à l’opération, si operationName a été effectuée à l’aide d’une API. S’il n’existe aucune API qui corresponde à cette opération, la version représente la version de cette opération si les propriétés associées à l’opération viennent à changer. |
| Category | Catégorie de journal de l’événement. La catégorie est la granularité selon laquelle vous pouvez activer ou désactiver des journaux d’activité sur une ressource particulière. Les propriétés qui apparaissent dans l’objet blob de propriétés d’un événement sont les mêmes au sein d’un type de ressource et d’une catégorie de journal spécifique. |
| ID de corrélation : | ID des événements corrélés. Peut être utilisé pour identifier les événements corrélés entre plusieurs tables. |
| Propriétés | Autres données applicables aux différents modes de Communication Services. |
| ID d’enregistrement | ID unique d’un enregistrement d’utilisation donné. |
| Type d'utilisation | Mode d’utilisation. (par exemple, conversation, RTC, NAT, etc.) |
| Unit, type | Type d’unité sur lequel l’utilisation est basée pour un mode d’utilisation donné. (par exemple, minutes, mégaoctets, messages, etc.) |
| Quantité | Nombre d’unités utilisées ou consommées pour cet enregistrement. |

### <a name="chat-operational-logs"></a>Journaux des opérations de la conversation

| Propriété | Description |
| -------- | ---------------|
| TimeGenerated | Horodatage (UTC) de la génération du journal. |
| NomOpération | Opération associée à l’enregistrement du journal. |
| CorrelationID | ID des événements corrélés. Peut être utilisé pour identifier les événements corrélés entre plusieurs tables. |
| OperationVersion | Version d’api associée à l’opération, si operationName a été effectuée à l’aide d’une API. S’il n’existe aucune API qui corresponde à cette opération, la version représente la version de cette opération si les propriétés associées à l’opération viennent à changer. |
| Category | Catégorie de journal de l’événement. La catégorie est la granularité selon laquelle vous pouvez activer ou désactiver des journaux d’activité sur une ressource particulière. Les propriétés qui apparaissent dans l’objet blob de propriétés d’un événement sont les mêmes au sein d’un type de ressource et d’une catégorie de journal spécifique. |
| ResultType | État de l'opération. |
| ResultSignature | Sous-état de l’opération. Si cette opération correspond à un appel d’API REST, ce champ est le code d’état HTTP de l’appel REST correspondant. |
| resultDescription | Description textuelle statique de cette opération. |
| DurationMs | Durée de l’opération en millisecondes. |
| callerIpAddress | Adresse IP de l’appelant, si l’opération correspond à un appel d’API qui provient d’une entité avec une adresse IP disponible publiquement. |
| Level | Niveau de gravité de l’événement. |
| URI | URI de la requête. |
| UserId | ID d’utilisateur de l’émetteur de la demande. |
| ChatThreadId | ID de fil de conversation associé à la demande. |
| ChatMessageId | ID de message de conversation associé à la demande. |
| SdkType | Type de SDK utilisé dans la demande. |
| PlatformType | Type de plateforme utilisé dans la demande. |

### <a name="sms-operational-logs"></a>Journaux des opérations du service de SMS

| Propriété | Description |
| -------- | ---------------|
| TimeGenerated | Horodatage (UTC) de la génération du journal. |
| NomOpération | Opération associée à l’enregistrement du journal. |
| CorrelationID | ID des événements corrélés. Peut être utilisé pour identifier les événements corrélés entre plusieurs tables. |
| OperationVersion | Version d’api associée à l’opération, si operationName a été effectuée à l’aide d’une API. S’il n’existe aucune API qui corresponde à cette opération, la version représente la version de cette opération si les propriétés associées à l’opération viennent à changer. |
| Category | Catégorie de journal de l’événement. La catégorie est la granularité selon laquelle vous pouvez activer ou désactiver des journaux d’activité sur une ressource particulière. Les propriétés qui apparaissent dans l’objet blob de propriétés d’un événement sont les mêmes au sein d’un type de ressource et d’une catégorie de journal spécifique. |
| ResultType | État de l'opération. |
| ResultSignature | Sous-état de l’opération. Si cette opération correspond à un appel d’API REST, ce champ est le code d’état HTTP de l’appel REST correspondant. |
| resultDescription | Description textuelle statique de cette opération. |
| DurationMs | Durée de l’opération en millisecondes. |
| callerIpAddress | Adresse IP de l’appelant, si l’opération correspond à un appel d’API qui provient d’une entité avec une adresse IP disponible publiquement. |
| Level | Niveau de gravité de l’événement. |
| URI | URI de la requête. |
| OutgoingMessageLength | Nombre de caractères dans le message sortant. |
| IncomingMessageLength | Nombre de caractères dans le message entrant. |
| DeliveryAttempts | Nombre de tentatives effectuées pour remettre ce message. |
| PhoneNumber | Numéro de téléphone auquel le message SMS est envoyé. |
| SdkType | Type de SDK utilisé dans la demande. |
| PlatformType | Type de plateforme utilisé dans la demande. |
| Méthode | Méthode utilisée dans la demande. |

### <a name="authentication-operational-logs"></a>Journaux des opérations d’authentification

| Propriété | Description |
| -------- | ---------------|
| TimeGenerated | Horodatage (UTC) de la génération du journal. |
| NomOpération | Opération associée à l’enregistrement du journal. |
| CorrelationID | ID des événements corrélés. Peut être utilisé pour identifier les événements corrélés entre plusieurs tables. |
| OperationVersion | `api-version` associée à l’opération, si `operationName` a été effectuée à l’aide d’une API. S’il n’existe aucune API qui corresponde à cette opération, la version représente la version de cette opération si les propriétés associées à l’opération viennent à changer. |
| Category | Catégorie de journal de l’événement. La catégorie est la granularité selon laquelle vous pouvez activer ou désactiver des journaux d’activité sur une ressource particulière. Les propriétés qui apparaissent dans l’objet blob de propriétés d’un événement sont les mêmes au sein d’un type de ressource et d’une catégorie de journal spécifique. |
| ResultType | État de l'opération. |
| ResultSignature | Sous-état de l’opération. Si cette opération correspond à un appel d’API REST, ce champ est le code d’état HTTP de l’appel REST correspondant. |
| DurationMs | Durée de l’opération en millisecondes. |
| callerIpAddress | Adresse IP de l’appelant, si l’opération correspond à un appel d’API qui provient d’une entité avec une adresse IP disponible publiquement. |
| Level | Niveau de gravité de l’événement. |
| URI | URI de la requête. |
| SdkType | Type de SDK utilisé dans la demande. |
| PlatformType | Type de plateforme utilisé dans la demande. |
| Identité | Identité Communication Services associée à l’opération. |
| Étendues | Étendues Communication Services présentes dans le jeton d’accès. |
