---
title: Gestion du cycle de vie des machines virtuelles du service Azure VMware Solution
description: Découvrez comment gérer tous les aspects du cycle de vie des machines virtuelles de votre service Azure VMware Solution avec les outils natifs de Microsoft Azure.
ms.topic: conceptual
ms.date: 09/11/2020
ms.openlocfilehash: ff1032900fa936895f3adfcb0d8a872f24948aca
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326809"
---
# <a name="lifecycle-management-of-azure-vmware-solution-vms"></a>Gestion du cycle de vie des machines virtuelles du service Azure VMware Solution

Les outils natifs de Microsoft Azure vous permettent de surveiller et gérer vos machines virtuelles dans l’environnement Azure. Ils vous permettent également de surveiller et gérer vos machines virtuelles sur le service Azure VMware Solution et vos machines virtuelles locales. Cette vue d’ensemble examine les offres Azure d’architecture de surveillance intégrée et la façon dont vous pouvez utiliser ses outils natifs pour gérer les machines virtuelles de votre service Azure VMware Solution tout au long de leur cycle de vie.

## <a name="benefits"></a>Avantages

- Vous pouvez utiliser les services natifs d’Azure pour gérer vos machines virtuelles dans un environnement hybride (Azure, Azure VMware Solution et local).
- Surveillance et visibilité intégrées de vos machines virtuelles Azure, de Azure VMware Solution et locales.
- Avec Azure Update Management dans Azure Automation, vous pouvez gérer les mises à jour de système d’exploitation pour vos machines Windows et Linux. 
- Azure Security Center offre une protection avancée contre les menaces, à savoir :
    - Monitoring d’intégrité de fichier
    - Alertes de sécurité sans fichier
    - Évaluation de correctif du système d’exploitation
    - Évaluation des erreurs de configuration de la sécurité
    - Évaluation de la protection des points de terminaison 
- Déployez facilement l’agent Log Analytics à l’aide des serveurs Azure Arc prenant en charge l’extension de machine virtuelle pour les machines virtuelles nouvelles et existantes. 
- Votre espace de travail Log Analytics dans Azure Monitor active la collecte des journaux et la collecte des compteurs de performances à l’aide de l’agent ou d’extensions Log Analytics. Collectez les données et les journaux sur un point unique, et présentez-les à différents services natifs Azure. 
- Les avantages supplémentaires d’Azure Monitor sont les suivants : 
    - Surveillance transparente 
    - Meilleure visibilité de l’infrastructure 
    - Notifications instantanées 
    - Résolution automatique 
    - Rentabilité 

## <a name="integrated-azure-monitoring-architecture"></a>Architecture de surveillance Azure intégrée

Le diagramme suivant illustre l’architecture de surveillance intégrée pour les machines virtuelles du service Azure VMware Solution.

![Architecture de surveillance Azure intégrée](media/lifecycle-management-azure-vmware-solutions-virtual-machines/integrated-azure-monitoring-architecture.png)

## <a name="before-you-start"></a>Avant de commencer

Si vous débutez avec Azure ou si vous découvrez les services mentionnés précédemment, consultez les articles suivants :

- [Vue d’ensemble de l’authentification des comptes Automation](../automation/automation-security-overview.md)
- [Conception de votre déploiement de journaux Azure Monitor](../azure-monitor/platform/design-logs-deployment.md) et [Azure Monitor](../azure-monitor/overview.md)
- [Planification](../security-center/security-center-planning-and-operations-guide.md) et [Plateforme prises en charge](../security-center/security-center-os-coverage.md) pour Azure Security Center
- [Vue d’ensemble de l’activation d’Azure Monitor pour machines virtuelles](../azure-monitor/insights/vminsights-enable-overview.md)
- [Que sont les serveurs Azure Arc ?](../azure-arc/servers/overview.md) et [Que sont les Kubernetes Azure Arc ?](../azure-arc/kubernetes/overview.md)
- [Vue d’ensemble de Update Management](../automation/update-management/overview.md)

## <a name="integrating-and-deploying-azure-native-services"></a>Intégration et déploiement des services natifs d’Azure

### <a name="enable-azure-update-management"></a>Activer Azure Update Management

Le service Azure Update Management dans Azure Automation gère les mises à jour du système d’exploitation de vos ordinateurs Windows et Linux dans un environnement hybride. Il surveille la conformité des mises à jour correctives, et transfère les alertes d’écart de mise à jour corrective à Azure Monitor à des fins de correction. La solution Azure Update Management doit se connecter à votre espace de travail Log Analytics pour utiliser les données stockées afin d’évaluer l’état des mises à jour sur vos machines virtuelles.

1.  Avant de pouvoir ajouter Log Analytics à la solution Azure Update Management, vous devez commencer par [Créer un compte Azure Automation](../automation/automation-create-standalone-account.md). Si vous préférez créer votre compte à l’aide d’un modèle, consultez [Créer un compte Automation à l’aide d’un modèle Azure Resource Manager](../automation/quickstart-create-automation-account-template.md).

2. **Espace de travail Log Analytics** active la collecte des journaux et la collecte des compteurs de performances à l’aide de l’agent ou d’extensions Log Analytics. Pour créer votre espace de travail Log Analytics, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../azure-monitor/learn/quick-create-workspace.md). Si vous préférez, vous pouvez également créer un espace de travail via [interface de ligne de commande](../azure-monitor/learn/quick-create-workspace-cli.md), [PowerShell](../azure-monitor/platform/powershell-workspace-configuration.md) ou [modèle Azure Resource Manager](../azure-monitor/samples/resource-manager-workspace.md).

3. Pour activer la solution Azure Update Management pour vos machines virtuelles, consultez [Activer Update Management à partir d’un compte Automation](../automation/update-management/enable-from-automation-account.md). Dans le processus, vous allez associer votre espace de travail Log Analytics à votre compte Automation. 
 
4. Une fois que vous avez ajouté des machines virtuelles à Azure Update Management, vous pouvez [déployer des mises à jour sur des machines virtuelles et examiner les résultats](../automation/update-management/deploy-updates.md). 

### <a name="enable-azure-security-center"></a>Activation d’Azure Security Center

Le service Azure Security Center offre une protection avancée contre les menaces pesant sur vos charges de travail hybrides dans le cloud et localement. Il évalue la vulnérabilité des machines virtuelles du service Azure VMware Solution, et déclenche des alertes en fonction des besoins. Ces alertes de sécurité peuvent être transférées à Azure Monitor à des fins de résolution.

Le service Azure Security Center ne nécessite pas de déploiement. Pour plus d’informations, consultez la liste des [Fonctionnalités prises en charge pour les machines virtuelles](../security-center/security-center-services.md).

1. Pour ajouter des machines virtuelles Azure VMware Solution et des machines virtuelles non Azure à Security Center, consultez [Démarrage rapide : Configuration d’Azure Security Center](../security-center/security-center-get-started.md). 

2. Après avoir ajouté des machines virtuelles Azure VMware Solution ou des machines virtuelles provenant d’un environnement autre que Azure, activez Azure Defender dans Security Center. Security Center évalue les machines virtuelles pour détecter les éventuels problèmes de sécurité. Il affiche également des recommandations sous l’onglet Vue d’ensemble. Pour plus d’informations, consultez les [recommandations de sécurité dans Azure Security Center](../security-center/security-center-recommendations.md).

3. Vous pouvez définir des stratégies de sécurité dans Azure Security Center. Pour plus d’informations sur la configuration de vos stratégies de sécurité, consultez [Utilisation de stratégies de sécurité](../security-center/tutorial-security-policy.md).

### <a name="onboard-vms-to-azure-arc-enabled-servers"></a>Intégrer des machines virtuelles à des serveurs Azure Arc

Azure Arc étend la gestion d’Azure à toute infrastructure, notamment service Azure VMware Solution, infrastructure locale ou autre plateforme cloud.

- Consultez [Connecter des machines hybrides à Azure à grande échelle](../azure-arc/servers/onboard-service-principal.md) pour activer les serveurs Azure Arc pour plusieurs machines virtuelles Windows ou Linux.

### <a name="onboard-hybrid-kubernetes-clusters-with-arc-enabled-kubernetes"></a>Intégrer des clusters Kubernetes hybrides avec les Kubernetes Arc

Vous pouvez joindre un cluster de Kubernetes hébergé dans votre environnement Azure VMware Solution à l’aide de Kubernetes Azure Arc. 

- Consultez [Créer un principal de service d’intégration compatible avec Azure Arc](../azure-arc/kubernetes/create-onboarding-service-principal.md).

### <a name="deploy-the-log-analytics-agent"></a>Déployer l’agent Log Analytics

Vous pouvez surveiller les machines virtuelles du service Azure VMware Solution via l’agent Log Analytics (également appelé agent Microsoft Monitoring Agent (MMA) ou agent Linux OMS). Vous avez déjà créé un espace de travail Log Analytics tout en activant Azure Automation Update Management.

- Déployez l’agent Log Analytics à l’aide des [serveurs Azure Arc prenant en charge l’extension de machine virtuelle](../azure-arc/servers/manage-vm-extensions.md).

### <a name="enable-azure-monitor"></a>Activer Azure Monitor

Le service Azure Monitor est une solution complète pour la collecte, l’analyse et l’exploitation de la télémétrie de vos environnements cloud et locaux. Il ne nécessite aucun déploiement. Avec Azure Monitor, vous pouvez surveiller les performances du système d’exploitation invité, et découvrir et mapper les dépendances d’application pour Azure VMware Solution ou des machines virtuelles locales.

- Le service Azure Monitor vous permet de collecter des données à partir de différentes sources à des fins de surveillance et d’analyse. Pour plus d’informations, consultez [Sources des données de surveillance pour Azure Monitor](../azure-monitor/platform/data-sources.md).

- Collectez différents types de données à des fins d’analyse, de visualisation et de génération d’alertes. Pour plus d’informations, consultez [Plateforme de données Azure Monitor](../azure-monitor/platform/data-platform.md).

- Pour configurer Azure Monitor avec votre espace de travail Log Analytics, consultez [Configurer l’espace de travail Log Analytics pour Azure Monitor pour machines virtuelles](../azure-monitor/insights/vminsights-configure-workspace.md).

- Vous pouvez créer des règles d’alerte afin d’identifier des problèmes dans votre environnement, tels que l’utilisation intensive de ressources, des correctifs manquants, un espace disque faible et des pulsations de vos machines virtuelles. Vous pouvez également définir une réponse automatisée aux événements détectés en envoyant une alerte aux outils de gestion des services informatiques (ITSM). Une notification de détection d’alerte peut également être envoyée par e-mail. Pour créer des règles de ce type, consultez :
    - [Créer, afficher et gérer des alertes de mesure à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-metric.md).
    - [Créer, afficher et gérer des alertes de journal à l’aide d’Azure Monitor](../azure-monitor/platform/alerts-log.md).
    - [Règles d’action](../azure-monitor/platform/alerts-action-rules.md) pour définir des actions et notifications automatisées.
    - [Connecter Azure aux outils de gestion des services informatiques (ITSM) à l’aide du connecteur de gestion des services informatiques](../azure-monitor/platform/itsmc-overview.md).