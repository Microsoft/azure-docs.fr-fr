---
title: Meilleures pratiques Azure Automanage pour machines virtuelles
description: Découvrez les meilleures pratiques Azure Automanage pour machines virtuelles en lien avec les services qui sont automatiquement intégrés et configurés pour vous.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 09/04/2020
ms.author: deanwe
ms.openlocfilehash: 208c1ba6a0fb059640b3f8bb19bb4bc90e5eb116
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183649"
---
# <a name="azure-automanage-for-virtual-machines-best-practices"></a>Meilleures pratiques Azure Automanage pour machines virtuelles


Ces services Azure sont automatiquement intégrés pour vous lorsque vous utilisez Automanage pour machines virtuelles. Ils sont essentiels pour notre livre blanc sur les meilleures pratiques, que vous pouvez trouver dans notre [Cloud Adoption Framework](/azure/cloud-adoption-framework/manage/azure-server-management).

Pour tous ces services, nous les intégrons et les configurons automatiquement, surveillons leur dérive et corrigeons toute dérive détectée. Pour en savoir plus sur ce processus, consultez [Azure Automanage pour machines virtuelles](automanage-virtual-machines.md).


## <a name="participating-services"></a>Services participant

|Service    |Description    |Profils pris en charge<sup>1</sup>    |Préférences prises en charge<sup>1</sup>    |
|-----------|---------------|----------------------|-------------------------|
|Surveillance des insights de machine virtuelle    |Azure Monitor pour machines virtuelles surveille les performances et l’intégrité de vos machines virtuelles, y compris de leurs processus en cours d’exécution et de leurs dépendances vis-à-vis d’autres ressources. En savoir [plus](../azure-monitor/insights/vminsights-overview.md).    |Meilleures pratiques pour les machines virtuelles Azure – Production    |Non    |
|Backup    |Le service Sauvegarde Azure propose des sauvegardes indépendantes et isolées pour éviter une destruction involontaire des données de vos machines virtuelles. En savoir [plus](../backup/backup-azure-vms-introduction.md). Les frais sont calculés en fonction du nombre et de la taille des machines virtuelles protégées. En savoir [plus](https://azure.microsoft.com/pricing/details/backup/).    |Meilleures pratiques pour les machines virtuelles Azure – Production    |Oui    |
|Azure Security Center    |Azure Security Center est un système de gestion de la sécurité de l’infrastructure unifié qui renforce la posture de sécurité de vos centres de données et fournit une protection avancée contre les menaces pour vos charges de travail hybrides dans le cloud. En savoir [plus](../security-center/security-center-introduction.md).  Automanage configure l’abonnement dans lequel votre machine virtuelle réside en offre de niveau gratuit d’Azure Security Center. Si votre abonnement est déjà intégré à Azure Security Center, Automanage ne le reconfigure pas.    |Meilleures pratiques pour les machines virtuelles Azure – Production, meilleures pratiques pour les machines virtuelles Azure – Dev/Test    |Non    |
|Microsoft Antimalware    |Microsoft Antimalware pour Azure offre une protection en temps réel qui permet d’identifier et de supprimer les virus, logiciels espions et autres logiciels malveillants. Il fournit des alertes lorsqu’un logiciel malveillant ou indésirable connu essaye de s’installer ou de s’exécuter sur vos systèmes Azure. En savoir [plus](../security/fundamentals/antimalware.md). |Meilleures pratiques pour les machines virtuelles Azure – Production, meilleures pratiques pour les machines virtuelles Azure – Dev/Test    |Oui    |
|Update Management    |Vous pouvez utiliser la solution Update Management d’Azure Automation pour gérer les mises à jour du système d’exploitation de vos machines virtuelles. Vous pouvez rapidement évaluer l’état des mises à jour disponibles sur toutes les machines d’agent et gérer le processus d’installation des mises à jour nécessaires pour les serveurs. En savoir [plus](../automation/update-management/overview.md).    |Meilleures pratiques pour les machines virtuelles Azure – Production, meilleures pratiques pour les machines virtuelles Azure – Dev/Test    |Non    |
|Suivi des modifications et inventaire    |La fonctionnalité Suivi des modifications et inventaire combine les fonctions de suivi des modifications et d’inventaire pour vous permettre d’effectuer le suivi des modifications apportées à l’infrastructure des machines virtuelles et des serveurs. Le service prend en charge le suivi des modifications sur les services, les démons, les logiciels, le registre et les fichiers de votre environnement pour vous aider à diagnostiquer les changements indésirables et à déclencher des alertes. La prise en charge de l’inventaire vous permet d’interroger les ressources intégrées pour voir les applications installées et d’autres éléments de configuration.  En savoir [plus](../automation/change-tracking/overview.md).    |Meilleures pratiques pour les machines virtuelles Azure – Production, meilleures pratiques pour les machines virtuelles Azure – Dev/Test    |Non    |
|Configuration d’invité Azure    | La stratégie Guest Configuration est utilisée pour superviser la configuration et signaler la conformité de la machine. Le service de gestion automatique installe les [lignes de base de sécurité Windows](/windows/security/threat-protection/windows-security-baselines) à l’aide de l’extension de configuration invitée. En savoir [plus](../governance/policy/concepts/guest-configuration.md).    |Meilleures pratiques pour les machines virtuelles Azure – Production, meilleures pratiques pour les machines virtuelles Azure – Dev/Test    |Non    |
|Compte Azure Automation    |Azure Automation prend en charge la gestion tout au long du cycle de vie de votre infrastructure et des applications. En savoir [plus](../automation/automation-intro.md).    |Meilleures pratiques pour les machines virtuelles Azure – Production, meilleures pratiques pour les machines virtuelles Azure – Dev/Test    |Non    |
|Espace de travail Log Analytics    |Azure Monitor stocke des données de journal dans un espace de travail Log Analytics, qui est une ressource Azure et un conteneur où les données sont collectées, agrégées, et qui sert de limite d’administration. En savoir [plus](../azure-monitor/platform/design-logs-deployment.md).    |Meilleures pratiques pour les machines virtuelles Azure – Production, meilleures pratiques pour les machines virtuelles Azure – Dev/Test    |Non    |


<sup>1</sup> Les profils de configuration sont disponibles lorsque vous activez Automanage. En savoir [plus](automanage-virtual-machines.md#configuration-profiles). Vous pouvez également ajuster les paramètres par défaut du profil de configuration et définir vos propres préférences dans les contraintes liées aux meilleures pratiques.


## <a name="next-steps"></a>Étapes suivantes

Essayez d’activer le service Automanage pour machines virtuelles dans le portail Azure.

> [!div class="nextstepaction"]
> [Activer le service Automanage pour machines virtuelles dans le portail Azure](quick-create-virtual-machines-portal.md)