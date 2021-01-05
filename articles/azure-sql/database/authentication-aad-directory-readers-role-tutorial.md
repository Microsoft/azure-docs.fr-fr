---
title: Attribuer le rôle Lecteurs d’annuaires à un groupe Azure AD et gérer les attributions de rôles
description: Cet article vous guide tout au long de l’activation du rôle Lecteurs d’annuaires à l’aide de groupes Azure AD pour gérer les attributions de rôles Azure AD avec Azure SQL Database, Azure SQL Managed Instance et Azure Synapse Analytics
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse
ms.topic: tutorial
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 08/14/2020
ms.openlocfilehash: 88483b29c8951f8e3f38f7cdc5bbdfb80eeca2b1
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92370115"
---
# <a name="tutorial-assign-directory-readers-role-to-an-azure-ad-group-and-manage-role-assignments"></a>Tutoriel : Attribuer le rôle Lecteurs d’annuaires à un groupe Azure AD et gérer les attributions de rôles

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

> [!NOTE]
> L’attribution du rôle **Lecteurs d’annuaires** à un groupe décrite dans cet article est disponible en **préversion publique**. 

Cet article vous guide tout au long de la création d’un groupe dans Azure Active Directory (Azure AD) et de l’affectation de ce groupe au rôle [**Lecteurs d’annuaires**](../../active-directory/roles/permissions-reference.md#directory-readers). Les autorisations Lecteurs d’annuaires permettent aux propriétaires de groupe d’ajouter des membres au groupe, par exemple, une [identité managée](../../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) d’[Azure SQL Database](sql-database-paas-overview.md), [Azure SQL Managed Instance](../managed-instance/sql-managed-instance-paas-overview.md) et [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md). Ainsi, il n’est pas nécessaire qu’un [administrateur général](../../active-directory/roles/permissions-reference.md#global-administrator--company-administrator) ou un [administrateur de rôle privilégié](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) attribue le rôle Lecteurs d’annuaires directement pour chaque identité de serveur logique Azure SQL dans le locataire.

Ce tutoriel utilise la fonctionnalité introduite dans [Utiliser des groupes cloud pour gérer les attributions de rôles dans Azure Active Directory (préversion)](../../active-directory/roles/groups-concept.md). 

Pour plus d’informations sur les avantages de l’attribution du rôle Lecteurs d’annuaires à un groupe Azure AD pour Azure SQL, consultez [Rôle Lecteurs d’annuaires dans Azure Active Directory pour Azure SQL](authentication-aad-directory-readers-role.md).

## <a name="prerequisites"></a>Prérequis

- Une instance Azure AD. Pour plus d’informations, consultez [Configurer et gérer l’authentification Azure AD avec Azure SQL](authentication-aad-configure.md).
- Une instance SQL Database, SQL Managed Instance ou Azure Synapse.

## <a name="directory-readers-role-assignment-using-the-azure-portal"></a>Attribution du rôle Lecteurs d’annuaires à l’aide du portail Azure

### <a name="create-a-new-group-and-assign-owners-and-role"></a>Créer un groupe et affecter des propriétaires et un rôle

1. Un utilisateur disposant des autorisations [Administrateur général](../../active-directory/roles/permissions-reference.md#global-administrator--company-administrator) ou [Administrateur de rôle privilégié](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) est nécessaire pour cette configuration initiale.
1. En tant qu’utilisateur privilégié, connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez à la ressource **Azure Active Directory** . Sous **Géré**, accédez à **Groupes**. Sélectionnez **Nouveau groupe** pour créer un groupe.
1. Sélectionnez **Sécurité** comme type de groupe et renseignez le reste des champs. Assurez-vous que le paramètre **Des rôles Azure AD peuvent être attribués au groupe (préversion)** est défini sur **Oui**. Ensuite, affectez au groupe le rôle Azure AD **Lecteurs d’annuaires**.
1. Affectez des utilisateurs Azure AD en tant que propriétaire(s) au groupe qui a été créé. Un propriétaire de groupe peut être un utilisateur AD ordinaire sans rôle administratif d’Azure AD. Le propriétaire doit être un utilisateur qui gère votre instance SQL Database, SQL Managed Instance ou Azure Synapse.

   :::image type="content" source="media/authentication-aad-directory-readers-role/new-group.png" alt-text="Ajouter un nouveau groupe":::

1. Sélectionnez **Créer**

### <a name="checking-the-group-that-was-created"></a>Vérification du groupe qui a été créé

> [!NOTE]
> Assurez-vous que le **Type de groupe** est **Sécurité**. Les groupes *Microsoft 365* ne sont pas pris en charge pour Azure SQL.

Pour vérifier et gérer le groupe qui a été créé, revenez au volet **Groupes** dans le portail Azure et recherchez le nom de votre groupe. Après avoir sélectionné votre groupe, vous pouvez ajouter des propriétaires et des membres sous le menu **Propriétaires** et **Membres** du paramètre **Gérer**. Vous pouvez également consulter les **Rôles affectés** pour le groupe.

:::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-group-created.png" alt-text="Ajouter un nouveau groupe":::

### <a name="add-azure-sql-managed-identity-to-the-group"></a>Ajouter une identité managée Azure SQL au groupe

> [!NOTE]
> Nous utilisons SQL Managed Instance pour cet exemple, mais vous pouvez obtenir les mêmes résultats en suivant des étapes similaires pour SQL Database ou Azure Synapse.

Pour les étapes suivantes, l’utilisateur doté de l’autorisation Administrateur général ou Administrateur de rôle privilégié n’est plus nécessaire.

1. Connectez-vous au portail Azure en tant qu’utilisateur gérant l’instance SQL Managed Instance, qui est propriétaire du groupe créé.

1. Recherchez le nom votre ressource **SQL Managed Instance** dans le portail Azure.

   :::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-managed-instance.png" alt-text="Ajouter un nouveau groupe":::

   Lors de la création de votre instance SQL Managed Instance, une identité Azure a été créée pour celle-ci. L’identité créée porte le même nom que le préfixe du nom de votre instance SQL Managed Instance. Pour trouver le principal de service de l’identité SQL Managed Instance qui a été créée en tant qu’application Azure AD, suivez ces étapes :

    - Accédez à la ressource **Azure Active Directory** . Sous le paramètre **Gérer**, sélectionnez **Applications d’entreprise**. L’**ID d’objet** est l’identité de l’instance.
    
    :::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-managed-instance-service-principal.png" alt-text="Ajouter un nouveau groupe":::

1. Accédez à la ressource **Azure Active Directory** . Sous **Géré**, accédez à **Groupes**. Sélectionnez le groupe que vous avez créé. Sous le paramètre **Géré** de votre groupe, sélectionnez **Membres**. Sélectionnez **Ajouter des membres** et ajoutez le principal de service de votre instance SQL Managed Instance en tant que membre du groupe en recherchant le nom trouvé ci-dessus.

   :::image type="content" source="media/authentication-aad-directory-readers-role/azure-ad-add-managed-instance-service-principal.png" alt-text="Ajouter un nouveau groupe":::

> [!NOTE]
> Plusieurs minutes peuvent être nécessaires pour propager les autorisations du principal de service via le système Azure et autoriser l’accès à l’API Graph Azure AD. Vous devrez peut-être patienter quelques minutes avant de provisionner un administrateur Azure AD pour SQL Managed Instance.

### <a name="remarks"></a>Notes

Pour SQL Database et Azure Synapse, l’identité du serveur peut être créée lors de la création du serveur logique SQL Azure ou après celle-ci. Pour plus d’informations sur la création ou la définition de l’identité de serveur dans une instance SQL Database ou Azure Synapse, consultez [Permettre aux principaux de service de créer des utilisateurs Azure AD](authentication-aad-service-principal.md#enable-service-principals-to-create-azure-ad-users).

Pour SQL Managed Instance, vous devez affecter le rôle **Lecteurs d’annuaires** à l’identité de l’instance managée avant de pouvoir [configurer un administrateur Azure AD pour l’instance managée](authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

L’attribution du rôle **Lecteurs d’annuaires** à l’identité du serveur n’est pas nécessaire pour SQL Database ou Azure Synapse lors de la configuration d’un administrateur Azure AD pour le serveur logique. Cependant, pour permettre la création d’un objet Azure AD dans SQL Database ou Azure Synapse pour le compte d’une application Azure AD, le rôle **Lecteurs d’annuaires** est requis. Si le rôle n’est pas affecté à l’identité du serveur logique SQL, la création d’utilisateurs Azure AD dans Azure SQL échoue. Pour plus d’informations, consultez [Principal de service Azure Active Directory avec Azure SQL](authentication-aad-service-principal.md).

## <a name="directory-readers-role-assignment-using-powershell"></a>Attribution du rôle Lecteurs d’annuaires à l’aide de PowerShell

> [!IMPORTANT]
> Un [administrateur général](../../active-directory/roles/permissions-reference.md#global-administrator--company-administrator) ou un [administrateur de rôle privilégié](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) doit exécuter ces étapes initiales. En plus de PowerShell, Azure AD offre l’API Microsoft Graph pour [créer un groupe avec attribution de rôle dans Azure AD](../../active-directory/roles/groups-create-eligible.md#using-microsoft-graph-api).

1. Téléchargez le module PowerShell Azure AD Preview à l’aide des commandes suivantes. Vous devrez peut-être exécuter PowerShell en tant qu’administrateur.

    ```powershell
    Install-Module azureadpreview
    Import-Module azureadpreview
    #To verify that the module is ready to use, use the following command:
    Get-Module azureadpreview
    ```

1. Connectez-vous à votre locataire Azure AD.

    ```powershell
    Connect-AzureAD
    ```

1. Créez un groupe de sécurité pour affecter le rôle **Lecteurs d’annuaires**.

    - Vous pouvez changer `DirectoryReaderGroup`, `Directory Reader Group`et `DirRead` en fonction de vos préférences.

    ```powershell
    $group = New-AzureADMSGroup -DisplayName "DirectoryReaderGroup" -Description "Directory Reader Group" -MailEnabled $False -SecurityEnabled $true -MailNickName "DirRead" -IsAssignableToRole $true
    $group
    ```

1. Attribuer le rôle **Lecteurs d’annuaires** au groupe.

    ```powershell
    # Displays the Directory Readers role information
    $roleDefinition = Get-AzureADMSRoleDefinition -Filter "displayName eq 'Directory Readers'" 
    $roleDefinition
    ```

    ```powershell
    # Assigns the Directory Readers role to the group
    $roleAssignment = New-AzureADMSRoleAssignment -ResourceScope '/' -RoleDefinitionId $roleDefinition.Id -PrincipalId $group.Id
    $roleAssignment
    ```

1. Attribuez des propriétaires au groupe.

    - Remplacez `<username>` par l’utilisateur qui doit posséder ce groupe. Vous pouvez ajouter plusieurs propriétaires en répétant ces étapes.

    ```powershell
    $RefObjectID = Get-AzureADUser -ObjectId "<username>"
    $RefObjectID
    ```

    ```powershell
    $GrOwner = Add-AzureADGroupOwner -ObjectId $group.ID -RefObjectId $RefObjectID.ObjectID
    ```

    Vérifiez les propriétaires du groupe à l’aide de la commande suivante :

    ```powershell
    Get-AzureADGroupOwner -ObjectId $group.ID
    ```

    Vous pouvez également vérifier les propriétaires du groupe dans le [portail Azure](https://portal.azure.com). Suivez les étapes indiquées dans [Vérification du groupe qui a été créé](#checking-the-group-that-was-created).

### <a name="assigning-the-service-principal-as-a-member-of-the-group"></a>Attribution du principal de service en tant que membre du groupe

Pour les étapes suivantes, l’utilisateur doté de l’autorisation Administrateur général ou Administrateur de rôle privilégié n’est plus nécessaire.

1. À l’aide d’un propriétaire du groupe, qui gère également la ressource Azure SQL, exécutez la commande suivante pour vous connecter à votre locataire Azure AD.

    ```powershell
    Connect-AzureAD
    ```

1. Attribuez le principal de service en tant que membre du groupe que vous avez créé.

    - Remplacez `<ServerName>` par le nom de votre serveur logique Azure SQL ou par le nom de votre instance Managed Instance. Pour plus d’informations, consultez la section [Ajouter une identité managée Azure SQL au groupe](#add-azure-sql-managed-identity-to-the-group)

    ```powershell
    # Returns the service principal of your Azure SQL resource
    $miIdentity = Get-AzureADServicePrincipal -SearchString "<ServerName>"
    $miIdentity
    ```

    ```powershell
    # Adds the service principal to the group as a member
    Add-AzureADGroupMember -ObjectId $group.ID -RefObjectId $miIdentity.ObjectId 
    ```

    La commande suivante retourne l’ID d’objet du principal de service indiquant que ce dernier a été ajouté au groupe :

    ```powershell
    Add-AzureADGroupMember -ObjectId $group.ID -RefObjectId $miIdentity.ObjectId
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Rôle Lecteurs d’annuaires dans Azure Active Directory pour Azure SQL](authentication-aad-directory-readers-role.md)
- [Tutoriel : Créer des utilisateurs Azure AD avec des applications Azure AD](authentication-aad-service-principal-tutorial.md)
- [Configurer et gérer l’authentification Azure AD avec Azure SQL](authentication-aad-configure.md)