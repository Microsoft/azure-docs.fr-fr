---
title: Procéder à la mise à l'échelle des hôtes de session à l'aide d'Azure Automation - Azure
description: Mise à l'échelle automatique des hôtes de session Windows Virtual Desktop à l'aide d'Azure Automation
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 12a15ab1a4c7369c448e9f65862121b03ca05bba
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078552"
---
# <a name="scale-session-hosts-using-azure-automation"></a>Procéder à la mise à l'échelle des hôtes de session à l'aide d'Azure Automation

Vous pouvez réduire le coût total de déploiement de Windows Virtual Desktop en procédant à la mise à l'échelle de vos machines virtuelles. Cela implique l'arrêt et la libération des machines virtuelles hôtes de session aux heures creuses, puis leur réactivation et leur réallocation aux heures de pointe.

Cet article présente l’outil de mise à l’échelle créé avec le compte Azure Automation et l’application logique Azure pour procéder à la mise à l’échelle automatique des machines virtuelles hôtes de session dans votre environnement Windows Virtual Desktop. Pour en savoir plus sur l'utilisation de l'outil de mise à l'échelle, consultez la section [Conditions préalables](#prerequisites).

## <a name="how-the-scaling-tool-works"></a>Fonctionnement de l'outil de mise à l'échelle

L'outil de mise à l'échelle fournit une option d'automatisation à faible coût pour les clients qui souhaitent optimiser les coûts de leurs machines virtuelles hôtes de session.

Vous pouvez utiliser l'outil de mise à l'échelle pour :

- Planifier le démarrage et l'arrêt des machines virtuelles en fonction des heures de pointe et des heures creuses
- Effectuez un scale-out des machines virtuelles en fonction du nombre de sessions par cœur de processeur.
- Effectuez un scale-in des machines virtuelles pendant les heures creuses, en continuant à exécuter le nombre minimum de machines virtuelles hôtes de session

L’outil de mise à l’échelle fonctionne à l’aide d’une combinaison de compte Azure Automation, de runbook PowerShell, de webhook et d’application logique Azure. Lorsque l’outil s’exécute, l’application logique Azure appelle un webhook pour lancer le runbook Azure Automation. Le runbook crée ensuite un travail.

Aux heures de pointe, le travail vérifie le nombre de sessions actives et la capacité en machines virtuelles de l'hôte de session en cours d'exécution pour chaque pool d'hôtes. Il utilise ces informations pour déterminer si les machines virtuelles hôtes de session en cours d’exécution peuvent prendre en charge les sessions existantes en fonction du paramètre *SessionThresholdPerCPU* défini dans le fichier **CreateOrUpdateAzLogicApp.ps1**. Si les machines virtuelles hôtes de session ne peuvent pas prendre en charge les sessions existantes, le travail démarre des machines virtuelles hôtes de session supplémentaires dans le pool d'hôtes.

>[!NOTE]
>*SessionThresholdPerCPU* ne limite pas le nombre de sessions sur la machine virtuelle. Ce paramètre détermine uniquement quand de nouvelles machines virtuelles doivent être démarrées pour équilibrer la charge des connexions. Pour limiter le nombre de sessions, vous devez suivre les instructions [Update-AzWvdHostPool](configure-host-pool-load-balancing.md#configure-breadth-first-load-balancing) et configurer le paramètre *MaxSessionLimit* en conséquence.

Aux heures creuses, le travail détermine combien de machines virtuelles hôtes de session doivent être arrêtées, conformément au paramètre *MinimumNumberOfRDSH*. Si vous affectez une valeur positive différente de zéro au paramètre *LimitSecondsToForceLogOffUser*, le travail configure les machines virtuelles hôtes de session en mode maintenance pour empêcher les nouvelles sessions de se connecter aux hôtes. Le travail envoie une notification aux utilisateurs actuellement connectés pour leur demander d’enregistrer leur travail, attend le laps de temps configuré, puis force les utilisateurs à se déconnecter. Une fois que toutes les sessions utilisateur ont été déconnectées sur la machine virtuelle hôte de session, le travail arrête la machine virtuelle. Une fois la machine virtuelle arrêtée, le travail réinitialise le mode maintenance de son hôte de session.

>[!NOTE]
>Si vous configurez manuellement la machine virtuelle hôte de session en mode maintenance, le travail ne gère pas la machine virtuelle hôte de session. Si la machine virtuelle hôte de session est en cours d’exécution et configurée en mode maintenance, elle est considérée comme non disponible, et le travail démarrera des machines virtuelles supplémentaires pour gérer la charge. Nous vous recommandons d’étiqueter les machines virtuelles Azure avant de les configurer manuellement en mode maintenance. Vous pouvez nommer l’étiquette avec le paramètre *MaintenanceTagName* lors de la création du planificateur d’application logiques Azure. Les étiquettes vous aideront à distinguer ces machines virtuelles de celles gérées par l’outil de mise à l’échelle. La définition de l’étiquette de maintenance empêche également l’outil de mise à l’échelle d’apporter des modifications à la machine virtuelle jusqu’à ce que vous supprimiez l’étiquette.

Si vous affectez la valeur zéro au paramètre *LimitSecondsToForceLogOffUser*, le travail autorise le paramètre de configuration de session des stratégies de groupe spécifiées à gérer la déconnexion des sessions utilisateur. Pour afficher ces stratégies de groupe, accédez à **Configuration ordinateur** > **Stratégies** > **Modèles d’administration** > **Composants Windows** > **Services Bureau à distance** > **Hôte de session Bureau à distance** > **Délais d’expiration des sessions**. S'il reste des sessions en cours sur une machine virtuelle hôte de session, le travail laisse la machine virtuelle hôte de session s'exécuter. S’il ne reste aucune session en cours, le travail arrête la machine virtuelle hôte de session.

À tout moment, le travail prend également en compte le paramètre *MaxSessionLimit* du pool d’hôtes pour déterminer si le nombre actuel de sessions est supérieur à 90 % de la capacité maximale. Si c’est le cas, le travail démarre des machines virtuelles hôtes de session supplémentaires.

Le travail s'exécute périodiquement sur la base d'un intervalle de périodicité défini. Vous pouvez modifier cet intervalle en fonction de la taille de votre environnement Windows Virtual Desktop, mais n’oubliez pas que le démarrage et l’arrêt des machines virtuelles peuvent prendre un certain temps ; vous devez donc tenir compte de ce délai. Nous vous recommandons de définir l'intervalle de périodicité sur : Toutes les 15 minutes.

Cela dit, l'outil présente également les limitations suivantes :

- Cette solution s’applique uniquement aux machines virtuelles hôtes de sessions multiples mises en pool.
- Cette solution permet de gérer les machines virtuelles de toutes les régions, mais ne peut être utilisée qu’au sein du même abonnement que votre compte Azure Automation et l’application logique Azure.
- La durée d’exécution maximale d’un travail dans le runbook est de trois heures. Si le démarrage ou l’arrêt des machines virtuelles dans le pool d’hôtes prend plus de temps, le travail échoue. Pour plus d’informations, consultez [Ressources partagées](../automation/automation-runbook-execution.md#fair-share).

>[!NOTE]
>L’outil de mise à l’échelle contrôle le mode d’équilibrage de charge du pool d’hôtes qu’il met actuellement à l’échelle. L’outil utilise le mode d’équilibrage de charge à largeur prioritaire pour les heures de pointe et les heures creuses.

## <a name="prerequisites"></a>Prérequis

Avant de commencer à configurer l'outil de mise à l'échelle, assurez-vous que vous disposez de ce qui suit :

- Un [pool d’hôtes Windows Virtual Desktop](create-host-pools-azure-marketplace.md)
- Machines virtuelles du pool d'hôtes de session configurées et inscrites auprès du service Windows Virtual Desktop
- Un utilisateur doté d'un [accès Contributeur](../role-based-access-control/role-assignments-portal.md) sur un abonnement Azure

L'ordinateur que vous utilisez pour déployer l'outil doit disposer de ce qui suit :

- Windows PowerShell 5.1 ou version ultérieure
- Module Microsoft AZ PowerShell

Si tout est prêt, commençons.

## <a name="create-or-update-an-azure-automation-account"></a>Créer ou mettre à jour un compte Azure Automation

>[!NOTE]
>Si vous disposez déjà d’un compte Azure Automation avec un runbook exécutant une version antérieure du script de mise à l’échelle, il vous suffit de suivre les instructions ci-dessous pour vous assurer qu’il est mis à jour.

Tout d'abord, vous devez disposer d'un compte Azure Automation pour exécuter le runbook PowerShell. Le processus décrit dans cette section est valide même si vous disposez d’un compte Azure Automation existant que vous souhaitez utiliser pour configurer le runbook PowerShell. Voici comment le configurer :

1. Ouvrez Windows PowerShell.

2. Exécutez l’applet de commande suivante pour vous connecter à votre compte Azure.

    ```powershell
    Login-AzAccount
    ```

    >[!NOTE]
    >Votre compte doit disposer de droits de contributeur sur l’abonnement Azure où vous souhaitez déployer l’outil de mise à l’échelle.

3. Exécutez la cmdlet suivante pour télécharger le script de création du compte Azure Automation :

    ```powershell
    New-Item -ItemType Directory -Path "C:\Temp" -Force
    Set-Location -Path "C:\Temp"
    $Uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzAutoAccount.ps1"
    # Download the script
    Invoke-WebRequest -Uri $Uri -OutFile ".\CreateOrUpdateAzAutoAccount.ps1"
    ```

4. Exécutez l’applet de commande suivante pour exécuter le script et créer le compte Azure Automation. Vous pouvez soit renseigner les valeurs des paramètres, soit les commenter pour utiliser leurs valeurs par défaut.

    ```powershell
    $Params = @{
         "AADTenantId"           = "<Azure_Active_Directory_tenant_ID>"   # Optional. If not specified, it will use the current Azure context
         "SubscriptionId"        = "<Azure_subscription_ID>"              # Optional. If not specified, it will use the current Azure context
         "UseARMAPI"             = $true
         "ResourceGroupName"     = "<Resource_group_name>"                # Optional. Default: "WVDAutoScaleResourceGroup"
         "AutomationAccountName" = "<Automation_account_name>"            # Optional. Default: "WVDAutoScaleAutomationAccount"
         "Location"              = "<Azure_region_for_deployment>"
         "WorkspaceName"         = "<Log_analytics_workspace_name>"       # Optional. If specified, Log Analytics will be used to configure the custom log table that the runbook PowerShell script can send logs to
    }

    .\CreateOrUpdateAzAutoAccount.ps1 @Params
    ```

5. La sortie de la cmdlet inclura un URI de webhook. Prenez note de l’URI, car vous l’utiliserez comme paramètre lors de la configuration du calendrier d’exécution pour l’application logique Azure.

6. Si vous avez spécifié le paramètre **WorkspaceName** pour Log Analytics, la sortie de l’applet de commande inclut également l’ID de l’espace de travail Log Analytics et sa clé primaire. Prenez note de l’URI, car vous le réutiliserez comme paramètre lors de la configuration du calendrier d’exécution pour l’application logique Azure.

7. Après avoir configuré votre compte Azure Automation, connectez-vous à votre abonnement Azure et vérifiez que votre compte Azure Automation et le runbook correspondant apparaissent dans le groupe de ressources spécifié, comme illustré ci-dessous :

    >[!div class="mx-imgBorder"]
    >![Illustration de la page de présentation Azure affichant le compte Azure Automation et le runbook nouvellement créés.](media/automation-account.png)

    Pour vérifier si votre webhook se trouve au bon emplacement, sélectionnez le nom de votre runbook. Accédez ensuite à la section des ressources de votre runbook, puis sélectionnez **Webhooks**.

## <a name="create-an-azure-automation-run-as-account"></a>Créer un compte d'identification Azure Automation

Maintenant que vous disposez d’un compte Azure Automation, vous devez également créer un compte d’identification Azure Automation si vous n’en avez pas encore. Ce compte permet à l’outil d’accéder à vos ressources Azure.

Un [compte d’identification Azure Automation](../automation/manage-runas-account.md) fournit l’authentification nécessaire à la gestion des ressources Azure à l’aide d’applets de commande Azure. Quand vous créez un compte d’identification, il crée un nouvel utilisateur du principal du service dans Azure Active Directory et attribue le rôle de contributeur à cet utilisateur au niveau de l’abonnement. Un compte d’identification Azure est un excellent moyen de s’authentifier de manière sécurisée avec des certificats et un nom de principal du service sans avoir à stocker un nom d’utilisateur et un mot de passe dans un objet d’informations d’identification. Pour en savoir plus sur l’authentification avec un compte d’identification, consultez [Limiter les autorisations d’un compte d’identification](../automation/manage-runas-account.md#limit-run-as-account-permissions).

Tout utilisateur membre du rôle Administrateurs des abonnements et coadministrateur de l’abonnement peut créer un compte d’identification.

Pour créer un compte d’identification dans votre compte Azure Automation :

1. Dans le portail Azure, sélectionnez **Tous les services**. Dans la liste des ressources, entrez et sélectionnez **Comptes Automation**.

2. Dans la page **Comptes Automation**, sélectionnez le nom de votre compte Azure Automation.

3. Dans le volet situé à gauche de la fenêtre, sélectionnez **Comptes d’identification** dans la section **Paramètres du compte**.

4. Sélectionnez **Compte d’identification Azure**. Lorsque le volet **Ajouter un compte d’identification Azure** apparaît, passez en revue les informations de présentation, puis sélectionnez **Créer** pour lancer le processus de création de compte.

5. Patientez quelques minutes, le temps qu'Azure crée le compte d'identification. Vous pouvez suivre la progression de la création dans le menu situé sous Notifications.

6. Au terme du processus, une ressource nommée **AzureRunAsConnection** est créée dans le compte Azure Automation spécifié. Sélectionnez **Compte d’identification Azure**. La ressource de connexion contient l'ID de l'application, l'ID du locataire, l'ID de l'abonnement et l'empreinte du certificat. Vous pouvez également trouver les mêmes informations dans la page **Connexions**. Pour accéder à cette page, dans le volet situé à gauche de la fenêtre, sélectionnez **Connexions** dans la section **Ressources partagées**, puis cliquez sur la ressource de connexion nommée **AzureRunAsConnection**.

## <a name="create-the-azure-logic-app-and-execution-schedule"></a>Créer l'application logique Azure et le calendrier d'exécution

Enfin, vous devrez créer l'application logique Azure et configurer un calendrier d'exécution pour votre nouvel outil de mise à l'échelle. Si ce n’est déjà fait, commencez par télécharger et importer le [module Desktop Virtualization PowerShell](powershell-module.md) à utiliser dans votre session PowerShell.

1. Ouvrez Windows PowerShell.

2. Exécutez l’applet de commande suivante pour vous connecter à votre compte Azure.

    ```powershell
    Login-AzAccount
    ```

3. Exécutez l’applet de commande suivante pour télécharger le script de création de l’application logique Azure.

    ```powershell
    New-Item -ItemType Directory -Path "C:\Temp" -Force
    Set-Location -Path "C:\Temp"
    $Uri = "https://raw.githubusercontent.com/Azure/RDS-Templates/master/wvd-templates/wvd-scaling-script/CreateOrUpdateAzLogicApp.ps1"
    # Download the script
    Invoke-WebRequest -Uri $Uri -OutFile ".\CreateOrUpdateAzLogicApp.ps1"
    ```

4. Exécutez le script PowerShell suivant pour créer l’application logique Azure et le calendrier d’exécution pour votre pool d’hôtes.

    >[!NOTE]
    >Vous devez exécuter ce script pour chaque pool d’hôtes que vous souhaitez mettre à l’échelle automatiquement, mais vous n’avez besoin que d’un seul compte Azure Automation.

    ```powershell
    $AADTenantId = (Get-AzContext).Tenant.Id

    $AzSubscription = Get-AzSubscription | Out-GridView -OutputMode:Single -Title "Select your Azure Subscription"
    Select-AzSubscription -Subscription $AzSubscription.Id

    $ResourceGroup = Get-AzResourceGroup | Out-GridView -OutputMode:Single -Title "Select the resource group for the new Azure Logic App"

    $WVDHostPool = Get-AzResource -ResourceType "Microsoft.DesktopVirtualization/hostpools" | Out-GridView -OutputMode:Single -Title "Select the host pool you'd like to scale"

    $LogAnalyticsWorkspaceId = Read-Host -Prompt "If you want to use Log Analytics, enter the Log Analytics Workspace ID returned by when you created the Azure Automation account, otherwise leave it blank"
    $LogAnalyticsPrimaryKey = Read-Host -Prompt "If you want to use Log Analytics, enter the Log Analytics Primary Key returned by when you created the Azure Automation account, otherwise leave it blank"
    $RecurrenceInterval = Read-Host -Prompt "Enter how often you'd like the job to run in minutes, e.g. '15'"
    $BeginPeakTime = Read-Host -Prompt "Enter the start time for peak hours in local time, e.g. 9:00"
    $EndPeakTime = Read-Host -Prompt "Enter the end time for peak hours in local time, e.g. 18:00"
    $TimeDifference = Read-Host -Prompt "Enter the time difference between local time and UTC in hours, e.g. +5:30"
    $SessionThresholdPerCPU = Read-Host -Prompt "Enter the maximum number of sessions per CPU that will be used as a threshold to determine when new session host VMs need to be started during peak hours"
    $MinimumNumberOfRDSH = Read-Host -Prompt "Enter the minimum number of session host VMs to keep running during off-peak hours"
    $MaintenanceTagName = Read-Host -Prompt "Enter the name of the Tag associated with VMs you don't want to be managed by this scaling tool"
    $LimitSecondsToForceLogOffUser = Read-Host -Prompt "Enter the number of seconds to wait before automatically signing out users. If set to 0, any session host VM that has user sessions, will be left untouched"
    $LogOffMessageTitle = Read-Host -Prompt "Enter the title of the message sent to the user before they are forced to sign out"
    $LogOffMessageBody = Read-Host -Prompt "Enter the body of the message sent to the user before they are forced to sign out"

    $AutoAccount = Get-AzAutomationAccount | Out-GridView -OutputMode:Single -Title "Select the Azure Automation account"
    $AutoAccountConnection = Get-AzAutomationConnection -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName | Out-GridView -OutputMode:Single -Title "Select the Azure RunAs connection asset"

    $WebhookURIAutoVar = Get-AzAutomationVariable -Name 'WebhookURIARMBased' -ResourceGroupName $AutoAccount.ResourceGroupName -AutomationAccountName $AutoAccount.AutomationAccountName

    $Params = @{
         "AADTenantId"                   = $AADTenantId                             # Optional. If not specified, it will use the current Azure context
         "SubscriptionID"                = $AzSubscription.Id                       # Optional. If not specified, it will use the current Azure context
         "ResourceGroupName"             = $ResourceGroup.ResourceGroupName         # Optional. Default: "WVDAutoScaleResourceGroup"
         "Location"                      = $ResourceGroup.Location                  # Optional. Default: "West US2"
         "UseARMAPI"                     = $true
         "HostPoolName"                  = $WVDHostPool.Name
         "HostPoolResourceGroupName"     = $WVDHostPool.ResourceGroupName           # Optional. Default: same as ResourceGroupName param value
         "LogAnalyticsWorkspaceId"       = $LogAnalyticsWorkspaceId                 # Optional. If not specified, script will not log to the Log Analytics
         "LogAnalyticsPrimaryKey"        = $LogAnalyticsPrimaryKey                  # Optional. If not specified, script will not log to the Log Analytics
         "ConnectionAssetName"           = $AutoAccountConnection.Name              # Optional. Default: "AzureRunAsConnection"
         "RecurrenceInterval"            = $RecurrenceInterval                      # Optional. Default: 15
         "BeginPeakTime"                 = $BeginPeakTime                           # Optional. Default: "09:00"
         "EndPeakTime"                   = $EndPeakTime                             # Optional. Default: "17:00"
         "TimeDifference"                = $TimeDifference                          # Optional. Default: "-7:00"
         "SessionThresholdPerCPU"        = $SessionThresholdPerCPU                  # Optional. Default: 1
         "MinimumNumberOfRDSH"           = $MinimumNumberOfRDSH                     # Optional. Default: 1
         "MaintenanceTagName"            = $MaintenanceTagName                      # Optional.
         "LimitSecondsToForceLogOffUser" = $LimitSecondsToForceLogOffUser           # Optional. Default: 1
         "LogOffMessageTitle"            = $LogOffMessageTitle                      # Optional. Default: "Machine is about to shutdown."
         "LogOffMessageBody"             = $LogOffMessageBody                       # Optional. Default: "Your session will be logged off. Please save and close everything."
         "WebhookURI"                    = $WebhookURIAutoVar.Value
    }

    .\CreateOrUpdateAzLogicApp.ps1 @Params
    ```

    Une fois le script exécuté, l’application logique Azure doit apparaître dans un groupe de ressources, comme illustré ci-dessous.

    >[!div class="mx-imgBorder"]
    >![Illustration de la page de présentation d’un exemple d’application logique Azure.](media/logic-app.png)

    Pour apporter des modifications au calendrier d’exécution, telles que la modification de l’intervalle de périodicité ou du fuseau horaire, accédez au planificateur de mise à l’échelle automatique de l’application logique Azure et sélectionnez **Modifier** pour accéder au concepteur d’application logique Azure.

    >[!div class="mx-imgBorder"]
    >![Image du concepteur d’application logique Azure. Les menus Périodicité et webhook qui permettent à l’utilisateur de modifier la périodicité et le fichier webhook sont ouverts.](media/logic-apps-designer.png)

## <a name="manage-your-scaling-tool"></a>Gérer votre outil de mise à l'échelle

Maintenant que vous avez créé votre outil de mise à l'échelle, vous pouvez accéder à sa sortie. Cette section décrit quelques fonctionnalités qui pourraient vous être utiles.

### <a name="view-job-status"></a>Afficher le statut de la tâche

Vous pouvez afficher un résumé de l'état de tous les travaux du runbook ou afficher un état plus approfondi d'un travail spécifique sur le portail Azure.

À droite du compte Azure Automation sélectionné, sous « Statistiques des travaux », vous pouvez afficher la liste des résumés de tous les travaux du runbook. Ouvrez la page **Travaux** sur le côté gauche de la fenêtre pour afficher l'état actuel des travaux, ainsi que les heures de début et de fin de ceux-ci.

>[!div class="mx-imgBorder"]
>![Capture d’écran de la page d’état des travaux.](media/jobs-status.png)

### <a name="view-logs-and-scaling-tool-output"></a>Afficher les journaux et la sortie de l'outil de mise à l'échelle

Vous pouvez consulter les journaux des opérations de scale-out et de scale-in en ouvrant votre runbook et en sélectionnant le travail.

Accédez au runbook dans le groupe de ressources qui héberge le compte Azure Automation et sélectionnez **Présentation**. Dans la page de présentation, sélectionnez un travail sous **Travaux récents** pour afficher la sortie de son outil de mise à l’échelle, comme illustré ci-dessous.

>[!div class="mx-imgBorder"]
>![Illustration de la fenêtre de sortie de l’outil de mise à l’échelle.](media/tool-output.png)

### <a name="check-the-runbook-script-version-number"></a>Vérifier le numéro de version du script de runbook

Vous pouvez vérifier la version du script de runbook que vous utilisez en ouvrant le fichier de runbook dans votre compte Azure Automation et en sélectionnant **Afficher**. Un script pour le runbook s’affiche sur le côté droit de l’écran. Dans le script, vous verrez le numéro de version au format `v#.#.#` sous la section `SYNOPSIS`. Vous trouverez [ici](https://github.com/Azure/RDS-Templates/blob/master/wvd-templates/wvd-scaling-script/ARM_based/basicScale.ps1#L1) le numéro de version le plus récent. Si vous ne voyez pas de numéro de version dans votre script de runbook, cela signifie que vous exécutez une version antérieure du script et que vous devez le mettre à jour immédiatement. Si vous devez mettre à jour votre script de runbook, suivez les instructions fournies dans [Créer ou mettre à jour un compte Azure Automation](#create-or-update-an-azure-automation-account).

### <a name="reporting-issues"></a>Signaler un problème

Quand vous signalez un problème, vous devez fournir les informations suivantes pour nous aider à le résoudre :

- Un journal complet obtenu sous l’onglet **Tous les journaux** du travail à l’origine du problème. Pour savoir comment obtenir le journal, suivez les instructions fournies dans [Afficher les journaux et la sortie de l’outil de mise à l’échelle](#view-logs-and-scaling-tool-output). Si le journal contient des informations sensibles ou privées, vous pouvez les supprimer avant de nous soumettre le problème.

- La version du script de runbook que vous utilisez. Pour savoir comment obtenir le numéro de version, consultez [Vérifier le numéro de version du script de runbook](#check-the-runbook-script-version-number).

- Le numéro de version de chacun des modules PowerShell suivants installés dans votre compte Azure Automation. Pour trouver ces modules, ouvrez le compte Azure Automation, sélectionnez **Modules** dans la section **Ressources partagées** dans le volet gauche de la fenêtre, puis recherchez le nom du module.
    - Az.Accounts
    - Az.Compute
    - Az.Resources
    - Az.Automation
    - OMSIngestionAPI
    - Az.DesktopVirtualization

- La date d’expiration de votre [compte d’identification](#create-an-azure-automation-run-as-account). Pour la trouver, ouvrez votre compte Azure Automation, puis sélectionnez **Comptes d’identification** sous **Paramètres du compte** dans le volet de gauche de la fenêtre. La date d’expiration doit figurer sous **Compte d’identification Azure**.

### <a name="log-analytics"></a>Log Analytics

Si vous avez décidé d’utiliser Log Analytics, vous pouvez afficher toutes les données des journaux dans un journal personnalisé nommé **WVDTenantScale_CL** sous **Journaux personnalisés** dans la vue **Journaux** de votre espace de travail Log Analytics. Nous avons listé certains exemples de requêtes qui peuvent vous être utiles.

- Pour afficher tous les journaux d’un pool d’hôtes, entrez la requête suivante :

    ```Kusto
    WVDTenantScale_CL
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Pour afficher le nombre total de machines virtuelles hôtes de session en cours d’exécution et de sessions utilisateur actives dans votre pool d’hôtes, entrez la requête suivante :

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "Number of running session hosts:"
         or logmessage_s contains "Number of user sessions:"
         or logmessage_s contains "Number of user sessions per Core:"
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Pour afficher l’état de toutes les machines virtuelles hôtes de session dans un pool d’hôtes, entrez la requête suivante :

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "Session host:"
    | where hostpoolName_s == "<host_pool_name>"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

- Pour afficher les erreurs et les avertissements, entrez la requête suivante :

    ```Kusto
    WVDTenantScale_CL
    | where logmessage_s contains "ERROR:" or logmessage_s contains "WARN:"
    | project TimeStampUTC = TimeGenerated, TimeStampLocal = TimeStamp_s, HostPool = hostpoolName_s, LineNumAndMessage = logmessage_s, AADTenantId = TenantId
    ```

## <a name="report-issues"></a>Signaler des problèmes

Les rapports de problèmes pour l’outil de mise à l’échelle sont actuellement traités par le Support Microsoft. Lorsque vous créez un rapport de problème, veillez à suivre les instructions fournies dans [Signalement de problèmes](#reporting-issues). Si vous avez des commentaires sur l’outil ou si vous souhaitez demander de nouvelles fonctionnalités, ouvrez un problème GitHub intitulé « 4-WVD-scaling-tool » sur la [page RDS GitHub](https://github.com/Azure/RDS-Templates/issues?q=is%3Aissue+is%3Aopen+label%3A4-WVD-scaling-tool).
