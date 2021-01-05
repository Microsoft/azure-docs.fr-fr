---
title: Azure AD Connect Health - Alerte indiquant que les données du Service de contrôle d’intégrité ne sont pas mises à jour | Microsoft Docs
description: Ce document décrit la cause de l’alerte indiquant que les données du Service de contrôle d’intégrité ne sont pas mises à jour et explique comment y remédier.
services: active-directory
documentationcenter: ''
author: zhiweiwangmsft
manager: SamuelD
editor: ''
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 02/26/2018
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 63d1d87d9b576a8e181b5b339052a6b6512f18a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85359226"
---
# <a name="health-service-data-is-not-up-to-date-alert"></a>Les données du Service de contrôle d’intégrité ne sont pas mises à jour

## <a name="overview"></a>Vue d’ensemble

Les agents sur les machines locales supervisées par Azure AD Connect Health chargent périodiquement des données vers le service Azure AD Connect Health. Si le service ne reçoit pas de données d’un agent, les informations que le portail présente sont obsolètes. Pour mettre en évidence le problème, le service déclenche l’alerte **Les données du service de contrôle d’intégrité ne sont pas à jour**. Cette alerte est générée lorsque le service n’a pas reçu de données complètes au cours des deux dernières heures.  

- L’alerte d’état **Avertissement** se déclenche si le service de contrôle d’intégrité n’a reçu que des types de données **partiels** du serveur au cours des deux dernières heures. L’alerte d’état d’avertissement ne déclenche pas l’envoi de notifications par e-mail aux destinataires configurés. 
- L’alerte d’état **Erreur** se déclenche si le service de contrôle d’intégrité n’a reçu aucun type de données du serveur au cours des deux dernières heures. L’alerte d’état d’erreur déclenche l’envoi de notifications par e-mail aux destinataires configurés.

Le service obtient les données d’agents qui s’exécutent sur les ordinateurs locaux, selon le type de service. Le tableau suivant répertorie les agents qui s’exécutent sur l’ordinateur, ce qu’ils font et les types de données que le service génère. Dans certains cas, plusieurs services sont impliqués dans le processus. Chacun d’eux peut alors être la cause du problème. 

## <a name="understanding-the-alert"></a>Compréhension de l’alerte

Le panneau **Détails de l’alerte** indique quand l’alerte s’est produite et a été détectée pour la dernière détection. Un processus en arrière-plan qui s’exécute toutes les deux heures génère et réévalue l’alerte. Dans l’exemple suivant, l’alerte initiale s’est produite le 10/03 à 9:59. L’alerte existait toujours le 12/03 à 10:00 quand elle a été réévaluée. Le panneau indique également l’heure à laquelle le service de contrôle d’intégrité a reçu pour la dernière fois un type de données particulier. 
 
 ![Détails d’alerte Azure Active Connect Health](./media/how-to-connect-health-data-freshness/data-freshness-details.png)
 
Le tableau suivant mappe les types de services aux types de données requises correspondants :

| Type de service | Agent (nom du service Windows) | Objectif | Type de données généré  |
| --- | --- | --- | --- |  
| Azure AD Connect (Sync) | Azure AD Connect Health Sync Insights Service | Collecter des informations spécifiques d’AAD Connect (connecteurs, règles de synchronisation, etc.) | - AadSyncService-SynchronizationRules <br />  - AadSyncService-Connectors <br /> - AadSyncService-GlobalConfigurations  <br />  - AadSyncService-RunProfileResults <br /> - AadSyncService-ServiceConfigurations <br /> - AadSyncService-ServiceStatus   |
|  | Azure AD Connect Health Sync Monitoring Service | Collecter les compteurs de performances, traces ETW et fichiers spécifiques d’AAD Connect | Compteur de performances |
| AD DS | Azure AD Connect Health AD DS Insights Service | Effectuer des tests synthétiques, collecter des informations sur la topologie, métadonnées de réplication |  - Adds-TopologyInfo-Json <br /> - Common-TestData-Json (crée les résultats de test)   | 
|  | Azure AD Connect Health AD DS Monitoring Service | Collecter les compteurs de performances, traces ETW et fichiers spécifiques d’ADDS | - Compteur de performances  <br /> - Common-TestData-Json (charge les résultats de tests)  |
| AD FS | Azure AD Connect Health AD FS Diagnostics Service | Effectuer des tests synthétiques | TestResult (crée les résultats de test) | 
| | Azure AD Connect Health AD FS Insights Service  | Collecter les métriques d’utilisation AD FS | Adfs-UsageMetrics |
| | Azure AD Connect Health AD FS Monitoring Service | Collecter les compteurs de performances, traces ETW et fichiers spécifiques d’AD FS | TestResult (charge les résultats de test) |

## <a name="troubleshooting-steps"></a>Étapes de dépannage 

Les étapes requises pour diagnostiquer le problème sont décrites ci-dessous. La première est un ensemble de vérifications de base communes à tous les types de services. 

> [!IMPORTANT] 
> Cette alerte suit la [stratégie de conservation des données](reference-connect-health-user-privacy.md#data-retention-policy) de Connect Health.

* Vérifiez que les dernières versions des agents sont installées. Voir l’[Historique des mises en production](reference-connect-health-version-history.md). 
* Assurez-vous que les services des agents Azure AD Connect Health sont **en cours d’exécution** sur l’ordinateur. Par exemple, Connect Health pour AD FS doit avoir trois services.
  ![Vérifier Azure AD Connect Health](./media/how-to-connect-health-agent-install/install5.png)

* N’oubliez pas de prendre connaissance et de respecter la [section Configuration requise](how-to-connect-health-agent-install.md#requirements).
* Utilisez l’[outil de test de connectivité](how-to-connect-health-agent-install.md#test-connectivity-to-azure-ad-connect-health-service) pour découvrir les problèmes de connectivité.
* Si vous disposez d'un proxy HTTP, suivez cette [procédure de configuration](how-to-connect-health-agent-install.md#configure-azure-ad-connect-health-agents-to-use-http-proxy). 


## <a name="next-steps"></a>Étapes suivantes
Si l’une des étapes ci-dessus a détecté un problème, corrigez-le et attendez que l’alerte soit résolue. Le processus en arrière-plan d’alerte s’exécutant toutes les 2 heures, la résolution de l’alerte peut prendre jusqu’à 2 heures. 

* [Stratégie de conservation de données d’Azure AD Connect Health](reference-connect-health-user-privacy.md#data-retention-policy)
* [Forum Aux Questions (FAQ) Azure AD Connect Health](reference-connect-health-faq.md)
