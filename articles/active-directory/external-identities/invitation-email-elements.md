---
title: Éléments de l’e-mail d’invitation de B2B - Azure Active Directory | Microsoft Docs
description: Modèle d’e-mail d’invitation de collaboration d’Azure Active Directory B2B
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 10/20/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: bb134a2fb784e02f5e00c9e88ab0df1794489e0c
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860590"
---
# <a name="the-elements-of-the-b2b-collaboration-invitation-email---azure-active-directory"></a>Éléments de l’e-mail d’invitation de collaboration B2B - Azure Active Directory

Les e-mails d’invitation sont un composant essentiel pour intégrer des partenaires comme utilisateurs de collaboration B2B dans Azure AD. Bien qu’il ne soit [pas obligatoire d’envoyer un e-mail pour inviter quelqu’un avec B2B Collaboration](add-user-without-invite.md), cela permet de donner à l’utilisateur toutes les informations dont il a besoin pour décider d’accepter ou non l’invitation. L’e-mail leur fournit également un lien qui pourra lui être utile lorsqu’il devra revenir consulter vos ressources.

![Capture d’écran montrant l’e-mail d’invitation B2B](media/invitation-email-elements/invitation-email.png)

## <a name="explaining-the-email"></a>Explication de l’e-mail

Examinons quelques-uns des éléments de l’e-mail pour savoir comment utiliser au mieux ces fonctionnalités.

### <a name="subject"></a>Objet

L’objet de l’e-mail suit ce modèle :

&lt;nom_utilisateur&gt; vous invite à accéder aux applications de son organisation.

### <a name="from-address"></a>Adresse de l’expéditeur

Nous utilisons un modèle similaire à LinkedIn pour l’adresse De. Ce modèle doit préciser que, même si l’e-mail est envoyé par invites@microsoft.com, l’invitation provient d’une autre organisation. Le format est le suivant : Microsoft Invitations <invites@microsoft.com> ou Invitations Microsoft pour le compte de &lt;nom_locataire&gt; <invites@microsoft.com>. 

### <a name="reply-to"></a>Adresse de réponse

L’adresse e-mail de réponse est définie sur l’adresse e-mail de l’inviteur quand elle est disponible : ainsi, répondre à l’e-mail envoie un e-mail en retour à l’inviteur.

### <a name="phishing-warning"></a>Avertissement d’hameçonnage

L’e-mail commence par un bref avertissement sur l’hameçonnage, notifiant à l’utilisateur qu’il ne doit accepter que les invitations attendues. Il est recommandé de s’assurer que les partenaires que vous invitez ne seront pas surpris par votre invitation en la mentionnant à l’avance.

![Image de l’avertissement d’hameçonnage dans l’e-mail](media/invitation-email-elements/phishing-warning.png)

### <a name="inviters-information-and-invitation-message"></a>Message d’informations et d’invitation de l’inviteur

L’e-mail contient le nom et le domaine principal associés à l’organisation qui envoie l’invitation. Toutes ces informations doivent aider l’invité à prendre une décision informée concernant l’invitation. Si l’inviteur ajoute un message à l’invitation lorsqu’il [invite un utilisateur à l’annuaire, au groupe ou à l’application](add-users-administrator.md) ou lorsqu’il [utilise l’API d’invitation](customize-invitation-api.md), le message est mis en surbrillance dans la section principale de l’e-mail. Le nom et l’image de profil de l’inviteur sont également indiqué s’il les a définis. Le message lui-même est une zone de texte. Il ne traite donc pas les balises HTML pour des raisons de sécurité.

![Image du message d’invitation dans l’e-mail](media/invitation-email-elements/invitation-message-inviters-info.png)

### <a name="accept-button-and-redirect-url"></a>Bouton Accepter et URL de redirection

La section suivante de l’e-mail contient des informations sur l’endroit où l’invité sera redirigé une fois qu’il aura accepté l’invitation, ainsi que le bouton correspondant.  Par la suite, l’invité pourra toujours utiliser ce lien pour revenir directement à vos ressources.

![Image du bouton Accepter et de l’URL de redirection dans l’e-mail](media/invitation-email-elements/accept-button.png)

### <a name="footer-section"></a>Section Pied de page

Le pied de page contient d’autres informations sur l’invitation envoyée. Il comporte toujours une option permettant à l’invité de bloquer les futures invitations. Si l’organisation a [défini une déclaration de confidentialité](../fundamentals/active-directory-properties-area.md), le lien associé est affiché ici.  Sinon, une note indique que l’organisation n’a pas défini de déclaration de confidentialité.

![Image de la section de pied de page dans l’e-mail](media/invitation-email-elements/footer-section.png)

### <a name="blocking-an-organization-unsubscribing"></a>Blocage d’une organisation (annulation d’abonnement)

Dans l’invitation d’une organisation, le pied de page contient une option permettant de **bloquer les futures invitations**. Un utilisateur invité peut sélectionner ce lien pour bloquer les futures invitations de l’organisation. Cette action ajoute également l’organisation à la liste des utilisateurs désabonnés à [https://invitations.microsoft.com/unsubscribe/manage](https://invitations.microsoft.com/unsubscribe/manage).

### <a name="viewing-organizations-youve-blocked"></a>Affichage des organisations que vous avez bloquées

Un utilisateur invité peut suivre les étapes suivantes pour afficher ou exporter les organisations qu’il a bloquées :

1. Accédez à [https://invitations.microsoft.com/unsubscribe/manage](https://invitations.microsoft.com/unsubscribe/manage).
2. Entrez votre adresse e-mail et suivez les étapes de connexion pour l’authentification par code secret ponctuelle d’e-mail.
3. Affichez les organisations que vous avez bloquées ou exportez les noms à l’aide d’une opérations de copier-coller.
   > [!NOTE]
   > Si vous souhaitez autoriser une organisation que vous avez bloquée à vous inviter à nouveau, vous pouvez choisir l’organisation, puis sélectionner **Suivant**.

## <a name="how-the-language-is-determined"></a>Comment la langue est-elle déterminée ?

Les paramètres suivants déterminent la langue présentée à l’utilisateur invité dans l’e-mail d’invitation. Ces paramètres sont répertoriés par ordre de priorité. Si un paramètre n’est pas configuré, le paramètre suivant de la liste détermine la langue.

- Propriété **messageLanguage** de l'objet [invitedUserMessageInfo](/graph/api/resources/invitedusermessageinfo) si l'API Créer une invitation est utilisée
-   Propriété **preferredLanguage** spécifiée dans l'[objet utilisateur](/graph/api/resources/user) de l'invité
-   **Langue de notification** définie dans les propriétés du client de base de l'utilisateur invité (pour les clients Azure AD uniquement)
-   **Langue de Notification** définie dans les propriétés du client de la ressource

Si aucun de ces paramètres n'est configuré, l'anglais (États-Unis) est la langue par défaut.

## <a name="next-steps"></a>Étapes suivantes

Consultez les articles suivants sur Azure AD B2B Collaboration :

- [Qu’est-ce qu’Azure AD B2B Collaboration ?](what-is-b2b.md)
- [Comment les administrateurs Azure Active Directory ajoutent-ils des utilisateurs B2B Collaboration ?](add-users-administrator.md)
- [Comment les professionnels de l’information ajoutent-ils des utilisateurs B2B Collaboration ?](add-users-information-worker.md)
- [Utilisation d’une invitation B2B Collaboration](redemption-experience.md)
- [Ajouter des utilisateurs B2B Collaboration sans invitation](add-user-without-invite.md)
