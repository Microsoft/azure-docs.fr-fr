---
title: Résoudre les problèmes d’authentification et d’autorisation - Azure Event Hubs
description: Cet article fournit des informations sur la résolution des problèmes d’authentification et d’autorisation avec Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 368fd8efda1b828f99bc41da0743768989c1a601
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92329608"
---
# <a name="troubleshoot-authentication-and-authorization-issues---azure-event-hubs"></a>Résoudre les problèmes d’authentification et d’autorisation - Azure Event Hubs
L’article [Résoudre les problèmes de connectivité](troubleshooting-guide.md) fournit des conseils pour résoudre les problèmes de connectivité avec Azure Event Hubs. Cet article fournit des conseils et des suggestions sur la résolution des problèmes d’authentification et d’autorisation avec Azure Event Hubs. 

## <a name="if-you-are-using-azure-active-directory"></a>Si vous utilisez Azure Active Directory
Si vous utilisez Azure Active Directory (Azure AD) pour vous authentifier et autoriser avec Azure Event Hubs, vérifiez que l’identité qui accède à Event Hub est un membre du **rôle Azure** avec l’**étendue de ressources appropriée** (groupe de consommateurs, Event Hub, espace de noms, groupe de ressources ou abonnement).

### <a name="azure-roles"></a>Rôles Azure
- [Le propriétaire de données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-owner) pour un accès complet aux ressources Event Hubs.
- [Expéditeur des données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-receiver) pour l’accès d’envoi.
- [Récepteur des données Azure Event Hubs](../role-based-access-control/built-in-roles.md#azure-event-hubs-data-sender) pour l’accès de réception.

Pour les rôles intégrés du registre de schémas, consultez [Rôles du registre de schémas](schema-registry-overview.md#azure-role-based-access-control).

### <a name="resource-scopes"></a>Étendues de ressources
- **Groupe de consommateurs** : Dans cette étendue, l’attribution de rôle s’applique uniquement à cette entité. Actuellement, le portail Azure ne prend pas en charge l’attribution d’un rôle Azure à un principal de sécurité de ce niveau. 
- **Hub d'événements** : L’attribution de rôle s’applique à l’entité Event Hub et au groupe de consommateurs sous-jacent.
- **Espace de noms** : L’attribution de rôle s’étend à toute la topologie d'Event Hubs sous l’espace de noms et au groupe de consommateurs qui lui est associé.
- **Groupe de ressources** : L’attribution de rôle s’applique à toutes les ressources Event Hubs sous le groupe de ressources.
- **Abonnement**: L’attribution de rôle s’applique à toutes les ressources Event Hubs dans tous les groupes de ressources de l’abonnement.

Pour plus d’informations, consultez les articles suivants :

- [Authentifier une application avec Azure Active Directory pour accéder aux ressources Event Hubs](authenticate-application.md)
- [Autoriser l’accès aux ressources Event Hubs à l’aide d’Azure Active Directory](authorize-access-azure-active-directory.md)

## <a name="if-you-are-using-shared-access-signatures-sas"></a>Si vous utilisez des signatures d’accès partagé (SAP)
Si vous utilisez des [SAS](authenticate-shared-access-signature.md), procédez comme suit : 

- Vérifiez que la clé SAS que vous utilisez est correcte. Si ce n’est pas le cas, utilisez la clé SAS appropriée.
- Vérifiez que la clé dispose des autorisations appropriées (envoi, réception ou gestion). Si ce n’est pas le cas, utilisez une clé qui dispose de l’autorisation dont vous avez besoin. 
- Vérifiez si la clé a expiré. Nous vous recommandons de renouveler la signature d’accès partagé bien avant l’expiration. En cas de variations d'horloges entre le client et les nœuds de service Event Hubs, le jeton d’authentification peut expirer avant que le client ne l’ait réalisé. Les variations d'horloges des comptes d’implémentation actuels sont de 5 minutes, c’est-à-dire que le client renouvelle le jeton 5 minutes avant son expiration. Par conséquent, si les variations d'horloges sont supérieures à 5 minutes, le client peut observer des défaillances d’authentification intermittentes.
- Si **l’heure de début SAS** est définie sur **maintenant**, vous pouvez constater des défaillances intermittentes pour les premières minutes en raison des variations d'horloges (heures différentes sur les différents ordinateurs). Définissez une heure de début au moins 15 minutes avant l’heure courante ou ne définissez aucune heure. Généralement, le même principe s’applique également à l’heure d’expiration. 

Pour plus d’informations, consultez les articles suivants : 

- [S’authentifier à l’aide de signatures d’accès partagé (SAS)](authenticate-shared-access-signature.md). 
- [Autorisation de l’accès aux ressources Event Hubs à l’aide de signatures d’accès partagé](authorize-access-shared-access-signature.md)

## <a name="next-steps"></a>Étapes suivantes

Voir les articles suivants :

* [Résoudre les problèmes de connectivité](troubleshooting-guide.md)
