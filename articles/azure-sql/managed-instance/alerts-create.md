---
title: Configurer des alertes et des notifications pour Managed Instance (portail Azure)
description: Utilisez le portail Azure pour créer des alertes SQL Managed Instance permettant de déclencher des notifications ou Automation lorsque les conditions spécifiées sont remplies.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: performance
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: danimir
ms.author: danil
ms.reviewer: wiassaf, sstein
ms.date: 05/04/2020
ms.openlocfilehash: a85bf6510163c47fa39bdb28debff806e1812f58
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499269"
---
# <a name="create-alerts-for-azure-sql-managed-instance-using-the-azure-portal"></a>Créer des alertes pour Azure SQL Managed Instance à l’aide du portail Azure
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Cet article explique comment configurer des alertes pour les bases de données dans Azure SQL Database Managed Instance à l’aide du portail Azure. Les alertes peuvent vous envoyer un e-mail, appeler un webhook, exécuter une fonction Azure Functions, un runbook, appeler un système de tickets externe compatible ITSM, vous appeler sur le téléphone ou envoyer un SMS quand une certaine mesure (par exemple la taille de stockage de l’instance ou l’utilisation de l’UC) atteint un seuil prédéfini. Il présente également les meilleures pratiques à adopter pour définir les périodes d’alerte.


## <a name="overview"></a>Vue d’ensemble

Vous pouvez recevoir une alerte en fonction de métriques de surveillance pour vos services Azure ou d'événements sur ces derniers.

* **Valeurs de métriques** : l’alerte se déclenche lorsque la valeur d’une métrique spécifiée dépasse un seuil que vous affectez dans un des deux sens. C’est-à-dire que le déclenchement se fait à la fois lorsque la condition est remplie et par la suite une fois que la condition n’est plus remplie.

Vous pouvez configurer une alerte pour effectuer les opérations suivantes lors de son déclenchement :

* Envoyer des notifications par e-mail à l’administrateur de service et aux coadministrateurs
* Envoyer un e-mail à d’autres adresses que vous spécifiez
* Appeler un numéro de téléphone avec l’invite vocale
* Envoyer un SMS à un numéro de téléphone
* Appeler un webhook
* Appeler une fonction Azure Functions
* Appeler un runbook Azure
* Appeler un système de tickets externe compatible ITSM

Vous pouvez configurer et obtenir des informations sur les règles d'alerte à l'aide [du portail Azure, de PowerShell ou d'Azure CLI](../../azure-monitor/platform/alerts-classic-portal.md) ou de l'[API REST Azure Monitor](/rest/api/monitor/alertrules). 

## <a name="alerting-metrics-available-for-managed-instance"></a>Métriques d’alerte disponibles pour l’instance gérée

> [!IMPORTANT]
> Les métriques d’alerte sont disponibles pour l’instance gérée uniquement. Les métriques d’alerte pour les bases de données individuelles dans l’instance gérée ne sont pas disponibles. La télémétrie des diagnostics de base de données est en revanche disponible sous la forme de [journaux de diagnostic](../database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md#diagnostic-telemetry-for-export). Les alertes sur les journaux de diagnostic peuvent être configurées à partir du produit [SQL Analytics](../../azure-monitor/insights/azure-sql.md) à l’aide de [scripts d’alerte de journal](../../azure-monitor/insights/azure-sql.md#creating-alerts-for-sql-managed-instance) pour l’instance gérée.

Les métriques d’instance gérée suivantes sont disponibles pour la configuration des alertes :

| Métrique | Description | Unités de mesure/valeurs possibles |
| :--------- | --------------------- | ----------- |
| Pourcentage d’UC moyenne | Pourcentage moyen d’utilisation de l’UC dans la période sélectionnée. | 0-100 (pour cent) |
| Octets d’E/S lus | Octets d’E/S lus dans la période sélectionnée. | Octets |
| Octets d’E/S écrits | Octets d’E/S écrits dans la période sélectionnée. | Octets |
| Nombre de requêtes d’E/S | Nombre de requêtes d’E/S dans la période sélectionnée. | Numérique |
| Espace de stockage réservé | Espace de stockage maximum actuel réservé pour l’instance gérée. Une modifications a lieu avec l’opération de mise à l’échelle de la ressource. | Mo (mégaoctets) |
| Espace de stockage utilisé | Espace de stockage utilisé dans la période sélectionnée. Une modifications a lieu avec la consommation de stockage par bases de données et l’instance. | Mo (mégaoctets) |
| Nombre de cœurs virtuels | vCores provisionnés pour l’instance gérée. Une modifications a lieu avec l’opération de mise à l’échelle de la ressource. | 4-80 (vCores) |

## <a name="create-an-alert-rule-on-a-metric-with-the-azure-portal"></a>Créer une règle d’alerte sur une métrique avec le Portail Azure

1. Sur le [portail](https://portal.azure.com/) Azure, localisez l'instance gérée que vous souhaitez surveiller et sélectionnez-la.

2. Sélectionnez l’élément de menu **Métriques** à partir de la section Surveillance.

   ![Surveillance](./media/alerts-create/mi-alerting-menu-annotated.png)
  
3. Dans le menu déroulant, sélectionnez l’une des métriques pour lesquelles vous souhaitez configurer votre alerte (l’espace de stockage utilisé est indiqué dans l’exemple).

4. Sélectionnez la période d’agrégation (moyenne, minimum ou maximum) atteinte dans la période indiquée (moy, min ou max). 

5. Sélectionnez **Nouvelle règle d’alerte**

6. Dans le volet Créer une règle d’alerte, cliquez sur **Nom de la condition** (l’espace de stockage utilisé est indiqué dans l’exemple).

   ![Définir la condition](./media/alerts-create/mi-create-metrics-alert-smaller-annotated.png)

7. Dans le volet Configurer la logique du signal, définissez l’opérateur, le type d’agrégation et la valeur de seuil.

   * Les options de type d’opérateur sont Supérieur à , Égal à et Inférieur à (la valeur de seuil).
   * Les options de type d’agrégation sont Minimal, Maximal ou Moyen (dans la période de granularité d’agrégation)
   * La valeur de seuil est la valeur d’alerte qui sera évaluée en fonction de l’opérateur et des critères d’agrégation
   
   ![Configure_signal_logic](./media/alerts-create/mi-configure-signal-logic-annotated.png)
   
   Dans l’exemple présenté dans la capture d’écran, la valeur de 1 840 876 Mo est utilisée pour représenter une valeur de seuil de 1,8 To. Comme l’opérateur dans l’exemple est défini sur Supérieur à, l’alerte est créée si la consommation d’espace de stockage sur l’instance gérée dépasse 1,8 To. Notez que la valeur de seuil pour les métriques de l’espace de stockage doit être exprimée en Mo.

8. Définissez la période d’évaluation - précision d’agrégation en minutes et la fréquence d’évaluation. La fréquence d’évaluation indique l’heure à laquelle le système d’alerte vérifie périodiquement si la condition de seuil est remplie.

9. Sélectionnez un groupe d’actions. Le volet du groupe d’actions s’affiche pour vous permettre de sélectionner une action ou d’en créer une. Cette action définit ce qui se produit lors du déclenchement d’une alerte (par exemple, l’envoi d’un e-mail, un appel sur le téléphone, l’exécution d’un webhook, d’une fonction Azure ou d’un runbook).

   ![Select_action_group](./media/alerts-create/mi-select-action-group-smaller-annotated.png)

   * Pour créer un groupe d’actions, sélectionnez **+ Créer un groupe d’actions**.

      ![Create_action_group_alerts](./media/alerts-create/mi-create-alert-action-group-smaller-annotated.png)
   
   * Définissez comment recevoir l’alerte : entrez le nom du groupe d’actions, le nom abrégé, le nom de l’action et sélectionnez le type d’action. Le type d’action définit si vous êtes averti par e-mail, SMS, appel vocal ou, le cas échéant, si le webhook, la fonction Azure, le runbook est exécuté ou si le ticket ITSM est créé dans votre système compatible.

      ![Define_how_to_be_alerted](./media/alerts-create/mi-add-alerts-action-group-annotated.png)

10. Renseignez les détails de la règle d’alerte pour vos archives, puis sélectionnez le type de gravité.

      ![Rule_description](./media/alerts-create/mi-rule-details-complete-smaller-annotated.png)

   * Terminez la création de la règle d’alerte en cliquant sur le bouton **Créer une règle d’alerte**.

La nouvelle règle d’alerte est active en quelques minutes et est déclenchée en fonction de vos paramètres.

## <a name="verifying-alerts"></a>Vérification des alertes

> [!NOTE]
> Pour supprimer les alertes inutiles, consultez [Suppression des alertes à l’aide de règles d’action](../../azure-monitor/platform/alerts-action-rules.md#suppression-of-alerts).

Lors de la configuration d’une règle d’alerte, vérifiez que vous êtes satisfait du déclencheur d’alerte et de sa fréquence. Dans l’exemple indiqué sur cette page pour la configuration d’une alerte sur l’espace de stockage utilisé, si votre option d’alerte est E-mail, vous pouvez recevoir un e-mail comme celui indiqué ci-dessous.

   ![alert_example](./media/alerts-create/mi-email-alert-example-smaller-annotated.png)

L’e-mail indique le nom de l’alerte, les détails du seuil et la raison pour laquelle l’alerte a été déclenchée pour vous aider à vérifier et à résoudre les problèmes de votre alerte. Vous pouvez utiliser le bouton **Voir dans le portail Azure** pour afficher l’alerte reçue par e-mail dans le portail Azure. 

## <a name="view-suspend-activate-modify-and-delete-existing-alert-rules"></a>Afficher, suspendre, activer, modifier et supprimer des règles d’alerte existantes

> [!NOTE]
> Les alertes existantes doivent être gérées à partir du menu Alertes du tableau de bord du portail Azure. Les alertes existantes ne peuvent pas être modifiées à partir du panneau de ressource de l’instance gérée.

Pour afficher, suspendre, activer, modifier et supprimer des alertes existantes :

1. Recherchez des alertes à l’aide de la recherche dans le portail Azure. Cliquez sur Alertes.

   ![find_alerts](./media/alerts-create/mi-manage-alerts-browse-smaller-annotated.png)

   Vous pouvez également cliquer sur Alertes dans la barre de navigation Azure, si vous l’avez configurée.

2. Dans le volet Alertes, sélectionnez Gérer les règles d’alerte.

   ![modify_alerts](./media/alerts-create/mi-manage-alert-rules-smaller-annotated.png)

   La liste des alertes existantes s’affiche. Sélectionnez une règle d’alerte existante pour la gérer. Les règles actives existantes peuvent être modifiées et ajustées à votre convenance. Les règles actives peuvent également être suspendues sans être supprimées. 

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur le système d’alertes Azure Monitor, consultez [Vue d’ensemble des alertes dans Microsoft Azure](../../azure-monitor/platform/alerts-overview.md).
* Pour en savoir plus sur les alertes de métrique, consultez [Comprendre le fonctionnement des alertes de métrique dans Azure Monitor](../../azure-monitor/platform/alerts-metric-overview.md).
* Pour en savoir plus sur la configuration d’un webhook dans des alertes, consultez [Appeler un webhook avec une alerte de métrique classique](../../azure-monitor/platform/alerts-webhooks.md)
* Pour en savoir plus sur la configuration et la gestion des alertes à l’aide de PowerShell, consultez [Règles d’action](/powershell/module/az.monitor/add-azmetricalertrulev2)
* Pour en savoir plus sur la configuration et la gestion des alertes à l’aide de l’API, consultez [Informations de référence sur l’API REST Azure Monitor](/rest/api/monitor/)