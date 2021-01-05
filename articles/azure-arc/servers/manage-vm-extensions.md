---
title: Gestion des extensions de machine virtuelle avec des serveur activés par Azure Arc
description: Les serveurs activés par Azure Arc peuvent gérer le déploiement d’extensions de machine virtuelle qui permettent d’effectuer des tâches d’automatisation et de configuration post-déploiement sur des machines virtuelles non Azure.
ms.date: 11/06/2020
ms.topic: conceptual
ms.openlocfilehash: 7682f6c8631bbaf2310d501d7cee6aecb2311226
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358029"
---
# <a name="virtual-machine-extension-management-with-azure-arc-enabled-servers"></a>Gestion des extensions de machine virtuelle avec des serveurs activés par Azure Arc

Les extensions de machine virtuelle sont de petites applications qui permettent d’effectuer des tâches de configuration et d’automatisation de post-déploiement sur des machines virtuelles Azure. Par exemple, si une machine virtuelle nécessite l’installation d’un logiciel, une protection antivirus ou l’exécution d’un script, vous pouvez utiliser une extension de machine virtuelle.

Les serveurs activés par Azure Arc vous permettent de déployer des extensions de machine virtuelle Azure sur des machines virtuelles Linux et Windows non Azure, ce qui simplifie la gestion de vos environnements de machines hybrides locaux, périphériques et sur d’autres clouds tout au long de leur cycle de vie. Les extensions de machine virtuelle peuvent être gérées à l’aide des méthodes suivantes sur vos machines ou serveurs hybrides gérés par des serveurs compatibles Arc :

- [Portail Azure](manage-vm-extensions-portal.md)
- [Azure CLI](manage-vm-extensions-cli.md)
- [Azure PowerShell](manage-vm-extensions-powershell.md)
- [Modèles Azure Resource Manager](manage-vm-extensions-template.md)

## <a name="key-benefits"></a>Principaux avantages

La prise en charge des extensions de machine virtuelle des serveurs activés par Azure Arc offre les avantages clés suivants :

- Utilise [Azure Automation State Configuration](../../automation/automation-dsc-overview.md) pour stocker de manière centralisée les configurations et conserver l’état souhaité des machines connectées hybrides activées via l’extension de machine virtuelle DSC.

- Collecte les données de journaux à des fins d’analyse en activant [Journaux dans Azure Monitor](../../azure-monitor/platform/data-platform-logs.md) via l’extension de machine virtuelle de l’agent Log Analytics. Cela s’avère utile pour effectuer des analyses complexes de données issues de diverses sources.

- Avec [Azure Monitor pour machines virtuelles](../../azure-monitor/insights/vminsights-overview.md), analyse les performances de vos machines virtuelles Windows et Linux et supervise leurs processus et dépendances vis-à-vis d’autres ressources et les processus externes. Pour cela, il convient d’activer les extensions de machine virtuelle de l’agent Log Analytics et de Dependency Agent.

- Télécharge et exécute des scripts sur des machines connectées hybrides à l’aide de l’extension de script personnalisé. Elle est utile pour la configuration de post-déploiement, l’installation de logiciels ou toute autre tâche de configuration ou de gestion.

- Actualisation automatique des certificats stockés dans [Azure Key Vault](../../key-vault/general/overview.md).

## <a name="availability"></a>Disponibilité

La fonctionnalité d’extension de machine virtuelle n’est disponible que dans la liste des [régions prises en charge](overview.md#supported-regions). Veillez à intégrer votre machine dans l’une de ces régions.

## <a name="extensions"></a>Extensions

Dans cette version, nous prenons en charge les extensions de machine virtuelle suivantes sur les machines Windows et Linux.

|Extension |Système d’exploitation |Serveur de publication |Informations supplémentaires |
|----------|---|----------|-----------------------|
|CustomScriptExtension |Windows |Microsoft.Compute |[Extension de script personnalisée Windows](../../virtual-machines/extensions/custom-script-windows.md)|
|DSC |Windows |Microsoft.PowerShell|[Extension DSC Windows PowerShell](../../virtual-machines/extensions/dsc-windows.md)|
|Agent Log Analytics |Windows |Microsoft.EnterpriseCloud.Monitoring |[Extension de machine virtuelle Log Analytics pour Windows](../../virtual-machines/extensions/oms-windows.md)|
|Microsoft Dependency Agent | Windows |Microsoft.Compute | [Extension de machine virtuelle Dependency Agent pour Windows](../../virtual-machines/extensions/agent-dependency-windows.md)|
|Key Vault | Windows | Microsoft.Compute | [Extension de machine virtuelle Key Vault pour Windows](../../virtual-machines/extensions/key-vault-windows.md) |
|CustomScript|Linux |Microsoft.Azure.Extension |[Extension de script personnalisé Linux version 2](../../virtual-machines/extensions/custom-script-linux.md) |
|DSC |Linux |Microsoft.OSTCExtensions |[Extension DSC PowerShell pour Linux](../../virtual-machines/extensions/dsc-linux.md) |
|Agent Log Analytics |Linux |Microsoft.EnterpriseCloud.Monitoring |[Extension de machine virtuelle Log Analytics pour Linux](../../virtual-machines/extensions/oms-linux.md) |
|Microsoft Dependency Agent | Linux |Microsoft.Compute | [Extension de machine virtuelle Dependency Agent pour Linux](../../virtual-machines/extensions/agent-dependency-linux.md) |
|Key Vault | Linux | Microsoft.Compute | [Extension de machine virtuelle Key Vault pour Linux](../../virtual-machines/extensions/key-vault-linux.md) |

Pour découvrir le package d’agent Azure Connected Machine et obtenir des détails sur le composant Agent Extension, consultez [Vue d’ensemble de l’agent](agent-overview.md#agent-component-details).

## <a name="prerequisites"></a>Prérequis

Cette fonctionnalité dépend des fournisseurs de ressources Azure suivants de votre abonnement :

- **Microsoft.HybridCompute**
- **Microsoft.GuestConfiguration**

S’ils ne sont pas déjà inscrits, suivez les étapes indiquées sous [Inscrire des fournisseurs de ressources Azure](agent-overview.md#register-azure-resource-providers).

### <a name="log-analytics-vm-extension"></a>Extension de machine virtuelle Log Analytics

L’extension de machine virtuelle de l’agent Log Analytics pour Linux nécessite l’installation de Python 2.x sur la machine cible.

### <a name="azure-key-vault-vm-extension-preview"></a>Extension de machine virtuelle Azure Key Vault (préversion)

L’extension de machine virtuelle Key Vault (préversion) ne prend pas en charge les systèmes d’exploitation Linux suivants :

- CentOS Linux 7 (x64)
- Red Hat Enterprise Linux (RHEL) 7 (x64)
- Amazon Linux 2 (x64)

Le déploiement de l’extension de machine virtuelle Key Vault (préversion) n’est pris en charge qu’avec :

- L’interface Azure CLI
- Azure PowerShell
- Modèle Azure Resource Manager

Avant de déployer l’extension, vous devez effectuer les opérations suivantes :

1. [Créer un coffre et un certificat](../../key-vault/certificates/quick-create-portal.md) (auto-signé ou à importer).

2. Accordez au serveur compatible avec Azure Arc l’accès au secret du certificat. Si vous utilisez la [préversion du RBAC](../../key-vault/general/rbac-guide.md), recherchez le nom de la ressource Azure Arc et attribuez-lui le rôle **Utilisateur de secrets de Key Vault (préversion)** . Si vous utilisez la [stratégie d’accès à Key Vault](../../key-vault/general/assign-access-policy-portal.md), affectez au secret des autorisations **Get** pour l’identité attribuée par le système de la ressource Azure Arc.

### <a name="connected-machine-agent"></a>Agent Connected Machine

Vérifiez que votre machine correspond aux [versions prises en charge](agent-overview.md#supported-operating-systems) des systèmes d’exploitation Windows et Linux pour l’agent Azure Connected Machine.

La version minimale de l’agent Connected Machine prise en charge avec cette fonctionnalité sur Windows et Linux est la version 1.0.

Pour mettre à niveau votre machine vers la version exigée de l’agent, consultez [Mettre à niveau l’agent](manage-agent.md#upgrading-agent).

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez déployer, gérer et supprimer des extensions de machine virtuelle à l’aide d’[Azure CLI](manage-vm-extensions-cli.md), d’[Azure PowerShell](manage-vm-extensions-powershell.md), du [portail Azure](manage-vm-extensions-portal.md) ou de [modèles Azure Resource Manager](manage-vm-extensions-template.md).
