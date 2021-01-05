---
title: Prise en charge pour l’évaluation de serveurs physiques dans Azure Migrate
description: Découvrez la prise en charge pour l’évaluation de serveurs physiques à l’aide de l’outil Évaluation de serveur d’Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 06/03/2020
ms.openlocfilehash: cfbbc1d218f590241fab804e389acd689c009dac
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96754009"
---
# <a name="support-matrix-for-physical-server-assessment"></a>Tableau de prise en charge pour l’évaluation de serveurs physiques 

Cet article résume les conditions préalables et les exigences de prise en charge lors de l’évaluation de serveurs physiques pour la migration vers Azure et utilise l’outil [Azure Migrate : Évaluation de serveur](migrate-services-overview.md#azure-migrate-server-assessment-tool). Si vous souhaitez migrer des serveurs physiques vers Azure, passez en revue la [matrice de prise en charge de la migration](migrate-support-matrix-physical-migration.md).


Pour évaluer des serveurs physiques, vous créez un projet Azure Migrate et ajoutez l’outil Évaluation de serveur au projet. Une fois l’outil ajouté, vous déployez l’[appliance Azure Migrate](migrate-appliance.md). Celle-ci découvre en continu les machines locales et envoie leurs métadonnées et leurs données de performances à Azure. Une fois la découverte terminée, vous rassemblez les machines découvertes dans des groupes et effectuez l’évaluation d’un groupe.


## <a name="limitations"></a>Limites

**Support** | **Détails**
--- | ---
**Limites d’évaluation** | Vous pouvez découvrir et évaluer jusqu’à 35 000 serveurs physiques dans un même [projet Azure Migrate](migrate-support-matrix.md#azure-migrate-projects).
**Limites de projet** | Vous pouvez créer plusieurs projets dans un abonnement Azure. En plus de serveurs physiques, un projet peut inclure des machines virtuelles VMware et Hyper-V, jusqu’aux limites d’évaluation pour chaque catégorie.
**Découverte** | L’appliance Azure Migrate peut découvrir jusqu’à 1000 serveurs physiques.
**Évaluation** | Vous pouvez ajouter jusqu’à 35 000 machines dans un groupe unique.<br/><br/> Vous pouvez évaluer jusqu’à 35 000 machines par évaluation.

[Apprenez-en davantage](concepts-assessment-calculation.md) sur les évaluations.

## <a name="physical-server-requirements"></a>Conditions requises des serveurs physiques

**Déploiement de serveur physique :** Le serveur physique peut être autonome ou déployé dans un cluster.

**Système d’exploitation :** Tous les systèmes d’exploitation Windows et Linux peuvent être évalués pour la migration.

**Autorisations :**
- En ce qui concerne les serveurs Windows, utilisez un compte de domaine pour les ordinateurs joints à un domaine et un compte local pour les autres. Le compte d’utilisateur doit être ajouté à ces groupes : Utilisateurs de gestion à distance, Utilisateurs de l’Analyseur de performances et Utilisateurs du Journal des performances.
- Pour les serveurs Linux, vous devez disposer d’un compte racine sur les serveurs Linux que vous souhaitez découvrir. Vous pouvez également définir un compte non racine doté des fonctionnalités requises à l’aide des commandes suivantes :

**Commande** | **Objectif**
--- | --- |
setcap CAP_DAC_READ_SEARCH+eip /usr/sbin/fdisk <br></br> setcap CAP_DAC_READ_SEARCH+eip /sbin/fdisk _(si /usr/sbin/fdisk n’est pas présent)_ | Collecter les données de configuration du disque
setcap "cap_dac_override,cap_dac_read_search,cap_fowner,cap_fsetid,cap_setuid,<br>cap_setpcap,cap_net_bind_service,cap_net_admin,cap_sys_chroot,cap_sys_admin,<br>cap_sys_resource,cap_audit_control,cap_setfcap=+eip" /sbin/lvm | Collecter les données de performances du disque
setcap CAP_DAC_READ_SEARCH+eip /usr/sbin/dmidecode | Collecter le numéro de série du BIOS
chmod a+r /sys/class/dmi/id/product_uuid | Collecter le GUID du BIOS



## <a name="azure-migrate-appliance-requirements"></a>Conditions requises de l’appliance Azure Migrate

Azure Migrate utilise l’[appliance Azure Migrate](migrate-appliance.md) pour la découverte et l’évaluation. L’appliance pour les serveurs physiques peut s’exécuter sur une machine virtuelle ou sur une machine physique. 

- En savoir plus sur les [conditions requises de l’appliance](migrate-appliance.md#appliance---physical) pour les serveurs physiques.
- Découvrez les URL auxquelles l’appliance doit accéder dans les clouds [publics](migrate-appliance.md#public-cloud-urls) et du [secteur public](migrate-appliance.md#government-cloud-urls).
- Vous configurez l’appliance à l’aide d’un [script PowerShell](how-to-set-up-appliance-physical.md) que vous téléchargez à partir du Portail Azure.
Dans Azure Government, déployez l’appliance [avec ce script](deploy-appliance-script-government.md).

## <a name="port-access"></a>Accès au port

Le tableau suivant résume les exigences du port pour l’évaluation.

**Appareil** | **Connection**
--- | ---
**Appliance** | Connexions entrantes sur le port TCP 3389 pour permettre des connexions Bureau à distance avec l’appliance.<br/><br/> Connexions entrantes sur le port 44368 pour accéder à distance à l’application de gestion de l’appliance via l’URL : ``` https://<appliance-ip-or-name>:44368 ```.<br/><br/> Connexions sortantes sur les ports 443 (HTTPS) pour envoyer les métadonnées de découverte et de performances à Azure Migrate.
**Serveurs physiques** | **Windows :** Connexion entrante sur le port WinRM 5985 (HTTP) pour extraire les métadonnées de configuration et de performances des serveurs Windows. <br/><br/> **Linux :**  Connexions entrantes sur le port 22 (TCP) pour extraire les métadonnées de configuration et de performances des serveurs Linux. |

## <a name="agent-based-dependency-analysis-requirements"></a>Conditions requises de l’analyse des dépendances basées sur un agent

L’[analyse des dépendances](concepts-dependency-visualization.md) vous permet d’identifier les dépendances entre les machines locales que vous souhaitez évaluer et faire migrer vers Azure. Le tableau récapitule les conditions requises pour la configuration de l’analyse des dépendances basées sur un agent. Actuellement, seule l’analyse des dépendances basées sur un agent est prise en charge pour les serveurs physiques.

**Prérequis** | **Détails** 
--- | --- 
**Avant le déploiement** | Vous devez disposer d’un projet Azure Migrate, avec l’outil Évaluation de serveur ajouté au projet.<br/><br/>  Vous déployez la visualisation des dépendances après avoir configuré une appliance Azure Migrate pour découvrir vos machines locales.<br/><br/> [Découvrez comment](create-manage-projects.md) créer un projet pour la première fois.<br/> [Découvrez comment](how-to-assess.md) ajouter un outil d’évaluation à un projet existant.<br/> Découvrez comment configurer l’appliance Azure Migrate pour l’évaluation de serveurs [Hyper-V](how-to-set-up-appliance-hyper-v.md), [VMware](how-to-set-up-appliance-vmware.md) ou physiques.
**Azure Government** | La visualisation des dépendances n’est pas disponible dans Azure Government.
**Log Analytics** | Azure Migrate utilise la solution [Service Map](../azure-monitor/insights/service-map.md) dans [Journaux d’activité Azure Monitor](../azure-monitor/log-query/log-query-overview.md) pour la visualisation des dépendances.<br/><br/> Vous associez un espace de travail Log Analytics nouveau ou déjà existant à un projet Azure Migrate. L’espace de travail d’un projet Azure Migrate ne peut pas être modifié une fois qu’il a été ajouté. <br/><br/> L’espace de travail doit se trouver dans le même abonnement que le projet Azure Migrate.<br/><br/> L’espace de travail doit résider dans les régions USA Est, Asie Sud-Est ou Europe Ouest. Les espaces de travail des autres régions ne peuvent pas être associés à un projet.<br/><br/> L’espace de travail doit se trouver au sein d’une région dans laquelle [Service Map est pris en charge](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br/><br/> Dans Log Analytics, l’espace de travail associé à Azure Migrate est marqué avec la clé de projet de migration et le nom du projet.
**Agents nécessaires** | Sur chaque machine à analyser, installez les agents suivants :<br/><br/> [MMA (Microsoft Monitoring Agent)](../azure-monitor/platform/agent-windows.md)<br/> [Dependency Agent](../azure-monitor/platform/agents-overview.md#dependency-agent)<br/><br/> Si les machines locales ne sont pas connectées à Internet, vous devez télécharger et installer la passerelle de l’espace de travail Log Analytics sur celles-ci.<br/><br/> En savoir plus sur l’installation de [Dependency Agent](how-to-create-group-machine-dependencies.md#install-the-dependency-agent) et de [MMA](how-to-create-group-machine-dependencies.md#install-the-mma).
**Espace de travail Log Analytics** | L’espace de travail doit se trouver dans le même abonnement que le projet Azure Migrate.<br/><br/> Azure Migrate prend en charge les espaces de travail résidant dans les régions USA Est, Asie Sud-Est et Europe Ouest.<br/><br/>  L’espace de travail doit se trouver au sein d’une région dans laquelle [Service Map est pris en charge](../azure-monitor/insights/vminsights-configure-workspace.md#supported-regions).<br/><br/> L’espace de travail d’un projet Azure Migrate ne peut pas être modifié une fois qu’il a été ajouté.
**Coûts** | La solution Service Map n’entraîne aucun frais pendant les 180 premiers jours (à compter du jour où vous associez l’espace de travail Log Analytics au projet Azure Migrate).<br/><br/> Au bout de 180 jours, des frais Log Analytics standard s’appliquent.<br/><br/> L’utilisation d’une autre solution que Service Map dans l’espace de travail Log Analytics associé entraîne des [frais standard](https://azure.microsoft.com/pricing/details/log-analytics/) pour l’espace de travail Log Analytics.<br/><br/> Lorsque le projet Azure Migrate est supprimé, l’espace de travail ne l’est pas. Une fois le projet supprimé, l’utilisation de Service Map n’est plus gratuite, et chaque nœud est facturé en fonction du niveau payant de l’espace de travail Log Analytics<br/><br/>Si vous avez créé des projets avant la disponibilité générale d’Azure Migrate (28 février 2018), vous avez peut-être fait l’objet de frais supplémentaires pour Service Map. Pour veiller à payer uniquement après 180 jours, nous vous recommandons de créer un autre projet, car les espaces de travail présents avant la disponibilité générale restent facturables.
**Gestion** | Lorsque vous inscrivez des agents dans l’espace de travail, vous utilisez l’ID et la clé fournis par le projet Azure Migrate.<br/><br/> Vous pouvez utiliser l’espace de travail Log Analytics en dehors d’Azure Migrate.<br/><br/> Si vous supprimez le projet Azure Migrate associé, l’espace de travail n’est pas automatiquement supprimé. [Supprimez-le manuellement](../azure-monitor/platform/manage-access.md).<br/><br/> Ne supprimez pas l’espace de travail créé par Azure Migrate, sauf si vous supprimez le projet Azure Migrate. La suppression de l’espace de travail entraînerait un dysfonctionnement de la fonctionnalité de visualisation des dépendances.
**Connectivité Internet** | Si les machines ne sont pas connectées à Internet, vous devez installer la passerelle Log Analytics sur celles-ci.
**Azure Government** | L'analyse des dépendances basée sur un agent n'est pas prise en charge.

## <a name="next-steps"></a>Étapes suivantes

[Préparer l’évaluation des serveurs physiques](./tutorial-discover-physical.md).