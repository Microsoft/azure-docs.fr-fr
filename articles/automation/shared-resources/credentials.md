---
title: Gérer les informations d’identification dans Azure Automation
description: Cet article explique comment créer des ressources d’informations d’identification et les utiliser dans un runbook ou une configuration DSC.
services: automation
ms.subservice: shared-capabilities
ms.date: 12/03/2020
ms.topic: conceptual
ms.openlocfilehash: ec35653f67c46a7032e834020d8e2ca4ab3125c8
ms.sourcegitcommit: 65a4f2a297639811426a4f27c918ac8b10750d81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96558831"
---
# <a name="manage-credentials-in-azure-automation"></a>Gérer les informations d’identification dans Azure Automation

Une ressource d’informations d’identification Automation détient un objet constitué d’informations d’identification de sécurité, comme un nom d’utilisateur et un mot de passe. Les runbooks et les configurations DSC utilisent des applets de commande qui acceptent un objet [PSCredential](/dotnet/api/system.management.automation.pscredential) pour l’authentification. Ils peuvent aussi extraire le nom d’utilisateur et le mot de passe de l’objet `PSCredential` à fournir à une application ou à un service nécessitant une authentification.

>[!NOTE]
>Les ressources sécurisées dans Azure Automation incluent les informations d'identification, les certificats, les connexions et les variables chiffrées. Ces ressources sont chiffrées et stockées dans Azure Automation en utilisant une clé unique générée pour chaque compte Automation. Azure Automation stocke la clé dans le coffre de clés géré par le système. Avant de stocker une ressource sécurisée, Automation charge la clé à partir de Key Vault, puis l’utilise pour chiffrer la ressource. 

[!INCLUDE [gdpr-dsr-and-stp-note.md](../../../includes/gdpr-dsr-and-stp-note.md)]

## <a name="powershell-cmdlets-used-to-access-credentials"></a>Cmdlets PowerShell utilisées pour accéder aux informations d’identification

Les cmdlets du tableau suivant créent et gèrent les informations d’identification Automation avec PowerShell. Elles sont fournies dans le cadre des [modules Az](modules.md#az-modules).

| Applet de commande | Description |
|:--- |:--- |
| [Get-AzAutomationCredential](/powershell/module/az.automation/get-azautomationcredential) |Récupère un objet [CredentialInfo](/dotnet/api/microsoft.azure.commands.automation.model.credentialinfo) contenant les métadonnées relatives aux informations d’identification. La cmdlet ne récupère pas l’objet `PSCredential` proprement dit.  |
| [New-AzAutomationCredential](/powershell/module/az.automation/new-azautomationcredential) |Crée de nouvelles informations d’identification Automation. |
| [Remove-AzAutomationCredential](/powershell/module/az.automation/remove-azautomationcredential) |Supprime des informations d’identification Automation. |
| [Set-AzAutomationCredential](/powershell/module/az.automation/set-azautomationcredential) |Définit les propriétés d’informations d’identification Automation existantes. |

## <a name="other-cmdlets-used-to-access-credentials"></a>Autres cmdlets utilisées pour accéder aux informations d’identification

Les cmdlets du tableau suivant permettent d’accéder aux informations d’identification enregistrées dans vos runbooks et vos configurations DSC. 

| Applet de commande | Description |
|:--- |:--- |
| `Get-AutomationPSCredential` |Récupère un objet `PSCredential` à utiliser dans un runbook ou une configuration DSC. Dans la plupart des cas, il est préférable d’utiliser cette [cmdlet interne](modules.md#internal-cmdlets) plutôt que la cmdlet `Get-AzAutomationCredential`, car cette dernière ne récupère que les informations d’identification, qu’il n’est généralement pas utile de passer à une autre cmdlet. |
| [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential) |Obtient des informations d’identification avec une invite pour le nom d’utilisateur et le mot de passe. Cette cmdlet fait partie du module Microsoft.PowerShell.Security par défaut. Voir [Modules par défaut](modules.md#default-modules).|
| [New-AzureAutomationCredential](/powershell/module/servicemanagement/azure.service/new-azureautomationcredential) | Crée des informations d’identification. Cette cmdlet fait partie du module Azure par défaut. Voir [Modules par défaut](modules.md#default-modules).|

Pour récupérer des objets `PSCredential` dans votre code, vous devez importer le module `Orchestrator.AssetManagement.Cmdlets`. Pour plus d’informations, consultez [Gestion des modules dans Azure Automation](modules.md).

```powershell
Import-Module Orchestrator.AssetManagement.Cmdlets -ErrorAction SilentlyContinue
```

> [!NOTE]
> Il est déconseillé d’utiliser des variables dans le paramètre `Name` de `Get-AutomationPSCredential`. Leur utilisation peut contrarier la découverte des dépendances entre les runbooks ou les configurations DSC et les informations d’identification au moment de la conception.

## <a name="python-2-functions-that-access-credentials"></a>Fonctions Python 2 qui accèdent aux informations d’identification

La fonction dans le tableau suivant est utilisée pour accéder aux informations d’identification dans un runbook Python 2.

| Fonction | Description |
|:---|:---|
| `automationassets.get_automation_credential` | Récupère des informations sur une ressource d’informations d’identification. |

> [!NOTE]
> Importez le module `automationassets` en haut de votre runbook Python pour accéder aux fonctions des ressources.

## <a name="create-a-new-credential-asset"></a>Créer une ressource d’informations d’identification

Vous pouvez créer une ressource d’informations d’identification à partir du portail Azure ou de Windows PowerShell.

### <a name="create-a-new-credential-asset-with-the-azure-portal"></a>Créer une ressource d’informations d’identification avec le portail Azure

1. À partir de votre compte Automation, dans le volet gauche, sélectionnez **Informations d’identification** sous **Ressources partagées**.
2. Dans la page **Informations d’identification**, sélectionnez **Ajouter des informations d’identification**.
3. Dans le volet Nouvelles informations d’identification, entrez un nom d’informations d’identification approprié en suivant votre convention de nommage.
4. Tapez votre ID d’accès dans le champ **Nom d’utilisateur**.
5. Pour les deux champs de mot de passe, entrez votre clé d’accès secrète.

    ![Créer de nouvelles informations d’identification](../media/credentials/credential-create.png)

6. Si la case Authentification multifacteur est cochée, décochez-la.
7. Cliquez sur **Créer** pour enregistrer la nouvelle ressource d’informations d’identification.

> [!NOTE]
> Azure Automation ne prend pas en charge les comptes d’utilisateurs qui utilisent l’authentification multifacteur.

### <a name="create-a-new-credential-asset-with-windows-powershell"></a>Créer une ressource d’informations d’identification avec Windows PowerShell

Les exemples suivants montrent comment créer une ressource d’informations d’identification Automation. Un objet `PSCredential` est d’abord créé à partir du nom et du mot de passe, et est ensuite utilisé pour créer la ressource d’informations d’identification. Au lieu de cela, vous pouvez utiliser l’applet de commande `Get-Credential` pour inviter l’utilisateur à taper un nom et un mot de passe.

```powershell
$user = "MyDomain\MyUser"
$pw = ConvertTo-SecureString "PassWord!" -AsPlainText -Force
$cred = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $user, $pw
New-AzureAutomationCredential -AutomationAccountName "MyAutomationAccount" -Name "MyCredential" -Value $cred
```

## <a name="get-a-credential-asset"></a>Obtenir une ressource d’informations d’identification

Un runbook ou une configuration DSC récupère une ressource d’informations d’identification à l’aide de la cmdlet `Get-AutomationPSCredential` interne. Cette cmdlet récupère un objet `PSCredential` que vous pouvez utiliser avec une cmdlet qui nécessite des informations d’identification. Vous pouvez également récupérer les propriétés de l’objet d’informations d’identification pour les utiliser individuellement. L’objet a des propriétés pour le nom d’utilisateur et le mot de passe sécurisé. 

> [!NOTE]
> La cmdlet `Get-AzAutomationCredential` ne récupère pas d’objet `PSCredential` pouvant être utilisé pour l’authentification. Elle fournit uniquement des informations sur les informations d’identification. Si vous devez utiliser des informations d’identification dans un runbook, vous devez les récupérer sous la forme d’un objet `PSCredential` à l’aide de `Get-AutomationPSCredential`.

Vous pouvez aussi utiliser la méthode [GetNetworkCredential](/dotnet/api/system.management.automation.pscredential.getnetworkcredential) pour récupérer un objet [NetworkCredential](/dotnet/api/system.net.networkcredential) qui représente une version non sécurisée du mot de passe.

### <a name="textual-runbook-example"></a>Exemple de runbook textuel

L’exemple suivant montre comment utiliser des informations d’identification PowerShell dans un runbook. Il récupère les informations d’identification et affecte le nom d’utilisateur et mot de passe associés à des variables.

```powershell
$myCredential = Get-AutomationPSCredential -Name 'MyCredential'
$userName = $myCredential.UserName
$securePassword = $myCredential.Password
$password = $myCredential.GetNetworkCredential().Password
```

Vous pouvez également utiliser des informations d’identification pour vous authentifier auprès d’Azure avec la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Dans la plupart des cas, vous devez utiliser un [compte d’identification](../manage-runas-account.md) et récupérer la connexion avec [Get-AzAutomationConnection](../automation-connections.md).

```powershell
$myCred = Get-AutomationPSCredential -Name 'MyCredential'
$userName = $myCred.UserName
$securePassword = $myCred.Password
$password = $myCred.GetNetworkCredential().Password

$myPsCred = New-Object System.Management.Automation.PSCredential ($userName,$securePassword)

Connect-AzAccount -Credential $myPsCred
```

### <a name="graphical-runbook-example"></a>Exemple de runbook graphique

Vous pouvez ajouter une activité pour la cmdlet `Get-AutomationPSCredential` interne à un runbook graphique en cliquant avec le bouton droit sur les informations d’identification dans le volet Bibliothèque de l’éditeur graphique et en sélectionnant **Ajouter au canevas**.

![Ajout une cmdlet d’informations d’identification au canevas](../media/credentials/credential-add-canvas.png)

L’image suivante montre un exemple d’utilisation d’informations d’identification dans un Runbook graphique. Dans ce cas, les informations d’identification permettent à un runbook de s’authentifier auprès de ressources Azure, comme décrit dans [Utiliser Azure AD dans Azure Automation pour s’authentifier dans Azure](../automation-use-azure-ad.md). La première activité récupère les informations d’identification ayant accès à l’abonnement Azure. L’activité de connexion de compte utilise ensuite ces informations d’identification pour assurer l’authentification pour les activités qui suivent. Un [lien de pipeline](../automation-graphical-authoring-intro.md#use-links-for-workflow) est utilisé ici, car le paramètre `Get-AutomationPSCredential` attend un seul objet.  

![Exemple de workflow d’informations d’identification avec lien de pipeline](../media/credentials/get-credential.png)

## <a name="use-credentials-in-a-dsc-configuration"></a>Utiliser des informations d’identification dans une configuration DSC

Même si les configurations DSC dans Azure Automation peuvent utiliser des ressources d’informations d’identification au moyen de `Get-AutomationPSCredential`, elles peuvent aussi transmettre des ressources d’informations d’identification via des paramètres. Pour plus d’informations, consultez [Compilation de configurations dans Azure Automation DSC](../automation-dsc-compile.md#credential-assets).

## <a name="use-credentials-in-a-python-2-runbook"></a>Utiliser des informations d’identification dans un runbook Python 2

L’exemple suivant montre comment accéder à des informations d’identification dans des runbooks Python 2.

```python
import automationassets
from automationassets import AutomationAssetNotFound

# get a credential
cred = automationassets.get_automation_credential("credtest")
print cred["username"]
print cred["password"]
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les applets de commande utilisées pour accéder aux certificats, consultez [Gérer les modules dans Azure Automation](modules.md).
* Pour obtenir des informations générales sur les runbooks, consultez [Exécution d’un runbook dans Azure Automation](../automation-runbook-execution.md).
* Pour obtenir des informations détaillées sur les configurations DSC, consultez [Vue d’ensemble d’Azure Automation State Configuration](../automation-dsc-overview.md).