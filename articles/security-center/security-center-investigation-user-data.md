---
title: Gérer les données utilisateur trouvées dans une enquête Azure Security Center
description: " Découvrez comment gérer les données utilisateur trouvées dans la fonctionnalité d’enquête d’Azure Security Center. "
services: operations-management-suite
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 411d7bae-c9d4-4e83-be63-9f2f2312b075
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/20/2018
ms.author: memildin
ms.openlocfilehash: 88fd707d769c7aed53160a9f22fefd15cce19a4b
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92340731"
---
# <a name="manage-user-data-found-in-an-azure-security-center-investigation"></a>Gérer les données utilisateur trouvées dans une enquête Azure Security Center
Cet article explique comment gérer les données utilisateur trouvées dans la fonctionnalité d’enquête d’Azure Security Center. Les données d’enquête sont stockées dans [Journaux Azure Monitor](../azure-monitor/log-query/log-query-overview.md) et exposées dans Security Center. La gestion des données utilisateur inclut la possibilité de supprimer ou d’exporter des données.

[!INCLUDE [gdpr-intro-sentence.md](../../includes/gdpr-intro-sentence.md)]

## <a name="searching-for-and-identifying-personal-data"></a>Recherche et identification des données personnelles
Dans le Portail Azure, vous pouvez utiliser la fonctionnalité d’enquête de Security Center pour rechercher des données personnelles. La fonctionnalité d’enquête est disponible sous **Alertes de sécurité**.

La fonctionnalité d’enquête montre toutes les entités, informations utilisateur et données sous l’onglet **Entités**.

## <a name="securing-and-controlling-access-to-personal-information"></a>Sécurisation et contrôle de l’accès aux informations personnelles
Un utilisateur de Security Center disposant du rôle Lecteur, Propriétaire, Contributeur ou Administrateur de compte peut accéder aux données client dans l’outil.

Pour en savoir plus sur les rôles Lecteur, Propriétaire et Contributeur, consultez [Rôles intégrés pour le contrôle d’accès en fonction du rôle Azure](../role-based-access-control/built-in-roles.md). Consultez [Ajout ou modification des administrateurs d’abonnements Azure](../cost-management-billing/manage/add-change-subscription-administrator.md) pour en savoir plus sur le rôle Administrateur de compte.

## <a name="deleting-personal-data"></a>Suppression des données personnelles
Un utilisateur de Security Center disposant du rôle Propriétaire, Contributeur ou Administrateur de compte peut supprimer les informations d’enquête.

Pour supprimer une enquête, vous pouvez envoyer une requête `DELETE` à l’API REST Azure Resource Manager :

```HTTP
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/features/security/incidents/{incidentName}
```

L’entrée `incidentName` peut être trouvée en répertoriant tous les incidents à l’aide d’une requête `GET` :

```HTTP
GET
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.OperationalInsights/workspaces/{workspaceName}/features/security/incidents
```

## <a name="exporting-personal-data"></a>Exportation des données personnelles
Un utilisateur de Security Center disposant du rôle Propriétaire, Contributeur ou Administrateur de compte peut exporter les informations d’enquête. Pour exporter les informations d’enquête, accédez à l’onglet **Entités** pour copier et coller les informations pertinentes.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la gestion des données utilisateur, consultez [Manage user data in Azure Security Center](security-center-privacy.md) (Gérer les données utilisateur dans Azure Security Center).
Pour en savoir plus sur la suppression des données privées dans Journaux Azure Monitor, consultez [Guide pratique pour exporter et supprimer des données privées](../azure-monitor/platform/personal-data-mgmt.md#how-to-export-and-delete-private-data).