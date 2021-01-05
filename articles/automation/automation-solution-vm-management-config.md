---
title: Configurer Azure Automation Start/Stop VMs during off-hours
description: Cet article explique comment configurer la fonctionnalité Start/Stop VMs during off-hours pour prendre en charge différents scénarios ou cas d’usage.
services: automation
ms.subservice: process-automation
ms.date: 06/01/2020
ms.topic: conceptual
ms.openlocfilehash: 552cac01cd492229a19bdbc297665eb7d3ccb401
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96751289"
---
# <a name="configure-startstop-vms-during-off-hours"></a>Configurer Start/Stop VMs during off-hours

Cet article explique comment configurer la fonctionnalité [Start/Stop VMs during off-hours](automation-solution-vm-management.md) pour prendre en charge les scénarios décrits. Vous pouvez également apprendre à :

* [Configurer les notifications par e-mail](#configure-email-notifications)
* [Ajouter une machine virtuelle](#add-a-vm)
* [Exclure une machine virtuelle](#exclude-a-vm)
* [Modifier les planifications de démarrage et d’arrêt](#modify-the-startup-and-shutdown-schedules)

## <a name="scenario-1-startstop-vms-on-a-schedule"></a><a name="schedule"></a>Scénario 1 : Démarrer/arrêter les machines virtuelles selon une planification

Ce scénario est la configuration par défaut quand vous déployez la fonctionnalité Start/Stop VMs during off-hours pour la première fois. Par exemple, vous pouvez le configurer pour arrêter toutes les machines virtuelles d’un abonnement quand vous quittez le travail le soir, et pour les démarrer le matin quand vous arrivez au bureau. Quand vous configurez les planifications **Scheduled-StartVM** et **Scheduled-StopVM** pendant le déploiement, celles-ci démarrent et arrêtent les machines virtuelles cibles. 

La configuration de la fonctionnalité pour arrêter simplement les machines virtuelles est prise en charge. Consultez la section [Modifier les planifications de démarrage et d’arrêt](#modify-the-startup-and-shutdown-schedules) pour savoir comment configurer une planification personnalisée.

> [!NOTE]
> Le fuseau horaire utilisé par votre fonctionnalité est votre fuseau horaire actuel quand vous configurez le paramètre du temps de planification. Toutefois, Azure Automation le stocke au format UTC dans Azure Automation. Il est inutile de convertir un fuseau horaire car cela est géré pendant le déploiement de la machine.

Pour contrôler les machines virtuelles qui sont dans l’étendue, configurez les variables : `External_Start_ResourceGroupNames`, `External_Stop_ResourceGroupNames`et `External_ExcludeVMNames`.

Vous pouvez activer cette solution pour prendre un charge un abonnement et un groupe de ressources, ou bien une liste spécifique de machines virtuelles, mais pas les deux.

### <a name="target-the-start-and-stop-actions-against-a-subscription-and-resource-group"></a>Configurer le démarrage et l’arrêt pour un abonnement et un groupe de ressources

1. Configurez les variables `External_Stop_ResourceGroupNames` et `External_ExcludeVMNames` pour spécifier les machines virtuelles cibles.

2. Activez et mettez à jour les planifications **Scheduled-StartVM** et **Scheduled-StopVM**.

3. Exécutez le runbook **ScheduledStartStop_Parent** avec le champ du paramètre **ACTION** défini sur **start** et le champ du paramètre **WHATIF** défini sur True pour afficher un aperçu des modifications.

### <a name="target-the-start-and-stop-action-by-vm-list"></a>Configurer le démarrage et l’arrêt pour une liste de machines virtuelles

1. Exécutez le runbook **ScheduledStartStop_Parent** en définissant **ACTION** avec la valeur **start**.

2. Ajoutez une liste de machines virtuelles séparées par des virgules (sans espaces) dans le champ de paramètre **VMList**. Voici un exemple de liste : `vm1,vm2,vm3`.

3. Affectez au champ de paramètre **WHATIF** la valeur True pour afficher un aperçu de vos modifications.

4. Configurez la variable `External_ExcludeVMNames` avec une liste de machines virtuelles séparées par des virgules (VM1, VM2, VM3), sans espaces entre les valeurs séparées par des virgules.

5. Ce scénario ne respecte pas les variables `External_Start_ResourceGroupNames` et `External_Stop_ResourceGroupnames`. Pour ce scénario, vous devez créer votre propre planification Automation. Pour plus d’informations, consultez [Planification d'un Runbook dans Azure Automation](shared-resources/schedules.md).

    > [!NOTE]
    > La valeur correspondant aux **Noms de groupe de ressources cibles** est stockée comme valeur pour `External_Start_ResourceGroupNames` et `External_Stop_ResourceGroupNames`. Pour plus de granularité, vous pouvez modifier chacune de ces variables pour cibler des groupes de ressources différents. Utilisez `External_Start_ResourceGroupNames` pour l'action de démarrage et `External_Stop_ResourceGroupNames` pour l’action d’arrêt. Les machines virtuelles sont automatiquement ajoutées aux planifications de démarrage et d’arrêt.

## <a name="scenario-2-startstop-vms-in-sequence-by-using-tags"></a><a name="tags"></a>Scénario 2 : Démarrer/arrêter les machines virtuelles dans un certain ordre en utilisant des balises

Dans un environnement comprenant deux composants ou plus sur plusieurs machines virtuelles prenant en charge une charge de travail distribuée, il est important de prendre en charge l’ordre de démarrage et d’arrêt des composants. 

### <a name="target-the-start-and-stop-actions-against-a-subscription-and-resource-group"></a>Configurer le démarrage et l’arrêt pour un abonnement et un groupe de ressources

1. Ajoutez des balises `sequencestart` et `sequencestop` avec une valeur entière positive aux machines virtuelles ciblées dans des variables `External_Start_ResourceGroupNames` et `External_Stop_ResourceGroupNames`. Les actions de démarrage et d’arrêt sont effectuées dans l’ordre croissant. Pour en savoir plus sur le balisage d’une machine virtuelle, consultez [Baliser une machine virtuelle Windows dans Azure](../virtual-machines/tag-portal.md) et [Baliser une machine virtuelle Linux dans Azure](../virtual-machines/tag-cli.md).

2. Modifiez les planifications **Sequenced-StartVM** et **StopVM-Sequenced** avec la date et l’heure répondant à vos besoins et activez la planification.

3. Exécutez le runbook **SequencedStartStop_Parent** avec **ACTION** défini sur **start** et **WHATIF** défini sur True pour afficher un aperçu des modifications.

4. Affichez un aperçu de l’action et apportez les modifications nécessaires avant l’implémentation des machines virtuelles de production. Une fois prêt, vous pouvez exécuter manuellement le runbook avec le paramètre défini sur **False** ou laisser les planifications de Automation **Sequenced-StartVM** et **Sequenced-StopVM** s’exécuter automatiquement d’après votre planification établie.

### <a name="target-the-start-and-stop-actions-by-vm-list"></a>Configurer le démarrage et l’arrêt pour une liste de machines virtuelles

1. Ajoutez des balises `sequencestart` et `sequencestop` contenant une valeur entière positive aux machines virtuelles que vous prévoyez d’ajouter au paramètre `VMList`.

2. Exécutez le runbook **SequencedStartStop_Parent** en définissant **ACTION** avec la valeur **start**.

3. Ajoutez une liste de machines virtuelles séparées par des virgules (sans espaces) dans le champ de paramètre **VMList**. Voici un exemple de liste : `vm1,vm2,vm3`.

4. Définissez **WHATIF** sur True pour afficher un aperçu de vos modifications. 

5. Configurez la variable `External_ExcludeVMNames` avec une liste de machines virtuelles séparées par des virgules, sans espaces entre les valeurs.

6. Ce scénario ne respecte pas les variables `External_Start_ResourceGroupNames` et `External_Stop_ResourceGroupnames`. Pour ce scénario, vous devez créer votre propre planification Automation. Pour plus d’informations, consultez [Planification d'un Runbook dans Azure Automation](shared-resources/schedules.md).

7. Affichez un aperçu de l’action et apportez les modifications nécessaires avant l’implémentation des machines virtuelles de production. Lorsque vous êtes prêt, exécutez manuellement **monitoring-and-diagnostics/monitoring-action-groupsrunbook** avec le paramètre défini sur **False**. Vous pouvez également laisser à Automation le soin de planifier l'exécution automatique de **Sequenced-StartVM** et **Sequenced-StopVM** à la suite de la planification prescrite.

## <a name="scenario-3-start-or-stop-automatically-based-on-cpu-utilization"></a><a name="cpuutil"></a>Scénario 3 : Démarrer ou arrêter automatiquement les machines virtuelles en fonction de l’utilisation de l’UC

Start/Stop VMs during off-hours peut aider à gérer le coût de l’exécution de machines virtuelles Azure Resource Manager et classiques dans votre abonnement en évaluant les ordinateurs qui ne sont pas utilisés pendant les périodes creuses, par exemple après les heures de travail, et les arrêtant automatiquement si l’utilisation du processeur est inférieure à un pourcentage spécifié.

Par défaut, la solution est préconfigurée de manière à évaluer les mesures du pourcentage d’utilisation de l’UC pour vérifier si l’utilisation moyenne est de 5 % ou moins. Ce scénario est contrôlé par les variables suivantes et peut être modifié si leurs valeurs par défaut ne répondent pas à vos besoins :

* `External_AutoStop_MetricName`
* `External_AutoStop_Threshold`
* `External_AutoStop_TimeAggregationOperator`
* `External_AutoStop_TimeWindow`
* `External_AutoStop_Frequency`
* `External_AutoStop_Severity`

Vous pouvez activer et cibler l’action sur un charge un abonnement et un groupe de ressources, ou bien cibler une liste spécifique de machines virtuelles.

Lorsque vous exécutez le runbook **AutoStop_CreateAlert_Parent**, celui-ci vérifie que l’abonnement, les groupes de ressources et les machines virtuelles ciblés existent. Si les machines virtuelles existent, le runbook appelle ensuite le runbook **AutoStop_CreateAlert_Child** pour chaque machine virtuelle vérifiée par le runbook parent. Ce runbook enfant :

* Crée une règle d’alerte métrique pour chaque machine virtuelle vérifiée.
* Déclenche le runbook **AutoStop_VM_Child** pour une machine virtuelle spécifique si l’utilisation de l’UC est inférieure au seuil configuré pour l’intervalle de temps spécifié. 
* Tente d’arrêter la machine virtuelle.

### <a name="target-the-autostop-action-against-all-vms-in-a-subscription"></a>Cibler l’action d’arrêt automatique sur toutes les machines virtuelles d’un abonnement

1. Assurez-vous que la variable `External_Stop_ResourceGroupNames` est vide ou définie sur * (caractère générique).

2. [Facultatif] Si vous souhaitez exclure certaines machines virtuelles de l’arrêt automatique, vous pouvez ajouter une liste de noms de machines virtuelles séparés par des virgules à la variable `External_ExcludeVMNames`.

3. Activez l’exécution de la planification **Schedule_AutoStop_CreateAlert_Parent** pour créer les règles d’alerte métrique Arrêter une machine virtuelle requises pour toutes les machines virtuelles de votre abonnement. L'exécution de ce type de planification vous permet de créer des règles d’alerte métrique lorsque de nouvelles machines virtuelles seront ajoutées à l’abonnement.

### <a name="target-the-autostop-action-against-all-vms-in-a-resource-group-or-multiple-resource-groups"></a>Cibler les actions d’arrêt automatique sur toutes les machines virtuelles dans un ou plusieurs groupes de ressources

1. Ajoutez une liste de noms de groupes de ressources séparés par des virgules à la variable `External_Stop_ResourceGroupNames`.

2. Si vous souhaitez exclure certaines machines virtuelles de l’arrêt automatique, vous pouvez ajouter une liste de noms de machines virtuelles, séparés par une virgule, à la variable `External_ExcludeVMNames`.

3. Activez l’exécution de la planification **Schedule_AutoStop_CreateAlert_Parent** pour créer les règles d’alerte métrique Arrêter une machine virtuelle requises pour toutes les machines virtuelles de vos groupes de ressources. L'exécution de cette opération vous permet de créer des règles d’alerte métrique lorsque de nouvelles machines virtuelles seront ajoutées aux groupes de ressources.

### <a name="target-the-autostop-action-to-a-list-of-vms"></a>Cibler l’action d’arrêt automatique vers une liste de machines virtuelles

1. Créez une [paramètre](shared-resources/schedules.md#create-a-schedule) et liez-la au runbook **AutoStop_CreateAlert_Parent**, en ajoutant une liste de noms de machines virtuelles, séparés par une virgule, au paramètre `VMList`.

2. Si vous souhaitez exclure certaines machines virtuelles de l’arrêt automatique, vous pouvez ajouter une liste de noms de machines virtuelles séparés par des virgules (sans espaces), à la variable `External_ExcludeVMNames`.

## <a name="configure-email-notifications"></a>Configurer les notifications par e-mail

Pour modifier les notifications par e-mail une fois la solution Start/Stop VMs during off-hours déployée, modifiez le groupe d’actions créé lors du déploiement.  

> [!NOTE]
> Les abonnements dans le Cloud Azure Government ne prennent pas en charge la fonctionnalité e-mail de cette solution.

1. Dans le portail Azure, accédez à **Surveiller**, puis **Groupes d'actions**. Sélectionnez le groupe d’actions nommé **StartStop_VM_Notication**.

    :::image type="content" source="media/automation-solution-vm-management/azure-monitor.png" alt-text="Capture d’écran montrant la page Analyse – Groupes d’actions.":::

2. Dans la page StartStop_VM_Notification, cliquez sur **Modifier les détails** sous **Détails**. La page E-mail/SMS/Push/Voix s’ouvre. Mettez à jour l’adresse e-mail, puis cliquez sur **OK** pour enregistrer vos modifications.

    :::image type="content" source="media/automation-solution-vm-management/change-email.png" alt-text="Capture d’écran montrant la page E-mail/SMS/Push/Voice affichant un exemple d’adresse e-mail mise à jour.":::

    Vous pouvez également ajouter des actions supplémentaires au groupe d’actions. Pour plus d’informations sur les groupes d’actions, consultez [Groupes d’actions](../azure-monitor/platform/action-groups.md).

Voici un exemple d’e-mail envoyé lorsque la solution arrête les machines virtuelles.

:::image type="content" source="media/automation-solution-vm-management/email.png" alt-text="Capture d’écran montrant un exemple de message électronique envoyé lorsque la fonctionnalité arrête des machines virtuelles.":::

## <a name="add-or-exclude-vms"></a><a name="add-exclude-vms"></a>Ajouter ou exclure des machines virtuelles

La fonctionnalité vous permet d’ajouter des machines virtuelles à cibler ou à exclure. 

### <a name="add-a-vm"></a>Ajouter une machine virtuelle

Il existe deux façons de s’assurer qu’une machine virtuelle est incluse lors de l’exécution de la fonctionnalité :

* Chaque [runbook](automation-solution-vm-management.md#runbooks) parent de la solution a un paramètre `VMList`. Vous pouvez passer d’une liste de noms de machines virtuelles, séparés par des virgules (sans espaces), à ce paramètre lors de la planification du runbook parent approprié pour votre situation. Ces machines virtuelles seront alors incluses lors de l’exécution de la solution.

* Pour sélectionner plusieurs machines virtuelles, définissez `External_Start_ResourceGroupNames` et `External_Stop_ResourceGroupNames` à l'aide des noms de groupes de ressources contenant les machines virtuelles que vous souhaitez démarrer ou arrêter. Vous pouvez également définir les variables sur une valeur de `*` pour que la solution s’exécute sur tous les groupes de ressources dans l’abonnement.

### <a name="exclude-a-vm"></a>Exclure une machine virtuelle

Pour exclure une machine virtuelle de Stop/Start VMs during off-hours, vous pouvez ajouter son nom à la variable `External_ExcludeVMNames`. Cette variable est une liste de machines virtuelles, séparées par des virgules (sans espaces), à exclure de la fonctionnalité. Le nombre de machines virtuelles est limité à 140. Si vous ajoutez plus de 140 machines virtuelles à la liste, celles définies comme exclues pourraient être démarrées ou arrêtées par inadvertance.

## <a name="modify-the-startup-and-shutdown-schedules"></a>Modifier les planifications de démarrage et d’arrêt

La gestion de la planification du démarrage et de l’arrêt dans cette solution suit la procédure indiquée dans [Schedule a runbook in Azure Automation (Planification d’un Runbook dans Azure Automation)](shared-resources/schedules.md). Des planifications distinctes sont requises pour démarrer et arrêter des machines virtuelles.

Il est possible de configurer la solution pour simplement arrêter les machines virtuelles à une certaine heure. Dans ce scénario vous créez simplement une planification d’arrêt, sans planification de démarrage correspondante. 

1. Assurez-vous d’avoir ajouté les groupes de ressources des machines virtuelles à arrêter dans la variable `External_Stop_ResourceGroupNames`.

2. Créez votre propre planification pour l’heure à laquelle vous souhaitez arrêter les machines virtuelles.

3. Accédez au runbook **ScheduledStartStop_Parent** et cliquez sur **Planification**. Cela vous permet de sélectionner la planification créée à l’étape précédente.

4. Sélectionnez **Paramètres et valeurs pour l’exécution** et définissez le champ **ACTION** sur **Stop**.

5. Sélectionnez **OK** pour enregistrer vos modifications.

## <a name="next-steps"></a>Étapes suivantes

* Pour surveiller la fonctionnalité pendant l’opération, consultez [Les journaux de requêtes à partir de Start/Stop VMs during off-hours](automation-solution-vm-management-logs.md).
* Pour gérer les problèmes lors de la gestion des machines virtuelles, consultez [Résoudre les problèmes liés à Start/Stop VMs during off-hours](troubleshoot/start-stop-vm.md).
