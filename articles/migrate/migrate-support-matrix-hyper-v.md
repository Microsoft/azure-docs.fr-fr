---
title: Prise en charge de l’évaluation Hyper-V dans Azure Migrate
description: Découvrez la prise en charge pour l’évaluation de machines virtuelles Hyper-V à l’aide de l’outil Évaluation de serveur d’Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 06/14/2020
ms.openlocfilehash: 5b5c85b599f02cedc3bb1bda84c28ef2169c8e2d
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754043"
---
# <a name="support-matrix-for-hyper-v-assessment"></a>Tableau de prise en charge pour l’évaluation Hyper-V

Cet article résume les conditions préalables et les exigences de prise en charge lors de l’évaluation de machines virtuelles Hyper-V pour la migration vers Azure et utilise l’outil [Azure Migrate : Évaluation de serveur](migrate-services-overview.md#azure-migrate-server-assessment-tool). Si vous souhaitez migrer des machines virtuelles Hyper-V vers Azure, passez en revue la [matrice de prise en charge de la migration](migrate-support-matrix-hyper-v-migration.md).

Pour configurer l’évaluation d’une machine virtuelle Hyper-V, vous créez un projet Azure Migrate et ajoutez l’outil Évaluation de serveur au projet. Une fois l’outil ajouté, vous déployez l’[appliance Azure Migrate](migrate-appliance.md). Celle-ci découvre en continu les machines locales et envoie leurs métadonnées et leurs données de performances à Azure. Une fois la découverte terminée, vous rassemblez les machines découvertes dans des groupes et effectuez l’évaluation d’un groupe.


## <a name="limitations"></a>Limites

**Support** | **Détails**
--- | ---
**Limites d’évaluation** | Vous pouvez découvrir et évaluer jusqu’à 35 000 machines virtuelles Hyper-V dans un même [projet Azure Migrate](migrate-support-matrix.md#azure-migrate-projects).
**Limites de projet** | Vous pouvez créer plusieurs projets dans un abonnement Azure. En plus des machines virtuelles Hyper-V, un projet peut inclure des machines virtuelles VMware et des serveurs physiques, jusqu’aux limites d’évaluation pour chaque catégorie.
**Découverte** | L’appliance Azure Migrate peut découvrir jusqu’à 5 000 machines virtuelles Hyper-V.<br/><br/> L’appliance peut se connecter à jusqu’à 300 hôtes Hyper-V.
**Évaluation** | Vous pouvez ajouter jusqu’à 35 000 machines dans un groupe unique.<br/><br/> Vous pouvez évaluer jusqu’à 35 000 machines virtuelles par évaluation pour un groupe.

[Apprenez-en davantage](concepts-assessment-calculation.md) sur les évaluations.



## <a name="hyper-v-host-requirements"></a>Configuration requise pour l’hôte Hyper-V

| **Support**                | **Détails**               
| :-------------------       | :------------------- |
| **Hôte Hyper-V**       | L’hôte Hyper-V peut être autonome ou déployé dans un cluster.<br/><br/> L’hôte Hyper-V peut exécuter Windows Server 2019, Windows Server 2016 ou Windows Server 2012 R2. L’installation Server Core de ces systèmes d’exploitation est également prise en charge. <br/>Vous ne pouvez pas évaluer des machines virtuelles situées sur des hôtes Hyper-V exécutant Windows Server 2012.
| **autorisations**           | Vous avez besoin des privilèges Administrateur sur l’hôte Hyper-V. <br/> Si vous ne souhaitez pas attribuer de privilèges Administrateur, créez un compte d’utilisateur local ou de domaine et ajoutez le compte d’utilisateur à ces groupes : Utilisateurs de gestion à distance, Administrateurs Hyper-V et Utilisateurs de l’Analyseur de performances. |
| **Communication à distance PowerShell**   | La [communication à distance PowerShell](/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7) doit être activée sur chaque hôte Hyper-V. |
| **Réplica Hyper-V**       | Si vous utilisez un réplica Hyper-V (ou si vous avez plusieurs machines virtuelles avec les mêmes identificateurs de machines virtuelles) et que vous découvrez les machines virtuelles originales et répliquées à l’aide d’Azure Migrate, l’évaluation générée par Azure Migrate pourrait ne pas être exacte. |


## <a name="vm-requirements"></a>Configuration requise pour les machines virtuelles

| **Support**                  | **Détails**               
| :----------------------------- | :------------------- |
| **Système d’exploitation** | Tous les systèmes d’exploitation peuvent être évalués dans une optique de migration.  |
| **Services d’intégration**       | Les [services d'intégration Hyper-V](/virtualization/hyper-v-on-windows/reference/integration-services) doivent fonctionner sur les machines virtuelles que vous évaluez afin de capturer les informations du système d'exploitation. |
| **Stockage** | Disque local, DAS, JBOD, espaces de stockage, CSV, SMB. Ces stockages d’hôte Hyper-V sur lesquels sont stockés les disques VHD/VHDX sont pris en charge. <br/> Les contrôleurs virtuels IDE et SCSI sont pris en charge| 

## <a name="azure-migrate-appliance-requirements"></a>Conditions requises de l’appliance Azure Migrate

Azure Migrate utilise l’[appliance Azure Migrate](migrate-appliance.md) pour la découverte et l’évaluation. Vous pouvez déployer l’appliance à l’aide d’un disque dur virtuel Hyper-V compressé que vous téléchargez à partir du portail, ou à l’aide d’un [script PowerShell](deploy-appliance-script.md).

- En savoir plus sur les [conditions requises de l’appliance](migrate-appliance.md#appliance---hyper-v) pour Hyper-V.
- Découvrez les URL auxquelles l’appliance doit accéder dans les clouds [publics](migrate-appliance.md#public-cloud-urls) et du [secteur public](migrate-appliance.md#government-cloud-urls).
- Dans Azure Government, vous devez déployer l’appliance [avec ce script](deploy-appliance-script-government.md).

## <a name="port-access"></a>Accès au port

Le tableau suivant résume les exigences du port pour l’évaluation.

**Appareil** | **Connection**
--- | ---
**Appliance** | Connexions entrantes sur le port TCP 3389 pour permettre des connexions Bureau à distance avec l’appliance.<br/><br/> Connexions entrantes sur le port 44368 pour accéder à distance à l’application de gestion de l’appliance via l’URL : ``` https://<appliance-ip-or-name>:44368 ```<br/><br/> Connexions sortantes sur les ports 443 (HTTPS) pour envoyer les métadonnées de découverte et de performances à Azure Migrate.
**Hôte/cluster Hyper-V** | Connexions entrantes sur le port WinRM 5985 (HTTP) ou 5986 (HTTPS) pour extraire les métadonnées et les données de performances des machines virtuelles Hyper-V en utilisant une session Common Information Model (CIM).

## <a name="agent-based-dependency-analysis-requirements"></a>Conditions requises de l’analyse des dépendances basées sur un agent

L’[analyse des dépendances](concepts-dependency-visualization.md) vous permet d’identifier les dépendances entre les machines locales que vous souhaitez évaluer et faire migrer vers Azure. Le tableau récapitule les conditions requises pour la configuration de l’analyse des dépendances basées sur un agent. Actuellement, Hyper-V prend uniquement en charge la visualisation des dépendances basées sur un agent. 

**Prérequis** | **Détails** 
--- | --- 
**Avant le déploiement** | Vous devez disposer d’un projet Azure Migrate, avec l’outil Évaluation de serveur ajouté au projet.<br/><br/>  Vous déployez la visualisation des dépendances après avoir configuré une appliance Azure Migrate pour découvrir vos machines locales.<br/><br/> [Découvrez comment](create-manage-projects.md) créer un projet pour la première fois.<br/> [Découvrez comment](how-to-assess.md) ajouter un outil d’évaluation à un projet existant.<br/> Découvrez comment configurer l’appliance Azure Migrate pour l’évaluation des [machines virtuelles Hyper-V](how-to-set-up-appliance-hyper-v.md).
**Azure Government** | La visualisation des dépendances n’est pas disponible dans Azure Government.
**Log Analytics** | Azure Migrate utilise la solution [Service Map](../azure-monitor/insights/service-map.md) dans [Journaux d’activité Azure Monitor](../azure-monitor/log-query/log-query-overview.md) pour la visualisation des dépendances.<br/><br/> Vous associez un espace de travail Log Analytics nouveau ou déjà existant à un projet Azure Migrate. L’espace de travail d’un projet Azure Migrate ne peut pas être modifié une fois qu’il a été ajouté. <br/><br/> L’espace de travail doit se trouver dans le même abonnement que le projet Azure Migrate.<br/><br/> L’espace de travail doit résider dans les régions USA Est, Asie Sud-Est ou Europe Ouest. Les espaces de travail des autres régions ne peuvent pas être associés à un projet.<br/><br/> L’espace de travail doit se trouver au sein d’une région dans laquelle [Service Map est pris en charge](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br/><br/> Dans Log Analytics, l’espace de travail associé à Azure Migrate est marqué avec la clé de projet de migration et le nom du projet.
**Agents nécessaires** | Sur chaque machine à analyser, installez les agents suivants :<br/><br/> [MMA (Microsoft Monitoring Agent)](../azure-monitor/platform/agent-windows.md)<br/> [Dependency Agent](../azure-monitor/platform/agents-overview.md#dependency-agent)<br/><br/> Si les machines locales ne sont pas connectées à Internet, vous devez télécharger et installer la passerelle de l’espace de travail Log Analytics sur celles-ci.<br/><br/> En savoir plus sur l’installation de [Dependency Agent](how-to-create-group-machine-dependencies.md#install-the-dependency-agent) et de [MMA](how-to-create-group-machine-dependencies.md#install-the-mma).
**Espace de travail Log Analytics** | L’espace de travail doit se trouver dans le même abonnement que le projet Azure Migrate.<br/><br/> Azure Migrate prend en charge les espaces de travail résidant dans les régions USA Est, Asie Sud-Est et Europe Ouest.<br/><br/>  L’espace de travail doit se trouver au sein d’une région dans laquelle [Service Map est pris en charge](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br/><br/> L’espace de travail d’un projet Azure Migrate ne peut pas être modifié une fois qu’il a été ajouté.
**Coûts** | La solution Service Map n’entraîne aucun frais pendant les 180 premiers jours (à compter du jour où vous associez l’espace de travail Log Analytics au projet Azure Migrate).<br/><br/> Au bout de 180 jours, des frais Log Analytics standard s’appliquent.<br/><br/> L’utilisation d’une autre solution que Service Map dans l’espace de travail Log Analytics associé entraîne des [frais standard](https://azure.microsoft.com/pricing/details/log-analytics/) pour l’espace de travail Log Analytics.<br/><br/> Lorsque le projet Azure Migrate est supprimé, l’espace de travail ne l’est pas. Une fois le projet supprimé, l’utilisation de Service Map n’est plus gratuite, et chaque nœud est facturé en fonction du niveau payant de l’espace de travail Log Analytics<br/><br/>Si vous avez créé des projets avant la disponibilité générale d’Azure Migrate (28 février 2018), vous avez peut-être fait l’objet de frais supplémentaires pour Service Map. Pour veiller à payer uniquement après 180 jours, nous vous recommandons de créer un autre projet, car les espaces de travail présents avant la disponibilité générale restent facturables.
**Gestion** | Lorsque vous inscrivez des agents dans l’espace de travail, vous utilisez l’ID et la clé fournis par le projet Azure Migrate.<br/><br/> Vous pouvez utiliser l’espace de travail Log Analytics en dehors d’Azure Migrate.<br/><br/> Si vous supprimez le projet Azure Migrate associé, l’espace de travail n’est pas automatiquement supprimé. [Supprimez-le manuellement](../azure-monitor/platform/manage-access.md).<br/><br/> Ne supprimez pas l’espace de travail créé par Azure Migrate, sauf si vous supprimez le projet Azure Migrate. La suppression de l’espace de travail entraînerait un dysfonctionnement de la fonctionnalité de visualisation des dépendances.
**Connectivité Internet** | Si les machines ne sont pas connectées à Internet, vous devez installer la passerelle Log Analytics sur celles-ci.
**Azure Government** | L'analyse des dépendances basée sur un agent n'est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

[Préparer l’évaluation des machines virtuelles Hyper-V](./tutorial-discover-hyper-v.md)