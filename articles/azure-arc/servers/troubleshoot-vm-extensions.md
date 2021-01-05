---
title: Résoudre les problèmes d'extensions de machine virtuelle liés aux serveurs Azure Arc
description: Cet article explique comment détecter et résoudre les problèmes d'extensions de machine virtuelle Azure qui surviennent avec les serveurs Azure Arc.
ms.date: 09/24/2020
ms.topic: conceptual
ms.openlocfilehash: ffd7db5ff7da3d7f60762117f80d7b9b5af6f646
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91339052"
---
# <a name="troubleshoot-arc-enabled-servers-vm-extension-issues"></a>Résoudre les problèmes d'extensions de machine virtuelle liés aux serveurs Azure Arc

Cet article fournit des informations sur la résolution des problèmes qui peuvent survenir lors de la tentative de déploiement ou de suppression d'extensions de machine virtuelle Azure sur des serveurs Azure Arc. Pour obtenir des informations générales, consultez [Gérer et utiliser les extensions de machine virtuelle Azure](./manage-vm-extensions.md).

## <a name="general-troubleshooting"></a>Résolution générale des problèmes

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure.

La procédure de résolution des problèmes ci-après s’applique à toutes les extensions de machine virtuelle.

1. Pour consulter le journal de l’agent invité, examinez l’activité au moment où votre extension a été provisionnée dans `%SystemDrive%\ProgramData\GuestConfig\ext_mgr_logs` pour Windows, et pour Linux sous `/var/lib/GuestConfig/ext_mgr_logs`.

2. Pour plus de détails, consultez les journaux de l’extension en question dans `%SystemDrive%\ProgramData\GuestConfig\extension_logs\<Extension>` pour Windows. La sortie d’extension est journalisée dans un fichier pour chaque extension installée sur Linux sous `/var/lib/GuestConfig/extension_logs`.

3. Consultez les sections de résolution des problèmes dans la documentation de l’extension concernée pour vous renseigner sur les codes d’erreur, les problèmes connus, etc. Vous trouverez des informations de dépannage supplémentaires pour chaque extension dans la section **Résolution des problèmes et support** de la vue d’ensemble de l’extension. Vous trouverez notamment une description des codes d’erreur écrits dans le journal. Le [tableau des extensions](manage-vm-extensions.md#extensions) propose des liens vers les articles consacrés aux différentes extensions.

4. Examinez les journaux d’activité système. Vérifiez si d’autres opérations sont susceptibles d’avoir interféré avec l’extension, par exemple une longue installation d’une autre application nécessitant un accès exclusif au gestionnaire de package.

## <a name="troubleshooting-specific-extension-scenarios"></a>Résolution des problèmes liés à des scénarios d'extension spécifiques

### <a name="vm-insights"></a>Insights de machine virtuelle

- Lors de l'activation d'Insights de machine virtuelle pour un serveur Azure Arc, les agents Dependency et Log Analytics sont installés. Sur un ordinateur lent ou dont la connexion réseau est lente, des délais d'attente peuvent survenir pendant le processus d'installation. Microsoft prend actuellement des mesures pour y remédier dans l'agent Connected Machine. Dans l'intervalle, une nouvelle tentative d'installation peut aboutir.

### <a name="log-analytics-agent-for-linux"></a>Agent Log Analytics pour Linux

- L'agent Log Analytics version 1.13.9 (dont la version d'extension correspondante est 1.13.15) ne marque pas correctement les données chargées avec l'ID de ressource du serveur Azure Arc. Les journaux sont envoyés au service, mais lorsque vous essayez d'afficher les données du serveur activé sélectionné après avoir sélectionné **Journaux** ou **Insights**, aucune donnée n'est renvoyée. Vous pouvez consulter ses données en exécutant des requêtes à partir des journaux Azure Monitor ou à partir d'Azure Monitor pour les machines virtuelles ; celles-ci sont étendues à l'espace de travail.

- Certaines distributions ne sont actuellement pas prises en charge par l'agent Log Analytics pour Linux. L'agent requiert l'installation de dépendances supplémentaires, dont Python 2. Reportez-vous à la matrice de prise en charge et aux prérequis, disponibles [ici](../../azure-monitor/platform/agents-overview.md#supported-operating-systems).

- Le code d'erreur 52 qui apparaît dans le message d'état indique qu'il manque une dépendance. Pour plus d'informations sur la dépendance manquante, consultez la sortie et les journaux.

- Si une installation échoue, consultez la section **Résolution des problèmes et support technique** de la présentation de l'extension. Dans la plupart des cas, un code d'erreur est inclus dans le message d'état. En ce qui concerne l'agent Log Analytics pour Linux, les messages d'état sont décrits [ici](../../virtual-machines/extensions/oms-linux.md#troubleshoot-and-support), avec des informations générales sur la résolution des problèmes liés à cette extension de machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas ici ou que vous ne pouvez pas le résoudre, utilisez un des canaux suivants pour obtenir une aide supplémentaire :

- Obtenez des réponses d'experts Azure par le biais de [Microsoft Q&A](/answers/topics/azure-arc.html).

- Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Le Support Azure fournit à la communauté Azure des réponses, un support technique et des experts.

- Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**.
