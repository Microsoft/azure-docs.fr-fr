---
title: 'Azure AD Connect : Configurer les autorisations du compte de connecteur AD DS | Microsoft Docs'
description: Ce document explique en détail comment configurer le compte de connecteur AD DS à l’aide du nouveau module PowerShell ADSyncConfig
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/18/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 62bfc528886767bc09159ca2a2696c8c9264b307
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349937"
---
# <a name="azure-ad-connectconfigure-ad-ds-connector-account-permissions"></a>Azure AD Connect : Configurer les autorisations du compte de connecteur AD DS 

Le module PowerShell dénommé [ADSyncConfig.psm1](reference-connect-adsyncconfig.md) a été introduit avec la version 1.1.880.0 (sortie en août 2018). Il fournit une collection de cmdlets pour vous aider à configurer les autorisations Active Directory adéquates pour votre déploiement Azure AD Connect. 

## <a name="overview"></a>Vue d’ensemble 
Les applets de commande PowerShell suivantes vous permettent de définir les autorisations du compte de connecteur AD DS, pour chaque fonctionnalité que vous sélectionnez pour l’activer dans Azure AD Connect. Pour éviter tout problème, vous devez définir les autorisations Active Directory préalablement à toute installation d’Azure AD Connect à l’aide d’un compte de domaine personnalisé, pour pouvoir ensuite vous connecter à votre forêt. Ce module ADSyncConfig est également utile pour configurer les autorisations après le déploiement d’Azure AD Connect.

![présentation d’un compte ad ds](media/how-to-connect-configure-ad-ds-connector-account/configure1.png)

Dans le cadre de l’installation Express d’Azure AD Connect, un compte généré automatiquement (MSOL_nnnnnnnnnn) est créé dans Active Directory avec toutes les autorisations nécessaires. Vous n’avez donc pas besoin du module ADSyncConfig, sauf si vous avez bloqué l’héritage des autorisations sur les unités d’organisation ou sur des objets Active Directory spécifiques que vous souhaitez synchroniser dans Azure AD. 
 
### <a name="permissions-summary"></a>Résumé des autorisations 
Le tableau suivant récapitule les autorisations nécessaires sur les objets AD : 

| Fonctionnalité | Autorisations |
| --- | --- |
| Fonctionnalité de ms-DS-ConsistencyGuid |Autorisations de lecture et d’écriture sur l’attribut ms-DS-ConsistencyGuid documenté dans [Principes de conception Azure AD Connect - Utilisation de ms-DS-ConsistencyGuid en tant qu’attribut sourceAnchor](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor). | 
| Synchronisation de hachage de mot de passe |<li>Répliquer les changements d’annuaires</li>  <li>Répliquer les changements d’annuaire Tout |
| Déploiement Exchange hybride |Autorisations de lecture et d’écriture sur les attributs documentés dans [Écriture différée d’Exchange hybride](reference-connect-sync-attributes-synchronized.md#exchange-hybrid-writeback) pour les utilisateurs, les groupes et les contacts. |
| Dossier public de messagerie Exchange |Autorisations de lecture sur les attributs documentées dans [Dossier public de messagerie Exchange](reference-connect-sync-attributes-synchronized.md#exchange-mail-public-folder) pour les dossiers publics. | 
| Réécriture du mot de passe |Autorisations de lecture et d’écriture sur les attributs documentés dans [Bien démarrer avec la gestion des mots de passe](../authentication/tutorial-enable-sspr-writeback.md) pour les utilisateurs. |
| Écriture différée des appareils |Autorisations de lecture et d’écriture sur les objets et conteneurs d’appareil documentés dans la [réécriture d’appareil](how-to-connect-device-writeback.md). |
| Écriture différée de groupe |Lire, créer, mettre à jour et supprimer des objets de groupe pour les **groupes Office 365** synchronisés.|

## <a name="using-the-adsyncconfig-powershell-module"></a>Utilisation du module PowerShell ADSyncConfig 
Le module ADSyncConfig nécessite les [outils d’administration de serveur distant pour AD DS](/windows-server/remote/remote-server-administration-tools), car il dépend des outils et du module PowerShell AD DS. Pour installer les outils d’administration de serveur distant pour AD DS, ouvrez une fenêtre Windows PowerShell avec l’option « Exécuter en tant qu’administrateur » et exécutez la commande suivante : 

``` powershell
Install-WindowsFeature RSAT-AD-Tools 
```
![Configurer](media/how-to-connect-configure-ad-ds-connector-account/configure2.png)

>[!NOTE]
>Vous pouvez également copier le fichier **C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\ADSyncConfig.psm1** sur un contrôleur de domaine où sont déjà installés les outils d’administration de serveur distant pour AD DS et utiliser ensuite ce module PowerShell à partir de ce contrôleur.

Pour commencer à utiliser le module ADSyncConfig, vous devez d’abord le charger dans une fenêtre Windows PowerShell : 

``` powershell
Import-Module "C:\Program Files\Microsoft Azure Active Directory Connect\AdSyncConfig\AdSyncConfig.psm1" 
```

Pour afficher toutes les applets de commande incluses dans ce module, entrez la commande suivante :  

``` powershell
Get-Command -Module AdSyncConfig  
```

![Vérification](media/how-to-connect-configure-ad-ds-connector-account/configure3.png)

Chaque applet de commande utilise les mêmes paramètres d’entrée du compte de connecteur AD DS, ainsi qu’un commutateur AdminSDHolder. Pour spécifier votre compte de connecteur AD DS, entrez le nom du compte et le domaine correspondants, ou entrez simplement le nom unique du compte.

Par exemple :

```powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountName <ADAccountName> -ADConnectorAccountDomain <ADDomainName>
```

Ou :

```powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountDN <ADAccountDN>
```

Veillez à remplacer `<ADAccountName>`, `<ADDomainName>` et `<ADAccountDN>` par les valeurs de votre environnement.

Si vous ne souhaitez pas changer les autorisations sur le conteneur AdminSDHolder, utilisez le commutateur `-SkipAdminSdHolders`. 

Par défaut, toutes les applets de commande de définition des autorisations tentent de définir les autorisations AD DS à la racine de chaque domaine dans la forêt. L’utilisateur ayant ouvert la session PowerShell doit donc avoir des droits d’administrateur de domaine sur chaque domaine dans la forêt.  En raison de cette exigence, il est recommandé d’utiliser un compte Administrateur d’entreprise à partir de la racine de la forêt. Si votre déploiement Azure AD Connect utilise plusieurs connecteurs AD DS, la même applet de commande doit être exécutée sur chaque forêt contenant un connecteur AD DS. 

Vous pouvez également définir des autorisations sur une unité d’organisation ou un objet AD DS spécifique en ajoutant le paramètre `-ADobjectDN` suivi du nom unique de l’objet cible sur lequel vous souhaitez définir les autorisations. Si vous utilisez un ADobjectDN cible, l’applet de commande définit les autorisations uniquement sur cet objet, et pas sur la racine du domaine ou le conteneur AdminSDHolder. Ce paramètre peut être utile si vous avez bloqué l’héritage des autorisations sur des unités d’organisation ou objets AD DS spécifiques (voir la section « Rechercher les objets AD DS pour lesquels l’héritage des autorisations est désactivé ») 

Il y a deux exceptions à ces paramètres communs : l’applet de commande `Set-ADSyncRestrictedPermissions`, qui s’utilise pour définir les autorisations sur le compte de connecteur AD DS lui-même, et l’applet de commande `Set-ADSyncPasswordHashSyncPermissions`, qui définit les autorisations nécessaires pour la synchronisation du hachage de mot de passe uniquement à la racine de domaine et n’inclut donc pas les paramètres `-ObjectDN` et `-SkipAdminSdHolders`.

### <a name="determine-your-ad-ds-connector-account"></a>Identifier votre compte de connecteur AD DS 
Si Azure AD Connect est déjà installé et que vous souhaitez savoir quel compte de connecteur AD DS est actuellement utilisé par Azure AD Connect, exécutez l’applet de commande suivante : 

``` powershell
Get-ADSyncADConnectorAccount 
```
### <a name="locate-ad-ds-objects-with-permission-inheritance-disabled"></a>Rechercher les objets AD DS pour lesquels l’héritage des autorisations est désactivé 
Si vous souhaitez savoir si l’héritage des autorisations a été désactivé pour certains objets AD DS, exécutez cette commande : 

``` powershell
Get-ADSyncObjectsWithInheritanceDisabled -SearchBase '<DistinguishedName>' 
```
Par défaut, cette applet de commande recherche uniquement les unités d’organisation pour lesquelles l’héritage a été désactivé, mais vous pouvez spécifier d’autres classes d’objets AD DS à l’aide du paramètre `-ObjectClass` ou spécifier toutes les classes d’objets en utilisant le caractère « * », comme ceci : 

``` powershell
Get-ADSyncObjectsWithInheritanceDisabled -SearchBase '<DistinguishedName>' -ObjectClass * 
```
 
### <a name="view-ad-ds-permissions-of-an-object"></a>Afficher les autorisations AD DS définies sur un objet 
Pour afficher la liste des autorisations actuellement définies sur un objet Active Directory, utilisez la cmdlet ci-dessous en indiquant le nom unique (DistinguishedName) de l’objet : 

``` powershell
Show-ADSyncADObjectPermissions -ADobjectDN '<DistinguishedName>' 
```

## <a name="configure-ad-ds-connector-account-permissions"></a>Configurer les autorisations du compte de connecteur AD DS 
 
### <a name="configure-basic-read-only-permissions"></a>Configurer des autorisations de lecture seule de base 
Si vous n’utilisez aucune fonctionnalité Azure AD Connect et souhaitez définir des autorisations de lecture seule de base pour le compte de connecteur AD DS , exécutez : 

``` powershell
Set-ADSyncBasicReadPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


Ou : 

``` powershell
Set-ADSyncBasicReadPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```


Cette applet de commande définit les autorisations suivantes : 
 

|Type |Nom |Accès |S'applique à| 
|-----|-----|-----|-----|
|Allow |Compte de connecteur AD DS |Lire toutes les propriétés |Objets appareil descendants| 
|Allow |Compte de connecteur AD DS|Lire toutes les propriétés |Objets InetOrgPerson descendants| 
|Allow |Compte de connecteur AD DS |Lire toutes les propriétés |Objets ordinateur descendants| 
|Allow |Compte de connecteur AD DS |Lire toutes les propriétés |Objets foreignSecurityPrincipal descendants| 
|Allow |Compte de connecteur AD DS |Lire toutes les propriétés |Objets groupe descendants| 
|Allow |Compte de connecteur AD DS |Lire toutes les propriétés |Objets utilisateur descendants| 
|Allow |Compte de connecteur AD DS |Lire toutes les propriétés |Objets contact descendants| 

 
### <a name="configure-ms-ds-consistency-guid-permissions"></a>Configurer les autorisations MS-DS-Consistency-Guid 
Pour définir les autorisations du compte de connecteur AD DS en utilisant l’attribut ms-Ds-Consistency-Guid comme ancre source (option « Laisser Azure gérer l’ancre source pour moi »), exécutez : 

``` powershell
Set-ADSyncMsDsConsistencyGuidPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```

Ou : 

``` powershell
Set-ADSyncMsDsConsistencyGuidPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```

Cette applet de commande définit les autorisations suivantes : 

|Type |Nom |Accès |S'applique à|
|-----|-----|-----|-----| 
|Allow|Compte de connecteur AD DS|Lecture/Écriture de la propriété|Objets utilisateur descendants|

### <a name="permissions-for-password-hash-synchronization"></a>Autorisations pour la synchronisation du hachage de mot de passe 
Pour définir les autorisations du compte de connecteur AD DS quand la synchronisation du hachage de mot de passe est activée, exécutez : 

``` powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [<CommonParameters>] 
```


Ou : 

``` powershell
Set-ADSyncPasswordHashSyncPermissions -ADConnectorAccountDN <String> [<CommonParameters>] 
```

Cette applet de commande définit les autorisations suivantes : 

|Type |Nom |Accès |S'applique à|
|-----|-----|-----|-----| 
|Allow |Compte de connecteur AD DS |Réplication des modifications de l’annuaire |Cet objet uniquement (racine du domaine)| 
|Allow |Compte de connecteur AD DS |Réplication de toutes les modifications de l’annuaire |Cet objet uniquement (racine du domaine)| 
  
### <a name="permissions-for-password-writeback"></a>Autorisations pour la réécriture du mot de passe 
Pour définir les autorisations du compte de connecteur AD DS quand la réécriture du mot de passe est activée, exécutez : 

``` powershell
Set-ADSyncPasswordWritebackPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


Ou :

``` powershell
Set-ADSyncPasswordWritebackPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```
Cette applet de commande définit les autorisations suivantes : 

|Type |Nom |Accès |S'applique à|
|-----|-----|-----|-----| 
|Allow |Compte de connecteur AD DS |Réinitialiser le mot de passe |Objets utilisateur descendants| 
|Allow |Compte de connecteur AD DS |Écriture de la propriété lockoutTime |Objets utilisateur descendants| 
|Allow |Compte de connecteur AD DS |Écriture de la propriété pwdLastSet |Objets utilisateur descendants| 

### <a name="permissions-for-group-writeback"></a>Autorisations pour la réécriture de groupe 
Pour définir les autorisations du compte de connecteur AD DS quand la réécriture de groupe est activée, exécutez : 

``` powershell
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```
Ou : 

``` powershell
Set-ADSyncUnifiedGroupWritebackPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>]
```
 
Cette applet de commande définit les autorisations suivantes : 

|Type |Nom |Accès |S'applique à|
|-----|-----|-----|-----| 
|Allow |Compte de connecteur AD DS |Lecture/Écriture générique |Tous les attributs d’un groupe de types d’objets et des sous-objets| 
|Allow |Compte de connecteur AD DS |Création/Suppression de l’objet enfant |Tous les attributs d’un groupe de types d’objets et des sous-objets| 
|Allow |Compte de connecteur AD DS |Suppression/Suppression d’objets d’arborescence|Tous les attributs d’un groupe de types d’objets et des sous-objets|

### <a name="permissions-for-exchange-hybrid-deployment"></a>Autorisations pour le déploiement Exchange hybride 
Pour définir les autorisations du compte de connecteur AD DS quand le déploiement Exchange hybride est activé, exécutez : 

``` powershell
Set-ADSyncExchangeHybridPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


Ou : 

``` powershell
Set-ADSyncExchangeHybridPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```

Cette applet de commande définit les autorisations suivantes :  
 

|Type |Nom |Accès |S'applique à|
|-----|-----|-----|-----| 
|Allow |Compte de connecteur AD DS |Lecture/écriture de toutes les propriétés |Objets utilisateur descendants| 
|Allow |Compte de connecteur AD DS |Lecture/écriture de toutes les propriétés |Objets InetOrgPerson descendants| 
|Allow |Compte de connecteur AD DS |Lecture/écriture de toutes les propriétés |Objets groupe descendants| 
|Allow |Compte de connecteur AD DS |Lecture/écriture de toutes les propriétés |Objets contact descendants| 

### <a name="permissions-for-exchange-mail-public-folders-preview"></a>Autorisations pour les dossiers publics de la messagerie Exchange (préversion) 
Pour définir les autorisations du compte de connecteur AD DS quand les dossiers publics de la messagerie Exchange sont activés, exécutez : 

``` powershell
Set-ADSyncExchangeMailPublicFolderPermissions -ADConnectorAccountName <String> -ADConnectorAccountDomain <String> [-SkipAdminSdHolders] [<CommonParameters>] 
```


Ou : 

``` powershell
Set-ADSyncExchangeMailPublicFolderPermissions -ADConnectorAccountDN <String> [-ADobjectDN <String>] [<CommonParameters>] 
```
Cette applet de commande définit les autorisations suivantes : 

|Type |Nom |Accès |S'applique à|
|-----|-----|-----|-----| 
|Allow |Compte de connecteur AD DS |Lire toutes les propriétés |Objets PublicFolder descendants| 

### <a name="restrict-permissions-on-the-ad-ds-connector-account"></a>Restreindre les autorisations du compte de connecteur AD DS 
Ce script PowerShell réduit les autorisations du compte de connecteur AD fourni comme paramètre. La réduction des autorisations implique les étapes suivantes : 

- Désactivez l’héritage sur l’objet spécifié 
- Supprimer toutes les entrées de contrôle d’accès (ACE) sur l’objet spécifique, à l’exception de celles propres à SELF, car nous souhaitons conserver les autorisations par défaut pour SELF. 
 
  Le paramètre -ADConnectorAccountDN est le compte AD dont les autorisations doivent être réduites. Il s’agit généralement du compte de domaine MSOL_nnnnnnnnnnnn qui est configuré dans le connecteur AD DS (voir la section « Identifier votre compte de connecteur AD DS »). Le paramètre -Credential est utilisé afin de spécifier le compte d’administrateur qui a les privilèges nécessaires pour restreindre les autorisations Active Directory sur l’objet AD cible. Il s’agit généralement du compte d’administrateur d’entreprise ou de domaine.  

``` powershell
Set-ADSyncRestrictedPermissions [-ADConnectorAccountDN] <String> [-Credential] <PSCredential> [-DisableCredentialValidation] [-WhatIf] [-Confirm] [<CommonParameters>] 
```
 
Par exemple : 

``` powershell
$credential = Get-Credential 
Set-ADSyncRestrictedPermissions -ADConnectorAccountDN'CN=ADConnectorAccount,CN=Users,DC=Contoso,DC=com' -Credential $credential  
```

Cette applet de commande définit les autorisations suivantes : 

|Type |Nom |Accès |S'applique à|
|-----|-----|-----|-----| 
|Allow |SYSTEM |Contrôle total |Cet objet 
|Allow |Administrateurs de l’entreprise |Contrôle total |Cet objet 
|Allow |Admins du domaine |Contrôle total |Cet objet 
|Allow |Administrateurs |Contrôle total |Cet objet 
|Allow |Contrôleurs de domaine d’entreprise |Lister le contenu |Cet objet 
|Allow |Contrôleurs de domaine d’entreprise |Lire toutes les propriétés |Cet objet 
|Allow |Contrôleurs de domaine d’entreprise |Autorisations de lecture |Cet objet 
|Allow |Utilisateurs authentifiés |Lister le contenu |Cet objet 
|Allow |Utilisateurs authentifiés |Lire toutes les propriétés |Cet objet 
|Allow |Utilisateurs authentifiés |Autorisations de lecture |Cet objet 

## <a name="next-steps"></a>Étapes suivantes
- [Azure AD Connect : comptes et autorisations](reference-connect-accounts-permissions.md)
- [Installation Express](how-to-connect-install-express.md)
- [Installation personnalisée](how-to-connect-install-custom.md)
- [Référence ADSyncConfig](reference-connect-adsyncconfig.md)