---
title: Envoyer un e-mail à partir d’un runbook Azure Automation
description: Cet article explique comment envoyer un e-mail à partir d’un runbook.
services: automation
ms.subservice: process-automation
ms.date: 07/15/2019
ms.topic: conceptual
ms.openlocfilehash: a81ae680a5f04eca0a6cc01ee24b474cc5daabea
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97005203"
---
# <a name="send-an-email-from-a-runbook"></a>Envoyer un e-mail depuis un runbook

Vous pouvez envoyer un e-mail à partir d’un runbook avec [SendGrid](https://sendgrid.com/solutions) à l’aide de PowerShell. 

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou [créer un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Un compte SendGrid](../sendgrid-dotnet-how-to-send-email.md#create-a-sendgrid-account)
* [Un compte Automation](./index.yml) avec modules **Az**
* [Un compte d’identification](./manage-runas-account.md) pour stocker et exécuter le runbook

## <a name="create-an-azure-key-vault"></a>Créer un Azure Key Vault

Vous pouvez créer un coffre de clés Azure à l’aide du script PowerShell suivant. Remplacez les valeurs des variables par les valeurs propres à votre environnement. Utilisez l’interface intégrée Azure Cloud Shell via le bouton **Essayer** en haut à droite du bloc de code. Vous pouvez également copier et exécuter le code localement si les [modules Az](/powershell/azure/install-az-ps) sont installés sur votre ordinateur local.

> [!NOTE]
> Pour récupérer votre clé API, suivez les étapes décrites dans [Rechercher votre clé API SendGrid](../sendgrid-dotnet-how-to-send-email.md#to-find-your-sendgrid-api-key).

```azurepowershell-interactive
$SubscriptionId  =  "<subscription ID>"

# Sign in to your Azure account and select your subscription
# If you omit the SubscriptionId parameter, the default subscription is selected.
Connect-AzAccount -SubscriptionId $SubscriptionId

# Use Get-AzLocation to see your available locations.
$region = "southcentralus"
$KeyVaultResourceGroupName  = "mykeyvaultgroup"
$VaultName = "<Enter a universally unique vault name>"
$SendGridAPIKey = "<SendGrid API key>"
$AutomationAccountName = "testaa"

# Create new Resource Group, or omit this step if you already have a resource group.
New-AzResourceGroup -Name $KeyVaultResourceGroupName -Location $region

# Create the new key vault
$newKeyVault = New-AzKeyVault -VaultName $VaultName -ResourceGroupName $KeyVaultResourceGroupName -Location $region
$resourceId = $newKeyVault.ResourceId

# Convert the SendGrid API key into a SecureString
$Secret = ConvertTo-SecureString -String $SendGridAPIKey -AsPlainText -Force
Set-AzKeyVaultSecret -VaultName $VaultName -Name 'SendGridAPIKey' -SecretValue $Secret

# Grant access to the Key Vault to the Automation Run As account.
$connection = Get-AzAutomationConnection -ResourceGroupName $KeyVaultResourceGroupName -AutomationAccountName $AutomationAccountName -Name AzureRunAsConnection
$appID = $connection.FieldDefinitionValues.ApplicationId
Set-AzKeyVaultAccessPolicy -VaultName $VaultName -ServicePrincipalName $appID -PermissionsToSecrets Set, Get
```

Pour découvrir d’autres méthodes de création d’un coffre de clés Azure et de stockage d’un secret, consultez [Démarrages rapides Key Vault](../key-vault/index.yml).

## <a name="import-required-modules-into-your-automation-account"></a>Importer les modules requis dans votre compte Automation

Pour utiliser Azure Key Vault dans un runbook, vous devez importer les modules suivants dans votre compte Automation :

* [Az.Profile](https://www.powershellgallery.com/packages/Az.Profile)
* [Az.KeyVault](https://www.powershellgallery.com/packages/Az.KeyVault)

Pour obtenir des instructions, consultez [Importer des modules Az](shared-resources/modules.md#import-az-modules).

## <a name="create-the-runbook-to-send-an-email"></a>Créer le runbook pour envoyer un e-mail

Une fois que vous avez créé un coffre de clés et stocké votre clé API `SendGrid`, il est temps de créer le runbook qui récupère la clé API et envoie un e-mail. Utilisons un runbook qui utilise `AzureRunAsConnection` comme [compte d’identification](./manage-runas-account.md) pour s’authentifier auprès d’Azure afin de récupérer le secret d’Azure Key Vault. Nous allons appeler le runbook **Send-GridMailMessage**. Vous pouvez modifier le script PowerShell utilisé à des fins d’exemple et le réutiliser pour différents scénarios.

1. Allez sur votre compte Azure Automation.
2. Sous **Automatisation de processus**, sélectionnez **Runbooks**.
3. En haut de la liste des runbooks, sélectionnez **+ Créer un runbook**.
4. Dans la page Ajouter un runbook, entrez **Send- GridMailMessage** comme nom de runbook. Pour le type de runbook, sélectionnez **PowerShell**. Sélectionnez ensuite **Create** (Créer).
   ![Créer un runbook](./media/automation-send-email/automation-send-email-runbook.png)
5. Le runbook est créé et la page Modifier un runbook PowerShell s’ouvre.
   ![Modifier le runbook](./media/automation-send-email/automation-send-email-edit.png)
6. Copiez l’exemple PowerShell suivant dans la page Modifier. Vérifiez que le `VaultName` spécifie le nom que vous avez choisi pour votre coffre de clés.

    ```powershell-interactive
    Param(
      [Parameter(Mandatory=$True)]
      [String] $destEmailAddress,
      [Parameter(Mandatory=$True)]
      [String] $fromEmailAddress,
      [Parameter(Mandatory=$True)]
      [String] $subject,
      [Parameter(Mandatory=$True)]
      [String] $content
    )

    $Conn = Get-AutomationConnection -Name AzureRunAsConnection
    Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint | Out-Null
    $VaultName = "<Enter your vault name>"
    $SENDGRID_API_KEY = (Get-AzKeyVaultSecret -VaultName $VaultName -Name "SendGridAPIKey").SecretValue
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add("Authorization", "Bearer " + $SENDGRID_API_KEY)
    $headers.Add("Content-Type", "application/json")

    $body = @{
    personalizations = @(
        @{
            to = @(
                    @{
                        email = $destEmailAddress
                    }
            )
        }
    )
    from = @{
        email = $fromEmailAddress
    }
    subject = $subject
    content = @(
        @{
            type = "text/plain"
            value = $content
        }
    )
    }

    $bodyJson = $body | ConvertTo-Json -Depth 4

    $response = Invoke-RestMethod -Uri https://api.sendgrid.com/v3/mail/send -Method Post -Headers $headers -Body $bodyJson
    ```

7. Sélectionnez **Publier** pour enregistrer et publier le runbook.

Pour vérifier que le runbook s’exécute correctement, vous pouvez suivre les étapes décrites dans [Tester un runbook](manage-runbooks.md#test-a-runbook) ou [Démarrer un runbook](start-runbooks.md).

Si vous ne voyez pas votre e-mail de test, vérifiez vos dossiers de **courrier indésirable** et de **spam**.

## <a name="clean-up-resources-after-the-email-operation"></a>Nettoyer les ressources après l’opération d’e-mail

1. Quand le runbook n’est plus nécessaire, sélectionnez-le dans la liste de runbooks et cliquez sur **Supprimer**.

2. Supprimez le coffre de clés à l’aide de l’applet de commande [Remove-AzKeyVault](/powershell/module/az.keyvault/remove-azkeyvault?view=azps-3.7.0).

```azurepowershell-interactive
$VaultName = "<your KeyVault name>"
$ResourceGroupName = "<your ResourceGroup name>"
Remove-AzKeyVault -VaultName $VaultName -ResourceGroupName $ResourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

* Pour envoyer des données de travaux de runbook à votre espace de travail Log Analytics, consultez [Transférer des données de travaux Azure Automation aux journaux Azure Monitor](automation-manage-send-joblogs-log-analytics.md).
* Pour superviser les journaux et les métriques de base, consultez [Utiliser une alerte pour déclencher un runbook Azure Automation](automation-create-alert-triggered-runbook.md).
* Pour corriger les problèmes liés aux opérations de runbook, consultez [Résoudre les problèmes de runbook](./troubleshoot/runbooks.md).
