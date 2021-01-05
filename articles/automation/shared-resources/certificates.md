---
title: Gérer les certificats dans Azure Automation
description: Cet article explique comment utiliser des certificats pour l’accès par les runbooks et les configurations DSC.
services: automation
ms.subservice: shared-capabilities
ms.date: 09/10/2020
ms.topic: conceptual
ms.openlocfilehash: 1c79b7c239c41e8d195230423b17fa3c5a7f51a6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91825818"
---
# <a name="manage-certificates-in-azure-automation"></a>Gérer les certificats dans Azure Automation

Azure Automation stocke les certificats de manière sécurisée pour l’accès par les runbooks et les configurations DSC, en utilisant l’applet de commande [Get-AzAutomationCertificate](/powershell/module/Az.Automation/Get-AzAutomationCertificate) pour les ressources Azure Resource Manager. Le stockage sécurisé de certificats vous permet de créer des runbooks et des configurations DSC qui utilisent des certificats pour l’authentification, ou les ajoute aux ressources Azure ou tierces.

>[!NOTE]
>Les ressources sécurisées dans Azure Automation incluent les informations d'identification, les certificats, les connexions et les variables chiffrées. Ces ressources sont chiffrées et stockées dans Automation en utilisant une clé unique générée pour chaque compte Automation. Automation stocke la clé dans le service Key Vault managé par le système. Avant de stocker une ressource sécurisée, Automation charge la clé à partir de Key Vault, puis l’utilise pour chiffrer la ressource. 

## <a name="powershell-cmdlets-to-access-certificates"></a>Applets de commande PowerShell pour accéder aux certificats

Les applets de commande du tableau suivant créent et gèrent les certificats Automation avec Windows PowerShell. Elles sont fournies dans le cadre des [modules Az](modules.md#az-modules).

|Applet de commande |Description|
| --- | ---|
|[Get-AzAutomationCertificate](/powershell/module/Az.Automation/Get-AzAutomationCertificate)|Récupère des informations sur un certificat à utiliser dans un Runbook ou dans une configuration DSC. Vous pouvez seulement récupérer le certificat lui-même à l’aide de l’applet de commande `Get-AutomationCertificate` interne.|
|[New-AzAutomationCertificate](/powershell/module/Az.Automation/New-AzAutomationCertificate)|Crée un nouveau certificat dans Automation.|
|[Remove-AzAutomationCertificate](/powershell/module/Az.Automation/Remove-AzAutomationCertificate)|Supprime un certificat dans Automation.|
|[Set-AzAutomationCertificate](/powershell/module/Az.Automation/Set-AzAutomationCertificate)|Définit les propriétés d’un certificat existant, y compris le chargement du fichier de certificat et la définition du mot de passe d’un fichier **.pfx**.|

L’applet de commande [Add-AzureCertificate](/powershell/module/servicemanagement/azure.service/add-azurecertificate) peut aussi être utilisée pour charger un certificat de service destiné au service cloud spécifié.

## <a name="internal-cmdlets-to-access-certificates"></a>Applets de commande internes pour accéder aux certificats

Dans le tableau suivant, l’applet de commande interne est utilisée pour accéder aux certificats dans vos runbooks. Cette applet de commande est fournie avec le module global `Orchestrator.AssetManagement.Cmdlets`. Pour plus d’informations, consultez [cmdlets internes](modules.md#internal-cmdlets).

| Cmdlet interne | Description |
|:---|:---|
|`Get-AutomationCertificate`|Obtient un certificat à utiliser dans un Runbook ou dans une configuration DSC. Retourne un objet [System.Security.Cryptography.X509Certificates.X509Certificate2](/dotnet/api/system.security.cryptography.x509certificates.x509certificate2).|

> [!NOTE] 
> Vous devez éviter d’utiliser des variables dans le paramètre `Name` de `Get-AutomationCertificate` dans un runbook ou une configuration DSC. De telles variables peuvent compliquer la détection des dépendances entre les runbooks ou les configurations DSC et les variables Automation au moment de la conception.

## <a name="python-2-functions-to-access-certificates"></a>Fonctions Python 2 pour accéder aux certificats

Utilisez la fonction dans le tableau suivant pour accéder aux certificats dans un runbook Python 2.

| Fonction | Description |
|:---|:---|
| `automationassets.get_automation_certificate` | Récupère des informations sur les ressources d’un certificat. |

> [!NOTE]
> Vous devez importer le module `automationassets` au début de votre runbook Python pour accéder aux fonctions des ressources.

## <a name="create-a-new-certificate"></a>Créer un certificat

Lorsque vous créez un certificat, vous chargez un fichier .cer ou .pfx dans Automation. Si vous marquez le certificat comme exportable, vous pouvez également le transférer du magasin de certificats Automation. S’il n’est pas exportable, il ne peut servir que pour la signature dans le runbook ou la configuration DSC. Automation demande que le certificat provienne du **Fournisseur de services de chiffrement Microsoft Enhanced RSA et AES**.

### <a name="create-a-new-certificate-with-the-azure-portal"></a>Créer un certificat avec le portail Azure

1. À partir de votre compte Automation, dans le volet gauche, sélectionnez **Certificats** sous **Ressources partagées**.
1. Sur la page **Certificats**, sélectionnez **Ajouter un certificat**.
1. Dans le champ **Nom**, tapez un nom pour le certificat.
1. Pour rechercher un fichier **.cer** ou **.pfx**, sous **Charger un fichier de certificat**, choisissez **Sélectionner un fichier**. Si vous sélectionnez un fichier **.pfx**, spécifiez un mot de passe et indiquez s’il est exportable.
1. Sélectionnez **Créer** pour enregistrer la nouvelle ressource de certificat.

### <a name="create-a-new-certificate-with-powershell"></a>Créer un certificat avec PowerShell

L’exemple suivant démontre la création d’un certificat Automation et sont marquage comme exportable. Cet exemple importe un fichier **.pfx** existant.

```powershell-interactive
$certificateName = 'MyCertificate'
$PfxCertPath = '.\MyCert.pfx'
$CertificatePassword = ConvertTo-SecureString -String 'P@$$w0rd' -AsPlainText -Force
$ResourceGroup = "ResourceGroup01"

New-AzAutomationCertificate -AutomationAccountName "MyAutomationAccount" -Name $certificateName -Path $PfxCertPath –Password $CertificatePassword -Exportable -ResourceGroupName $ResourceGroup
```

### <a name="create-a-new-certificate-with-a-resource-manager-template"></a>Créer un certificat avec un modèle Resource Manager

L’exemple suivant montre comment déployer un certificat sur votre compte Automation en utilisant un modèle Resource Manager via PowerShell :

```powershell-interactive
$AutomationAccountName = "<automation account name>"
$PfxCertPath = '<PFX cert path and filename>'
$CertificatePassword = '<password>'
$certificateName = '<certificate name>' #A name of your choosing
$ResourceGroupName = '<resource group name>' #The one that holds your automation account
$flags = [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::Exportable `
    -bor [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::PersistKeySet `
    -bor [System.Security.Cryptography.X509Certificates.X509KeyStorageFlags]::MachineKeySet
# Load the certificate into memory
$PfxCert = New-Object -TypeName System.Security.Cryptography.X509Certificates.X509Certificate2 -ArgumentList @($PfxCertPath, $CertificatePassword, $flags)
# Export the certificate and convert into base 64 string
$Base64Value = [System.Convert]::ToBase64String($PfxCert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pkcs12))
$Thumbprint = $PfxCert.Thumbprint


$json = @"
{
    '`$schema': 'https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#',
    'contentVersion': '1.0.0.0',
    'resources': [
        {
            'name': '$AutomationAccountName/$certificateName',
            'type': 'Microsoft.Automation/automationAccounts/certificates',
            'apiVersion': '2015-10-31',
            'properties': {
                'base64Value': '$Base64Value',
                'thumbprint': '$Thumbprint',
                'isExportable': true
            }
        }
    ]
}
"@

$json | out-file .\template.json
New-AzResourceGroupDeployment -Name NewCert -ResourceGroupName $ResourceGroupName -TemplateFile .\template.json
```

## <a name="get-a-certificate"></a>Obtenir un certificat

Pour récupérer un certificat, utilisez l’applet de commande interne `Get-AutomationCertificate`. L’applet de commande [Get-AzAutomationCertificate](/powershell/module/Az.Automation/Get-AzAutomationCertificate) n’est pas utilisable, car elle retourne des informations sur la ressource de certificat, et non sur le certificat proprement dit.

### <a name="textual-runbook-example"></a>Exemple de runbook textuel

L’exemple suivant montre comment ajouter un certificat à un service cloud dans un Runbook. Dans cet exemple, le mot de passe est récupéré à partir d’une variable Automation chiffrée.

```powershell-interactive
$serviceName = 'MyCloudService'
$cert = Get-AutomationCertificate -Name 'MyCertificate'
$certPwd = Get-AzAutomationVariable -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAccount" –Name 'MyCertPassword'
Add-AzureCertificate -ServiceName $serviceName -CertToDeploy $cert
```

### <a name="graphical-runbook-example"></a>Exemple de runbook graphique

Ajoutez une activité à un runbook graphique pour l’applet de commande interne `Get-AutomationCertificate` en cliquant avec le bouton droit sur le certificat dans le volet Bibliothèque et en sélectionnant **Ajouter au canevas**.

![Capture d’écran de l’ajout d’un certificat au canevas](../media/certificates/automation-certificate-add-to-canvas.png)

L’image suivante montre un exemple d’utilisation d’un certificat dans un Runbook graphique.

![Capture d’écran d’un exemple de création graphique](../media/certificates/graphical-runbook-add-certificate.png)

### <a name="python-2-example"></a>Exemple Python 2

L’exemple suivant montre comment accéder aux certificats dans les runbooks Python 2.

```python
# get a reference to the Azure Automation certificate
cert = automationassets.get_automation_certificate("AzureRunAsCertificate")

# returns the binary cert content  
print cert
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les applets de commande utilisées pour accéder aux certificats, consultez [Gérer les modules dans Azure Automation](modules.md).
* Pour obtenir des informations générales sur les runbooks, consultez [Exécution d’un runbook dans Azure Automation](../automation-runbook-execution.md).
* Pour obtenir des informations détaillées sur les configurations DSC, consultez [Vue d’ensemble d’Azure Automation State Configuration](../automation-dsc-overview.md).
