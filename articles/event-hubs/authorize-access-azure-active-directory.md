---
title: Autoriser l’accès avec Azure Active Directory
description: Cet article fournit des informations sur l’autorisation d’accès aux ressources Event Hubs à l’aide d'Azure Active Directory.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: d794b03fdbb5429983788c74cbb05a7c13bf2d76
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92910795"
---
# <a name="authorize-access-to-event-hubs-resources-using-azure-active-directory"></a>Autoriser l’accès aux ressources Event Hubs à l'aide d'Azure Active Directory
Azure Event Hubs prend en charge l’utilisation d'Azure Active Directory (Azure AD) pour autoriser les requêtes de ressources Event Hubs. Azure AD vous permet d’utiliser le contrôle d’accès en fonction du rôle (Azure RBAC) pour accorder des autorisations à un principal de sécurité, qui peut être un utilisateur ou un principal de service d’application. Pour en savoir plus sur les rôles et les attributions de rôles, consultez [Comprendre les différents rôles](../role-based-access-control/overview.md).

## <a name="overview"></a>Vue d’ensemble
Lorsqu’un principal de sécurité (un utilisateur ou une application) tente d’accéder à une ressource Event Hubs, la requête doit être autorisée. Avec Azure AD, l’accès à une ressource est un processus en deux étapes. 

 1. Pour commencer, l’identité du principal de sécurité est authentifiée, et un jeton OAuth 2.0 est renvoyé. Le nom de la ressource pour demander un jeton est `https://eventhubs.azure.net/`. Il est identique pour tous les clouds/locataires. Pour les clients Kafka, la ressource pour demander un jeton est `https://<namespace>.servicebus.windows.net`.
 1. Ensuite, ce jeton est transmis dans le cadre d’une requête adressée au service Event Hubs pour autoriser l’accès à la ressource spécifiée.

L’étape d’authentification nécessite qu’une requête d’application contienne un jeton d’accès OAuth 2.0 au moment de l’exécution. Si une application s’exécute à partir d’une entité Azure telle qu’une machine virtuelle Azure, un groupe de machines virtuelles identiques ou une application Azure Function, elle peut utiliser une identité managée pour accéder aux ressources. Pour plus d’informations sur l’authentification des requêtes adressées par une identité managée au service Event Hubs, consultez l’article [Authentifier l’accès aux ressources Azure Event Hubs avec Azure Active Directory et les identités managées pour les ressources Azure](authenticate-managed-identity.md). 

L’étape d’autorisation exige qu’un ou plusieurs rôles Azure soient attribués au principal de sécurité. Azure Event Hubs fournit des rôles Azure qui englobent des jeux d’autorisations pour les ressources Event Hubs. Les rôles qui sont attribués à un principal de sécurité déterminent les autorisations dont disposera le principal. Pour plus d’informations sur les rôles Azure, consultez [Rôles intégrés Azure pour Azure Event Hubs](#azure-built-in-roles-for-azure-event-hubs). 

Les applications natives et applications web qui adressent des requêtes à Event Hubs peuvent également autoriser l’accès avec Azure AD. Pour savoir comment demander un jeton d’accès et l’utiliser pour autoriser les requêtes aux ressources Event Hubs, consultez [Authentifier l'accès à Azure Event Hubs avec Azure AD à partir d’une application](authenticate-application.md). 

## <a name="assign-azure-roles-for-access-rights"></a>Attribuer des rôles Azure pour les droits d’accès
Azure Active Directory (Azure AD) autorise les droits d’accès aux ressources sécurisées par le biais du [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md). Azure Event Hubs définit un ensemble de rôles intégrés Azure qui englobent les jeux d’autorisations communs utilisés pour accéder aux données Event Hubs, et vous pouvez également définir des rôles personnalisés pour accéder aux données.

Lorsqu’un rôle Azure est attribué à un principal de sécurité Azure AD, Azure octroie l’accès à ces ressources pour ce principal de sécurité. L’accès peut être limité au niveau de l’abonnement, du groupe de ressources, de l’espace de noms Event Hubs ou de toute ressource sous-jacente. Un principal de sécurité Azure AD peut correspondre à un utilisateur, à un principal de service d’application ou à une [identité managée pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="azure-built-in-roles-for-azure-event-hubs"></a>Rôles intégrés Azure pour Azure Event Hubs
Azure fournit les rôles intégrés Azure suivants pour autoriser l’accès aux données Event Hubs à l’aide d’Azure AD et d’OAuth :

| Role | Description | 
| ---- | ----------- | 
| [Propriétaire de données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-owner) | Utilisez ce rôle pour accorder un accès complet aux ressources Event Hubs. |
| [Expéditeur de données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-sender) | Utilisez ce rôle pour accorder un accès en envoi aux ressources Event Hubs. |
| [Récepteur de données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-receiver) | Utilisez ce rôle pour accorder un accès en réception/utilisation aux ressources Event Hubs. |

Pour les rôles intégrés du registre de schémas, consultez [Rôles du registre de schémas](schema-registry-overview.md#azure-role-based-access-control).

## <a name="resource-scope"></a>Étendue des ressources 
Avant d’attribuer un rôle Azure à un principal de sécurité, déterminez l’étendue de l’accès dont doit disposer le principal de sécurité. Selon les bonnes pratiques, il est toujours préférable d’accorder la plus petite étendue possible.

La liste suivante décrit les niveaux auxquels vous pouvez étendre l’accès aux ressources Event Hubs, en commençant par la plus petite étendue :

- **Groupe de consommateurs**  : Dans cette étendue, l’attribution de rôle s’applique uniquement à cette entité. Actuellement, le portail Azure ne prend pas en charge l’attribution d’un rôle Azure à un principal de sécurité de ce niveau. 
- **Hub d'événements**  : L’attribution de rôle s’applique à l’entité Event Hub et au groupe de consommateurs sous-jacent.
- **Espace de noms**  : L’attribution de rôle s’étend à toute la topologie d'Event Hubs sous l’espace de noms et au groupe de consommateurs qui lui est associé.
- **Groupe de ressources**  : L’attribution de rôle s’applique à toutes les ressources Event Hubs sous le groupe de ressources.
- **Abonnement** : L’attribution de rôle s’applique à toutes les ressources Event Hubs dans tous les groupes de ressources de l’abonnement.

> [!NOTE]
> - Gardez à l’esprit que la propagation des attributions de rôles Azure peut prendre cinq minutes. 
> - Ce contenu s’applique à la fois à Event Hubs et à Event Hubs pour Apache Kafka. Pour plus d’informations sur la prise en charge d’Event Hubs pour Kafka, consultez [Event Hubs pour Kafka - Sécurité et authentification](event-hubs-for-kafka-ecosystem-overview.md#security-and-authentication).


Pour plus d’informations sur la définition des rôles intégrés, consultez [Comprendre les définitions de rôles](../role-based-access-control/role-definitions.md#management-and-data-operations). Pour plus d’informations sur la création de rôles personnalisés Azure, consultez [Rôles personnalisés Azure](../role-based-access-control/custom-roles.md).



## <a name="samples"></a>Exemples
- [Exemples Microsoft.Azure.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    Ces exemples utilisent l'ancienne bibliothèque **Microsoft.Azure.EventHubs** , mais vous pouvez facilement mettre celle-ci à jour à l'aide de la dernière bibliothèque **Azure.Messaging.EventHubs**. Pour migrer l'exemple de l'ancienne bibliothèque vers la nouvelle, consultez le [Guide de migration de Microsoft.Azure.EventHubs vers Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
- [Exemples Azure.Messaging.EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp)

    Cet exemple a été mis à jour pour utiliser la dernière bibliothèque **Azure.Messaging.EventHubs**.
- [Event Hubs pour Kafka - Exemples OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth). 


## <a name="next-steps"></a>Étapes suivantes
- Pour savoir comment attribuer un rôle intégré Azure à un principal de sécurité, consultez [Authentifier l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory.](authenticate-application.md)
- Découvrez [comment créer des rôles personnalisés avec Azure RBAC](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac/CustomRole).
- Découvrez [comment utiliser Azure Active Directory avec EH](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac/AzureEventHubsSDK)

Consultez les articles associés suivants :

- [Authentifier les requêtes adressées à Azure Event Hubs à partir d’une application à l’aide d’Azure Active Directory](authenticate-application.md)
- [Authentifier une identité managée avec Azure Active Directory pour accéder aux ressources Event Hubs](authenticate-managed-identity.md)
- [Authentifier les requêtes adressées à Azure Event Hubs à l’aide de signatures d’accès partagé](authenticate-shared-access-signature.md)
- [Authentifier l’accès aux ressources Event Hubs avec des signatures d’accès partagé](authorize-access-shared-access-signature.md)
