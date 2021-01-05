---
title: Journal de sécurité et audit Azure | Microsoft Docs
description: Découvrez les journaux disponibles dans Azure et les insights de sécurité que vous pouvez obtenir.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: TomSh
ms.assetid: ''
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/31/2019
ms.author: terrylan
ms.openlocfilehash: 79c15f2df82125f20dcfaf9992d9047b632c253d
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94412797"
---
# <a name="azure-security-logging-and-auditing"></a>Journal de sécurité et audit Azure

Azure offre un large éventail d’options de journalisation et d’audit de sécurité configurables pour vous aider à identifier les failles dans vos mécanismes et stratégies de sécurité. Cet article décrit la génération, la collecte et l’analyse des journaux d’activité de sécurité des services hébergés sur Azure.

> [!Note]
> Certaines recommandations contenues dans cet article risquent d’entraîner une augmentation de l’utilisation des données, des réseaux ou des ressources de calcul, et donc une augmentation des coûts de licence ou d’abonnement.

## <a name="types-of-logs-in-azure"></a>Types de journaux d’activité dans Azure

Les applications cloud sont complexes, et se composent de nombreux éléments mobiles. L’enregistrement des données peuvent fournir des Insights sur vos applications et vous aider à :

- détecter des problèmes antérieurs et éviter des problèmes futurs
- améliorer les performances et la maintenabilité des applications
- automatiser des actions qui nécessitent autrement une intervention manuelle

Les journaux d’activité Azure sont classés par type :
* Les **journaux d’activité de contrôle/gestion** fournissent des informations sur les opérations CREATE, UPDATE, and DELETE Azure Resource Manager. Pour plus d’informations, consultez [Journaux d’activité des activités Azure](../../azure-monitor/platform/platform-logs-overview.md).

* Les **journaux de plan de données** fournissent des informations sur les événements déclenchés lors de l’utilisation des ressources Azure. Il s’agit par exemple des journaux d’activité du système d’événements, de la sécurité et des applications Windows dans une machine virtuelle, et des [journaux de diagnostic](../../azure-monitor/platform/platform-logs-overview.md) qui sont configurés via Azure Monitor.

* Les **événements traités** fournissent des informations sur les événements/alertes analysés en votre nom. Les alertes [Azure Security Center](../../security-center/security-center-managing-and-responding-alerts.md) en sont un exemple, où [Azure Security Center](../../security-center/security-center-introduction.md) a traité et analysé votre abonnement et fournit des alertes de sécurité très concises.

Le tableau suivant liste les principaux types de journaux d’activité disponibles dans Azure :

| Catégorie de journal | Type de journal | Usage | Intégration |
| ------------ | -------- | ------ | ----------- |
|[Journaux d’activité](../../azure-monitor/platform/platform-logs-overview.md)|Événements de plan de contrôle sur les ressources d’Azure Resource Manager|  Fournissent des informations sur les opérations qui ont été effectuées sur les ressources de votre abonnement.|    API Rest, [Azure Monitor](../../azure-monitor/platform/platform-logs-overview.md)|
|[Journaux de ressources Azure](../../azure-monitor/platform/platform-logs-overview.md)|Données fréquentes sur les opérations des ressources Azure Resource Manager de l’abonnement|   Fournissent des insights sur les opérations que votre ressource réalise elle-même.| Azure Monitor|
|[Compte-rendu Azure Active Directory](../../active-directory/reports-monitoring/overview-reports.md)|Journaux d’activité et rapports | Signalent les activités de connexion des utilisateurs et fournissent des informations sur l’activité système relative à la gestion des utilisateurs et des groupes.|[API Graph](../../active-directory/develop/microsoft-graph-intro.md)|
|[Machines virtuelles et services cloud](../../azure-monitor/learn/quick-collect-azurevm.md)|Service du journal des événements Windows et Syslog Linux|  Capture les données système et les données de journalisation sur les machines virtuelles, puis les transfère vers un compte de stockage de votre choix.|   Windows (avec le stockage Diagnostics Azure pour Windows [[WAD](../../azure-monitor/platform/diagnostics-extension-overview.md)]) et Linux dans Azure Monitor|
|[Azure Storage Analytics](/rest/api/storageservices/fileservices/storage-analytics)|Journalisation du stockage, fournit les données de métriques d’un compte de stockage|Fournit des informations sur les demandes de trace, analyse les tendances d’utilisation et diagnostique les problèmes de votre compte de stockage.| API REST ou [bibliothèque cliente](/dotnet/api/overview/azure/storage)|
|[Journaux de flux du groupe de sécurité réseau (NSG)](../../network-watcher/network-watcher-nsg-flow-logging-overview.md)|Format JSON, affiche les flux entrants et sortants, par règle|Affiche des informations sur le trafic IP entrant et sortant via un groupe de sécurité réseau.|[Azure Network Watcher](../../network-watcher/network-watcher-monitoring-overview.md)|
|[Application Insights](../../azure-monitor/app/app-insights-overview.md)|Journaux d’activité, exceptions et diagnostics personnalisés|  Fournit un service de monitoring des performances de l’application (APM) aux développeurs web sur de nombreuses plateformes.| API REST, [Power BI](https://powerbi.microsoft.com/documentation/powerbi-azure-and-power-bi/)|
|[Traitement des données/alertes de sécurité](../../security-center/security-center-introduction.md)|   Alertes Azure Security Center, alertes de journaux Azure Monitor|    Fournit des alertes et des informations sur la sécurité.|  API REST, JSON|

## <a name="log-integration-with-on-premises-siem-systems"></a>Intégration des journaux avec les systèmes SIEM locaux
[Intégration des alertes Security Center](../../security-center/security-center-partner-integration.md) indique comment synchroniser des alertes Security Center, les événements de sécurité des machines virtuelles collectés par les journaux de diagnostic Azure et les journaux d’audit Azure avec votre solution SIEM ou vos journaux d’activité Azure Monitor.

## <a name="next-steps"></a>Étapes suivantes

- [Audit et journalisation](management-monitoring-overview.md) : protégez les données en conservant le niveau de visibilité et en répondant rapidement aux alertes de sécurité reçues.

- [Collecte des journaux de sécurité et d’audit dans Azure](https://azure.microsoft.com/resources/videos/security-logging-and-audit-log-collection/) : appliquez ces paramètres pour vous assurer que vos instances Azure collectent les journaux d’audit et de sécurité appropriés.

- [Configurer les paramètres d’audit pour une collection de sites](https://support.office.com/article/Configure-audit-settings-for-a-site-collection-A9920C97-38C0-44F2-8BCB-4CF1E2AE22D2?ui=&rs=&ad=US) : si vous administrez une collection de sites, récupérez l’historique des actions de chaque utilisateur et l’historique des actions réalisées pendant une période donnée.

- [Recherchez le journal d’audit dans le Centre de sécurité Microsoft 365](/microsoft-365/compliance/search-the-audit-log-in-security-and-compliance) : utilisez le Centre de sécurité Microsoft 365 pour effectuer des recherches dans le journal d’audit unifié et afficher les activités utilisateur et administrateur effectuées dans votre organisation.