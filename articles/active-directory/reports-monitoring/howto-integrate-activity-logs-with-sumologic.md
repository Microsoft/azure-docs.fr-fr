---
title: Envoyer en streaming les journaux vers SumoLogic avec Azure Monitor | Microsoft Docs
description: Découvrez comment intégrer des journaux Azure Active Directory avec SumoLogic à l’aide d’Azure Monitor.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: 2c3db9a8-50fa-475a-97d8-f31082af6593
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
ms.openlocfilehash: 51e1f45c787c319c32358e7f310108131647d60e
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91335831"
---
# <a name="integrate-azure-active-directory-logs-with-sumologic-using-azure-monitor"></a>Intégrer des journaux d’activité Azure Active Directory avec SumoLogic à l’aide d’Azure Monitor

Cet article explique comment intégrer des journaux d’activité Azure Active Directory (Azure AD) avec SumoLogic à l’aide d’Azure Monitor. Vous commencez pas router les journaux d’activité vers un hub d’événements Azure, puis vous intégrez ce hub d’événements avec SumoLogic.

## <a name="prerequisites"></a>Prérequis

Pour utiliser cette fonctionnalité, vous avez besoin des éléments suivants :
* Un hub d’événements Azure contenant les journaux d’activité d’Azure AD. Découvrez comment [diffuser en continu vos journaux d’activité sur un hub d’événements](./tutorial-azure-monitor-stream-logs-to-event-hub.md). 
* Un abonnement avec l’authentification unique activée à SumoLogic.

## <a name="steps-to-integrate-azure-ad-logs-with-sumologic"></a>Étapes permettant d’intégrer des journaux d’activité Azure AD avec SumoLogic 

1. Commencez par [diffuser en continu des journaux d’activité Azure AD sur un hub d’événements Azure](./tutorial-azure-monitor-stream-logs-to-event-hub.md).
2. Configurez votre instance SumoLogic pour [collecter des journaux d’activité pour Azure Active Directory](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Collect_Logs_for_Azure_Active_Directory).
3. [Installez l’application Azure AD SumoLogic](https://help.sumologic.com/Send-Data/Applications-and-Other-Data-Sources/Azure_Active_Directory/Install_the_Azure_Active_Directory_App_and_View_the_Dashboards) pour utiliser les tableaux de bord préconfigurés qui fournissent une analyse en temps réel de votre environnement.

   ![tableau de bord](./media/howto-integrate-activity-logs-with-sumologic/overview-dashboard.png)

## <a name="next-steps"></a>Étapes suivantes

* [Interpréter le schéma des journaux d’audit dans Azure Monitor](reference-azure-monitor-audit-log-schema.md)
* [Interpréter le schéma des journaux d’activité de connexion dans Azure Monitor](reference-azure-monitor-sign-ins-log-schema.md)
* [Questions fréquentes et problèmes connus](concept-activity-logs-azure-monitor.md#frequently-asked-questions)