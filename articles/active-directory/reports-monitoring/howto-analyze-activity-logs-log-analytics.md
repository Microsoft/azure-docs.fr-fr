---
title: Analyser les journaux d'activité à l'aide des journaux Azure Monitor | Microsoft Docs
description: Apprenez à analyser les journaux d'activité Azure Active Directory à l'aide des journaux Azure Monitor
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 4535ae65-8591-41ba-9a7d-b7f00c574426
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 04/18/2019
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0c4fde22b1b8d72ae8ae775c090e0da25ce0665f
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96181167"
---
# <a name="analyze-azure-ad-activity-logs-with-azure-monitor-logs"></a>Analyser les journaux d'activité Azure AD à l'aide des journaux Azure Monitor

Après avoir [intégré les journaux d’activité Azure AD aux journaux d’activité Azure Monitor](howto-integrate-activity-logs-with-log-analytics.md), vous pouvez exploiter la puissance des journaux d’activité Azure Monitor pour obtenir des insights sur votre environnement. Vous pouvez également installer les [vues Log Analytics pour les journaux d’activité Azure AD](howto-install-use-log-analytics-views.md). Vous avez ainsi accès à différents rapports prédéfinis qui affichent des événements d’audit et de connexion qui se produisent dans votre environnement.

Cet article vous montre comment analyser les journaux d’activité Azure AD dans votre espace de travail Log Analytics. 

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="prerequisites"></a>Prérequis 

Pour suivre l’article, vous devez préalablement :

* Espace de travail Log Analytics dans votre abonnement Azure. Découvrez comment [créer un espace de travail Log Analytics](../../azure-monitor/learn/quick-create-workspace.md).
* Effectuer les étapes nécessaires pour [envoyer les journaux d’activité Azure AD dans votre espace de travail Log Analytics](howto-integrate-activity-logs-with-log-analytics.md).
*  [Accéder](../../azure-monitor/platform/manage-access.md#manage-access-using-workspace-permissions) à l’espace de travail Log Analytics
* Les rôles suivants dans Azure Active Directory (si vous accédez à Log Analytics via le portail Azure Active Directory)
    - Administrateur de la sécurité
    - Lecteur de sécurité
    - Lecteur de rapport
    - Administrateur général
    
## <a name="navigate-to-the-log-analytics-workspace"></a>Accéder à l’espace de travail Log Analytics

1. Connectez-vous au [portail Azure](https://portal.azure.com). 

2. Sélectionnez **Azure Active Directory**, puis sélectionnez **Journaux d’activité** dans la section **Supervision** pour ouvrir votre espace de travail Log Analytics. L’espace de travail s’ouvre avec une requête par défaut.

    ![Requête par défaut](./media/howto-analyze-activity-logs-log-analytics/defaultquery.png)


## <a name="view-the-schema-for-azure-ad-activity-logs"></a>Afficher le schéma pour les journaux d’activité Azure AD

Les journaux d’activité sont envoyés (push) dans les tableaux **AuditLogs** et **SigninLogs** dans l’espace de travail. Pour afficher le schéma de ces tableaux :

1. Dans la vue de la requête par défaut, de la section précédente, sélectionnez **Schéma** et développez l’espace de travail. 

2. Développez la section **Gestion des journaux**, puis développez **AuditLogs** ou **SigninLogs** pour afficher le schéma des journaux.
    ![Journaux d’audit](./media/howto-analyze-activity-logs-log-analytics/auditlogschema.png) ![Journaux de connexion](./media/howto-analyze-activity-logs-log-analytics/signinlogschema.png)

## <a name="query-the-azure-ad-activity-logs"></a>Exécuter des requêtes sur les journaux d’activité Azure AD

Maintenant que vous voyez les journaux d’activité dans votre espace de travail, vous pouvez exécuter des requêtes dessus. Par exemple, pour savoir quelles applications ont été le plus utilisées la semaine dernière, remplacez la requête par défaut par la requête suivante et sélectionnez **Exécuter**

```
SigninLogs 
| where CreatedDateTime >= ago(7d)
| summarize signInCount = count() by AppDisplayName 
| sort by signInCount desc 
```

Pour obtenir les principaux événements d’audit générés la semaine dernière, utilisez la requête suivante :

```
AuditLogs 
| where TimeGenerated >= ago(7d)
| summarize auditCount = count() by OperationName 
| sort by auditCount desc 
```
## <a name="alert-on-azure-ad-activity-log-data"></a>Définir des alertes sur les données des journaux d’activité Azure AD

Vous pouvez également définir des alertes sur votre requête. Par exemple, pour définir une alerte qui se déclenche quand plus de dix applications ont été utilisées au cours de la semaine dernière :

1. Dans l’espace de travail, sélectionnez **Définir une alerte** pour ouvrir la page **Créer une règle**.

    ![Définir une alerte](./media/howto-analyze-activity-logs-log-analytics/setalert.png)

2. Sélectionnez les **critères d’alerte** par défaut créés dans l’alerte et changez la valeur **Seuil** de la métrique par défaut à 10.

    ![Critères d’alerte](./media/howto-analyze-activity-logs-log-analytics/alertcriteria.png)

3. Entrez un nom et une description pour l’alerte, et choisissez le niveau de gravité. Dans notre exemple, nous pourrions choisir le niveau **Information**.

4. Sélectionnez le **groupe d’actions** auquel envoyer l’alerte si le signal se produit. Vous pouvez choisir d’avertir votre équipe par e-mail ou SMS, ou automatiser l’action avec des webhooks, des fonctions Azure ou des applications logiques. Découvrez plus en détail comment [créer et gérer des groupes d’alertes dans le portail Azure](../../azure-monitor/platform/action-groups.md).

5. Une fois que vous avez défini l’alerte, sélectionnez **Créer l’alerte** pour l’activer. 

## <a name="use-pre-built-workbooks-for-azure-ad-activity-logs"></a>Utilisation de classeurs prédéfinis pour les journaux d’activité Azure AD

Les classeurs fournissent plusieurs rapports liés à des scénarios courants impliquant des d’événements d’audit, de connexion et de provisionnement. Vous pouvez aussi créer des alertes sur les données figurant dans ces rapports, en effectuant les étapes décrites dans la section précédente.

* **Analyse du provisionnement** : ce [classeur](../app-provisioning/application-provisioning-log-analytics.md) affiche les rapports d’audit de l’activité de provisionnement (par exemple, le nombre de nouveaux utilisateurs attribués et les échecs d’attribution, le nombre d’utilisateurs mis à jour et les échecs de mise à jour, ainsi que le nombre d’utilisateurs dont l’attribution a été annulée et les échecs d’annulation d’attribution).    
* **Événements de connexion** : ce classeur affiche les rapports les plus pertinents relatifs à la supervision de l’activité de connexion (par exemple, les connexions par application, par utilisateur et par appareil, ainsi qu’un récapitulatif de l’évolution du nombre de connexions).
* **Insights sur l’accès conditionnel** : le [classeur](../conditional-access/howto-conditional-access-insights-reporting.md) Insights et rapports sur l’accès conditionnel vous permet de comprendre l’évolution de l’impact des stratégies d’accès conditionnel sur votre organisation. 

## <a name="next-steps"></a>Étapes suivantes

* [Bien démarrer avec les requêtes de journal d’activité Azure Monitor](../../azure-monitor/log-query/get-started-queries.md)
* [Créer et gérer des groupes d’alertes dans le portail Azure](../../azure-monitor/platform/action-groups.md)
* [Installer et utiliser les vues Log Analytics pour Azure Active Directory](howto-install-use-log-analytics-views.md)