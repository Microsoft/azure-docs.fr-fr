---
title: Extensions et fonctionnalités des machines virtuelles Azure
description: En savoir plus sur les extensions de machine virtuelle Azure
services: virtual-machines
author: amjads1
ms.service: virtual-machines
ms.subservice: extensions
ms.topic: article
ms.workload: infrastructure-services
ms.date: 08/03/2020
ms.author: amjads
ms.openlocfilehash: b1dd26fce2e0a761ceed211933cb79ce518905e2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965882"
---
# <a name="azure-virtual-machine-extensions-and-features"></a>Extensions et fonctionnalités des machines virtuelles Azure
Les extensions sont de petites applications qui proposent une configuration post-déploiement et une automatisation sur les machines virtuelles Azure. La plateforme Azure héberge de nombreuses extensions, notamment des applications de configuration, de surveillance, de sécurité et utilitaires de machine virtuelle. Les éditeurs prennent une application, l’enveloppent dans une extension et simplifient l’installation. Il vous suffit de définir les paramètres obligatoires. 

## <a name="how-can-i-find-what-extensions-are-available"></a>Comment savoir quelles extensions sont disponibles ?
Vous pouvez afficher les extensions disponibles en sélectionnant une machine virtuelle, puis **Extensions** dans le menu situé à gauche. Pour dérouler une liste complète des extensions, consultez [Découverte des extensions de machine virtuelle sous Linux](features-linux.md) et [Découverte des extensions de machine virtuelle sous Windows](features-windows.md).

## <a name="how-can-i-install-an-extension"></a>Comment installer une extension ?
Les extensions de machine virtuelle Azure peuvent être managées à l’aide d’Azure CLI, de PowerShell, des modèles Resource Manager et du portail Azure. Pour tester une extension, accédez au portail Azure, sélectionnez l’extension de script personnalisé, puis transmettez une commande ou un script pour exécuter les extensions.

Pour en savoir plus, consultez [Extension de script personnalisé Windows](custom-script-windows.md) et [Extension de script personnalisé Linux](custom-script-linux.md).

## <a name="how-do-i-manage-extension-application-lifecycle"></a>Comment gérer le cycle de vie de l’application de l’extension ?
Vous ne devez pas vous connecter directement à une machine virtuelle pour installer ou supprimer une extension. Le cycle de vie de l’extension Azure est managé en dehors de la machine virtuelle et est intégré à la plateforme Azure.

## <a name="anything-else-i-should-be-thinking-about-for-extensions"></a>Que savoir d’autre sur les extensions ?
Certaines applications d’extensions de machine virtuelle individuelle peuvent avoir leurs propres prérequis environnementaux, comme l’accès à un point de terminaison. Chaque extension contient un article qui explique les conditions préalables, notamment les systèmes d’exploitation pris en charge.

## <a name="troubleshoot-extensions"></a>Résoudre les problèmes liés aux extensions

Vous trouverez des informations pour résoudre les problèmes liés à chaque extension dans la section **Résolution des problèmes et support** de la vue d’ensemble de l’extension. Voici la liste des informations disponibles pour résoudre les problèmes :

| Espace de noms | Dépannage |
|-----------|-----------------|
| microsoft.azure.monitoring.dependencyagent.dependencyagentlinux | [Azure Monitor Dependency pour Linux](agent-dependency-linux.md#troubleshoot-and-support) |
| microsoft.azure.monitoring.dependencyagent.dependencyagentwindows | [Azure Monitor Dependency pour Windows](agent-dependency-windows.md#troubleshoot-and-support) |
| microsoft.azure.security.azurediskencryptionforlinux | [Azure Disk Encryption pour Linux](azure-disk-enc-linux.md#troubleshoot-and-support) |
| microsoft.azure.security.azurediskencryption | [Azure Disk Encryption pour Windows](azure-disk-enc-windows.md#troubleshoot-and-support) |
| microsoft.compute.customscriptextension | [Script personnalisé pour Windows](custom-script-windows.md#troubleshoot-and-support) |
| microsoft.ostcextensions.customscriptforlinux | [Desired State Configuration pour Linux](dsc-linux.md#troubleshoot-and-support) |
| microsoft.powershell.dsc | [Desired State Configuration pour Windows](dsc-windows.md#troubleshoot-and-support) |
| microsoft.hpccompute.nvidiagpudriverlinux | [Extension du pilote GPU NVIDIA pour Linux](hpccompute-gpu-linux.md#troubleshoot-and-support) |
| microsoft.hpccompute.nvidiagpudriverwindows | [Extension du pilote GPU NVIDIA pour Windows](hpccompute-gpu-windows.md#troubleshoot-and-support) |
| microsoft.azure.security.iaasantimalware | [Extension anti-programme malveillant pour Windows](iaas-antimalware-windows.md#troubleshoot-and-support) |
| microsoft.enterprisecloud.monitoring.omsagentforlinux | [Azure Monitor pour Linux](oms-linux.md#troubleshoot-and-support)
| microsoft.enterprisecloud.monitoring.microsoftmonitoringagent | [Azure Monitor pour Windows](oms-windows.md#troubleshoot-and-support) |
| stackify.linuxagent.extension.stackifylinuxagentextension | [Stackify Retrace pour Linux](stackify-retrace-linux.md#troubleshoot-and-support) |
| vmaccessforlinux.microsoft.ostcextensions | [Réinitialiser un mot de passe sous Linux](vmaccess.md#troubleshoot-and-support) |
| microsoft.recoveryservices.vmsnapshot | [Capture instantanée pour Linux](vmsnapshot-linux.md#troubleshoot-and-support) |
| microsoft.recoveryservices.vmsnapshot | [Capture instantanée pour Windows](vmsnapshot-windows.md#troubleshoot-and-support) |


## <a name="next-steps"></a>Étapes suivantes
* Pour en savoir plus sur le fonctionnement des extensions et de l’agent Linux, consultez [Fonctionnalités et extensions des machines virtuelles Azure pour Linux](features-linux.md).
* Pour en savoir plus sur le fonctionnement des extensions et de l’agent invité de Windows, consultez [Fonctionnalités et extensions des machines virtuelles Azure pour Windows](features-windows.md).  
* Pour installer l’agent invité Windows, consultez [Vue d’ensemble de l’agent de machine virtuelle Azure pour Windows](agent-windows.md).  
* Pour installer l’agent Linux, consultez [Vue d’ensemble de l’agent de machine virtuelle Azure pour Linux](agent-linux.md).  

