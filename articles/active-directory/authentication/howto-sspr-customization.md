---
title: Personnaliser la réinitialisation de mot de passe en libre-service - Azure Active Directory
description: Découvrez comment personnaliser les options d’affichage et d’expérience utilisateur pour la réinitialisation de mot de passe en libre-service Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/17/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3766db0d5070f15216ac612353e2c25737ad092a
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741624"
---
# <a name="customize-the-user-experience-for-azure-active-directory-self-service-password-reset"></a>Personnalisation de l’expérience utilisateur pour la réinitialisation de mot de passe en libre-service Azure Active Directory

La réinitialisation de mot de passe en libre-service (SSPR) permet aux utilisateurs d’Azure Active Directory (Azure AD) de modifier ou de réinitialiser leur mot de passe sans intervention d’un administrateur ou du support technique. Si le compte d’un utilisateur est verrouillé ou si ce dernier oublie son mot de passe, il peut suivre des invites afin de se débloquer et de reprendre son travail. Cette fonctionnalité réduit les appels au support technique et la perte de productivité quand l’utilisateur ne parvient pas à se connecter à son appareil ou à une application.

Pour améliorer l’expérience SSPR des utilisateurs, vous pouvez personnaliser l’apparence de la page de réinitialisation de mot de passe, des notifications par e-mail ou des pages de connexion. Ces options de personnalisation permettent d’indiquer clairement à l’utilisateur qu’il se trouve au bon endroit et de lui donner la certitude qu’il accède bien aux ressources de l’entreprise.
    
Cet article montre comment personnaliser le lien d’e-mail SSPR pour les utilisateurs, l’image de l’entreprise et le lien de la page de connexion AD FS.

## <a name="customize-the-contact-your-administrator-link"></a>Personnaliser le lien « Contactez votre administrateur »

Pour aider les utilisateurs à demander de l’aide concernant la réinitialisation de mot de passe en libre-service, un lien « Contactez votre administrateur » est proposé sur le portail de réinitialisation de mot de passe. Il a deux effets possibles :

* Si l’état par défaut de ce lien de contact a été conservé, un e-mail est envoyé à vos administrateurs pour leur demander de l’aide concernant la modification du mot de passe de l’utilisateur. Voici cet e-mail par défaut :

    ![Exemple d’e-mail de demande de réinitialisation envoyé à l’administrateur](./media/howto-sspr-customization/sspr-contact-admin.png)

* En cas de personnalisation, il renvoie l’utilisateur vers une page web ou une adresse e-mail spécifiée par l’administrateur afin d’obtenir de l’aide.
    * Si vous le personnalisez, nous vous recommandons d’opter pour un support que les utilisateurs connaissent.

    > [!WARNING]
    > Si vous personnalisez ce paramètre avec une adresse e-mail et un compte nécessitant une réinitialisation du mot de passe, l’utilisateur peut ne pas être en mesure de demander de l’aide.

### <a name="default-email-behavior"></a>Comportement d’e-mail par défaut

L’e-mail de contact par défaut est envoyé aux destinataires dans l’ordre suivant :

1. Si les rôles *Administrateur du support technique* ou *Administrateur de mot de passe* est affecté, les administrateurs disposant de ces rôles sont avertis.
1. Si aucun administrateur du support technique ou administrateur de mots de passe n’est affecté, ce sont les administrateurs disposant du rôle *Administrateur d’utilisateurs* qui sont informés.
1. Si aucun des rôles précédents n’a été affecté, les *administrateurs globaux* sont avertis.

Dans tous les cas, un maximum de 100 destinataires sont informés.

Pour en savoir plus sur les différents rôles d’administrateur et sur la façon de les affecter, consultez [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).

### <a name="disable-contact-your-administrator-emails"></a>Désactiver les e-mails « Contactez votre administrateur »

Si votre organisation ne souhaite pas notifier les administrateurs au sujet des demandes de réinitialisation de mot de passe, les options de configuration suivantes peuvent être utilisées :

* Personnalisez le lien du support technique pour fournir une URL web ou un lien mailto: que les utilisateurs peuvent utiliser pour obtenir une assistance. Cette option se trouve sous **Réinitialisation de mot de passe** > **Personnalisation** > **Adresse e-mail ou URL du support technique**.
* Activez la réinitialisation de mot de passe en libre-service pour tous les utilisateurs. Cette option se trouve sous **Réinitialisation de mot de passe** > **Propriétés**. Si vous ne voulez pas que les utilisateurs réinitialisent leurs propres mots de passe, vous pouvez étendre l’accès à un groupe vide. *Nous ne recommandons pas cette option.*

## <a name="customize-the-sign-in-page-and-access-panel"></a>Personnalisation de la page de connexion et du volet d’accès

Vous pouvez personnaliser la page de connexion, par exemple ajouter un logo à côté de l’image correspondant à la marque de votre société. Pour plus d’informations sur la configuration de la personnalisation d’entreprise, consultez [Ajout d’une personnalisation d’entreprise à la page de connexion dans Azure AD](../fundamentals/customize-branding.md).

Les graphiques que vous choisissez s’affichent dans les circonstances suivantes :

* Quand un utilisateur entre son nom d’utilisateur
* Si l’utilisateur accède à l’URL personnalisée :
   * En passant le paramètre `whr` à la page de réinitialisation du mot de passe, par exemple `https://login.microsoftonline.com/?whr=contoso.com`
   * En passant le paramètre `username` à la page de réinitialisation du mot de passe, par exemple `https://login.microsoftonline.com/?username=admin@contoso.com`

### <a name="directory-name"></a>Nom de l’annuaire

Si vous souhaitez rendre l’apparence plus conviviale, vous pouvez modifier le nom de l’organisation sur le portail et dans les communications automatisées. Pour changer l’attribut de nom de répertoire sur le Portail Azure, accédez à **Azure Active Directory** > **Propriétés**. Cette option de nom convivial de l’organisation est particulièrement visible dans les e-mails automatisés, comme dans les exemples suivants :

* Le nom convivial dans l’e-mail, par exemple « *Microsoft pour le compte de la démonstration CONTOSO* »
* La ligne d’objet dans l’e-mail, par exemple « *Code de vérification d’e-mail du compte de démonstration CONTOSO* »

## <a name="customize-the-ad-fs-sign-in-page"></a>Personnalisation de la page de connexion AD FS

Si vous utilisez les services de fédération Active Directory (AD FS) pour les événements de connexion utilisateur, vous pouvez ajouter un lien à la page de connexion en suivant l’aide donnée dans l’article [Ajout de la description de la page de connexion](/windows-server/identity/ad-fs/operations/add-sign-in-page-description).

Fournissez aux utilisateurs un lien vers la page leur permettant d’accéder au flux de travail SSPR, par exemple *https://passwordreset.microsoftonline.com* . Pour ajouter un lien vers la page de connexion AD FS, exécutez la commande suivante sur votre serveur AD FS :

``` powershell
Set-ADFSGlobalWebContent -SigninPageDescriptionText "<p><a href='https://passwordreset.microsoftonline.com' target='_blank'>Can't access your account?</a></p>"
```

## <a name="next-steps"></a>Étapes suivantes

Pour comprendre l’utilisation de la réinitialisation SSPR dans votre environnement, consultez [Options de reporting pour la gestion des mots de passe Azure AD](howto-sspr-reporting.md).

Si vos utilisateurs ou vous-même rencontrez des problèmes de réinitialisation SSPR, consultez [Résolution des problèmes de réinitialisation de mot de passe en libre-service](./troubleshoot-sspr.md).