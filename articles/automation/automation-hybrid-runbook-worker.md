---
title: Vue d’ensemble du Runbook Worker hybride d’Azure Automation
description: Cet article fournit une vue d’ensemble du Runbook Worker hybride qui vous permet d’exécuter des runbooks sur des machines de votre centre de données local ou de votre fournisseur de cloud.
services: automation
ms.subservice: process-automation
ms.date: 11/23/2020
ms.topic: conceptual
ms.openlocfilehash: 7feac3ccb94cd8b4b0fab509477d4dbf772df2ae
ms.sourcegitcommit: 2ba6303e1ac24287762caea9cd1603848331dd7a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97505526"
---
# <a name="hybrid-runbook-worker-overview"></a>Vue d’ensemble des Runbook Worker hybrides

Dans Azure Automation, les Runbooks peuvent ne pas avoir accès aux ressources d’autres clouds ou de votre environnement local, car ils s’exécutent dans la plateforme cloud Azure. Vous pouvez utiliser la fonctionnalité Runbook Worker hybride d’Azure Automation pour exécuter des runbooks directement sur la machine qui héberge le rôle et sur les ressources de l’environnement afin de gérer ces ressources locales. Les runbooks sont stockés et gérés dans Azure Automation et remis ensuite à une ou plusieurs machines assignées.

L’image suivante illustre cette fonctionnalité :

![Vue d’ensemble des Runbook Worker hybrides](media/automation-hybrid-runbook-worker/automation.png)

Il existe deux types de Runbook Worker : système et utilisateur. Le tableau suivant décrit les différences entre les deux.

|Type | Description |
|-----|-------------|
|**Système** |Prend en charge un ensemble de runbooks masqués utilisés par la fonctionnalité Update Management et conçus pour installer des mises à jour spécifiées par l’utilisateur sur des ordinateurs Windows et Linux.<br> Ce type de Runbook Worker hybride n’est pas membre d’un groupe Runbook Worker hybride et, par conséquent, n’exécute pas des runbooks qui ciblent un groupe Runbook Worker. |
|**Utilisateur** |Prend en charge des runbooks définis par l’utilisateur, destinés à s’exécuter directement sur l’ordinateur Windows et Linux, et qui sont membres d’un ou plusieurs groupes Runbook Worker. |

Un Runbook Worker hybride peut s’exécuter sur le système d’exploitation Windows ou Linux, et ce rôle s’appuie sur [l’agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md) créant des rapports dans un [espace de travail Log Analytics](../azure-monitor/platform/design-logs-deployment.md) Azure Monitor. L’espace de travail ne vise pas seulement à superviser la machine pour le système d’exploitation pris en charge ; il sert aussi à télécharger les composants nécessaires pour installer le Runbook Worker hybride.

Quand la solution [Update Management](./update-management/overview.md) d’Azure Automation est activée, toute machine connectée à votre espace de travail Log Analytics est automatiquement configurée en tant que Runbook Worker hybride système.

Chaque Runbook Worker hybride utilisateur est membre d'un groupe Runbook Worker hybride que vous spécifiez lorsque vous installez le worker. Un groupe peut inclure un seul worker, mais vous pouvez inclure plusieurs workers dans un groupe pour une haute disponibilité. Chaque machine peut héberger un Runbook Worker hybride créant des rapports dans un compte Automation. Vous ne pouvez pas inscrire le worker hybride dans plusieurs comptes Automation. En effet, un worker hybride ne peut écouter des travaux qu’à partir d’un seul compte Automation. Les machines qui hébergent le Runbook Worker hybride système géré par Update Management peuvent être ajoutées à un groupe Runbook Worker hybride. Toutefois, vous devez utiliser le même compte Automation pour Update Management et l’appartenance au groupe Runbook Worker hybride.

Lorsque vous démarrez un runbook sur un Runbook Worker hybride utilisateur, vous spécifiez le groupe sur lequel il s’exécute. Chaque worker du groupe interroge Azure Automation pour voir si des travaux sont disponibles. Si un travail est disponible, le premier worker qui le reçoit s’en occupe. Le temps de traitement de la file d’attente des travaux varie selon la charge et le profil matériel du Worker hybride. Vous ne pouvez pas spécifier un worker en particulier. Le worker hybride travaille selon un mécanisme d’interrogation (toutes les 30 secondes) et suit l’ordre du premier arrivé, premier servi. Selon le moment auquel un travail a été envoyé (Push), le worker hybride qui effectue un test ping du service Automation récupère le travail. Un seul worker hybride peut généralement récupérer quatre travaux par test Ping (c’est-à-dire, toutes les 30 secondes). Si votre taux d’envoi de travaux est supérieur à quatre toutes les 30 secondes, il est possible qu’un autre worker hybride du groupe Runbook Worker hybride ait récupéré le travail.

Pour contrôler la distribution de runbooks sur les Runbook Workers hybrides, ainsi que le moment ou la façon dont les travaux sont déclenchés, vous pouvez inscrire le worker hybride dans différents groupes Runbook Worker hybrides dans votre compte Automation. Ciblez les travaux sur le ou les groupes spécifiques afin de prendre en charge votre organisation d’exécution.

Utilisez un Runbook Worker hybride plutôt qu’un [bac à sable (sandbox) Azure](automation-runbook-execution.md#runbook-execution-environment), parce qu’il est exempt de bon nombre des [limites](../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits) du bac à sable (sandbox) en matière d’espace disque, de mémoire ou de sockets réseau. Les limites d’un Worker hybride n’ont trait qu’aux ressources propres de ce dernier.

> [!NOTE]
> Les Runbook Worker hybrides ne sont pas contraints par la limite de temps de [répartition de charge équilibrée](automation-runbook-execution.md#fair-share) imposée au bac à sable Azure.

## <a name="hybrid-runbook-worker-installation"></a>Installation de Runbook Worker hybride

Le processus d’installation d’un Runbook Worker hybride utilisateur diffère selon le système d’exploitation. Le tableau ci-dessous définit les types de déploiement.

|Système d’exploitation  |Types de déploiement  |
|---------|---------|
|Windows     | [Automatisé](automation-windows-hrw-install.md#automated-deployment)<br>[Manuel](automation-windows-hrw-install.md#manual-deployment)        |
|Linux     | [Manuel](automation-linux-hrw-install.md#install-a-linux-hybrid-runbook-worker)        |

La méthode d’installation recommandée pour un ordinateur Windows consiste à utiliser un runbook Azure Automation afin d’automatiser entièrement le processus de configuration. Si cela n’est pas possible, vous pouvez suivre une procédure pas à pas pour installer et configurer le rôle manuellement. Pour une machine Linux, vous exécutez un script Python pour installer l’agent.

## <a name="network-planning"></a><a name="network-planning"></a>Planification réseau

Pour qu’un Runbook Worker hybride système et utilisateur se connecte à Azure Automation et y soit inscrit, il doit avoir accès au numéro de port et aux URL décrites dans cette section. Le Worker doit également avoir accès aux [ports et URL requis pour l’agent Log Analytics](../azure-monitor/platform/agent-windows.md) pour pouvoir se connecter à l’espace de travail Azure Monitor Log Analytics.

Le port et les URL suivants sont requis pour le Runbook Worker hybride :

* Port : seul le port TCP 443 est nécessaire pour l’accès Internet sortant
* URL globale : `*.azure-automation.net`
* URL globale de US Gov Virginie : `*.azure-automation.us`
* Service Agent : `https://<workspaceId>.agentsvc.azure-automation.net`

Si vous avez un compte Automation défini pour une région spécifique, vous pouvez restreindre la communication du Runbook Worker hybride à ce centre de données régional. Examinez les [enregistrements DNS utilisés par Azure Automation](how-to/automation-region-dns-records.md) pour identifier les enregistrements DNS obligatoires.

### <a name="proxy-server-use"></a>Utilisation du serveur proxy

Si vous utilisez un serveur proxy pour la communication entre Azure Automation et les machines exécutant l’agent Log Analytics, veillez à ce que les ressources appropriées soient accessibles. Le délai d’expiration pour les demandes du Runbook Worker hybride et des services Automation est de 30 secondes. Après trois tentatives, une demande échoue.

### <a name="firewall-use"></a>Utilisation du pare-feu

Si vous utilisez un pare-feu pour restreindre l’accès à Internet, vous devez configurer le pare-feu pour autoriser l’accès. Si vous utilisez la passerelle Log Analytics en tant que proxy, vérifiez qu’elle est configurée pour les Runbook Workers hybrides. Consultez [Configurer de la passerelle Log Analytics pour les Runbooks Workers hybrides Azure Automation](../azure-monitor/platform/gateway.md).

### <a name="service-tags"></a>Étiquettes de service

Azure Automation prend en charge les étiquettes de service de réseau virtuel Azure, à commencer par l’étiquette de service [GuestAndHybridManagement](../virtual-network/service-tags-overview.md). Vous pouvez utiliser des étiquettes de service pour définir des contrôles d’accès réseau sur des [groupes de sécurité réseau](../virtual-network/network-security-groups-overview.md#security-rules) ou sur le [pare-feu Azure](../firewall/service-tags.md). Les étiquettes de service peuvent être utilisées à la place d’adresses IP spécifiques pendant la création de règles de sécurité. En spécifiant le nom d’étiquette de service **GuestAndHybridManagement** dans le champ source ou de destination approprié d’une règle, vous pouvez autoriser ou refuser le trafic pour le service Automation. Cette étiquette de service ne permet pas d’autoriser un contrôle plus précis en limitant les plages d’adresses IP à une région spécifique.

L’étiquette de service du service Azure Automation fournit uniquement les adresses IP utilisées pour les scénarios suivants :

* Déclencher des webhooks à partir de votre réseau virtuel
* Autoriser les Runbooks Worker hybrides ou les agents State Configuration de votre réseau virtuel à communiquer avec le service Automation

>[!NOTE]
>L’étiquette de service **GuestAndHybridManagement** ne prend actuellement pas en charge l’exécution de tâches de runbook dans un bac à sable (sandbox) Azure, uniquement directement sur un Runbook Worker hybride.

## <a name="support-for-impact-level-5-il5"></a>Prise en charge d’Impact Level 5 (IL5)

Le Runbook Worker hybride d’Azure Automation peut être utilisé dans Azure Government pour prendre en charge les charges de travail Impact Level 5 dans l’une des deux configurations suivantes :

* [Machine virtuelle isolée](../azure-government/documentation-government-impact-level-5.md#isolated-virtual-machines). Une fois déployées, elles consomment la totalité de l’hôte physique pour cette machine, fournissant ainsi le niveau d’isolation nécessaire pour prendre en charge les charges de travail IL5.

* Les [hôtes dédiés Azure](../azure-government/documentation-government-impact-level-5.md#azure-dedicated-host), qui fournissent des serveurs physiques capables d’héberger une ou plusieurs machines virtuelles et dédiés à un abonnement Azure.

>[!NOTE]
>L’isolation de calcul via le rôle Runbook Worker hybride est disponible pour les clouds commerciaux Azure et le cloud US Government. 

### <a name="update-management-addresses-for-hybrid-runbook-worker"></a>Adresses Update Management pour Runbook Worker hybride

En plus des adresses et des ports standard requis pour le Runbook Worker hybride, Update Management a des exigences supplémentaires en matière de configuration du réseau décrites dans la section relative à la [planification du réseau](./update-management/overview.md#ports).

## <a name="azure-automation-state-configuration-on-a-hybrid-runbook-worker"></a>Azure Automation State Configuration sur un Runbook Worker hybride

Vous pouvez exécuter [Azure Automation State Configuration](automation-dsc-overview.md) sur un Runbook Worker hybride. Pour gérer la configuration des serveurs qui prennent en charge le Runbook Worker hybride, vous devez les ajouter en tant que nœuds DSC. Consultez [Activer la gestion des machines par Azure Automation State Configuration](automation-dsc-onboarding.md).

## <a name="runbook-worker-limits"></a>Limites de Runbook Worker

Le nombre maximal de groupes Worker hybrides par compte Automation est de 4 000 et s’applique à la fois aux workers hybrides système et utilisateur. Si vous avez plus de 4 000 machines à gérer, nous vous recommandons de créer des comptes Automation supplémentaires.

## <a name="runbooks-on-a-hybrid-runbook-worker"></a>Runbooks sur un Runbook Worker hybride

Vous pouvez faire en sorte que les runbooks gèrent les ressources sur la machine locale ou qu’ils s’exécutent sur des ressources de l’environnement local dans lequel un Runbook Worker hybride utilisateur est déployé. Dans ce cas, vous pouvez choisir d’exécuter vos runbooks sur le Worker hybride plutôt que dans un compte Automation. Les runbooks exécutés sur un runbook Worker hybride sont identiques en structure à ceux que vous exécutez dans le compte Automation. Consultez [Exécuter des runbooks sur un runbook Worker hybride](automation-hrw-run-runbooks.md).

### <a name="hybrid-runbook-worker-jobs"></a>Travaux Runbook Worker hybride

Les travaux du Runbook Worker hybride s’exécutent sous le compte **Système** local sur Windows, ou le compte [nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) sur Linux. Azure Automation gère les travaux exécutés sur des Runbooks Workers hybrides différemment des travaux exécutés dans des bacs à sable Azure. Consultez [Environnement d’exécution de runbook](automation-runbook-execution.md#runbook-execution-environment).

Si la machine hôte du runbook Worker hybride redémarre, tous les travaux du runbook en cours d’exécution redémarrent au début ou au dernier point de contrôle pour les runbooks PowerShell Workflow. Quand un travail de runbook a redémarré plus de trois fois, il est suspendu.

### <a name="runbook-permissions-for-a-hybrid-runbook-worker"></a>Autorisations des runbooks pour un runbook Worker hybride

Étant donné qu’ils accèdent à des ressources non-Azure, les runbooks s’exécutant sur un Runbook Worker hybride utilisateur ne peuvent pas utiliser le mécanisme d’authentification généralement utilisé par les runbooks pour s’authentifier auprès des ressources Azure. Un runbook peut fournir sa propre authentification auprès des ressources locales ou en configurer une à l’aide d’[identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-arm.md#grant-your-vm-access-to-a-resource-group-in-resource-manager). Vous pouvez également spécifier un compte d’identification pour fournir un contexte utilisateur pour l’ensemble des runbooks.

## <a name="view-system-hybrid-runbook-workers"></a>Afficher les Runbooks Workers hybrides système

Une fois la fonctionnalité Update Management activée sur les machines Windows ou Linux, vous pouvez inventorier la liste du groupe Runbook Workers hybrides système dans le portail Azure. Vous pouvez afficher un maximum de 2 000 Workers sur le portail en sélectionnant l'onglet **Groupe de Workers hybrides du système** de l'option **Groupe de Workers hybrides** dans le volet de gauche du compte Automation sélectionné.

:::image type="content" source="./media/automation-hybrid-runbook-worker/system-hybrid-workers-page.png" alt-text="Page des groupes de Workers hybrides du système sur le compte Automation" border="false" lightbox="./media/automation-hybrid-runbook-worker/system-hybrid-workers-page.png":::

Si vous disposez de plus de 2 000 Workers hybrides, pour obtenir la liste complète de ceux-ci, vous pouvez exécuter le script PowerShell suivant :

```powershell
"Get-AzSubscription -SubscriptionName "<subscriptionName>" | Set-AzContext
$workersList = (Get-AzAutomationHybridWorkerGroup -ResourceGroupName "<resourceGroupName>" -AutomationAccountName "<automationAccountName>").Runbookworker
$workersList | export-csv -Path "<Path>\output.csv" -NoClobber -NoTypeInformation"
```

## <a name="next-steps"></a>Étapes suivantes

* Pour apprendre à configurer vos Runbooks afin d’automatiser les processus dans votre centre de données local ou un autre environnement cloud, consultez la rubrique [Exécuter des Runbooks sur un Runbook Worker hybride](automation-hrw-run-runbooks.md).

* Pour savoir comment résoudre les problèmes de vos Runbook Workers hybrides, consultez [Résoudre les problèmes liés à la fonctionnalité Runbook Worker hybride](troubleshoot/hybrid-runbook-worker.md#general).