---
title: Latences de création de rapports Azure Active Directory | Microsoft Docs
description: Découvrir le délai nécessaire pour que les événements de rapports apparaissent dans votre portail Azure
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 9b88958d-94a2-4f4b-a18c-616f0617a24e
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 05/13/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0498ee1c57cfa661884fe3209d4e089b54996fae
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89231059"
---
# <a name="azure-active-directory-reporting-latencies"></a>Latences de création de rapports Azure Active Directory

La latence est le temps qu’il faut pour que les données de rapport Azure Active Directory (Azure AD) s’affichent dans le [portail Azure](https://portal.azure.com). Cet article répertorie la latence attendue pour les différents types de rapports. 

## <a name="activity-reports"></a>Rapports d’activité

Il existe deux types de rapports d’activité :

- [Connexions](concept-sign-ins.md) – Fournit des informations sur l’utilisation des applications managées et les activités de connexion des utilisateurs
- [Journaux d'audit](concept-audit-logs.md) – Fournit des informations sur les utilisateurs et les groupes, les applications gérées et les activités de répertoire

Le tableau suivant répertorie les informations de latence pour les rapports d’activité. 

> [!NOTE]
> **Latence (95e centile)** fait référence au délai auquel 95 % des journaux d’activité seront déclarés et **Latence (99e centile)** fait référence au délai auquel 99 % des journaux d’activité seront déclarés. 
>

| Rapport | Latence (95e centile) |Latence (99e centile)|
| :-- | --- | --- |
| Journaux d’audit | 2 minutes  | 5 minutes  |
| Connexions | 2 minutes  | 5 minutes |

### <a name="how-soon-can-i-see-activities-data-after-getting-a-premium-license"></a>Au bout de combien de temps puis-je consulter les données d'activité après avoir obtenu une licence Premium ?

Si vous avez déjà des données d'activités avec votre licence gratuite, vous pouvez les voir immédiatement lors de la mise à niveau. Si vous n'avez pas encore de données, il faudra un ou deux jours pour que les données apparaissent dans les rapports après la mise à niveau vers une licence Premium.

## <a name="security-reports"></a>Rapports de sécurité

Il existe deux types de rapports de sécurité :

- [Connexions risquées](../identity-protection/overview-identity-protection.md) : une connexion risquée est une tentative de connexion susceptible de provenir d’un utilisateur autre que le propriétaire légitime d’un compte d’utilisateur. 
- [Utilisateurs avec indicateur de risque](../identity-protection/overview-identity-protection.md) : il s’agit d’un compte d’utilisateur susceptible d’être compromis. 

Le tableau suivant répertorie les informations de latence pour les rapports de sécurité.

| Rapport | Minimum | Average | Maximale |
| :-- | --- | --- | --- |
| Les utilisateurs à risque          | 5 minutes   | 15 minutes  | 2 heures  |
| Connexions risquées         | 5 minutes   | 15 minutes  | 2 heures  |

## <a name="risk-detections"></a>Détections de risques

Azure AD utilise les algorithmes Machine Learning et des modèles heuristiques adaptatifs pour détecter les actions suspectes liées aux comptes de votre utilisateur. Chaque action suspecte détectée est stockée dans un enregistrement appelé **détection d’événement à risque**.

Le tableau suivant répertorie les informations de latence pour les détections de risques.

| Rapport | Minimum | Average | Maximale |
| :-- | --- | --- | --- |
| Connexions depuis des adresses IP anonymes |5 minutes |15 minutes |2 heures |
| Connexions depuis des emplacements non connus |5 minutes |15 minutes |2 heures |
| Utilisateurs avec des informations d’identification volées |2 heures |4 heures |8 heures |
| Voyage impossible vers des emplacements inhabituels |5 minutes |1 heure |8 heures  |
| Connexions depuis des appareils infectés |2 heures |4 heures |8 heures  |
| Connexions depuis des adresses IP avec des activités suspectes |2 heures |4 heures |8 heures  |


## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble des rapports Azure AD](overview-reports.md)
* [Accès par programmation aux rapports Azure AD](concept-reporting-api.md)
* [Détections de risques dans Azure Active Directory](../identity-protection/overview-identity-protection.md)