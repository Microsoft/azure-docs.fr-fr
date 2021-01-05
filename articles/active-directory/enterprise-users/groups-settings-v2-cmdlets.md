---
title: Exemples PowerShell V2 pour gérer des groupes – Azure AD | Microsoft Docs
description: Cette page fournit des exemples PowerShell pour vous aider à gérer vos groupes dans Azure Active Directory
keywords: Azure AD, Azure Active Directory, PowerShell, Groupes, Gestion des groupes
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: enterprise-users
ms.topic: how-to
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 12613362d9fc67dea14b41ece19a34febb489dda
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860641"
---
# <a name="azure-active-directory-version-2-cmdlets-for-group-management"></a>Cmdlets d’Azure Active Directory version 2 pour la gestion de groupe

> [!div class="op_single_selector"]
> - [Azure portal](../fundamentals/active-directory-groups-create-azure-portal.md?context=azure/active-directory/users-groups-roles/context/ugr-context)
> - [PowerShell](../enterprise-users/groups-settings-v2-cmdlets.md)
>
>

Cet article contient des exemples expliquant comment utiliser PowerShell pour gérer vos groupes dans Azure Active Directory (Azure AD).  Il fournit également des informations sur la configuration à l’aide du module Azure AD PowerShell. Vous devez tout d’abord [télécharger le module Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureAD/).

## <a name="install-the-azure-ad-powershell-module"></a>Installer le module Azure AD PowerShell

Pour installer le module PowerShell Azure AD, utilisez les commandes suivantes :

```powershell
    PS C:\Windows\system32> install-module azuread
    PS C:\Windows\system32> import-module azuread
```

Pour vérifier que le module est prêt à l’emploi, utilisez la commande suivante :

```powershell
    PS C:\Windows\system32> get-module azuread

    ModuleType Version      Name                                ExportedCommands
    ---------- ---------    ----                                ----------------
    Binary     2.0.0.115    azuread                      {Add-AzureADAdministrati...}
```

Vous pouvez désormais utiliser les applets de commande dans le module. Pour obtenir une description complète des applets de commande du module Azure AD, consultez la documentation de référence en ligne pour [Azure Active Directory PowerShell Version 2](/powershell/azure/active-directory/install-adv2).

> [!NOTE]
> Les cmdlets Azure AD PowerShell ne fonctionnent pas avec le nouveau PowerShell 7, car celui-ci est basé sur .NET Core. Nous sommes conscients qu’il s’agit d’un processus de mise à jour. À ce stade, nous vous suggérons d’utiliser le module Windows PowerShell 5.x pour les opérations Azure AD PowerShell. 


## <a name="connect-to-the-directory"></a>Se connecter à l’annuaire

Avant de pouvoir gérer des groupes à l’aide des applets de commande Azure AD PowerShell, vous devez connecter votre session PowerShell au répertoire que vous voulez gérer. Utilisez la commande suivante :

```powershell
    PS C:\Windows\system32> Connect-AzureAD
```

L’applet de commande vous demande de fournir les informations d’identification à utiliser pour accéder à votre répertoire. Dans cet exemple, nous utilisons karen@drumkit.onmicrosoft.com pour accéder au répertoire de démonstration. L’applet de commande renvoie une confirmation pour indiquer que la session a été correctement connectée à votre répertoire :

```powershell
    Account                       Environment Tenant ID
    -------                       ----------- ---------
    Karen@drumkit.onmicrosoft.com AzureCloud  85b5ff1e-0402-400c-9e3c-0f…
```

À présent, vous pouvez commencer à utiliser les applets de commande Azure AD pour gérer des groupes dans votre répertoire.

## <a name="retrieve-groups"></a>Récupérer des groupes

Pour récupérer des groupes existants à partir de votre annuaire, utilisez l’applet de commande Get-AzureADGroups. 

Pour récupérer tous les groupes dans le répertoire, utilisez l’applet de commande sans paramètres :

```powershell
    PS C:\Windows\system32> get-azureadgroup
```

L’applet de commande renvoie tous les groupes dans le répertoire connecté.

Vous pouvez utiliser le paramètre -objectID pour récupérer un groupe spécifique pour lequel vous spécifiez le paramètre objectID du groupe :

```powershell
    PS C:\Windows\system32> get-azureadgroup -ObjectId e29bae11-4ac0-450c-bc37-6dae8f3da61b
```

L’applet de commande renvoie à présent l’objectID correspond à la valeur du paramètre que vous avez saisi :

```powershell
    DeletionTimeStamp            :
    ObjectId                     : e29bae11-4ac0-450c-bc37-6dae8f3da61b
    ObjectType                   : Group
    Description                  :
    DirSyncEnabled               :
    DisplayName                  : Pacific NW Support
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 9bb4139b-60a1-434a-8c0d-7c1f8eee2df9
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

Vous pouvez rechercher un groupe spécifique en utilisant le paramètre -filter. Ce paramètre utilise une clause de filtre ODATA et renvoie tous les groupes qui correspondent au filtre, comme dans l’exemple suivant :

```powershell
    PS C:\Windows\system32> Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"


    DeletionTimeStamp            :
    ObjectId                     : 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
    ObjectType                   : Group
    Description                  : Intune Administrators
    DirSyncEnabled               :
    DisplayName                  : Intune Administrators
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 4dd067a0-6515-4f23-968a-cc2ffc2eff5c
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

> [!NOTE]
> Les applets de commande Azure AD PowerShell mettent en œuvre la norme de requête OData. Pour plus d’informations, consultez **$filter** dans [options de requête système OData à l’aide du point de terminaison OData](/previous-versions/dynamicscrm-2015/developers-guide/gg309461(v=crm.7)#BKMK_filter).

## <a name="create-groups"></a>Créer des groupes

Pour créer un nouveau groupe dans votre répertoire, utilisez l’applet de commande New-AzureADGroup. Cette applet de commande crée un nouveau groupe de sécurité appelé « Marketing » :

```powershell
    PS C:\Windows\system32> New-AzureADGroup -Description "Marketing" -DisplayName "Marketing" -MailEnabled $false -SecurityEnabled $true -MailNickName "Marketing"
```

## <a name="update-groups"></a>Mettre à jour des groupes

Pour mettre à jour un groupe existant, utilisez l’applet de commande Set-AzureADGroup. Dans cet exemple, nous modifions la propriété DisplayName du groupe « Administrateurs Intune ». Tout d’abord, nous recherchons le groupe à l’aide de l’applet de commande Get-AzureADGroup et filtrons à l’aide de l’attribut DisplayName :

```powershell
    PS C:\Windows\system32> Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"


    DeletionTimeStamp            :
    ObjectId                     : 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
    ObjectType                   : Group
    Description                  : Intune Administrators
    DirSyncEnabled               :
    DisplayName                  : Intune Administrators
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 4dd067a0-6515-4f23-968a-cc2ffc2eff5c
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

Ensuite, nous attribuons à la propriété Description la nouvelle valeur « Administrateurs d’appareil Intune » :

```powershell
    PS C:\Windows\system32> Set-AzureADGroup -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -Description "Intune Device Administrators"
```

À présent, si nous recherchons à nouveau le groupe, nous constatons que la propriété Description est mise à jour pour refléter la nouvelle valeur :

```powershell
    PS C:\Windows\system32> Get-AzureADGroup -Filter "DisplayName eq 'Intune Administrators'"

    DeletionTimeStamp            :
    ObjectId                     : 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
    ObjectType                   : Group
    Description                  : Intune Device Administrators
    DirSyncEnabled               :
    DisplayName                  : Intune Administrators
    LastDirSyncTime              :
    Mail                         :
    MailEnabled                  : False
    MailNickName                 : 4dd067a0-6515-4f23-968a-cc2ffc2eff5c
    OnPremisesSecurityIdentifier :
    ProvisioningErrors           : {}
    ProxyAddresses               : {}
    SecurityEnabled              : True
```

## <a name="delete-groups"></a>Supprimer des groupes

Pour supprimer des groupes de votre répertoire, utilisez l’applet de commande Remove-AzureADGroup comme suit :

```powershell
    PS C:\Windows\system32> Remove-AzureADGroup -ObjectId b11ca53e-07cc-455d-9a89-1fe3ab24566b
```

## <a name="manage-group-membership"></a>Gérer l’appartenance au groupe

### <a name="add-members"></a>Ajouter des membres

Pour ajouter de nouveaux membres à un groupe, utilisez l’applet de commande Add-AzureADGroupMember. Cette commande ajoute un membre au groupe Administrateurs Intune que nous avons utilisé dans l’exemple précédent :

```powershell
    PS C:\Windows\system32> Add-AzureADGroupMember -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -RefObjectId 72cd4bbd-2594-40a2-935c-016f3cfeeeea
```

Le paramètre -ObjectId est l’ObjectID du groupe auquel nous souhaitons ajouter un membre et le paramètre -RefObjectId est l’ObjectID de l’utilisateur que nous souhaitons ajouter au groupe en tant que membre.

### <a name="get-members"></a>Obtenir les membres

Pour obtenir les membres existants d’un groupe, utilisez l’applet de commande Get-AzureADGroupMember, comme dans cet exemple :

```powershell
    PS C:\Windows\system32> Get-AzureADGroupMember -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df

    DeletionTimeStamp ObjectId                             ObjectType
    ----------------- --------                             ----------
                          72cd4bbd-2594-40a2-935c-016f3cfeeeea User
                          8120cc36-64b4-4080-a9e8-23aa98e8b34f User
```

### <a name="remove-members"></a>Supprimer des membres

Pour supprimer le membre, que nous avons ajouté précédemment au groupe, utilisez l’applet de commande Remove-AzureADGroupMember, comme illustré ici :

```powershell
    PS C:\Windows\system32> Remove-AzureADGroupMember -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -MemberId 72cd4bbd-2594-40a2-935c-016f3cfeeeea
```

### <a name="verify-members"></a>Vérifier les membres

Pour vérifier l’appartenance d’un utilisateur à un groupe, utilisez l’applet de commande Select-AzureADGroupIdsUserIsMemberOf. Cette applet de commande utilise comme paramètres l’ObjectId de l’utilisateur pour lequel vérifier l’appartenance au groupe et une liste de groupes pour lesquels vérifier les appartenances. La liste des groupes doit être spécifiée sous la forme d’une variable complexe de type « Microsoft.Open.AzureAD.Model.GroupIdsForMembershipCheck », donc nous devons tout d’abord créer une variable de ce type :

```powershell
    PS C:\Windows\system32> $g = new-object Microsoft.Open.AzureAD.Model.GroupIdsForMembershipCheck
```

Ensuite, nous fournissons des valeurs pour les groupIds à vérifier dans l’attribut « GroupIds » de cette variable complexe :

```powershell
    PS C:\Windows\system32> $g.GroupIds = "b11ca53e-07cc-455d-9a89-1fe3ab24566b", "31f1ff6c-d48c-4f8a-b2e1-abca7fd399df"
```

À présent, pour vérifier les appartenances aux groupes d’un utilisateur avec l’ObjectID 72cd4bbd-2594-40a2-935c-016f3cfeeeea sur les groupes dans $g, nous devons utiliser :

```powershell
    PS C:\Windows\system32> Select-AzureADGroupIdsUserIsMemberOf -ObjectId 72cd4bbd-2594-40a2-935c-016f3cfeeeea -GroupIdsForMembershipCheck $g

    OdataMetadata                                                                                                 Value
    -------------                                                                                                  -----
    https://graph.windows.net/85b5ff1e-0402-400c-9e3c-0f9e965325d1/$metadata#Collection(Edm.String)             {31f1ff6c-d48c-4f8a-b2e1-abca7fd399df}
```

La valeur retournée est une liste de groupes dont cet utilisateur est membre. Vous pouvez également appliquer cette méthode pour vérifier l’appartenance aux Contacts, Groupes ou Principaux du service pour une liste donnée de groupes, à l’aide de Select-AzureADGroupIdsContactIsMemberOf, Select-AzureADGroupIdsGroupIsMemberOf ou Select-AzureADGroupIdsServicePrincipalIsMemberOf

## <a name="disable-group-creation-by-your-users"></a>Désactiver la création d’un groupe par vos utilisateurs

Vous pouvez empêcher des utilisateurs non-administrateurs de créer des groupes de sécurité. Le comportement par défaut dans Microsoft Online Directory Services (MSODS) consiste à permettre aux utilisateurs non-administrateurs de créer des groupes, que la gestion de groupes en libre-service soit ou non également activée. Le paramétrage de la gestion de groupes en libre-service contrôle ce comportement uniquement dans le panneau d’accès Mes applications.

Pour désactiver la création d’un groupe pour les utilisateurs non-administrateurs :

1. Vérifiez que les utilisateurs non-administrateurs sont autorisés à créer des groupes :
   
   ```powershell
   PS C:\> Get-MsolCompanyInformation | fl UsersPermissionToCreateGroupsEnabled
   ```
  
2. Si la commande retourne `UsersPermissionToCreateGroupsEnabled : True`, les utilisateurs non-administrateurs peuvent créer des groupes. Pour désactiver cette fonctionnalité :
  
   ```powershell 
   Set-MsolCompanySettings -UsersPermissionToCreateGroupsEnabled $False
   ```
  
## <a name="manage-owners-of-groups"></a>Gérer les propriétaires de groupes

Pour ajouter des propriétaires à un groupe, utilisez l’applet de commande Add-AzureADGroupOwner :

```powershell
    PS C:\Windows\system32> Add-AzureADGroupOwner -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -RefObjectId 72cd4bbd-2594-40a2-935c-016f3cfeeeea
```

Le paramètre -ObjectId est l’ObjectID du groupe auquel nous souhaitons ajouter un propriétaire et le paramètre -RefObjectId est l’ObjectID de l’utilisateur ou du principal du service que nous souhaitons ajouter au groupe en tant que propriétaire.

Pour récupérer les propriétaires d’un groupe, utilisez l’applet de commande Get-AzureADGroupOwner :

```powershell
    PS C:\Windows\system32> Get-AzureADGroupOwner -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df
```

L’applet de commande renvoie la liste des propriétaires (utilisateurs et principaux du service) du groupe spécifié :

```powershell
    DeletionTimeStamp ObjectId                             ObjectType
    ----------------- --------                             ----------
                          e831b3fd-77c9-49c7-9fca-de43e109ef67 User
```

Si vous souhaitez supprimer un propriétaire d’un groupe, utilisez l’applet de commande Remove-AzureADGroupOwner :

```powershell
    PS C:\Windows\system32> remove-AzureADGroupOwner -ObjectId 31f1ff6c-d48c-4f8a-b2e1-abca7fd399df -OwnerId e831b3fd-77c9-49c7-9fca-de43e109ef67
```

## <a name="reserved-aliases"></a>Alias réservés

Quand un groupe est créé, certain points de terminaison autorisent l’utilisateur final à spécifier un mailNickname ou alias à utiliser dans l’adresse e-mail du groupe.  Les groupes avec les alias de messagerie hautement privilégiés suivants peuvent uniquement être créés par un administrateur général Azure AD. 
  
* abuse
* admin
* administrator
* hostmaster
* majordomo
* postmaster
* root
* secure
* security
* ssl-admin
* webmaster

## <a name="group-writeback-to-on-premises-preview"></a>Écriture différée de groupe au niveau local (préversion)

De nos jours, de nombreux groupes sont toujours gérés localement dans Active Directory. Pour répondre à des requêtes visant à resynchroniser localement des groupes cloud, la fonctionnalité d’écriture différée dans les groupes Microsoft 365 pour Azure AD est désormais disponible en préversion.

Les groupes Microsoft 365 sont créés et gérés dans le cloud. La fonctionnalité d’écriture différée vous permet de mettre à jour les groupes Microsoft 365 en tant que groupes de distribution dans une forêt Active Directory sur laquelle est installé Exchange. Les utilisateurs disposant de boîtes aux lettres Exchange locales peuvent envoyer et recevoir des e-mails à partir de ces groupes. La fonctionnalité d’écriture différée de groupe ne prend pas en charge les groupes de sécurité ou les groupes de distribution Azure AD.

Pour plus d’informations, reportez-vous à la documentation relative au [service de synchronisation Azure AD Connect](../hybrid/how-to-connect-syncservice-features.md).

L’écriture différée de groupe Microsoft 365 est une fonctionnalité d’évaluation publique d’Azure Active Directory (Azure AD) qui est également disponible avec un plan de licence Azure AD payant. Pour plus d’informations légales sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="next-steps"></a>Étapes suivantes

Vous trouverez plus d’informations sur Azure Active Directory PowerShell dans la page dédiée aux [applets de commande Azure Active Directory](/powershell/azure/active-directory/install-adv2).

* [Gestion de l’accès aux ressources avec les groupes Azure Active Directory](../fundamentals/active-directory-manage-groups.md?context=azure/active-directory/users-groups-roles/context/ugr-context)
* [Intégration des identités locales dans Azure Active Directory](../hybrid/whatis-hybrid-identity.md?context=azure/active-directory/users-groups-roles/context/ugr-context)
