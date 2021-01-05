---
title: Remédier aux serveurs Azure Automation State Configuration non conformes
description: Cet article explique comment réappliquer des configurations à la demande sur des serveurs dont l’état de la configuration a dérivé.
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.topic: conceptual
ms.date: 07/17/2019
manager: nirb
ms.openlocfilehash: 4430b8cdfe9414ddbfd7aad3c3fe7827adbc8705
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186365"
---
# <a name="remediate-noncompliant-azure-automation-state-configuration-servers"></a>Remédier aux serveurs Azure Automation State Configuration non conformes

Quand des serveurs sont inscrits auprès d’Azure Automation State Configuration, le mode de configuration est défini sur `ApplyOnly`, `ApplyandMonitor` ou `ApplyAndAutoCorrect`. Si le mode n’est pas défini sur `ApplyAndAutoCorrect`, les serveurs qui quittent un état conforme pour une raison quelconque restent non conformes jusqu’à ce qu’ils soient corrigés manuellement.

Azure Compute offre une fonctionnalité nommée Run Command qui permet aux clients d’exécuter des scripts dans des machines virtuelles.
Ce document fournit des exemples de scripts pour cette fonctionnalité lors de la correction manuelle de dérives de configuration.

## <a name="correct-drift-of-windows-virtual-machines-using-powershell"></a>Correction de la dérive des machines virtuelles Windows à l’aide de PowerShell

Vous pouvez corriger la dérive des machines virtuelles Windows à l’aide de la commande `Run`. Voir [Exécuter des scripts PowerShell dans votre machine virtuelle Windows avec la commande Run](../virtual-machines/windows/run-command.md).

Pour forcer un nœud Azure Automation State Configuration à télécharger la configuration la plus récente et à l’appliquer, utilisez l’applet de commande [Update-DscConfiguration](/powershell/module/psdesiredstateconfiguration/update-dscconfiguration).

```powershell
Update-DscConfiguration -Wait -Verbose
```

## <a name="correct-drift-of-linux-virtual-machines"></a>Corrigez la dérive des machines virtuelles Linux

Pour les machines virtuelles Linux, vous n’avez pas la possibilité d’utiliser la commande `Run`. Vous ne pouvez corriger la dérive pour ces ordinateurs qu’en répétant le processus d’inscription. 

Pour les nœuds Azure, vous pouvez corriger la dérive depuis le portail Azure ou avec des applets de commande du module Az. Pour plus d’informations sur ce processus, consultez [Activer une machine virtuelle à l’aide du portail Azure](automation-dsc-onboarding.md#enable-a-vm-using-azure-portal).

Pour les nœuds hybrides, vous pouvez corriger la dérive en utilisant les scripts Python. Voir [Performing DSC operations from the Linux computer](https://github.com/Microsoft/PowerShell-DSC-for-Linux#performing-dsc-operations-from-the-linux-computer).

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
- Pour obtenir un exemple d’utilisation d’Azure Automation State Configuration dans un pipeline de déploiement continu, consultez [Configurer un déploiement continu avec Chocolatey](automation-dsc-cd-chocolatey.md).
