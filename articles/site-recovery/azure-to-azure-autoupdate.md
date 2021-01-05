---
title: Mise à jour automatique du service Mobilité dans Azure Site Recovery
description: Vue d’ensemble de la mise à jour automatique du service Mobilité lors de la réplication de machines virtuelles Azure à l’aide d’Azure Site Recovery.
services: site-recovery
author: sideeksh
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 04/02/2020
ms.author: sideeksh
ms.openlocfilehash: 42d3c74229ab7eeec0ac716073a9e631775fd002
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96187338"
---
# <a name="automatic-update-of-the-mobility-service-in-azure-to-azure-replication"></a>Mise à jour automatique du service Mobilité dans la réplication interne à Azure

Azure Site Recovery utilise une cadence de publication mensuelle pour résoudre les problèmes et améliorer les fonctionnalités existantes ou en ajouter de nouvelles. Pour rester à jour avec le service, vous devez planifier le déploiement de correctifs chaque mois. Pour éviter la surcharge associée à chaque mise à niveau, vous pouvez autoriser Site Recovery à gérer les mises à jour de composant.

Comme mentionné dans l’[architecture de récupération d’urgence interne à Azure](azure-to-azure-architecture.md), le service Mobility est installé sur toutes les machines virtuelles pour lesquelles la réplication est activée, d’une région Azure à une autre. Lorsque vous utilisez des mises à jour automatiques, chaque nouvelle version met à jour l’extension du service Mobilité.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="how-automatic-updates-work"></a>Fonctionnement des mises à jour automatiques

Lorsque vous utilisez Site Recovery pour gérer les mises à jour, il déploie un runbook global (utilisé par les services Azure) via un compte Automation créé dans le même abonnement que le coffre. Chaque coffre utilise un seul compte Automation. Pour chaque machine virtuelle présente dans un coffre, le runbook vérifie les mises à jour automatiques actives. Si une version plus récente de l’extension du service Mobility est disponible, la mise à jour est installée.

La planification du runbook par défaut intervient tous les jours à 12h00 dans le fuseau horaire de la zone géographique de la machine virtuelle répliquée. Vous pouvez également modifier la planification du runbook via le compte Automation.

> [!NOTE]
> À partir du [correctif cumulatif 35](site-recovery-whats-new.md#updates-march-2019), vous pouvez choisir un compte Automation existant à utiliser pour les mises à jour. Avant le correctif cumulatif 35, Site Recovery créait le compte Automation par défaut. Vous pouvez uniquement sélectionner cette option lorsque vous activez la réplication pour une machine virtuelle. Elle n’est pas disponible pour une machine virtuelle pour laquelle la réplication est déjà activée. Le paramètre que vous sélectionnez s’applique à toutes les machines virtuelles Azure protégées dans le même coffre.

L’activation des mises à jour automatiques ne nécessite pas un redémarrage de vos machines virtuelles Azure ni n’affecte la réplication en cours.

La facturation du travail dans le compte Automation est basée sur le nombre de minutes d’exécution de travail réellement utilisées au cours d’un mois. L’exécution du travail prend quelques secondes à une minute environ chaque jour et elle est couverte en tant qu’unités gratuites. Par défaut, 500 minutes sont incluses en tant qu’unités gratuites pour un compte Automation, comme indiqué dans le tableau suivant :

| Unités gratuites incluses (chaque mois) | Price |
|---|---|
| Exécution de travail de 500 minutes | 0,14 ₹/minute

## <a name="enable-automatic-updates"></a>Activation des mises à jour automatiques

Site Recovery peut gérer les mises à jour d’extension de différentes manières :

- [Gérer dans le cadre de l’étape d’activation de la réplication](#manage-as-part-of-the-enable-replication-step)
- [Activer/désactiver les paramètres de mise à jour de l’extension dans le coffre](#toggle-the-extension-update-settings-inside-the-vault)
- [Gérer les mises à jour manuellement](#manage-updates-manually)

### <a name="manage-as-part-of-the-enable-replication-step"></a>Gérer dans le cadre de l’étape d’activation de la réplication

Lorsque vous activez la réplication pour une machine virtuelle [à partir de la vue de la machine virtuelle](azure-to-azure-quickstart.md) ou [à partir du coffre Recovery Services](azure-to-azure-how-to-enable-replication.md), vous pouvez autoriser Site Recovery à gérer les mises à jour de l’extension Site Recovery ou les gérer manuellement.

:::image type="content" source="./media/azure-to-azure-autoupdate/enable-rep.png" alt-text="Paramètres d’extension":::

### <a name="toggle-the-extension-update-settings-inside-the-vault"></a>Activer/désactiver les paramètres de mise à jour de l’extension dans le coffre

1. À partir du coffre Recovery Services, accédez à **Gérer** > **Infrastructure Site Recovery**.
1. Sous **Pour les machines virtuelles Azure** > **Paramètres de mise à jour de l’extension** > **Autoriser Site Recovery à gérer**, sélectionnez **Activer**.

   Pour gérer l’extension manuellement, sélectionnez **Désactiver**.

1. Sélectionnez **Enregistrer**.

:::image type="content" source="./media/azure-to-azure-autoupdate/vault-toggle.png" alt-text="Paramètres de mise à jour de l’extension":::

> [!IMPORTANT]
> Lorsque vous sélectionnez **Autoriser Site Recovery à gérer**, le paramètre est appliqué à toutes les machines virtuelles présentes dans le coffre.

> [!NOTE]
> Chaque option vous informe du compte Automation utilisé pour la gestion des mises à jour. Si vous activez cette fonctionnalité dans un coffre pour la première fois, un nouveau compte Automation est créé par défaut. Vous pouvez également personnaliser le paramètre et choisir un compte Automation existant. Une fois la fonctionnalité définie, toutes les actions suivantes pour activer la réplication dans le même coffre utiliseront le compte Automation sélectionné. Actuellement, le menu déroulant répertorie uniquement les comptes Automation qui se trouvent dans le même groupe de ressources que le coffre.

> [!IMPORTANT]
> Le script suivant doit être exécuté dans le contexte d’un compte Automation.
Pour un compte Automation personnalisé, utilisez le script suivant :

```azurepowershell
param(
    [Parameter(Mandatory=$true)]
    [String] $VaultResourceId,
    [Parameter(Mandatory=$true)]
    [ValidateSet("Enabled",'Disabled')]
    [Alias("Enabled or Disabled")]
    [String] $AutoUpdateAction,
    [Parameter(Mandatory=$false)]
    [String] $AutomationAccountArmId
)
$SiteRecoveryRunbookName = "Modify-AutoUpdateForVaultForPatner"
$TaskId = [guid]::NewGuid().ToString()
$SubscriptionId = "00000000-0000-0000-0000-000000000000"
$AsrApiVersion = "2018-01-10"
$RunAsConnectionName = "AzureRunAsConnection"
$ArmEndPoint = "https://management.azure.com"
$AadAuthority = "https://login.windows.net/"
$AadAudience = "https://management.core.windows.net/"
$AzureEnvironment = "AzureCloud"
$Timeout = "160"
function Throw-TerminatingErrorMessage
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [String]
        $Message
        )
    throw ("Message: {0}, TaskId: {1}.") -f $Message, $TaskId
}
function Write-Tracing
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [ValidateSet("Informational", "Warning", "ErrorLevel", "Succeeded", IgnoreCase = $true)]
                [String]
        $Level,
        [Parameter(Mandatory=$true)]
        [String]
        $Message,
            [Switch]
        $DisplayMessageToUser
        )
    Write-Output $Message
}
function Write-InformationTracing
{
        Param
    (
        [Parameter(Mandatory=$true)]
        [String]
        $Message
        )
    Write-Tracing -Message $Message -Level Informational -DisplayMessageToUser
}
function ValidateInput()
{
    try
    {
        if(!$VaultResourceId.StartsWith("/subscriptions", [System.StringComparison]::OrdinalIgnoreCase))
        {
            $ErrorMessage = "The vault resource id should start with /subscriptions."
            throw $ErrorMessage
        }
        $Tokens = $VaultResourceId.SubString(1).Split("/")
        if(!($Tokens.Count % 2 -eq 0))
        {
            $ErrorMessage = ("Odd Number of tokens: {0}." -f $Tokens.Count)
            throw $ErrorMessage
        }
        if(!($Tokens.Count/2 -eq 4))
        {
            $ErrorMessage = ("Invalid number of resource in vault ARM id expected:4, actual:{0}." -f ($Tokens.Count/2))
            throw $ErrorMessage
        }
        if($AutoUpdateAction -ieq "Enabled" -and [string]::IsNullOrEmpty($AutomationAccountArmId))
        {
            $ErrorMessage = ("The automation account ARM id should not be null or empty when AutoUpdateAction is enabled.")
            throw $ErrorMessage
        }
    }
    catch
    {
        $ErrorMessage = ("ValidateInput failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
function Initialize-SubscriptionId()
{
    try
    {
        $Tokens = $VaultResourceId.SubString(1).Split("/")
        $Count = 0
                $ArmResources = @{}
        while($Count -lt $Tokens.Count)
        {
            $ArmResources[$Tokens[$Count]] = $Tokens[$Count+1]
            $Count = $Count + 2
        }
                return $ArmResources["subscriptions"]
    }
    catch
    {
        Write-Tracing -Level ErrorLevel -Message ("Initialize-SubscriptionId: failed with [Exception: {0}]." -f $_.Exception) -DisplayMessageToUser
        throw
    }
}
function Invoke-InternalRestMethod($Uri, $Headers, [ref]$Result)
{
    $RetryCount = 0
    $MaxRetry = 3
    do
    {
        try
        {
            $ResultObject = Invoke-RestMethod -Uri $Uri -Headers $Headers
            ($Result.Value) += ($ResultObject)
            break
        }
        catch
        {
            Write-InformationTracing ("Retry Count: {0}, Exception: {1}." -f $RetryCount, $_.Exception)
            $RetryCount++
            if(!($RetryCount -le $MaxRetry))
            {
                throw
            }
            Start-Sleep -Milliseconds 2000
        }
    }while($true)
}
function Invoke-InternalWebRequest($Uri, $Headers, $Method, $Body, $ContentType, [ref]$Result)
{
    $RetryCount = 0
    $MaxRetry = 3
    do
    {
        try
        {
            $ResultObject = Invoke-WebRequest -Uri $UpdateUrl -Headers $Header -Method 'PATCH' `
                -Body $InputJson  -ContentType "application/json" -UseBasicParsing
            ($Result.Value) += ($ResultObject)
            break
        }
        catch
        {
            Write-InformationTracing ("Retry Count: {0}, Exception: {1}." -f $RetryCount, $_.Exception)
            $RetryCount++
            if(!($RetryCount -le $MaxRetry))
            {
                throw
            }
            Start-Sleep -Milliseconds 2000
        }
    }while($true)
}
function Get-Header([ref]$Header, $AadAudience, $AadAuthority, $RunAsConnectionName){
    try
    {
        $RunAsConnection = Get-AutomationConnection -Name $RunAsConnectionName
        $TenantId = $RunAsConnection.TenantId
        $ApplicationId = $RunAsConnection.ApplicationId
        $CertificateThumbprint = $RunAsConnection.CertificateThumbprint
        $Path = "cert:\CurrentUser\My\{0}" -f $CertificateThumbprint
        $Secret = Get-ChildItem -Path $Path
        $ClientCredential = New-Object Microsoft.IdentityModel.Clients.ActiveDirectory.ClientAssertionCertificate(
                $ApplicationId,
                $Secret)
        # Trim the forward slash from the AadAuthority if it exist.
        $AadAuthority = $AadAuthority.TrimEnd("/")
        $AuthContext = New-Object Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext(
            "{0}/{1}" -f $AadAuthority, $TenantId )
        $AuthenticationResult = $authContext.AcquireToken($AadAudience, $Clientcredential)
        $Header.Value['Content-Type'] = 'application\json'
        $Header.Value['Authorization'] = $AuthenticationResult.CreateAuthorizationHeader()
        $Header.Value["x-ms-client-request-id"] = $TaskId + "/" + (New-Guid).ToString() + "-" + (Get-Date).ToString("u")
    }
    catch
    {
        $ErrorMessage = ("Get-BearerToken: failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
function Get-ProtectionContainerToBeModified([ref] $ContainerMappingList)
{
    try
    {
        Write-InformationTracing ("Get protection container mappings : {0}." -f $VaultResourceId)
        $ContainerMappingListUrl = $ArmEndPoint + $VaultResourceId + "/replicationProtectionContainerMappings" + "?api-version=" + $AsrApiVersion
        Write-InformationTracing ("Getting the bearer token and the header.")
        Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
        $Result = @()
        Invoke-InternalRestMethod -Uri $ContainerMappingListUrl -Headers $header -Result ([ref]$Result)
        $ContainerMappings = $Result[0]
        Write-InformationTracing ("Total retrieved container mappings: {0}." -f $ContainerMappings.Value.Count)
        foreach($Mapping in $ContainerMappings.Value)
        {
            if(($Mapping.properties.providerSpecificDetails -eq $null) -or ($Mapping.properties.providerSpecificDetails.instanceType -ine "A2A"))
            {
                Write-InformationTracing ("Mapping properties: {0}." -f ($Mapping.properties))
                Write-InformationTracing ("Ignoring container mapping: {0} as the provider does not match." -f ($Mapping.Id))
                continue;
            }
            if($Mapping.Properties.State -ine "Paired")
            {
                Write-InformationTracing ("Ignoring container mapping: {0} as the state is not paired." -f ($Mapping.Id))
                continue;
            }
            Write-InformationTracing ("Provider specific details {0}." -f ($Mapping.properties.providerSpecificDetails))
            $MappingAutoUpdateStatus = $Mapping.properties.providerSpecificDetails.agentAutoUpdateStatus
            $MappingAutomationAccountArmId = $Mapping.properties.providerSpecificDetails.automationAccountArmId
            $MappingHealthErrorCount = $Mapping.properties.HealthErrorDetails.Count
            if($AutoUpdateAction -ieq "Enabled" -and
                ($MappingAutoUpdateStatus -ieq "Enabled") -and
                ($MappingAutomationAccountArmId -ieq $AutomationAccountArmId) -and
                ($MappingHealthErrorCount -eq 0))
            {
                Write-InformationTracing ("Provider specific details {0}." -f ($Mapping.properties))
                Write-InformationTracing ("Ignoring container mapping: {0} as the auto update is already enabled and is healthy." -f ($Mapping.Id))
                continue;
            }
            ($ContainerMappingList.Value).Add($Mapping.id)
        }
    }
    catch
    {
        $ErrorMessage = ("Get-ProtectionContainerToBeModified: failed with [Exception: {0}]." -f $_.Exception)
        Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
        Throw-TerminatingErrorMessage -Message $ErrorMessage
    }
}
$OperationStartTime = Get-Date
$ContainerMappingList = New-Object System.Collections.Generic.List[System.String]
$JobsInProgressList = @()
$JobsCompletedSuccessList = @()
$JobsCompletedFailedList = @()
$JobsFailedToStart = 0
$JobsTimedOut = 0
$Header = @{}
$AzureRMProfile = Get-Module -ListAvailable -Name AzureRM.Profile | Select Name, Version, Path
$AzureRmProfileModulePath = Split-Path -Parent $AzureRMProfile.Path
Add-Type -Path (Join-Path $AzureRmProfileModulePath "Microsoft.IdentityModel.Clients.ActiveDirectory.dll")
$Inputs = ("Tracing inputs VaultResourceId: {0}, Timeout: {1}, AutoUpdateAction: {2}, AutomationAccountArmId: {3}." -f $VaultResourceId, $Timeout, $AutoUpdateAction, $AutomationAccountArmId)
Write-Tracing -Message $Inputs -Level Informational -DisplayMessageToUser
$CloudConfig = ("Tracing cloud configuration ArmEndPoint: {0}, AadAuthority: {1}, AadAudience: {2}." -f $ArmEndPoint, $AadAuthority, $AadAudience)
Write-Tracing -Message $CloudConfig -Level Informational -DisplayMessageToUser
$AutomationConfig = ("Tracing automation configuration RunAsConnectionName: {0}." -f $RunAsConnectionName)
Write-Tracing -Message $AutomationConfig -Level Informational -DisplayMessageToUser
ValidateInput
$SubscriptionId = Initialize-SubscriptionId
Get-ProtectionContainerToBeModified ([ref]$ContainerMappingList)
$Input = @{
  "properties"= @{
    "providerSpecificInput"= @{
        "instanceType" = "A2A"
        "agentAutoUpdateStatus" = $AutoUpdateAction
        "automationAccountArmId" = $AutomationAccountArmId
    }
  }
}
$InputJson = $Input |  ConvertTo-Json
if ($ContainerMappingList.Count -eq 0)
{
    Write-Tracing -Level Succeeded -Message ("Exiting as there are no container mappings to be modified.") -DisplayMessageToUser
    exit
}
Write-InformationTracing ("Container mappings to be updated has been retrieved with count: {0}." -f $ContainerMappingList.Count)
try
{
    Write-InformationTracing ("Start the modify container mapping jobs.")
    ForEach($Mapping in $ContainerMappingList)
    {
    try {
            $UpdateUrl = $ArmEndPoint + $Mapping + "?api-version=" + $AsrApiVersion
            Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
            $Result = @()
            Invoke-InternalWebRequest -Uri $UpdateUrl -Headers $Header -Method 'PATCH' `
                -Body $InputJson  -ContentType "application/json" -Result ([ref]$Result)
            $Result = $Result[0]
            $JobAsyncUrl = $Result.Headers['Azure-AsyncOperation']
            Write-InformationTracing ("The modify container mapping job invoked with async url: {0}." -f $JobAsyncUrl)
            $JobsInProgressList += $JobAsyncUrl;
            # Rate controlling the set calls to maximum 60 calls per minute.
            # ASR throttling for set calls is 200 in 1 minute.
            Start-Sleep -Milliseconds 1000
        }
        catch{
            Write-InformationTracing ("The modify container mappings job creation failed for: {0}." -f $Ru)
            Write-InformationTracing $_
            $JobsFailedToStart++
        }
    }
    Write-InformationTracing ("Total modify container mappings has been initiated: {0}." -f $JobsInProgressList.Count)
}
catch
{
    $ErrorMessage = ("Modify container mapping jobs failed with [Exception: {0}]." -f $_.Exception)
    Write-Tracing -Level ErrorLevel -Message $ErrorMessage -DisplayMessageToUser
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
try
{
    while($JobsInProgressList.Count -ne 0)
    {
        Sleep -Seconds 30
        $JobsInProgressListInternal = @()
        ForEach($JobAsyncUrl in $JobsInProgressList)
        {
            try
            {
                Get-Header ([ref]$Header) $AadAudience $AadAuthority $RunAsConnectionName
                $Result = Invoke-RestMethod -Uri $JobAsyncUrl -Headers $header
                $JobState = $Result.Status
                if($JobState -ieq "InProgress")
                {
                    $JobsInProgressListInternal += $JobAsyncUrl
                }
                elseif($JobState -ieq "Succeeded" -or `
                    $JobState -ieq "PartiallySucceeded" -or `
                    $JobState -ieq "CompletedWithInformation")
                {
                    Write-InformationTracing ("Jobs succeeded with state: {0}." -f $JobState)
                    $JobsCompletedSuccessList += $JobAsyncUrl
                }
                else
                {
                    Write-InformationTracing ("Jobs failed with state: {0}." -f $JobState)
                    $JobsCompletedFailedList += $JobAsyncUrl
                }
            }
            catch
            {
                Write-InformationTracing ("The get job failed with: {0}. Ignoring the exception and retrying the next job." -f $_.Exception)
                # The job on which the tracking failed, will be considered in progress and tried again later.
                $JobsInProgressListInternal += $JobAsyncUrl
            }
            # Rate controlling the get calls to maximum 120 calls each minute.
            # ASR throttling for get calls is 10000 in 60 minutes.
            Start-Sleep -Milliseconds 500
        }
        Write-InformationTracing ("Jobs remaining {0}." -f $JobsInProgressListInternal.Count)
        $CurrentTime = Get-Date
        if($CurrentTime -gt $OperationStartTime.AddMinutes($Timeout))
        {
            Write-InformationTracing ("Tracing modify cloud pairing jobs has timed out.")
            $JobsTimedOut = $JobsInProgressListInternal.Count
            $JobsInProgressListInternal = @()
        }
        $JobsInProgressList = $JobsInProgressListInternal
    }
}
catch
{
    $ErrorMessage = ("Tracking modify cloud pairing jobs failed with [Exception: {0}]." -f $_.Exception)
    Write-Tracing -Level ErrorLevel -Message $ErrorMessage  -DisplayMessageToUser
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
Write-InformationTracing ("Tracking modify cloud pairing jobs completed.")
Write-InformationTracing ("Modify cloud pairing jobs success: {0}." -f $JobsCompletedSuccessList.Count)
Write-InformationTracing ("Modify cloud pairing jobs failed: {0}." -f $JobsCompletedFailedList.Count)
Write-InformationTracing ("Modify cloud pairing jobs failed to start: {0}." -f $JobsFailedToStart)
Write-InformationTracing ("Modify cloud pairing jobs timedout: {0}." -f $JobsTimedOut)
if($JobsTimedOut -gt  0)
{
    $ErrorMessage = "One or more modify cloud pairing jobs has timedout."
    Write-Tracing -Level ErrorLevel -Message ($ErrorMessage)
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
elseif($JobsCompletedSuccessList.Count -ne $ContainerMappingList.Count)
{
    $ErrorMessage = "One or more modify cloud pairing jobs failed."
    Write-Tracing -Level ErrorLevel -Message ($ErrorMessage)
    Throw-TerminatingErrorMessage -Message $ErrorMessage
}
Write-Tracing -Level Succeeded -Message ("Modify cloud pairing completed.") -DisplayMessageToUser
```

### <a name="manage-updates-manually"></a>Gérer les mises à jour manuellement

1. Si de nouvelles mises à jour pour le service Mobilité sont installées sur vos machines virtuelles, vous verrez la notification suivante : **Une nouvelle mise à jour de l’agent de réplication Site Recovery est disponible. Cliquez pour installer.**

   :::image type="content" source="./media/vmware-azure-install-mobility-service/replicated-item-notif.png" alt-text="Fenêtre Éléments répliqués":::

1. Sélectionnez la notification pour ouvrir la page de sélection de machine virtuelle.
1. Choisissez les machines virtuelles que vous voulez mettre à niveau, puis sélectionnez **OK**. Le service de mise à jour de la Mobilité démarre pour chaque machine virtuelle sélectionnée.

   :::image type="content" source="./media/vmware-azure-install-mobility-service/update-okpng.png" alt-text="Éléments répliqués - Liste des machines virtuelles":::

## <a name="common-issues-and-troubleshooting"></a>Problèmes courants et résolutions

En cas de problème lié aux mises à jour automatiques, vous voyez une notification d’erreur sous **Problèmes de configuration** dans le tableau de bord du coffre.

Si vous ne pouvez pas activer les mises à jour automatiques, consultez les erreurs courantes et actions recommandées suivantes :

- **Erreur** : Vous n’avez pas les autorisations nécessaires pour créer un compte d’identification Azure (principal du service) et octroyer le rôle Contributeur au principal du service.

  **Action recommandée** : Vérifiez que le compte de connexion est affecté en tant que Contributeur et réessayez. Pour plus d’informations sur l’affectation des autorisations, consultez la section relative aux autorisations requises de [Procédure : Utilisez le portail pour créer une application Azure AD et un principal du service pouvant accéder aux ressources](../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

  Pour résoudre la plupart des problèmes après avoir activé les mises à jour automatiques, sélectionnez **Réparer**. Si le bouton de réparation n’est pas disponible, consultez le message d’erreur affiché dans le volet des paramètres de mise à jour de l’extension.

  :::image type="content" source="./media/azure-to-azure-autoupdate/repair.png" alt-text="Bouton de réparation du service Site Recovery dans les paramètres de mise à jour de l’extension":::

- **Erreur** : le compte d’identification n’a pas l’autorisation d’accéder à la ressource Recovery Services.

  **Action recommandée** : Supprimez puis [recréez le compte d’identification](../automation/manage-runas-account.md). Ou vérifiez que le compte d’identification Automation de l’application Azure Active Directory peut accéder à la ressource Recovery Services.

- **Erreur** : le compte d’identification est introuvable. Un de ces éléments a été supprimé ou n’a pas été créé : Application Azure Active Directory, Principal du service, Rôle, Ressource de certificat Automation, Ressource de connexion Automation, ou l’empreinte numérique n’est pas identique entre le certificat et la connexion.

  **Action recommandée** : Supprimez puis [recréez le compte d’identification](../automation/manage-runas-account.md).

- **Erreur** : Le certificat d’identification d’Azure utilisé par le compte Automation va bientôt expirer.

  Le certificat auto-signé créé pour le compte d’identification expire un an après la date de création. Vous pouvez le renouveler à tout moment avant qu’il n’expire. Si vous êtes inscrit aux notifications par e-mail, vous recevrez également des e-mails lorsqu’une action de votre part est requise. Cette erreur s’affiche deux mois avant la date d’expiration et devient une erreur critique si le certificat a expiré. Une fois que le certificat a expiré, la mise à jour automatique ne fonctionne plus tant que vous ne le renouvelez pas.

  **Action recommandée** : Pour résoudre ce problème, sélectionnez **Réparer**, puis **Renouveler le certificat**.

  :::image type="content" source="./media/azure-to-azure-autoupdate/automation-account-renew-runas-certificate.PNG" alt-text="renew-cert":::

  > [!NOTE]
  > Une fois le certificat renouvelé, actualisez la page pour afficher l’état actuel.
