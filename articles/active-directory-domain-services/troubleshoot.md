---
title: Résolution des problèmes liés à Azure Active Directory Domain Services | Microsoft Docs'
description: Découvrez comment résoudre les erreurs qui se produisent couramment pendant la création ou la gestion des services Azure AD DS
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 4bc8c604-f57c-4f28-9dac-8b9164a0cf0b
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 89b04f86d41f8e4828580f70a9aec8acea3e0053
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618448"
---
# <a name="common-errors-and-troubleshooting-steps-for-azure-active-directory-domain-services"></a>Erreurs courantes et étapes de dépannage pour Azure Active Directory Domain Services

En tant qu’élément central de l’identité et de l’authentification des applications, Azure Active Directory Domain Services (Azure AD DS) rencontre parfois des problèmes. Si cela vous arrive, les messages d’erreur courants et les étapes de dépannage associées sont là pour vous aider à rétablir le service. De même, vous pouvez à tout moment [formuler une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

Cet article indique les étapes à suivre pour résoudre les problèmes courants dans Azure AD DS.

## <a name="you-cannot-enable-azure-ad-domain-services-for-your-azure-ad-directory"></a>Vous ne pouvez pas activer les services de domaine Azure AD pour votre annuaire Azure AD

Si vous avez des difficultés à activer Azure AD DS, examinez ci-dessous les erreurs courantes et les étapes à suivre pour les résoudre :

| **Exemple de message d’erreur** | **Résolution :** |
| --- |:--- |
| *Le nom aaddscontoso.com est déjà utilisé sur ce réseau. Spécifiez un nom qui n’est pas utilisé.* |[Conflit de nom de domaine dans le réseau virtuel](troubleshoot.md#domain-name-conflict) |
| *Les services de domaine n’ont pas pu être activés pour ce client Azure AD. Le service ne dispose pas des autorisations adéquates pour l’application appelée « Synchronisation des services de domaine Azure AD ». Supprimez l’application appelée « Synchronisation des services de domaine Azure AD » et réessayez d’activer les services de domaine pour votre client Azure AD.* |[L’application Domain Services ne dispose pas des autorisations adéquates pour l’application de synchronisation d’Azure AD Domain Services](troubleshoot.md#inadequate-permissions) |
| *Les services de domaine n’ont pas pu être activés pour ce client Azure AD. L’application de services de domaine dans votre client Azure AD n’a pas les autorisations requises pour activer les services de domaine. Supprimez l’application avec l’identificateur d’application d87dcbc6-a371-462e-88e3-28ad15ec4e64 et essayez ensuite d’activer les services de domaine pour votre client Azure AD.* |[L’application Domain Services n’est pas configurée correctement dans votre locataire Azure AD](troubleshoot.md#invalid-configuration) |
| *Les services de domaine n’ont pas pu être activés pour ce client Azure AD. L’application Microsoft Azure AD est désactivée dans votre client Azure AD. Activez l’application avec l’identificateur d’application 00000002-0000-0000-c000-000000000000 et essayez ensuite d’activer les services de domaine pour votre client Azure AD.* |[L’application Microsoft Graph est désactivée dans votre client Azure AD.](troubleshoot.md#microsoft-graph-disabled) |

### <a name="domain-name-conflict"></a>Conflit de nom de domaine

**Message d’erreur**

*Le nom aaddscontoso.com est déjà utilisé sur ce réseau. Spécifiez un nom qui n’est pas utilisé.*

**Résolution :**

Vérifiez que vous n’avez pas d’environnement AD DS existant avec le même nom de domaine sur le même réseau virtuel ou sur un réseau virtuel appairé. Par exemple, si vous disposez d’un domaine AD DS nommé *aaddscontoso.com* qui s’exécute sur des machines virtuelles Azure. Quand vous essayez d’activer un domaine managé Azure AD DS avec le même nom de domaine (*aaddscontoso.com*) sur le réseau virtuel, l’opération demandée échoue.

Cet échec est dû à des conflits de noms pour le nom de domaine sur le réseau virtuel. Une recherche DNS vérifie si un environnement AD DS existant répond au nom de domaine demandé. Pour résoudre cet échec, utilisez un nom différent pour configurer votre domaine managé, ou supprimez le domaine AD DS existant, puis réessayez d’activer Azure AD DS.

### <a name="inadequate-permissions"></a>Autorisations inappropriées

**Message d’erreur**

*Les services de domaine n’ont pas pu être activés pour ce client Azure AD. Le service ne dispose pas des autorisations adéquates pour l’application appelée « Synchronisation des services de domaine Azure AD ». Supprimez l’application appelée « Synchronisation des services de domaine Azure AD » et réessayez d’activer les services de domaine pour votre client Azure AD.*

**Résolution :**

Vérifiez s’il existe une application nommée *Azure AD Domain Services Sync* dans votre annuaire Azure AD. Si cette application existe, supprimez-la, puis réessayez d’activer Azure AD DS. Pour rechercher une application existante et la supprimer si nécessaire, effectuez les étapes suivantes :

1. Sur le portail Azure, sélectionnez **Azure Active Directory** dans le volet de navigation gauche.
1. Sélectionnez **Applications d’entreprise**. Choisissez *Toutes les application* dans le menu déroulant **Type d’application**, puis sélectionnez **Appliquer**.
1. Dans la zone de recherche, entrez *Azure AD Domain Services Sync*. Si l’application existe, sélectionnez-la et choisissez **Supprimer**.
1. Une fois que vous avez supprimé l’application, essayez à nouveau d’activer Azure AD DS.

### <a name="invalid-configuration"></a>Configuration non valide

**Message d’erreur**

*Les services de domaine n’ont pas pu être activés pour ce client Azure AD. L’application de services de domaine dans votre client Azure AD n’a pas les autorisations requises pour activer les services de domaine. Supprimez l’application avec l’identificateur d’application d87dcbc6-a371-462e-88e3-28ad15ec4e64 et essayez ensuite d’activer les services de domaine pour votre client Azure AD.*

**Résolution :**

Vérifiez si votre annuaire Azure AD contient une application nommée *AzureActiveDirectoryDomainControllerServices* avec l’identificateur d’application *d87dcbc6-a371-462e-88e3-28ad15ec4e64*. Si cette application existe, supprimez-la, puis essayez à nouveau d’activer Azure AD DS.

Utilisez le script PowerShell suivant pour rechercher une instance d’application existante et la supprimer si nécessaire :

```powershell
$InformationPreference = "Continue"
$WarningPreference = "Continue"

$aadDsSp = Get-AzureADServicePrincipal -Filter "AppId eq 'd87dcbc6-a371-462e-88e3-28ad15ec4e64'" -ErrorAction Ignore
if ($aadDsSp -ne $null)
{
    Write-Information "Found Azure AD Domain Services application. Deleting it ..."
    Remove-AzureADServicePrincipal -ObjectId $aadDsSp.ObjectId
    Write-Information "Deleted the Azure AD Domain Services application."
}

$identifierUri = "https://sync.aaddc.activedirectory.windowsazure.com"
$appFilter = "IdentifierUris eq '" + $identifierUri + "'"
$app = Get-AzureADApplication -Filter $appFilter
if ($app -ne $null)
{
    Write-Information "Found Azure AD Domain Services Sync application. Deleting it ..."
    Remove-AzureADApplication -ObjectId $app.ObjectId
    Write-Information "Deleted the Azure AD Domain Services Sync application."
}

$spFilter = "ServicePrincipalNames eq '" + $identifierUri + "'"
$sp = Get-AzureADServicePrincipal -Filter $spFilter
if ($sp -ne $null)
{
    Write-Information "Found Azure AD Domain Services Sync service principal. Deleting it ..."
    Remove-AzureADServicePrincipal -ObjectId $sp.ObjectId
    Write-Information "Deleted the Azure AD Domain Services Sync service principal."
}
```

### <a name="microsoft-graph-disabled"></a>Microsoft Graph désactivé

**Message d’erreur**

*Les services de domaine n’ont pas pu être activés pour ce client Azure AD. L’application Microsoft Azure AD est désactivée dans votre client Azure AD. Activez l’application avec l’identificateur d’application 00000002-0000-0000-c000-000000000000 et essayez ensuite d’activer les services de domaine pour votre client Azure AD.*

**Résolution :**

Vérifiez si vous avez désactivé une application associée à l’identificateur *00000002-0000-0000-c000-000000000000*. Cette application est l’application Microsoft Azure AD et fournit l’accès de l’API Graph à votre client Azure AD. Pour synchroniser votre locataire Azure AD, cette application doit être activée.

Pour vérifier l’état de cette application existante et l’activer si nécessaire, effectuez les étapes suivantes :

1. Sur le portail Azure, sélectionnez **Azure Active Directory** dans le volet de navigation gauche.
1. Sélectionnez **Applications d’entreprise**. Choisissez *Toutes les application* dans le menu déroulant **Type d’application**, puis sélectionnez **Appliquer**.
1. Dans la zone de recherche, entrez *00000002-0000-0000-c000-00000000000*. Sélectionnez l’application, puis choisissez **Propriétés**.
1. Si **Activé pour que les utilisateurs se connectent** est défini sur *Non*, définissez la valeur sur *Oui*, puis sélectionnez **Enregistrer**.
1. Une fois que vous avez activé l’application, essayez à nouveau d’activer Azure AD DS.

## <a name="users-are-unable-to-sign-in-to-the-azure-ad-domain-services-managed-domain"></a>Les utilisateurs sont incapables de se connecter aux services de domaine Asure AD gérés

Si des utilisateurs de votre locataire Azure AD ne peuvent pas se connecter au domaine managé, effectuez les étapes de dépannage suivantes :

* **Format d’informations d’identification** – Essayez d’utiliser le format UPN pour spécifier les informations d’identification, par exemple `dee@aaddscontoso.onmicrosoft.com`. Le format UPN est la méthode recommandée pour spécifier les informations d’identification dans Azure AD DS. Vérifiez que cet UPN est correctement configuré dans Azure AD.

    La valeur *SAMAccountName* de votre compte, par exemple *AADDSCONTOSO\driley*, peut être générée automatiquement s’il existe plusieurs utilisateurs avec le même préfixe UPN dans votre locataire ou si votre préfixe UPN est trop long. Par conséquent, le format *SAMAccountName* de votre compte peut être différent de ce que vous attendiez ou de celui que vous utilisez dans votre domaine local.

* **Synchronisation de mot de passe** – Veillez à activer la synchronisation de mot de passe pour les [utilisateurs cloud uniquement][cloud-only-passwords] ou pour les environnements hybrides [ utilisant Azure AD Connect][hybrid-phs].
    * **Comptes synchronisés hybrides :** si les comptes d’utilisateurs affectés sont synchronisés à partir d’un annuaire local, vérifiez les éléments suivants :
    
      * Vous avez déployé la [dernière version recommandée d’Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) ou procédé à une mise à jour vers cette version.
      * Vous avez configuré Azure AD Connect pour [effectuer une synchronisation complète][hybrid-phs].
      * Selon la taille de votre répertoire, la mise à disposition des comptes d’utilisateurs et des hachages d’informations d’identification dans le domaine managé peut prendre un certain temps. Veillez à attendre suffisamment longtemps avant d’essayer de vous authentifier pour le domaine managé.
      * Si le problème persiste après la vérification des étapes précédentes, essayez de redémarrer *Microsoft Azure Active Directory Sync Services*. À partir de votre serveur Azure AD Connect, ouvrez une invite de commandes et exécutez les commandes suivantes :
    
        ```console
        net stop 'Microsoft Azure AD Sync'
        net start 'Microsoft Azure AD Sync'
        ```

    * **Comptes cloud uniquement** : si le compte d’utilisateur en question est un compte d’utilisateur cloud uniquement, vérifiez que l’[utilisateur a changé son mot de passe après que vous avez activé Azure AD DS][cloud-only-passwords]. Cette réinitialisation de mot de passe entraîne la génération des hachages d’informations d’identification nécessaires à la génération du domaine managé.

* **Vérifiez que le compte d’utilisateur est actif**: par défaut, cinq tentatives de saisie de mot de passe non valide en 2 minutes dans le domaine managé entraînent le verrouillage d’un compte d’utilisateur pendant 30 minutes. L’utilisateur ne peut pas se connecter tant que le compte est verrouillé. Après ces 30 minutes, le compte d’utilisateur est automatiquement déverrouillé.
  * Les tentatives de saisie de mot de passe non valide dans le domaine managé ne verrouillent pas le compte d’utilisateur dans Azure AD. Le compte d’utilisateur est verrouillé uniquement dans votre domaine managé. Vérifiez l’état du compte d’utilisateur dans la *console d’administration Active Directory (ADAC)* en utilisant la [machine virtuelle de gestion][management-vm], et non dans Azure AD.
  * Vous pouvez aussi [configurer des stratégies de mot de passe affinées][password-policy] pour modifier le seuil et la durée de verrouillage par défaut.

* **Comptes externes** – Vérifiez que le compte d’utilisateur en question n’est pas un compte externe dans le locataire Azure AD. Les comptes Microsoft comme `dee@live.com` ou les comptes d’utilisateurs d’un annuaire Azure AD externe sont des exemples de comptes externes. Azure AD DS ne stocke pas les informations d’identification pour les comptes d’utilisateurs externes et ne peut donc pas se connecter au domaine managé.

## <a name="there-are-one-or-more-alerts-on-your-managed-domain"></a>Il existe une ou plusieurs alertes sur votre domaine géré

La présence d’alertes actives dans le domaine managé peut empêcher le bon fonctionnement du processus d’authentification.

Pour savoir s’il existe des alertes actives, [vérifiez l’état d’intégrité d’un domaine managé][check-health]. Si des alertes sont présentes, [résolvez les problèmes associés][troubleshoot-alerts].

## <a name="users-removed-from-your-azure-ad-tenant-are-not-removed-from-your-managed-domain"></a>Les utilisateurs supprimés de votre client Azure AD ne sont pas supprimés de votre domaine géré

Azure AD protège contre la suppression accidentelle d’objets utilisateur. Quand vous supprimez un compte d’utilisateur d’un locataire Azure AD, l’objet utilisateur correspondant est déplacé dans la corbeille. Quand cette opération de suppression est synchronisée avec votre domaine managé, le compte d’utilisateur correspondant est marqué comme étant désactivé. Cette fonctionnalité vous permet de récupérer le compte d’utilisateur, c’est-à-dire d’annuler sa suppression.

Le compte d’utilisateur reste dans un état désactivé dans le domaine managé, même si vous recréez un compte d’utilisateur avec le même UPN dans le répertoire Azure AD. Pour supprimer le compte d’utilisateur du domaine managé, vous devez forcer sa suppression dans le locataire Azure AD.

Pour supprimer entièrement un compte d’utilisateur d’un domaine managé, supprimez définitivement l’utilisateur de votre locataire Azure AD à l’aide du cmdlet PowerShell [Remove-MsolUser][Remove-MsolUser] avec le paramètre `-RemoveFromRecycleBin`.

## <a name="next-steps"></a>Étapes suivantes

Si vous continuez à rencontrer des problèmes, [ouvrez une requête de support Azure][azure-support] pour obtenir un résolution des problèmes supplémentaire.

<!-- INTERNAL LINKS -->
[cloud-only-passwords]: tutorial-create-instance.md#enable-user-accounts-for-azure-ad-ds
[hybrid-phs]: tutorial-configure-password-hash-sync.md
[management-vm]: tutorial-create-management-vm.md
[password-policy]: password-policy.md
[check-health]: check-health.md
[troubleshoot-alerts]: troubleshoot-alerts.md
[Remove-MsolUser]: /powershell/module/MSOnline/Remove-MsolUser
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
