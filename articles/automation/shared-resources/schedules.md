---
title: Gérer les planifications dans Azure Automation
description: Cet article explique comment créer et utiliser une planification dans Azure Automation.
services: automation
ms.subservice: shared-capabilities
ms.date: 09/10/2020
ms.topic: conceptual
ms.openlocfilehash: 844a45c9b596522b949443b6edc311308da7806c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90004610"
---
# <a name="manage-schedules-in-azure-automation"></a>Gérer les planifications dans Azure Automation

Pour planifier le démarrage d'un Runbook dans Azure Automation à une heure spécifiée, liez-le à une ou plusieurs planifications. Une planification peut être configurée pour une exécution des Runbooks ponctuelle ou régulière à des heures et jours précis dans le portail Azure. Vous pouvez également planifier une exécution hebdomadaire ou mensuelle, ou des jours spécifiques de la semaine ou du mois, ou encore un jour particulier du mois. Un Runbook peut être lié à plusieurs planifications et une planification peut avoir plusieurs Runbooks qui lui sont liés.

> [!NOTE]
> Azure Automation prend en charge l’heure d’été et la planifie de manière appropriée pour les opérations d’automatisation.

> [!NOTE]
> Actuellement, les planifications ne sont pas activées pour les configurations Azure Automation DSC.

## <a name="powershell-cmdlets-used-to-access-schedules"></a>Cmdlets PowerShell utilisées pour accéder aux planifications

Les cmdlets du tableau suivant créent et gèrent les planifications Automation à l’aide de PowerShell. Elles sont fournies dans le cadre des [modules Az](modules.md#az-modules).

| Applets de commande | Description |
|:--- |:--- |
| [Get-AzAutomationSchedule](/powershell/module/Az.Automation/Get-AzAutomationSchedule) |Récupère une planification. |
| [Get-AzAutomationScheduledRunbook](/powershell/module/az.automation/get-azautomationscheduledrunbook) |Récupère les Runbooks planifiés. |
| [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) |Crée une planification. |
| [Register-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Register-AzAutomationScheduledRunbook) |Associe un Runbook à une planification. |
| [Remove-AzAutomationSchedule](/powershell/module/Az.Automation/Remove-AzAutomationSchedule) |Supprime une planification. |
| [Set-AzAutomationSchedule](/powershell/module/Az.Automation/Set-AzAutomationSchedule) |Définit les propriétés d'une planification existante. |
| [Unregister-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Unregister-AzAutomationScheduledRunbook) |Dissocie un Runbook d'une planification. |

## <a name="create-a-schedule"></a>Créer une planification

Vous pouvez utiliser le Portail Azure ou PowerShell afin de créer une planification pour vos runbooks. Pour éviter d’affecter vos runbooks et les processus qu’ils automatisent, vous devez tout d’abord tester les runbooks ayant des planifications associées avec un compte Automation dédié au test. Un test valide que vos runbooks planifiés continuent de fonctionner correctement. En cas de problème, vous pouvez le corriger et appliquer les modifications requises avant de migrer la version de runbook mise à jour en production.

> [!NOTE]
> Votre compte Automation n’obtient pas automatiquement les nouvelles versions des modules, sauf si vous les avez mis à jour manuellement en sélectionnant l’option [Mettre à jour les modules Azure](../automation-update-azure-modules.md) du panneau **Modules**. Azure Automation utilise les modules les plus récents de votre compte Automation durant l’exécution d’un nouveau travail planifié. 

### <a name="create-a-new-schedule-in-the-azure-portal"></a>Créer une planification à l’aide du Portail Azure

1. À partir de votre compte Automation, dans le volet gauche, sélectionnez **Planifications** sous **Ressources partagées**.
2. Dans la page **Planifications**, sélectionnez **Ajouter une planification**.
3. Dans la page **Nouvelle planification**, saisissez un nom et éventuellement une description pour la nouvelle planification.

    >[!NOTE]
    >Actuellement, les planifications Automation ne prennent pas en charge l’utilisation de caractères spéciaux dans le nom de la planification.
    >

4. Indiquez si la planification doit s’exécuter une seule fois ou selon un calendrier récurrent en sélectionnant **Une fois** ou **Récurrent**. Si vous sélectionnez **Une fois**, indiquez une heure de début, puis sélectionnez **Créer**. Si vous sélectionnez **Récurrent**, spécifiez une heure de début. Pour **Répéter chaque**, sélectionnez la fréquence à laquelle vous souhaitez que le runbook se répète. Sélectionnez par heure, jour, semaine ou mois.

    * Si vous sélectionnez **Semaine**, vous pouvez choisir parmi les jours de la semaine. Sélectionnez autant de jours que vous le voulez. La première exécution de votre planification a lieu le premier jour sélectionné, après l’heure de début. Par exemple, si vous planifiez des runbooks pour le week-end, sélectionnez samedi et dimanche.

    ![Configuration d’une planification récurrente pour le week-end](../media/schedules/week-end-weekly-recurrence.png)

    * Si vous sélectionnez **Mois**, vous disposez de différentes options. Pour l’option **Occurrences mensuelles**, sélectionnez **Jours du mois** ou **Jours de la semaine**. Si vous sélectionnez **Jours du mois**, un calendrier s’affiche pour vous permettre de choisir autant de jours que vous le souhaitez. Si par exemple vous choisissez comme date le 31, et que le mois en cours ne dispose pas de 31e jour, la planification ne sera pas exécutée. Si vous souhaitez que la planification s’exécute le dernier jour, sélectionnez **Oui** sous **Run on last day of month** (Exécution le dernier jour du mois). Si vous sélectionnez **Jours de la semaine**, l’option **Tous les** s’affiche. Choisissez **Premier**, **Deuxième**, **Troisième**, **Quatrième** ou **Dernier**. Enfin, choisissez un jour pour la répétition.

    ![Planification mensuelle le premier, le quinzième et le dernier jours du mois](../media/schedules/monthly-first-fifteenth-last.png)

5. Quand vous avez terminé, sélectionnez **Créer**.

### <a name="create-a-new-schedule-with-powershell"></a>Créer une planification avec PowerShell

Vous utilisez la cmdlet [New-AzAutomationSchedule](/powershell/module/Az.Automation/New-AzAutomationSchedule) pour créer des planifications. Spécifiez l’heure de début de la planification et indiquez sa fréquence d’exécution. Les exemples suivants montrent comment créer de nombreux scénarios de planification différents.

>[!NOTE]
>Actuellement, les planifications Automation ne prennent pas en charge l’utilisation de caractères spéciaux dans le nom de la planification.
>

#### <a name="create-a-one-time-schedule"></a>Créer une planification unique

L’exemple suivant crée une planification unique.

```azurepowershell-interactive
$TimeZone = ([System.TimeZoneInfo]::Local).Id
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule01" -StartTime "23:00" -OneTime -ResourceGroupName "ResourceGroup01" -TimeZone $TimeZone
```

#### <a name="create-a-recurring-schedule"></a>Créer une planification périodique

L’exemple suivant montre comment créer une planification périodique qui s’exécute tous les jours à 13 h pendant un an.

```azurepowershell-interactive
$StartTime = Get-Date "13:00:00"
$EndTime = $StartTime.AddYears(1)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule02" -StartTime $StartTime -ExpiryTime $EndTime -DayInterval 1 -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-weekly-recurring-schedule"></a>Créer une planification périodique hebdomadaire

L’exemple suivant montre comment créer une planification hebdomadaire qui est exécutée uniquement en semaine.

```azurepowershell-interactive
$StartTime = (Get-Date "13:00:00").AddDays(1)
[System.DayOfWeek[]]$WeekDays = @([System.DayOfWeek]::Monday..[System.DayOfWeek]::Friday)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Schedule03" -StartTime $StartTime -WeekInterval 1 -DaysOfWeek $WeekDays -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-weekly-recurring-schedule-for-weekends"></a>Créer une planification périodique hebdomadaire pour le week-end

L’exemple suivant montre comment créer une planification hebdomadaire qui s’exécute uniquement le week-end.

```azurepowershell-interactive
$StartTime = (Get-Date "18:00:00").AddDays(1)
[System.DayOfWeek[]]$WeekendDays = @([System.DayOfWeek]::Saturday,[System.DayOfWeek]::Sunday)
New-AzAutomationSchedule -AutomationAccountName "ContosoAutomation" -Name "Weekends 6PM" -StartTime $StartTime -WeekInterval 1 -DaysOfWeek $WeekendDays -ResourceGroupName "ResourceGroup01"
```

#### <a name="create-a-recurring-schedule-for-the-first-fifteenth-and-last-days-of-the-month"></a>Créer une planification périodique pour le premier, le quinzième et le dernier jours du mois

L’exemple suivant montre comment créer une planification périodique qui s’exécute le premier, le quinzième et le dernier jours du mois.

```azurepowershell-interactive
$StartTime = (Get-Date "18:00:00").AddDays(1)
New-AzAutomationSchedule -AutomationAccountName "TestAzureAuto" -Name "1st, 15th and Last" -StartTime $StartTime -DaysOfMonth @("One", "Fifteenth", "Last") -ResourceGroupName "TestAzureAuto" -MonthInterval 1
```

## <a name="link-a-schedule-to-a-runbook"></a>Lier une planification à un Runbook

Un Runbook peut être lié à plusieurs planifications et une planification peut avoir plusieurs Runbooks qui lui sont liés. Si un Runbook possède des paramètres, vous pouvez leur fournir des valeurs. Vous devez fournir des valeurs pour tous les paramètres obligatoires et vous pouvez aussi fournir des valeurs pour les paramètres facultatifs. Ces valeurs sont utilisées à chaque démarrage du Runbook par cette planification. Vous pouvez attacher le même Runbook à une autre planification et spécifier différentes valeurs de paramètre.

### <a name="link-a-schedule-to-a-runbook-with-the-azure-portal"></a>Lier une planification à un runbook avec le Portail Azure

1. Dans le Portail Azure, à partir de votre compte Automation, sélectionnez **Runbooks** sous **Automatisation des processus**.
1. Sélectionnez le nom du Runbook à planifier.
1. Si le runbook n’est pas lié à une planification, vous avez la possibilité de créer une planification ou de le lier à une planification existante.
1. Si le runbook comprend des paramètres, vous pouvez sélectionner l’option **Modifier les paramètres d’exécution (par défaut : Azure)** et le volet **Paramètres** s’affiche. Vous pouvez y entrer les informations correspondantes.

### <a name="link-a-schedule-to-a-runbook-with-powershell"></a>Lier une planification à un runbook avec PowerShell

Utilisez la cmdlet [Register-AzAutomationScheduledRunbook](/powershell/module/Az.Automation/Register-AzAutomationScheduledRunbook) pour lier une planification. Vous pouvez spécifier les valeurs des paramètres du Runbook avec le paramètre Parameters. Pour plus d’informations sur la spécification des valeurs des paramètres, consultez [Démarrage d’un Runbook dans Azure Automation](../start-runbooks.md).
L’exemple suivant montre comment lier une planification à un runbook à l’aide d’une cmdlet Azure Resource Manager avec des paramètres.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$runbookName = "Test-Runbook"
$scheduleName = "Sample-DailySchedule"
$params = @{"FirstName"="Joe";"LastName"="Smith";"RepeatCount"=2;"Show"=$true}
Register-AzAutomationScheduledRunbook –AutomationAccountName $automationAccountName `
–Name $runbookName –ScheduleName $scheduleName –Parameters $params `
-ResourceGroupName "ResourceGroup01"
```

## <a name="schedule-runbooks-to-run-more-frequently"></a>Planifier l’exécution plus fréquente des runbooks

L’intervalle le plus fréquent pour lequel une planification dans Azure Automation peut être configurée est d’une heure. Si vous avez besoin d’exécuter les planifications plus fréquemment, il existe deux options :

* Créer un [webhook](../automation-webhooks.md) pour le runbook et utiliser [Azure Logic Apps](../../logic-apps/logic-apps-overview.md) pour appeler le webhook. Azure Logic Apps permet une plus grande précision lors de la définition d’une planification.

* Créer quatre planifications éloignées de 15 minutes et exécutées une fois par heure. Ce scénario permet au runbook de s’exécuter toutes les 15 minutes avec différentes planifications.

## <a name="disable-a-schedule"></a>Désactiver une planification

Lorsque vous désactivez une planification, les Runbooks qui y sont liés ne s'exécutent plus sur cette planification. Vous pouvez désactiver manuellement une planification ou affecter un délai d’expiration aux planifications selon une certaine fréquence lors de leur création. Lorsque le délai d'expiration est atteint, la planification est désactivée.

### <a name="disable-a-schedule-from-the-azure-portal"></a>Désactiver une planification à partir du Portail Azure

1. Dans votre compte Automation, dans le volet gauche, sélectionnez **Planifications** sous **Ressources partagées**.
1. Sélectionnez le nom d’une planification pour ouvrir le volet Détails.
1. Remplacez **Activé** par **Non**.

> [!NOTE]
> Si vous souhaitez désactiver une planification dont l’heure de début est située dans le passé, vous devez remplacer sa date de début par une date ultérieure avant de l’enregistrer.

### <a name="disable-a-schedule-with-powershell"></a>Désactiver une planification avec PowerShell

Utilisez la cmdlet [Set-AzAutomationSchedule](/powershell/module/Az.Automation/Set-AzAutomationSchedule) pour modifier les propriétés d’une planification existante. Pour désactiver la planification, spécifiez False pour le paramètre `IsEnabled`.

L’exemple suivant montre comment désactiver la planification d’un runbook à l’aide d’une cmdlet Azure Resource Manager.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Set-AzAutomationSchedule –AutomationAccountName $automationAccountName `
–Name $scheduleName –IsEnabled $false -ResourceGroupName "ResourceGroup01"
```

## <a name="remove-a-schedule"></a>Supprimer une planification

Lorsque vous êtes prêt à supprimer vos planifications, vous pouvez utiliser le Portail Azure ou PowerShell. N’oubliez pas que vous pouvez uniquement supprimer une planification qui a été désactivée comme décrit dans la section précédente.

### <a name="remove-a-schedule-using-the-azure-portal"></a>Supprimer une planification à l’aide du portail Azure

1. Dans votre compte Automation, dans le volet gauche, sélectionnez **Planifications** sous **Ressources partagées**.
2. Sélectionnez le nom d’une planification pour ouvrir le volet Détails.
3. Cliquez sur **Supprimer**.

### <a name="remove-a-schedule-with-powershell"></a>Supprimer une planification avec PowerShell

Vous pouvez utiliser la cmdlet `Remove-AzAutomationSchedule` comme indiqué ci-dessous pour supprimer une planification existante.

```azurepowershell-interactive
$automationAccountName = "MyAutomationAccount"
$scheduleName = "Sample-MonthlyDaysOfMonthSchedule"
Remove-AzAutomationSchedule -AutomationAccountName $automationAccountName `
-Name $scheduleName -ResourceGroupName "ResourceGroup01"
```

## <a name="next-steps"></a>Étapes suivantes

* Afin d’en savoir plus sur les cmdlets utilisées pour accéder aux planifications, consultez [Gérer les modules dans Azure Automation](modules.md).
* Pour obtenir des informations générales sur les runbooks, consultez [Exécution d’un runbook dans Azure Automation](../automation-runbook-execution.md).