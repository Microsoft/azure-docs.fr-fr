---
title: Gérer les connexions dans Azure Automation
description: Cet article explique comment gérer les connexions Azure Automation pour des applications ou services externes, et comment les utiliser dans des runbooks.
services: automation
ms.subservice: shared-capabilities
ms.date: 01/13/2020
ms.topic: conceptual
ms.custom: has-adal-ref
ms.openlocfilehash: 0a3cff616f814b8e5209b15f9d3f7439533452ca
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92071759"
---
# <a name="manage-connections-in-azure-automation"></a>Gérer les connexions dans Azure Automation

Une ressource de connexion Azure Automation contient les informations indiquées ci-dessous. Ces informations sont requises pour la connexion à un service ou à une application externes à partir d’un runbook ou d’une configuration DSC. 

* Informations nécessaires à l’authentification, telles que le nom d’utilisateur et le mot de passe
* Informations de connexion, telles que l’URL ou le port

La ressource de connexion conserve toutes les propriétés permettant de se connecter à une application particulière, ce qui rend inutile la création de plusieurs variables. Vous pouvez modifier les valeurs pour une connexion à un seul endroit, et vous pouvez transmettre le nom d’une connexion à un runbook ou à une configuration DSC dans un seul paramètre. Le runbook ou la configuration accèdent aux propriétés d’une connexion en utilisant la cmdlet `Get-AutomationConnection` interne.

Lorsque vous créez une connexion, vous devez spécifier un type de connexion. Le type de connexion est un modèle qui définit un ensemble de propriétés. Vous pouvez ajouter un type de connexion à Azure Automation en utilisant un module d’intégration avec un fichier de métadonnées. Vous pouvez aussi créer un type de connexion à l’aide de l’[API Azure Automation](/previous-versions/azure/reference/mt163818(v=azure.100)) si le module d’intégration comprend un type de connexion et est importé dans votre compte Automation. 

>[!NOTE]
>Les ressources sécurisées dans Azure Automation incluent les informations d'identification, les certificats, les connexions et les variables chiffrées. Ces ressources sont chiffrées et stockées dans Azure Automation en utilisant une clé unique générée pour chaque compte Automation. Azure Automation stocke la clé dans le coffre de clés géré par le système. Avant de stocker une ressource sécurisée, Automation charge la clé à partir de Key Vault, puis l’utilise pour chiffrer la ressource. 

## <a name="connection-types"></a>Types de connexion

Azure Automation met à disposition les types de connexion intégrés suivants :

* `Azure` : représente une connexion utilisée pour gérer des ressources classiques.
* `AzureServicePrincipal` : représente une connexion utilisée par le compte d’identification Azure.
* `AzureClassicCertificate` : représente une connexion utilisée par le compte d’identification Azure Classic.

Généralement, il n'est pas nécessaire de créer une ressource de connexion, car celle-ci est générée au moment où vous créez un [compte d'identification](manage-runas-account.md).

## <a name="powershell-cmdlets-to-access-connections"></a>Cmdlets PowerShell pour accéder aux connexions

Les cmdlets du tableau suivant créent et gèrent les connexions Automation avec PowerShell. Elles sont fournies avec les [Modules Az](shared-resources/modules.md#az-modules).

|Applet de commande|Description|
|---|---|
|[Get-AzAutomationConnection](/powershell/module/az.automation/get-azautomationconnection?view=azps-3.7.0)|Récupère des informations sur une connexion.|
|[New-AzAutomationConnection](/powershell/module/az.automation/new-azautomationconnection?view=azps-3.7.0)|Crée une connexion.|
|[Remove-AzAutomationConnection](/powershell/module/Az.Automation/Remove-AzAutomationConnection?view=azps-3.7.0)|Supprime une connexion existante.|
|[Set-AzAutomationConnectionFieldValue](/powershell/module/Az.Automation/Set-AzAutomationConnectionFieldValue?view=azps-3.7.0)|Définit la valeur d’un champ particulier d’une connexion existante.|

## <a name="internal-cmdlets-to-access-connections"></a>Cmdlets internes pour accéder aux connexions

La cmdlet interne dans le tableau suivant est utilisée pour accéder aux connexions dans vos runbooks et configurations DSC. Cette cmdlet est fournie avec le module global `Orchestrator.AssetManagement.Cmdlets`. Pour plus d’informations, consultez [cmdlets internes](shared-resources/modules.md#internal-cmdlets).

|Cmdlet interne|Description|
|---|---|
|`Get-AutomationConnection` | Récupère les valeurs des différents champs de la connexion et les renvoie sous forme de [table de hachage](/powershell/module/microsoft.powershell.core/about/about_hash_tables). Vous pouvez utiliser cette table de hachage avec les commandes appropriées dans le runbook ou la configuration DSC.|

>[!NOTE]
>Évitez d'utiliser des variables dans le paramètre `Name` de `Get-AutomationConnection`. L’utilisation de variables dans ce cas peut compliquer la détection des dépendances entre les runbooks ou les configurations DSC et les ressources de connexion au moment de la conception.

## <a name="python-2-functions-to-access-connections"></a>Fonctions Python 2 pour accéder aux connexions

La fonction présentée dans le tableau suivant est utilisée pour accéder aux connexions dans un runbook Python 2.

| Fonction | Description |
|:---|:---|
| `automationassets.get_automation_connection` | Récupère une connexion. Renvoie un dictionnaire avec les propriétés de la connexion. |

> [!NOTE]
> Vous devez importer le module `automationassets` en haut de votre runbook Python pour accéder aux fonctions des ressources.

## <a name="create-a-new-connection"></a>Créer une nouvelle connexion

### <a name="create-a-new-connection-with-the-azure-portal"></a>Créer une connexion avec le portail Azure

Pour créer une connexion dans le portail Azure :

1. Dans votre compte Automation, sous **Ressources partagées**, cliquez sur **Connexions**.
2. Dans la page Connexions, cliquez sur **+ Ajouter une connexion**.
4. Dans le volet Nouvelle connexion, dans le champ **Type**, sélectionnez le type de connexion à créer. Vos choix sont `Azure`, `AzureServicePrincipal` et `AzureClassicCertificate`. 
5. Le formulaire présente les propriétés du type de connexion que vous avez choisi. Remplissez le formulaire, puis cliquez sur **Créer** pour enregistrer la nouvelle connexion.

### <a name="create-a-new-connection-with-windows-powershell"></a>Créer une connexion avec Windows PowerShell

Pour créer une connexion avec Windows PowerShell, utilisez la cmdlet `New-AzAutomationConnection`. Cette cmdlet comporte un paramètre `ConnectionFieldValues` qui attend une table de hachage définissant des valeurs pour chacune des propriétés définies par le type de connexion.

Vous pouvez utiliser les exemples de commandes suivants comme alternative à la création d'un compte d'identification à partir du portail pour créer une nouvelle ressource de connexion.

```powershell
$ConnectionAssetName = "AzureRunAsConnection"
$ConnectionFieldValues = @{"ApplicationId" = $Application.ApplicationId; "TenantId" = $TenantID.TenantId; "CertificateThumbprint" = $Cert.Thumbprint; "SubscriptionId" = $SubscriptionId}
New-AzAutomationConnection -ResourceGroupName $ResourceGroup -AutomationAccountName $AutomationAccountName -Name $ConnectionAssetName -ConnectionTypeName AzureServicePrincipal -ConnectionFieldValues $ConnectionFieldValues
```

Lorsque vous créez votre compte Automation, celui-ci inclut par défaut plusieurs modules globaux, ainsi que le type de connexion `AzureServicePrincipal` pour créer la ressource de connexion `AzureRunAsConnection`. Si vous tentez de créer une ressource de connexion pour vous connecter à un service ou à une application avec une autre méthode d’authentification, l’opération échoue, car le type de connexion n’est pas encore défini dans votre compte Automation. Pour plus d’informations sur la création de votre propre type de connexion pour un module personnalisé, consultez [Ajouter un type de connexion](#add-a-connection-type).

## <a name="add-a-connection-type"></a>Ajouter un type de connexion

Si votre runbook ou configuration DSC se connectent à un service externe, vous devez définir un type de connexion dans un [module personnalisé](shared-resources/modules.md#custom-modules) appelé module d’intégration. Ce module comprend un fichier de métadonnées qui spécifie les propriétés du type de connexion et est nommé **&lt;ModuleName&gt;-Automation.json**, situé dans le dossier de module de votre fichier **.zip** compressé. Ce fichier contient les champs d’une connexion qui sont requis pour se connecter au système ou au service que le module représente. Ce fichier vous permet de définir les noms de champs, les types de données, l’état du chiffrement et un état facultatif pour le type de connexion. 

L’exemple suivant est un modèle au format de fichier **.json**, qui définit les propriétés de nom d’utilisateur et de mot de passe pour un type de connexion personnalisé nommé `MyModuleConnection` :

```json
{
   "ConnectionFields": [
   {
      "IsEncrypted":  false,
      "IsOptional":  true,
      "Name":  "Username",
      "TypeName":  "System.String"
   },
   {
      "IsEncrypted":  true,
      "IsOptional":  false,
      "Name":  "Password",
      "TypeName":  "System.String"
   }
   ],
   "ConnectionTypeName":  "MyModuleConnection",
   "IntegrationModuleName":  "MyModule"
}
```

## <a name="get-a-connection-in-a-runbook-or-dsc-configuration"></a>Obtenir une connexion dans un runbook ou une configuration DSC

Récupérer une connexion dans un runbook ou une configuration DSC avec la cmdlet `Get-AutomationConnection` interne. Cette cmdlet est préférable à la cmdlet `Get-AzAutomationConnection`, car elle récupère les valeurs de connexion au lieu d’informations sur la connexion. 

### <a name="textual-runbook-example"></a>Exemple de runbook textuel

L’exemple suivant montre comment utiliser le compte d’identification pour s’authentifier auprès de ressources Azure Resource Manager dans votre runbook. Il utilise une ressource de connexion qui représente le compte d’identification, qui fait référence au principal du service basé sur le certificat.

```powershell
$Conn = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
```

### <a name="graphical-runbook-examples"></a>Exemples de runbooks graphiques

Vous pouvez ajouter une activité pour la cmdlet `Get-AutomationConnection` interne à un runbook graphique. Cliquez avec le bouton droit sur la connexion dans le volet Bibliothèque de l’éditeur graphique, puis sélectionnez **Ajouter au canevas**.

![ajouter au canevas](media/automation-connections/connection-add-canvas.png)

L’image suivante montre un exemple d’utilisation d’un objet de connexion dans un runbook graphique. Il repose sur les données `Constant value` définies pour l’activité `Get RunAs Connection` qui utilise un objet de connexion pour l’authentification. Un [lien pipeline](automation-graphical-authoring-intro.md#use-links-for-workflow) est utilisé ici car l'ensemble de paramètres `ServicePrincipalCertificate`attend un seul objet.

![obtenir les connexions](media/automation-connections/automation-get-connection-object.png)

### <a name="python-2-runbook-example"></a>Exemple de runbook Python 2

L’exemple suivant montre comment s’authentifier à l’aide de la connexion d’identification dans un runbook Python 2.

```python
""" Tutorial to show how to authenticate against Azure resource manager resources """
import azure.mgmt.resource
import automationassets

def get_automation_runas_credential(runas_connection):
    """ Returns credentials to authenticate against Azure resoruce manager """
    from OpenSSL import crypto
    from msrestazure import azure_active_directory
    import adal

    # Get the Azure Automation Run As service principal certificate
    cert = automationassets.get_automation_certificate("AzureRunAsCertificate")
    pks12_cert = crypto.load_pkcs12(cert)
    pem_pkey = crypto.dump_privatekey(
        crypto.FILETYPE_PEM, pks12_cert.get_privatekey())

    # Get Run As connection information for the Azure Automation service principal
    application_id = runas_connection["ApplicationId"]
    thumbprint = runas_connection["CertificateThumbprint"]
    tenant_id = runas_connection["TenantId"]

    # Authenticate with service principal certificate
    resource = "https://management.core.windows.net/"
    authority_url = ("https://login.microsoftonline.com/" + tenant_id)
    context = adal.AuthenticationContext(authority_url)
    return azure_active_directory.AdalAuthentication(
        lambda: context.acquire_token_with_client_certificate(
            resource,
            application_id,
            pem_pkey,
            thumbprint)
    )


# Authenticate to Azure using the Azure Automation Run As service principal
runas_connection = automationassets.get_automation_connection(
    "AzureRunAsConnection")
azure_credential = get_automation_runas_credential(runas_connection)
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les cmdlets utilisées pour accéder aux connexions, consultez [Gérer les modules dans Azure Automation](shared-resources/modules.md).
* Pour obtenir des informations générales sur les runbooks, consultez [Exécution d’un runbook dans Azure Automation](automation-runbook-execution.md).
* Pour obtenir des informations détaillées sur les configurations DSC, consultez [Présentation de State Configuration](automation-dsc-overview.md).