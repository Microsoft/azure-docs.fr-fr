---
title: Présentation de l’agent Log Analytics
description: Cette rubrique aide à comprendre comment collecter des données et effectuer le monitoring d’ordinateurs hébergés dans un environnement Azure, local ou tout autre environnement cloud avec Log Analytics.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/21/2020
ms.openlocfilehash: 8563f734db8524d6e90171bb2272723f14533055
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185927"
---
# <a name="log-analytics-agent-overview"></a>Présentation de l’agent Log Analytics
L’agent Azure Log Analytics collecte la télémétrie de machines virtuelles Windows et Linux dans n’importe quel cloud, sur des ordinateurs locaux et sur des ordinateurs surveillés par [System Center Operations Manager](/system-center/scom/), puis envoie les données collectées à votre espace de travail Log Analytics dans Azure Monitor. L’agent Log Analytics prend également en charge des analyses et d’autres services dans Azure Monitor comme [Azure Monitor pour machines virtuelles](../insights/vminsights-enable-overview.md), [Azure Security Center](../../security-center/index.yml) et [Azure Automation](../../automation/automation-intro.md). Cet article propose une présentation détaillée des exigences en matière d’agent, de système et de réseau, ainsi que des méthodes de déploiement.

> [!NOTE]
> Vous pouvez également voir l’agent Log Analytics appelé Microsoft Monitoring Agent (MMA) ou agent Linux OMS.

## <a name="comparison-to-azure-diagnostics-extension"></a>Comparaison avec l’extension Diagnostics Azure
L’[extension Diagnostics Azure](diagnostics-extension-overview.md) dans Azure Monitor peut également être utilisée pour collecter des données de supervision à partir du système d’exploitation invité des machines virtuelles Azure. Vous pouvez choisir d’utiliser l’un ou l’autre, ou les deux en fonction de vos besoins. Pour une comparaison détaillée des agents Azure Monitor, consultez [Vue d’ensemble des agents Azure Monitor](agents-overview.md). 

Les principales différences à prendre en compte sont les suivantes :

- L’extension Diagnostics Azure peut être utilisée uniquement avec des machines virtuelles Azure. L’agent Log Analytics peut être utilisé avec des machines virtuelles dans Azure, dans d’autres clouds et en local.
- L’extension Diagnostics Azure envoie des données à Stockage Azure, à [Azure Monitor Metrics](data-platform-metrics.md) (Windows uniquement) et à Event Hubs. L’agent Log Analytics envoie des données aux [journaux Azure Monitor](data-platform-logs.md).
- L’agent Log Analytics est requis pour les [solutions](../monitor-reference.md#insights-and-core-solutions), [Azure Monitor pour machines virtuelles](../insights/vminsights-overview.md) et d’autres services tels qu’[Azure Security Center](../../security-center/index.yml).

## <a name="costs"></a>Coûts
Il n’y a aucun coût lié à l’agent Log Analytics, mais vous pouvez être facturé pour les données ingérées. Consultez [Gérer l’utilisation et les coûts avec les journaux d’activité Azure Monitor](manage-cost-storage.md) pour obtenir des informations détaillées sur la tarification des données collectées dans un espace de travail Log Analytics.

## <a name="supported-operating-systems"></a>Systèmes d’exploitation pris en charge

 Pour obtenir la liste des versions de système d’exploitation Windows et Linux que l’agent Log Analytics prend en charge, consultez [Systèmes d’exploitation pris en charge](agents-overview.md#supported-operating-systems). 


## <a name="data-collected"></a>Données collectées
Vous pouvez configurer un espace de travail Log Analytics pour qu’il collecte les types de données répertoriés dans le tableau suivant à partir de tous les agents connectés. Consultez [Quels sont les éléments supervisés par Azure Monitor ?](../monitor-reference.md) pour obtenir la liste des analyses, des solutions et d’autres solutions qui utilisent l’agent Log Analytics pour collecter d’autres types de données.

| source de données | Description |
| --- | --- |
| [Journaux d’événements Windows](data-sources-windows-events.md) | Informations envoyées au système de journalisation des événements Windows. |
| [Syslog](data-sources-syslog.md)                     | Informations envoyées au système de journalisation des événements Linux. |
| [Performances](data-sources-performance-counters.md)  | Valeurs numériques mesurant les performances de différents aspects du système d’exploitation et des charges de travail. |
| [Journaux d’activité IIS](data-sources-iis-logs.md)                 | Informations sur l’utilisation des sites web IIS s’exécutant sur le système d’exploitation invité. |
| [Journaux d’activité personnalisés](data-sources-custom-logs.md)           | Événements issus de fichiers texte sur des ordinateurs Windows et Linux. |

## <a name="data-destinations"></a>Destination des données
L’agent Log Analytics envoie des données à un espace de travail Log Analytics dans Azure Monitor. L’agent Windows peut être multirésident afin d’envoyer des données à plusieurs espaces de travail et groupes d’administration Operations Manager. L’agent Linux ne peut envoyer qu’à une seule destination, qu’il s’agisse d’un espace de travail ou d’un groupe d’administration.

## <a name="other-services"></a>Autres services
L’agent pour Linux et Windows ne se connecte pas uniquement à Azure Monitor. D’autres services, tels qu’Azure Security Center et Azure Sentinel dépendent de l’agent et de son espace de travail Log Analytics connecté. L’agent prend également en charge Azure Automation pour héberger le rôle de travail Runbook Worker hybride, ainsi que d’autres services tels que [Change Tracking](../../automation/change-tracking/overview.md), [Update Management](../../automation/update-management/overview.md) et [Azure Security Center](../../security-center/security-center-introduction.md). Pour plus d’informations sur le rôle Runbook Worker hybride, consultez la page [Runbook Worker hybride d’Azure Automation](../../automation/automation-hybrid-runbook-worker.md).  

## <a name="workspace-and-management-group-limitations"></a>Limitations d’espace de travail et de groupe d’administration

Pour plus d’informations sur la connexion d’un agent à un groupe d’administration Operations Manager, consultez [Configurer l’agent pour qu’il communique avec un groupe d’administration Operations Manager](agent-manage.md#configure-agent-to-report-to-an-operations-manager-management-group).

* Les agents Windows peuvent se connecter à jusqu’à quatre espaces de travail, même s’ils sont connectés à un groupe d’administration System Center Operations Manager.
* L’agent Linux ne prend pas en charge l’hébergement multiple et ne peut se connecter qu’à un seul espace de travail ou groupe d’administration.


## <a name="security-limitations"></a>Limitations de sécurité

* Les agents Windows et Linux prennent en charge la [norme FIPS 140](/windows/security/threat-protection/fips-140-validation), mais [les autres types de renforcement peuvent ne pas être pris en charge](agent-linux.md#supported-linux-hardening).


## <a name="installation-options"></a>Options d'installation

En fonction de vos besoins, il existe plusieurs méthodes pour installer l’agent Log Analytics et connecter votre ordinateur à Azure Monitor. Les sections suivantes répertorient les méthodes possibles pour différents types de machine virtuelle.
> [!NOTE]
> Le clonage d’un ordinateur avec l’agent Log Analytics déjà configuré n’est pas pris en charge. Si l’agent a déjà été associé à un espace de travail, cela ne fonctionnera pas pour les « images finales ».

### <a name="azure-virtual-machine"></a>Machine virtuelle Azure

- [Azure Monitor pour machines virtuelles](../insights/vminsights-enable-overview.md) fournit plusieurs méthodes activant des agents à grande échelle. Cela inclut l’installation de l’agent Log Analytics et de l’agent Dependency. 
- [Azure Security Center peut approvisionner l’agent Log Analytics](../../security-center/security-center-enable-data-collection.md) sur toutes les machines virtuelles Azure prises en charge et sur toutes les nouvelles créées si vous l’autorisez à surveiller les menaces et les failles de sécurité.
- L’extension de machine virtuelle Log Analytics pour [Windows](../../virtual-machines/extensions/oms-windows.md) ou [Linux](../../virtual-machines/extensions/oms-linux.md) peut être installée avec le portail Azure, Azure CLI, Azure PowerShell ou un modèle Azure Resource Manager.
- Effectuez l’installation pour les machines virtuelles Azure individuelles [manuellement à partir du portail Azure](../learn/quick-collect-azurevm.md?toc=%2fazure%2fazure-monitor%2ftoc.json).


### <a name="windows-virtual-machine-on-premises-or-in-another-cloud"></a>Machine virtuelle Windows locale ou dans un autre cloud 

- [Installez manuellement](agent-windows.md) l’agent à partir de la ligne de commande.
- Automatisez l’installation à l’aide d’[Azure Automation DSC](agent-windows.md#install-agent-using-dsc-in-azure-automation).
- Utilisez un [Modèle Resource Manager avec Azure Stack](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/MicrosoftMonitoringAgent-ext-win).

### <a name="linux-virtual-machine-on-premises-or-in-another-cloud"></a>Machine virtuelle Linux locale ou dans un autre cloud

- [Installez manuellement](../learn/quick-collect-linux-computer.md) l’agent qui appelle un script wrapper hébergé sur GitHub.
- System Center Operations Manager|[Intégrez Operations Manager avec Azure Monitor](./om-agents.md) pour transférer les données collectées à partir d’ordinateurs Windows rendant compte à un groupe d’administration.

## <a name="workspace-id-and-key"></a>ID et clé d’espace de travail
Quelle que soit la méthode d’installation utilisée, vous aurez besoin de l’ID et de la clé d’espace de travail pour l’espace de travail Log Analytics auquel l’agent se connectera. Sélectionnez l’espace de travail dans le menu **Espaces de travail Log Analytics** dans le portail Azure. Sélectionnez ensuite **Gestion des agents** dans la section **Paramètres**. 

[![Détails de l’espace de travail](media/log-analytics-agent/workspace-details.png)](media/log-analytics-agent/workspace-details.png#lightbox)

## <a name="tls-12-protocol"></a>Protocole TLS 1.2

Pour garantir la sécurité des données en transit vers les journaux d’activité Azure Monitor, nous vous encourageons vivement à configurer l’agent de façon à utiliser au moins le protocole TLS (Transport Layer Security) 1.2. Les versions antérieures de TLS/SSL (Secure Sockets Layer) se sont avérées vulnérables et bien qu’elles fonctionnent encore pour assurer la compatibilité descendante, elles sont **déconseillées**.  Pour plus d’informations, passez en revue [Envoi sécurisé de données via TLS 1.2](data-security.md#sending-data-securely-using-tls-12). 

## <a name="network-requirements"></a>Configuration requise pour le réseau
L’agent pour Linux et Windows communique en sortie avec le service Azure Monitor sur le port TCP 443. Si la machine se connecte via un pare-feu ou un serveur proxy pour communiquer sur Internet, consultez les exigences ci-dessous pour comprendre la configuration réseau requise. Si vos stratégies de sécurité informatique n’autorisent pas les ordinateurs du réseau à se connecter à Internet, vous pouvez configurer une [passerelle Log Analytics](gateway.md), puis configurer l’agent pour qu’il se connecte à Azure Monitor via la passerelle. L’agent peut ensuite recevoir des informations de configuration et envoyer les données collectées.

![Schéma de communication de l’agent Log Analytics](./media/log-analytics-agent/log-analytics-agent-01.png)

Le tableau suivant répertorie les informations de configuration du proxy et du pare-feu requises pour permettre aux agents Linux et Windows de communiquer avec les journaux d’activité Azure Monitor.

### <a name="firewall-requirements"></a>Configuration requise du pare-feu

|Ressource de l'agent|Ports |Sens |Ignorer l’inspection HTTPS|
|------|---------|--------|--------|   
|*.ods.opinsights.azure.com |Port 443 |Règle de trafic sortant|Oui |  
|*.oms.opinsights.azure.com |Port 443 |Règle de trafic sortant|Oui |  
|*.blob.core.windows.net |Port 443 |Règle de trafic sortant|Oui |
|\* .azure-automation.net |Port 443 |Règle de trafic sortant|Oui |

Pour obtenir les informations relatives au pare-feu nécessaires pour Azure Government, consultez [Azure Government Monitoring + Management](../../azure-government/compare-azure-government-global-azure.md#azure-monitor). 

Si vous envisagez d’utiliser le Runbook Worker hybride Azure Automation pour vous connecter et vous inscrire auprès du service Automation afin d’utiliser des runbooks et des solutions de gestion dans votre environnement, il doit avoir accès au numéro de port et aux URL décrites dans la section [Configurer votre réseau pour le Runbook Worker hybride](../../automation/automation-hybrid-runbook-worker.md#network-planning). 

### <a name="proxy-configuration"></a>Configuration du proxy

L’agent Windows et Linux prend en charge la communication par l’intermédiaire d’un serveur proxy ou de la passerelle Log Analytics vers Azure Monitor, au moyen du protocole HTTPS.  L’authentification anonyme et l’authentification de base (nom d’utilisateur/mot de passe) sont prises en charge.  Pour l’agent Windows connecté directement au service, la configuration du proxy est indiquée pendant l’installation ou [après le déploiement](agent-manage.md#update-proxy-settings), dans le panneau de configuration ou avec PowerShell.  

Pour l’agent Linux, le serveur proxy est indiqué pendant ou [après l’installation](agent-manage.md#update-proxy-settings) en modifiant le fichier de configuration du fichier proxy.conf.  La valeur de configuration du proxy de l’agent Linux adopte la syntaxe suivante :

`[protocol://][user:password@]proxyhost[:port]`

|Property| Description |
|--------|-------------|
|Protocol | https |
|utilisateur | Nom d’utilisateur facultatif pour l’authentification du proxy |
|mot de passe | Mot de passe facultatif pour l’authentification du proxy |
|proxyhost | Adresse ou nom de domaine complet du serveur proxy/de la passerelle Log Analytics |
|port | Numéro de port facultatif pour le serveur proxy/la passerelle Log Analytics |

Par exemple : `https://user01:password@proxy01.contoso.com:30443`

> [!NOTE]
> Si vous utilisez des caractères spéciaux comme « \@ » dans votre mot de passe, vous obtenez une erreur de connexion au proxy, car la valeur est mal analysée.  Pour contourner ce problème, encodez le mot de passe dans l’URL à l’aide d’un outil comme [URLDecode](https://www.urldecoder.org/).  



## <a name="next-steps"></a>Étapes suivantes

* Consultez les [sources de données](agent-data-sources.md) pour connaître les sources de données disponibles pour collecter des données sur votre système Windows ou Linux. 
* Découvrez les [requêtes dans les journaux](../log-query/log-query-overview.md) pour analyser les données collectées à partir de sources de données et de solutions. 
* Découvrez les [solutions de surveillance](../insights/solutions.md) qui ajoutent des fonctionnalités à Azure Monitor et collectent également des données dans l’espace de travail Log Analytics.