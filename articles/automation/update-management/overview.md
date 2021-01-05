---
title: Présentation d’Update Management Azure Automation
description: Cet article présente la fonctionnalité Update Management qui implémente les mises à jour de vos machines Windows et Linux.
services: automation
ms.subservice: update-management
ms.date: 12/09/2020
ms.topic: conceptual
ms.openlocfilehash: 4b557c9772e76b6b61cdf01799ee30ba6bc11807
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928424"
---
# <a name="update-management-overview"></a>Vue d’ensemble de Update Management

Vous pouvez utiliser Update Management dans Azure Automation pour gérer les mises à jour du système d’exploitation de vos machines virtuelles Windows et Linux dans Azure, dans des environnements locaux et dans d’autres environnements cloud. Vous pouvez rapidement évaluer l’état des mises à jour disponibles sur toutes les machines d’agent et gérer le processus d’installation des mises à jour nécessaires pour les serveurs.

> [!NOTE]
> Vous ne pouvez pas utiliser de machine configurée avec Update Management pour exécuter des scripts personnalisés à partir d’Azure Automation. Cette machine ne peut exécuter que le script de mise à jour signé par Microsoft.

> [!NOTE]
> À ce stade, l’activation d’Update Management directement à partir d’un serveur Azure Arc n’est pas prise en charge. Pour connaître les conditions requises et la façon de l’activer pour votre serveur, consultez [Activer Update Management à partir de votre compte Automation](../../automation/update-management/enable-from-automation-account.md).

Pour télécharger et installer automatiquement des correctifs *critiques* et *de sécurité* sur votre machine virtuelle Azure, consultez [Mise à jour corrective automatique de l’invité de machine virtuelle](../../virtual-machines/windows/automatic-vm-guest-patching.md) pour les machines virtuelles Windows.

Avant de déployer Update Management et d’activer vos machines pour la gestion, assurez-vous de bien comprendre les informations contenues dans les sections suivantes.  

## <a name="about-update-management"></a>À propos d’Update Management

Les machines gérées par Update Management reposent sur les éléments suivants pour effectuer l’évaluation et déployer des mises à jour :

* [Agent Log Analytics](../../azure-monitor/platform/log-analytics-agent.md) pour Windows ou Linux
* PowerShell DSC (Desired State Configuration, configuration d’état souhaité) pour Linux
* Runbook Worker hybride Automation (installé automatiquement quand vous activez Update Management sur la machine)
* Microsoft Update ou [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus) (WSUS) pour les machines Windows
* Un référentiel de mises à jour privé ou public pour les ordinateurs Linux

Le schéma suivant illustre la façon dont Update Management évalue les mises à jour de sécurité et les applique à tous les serveurs Windows Server et Linux connectés dans un espace de travail :

![Workflow Update Management](./media/overview/update-mgmt-updateworkflow.png)

Update Management peut être utilisé pour déployer en mode natif sur des machines dans plusieurs abonnements du même locataire.

Après la publication d’un package, comptez un délai de deux à trois heures avant l’affichage du correctif pour l’évaluation sur des machines Linux. Sur les machines Windows, ce délai est de 12 à 15 heures. Après qu’une machine a terminé l’analyse de conformité de la mise à jour, l’agent transfère les informations en bloc aux journaux Azure Monitor. Sur une machine Windows, l’analyse de conformité est effectuée toutes les 12 heures par défaut. Sur une machine Linux, l’analyse de conformité est effectuée toutes les heures par défaut. Si l’agent Log Analytics est redémarré, une analyse de conformité est démarrée dans les 15 minutes.

En plus de l’analyse planifiée, l’analyse de conformité des mises à jour est démarrée dans les 15 minutes suivant le redémarrage de l’agent Log Analytics, avant et après l’installation de la mise à jour.

Update Management rapporte l’état de mise à jour de la machine en fonction de la source avec laquelle vous avez configuré la synchronisation. Si la machine Windows est configurée pour rapporter aux [Windows Server Update Services](/windows-server/administration/windows-server-update-services/get-started/windows-server-update-services-wsus) (WSUS), en fonction de la date de dernière synchronisation de WSUS avec Microsoft Update, les résultats peuvent être différents de ce qu’indique Microsoft Update. Le comportement est le même pour les machines Linux configurées pour rapporter à un référentiel local et non pas à un référentiel public.

> [!NOTE]
> Pour pouvoir rapporter au service, Update Management nécessite certaines URL et l’activation de ports. Pour en savoir plus sur la configuration demandée, consultez la section [Configuration réseau](../automation-hybrid-runbook-worker.md#network-planning).

Vous pouvez déployer et installer des mises à jour logicielles sur des machines qui nécessitent les mises à jour en créant un déploiement planifié. Les mises à jour considérées comme facultatives ne sont pas incluses dans l’étendue du déploiement des machines Windows. Seules les mises à jour nécessaires sont incluses dans le déploiement.

Le déploiement planifié définit quelles machines cibles reçoivent les mises à jour applicables. Soit il désigne explicitement certaines machines, soit il sélectionne un [groupe d’ordinateurs](../../azure-monitor/platform/computer-groups.md) d’après des recherches dans les journaux d’un ensemble spécifique de machines (ou sur une [requête Azure](query-logs.md) qui sélectionne des machines virtuelles Azure de manière dynamique selon des critères spécifiés). Ces groupes diffèrent de la [configuration d’étendue](../../azure-monitor/insights/solution-targeting.md), utilisée pour contrôler le ciblage des machines qui reçoivent la configuration permettant d’activer Update Management. Cela les empêche de procéder à la conformité des mises à jour et de créer des rapports, puis d’installer les mises à jour nécessaires approuvées.

Lorsque vous définissez un déploiement, vous spécifiez également une planification pour approuver et définir la période pendant laquelle les mises à jour peuvent être installées. Cette période est appelée fenêtre de maintenance. Vingt minutes de la fenêtre de maintenance sont réservées aux redémarrages si un redémarrage est nécessaire et que vous avez sélectionné l’option de redémarrage appropriée. Si la mise à jour corrective prend plus longtemps que prévu et qu’il reste moins de vingt minutes dans la fenêtre de maintenance, il n’y aura pas de redémarrage.

Les mises à jour sont installées par des Runbooks dans Azure Automation. Vous ne pouvez pas visualiser ces runbooks, ils ne nécessitent par ailleurs aucune configuration. Lorsqu’un déploiement de mises à jour est créé, il génère une planification qui démarre un runbook de mise à jour principal au moment indiqué pour les machines incluses. Ce runbook principal lance un runbook enfant sur chaque agent pour installer les mises à jour obligatoires.

À la date et l’heure spécifiées dans le déploiement de mises à jour, les machines cibles exécutent le déploiement en parallèle. Avant l’installation, une analyse est lancée pour vérifier que les mises à jour sont encore requises. Pour les machines clientes WSUS, si les mises à jour ne sont pas approuvées dans WSUS, leur déploiement échoue.

L’inscription d’une machine auprès du service Update Management dans plusieurs espaces de travail Log Analytics (également appelé multihébergement) n’est pas prise en charge.

## <a name="clients"></a>Clients

### <a name="supported-client-types"></a>Types de clients pris en charge

Le tableau suivant liste les systèmes d’exploitation pris en charge pour les évaluations des mises à jour et les mises à jour correctives. Une mise à jour corrective nécessite un runbook Worker hybride, qui est automatiquement installé quand vous activez la machine virtuelle ou le serveur pour la gestion par Update Management. Pour plus d’informations sur la configuration requise pour le runbook Worker hybride, consultez [Déployer un runbook Worker hybride Windows](../automation-windows-hrw-install.md) et [Déployer un runbook Worker hybride Linux](../automation-linux-hrw-install.md).

> [!NOTE]
> L’évaluation des mises à jour des machines Linux est prise en charge uniquement dans certaines régions, comme listé dans la [table de mappages](../how-to/region-mappings.md#supported-mappings) du compte Automation et de l’espace de travail Log Analytics.

|Système d’exploitation  |Notes  |
|---------|---------|
|Windows Server 2019 (Datacenter/Datacenter Core/Standard)<br><br>Windows Server 2016 (Datacenter/Datacenter Core/Standard)<br><br>Windows Server 2012 R2 (Datacenter/Standard)<br><br>Windows Server 2012 ||
|Windows Server 2008 R2 (RTM et SP1 Standard)| Update Management prend uniquement en charge les évaluations et les mises à jour correctives pour ce système d’exploitation. La fonctionnalité [Runbook Worker hybride](../automation-windows-hrw-install.md) est prise en charge pour Windows Server 2008 R2. |
|CentOS 6 et 7 (x64)      | Les agents Linux nécessitent un accès à un référentiel de mise à jour. La mise à jour corrective basée sur la classification nécessite que `yum` retourne les données de sécurité que CentOS n’a pas dans ses versions RTM. Pour plus d’informations sur la mise à jour corrective basée sur des classifications sur CentOS, consultez [Mettre à jour des classifications sur Linux](view-update-assessments.md#linux).          |
|Red Hat Enterprise 6 et 7 (x64)     | Les agents Linux nécessitent un accès à un référentiel de mise à jour.        |
|SUSE Linux Enterprise Server 12 (x64)     | Les agents Linux nécessitent un accès à un référentiel de mise à jour.        |
|Ubuntu 14.04 LTS, 16.04 LTS et 18.04 LTS (x64)      |Les agents Linux nécessitent un accès à un référentiel de mise à jour.         |

> [!NOTE]
> Les groupes de machines virtuelles identiques Azure peuvent être gérés via Update Management. Update Management fonctionne sur les instances, mais pas sur l’image de base. Vous devez planifier les mises à jour de façon incrémentielle, afin que toutes les instances de machines virtuelles ne soient pas mises à jour en même temps. Vous pouvez ajouter des nœuds pour les groupes de machines virtuelles identiques en suivant la procédure décrite sous [Ajouter une machine non-Azure à Change Tracking and Inventory](../automation-tutorial-installed-software.md#add-a-non-azure-machine-to-change-tracking-and-inventory).

### <a name="unsupported-client-types"></a>Types de clients non pris en charge

Le tableau suivant répertorie les systèmes d’exploitation qui ne sont pas pris en charge :

|Système d’exploitation  |Notes  |
|---------|---------|
|Client Windows     | Les systèmes d’exploitation client (par exemple, Windows 7 et Windows 10) ne sont pas pris en charge.<br> Pour Azure Windows Virtual Desktop (WVD), la méthode recommandée<br> pour la gestion des mises à jour est [Microsoft Endpoint Configuration Manager](../../virtual-desktop/configure-automatic-updates.md) pour la gestion des correctifs pour les machines clientes Windows 10. |
|Windows Server 2016 Nano Server     | Non pris en charge.       |
|Nœuds Azure Kubernetes Service (AKS) | Non pris en charge. Utilisez le processus de correction décrit dans [Appliquer des mises à jour de sécurité et du noyau à des nœuds Linux dans Azure Kubernetes Service (AKS)](../../aks/node-updates-kured.md)|

### <a name="client-requirements"></a>Configuration requise des clients

Les informations suivantes décrivent la configuration du client propre au système d’exploitation. Pour obtenir des conseils supplémentaires, consultez [Planification réseau](#ports). Pour comprendre la configuration requise du client pour le protocole TLS 1.2, consultez [Application de TLS 1.2 pour Azure Automation](../automation-managing-data.md#tls-12-enforcement-for-azure-automation).

#### <a name="windows"></a>Windows

Les agents Windows doivent être configurés pour communiquer avec un serveur WSUS, ou ils nécessitent un accès à Microsoft Update. Pour les machines hybrides, nous vous recommandons d’installer l’agent Log Analytics pour Windows en connectant d’abord votre machine à [Azure Arc enabled servers](../../azure-arc/servers/overview.md), puis d’utiliser Azure Policy pour affecter la stratégie intégrée [Déployer l’agent Log Analytics sur des machines Azure Arc Windows](../../governance/policy/samples/built-in-policies.md#monitoring). Autrement, si vous envisagez de superviser les machines avec Azure Monitor pour machines virtuelles, utilisez plutôt l’initiative [Activer Azure Monitor pour machines virtuelles](../../governance/policy/samples/built-in-initiatives.md#monitoring).

Vous pouvez utiliser Update Management avec Microsoft Endpoint Configuration Manager. Pour en savoir plus sur les scénarios d’intégration, consultez [Intégrer Update Management à Windows Endpoint Configuration Manager](mecmintegration.md). L’[agent Log Analytics pour Windows](../../azure-monitor/platform/agent-windows.md) est nécessaire aux serveurs Windows gérés par les sites dans votre environnement Configuration Manager. 

Par défaut, les machines virtuelles Windows déployées à partir de la Place de marché Azure sont configurées pour recevoir des mises à jour automatiques du service Windows Update. Ce comportement ne change pas lorsque vous ajoutez des machines virtuelles Windows à votre espace de travail. Si vous ne gérez pas activement les mises à jour avec Update Management, le comportement par défaut (pour effectuer automatiquement les mises à jour) s’applique.

> [!NOTE]
> Vous pouvez modifier la stratégie de groupe afin que les redémarrages de la machine ne puissent être effectués que par l’utilisateur et non par le système. Les machines managées peuvent rester bloquées si Update Management ne dispose pas des droits nécessaires pour les redémarrer sans intervention manuelle de l'utilisateur. Pour plus d'informations, consultez [Configurer les paramètres de stratégie de groupe pour les mises à jour automatiques](/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates).

#### <a name="linux"></a>Linux

Pour Linux, la machine demande un accès à un référentiel de mise à jour, privé ou public. TLS 1.1 ou TLS 1.2 est exigé pour interagir avec Update Management. Update Management ne prend pas en charge les agents Log Analytics pour Linux qui sont configurés pour envoyer des rapports à plusieurs espaces de travail Log Analytics. Python 2.x doit également être installé sur la machine.

> [!NOTE]
> L’évaluation des mises à jour des machines Linux est uniquement prise en charge dans certaines régions. Consultez la [table des mappages](../how-to/region-mappings.md#supported-mappings) du compte Automation et de l’espace de travail Log Analytics.

Pour les machines hybrides, nous vous recommandons d’installer l’agent Log Analytics pour Linux en connectant d’abord votre machine à [Azure Arc enabled servers](../../azure-arc/servers/overview.md), puis d’utiliser Azure Policy pour affecter la stratégie intégrée [Déployer l’agent Log Analytics sur des machines Azure Arc Linux](../../governance/policy/samples/built-in-policies.md#monitoring). Autrement, si vous envisagez de superviser les machines avec Azure Monitor pour machines virtuelles, utilisez plutôt l’initiative [Activer Azure Monitor pour machines virtuelles](../../governance/policy/samples/built-in-initiatives.md#monitoring).

Les machines virtuelles créées à partir des images Red Hat Enterprise Linux (RHEL) à la demande, disponibles sur la Place de marché Azure, sont inscrites pour accéder à l’[infrastructure RHUI (Red Hat Update Infrastructure)](../../virtual-machines/workloads/redhat/redhat-rhui.md) déployée dans Azure. Toute autre distribution Linux doit être mise à jour à partir du référentiel des fichiers en ligne de la distribution, au moyen des méthodes prises en charge par la distribution.

## <a name="permissions"></a>Autorisations

Pour créer et gérer des déploiements de mises à jour, vous devez disposer d’autorisations spécifiques. Pour en savoir plus sur ces autorisations, consultez [Accès en fonction du rôle – Update Management](../automation-role-based-access-control.md#update-management-permissions).

## <a name="update-management-components"></a>Composants Update Management

Update Management utilise les ressources décrites dans cette section. Ces ressources sont automatiquement ajoutées à votre compte Automation quand vous activez Update Management.

### <a name="hybrid-runbook-worker-groups"></a>Groupes de Runbooks Workers hybrides

Après avoir activé Update Management, toute machine Windows directement connectée à votre espace de travail Log Analytics est automatiquement configurée en tant que runbook Worker hybride, afin de gérer les runbooks qui prennent en charge Update Management.

Chaque machine Windows gérée par Update Management est répertoriée dans le volet Groupes de Workers hybrides en tant que Groupe de Workers hybrides système pour le compte Automation. Les groupes utilisent la convention d’affectation de noms `Hostname FQDN_GUID`. Vous ne pouvez pas cibler ces groupes avec des Runbooks dans votre compte. Si vous essayez, la tentative échoue. Ces groupes sont destinés à prendre uniquement en charge Update Management. Pour en savoir plus sur l’affichage de la liste des machines Windows configurées en tant que Runbook Worker hybride, consultez [Afficher les Runbooks Workers hybrides](../automation-hybrid-runbook-worker.md#view-system-hybrid-runbook-workers).

Vous pouvez ajouter la machine Windows à un groupe de runbooks Workers hybrides dans votre compte Automation, afin de prendre en charge des runbooks Automation, à condition d’utiliser le même compte pour Update Management et pour l’appartenance au groupe de runbooks Workers hybrides. Cette fonctionnalité a été ajoutée à la version 7.2.12024.0 du Runbook Worker hybride.

### <a name="management-packs"></a>Packs d’administration

Si votre groupe d’administration Operations Manager est [connecté à un espace de travail Log Analytics](../../azure-monitor/platform/om-agents.md), les packs d’administration suivants sont installés dans Operations Manager. Ces packs d’administration sont également installés pour Update Management sur des machines Windows directement connectées. Il n’est pas nécessaire de les configurer ou de les gérer.

* Microsoft System Center Advisor Update Assessment Intelligence Pack (Microsoft.IntelligencePacks.UpdateAssessment)
* Microsoft.IntelligencePack.UpdateAssessment.Configuration (Microsoft.IntelligencePack.UpdateAssessment.Configuration)
* Pack d’administration du déploiement des mises à jour

> [!NOTE]
> Si vous avez un groupe d’administration Operations Manager 1807 ou 2019 connecté à un espace de travail Log Analytics avec des agents configurés dans le groupe d’administration pour collecter des données de journal, vous avez besoin de remplacer le paramètre `IsAutoRegistrationEnabled` et de le définir sur True dans la règle **Microsoft.IntelligencePacks.AzureAutomation.HybridAgent.Init**.

Pour plus d’informations sur les mises à jour des packs d’administration, consultez [Connecter Operations Manager aux journaux Azure Monitor](../../azure-monitor/platform/om-agents.md).

> [!NOTE]
> Pour que Update Management gère entièrement les machines avec l’agent Log Analytics, vous devez mettre à jour l’agent Log Analytics pour Windows ou l’agent Log Analytics pour Linux. Pour savoir comment mettre à jour l’agent, consultez [Guide pratique pour mettre à niveau un agent Operations Manager](/system-center/scom/deploy-upgrade-agents). Dans les environnements qui utilisent Operations Manager, vous devez exécuter System Center Operations Manager 2012 R2 UR 14 ou une version ultérieure.

## <a name="data-collection"></a>Collecte de données

### <a name="supported-sources"></a>Sources prises en charge

Le tableau suivant décrit les sources connectées que Update Management prend en charge :

| Source connectée | Prise en charge | Description |
| --- | --- | --- |
| Agents Windows |Oui |Update Management collecte des informations sur les mises à jour système à partir des agents Windows, puis démarre l’installation des mises à jour obligatoires. |
| Agents Linux |Oui |Update Management collecte des informations sur les mises à jour système à partir des agents Linux, puis démarre l’installation des mises à jour obligatoires sur les distributions prises en charge. |
| Groupe d’administration d’Operations Manager |Oui |La gestion des mises à jour collecte des informations sur les mises à jour système des agents dans un groupe d’administration connecté.<br/><br/>Une connexion directe entre l’agent Operations Manager et les journaux Azure Monitor n’est pas obligatoire. Les données sont transférées du groupe d’administration à l’espace de travail Log Analytics. |

### <a name="collection-frequency"></a>Fréquence de collecte

Update Management analyse les données des machines gérées à l’aide des règles suivantes. L’affichage sur le tableau de bord des données mises à jour provenant des machines gérées peut prendre entre 30 minutes et 6 heures.

* Chaque machine Windows : Update Management effectue une analyse deux fois par jour pour chaque machine.

* Chaque machine Linux : Update Management effectue une analyse toutes les heures.

La consommation moyenne de données par les journaux Azure Monitor pour une machine utilisant Update Management est d’environ 25 Mo par mois. Cette valeur est approximative et sujette à modification en fonction de votre environnement. Nous vous recommandons de surveiller votre environnement pour assurer le suivi de votre consommation exacte. Pour plus d'informations sur l'analyse de l'utilisation des données des journaux d'activité Azure Monitor, consultez [Gérer l'utilisation et les coûts](../../azure-monitor/platform/manage-cost-storage.md).

## <a name="network-planning"></a><a name="ports"></a>Planification réseau

Les adresses suivantes sont exigées particulièrement pour Update Management. La communication avec ces adresses s’effectue par le biais du port 443.

|Azure (public)  |Azure Government  |
|---------|---------|
|`*.ods.opinsights.azure.com`    | `*.ods.opinsights.azure.us`        |
|`*.oms.opinsights.azure.com`     | `*.oms.opinsights.azure.us`        |
|`*.blob.core.windows.net` | `*.blob.core.usgovcloudapi.net`|
|`*.azure-automation.net` | `*.azure-automation.us`|

Lorsque vous créez des règles de sécurité de groupe réseau ou configurez le pare-feu Azure pour autoriser le trafic vers le service Automation et l’espace de travail Log Analytics, utilisez la balise [service](../../virtual-network/service-tags-overview.md#available-service-tags) **GuestAndHybridManagement** et **AzureMonitor**. Cela simplifie la gestion continue de vos règles de sécurité réseau. Pour vous connecter au service Automation à partir de vos machines virtuelles Azure en toute sécurité et de façon privée, consultez [Utiliser Azure Private Link](../how-to/private-link-security.md). Pour obtenir la balise de service et les informations de plage actuelles à inclure dans le cadre de vos configurations du pare-feu local, consultez les [fichiers JSON téléchargeables](../../virtual-network/service-tags-overview.md#discover-service-tags-by-using-downloadable-json-files).

Pour les machines Windows, vous devez également autoriser le trafic vers tous les points de terminaison requis par Windows Update. Vous trouverez une liste actualisée des points de terminaison requis dans [Problèmes liés à HTTP/au proxy](/windows/deployment/update/windows-update-troubleshooting#issues-related-to-httpproxy). Si vous disposez d’un [serveur Windows Update](/windows-server/administration/windows-server-update-services/plan/plan-your-wsus-deployment) local, vous devez également autoriser le trafic vers le serveur spécifié dans votre [clé WSUS](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry).

Pour les machines Red Hat Linux, consultez [Adresses IP des serveurs de distribution de contenu RHUI](../../virtual-machines/workloads/redhat/redhat-rhui.md#the-ips-for-the-rhui-content-delivery-servers) pour les points de terminaison requis. Pour les autres distributions Linux, reportez-vous à la documentation du fournisseur.

Pour plus d’informations sur les ports nécessaires pour le Runbook Worker hybride, consultez [Adresses Update Management pour Runbook Worker hybride](../automation-hybrid-runbook-worker.md#update-management-addresses-for-hybrid-runbook-worker).

Si vos stratégies de sécurité informatique n’autorisent pas les machines du réseau à se connecter à Internet, vous pouvez configurer une [passerelle Log Analytics](../../azure-monitor/platform/gateway.md), puis configurer la machine pour qu’elle se connecte à Azure Automation et Azure Monitor via la passerelle.

## <a name="update-classifications"></a>Classifications des mises à jour

Le tableau suivant définit les classifications que Update Management prend en charge pour les mises à jour Windows.

|classification ;  |Description  |
|---------|---------|
|Mises à jour critiques     | Mise à jour d’un problème spécifique qui concerne un bogue critique non lié à la sécurité.        |
|Mises à jour de sécurité     | Mise à jour pour un problème de sécurité propre à un produit.        |
|Correctifs cumulatifs     | Cumul de correctifs logiciels regroupés pour faciliter leur déploiement.        |
|Packs de fonctionnalités     | Nouvelles fonctionnalités de produit distribuées en dehors d’une version de produit.        |
|Service Packs     | Cumul de correctifs logiciels appliqués à une application.        |
|Mises à jour de définitions     | Mise à jour de fichiers de virus ou d’autres définitions.        |
|Outils     | Utilitaire ou fonctionnalité permettant d’effectuer une ou plusieurs tâches.        |
|Mises à jour     | Mise à jour d’une application ou d’un fichier actuellement installé.        |

Le tableau suivant définit les classifications prises en charge pour les mises à jour Linux.

|classification ;  |Description  |
|---------|---------|
|Mises à jour critiques et de sécurité     | Mises à jour pour un problème spécifique ou un problème de sécurité propre à un produit.         |
|Autres mises à jour     | Toutes les autres mises à jour qui ne sont ni critiques par nature, ni des mises à jour de sécurité.        |

>[!NOTE]
>La classification des mises à jour pour les machines Linux n’est disponible que lorsqu’elle est utilisée dans les régions de cloud public Azure prises en charge. Quand Update Management est utilisé dans les régions de cloud national suivantes :
>* Azure US Government
>* 21Vianet en Chine
>
> il n’y a pas de classification de mises à jour Linux et elles sont signalées sous la catégorie **Autres mises à jour**. Update Management utilise les données publiées par les distributions prises en charge, en particulier leurs fichiers publiés [OVAL](https://oval.mitre.org/) (Open Vulnerability and Assessment Language). Étant donné que l’accès Internet à partir de ces clouds nationaux est limité, Update Management ne peut ni accéder à ces fichiers ni les utiliser.

Pour Linux, Update Management peut faire la différence entre les mises à jour critiques et les mises à jour de sécurité dans le cloud sous la classification **Sécurité** ou **Autres**, tout en affichant les données d’évaluation en raison de l’enrichissement des données dans le cloud. Pour la mise à jour corrective, Update Management s’appuie sur les données de classification disponibles sur l’ordinateur. Contrairement à d’autres distributions, CentOS n’a pas accès à ces informations dans la version RTM. Si vous disposez d’ordinateurs CentOS configurés pour retourner les données de sécurité pour la commande suivante, Update Management peut appliquer la mise à jour corrective en fonction des classifications.

```bash
sudo yum -q --security check-update
```

Il n’existe actuellement aucune méthode prise en charge permettant d’activer la disponibilité des données de classification natives sur CentOS. Pour le moment, une prise en charge limitée est proposée aux clients qui pourraient avoir activé cette fonctionnalité eux-mêmes.

Pour classifier les mises à jour sur Red Hat Enterprise version 6, vous devez installer le plug-in yum-security. Sur Red Hat Enterprise Linux 7, le plug-in faisant déjà partie de yum lui-même, il est inutile d’installer quoi que ce soit. Pour plus d’informations, consultez l’[article de base de connaissances](https://access.redhat.com/solutions/10021) suivant sur Red Hat.

Quand vous planifiez une mise à jour à exécuter sur un ordinateur Linux qui, par exemple, est configurée pour installer uniquement les mises à jour correspondant à la classification **Sécurité**, les mises à jour installées peuvent être différentes ou constituer une partie des mises à jour correspondant à cette classification. Quand une évaluation des mises à jour du système d’exploitation en attente pour votre ordinateur Linux est effectuée, les fichiers [OVAL](https://oval.mitre.org/) (Open Vulnerability and Assessment Language) fournis par le fournisseur de distribution Linux sont utilisés par Update Management pour la classification.

La catégorisation est effectuée pour les mises à jour Linux en tant que **Sécurité** ou **Autres** en fonction des fichiers OVAL, ce qui inclut les mises à jour traitant des problèmes de sécurité ou des vulnérabilités. Toutefois, lorsque la planification des mises à jour est exécutée, elle s’exécute sur l’ordinateur Linux à l’aide du gestionnaire de package approprié pour les installer (p. ex., YUM, APT ou ZYPPER). Le gestionnaire de package pour la distribution Linux peut avoir un mécanisme différent pour classifier les mises à jour, où les résultats peuvent différer de ceux obtenus à partir des fichiers OVAL par Update Management. Pour vérifier manuellement la machine et savoir quelles mises à jour sont pertinentes pour la sécurité du point de vue de votre gestionnaire de package, consultez [Résoudre les problèmes liés à Update Management pour Linux](../troubleshoot/update-management.md#updates-linux-installed-different).

## <a name="integrate-update-management-with-configuration-manager"></a>Intégrer Update Management à Configuration Manager

Les clients qui ont investi dans Microsoft Endpoint Configuration Manager pour gérer des PC, serveurs et appareils mobiles s’appuient aussi sur la puissance et la maturité de Configuration Manager pour gérer les mises à jour logicielles. Pour savoir comment intégrer Update Management à Configuration Manager, consultez [Intégrer Update Management à Windows Endpoint Configuration Manager](mecmintegration.md).

## <a name="third-party-updates-on-windows"></a>Mises à jour tierces sur Windows

Update Management s’appuie sur le référentiel de mise à jour configuré localement pour mettre à jour les systèmes Windows pris en charge : WSUS ou Windows Update. Des outils, tels que l’[éditeur de mise à jour System Center ](/configmgr/sum/tools/updates-publisher), vous permettent d’importer et de publier des mises à jour personnalisées avec WSUS. Ce scénario permet à Update Management de mettre à jour des machines qui utilisent Configuration Manager comme référentiel de mise à jour avec des logiciels tiers. Pour savoir comment configurer l’éditeur de mise à jour, consultez [Installer l’éditeur de mise à jour](/configmgr/sum/tools/install-updates-publisher).

## <a name="enable-update-management"></a>Activer Update Management

Voici comment vous pouvez activer Update Management et sélectionner les machines à gérer :

- Un [modèle Azure Resource Manager](enable-from-template.md) est disponible pour déployer Update Management sur un compte Automation et un espace de travail Log Analytics Azure Monitor nouveaux ou existants dans votre abonnement. Il ne configure pas l’étendue des machines qui doivent être gérées ; cette opération fait l’objet d’une étape distincte après l’utilisation du modèle.

- À partir de votre [compte Automation](enable-from-automation-account.md) pour une ou plusieurs machines Azure et non-Azure, notamment des serveurs compatibles avec Arc.

- Utilisation de la méthode de [runbook](enable-from-runbook.md) **Enable-AutomationSolution**.

- Pour une [machine virtuelle Azure sélectionnée](enable-from-vm.md) dans la page **Machine virtuelle** du portail Azure. Ce scénario est disponible pour les machines virtuelles Linux et Windows.

- Pour [plusieurs machines virtuelles Azure](enable-from-portal.md), sélectionnez-les dans la page **Machines virtuelles** du portail Azure.

> [!NOTE]
> Update Management vous demande d’établir un lien entre l’espace de travail Log Analytics et votre compte Automation. Pour obtenir la liste définitive des régions prises en charge, consultez [Mappages Azure Workspace](../how-to/region-mappings.md). Les mappages de région n’empêchent pas de gérer les machines virtuelles dans une autre région depuis votre compte Automation.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation d’Update Management, consultez [Gérer les mises à jour pour vos machines virtuelles](manage-updates-for-vm.md).

* Consultez les questions fréquemment posées au sujet d’Update Management sur le [forum aux questions Azure Automation](../automation-faq.md).
