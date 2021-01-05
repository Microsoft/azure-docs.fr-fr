---
title: Automatiser les tâches Azure Analysis Services à l’aide de principaux de service | Microsoft Docs
description: Découvrez comment créer des principaux de service pour automatiser les tâches administratives Azure Analysis Services.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: b04b9ababfe0e4c2a60d14044b9d3ee120837dc5
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96491041"
---
# <a name="automation-with-service-principals"></a>Automatisation à l’aide de principaux de service

Les principaux de service sont des ressources d’application Azure Active Directory que vous créez à l’intérieur de votre locataire pour effectuer des opérations sans assistance au niveau du service et des ressources. Il s’agit d’un type unique *d’identité utilisateur*, qui est constitué d’un ID d’application, et d’un mot de passe ou d’un certificat. Seules ces autorisations sont nécessaires pour qu’un principal de service effectue les tâches définies par les rôles et les autorisations attribués. 

Dans Analysis Services, les principaux de service sont utilisés avec Azure Automation, avec le mode sans assistance de PowerShell, avec des applications clientes personnalisées et avec des applications web, dans le but d’automatiser les tâches courantes. Par exemple, le provisionnement des serveurs, le déploiement des modèles, l’actualisation des données, la mise à l’échelle et les opérations de suspension/reprise peuvent être automatisés à l’aide de principaux de service. Les autorisations sont attribuées aux principaux de service via l’appartenance au rôle, comme les comptes UPN standard d’Azure AD.

Analysis Services prend également en charge les opérations effectuées par des identités gérées à l’aide de principaux du service. Pour en savoir plus, consultez [Identités managées pour ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) et [Services Azure prenant en charge l’authentification Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-analysis-services).    

## <a name="create-service-principals"></a>Créer des principaux de service
 
Vous pouvez créer des principaux de service dans le portail Azure ou avec PowerShell. Pour plus d'informations, consultez les rubriques suivantes :

[Créer un principal de service - Portail Azure](../active-directory/develop/howto-create-service-principal-portal.md)   
[Créer un principal de service - PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

## <a name="store-credential-and-certificate-assets-in-azure-automation"></a>Stocker des ressources d’informations d’identification et de certificats dans Azure Automation

Les informations d’identification et les certificats des principaux de service peuvent être stockés de manière sécurisée dans Azure Automation pour les opérations de runbook. Pour plus d'informations, consultez les rubriques suivantes :

[Ressources d’informations d’identification dans Azure Automation](../automation/shared-resources/credentials.md)   
[Ressources de certificats dans Azure Automation](../automation/shared-resources/certificates.md)

## <a name="add-service-principals-to-server-admin-role"></a>Ajouter des principaux de service au rôle d’administrateur de serveur

Avant de pouvoir utiliser un principal de service pour les opérations de gestion d’un serveur Analysis Services, vous devez l’ajouter au rôle Administrateurs de serveur. Les principaux de service doivent être ajoutés directement au rôle d’administrateur de serveur. L’ajout d’un principal de service à un groupe de sécurité, puis l’ajout de ce groupe de sécurité au rôle d’administrateur de serveur n’est pas pris en charge. Pour plus d’informations, consultez [Ajouter un principal de service au rôle d’administrateur du serveur](analysis-services-addservprinc-admins.md)

## <a name="service-principals-in-connection-strings"></a>Principaux de service dans les chaînes de connexion

Vous pouvez utiliser l’ID d’application et le mot de passe ou le certificat d’un principal de service dans les chaînes de connexion, de la même manière que pour un compte UPN.

### <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

#### <a name="using-azanalysisservices-module"></a><a name="azmodule"></a>Via le module Az.AnalysisServices

Si vous utilisez un principal de service pour les opérations de gestion des ressources avec le module [Az.AnalysisServices](/powershell/module/az.analysisservices), utilisez la cmdlet `Connect-AzAccount`. 

Dans l’exemple suivant, la valeur appID et un mot de passe sont utilisés pour effectuer des opérations de plan de contrôle pour la synchronisation des réplicas en lecture seule et des opérations scale up/out :

```powershell
Param (
        [Parameter(Mandatory=$true)] [String] $AppId,
        [Parameter(Mandatory=$true)] [String] $PlainPWord,
        [Parameter(Mandatory=$true)] [String] $TenantId
       )
$PWord = ConvertTo-SecureString -String $PlainPWord -AsPlainText -Force
$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $AppId, $PWord

# Connect using Az module
Connect-AzAccount -Credential $Credential -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxxx"

# Syncronize a database for query scale out
Sync-AzAnalysisServicesInstance -Instance "asazure://westus.asazure.windows.net/testsvr" -Database "testdb"

# Scale up the server to an S1, set 2 read-only replicas, and remove the primary from the query pool. The new replicas will hydrate from the synchronized data.
Set-AzAnalysisServicesServer -Name "testsvr" -ResourceGroupName "testRG" -Sku "S1" -ReadonlyReplicaCount 2 -DefaultConnectionMode Readonly
```

#### <a name="using-sqlserver-module"></a>Via le module SQLServer

Dans l’exemple suivant, un ID d’application et un mot de passe sont utilisés pour effectuer une opération d’actualisation d’une base de données model :

```powershell
Param (
        [Parameter(Mandatory=$true)] [String] $AppId,
        [Parameter(Mandatory=$true)] [String] $PlainPWord,
        [Parameter(Mandatory=$true)] [String] $TenantId
       )
$PWord = ConvertTo-SecureString -String $PlainPWord -AsPlainText -Force

$Credential = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $AppId, $PWord

Invoke-ProcessTable -Server "asazure://westcentralus.asazure.windows.net/myserver" -TableName "MyTable" -Database "MyDb" -RefreshType "Full" -ServicePrincipal -ApplicationId $AppId -TenantId $TenantId -Credential $Credential
```

### <a name="amo-and-adomd"></a>AMO et ADOMD 

Lorsque vous vous connectez à une application cliente ou à une application web, les packages installables des [bibliothèques clientes AMO et ADOMD](/analysis-services/client-libraries?view=azure-analysis-services-current&preserve-view=true) version 15.0.2 (et ultérieures) issus de NuGet prennent en charge l’utilisation des principaux de service dans les chaînes de connexion à l’aide de la syntaxe suivante : `app:AppID`, puis le mot de passe ou `cert:thumbprint`. 

Dans l’exemple suivant, `appID` et `password` sont utilisés pour effectuer une opération d’actualisation d’une base de données model :

```csharp
string appId = "xxx";
string authKey = "yyy";
string connString = $"Provider=MSOLAP;Data Source=asazure://westus.asazure.windows.net/<servername>;User ID=app:{appId};Password={authKey};";
Server server = new Server();
server.Connect(connString);
Database db = server.Databases.FindByName("adventureworks");
Table tbl = db.Model.Tables.Find("DimDate");
tbl.RequestRefresh(RefreshType.Full);
db.Model.SaveChanges();
```

## <a name="next-steps"></a>Étapes suivantes
[Se connecter avec Azure PowerShell](/powershell/azure/authenticate-azureps)   
[Actualiser avec Logic Apps](analysis-services-refresh-logic-app.md)  
[Actualiser avec Azure Automation](analysis-services-refresh-azure-automation.md)  
[Ajouter un principal de service au rôle d’administrateur du serveur](analysis-services-addservprinc-admins.md)  
[Automatiser des tâches d’espace de travail et de jeu de données Power BI Premium avec des principaux de service](/power-bi/admin/service-premium-service-principal)