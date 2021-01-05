---
title: Configurer l’authentification Azure Active Directory
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Découvrez comment vous connecter à SQL Database, SQL Managed Instance et Azure Synapse Analytics en utilisant une authentification Azure Active Directory (après avoir configuré Azure AD).
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse, has-adal-ref, sqldbrb=2, devx-track-azurecli
ms.devlang: ''
ms.topic: how-to
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto, sstein
ms.date: 08/17/2020
ms.openlocfilehash: 5558480f568e802637deb30c9f1b41c00826c9b5
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454497"
---
# <a name="configure-and-manage-azure-ad-authentication-with-azure-sql"></a>Configurer et gérer l’authentification Azure AD avec Azure SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

Cet article explique comment créer et remplir une instance Azure Active Directory (Azure AD), puis utiliser Azure AD avec [Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). Pour obtenir une vue d’ensemble, consultez [Authentification Azure Active Directory](authentication-aad-overview.md).

## <a name="azure-ad-authentication-methods"></a>Méthodes d’authentification Azure AD

L’authentification Azure AD prend en charge les méthodes d’authentification suivantes :

- Identités uniquement cloud Azure AD
- Identités hybrides Azure AD qui prennent en charge les méthodes d’authentification suivantes :
  - Authentification cloud avec deux options associées à une authentification unique (SSO) transparente
    - Authentification par hachage du mot de passe Azure AD
    - Authentification directe Azure AD
  - Authentification fédérée

Pour plus d’informations sur le choix d’une méthode d’authentification Azure AD, consultez [Choisir la méthode d’authentification adaptée à votre solution d’identité hybride Azure Active Directory](../../active-directory/hybrid/choose-ad-authn.md).

Pour plus d’informations sur les identités hybrides Azure AD, la configuration et la synchronisation, consultez :

- Authentification par hachage du mot de passe : [Implémenter la synchronisation de hachage du mot de passe avec la synchronisation Azure AD Connect](../../active-directory/hybrid/how-to-connect-password-hash-synchronization.md)
- Authentification directe : [Authentification directe Azure Active Directory](../../active-directory/hybrid/how-to-connect-pta-quick-start.md)
- Authentification fédérée : [Déploiement des services de fédération Active Directory (AD FS) dans Azure](/windows-server/identity/ad-fs/deployment/how-to-connect-fed-azure-adfs) et [Fédération avec Azure AD Connect](../../active-directory/hybrid/how-to-connect-fed-whatis.md)

## <a name="create-and-populate-an-azure-ad-instance"></a>Créer et renseigner une instance Azure AD

Créez une instance Azure AD et renseignez-la avec les utilisateurs et les groupes. Azure AD peut être le domaine managé Azure AD initial. Azure AD peut également être une instance locale de services de domaine Active Directory, fédérée avec l’annuaire Azure AD.

Pour plus d’informations, consultez [Intégration des identités locales avec Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md), [Ajout de votre propre nom de domaine à Azure AD](../../active-directory/fundamentals/add-custom-domain.md), [Microsoft Azure prend désormais en charge la fédération avec Windows Server Active Directory](https://azure.microsoft.com/blog/20../../windows-azure-now-supports-federation-with-windows-server-active-directory/), [Administration de votre annuaire Azure AD](../../active-directory/fundamentals/active-directory-whatis.md), [Gestion d’Azure AD à l’aide de Windows PowerShell](/powershell/azure/) et [Ports et protocoles nécessaires à l’identité hybride](../../active-directory/hybrid/reference-connect-ports.md).

## <a name="associate-or-add-an-azure-subscription-to-azure-active-directory"></a>Associer ou ajouter un abonnement Azure à Azure Active Directory

1. Pour associer votre abonnement Azure à Azure Active Directory, faites de l’annuaire un annuaire approuvé pour l’abonnement Azure qui héberge la base de données. Pour plus d’informations, consultez [Associer ou ajouter un abonnement Azure à votre locataire Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

2. Utilisez le sélecteur de répertoire dans le portail Azure pour basculer vers l’abonnement associé au domaine.

   > [!IMPORTANT]
   > Chaque abonnement Azure dispose d’une relation d’approbation avec une instance Azure AD. Cela signifie qu'il approuve ce répertoire pour authentifier les utilisateurs, les services et les appareils. Plusieurs abonnements peuvent approuver le même annuaire, mais un abonnement n’approuve qu’un seul annuaire. Cette relation de confiance qu’un abonnement possède avec un répertoire est contraire à celle établie entre un abonnement et toutes les autres ressources Azure (sites Web, bases de données, etc.), qui se rapprochent plus des ressources enfants d'un abonnement. Lorsqu’un abonnement expire, les autres ressources associées à l'abonnement deviennent également inaccessibles. Mais le répertoire reste dans Azure, et vous pouvez associer un autre abonnement à ce répertoire et continuer à gérer les utilisateurs du répertoire. Pour plus d’informations sur les ressources, consultez [Comprendre l’accès aux ressources dans Azure](../../active-directory/external-identities/add-users-administrator.md). Pour en savoir plus sur cette relation approuvée, consultez [Comment associer ou ajouter un abonnement Azure à Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="azure-ad-admin-with-a-server-in-sql-database"></a>Administrateur Azure AD avec serveur dans SQL Database

Dans Azure (qui héberge SQL Database ou Azure Synapse), chaque [serveur](logical-servers.md) démarre avec un compte d’administrateur de serveur unique qui est l’administrateur de l’ensemble du serveur. Créez un deuxième compte Administrateur en tant que compte Azure AD. Cet utilisateur principal est créé en tant qu’utilisateur de base de données autonome dans la base de données MASTER du serveur. Les comptes administrateur de serveur sont des membres du rôle **db_owner** de chaque base de données utilisateur, puis saisissez chaque base de données utilisateur en tant utilisateur **dbo**. Pour plus d'informations sur les comptes administrateur, consultez [Gestion des bases de données et des connexions](logins-create-manage.md).

Lorsque vous utilisez Azure Active Directory avec la géo-réplication, le compte administrateur de Microsoft Azure Active Directory doit être configuré pour le serveur principal et le serveur secondaire. Si un serveur ne dispose pas d’un administrateur Azure Active Directory, les utilisateurs Azure Active Directory reçoivent un message d’erreur `Cannot connect` au serveur.

> [!NOTE]
> Les utilisateurs qui ne sont pas basés sur un compte Azure AD (y compris le compte administrateur de serveur) ne peuvent pas créer d’utilisateurs Azure AD, car ils n’ont pas l’autorisation de valider des utilisateurs de base de données proposés avec Azure AD.

## <a name="provision-azure-ad-admin-sql-managed-instance"></a>Approvisionner un administrateur Azure AD (SQL Managed Instance)

> [!IMPORTANT]
> Suivez ces étapes uniquement si vous approvisionnez une instance Azure SQL Managed Instance. Cette opération peut être exécutée uniquement par l’administrateur global/de la société ou un administrateur de rôle privilégié dans Azure AD.
>
> Dans la **préversion publique**, vous pouvez affecter le rôle **Lecteurs de répertoire** à un groupe dans Azure AD. Les propriétaires du groupe peuvent ensuite ajouter l’identité de l’instance gérée en tant que membre de ce groupe, ce qui vous permet d’approvisionner un administrateur Azure AD pour SQL Managed Instance. Pour plus d’informations sur cette fonctionnalité, consultez [Rôle Lecteurs de répertoire dans Azure Active Directory pour Azure SQL](authentication-aad-directory-readers-role.md).

Votre instance SQL Managed Instance a besoin d’autorisations de lecture pour Azure AD afin d’effectuer correctement des tâches telles que l’authentification des utilisateurs via l’appartenance au groupe de sécurité ou la création de nouveaux utilisateurs. Pour ce faire, vous devez accorder l’autorisation SQL Managed Instance de lecture pour Azure AD. Vous pouvez effectuer cette opération dans le portail Azure ou à l’aide de PowerShell.

### <a name="azure-portal"></a>Portail Azure

Pour accorder à votre instance SQL Managed Instance Azure AD une autorisation de lecture à l’aide du Portail Azure, connectez-vous en tant qu’administrateur global/de la société dans Azure AD et procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), dans le coin supérieur droit, sélectionnez votre connexion à partir d’une liste déroulante de répertoires Active Directories possibles.

2. Choisissez l’annuaire Active Directory approprié en tant qu’Azure AD par défaut.

   Cette étape lie l’abonnement associé à Active Directory à SQL Managed Instance, en s’assurant que le même abonnement est bien utilisé pour l’instance Azure AD et SQL Managed Instance.

3. Accédez à l’instance SQL Managed Instance que vous souhaitez utiliser pour l’intégration d’Azure AD.

   ![Capture d'écran du portail Azure sur lequel la page d'administration d'Active Directory est ouverte pour l'instance SQL Managed Instance sélectionnée.](./media/authentication-aad-configure/aad.png)

4. Sélectionnez la bannière en haut de la page d’administration Active Directory et accordez les autorisations à l’utilisateur actuel.

    ![Capture d'écran de la boîte de dialogue permettant d'accorder des autorisations à une instance SQL Managed Instance pour l'accès à Active Directory. Le bouton Accorder des autorisations est sélectionné.](./media/authentication-aad-configure/grant-permissions.png)

5. Lorsque l’opération a réussi, la notification suivante s’affichera dans le coin supérieur droit :

    ![Capture d'écran d'une notification confirmant que les autorisations de lecture Active Directory ont été correctement mises à jour pour l'instance gérée.](./media/authentication-aad-configure/success.png)

6. Vous pouvez maintenant choisir votre administrateur Azure AD pour votre Instance SQL Managed Instance. Pour cela, à la page Administrateur Active Directory, sélectionnez la commande **Définir l’administrateur**.

    ![Capture d'écran représentant la commande Définir l'administrateur en surbrillance sur la page d'administration d'Active Directory pour l'instance SQL Managed Instance sélectionnée.](./media/authentication-aad-configure/set-admin.png)

7. Sur la page d’administration Azure AD, recherchez un utilisateur, sélectionnez l’utilisateur ou le groupe à définir en tant qu’administrateur, puis choisissez **Sélectionner**.

   La page Administrateur Active Directory affiche tous les membres et groupes présents dans Active Directory. Les utilisateurs ou les groupes grisés ne peuvent pas être sélectionnés, car ils ne sont pas pris en charge en tant qu’administrateurs Azure AD. Voir la liste des administrateurs pris en charge dans [Fonctionnalités et limitations Azure AD](authentication-aad-overview.md#azure-ad-features-and-limitations). Le contrôle d’accès basé sur les rôles Azure (Azure RBAC) s'applique uniquement au portail Azure et n’est pas propagé vers SQL Database, SQL Managed Instance ou Azure Synapse.

    ![Ajouter un administrateur Azure Active Directory](./media/authentication-aad-configure/add-azure-active-directory-admin.png)

8. En haut de la page Administrateur Active Directory, sélectionnez **Enregistrer**.

    ![Capture d'écran de la page d'administration d'Active Directory, avec le bouton Enregistrer sur la ligne supérieure en regard des boutons Définir l'administrateur et Supprimer l'administrateur.](./media/authentication-aad-configure/save.png)

    La procédure de changement de l’administrateur peut prendre plusieurs minutes. Le nouvel administrateur apparaît dans la zone Administrateur Active Directory.

Après avoir approvisionné un administrateur Azure AD pour votre instance SQL Managed Instance, vous pouvez commencer à créer des principaux de serveur (connexions) Azure AD avec la syntaxe <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN</a>. Pour plus d’informations, consultez [Vue d’ensemble de SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md#azure-active-directory-integration).

> [!TIP]
> Pour supprimer un administrateur, en haut de la page Administrateur Active Directory, sélectionnez **Supprimer l’administrateur**, puis **Enregistrer**.

### <a name="powershell"></a>PowerShell

Pour accorder à votre instance SQL Managed Instance Azure AD une autorisation de lecture à l’aide de PowerShell, exécutez le script suivant :

```powershell
# Gives Azure Active Directory read permission to a Service Principal representing the SQL Managed Instance.
# Can be executed only by a "Company Administrator", "Global Administrator", or "Privileged Role Administrator" type of user.

$aadTenant = "<YourTenantId>" # Enter your tenant ID
$managedInstanceName = "MyManagedInstance"

# Get Azure AD role "Directory Users" and create if it doesn't exist
$roleName = "Directory Readers"
$role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
if ($role -eq $null) {
    # Instantiate an instance of the role template
    $roleTemplate = Get-AzureADDirectoryRoleTemplate | Where-Object {$_.displayName -eq $roleName}
    Enable-AzureADDirectoryRole -RoleTemplateId $roleTemplate.ObjectId
    $role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
}

# Get service principal for your SQL Managed Instance
$roleMember = Get-AzureADServicePrincipal -SearchString $managedInstanceName
$roleMember.Count
if ($roleMember -eq $null) {
    Write-Output "Error: No Service Principals with name '$    ($managedInstanceName)', make sure that managedInstanceName parameter was     entered correctly."
    exit
}
if (-not ($roleMember.Count -eq 1)) {
    Write-Output "Error: More than one service principal with name pattern '$    ($managedInstanceName)'"
    Write-Output "Dumping selected service principals...."
    $roleMember
    exit
}

# Check if service principal is already member of readers role
$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
$selDirReader = $allDirReaders | where{$_.ObjectId -match     $roleMember.ObjectId}

if ($selDirReader -eq $null) {
    # Add principal to readers role
    Write-Output "Adding service principal '$($managedInstanceName)' to     'Directory Readers' role'..."
    Add-AzureADDirectoryRoleMember -ObjectId $role.ObjectId -RefObjectId     $roleMember.ObjectId
    Write-Output "'$($managedInstanceName)' service principal added to     'Directory Readers' role'..."

    #Write-Output "Dumping service principal '$($managedInstanceName)':"
    #$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
    #$allDirReaders | where{$_.ObjectId -match $roleMember.ObjectId}
}
else {
    Write-Output "Service principal '$($managedInstanceName)' is already     member of 'Directory Readers' role'."
}
```

### <a name="powershell-for-sql-managed-instance"></a>PowerShell pour SQL Managed Instance

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour exécuter les applets de commande PowerShell, Azure PowerShell doit être installé et en cours d’exécution. Pour plus de détails, consultez la rubrique [Installation et configuration d’Azure PowerShell](/powershell/azure/).

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager (RM) est toujours pris en charge par Azure SQL Managed Instance, mais tous les développements à venir sont destinés au module Az.Sql. Le module AzureRM continue à recevoir des résolutions de bogues jusqu’à au moins décembre 2020.  Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm. Pour en savoir plus sur leur compatibilité, consultez [Présentation du nouveau module Az Azure PowerShell](/powershell/azure/new-azureps-module-az).

Pour configurer un administrateur Azure AD, exécutez les commandes Azure PowerShell suivantes :

- Connect-AzAccount
- Select-AzSubscription

Les cmdlets utilisées pour configurer et gérer l’administrateur Azure AD pour votre instance SQL Managed Instance sont répertoriées dans le tableau suivant :

| Nom de l’applet de commande | Description |
| --- | --- |
| [Set-AzSqlInstanceActiveDirectoryAdministrator](/powershell/module/az.sql/set-azsqlinstanceactivedirectoryadministrator) |Approvisionne un administrateur Azure AD pour l’instance SQL Managed Instance dans l’abonnement actuel. (Doit être à partir de l’abonnement actuel)|
| [Remove-AzSqlInstanceActiveDirectoryAdministrator](/powershell/module/az.sql/remove-azsqlinstanceactivedirectoryadministrator) |Supprime un administrateur Azure AD pour l’instance SQL Managed Instance dans l’abonnement actuel. |
| [Get-AzSqlInstanceActiveDirectoryAdministrator](/powershell/module/az.sql/get-azsqlinstanceactivedirectoryadministrator) |Retourne des informations sur un administrateur Azure AD pour l’instance SQL Managed Instance dans l’abonnement actuel.|

La commande suivante obtient des informations sur un administrateur Azure AD pour une instance SQL Managed Instance nommée ManagedInstance01 associée à un groupe de ressources nommé ResourceGroup01.

```powershell
Get-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstance01"
```

La commande suivante approvisionne un groupe d’administrateurs Azure AD nommé DBA pour l’instance SQL Managed Instance nommée ManagedInstance01. Ce serveur est associé au groupe de ressources ResourceGroup01.

```powershell
Set-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstance01" -DisplayName "DBAs" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353b"
```

La commande suivante supprime l’administrateur Azure AD pour l’instance SQL Managed Instance nommée ManagedInstanceName01 associée au groupe de ressources ResourceGroup01.

```powershell
Remove-AzSqlInstanceActiveDirectoryAdministrator -ResourceGroupName "ResourceGroup01" -InstanceName "ManagedInstanceName01" -Confirm -PassThru
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Vous pouvez également approvisionner un administrateur Azure AD pour l’instance SQL Managed Instance en appelant les commandes CLI suivantes :

| Commande | Description |
| --- | --- |
|[az sql mi ad-admin create](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-create) | Approvisionne un administrateur Azure Active Directory pour l’instance SQL Managed Instance dans l’abonnement actuel. |
|[az sql mi ad-admin delete](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-delete) | Supprime un administrateur Azure Active Directory pour l’instance SQL Managed Instance. |
|[az sql mi ad-admin list](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-list) | Retourne des informations sur un administrateur Azure Active Directory actuellement configuré pour l’instance SQL Managed Instance. |
|[az sql mi ad-admin update](/cli/azure/sql/mi/ad-admin#az-sql-mi-ad-admin-update) | Met à jour l’administrateur Azure Active Directory pour l’instance SQL Managed Instance. |

Pour plus d’informations sur les commandes CLI, consultez [az sql mi](/cli/azure/sql/mi).

* * *

## <a name="provision-azure-ad-admin-sql-database"></a>Approvisionner un administrateur Azure AD (SQL Database)

> [!IMPORTANT]
> Suivez ces étapes uniquement si vous approvisionnez un [serveur](logical-servers.md) pour SQL Database ou Azure Synapse.

Les deux procédures suivantes vous montrent comment approvisionner un administrateur d’Active Directory Azure pour votre serveur dans le portail Azure et à l’aide de PowerShell.

### <a name="azure-portal"></a>Portail Azure

1. Dans le [portail Azure](https://portal.azure.com/), dans le coin supérieur droit, sélectionnez votre connexion pour développer une liste déroulante de répertoires Active Directories potentiels. Choisissez l’annuaire Active Directory approprié en tant qu’Azure AD par défaut. Cette étape lie l’association de l’abonnement avec Active Directory et le serveur, ce qui garantit que le même abonnement est utilisé à la fois pour Azure AD et le serveur.

2. Recherchez et sélectionnez **Serveur SQL**.

    ![Rechercher et sélectionner des serveurs SQL](./media/authentication-aad-configure/search-for-and-select-sql-servers.png)

    >[!NOTE]
    > Dans cette page, avant de choisir **Serveurs SQL**, vous pouvez sélectionner l **’étoile** en regard du nom pour ajouter la catégorie à vos *favoris* puis placer **Serveurs SQL** dans la barre de navigation gauche.

3. Sur la page **SQL Server**, sélectionnez **Administrateur Active Directory**.

4. À la page **Administrateur Active Directory**, sélectionnez **Définir l’administrateur**.

    ![Les serveurs SQL définissent l’administrateur Active Directory](./media/authentication-aad-configure/sql-servers-set-active-directory-admin.png)  

5. À la page **Ajouter un administrateur**, recherchez un utilisateur, sélectionnez l’utilisateur ou le groupe à définir en tant qu’administrateur, puis choisissez **Sélectionner**. (La page Administrateur Active Directory affiche tous les membres et groupes présents dans Active Directory. Les utilisateurs ou les groupes grisés ne peuvent être sélectionnés, car ils ne sont pas pris en charge en tant qu’administrateurs Azure AD. (Consultez la liste des administrateurs de prise en charge dans la section **Fonctionnalités et limitations d’Azure AD** de l’article [Utiliser l’authentification Azure Active Directory pour l’authentification auprès de SQL Database ou d’Azure Synapse](authentication-aad-overview.md).) Le contrôle d'accès basé sur les rôles Azure (Azure RBAC) s'applique uniquement au portail et n'est pas propagé vers SQL Server.

    ![Sélectionner un administrateur Azure Active Directory](./media/authentication-aad-configure/select-azure-active-directory-admin.png)  

6. En haut de la page **Administrateur Active Directory**, sélectionnez **ENREGISTRER**.

    ![enregistrer l’administrateur](./media/authentication-aad-configure/save-admin.png)

La procédure de changement de l’administrateur peut prendre plusieurs minutes. Le nouvel administrateur apparaîtra dans la zone **Administrateur Active Directory** .

   > [!NOTE]
   > Lors de la configuration de l’administrateur Azure AD, le nom du nouvel administrateur (utilisateur ou groupe) ne peut pas déjà être présent dans la base de données MASTER virtuelle en tant qu’utilisateur de l’authentification serveur. Si tel est le cas, la configuration de l’administrateur d’Azure AD échoue, annulant sa création et indiquant que cet administrateur (ce nom) existe déjà. Dans la mesure où un utilisateur de l’authentification serveur ne fait pas partie d’Azure AD, tout effort pour se connecter au serveur à l’aide de l’authentification Azure AD échoue.

Pour supprimer un administrateur, en haut de la page **Administrateur Active Directory**, sélectionnez **Supprimer l’administrateur**, puis **Enregistrer**.

### <a name="powershell-for-sql-database-and-azure-synapse"></a>PowerShell pour SQL Database et Azure Synapse

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour exécuter les applets de commande PowerShell, Azure PowerShell doit être installé et en cours d’exécution. Pour plus de détails, consultez la rubrique [Installation et configuration d’Azure PowerShell](/powershell/azure/). Pour configurer un administrateur Azure AD, exécutez les commandes Azure PowerShell suivantes :

- Connect-AzAccount
- Select-AzSubscription

Cmdlets utilisées pour approvisionner et gérer l’administrateur Azure AD pour SQL Database et Azure Synapse :

| Nom de l’applet de commande | Description |
| --- | --- |
| [Set-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/set-azsqlserveractivedirectoryadministrator) |Approvisionne un administrateur Azure Active Directory pour le serveur hébergeant SQL Database ou Azure Synapse. (Doit être à partir de l’abonnement actuel) |
| [Remove-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/remove-azsqlserveractivedirectoryadministrator) |Supprime un administrateur Azure Active Directory pour le serveur hébergeant SQL Database ou Azure Synapse.|
| [Get-AzSqlServerActiveDirectoryAdministrator](/powershell/module/az.sql/get-azsqlserveractivedirectoryadministrator) |Renvoie les informations sur un administrateur Azure Active Directory actuellement configuré pour le serveur hébergeant SQL Database ou Azure Synapse. |

Utilisez la commande Get-help PowerShell pour obtenir plus de détails sur chacune de ces commandes. Par exemple : `get-help Set-AzSqlServerActiveDirectoryAdministrator`.

Le script suivant configure un groupe d’administrateurs Azure AD nommé **DBA_Group** (ID d’objet `40b79501-b343-44ed-9ce7-da4c8cc7353f`) pour le serveur **demo_server** d’un groupe de ressources nommé **Group-23** :

```powershell
Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" -DisplayName "DBA_Group"
```

Le paramètre d’entrée **DisplayName** accepte le nom d’affichage Azure AD ou le nom principal de l’utilisateur. Par exemple, ``DisplayName="John Smith"`` et ``DisplayName="johns@contoso.com"``. Pour les groupes Azure AD, seul le nom d’affichage est pris en charge.

> [!NOTE]
> La commande Azure PowerShell ```Set-AzSqlServerActiveDirectoryAdministrator``` ne vous empêche pas de configurer des administrateurs Azure AD pour des utilisateurs non pris en charge. Un utilisateur non pris en charge peut être configuré, mais il ne peut pas se connecter à une base de données.

L'exemple suivant utilise l' **ObjectID** facultatif en option :

```powershell
Set-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" `
    -DisplayName "DBA_Group" -ObjectId "40b79501-b343-44ed-9ce7-da4c8cc7353f"
```

> [!NOTE]
> **L’ObjectID** Azure AD est requis lorsque le **DisplayName** n’est pas unique. Pour récupérer les valeurs **ObjectID** et **DisplayName**, utilisez la section Active Directory du portail Azure Classic et affichez les propriétés d’un utilisateur ou d’un groupe.

L’exemple suivant renvoie des informations sur l’administrateur Azure AD admin pour le serveur :

```powershell
Get-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server" | Format-List
```

L’exemple suivant supprime un administrateur Azure AD :

```powershell
Remove-AzSqlServerActiveDirectoryAdministrator -ResourceGroupName "Group-23" -ServerName "demo_server"
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Vous pouvez configurer un administrateur Azure AD en appelant les commandes CLI suivantes :

| Commande | Description |
| --- | --- |
|[az sql server ad-admin create](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-create) | Approvisionne un administrateur Azure Active Directory pour le serveur hébergeant SQL Database ou Azure Synapse. (Doit être à partir de l’abonnement actuel) |
|[az sql server ad-admin delete](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-delete) | Supprime un administrateur Azure Active Directory pour le serveur hébergeant SQL Database ou Azure Synapse. |
|[az sql server ad-admin list](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-list) | Renvoie les informations sur un administrateur Azure Active Directory actuellement configuré pour le serveur hébergeant SQL Database ou Azure Synapse. |
|[az sql server ad-admin update](/cli/azure/sql/server/ad-admin#az-sql-server-ad-admin-update) | Met à jour un administrateur Azure Active Directory pour le serveur hébergeant SQL Database ou Azure Synapse. |

Pour plus d’informations sur les commandes CLI, consultez [az sql server](/cli/azure/sql/server).

* * *

> [!NOTE]
> Vous pouvez également approvisionner un administrateur Azure Active Directory à l’aide de l’API REST. Pour plus d’informations, consultez [Informations de référence sur l’API REST de gestion des services et Opérations pour Azure SQL Database](/rest/api/sql/)

## <a name="configure-your-client-computers"></a>Configurer vos ordinateurs clients

Sur toutes les machines clientes à partir desquelles vos applications ou utilisateurs se connectent à SQL Database ou Azure Synapse à l’aide d’identités Azure AD, vous devez installer les logiciels suivants :

- .NET Framework 4.6 ou version ultérieure à partir de [https://msdn.microsoft.com/library/5a4x27ek.aspx](/dotnet/framework/install/guide-for-developers).
- Bibliothèque d’authentification Microsoft Active Directory pour SQL Server (*ADAL.DLL*). Voici les liens de téléchargement permettant d’installer la dernière version de SSMS, ODBC et le pilote OLE DB contenant la bibliothèque *ADAL.DLL*.
  - [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms)
  - [ODBC Driver 17 for SQL Server](https://www.microsoft.com/download/details.aspx?id=56567)
  - [OLE DB Driver 18 for SQL Server](https://www.microsoft.com/download/details.aspx?id=56730)

Vous pouvez répondre à ces exigences en procédant comme suit :

- L’installation de la dernière version de [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) ou [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt) respecte la configuration requise de .NET Framework 4.6.
  - SSMS installe la version x86 de *ADAL.DLL*.
  - SSDT installe la version amd64 de *ADAL.DLL*.
  - La dernière version de Visual Studio dans les [Téléchargements Visual Studio](https://www.visualstudio.com/downloads/download-visual-studio-vs) respecte la configuration requise de .NET Framework 4.6, mais n’installe pas la version amd64 nécessaire de *ADAL.DLL*.

## <a name="create-contained-users-mapped-to-azure-ad-identities"></a>Créer des utilisateurs à relation contenant-contenu mappés à des identités Azure AD

SQL Managed Instance prenant en charge les principaux de serveur Azure AD (connexions), le recours à des utilisateurs de base de données autonome n’est pas obligatoire. Les principaux de serveur (connexions) Azure AD vous permettent de créer des connexions à partir d’utilisateurs, de groupes ou d’applications Azure AD. Dès lors, vous pouvez vous authentifier auprès de votre instance SQL Managed Instance à l’aide de la connexion du serveur Azure AD plutôt que d’un utilisateur de base de données autonome. Pour plus d’informations, consultez [Vue d’ensemble de SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md#azure-active-directory-integration). Pour la syntaxe de création des principaux de serveur (connexions) Azure AD, consultez <a href="/sql/t-sql/statements/create-login-transact-sql?view=azuresqldb-mi-current">CREATE LOGIN</a>.

Toutefois, l’utilisation de l’authentification Azure Active Directory avec SQL Database et Azure Synapse requiert le recours à des utilisateurs de base de données autonome basés sur une identité Azure AD. Un utilisateur de base de données autonome ne dispose pas de connexion dans la base de données MASTER et est mappé à une identité dans Azure AD associée à la base de données. L’identité Azure AD peut être un compte d’utilisateur individuel ou un groupe. Pour plus d’informations sur les utilisateurs de base de données autonome, consultez [Utilisateurs de base de données - Rendre votre base de données portable](/sql/relational-databases/security/contained-database-users-making-your-database-portable).

> [!NOTE]
> Il n’est pas possible de créer des utilisateurs de base de données (à l’exception d’administrateurs) via le portail Azure. Les rôles Azure ne sont pas propagés à la base de données dans SQL Database, SQL Managed Instance ou Azure Synapse. Les rôles Azure sont utilisés pour la gestion des ressources Azure et ne s’appliquent pas aux autorisations de base de données. Par exemple, le rôle **Contributeur SQL Server** ne permet pas de se connecter à la base de données dans SQL Database, SQL Managed Instance ou Azure Synapse. L’accès doit être accordé directement dans la base de données à l’aide d’instructions Transact-SQL.

> [!WARNING]
> Les caractères spéciaux comme le deux-points `:` ou l’esperluette `&`, lorsqu’ils sont inclus dans les instructions T-SQL `CREATE LOGIN` et `CREATE USER`, ne sont pas pris en charge.

Pour créer un utilisateur de base de données autonome Azure AD (autre que l’administrateur du serveur propriétaire de la base de données), connectez-vous à la base de données avec une identité Azure AD en tant qu’utilisateur avec au moins l’autorisation **MODIFIER UN UTILISATEUR**. Utilisez ensuite la syntaxe Transact-SQL suivante :

```sql
CREATE USER <Azure_AD_principal_name> FROM EXTERNAL PROVIDER;
```

*Azure_AD_principal_name* peut être le nom d’utilisateur principal d’un utilisateur Azure AD ou le nom d’affichage d’un groupe Azure AD.

**Exemples :** pour créer une base de données autonome représentant un utilisateur de domaine fédéré ou géré Azure AD :

```sql
CREATE USER [bob@contoso.com] FROM EXTERNAL PROVIDER;
CREATE USER [alice@fabrikam.onmicrosoft.com] FROM EXTERNAL PROVIDER;
```

Pour créer un utilisateur de base de données autonome représentant un groupe de domaine Azure AD ou fédéré, définissez le nom complet d’un groupe de sécurité :

```sql
CREATE USER [ICU Nurses] FROM EXTERNAL PROVIDER;
```

Pour créer un utilisateur de base de données autonome représentant une application qui se connecte à l’aide d’un jeton Azure AD :

```sql
CREATE USER [appName] FROM EXTERNAL PROVIDER;
```

> [!NOTE]
> Cette commande impose que SQL accède à Azure AD (le « fournisseur externe ») pour le compte de l’utilisateur connecté. Parfois, les circonstances font qu’Azure AD retourne une exception à SQL. Dans ce cas, l’utilisateur verra l’erreur SQL 33134, qui doit contenir le message d’erreur propre à Azure AD. La plupart du temps, l’erreur indique que l’accès est refusé, ou que l’utilisateur doit s’inscrire pour l’authentification multifacteur afin d’accéder à la ressource, ou que l’accès entre les applications internes doit être géré par le biais de l’autorisation préalable. Dans les deux premiers cas, le problème est généralement dû à des stratégies d’accès conditionnel définies dans le locataire Azure AD de l’utilisateur : elles l’empêchent d’accéder au fournisseur externe. La mise à jour des stratégies d’accès conditionnel de façon à autoriser l’accès à l’application « 00000002-0000-0000-c000-000000000000 » (ID d’application de l’API Graph Azure AD) doit résoudre le problème. Si l’erreur indique que l’accès entre les applications internes doit être géré par le biais de l’autorisation préalable, le problème vient du fait que l’utilisateur est connecté en tant que principal du service. La commande doit aboutir si au lieu de cela elle est exécutée par un utilisateur.

> [!TIP]
> Vous ne pouvez pas créer directement un utilisateur à partir d’un annuaire Azure Active Directory autre que l’annuaire Azure Active Directory associé à votre abonnement Azure. Toutefois, les membres d’autres annuaires Active Directory qui sont des utilisateurs importés dans l’annuaire Active Directory associé (appelés utilisateurs externes) peuvent être ajoutés à un groupe Active Directory dans le client Active Directory. En créant un utilisateur de base de données autonome pour ce groupe AD, les utilisateurs de l’annuaire Active Directory externe peuvent accéder SQL Database.

Pour plus d’informations sur la création d’utilisateurs de base de données autonome basés sur des identités Azure Active Directory, voir [CRÉER UN UTILISATEUR (Transact-SQL)](/sql/t-sql/statements/create-user-transact-sql).

> [!NOTE]
> La suppression de l’administrateur Azure Active Directory pour le serveur empêche tout utilisateur de l’authentification Azure AD de se connecter au serveur. Si nécessaire, des utilisateurs Azure AD inutilisables peuvent être supprimés manuellement par un administrateur du service Base de données SQL.

> [!NOTE]
> Si vous recevez le message **Délai d’expiration de la connexion dépassé**, vous devrez peut-être définir le paramètre `TransparentNetworkIPResolution` de la chaîne de connexion sur la valeur false. Pour plus d’informations, consultez [Problème lié au délai d’expiration de la connexion avec .NET Framework 4.6.1 - TransparentNetworkIPResolution](/archive/blogs/dataaccesstechnologies/connection-timeout-issue-with-net-framework-4-6-1-transparentnetworkipresolution).

Lorsque vous créez un utilisateur de base de données, il reçoit l’autorisation **CONNECT** et peut se connecter à cette base de données en tant que membre du rôle **PUBLIC**. À l'origine, les seules autorisations disponibles pour l'utilisateur sont celles qui sont accordées au rôle **PUBLIC** ou aux groupes Azure AD dont il est membre. Une fois que vous avez configuré un utilisateur de base de données autonome Azure AD, vous pouvez octroyer à cet utilisateur des autorisations supplémentaires, de la même façon que vous accordez l’autorisation à un autre type d’utilisateur. En général, on accorde les autorisations aux rôles de base de données, puis on ajoute des utilisateurs aux rôles. Pour plus d’informations, consultez [Notions de base sur les autorisations de moteur de base de données](https://social.technet.microsoft.com/wiki/contents/articles/4433.database-engine-permission-basics.aspx). Pour plus d'informations sur les rôles de base de données SQL, consultez [Gestion des bases de données et des connexions dans Azure SQL Database](logins-create-manage.md).
Un compte d’utilisateur de domaine fédéré importé dans un domaine managé comme utilisateur externe doit utiliser l’identité de domaine managé.

> [!NOTE]
> Les utilisateurs AD Azure sont marqués dans les métadonnées de la base de données avec le type E (EXTERNAL_USER) et pour les groupes avec le type X (EXTERNAL_GROUPS). Pour plus d’informations, consultez [sys.database_principals](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql).

## <a name="connect-to-the-database-using-ssms-or-ssdt"></a>Se connecter à la base de données à l’aide de SSMS ou SSDT  

Pour vérifier que l’administrateur Azure AD est correctement configuré, connectez-vous à la base de données **master** en utilisant un compte d’administrateur Azure AD.
Pour configurer un utilisateur de base de données autonome Azure AD (autre que l’administrateur de serveur propriétaire de la base de données), connectez-vous à la base de données avec une identité Azure AD ayant accès à la base de données.

> [!IMPORTANT]
> La prise en charge de l’authentification Azure Active Directory est disponible avec [SQL Server 2016 Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) et [SQL Server Data Tools](/sql/ssdt/download-sql-server-data-tools-ssdt) dans Visual Studio 2015. La version d’août 2016 de SSMS inclut également la prise en charge de l’authentification universelle Active Directory, qui permet aux administrateurs d’exiger l’authentification multifacteur par appel téléphonique, SMS, cartes à puce avec code PIN ou notification d’application mobile.

## <a name="using-an-azure-ad-identity-to-connect-using-ssms-or-ssdt"></a>Utilisation d’une identité Azure AD pour se connecter à l’aide de SSMS ou de SSDT

Les procédures suivantes vous montrent comment se connecter à SQLDatabase avec une identité Azure AD à l’aide de SQL Server Management Studio ou de SQL Server Database Tools.

### <a name="active-directory-integrated-authentication"></a>Authentification intégrée d'Active Directory

Utilisez cette méthode si vous vous êtes connecté à Windows à l’aide de vos informations d’identification Azure Active Directory à partir d’un domaine fédéré ou d’un domaine managé configuré pour une authentification unique fluide à des fins d’authentification directe et d’authentification par hachage du mot de passe. Pour plus d’informations, consultez la page [Authentification unique transparente Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).

1. Démarrez Management Studio ou Data Tools et, dans la boîte de dialogue **Se connecter au serveur** (ou **Se connecter au moteur de base de données**), dans la zone **Authentification**, sélectionnez **Azure Active Directory – Authentification intégrée**. Aucun mot de passe n’est nécessaire ou ne peut être saisi, car les informations d’identification existantes sont présentées pour la connexion.

   ![Sélectionner l’authentification intégrée AD][11]

2. Sélectionnez le bouton **Options** puis, la page **Propriétés de connexion**, dans la zone **Se connecter à la base de données**, tapez le nom de la base de données utilisateur à laquelle vous souhaitez vous connecter. Pour plus d’informations, consultez l’article [Authentification Azure AD multifacteur](authentication-mfa-ssms-overview.md#azure-ad-domain-name-or-tenant-id-parameter) concernant les différences entre les propriétés de connexion pour SSMS 17.x et 18.x.

   ![Sélectionner le nom de la base de données][13]

### <a name="active-directory-password-authentication"></a>Authentification par mot de passe Active Directory

Utilisez cette méthode lors de la connexion avec un nom principal Azure AD à l’aide du domaine géré d’Azure AD. Vous pouvez également l’utiliser pour des comptes fédérés sans accéder au domaine, par exemple, lorsque vous travaillez à distance.

Utilisez cette méthode pour authentifier auprès de la base de données dans SQL Database ou SQL Managed Instance des utilisateurs d’identité uniquement cloud Azure AD, ou des personnes utilisant des identités hybrides Azure AD. Cette méthode prend en charge les utilisateurs désireux d’utiliser leurs informations d’identification Windows, mais dont l’ordinateur local n’est pas joint au domaine (par exemple, à l’aide d’un accès à distance). Dans ce cas, un utilisateur Windows peut indiquer ses compte de domaine et mot de passe, et s’authentifier auprès de la base de données dans SQL Database, SQL Managed Instance ou Azure Synapse.

1. Démarrez Management Studio ou Data Tools et, dans la boîte de dialogue **Se connecter au serveur** (ou **Se connecter au moteur de base de données**), dans la zone **Authentification**, sélectionnez **Azure Active Directory – Authentification par mot de passe**.

2. Dans la zone **Nom d’utilisateur** tapez votre nom d’utilisateur Azure Active Directory au format **nom_utilisateur\@domaine.com**. Les noms d’utilisateurs doivent correspondre à un compte Azure Active Directory ou à un compte d’un domaine managé ou fédéré avec Azure Active Directory.

3. Dans la zone **Mot de passe** , tapez votre mot de passe utilisateur pour le compte Azure Active Directory ou le compte de domaine managé/fédéré.

    ![Sélectionner l’authentification par mot de passe AD][12]

4. Sélectionnez le bouton **Options** puis, la page **Propriétés de connexion**, dans la zone **Se connecter à la base de données**, tapez le nom de la base de données utilisateur à laquelle vous souhaitez vous connecter. (Voir le graphique dans l’option précédente.)

### <a name="active-directory-interactive-authentication"></a>Authentification interactive Active Directory

Utilisez cette méthode pour une authentification interactive avec ou sans Multi-Factor Authentication (MFA), et avec le mot de passe demandé de manière interactive. Cette méthode peut être utilisée pour authentifier auprès de la base de données dans SQL Database, SQL Managed Instance et Azure Synapse des utilisateurs d’identité uniquement cloud Azure AD, ou des personnes utilisant des identités hybrides Azure AD.

Pour plus d’informations, consultez [Utilisation de l’authentification Azure AD multifacteur avec SQL Database et Azure Synapse (prise en charge de SSMS pour MFA)](authentication-mfa-ssms-overview.md).

## <a name="using-an-azure-ad-identity-to-connect-from-a-client-application"></a>Utilisation d’une identité Azure AD pour se connecter à partir d’une application cliente

Les procédures suivantes vous montrent comment se connecter à une instance SQL Database avec une identité Azure AD à partir d’une application cliente.

### <a name="active-directory-integrated-authentication"></a>Authentification intégrée d'Active Directory

Pour utiliser l’Authentification Windows intégrée, l’Active Directory de votre domaine doivent être fédéré avec Azure Active Directory ou être un domaine managé configuré pour l’authentification unique transparente, l’authentification directe ou l’authentification par hachage du mot de passe. Pour plus d’informations, consultez la page [Authentification unique transparente Azure Active Directory](../../active-directory/hybrid/how-to-connect-sso.md).

> [!NOTE]
> [MSAL.NET (Microsoft.Identity.Client)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#roadmap) pour l’authentification Windows intégrée n’est pas pris en charge pour une authentification unique fluide à des fins d’authentification directe et d’authentification par hachage du mot de passe.

Votre application cliente (ou un service) se connectant à la base de données doit être en cours d’exécution sur un ordinateur joint au domaine dont les informations d’identification de domaine sont celles d’un utilisateur.

Pour une connexion à une base de données à l’aide de l’authentification intégrée et d’une identité Azure AD, le mot clé d’authentification dans la chaîne de connexion de base de données doit avoir la valeur `Active Directory Integrated`. L’exemple de code C# suivant utilise ADO .NET.

```csharp
string ConnectionString = @"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Integrated; Initial Catalog=testdb;";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

Le mot clé de la chaîne de connexion `Integrated Security=True` n’est pas pris en charge pour la connexion à Azure SQL Database. Lorsque vous établissez une connexion ODBC, vous devez supprimer les espaces et définir l’authentification sur « ActiveDirectoryIntegrated ».

### <a name="active-directory-password-authentication"></a>Authentification par mot de passe Active Directory

Pour une connexion à une base de données à l’aide de comptes d’utilisateur d’identité uniquement cloud Azure AD, le mot clé d’authentification avoir la valeur `Active Directory Password`. La chaîne de connexion doit contenir les mots clés et valeurs d’ID utilisateur/UID et de mot de passe/PWD. L’exemple de code C# suivant utilise ADO .NET.

```csharp
string ConnectionString =
@"Data Source=n9lxnyuzhv.database.windows.net; Authentication=Active Directory Password; Initial Catalog=testdb;  UID=bob@contoso.onmicrosoft.com; PWD=MyPassWord!";
SqlConnection conn = new SqlConnection(ConnectionString);
conn.Open();
```

Pour en savoir plus sur les méthodes d’authentification Azure AD, utilisez les exemples de code de démonstration disponibles dans [Démonstration GitHub de l’authentification Azure AD](https://github.com/Microsoft/sql-server-samples/tree/master/samples/features/security/azure-active-directory-auth).

## <a name="azure-ad-token"></a>Jeton Azure AD

Cette méthode d’authentification permet à des services de niveau intermédiaire d’obtenir des [Jetons web JSON (JWT)](../../active-directory/develop/id-tokens.md) pour se connecter à la base de données dans SQL Database, SQL Managed Instance ou Azure Synapse en obtenant un jeton d’Azure AD. Cette méthode autorise divers scénarios d’application incluant des identités de service, des principaux de service et des applications utilisant une authentification basée sur des certificats. Vous devez effectuer quatre étapes de base pour utiliser l’authentification par jeton Azure AD :

1. Inscrivez votre application auprès d’Azure Active Directory et obtenez l’ID client de votre code.
2. Créez un utilisateur de base de données qui représente l’application. (Effectué à l’étape 6.)
3. Créez un certificat sur l’ordinateur client qui exécute l’application.
4. Ajoutez le certificat en tant que clé pour votre application.

Exemple de chaîne de connexion :

```csharp
string ConnectionString =@"Data Source=n9lxnyuzhv.database.windows.net; Initial Catalog=testdb;"
SqlConnection conn = new SqlConnection(ConnectionString);
conn.AccessToken = "Your JWT token"
conn.Open();
```

Pour plus d’informations, consultez le [Blog de sécurité de SQL Server](/archive/blogs/sqlsecurity/token-based-authentication-support-for-azure-sql-db-using-azure-ad-auth). Pour plus d’informations sur l’ajout de certificat, consultez [Bien démarrer avec l’authentification par certificat dans Azure Active Directory](../../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md).

### <a name="sqlcmd"></a>sqlcmd

Les instructions suivantes permettent de se connecter à l’aide de la version 13.1 de sqlcmd, qui est disponible dans le [Centre de téléchargement](https://www.microsoft.com/download/details.aspx?id=53591).

> [!NOTE]
> `sqlcmd` avec la commande `-G` ne fonctionne pas avec les identités système et nécessite une connexion du principal de l’utilisateur.

```cmd
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net -G  
sqlcmd -S Target_DB_or_DW.testsrv.database.windows.net -U bob@contoso.com -P MyAADPassword -G -l 30
```

## <a name="troubleshoot-azure-ad-authentication"></a>Résoudre les problèmes d’authentification Azure AD

Vous trouverez des conseils pour résoudre les problèmes liés à l’authentification Azure AD dans le blog suivant : <https://techcommunity.microsoft.com/t5/azure-sql-database/troubleshooting-problems-related-to-azure-ad-authentication-with/ba-p/1062991>

## <a name="next-steps"></a>Étapes suivantes

- Pour une vue d’ensemble des connexions, des utilisateurs, des rôles de base de données et des autorisations dans SQL Database, consultez [Connexions, utilisateurs, rôles de base de données et comptes d’utilisateur](logins-create-manage.md).
- Pour en savoir plus sur les principaux de base de données, voir [Principaux](/sql/relational-databases/security/authentication-access/principals-database-engine).
- Pour en savoir plus sur les rôles de base de données, voir [Rôles de base de données](/sql/relational-databases/security/authentication-access/database-level-roles).
- Pour en savoir plus sur les règles de pare-feu dans la base de données SQL, voir [Règles de pare-feu de la base de données SQL](firewall-configure.md).
- Pour savoir comment définir un utilisateur invité Azure AD comme administrateur Azure AD, consultez [Créer des utilisateurs invités Azure AD et les définir comme administrateur Azure AD](authentication-aad-guest-users.md).
- Pour obtenir des informations sur l’utilisation de principaux de service avec Azure SQL, consultez [Créer des utilisateurs Azure AD à l’aide d’applications Azure AD](authentication-aad-service-principal-tutorial.md)

<!--Image references-->

[11]: ./media/authentication-aad-configure/active-directory-integrated.png
[12]: ./media/authentication-aad-configure/12connect-using-pw-auth2.png
[13]: ./media/authentication-aad-configure/13connect-to-db2.png
