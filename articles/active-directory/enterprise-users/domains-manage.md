---
title: Ajouter et vérifier des noms de domaines personnalisés - Azure Active Directory | Microsoft Docs
description: Concepts de gestion et procédures pour gérer un nom de domaine dans Azure Active Directory
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: how-to
ms.date: 12/20/2020
ms.author: curtand
ms.reviewer: sumitp
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7b8867a137e38e7400da877f4c239e517cbebe05
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96859978"
---
# <a name="managing-custom-domain-names-in-your-azure-active-directory"></a>Gestion des noms de domaine personnalisés dans Azure Active Directory

Un nom de domaine est une partie importante de l’identificateur de nombreuses ressources Azure Active Directory (Azure AD) : il fait partie du nom ou de l’adresse e-mail d’un utilisateur, de l’adresse d’un groupe et parfois de l’URI d’ID d’une application. Une ressource dans Azure AD peut inclure un nom de domaine appartenant à l’organisation qui contient la ressource. Seul un administrateur général peut gérer des domaines dans Azure AD.

## <a name="set-the-primary-domain-name-for-your-azure-ad-organization"></a>Définir le nom de domaine principal pour votre organisation Azure AD

Lors de la création de l’organisation, le nom de domaine initial, par exemple « contoso.onmicrosoft.com », est également le nom de domaine principal. Le domaine principal est le nom de domaine par défaut pour un nouvel utilisateur lorsque vous créez un nouvel utilisateur. Le fait de définir un nom de domaine principal simplifie le processus permettant à un administrateur de créer des utilisateurs dans le portail. Pour modifier le nom de domaine principal, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en utilisant un compte d’administrateur général pour l’organisation.
2. Sélectionnez **Azure Active Directory**.
3. Sélectionnez **Noms de domaine personnalisés**.
  
   ![Ouvrir la page de gestion des utilisateurs](./media/domains-manage/add-custom-domain.png)
4. Sélectionnez le nom du domaine que vous souhaitez choisir comme domaine principal.
5. Sélectionnez la commande **Définir comme principal**. Confirmez votre choix lorsque vous y êtes invité.
  
   ![Créer un nom de domaine principal](./media/domains-manage/make-primary-domain.png)

Vous pouvez modifier le nom de domaine principal de votre organisation par n’importe quel domaine personnalisé vérifié qui n’est pas fédéré. La modification du domaine principal de votre organisation ne change pas le nom d’utilisateur des utilisateurs existants.

## <a name="add-custom-domain-names-to-your-azure-ad-organization"></a>Ajouter des noms de domaine personnalisé à votre organisation Azure AD

Vous pouvez ajouter jusqu’à 900 noms de domaine managé. Si vous configurez tous vos domaines pour la fédération avec Windows Server Active Directory, vous pouvez ajouter jusqu’à 450 noms de domaine dans chaque organisation.

## <a name="add-subdomains-of-a-custom-domain"></a>Ajouter des sous-domaines d’un domaine personnalisé

Si vous souhaitez ajouter un nom de sous-domaine, tel que « europe.contoso.com » à votre organisation, vous devez tout d’abord ajouter et vérifier le domaine racine, tel que « contoso.com ». Le sous-domaine est automatiquement vérifié par Azure AD. Pour voir que le sous-domaine ajouté est vérifié, actualisez la liste des domaines dans le navigateur.

Si vous avez déjà ajouté un domaine contoso.com à une organisation Azure AD, vous pouvez également vérifier le sous-domaine europe.contoso.com dans une autre organisation Azure AD. Lorsque vous ajoutez le sous-domaine, vous êtes invité à ajouter un enregistrement TXT au fournisseur d’hébergement DNS.



## <a name="what-to-do-if-you-change-the-dns-registrar-for-your-custom-domain-name"></a>Que faire en cas de modification du bureau d’enregistrement DNS pour votre nom de domaine personnalisé ?

Si vous modifiez les bureaux d’enregistrement DNS, aucune tâche de configuration supplémentaire n’est à effectuer dans Azure AD. Vous pouvez continuer à utiliser le nom de domaine avec Azure AD sans interruption. Si vous utilisez votre nom de domaine personnalisé avec Microsoft 365, Intune ou d’autres services s’appuyant sur des noms de domaine personnalisés dans Azure AD, consultez la documentation dédiée à ces services.

## <a name="delete-a-custom-domain-name"></a>Supprimer un nom de domaine personnalisé

Vous pouvez supprimer un nom de domaine personnalisé de votre répertoire Azure AD si votre organisation n’utilise plus ce nom de domaine ou si vous souhaitez l’utiliser avec un autre répertoire Azure AD.

Pour supprimer un nom de domaine personnalisé, vous devez d’abord vous assurer qu’aucune des ressources de votre organisation ne s’appuie sur le nom de domaine. Vous ne pouvez pas supprimer un nom de domaine de votre organisation si :

* L’utilisateur dispose d’un nom d’utilisateur, d’une adresse de messagerie ou d’une adresse de proxy qui incluent le nom de domaine.
* Le groupe dispose d’une adresse de messagerie ou d’une adresse de proxy qui incluent le nom de domaine.
* Une application dans Azure AD dispose d’une URI ID d’application qui inclut le nom de domaine.

Vous devez modifier ou supprimer n’importe quelle ressource de ce type dans votre organisation Azure AD pour pouvoir supprimer le nom de domaine personnalisé.

### <a name="forcedelete-option"></a>Option ForceDelete

Vous pouvez appliquer l’option **ForceDelete** à un nom de domaine dans le [Centre d’administration Azure AD](https://aad.portal.azure.com) ou à l’aide de [l’API Microsoft Graph](/graph/api/domain-forcedelete?view=graph-rest-beta&preserve-view=true). Ces options utilisent une opération asynchrone et mettent à jour toutes les références depuis le nom de domaine personnalisé comme « user@contoso.com » vers le nom de domaine par défaut initial comme « user@contoso.onmicrosoft.com ».

Pour appeler **ForceDelete** dans le Portail Azure, vous devez vous assurer qu’il y a moins de 1 000 références au nom de domaine, et toutes les références dans lesquelles Exchange est le service d’approvisionnement doivent être mises à jour ou supprimées dans le [ Centre d’administration Exchange](https://outlook.office365.com/ecp/). Cela inclut les listes distribuées et les groupes de sécurité à extension messagerie ; pour plus d’informations, consultez [Supprimer les groupes de sécurité à extension messagerie](/Exchange/recipients/mail-enabled-security-groups#Remove%20mail-enabled%20security%20groups&preserve-view=true). En outre, l’opération **ForceDelete** échoue si l’un des énoncés suivants est vrai :

* Vous avez acheté un domaine par le biais des services d’abonnement de domaine Microsoft 365
* Vous êtes un partenaire qui administre le compte d’une autre organisation cliente

Les actions suivantes sont effectuées dans le cadre de l’opération **ForceDelete** :

* Renommage des éléments UPN, EmailAddress et ProxyAddress des utilisateurs avec des références au nom de domaine personnalisé vers le nom de domaine par défaut initial.
* Renommage de l’élément EmailAddress des groupes avec des références au nom de domaine personnalisé vers le nom de domaine par défaut initial.
* Renommage des éléments identifierUris des applications avec des références au nom de domaine personnalisé vers le nom de domaine par défaut initial.

Une erreur est renvoyée quand :

* Le nombre d’objets à renommer est supérieur à 1 000
* L’une des applications à renommer est une application mutualisée

### <a name="frequently-asked-questions"></a>Forum aux questions

**Q : Pourquoi la suppression du domaine échoue-t-elle avec une erreur indiquant que je dispose de groupes Exchange contrôlés sur ce nom de domaine ?** <br>
**R :** Aujourd'hui, certains groupes tels que les groupes de sécurité à extension messagerie et les listes distribuées sont approvisionnés par Exchange et doivent être nettoyés manuellement dans le [Centre d'administration Exchange](https://outlook.office365.com/ecp/). Des éléments ProxyAddresses, qui reposent sur le nom de domaine personnalisé, peuvent être en attente et devront être mis à jour manuellement vers un autre nom de domaine. 

**Q : Je suis connecté en tant qu'administrateur\@contoso.com, mais je ne peux pas supprimer le nom de domaine « contoso.com ».**<br>
**R :** Vous ne pouvez pas référencer le nom de domaine personnalisé que vous cherchez à supprimer dans votre nom de compte d'utilisateur. Assurez-vous que le compte d’administrateur général utilise le nom de domaine par défaut initial (.onmicrosoft.com) comme admin@contoso.onmicrosoft.com. Connectez-vous avec un compte d’administrateur général différent comme admin@contoso.onmicrosoft.com ou un autre nom de domaine personnalisé comme « fabrikam.com » où le compte est admin@fabrikam.com.

**Q : J'ai cliqué sur le bouton Supprimer un domaine et je vois l'état `In Progress` pour l'opération de suppression. Combien de temps cela prend-il ? Que se passe-t-il en cas d’échec ?**<br>
**R :** L'opération de suppression de domaine est une tâche d'arrière-plan asynchrone qui renomme toutes les références au nom de domaine. Cette opération dure normalement une à deux minutes. En cas d’échec de suppression du domaine, vérifiez que vous n’avez pas :

* Des applications configurées sur le nom de domaine avec l’appIdentifierURI
* Des groupes à extension de messagerie référençant le nom de domaine personnalisé
* Plus de 1 000 références au nom de domaine

Si vous remarquez que l’une des conditions n’a pas été respectée, nettoyez manuellement les références et essayez une nouvelle fois de supprimer le domaine.

## <a name="use-powershell-or-the-microsoft-graph-api-to-manage-domain-names"></a>Utiliser PowerShell ou l’API Microsoft Graph pour gérer les noms de domaine

La plupart des tâches de gestion des noms de domaine dans Azure Active Directory peuvent également être accomplies à l’aide de Microsoft PowerShell ou par programmation à l’aide de l’API Microsoft Graph.

* [Utilisation de PowerShell pour gérer les noms de domaine dans Azure AD](/powershell/module/azuread/#domains&preserve-view=true)
* [Type de ressource de domaine](/graph/api/resources/domain?view=graph-rest-1.0&preserve-view=true)

## <a name="next-steps"></a>Étapes suivantes

* [Ajouter des noms de domaines personnalisés](../fundamentals/add-custom-domain.md?context=azure%2factive-directory%2fusers-groups-roles%2fcontext%2fugr-context)
* [Supprimer les groupes de sécurité à extension messagerie](/Exchange/recipients/mail-enabled-security-groups#Remove%20mail-enabled%20security%20groups&preserve-view=true)
* [ForceDelete a custom domain name with Microsoft Graph API](/graph/api/domain-forcedelete?view=graph-rest-beta&preserve-view=true) (Appliquer une opération ForceDelete sur un nom de domaine personnalisé avec l’API Microsoft Graph)
