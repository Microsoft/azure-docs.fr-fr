---
title: Réviser l'accès aux groupes et aux applications dans les révisions d'accès - Azure AD
description: Découvrez comment réviser l’accès des membres de groupes ou des applications dans les révisions d’accès Azure Active Directory.
services: active-directory
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/22/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: b5cd7a5737b2d13e63eabbbddd076cfc7aff83ac
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746794"
---
# <a name="review-access-to-groups-and-applications-in-azure-ad-access-reviews"></a>Réviser l'accès aux groupes et aux applications dans les révisions d'accès Azure AD

Azure Active Directory (Azure AD) simplifie la manière dont les entreprises gèrent l’accès aux applications et aux groupes dans Azure AD, ainsi que d’autres services en ligne Microsoft, avec une fonctionnalité appelée révision d’accès Azure AD. Cet article décrit comment un réviseur désigné effectue une révision d’accès pour les membres d’un groupe ou des utilisateurs ayant accès à une application. Si vous souhaitez réviser l’accès à un package, lisez [Révision de l’accès d’un package d’accès dans la gestion des droits d’utilisation Azure AD](entitlement-management-access-reviews-review-access.md).

## <a name="perform-access-review-using-my-apps"></a>Effectuer une révision d’accès à l’aide de Mes applications

Vous pouvez démarrer le processus de révision d’accès à partir de l’e-mail de notification ou en accédant directement au site.

- **E-mail** :

>[!IMPORTANT]
> Il peut y avoir des retards dans la réception de l’e-mail et, dans certains cas, cela peut prendre jusqu’à 24 heures. Ajoutez azure-noreply@microsoft.com à votre liste des destinataires approuvés pour vous assurer de recevoir tous les e-mails.

1. Recherchez un e-mail de Microsoft vous invitant à réviser les accès. Voici un exemple d’e-mail vous invitant à réviser l’accès d’un groupe.

    ![Capture d’écran représentant un exemple d’e-mail de Microsoft pour la révision d’accès à un groupe](./media/perform-access-review/access-review-email.png)

1. Cliquez sur le lien **Commencer une révision d'accès** pour ouvrir la révision d’accès.

- **Si vous n’avez pas reçu l’e-mail**, vous trouverez les révisions d’accès en attente en procédant comme suit.

    1. Connectez-vous au portail Mes applications à l’adresse [https://myapps.microsoft.com](https://myapps.microsoft.com).

        ![Portail Mes applications répertoriant les applications auxquelles vous êtes autorisé à accéder](./media/perform-access-review/myapps-access-panel.png)

    1. Dans l’angle supérieur droit de la page, cliquez sur l’utilisateur en regard de votre nom et de votre organisation par défaut. Si plusieurs organisations sont listées, sélectionnez l’organisation qui a demandé une révision d’accès.

    1. Cliquez sur la vignette **Révisions d’accès** pour afficher la liste des révisions d’accès en attente.

        > [!NOTE]
        > Si la vignette **Révisions d’accès** n’est pas visible, cela signifie qu’il n’y a pas de révisions d’accès à effectuer pour cette organisation et qu’aucune action n’est nécessaire pour l’instant.

        ![Capture d’écran représentant la liste de révisions d’accès en attente pour les applications et les groupes.](./media/perform-access-review/access-reviews-list.png)

    1. Cliquez sur le lien **Commencer la révision** associé à la révision d’accès que vous souhaitez effectuer.

Une fois que vous avez ouvert la révision d’accès, vous pouvez voir les noms des utilisateurs dont l’accès doit être révisé.

Si la requête vise à réviser votre propre accès, la page aura un aspect différent. Pour plus d’informations, consultez [Réviser son accès à des groupes ou à des applications](review-your-access.md).

![Ouvrir la liste des utilisateurs dont l’accès doit être révisé](./media/perform-access-review/perform-access-review.png)

Il y a deux manières d’approuver ou de refuser l’accès  :

- Vous pouvez approuver ou refuser l’accès d’un ou plusieurs utilisateurs manuellement en choisissant l’action appropriée pour chaque demande d’utilisateur.
- Vous pouvez accepter les recommandations du système.

### <a name="approve-or-deny-access-for-one-or-more-users"></a>Approuver ou refuser l’accès pour un ou plusieurs utilisateurs

1. Passez en revue la liste des utilisateurs pour décider s’il faut approuver ou refuser leur accès permanent.

    - Pour approuver ou refuser l’accès pour un seul utilisateur, cliquez sur la ligne pour ouvrir une fenêtre et spécifier l’action à entreprendre. 
    - Pour approuver ou refuser l’accès pour plusieurs utilisateurs, ajoutez des coches en regard des utilisateurs, puis sur le bouton **Réviser X utilisateur(s)** pour ouvrir une fenêtre et spécifier l’action à entreprendre.

1. Cliquez sur **Approuver** ou **Refuser**. 

    ![Fenêtre d’action qui inclut les options Approuver, Refuser et Je ne sais pas](./media/perform-access-review/approve-deny.png)
    >[!NOTE]
    > Si vous n’êtes pas sûr, vous pouvez cliquer sur **Je ne sais pas**. L’utilisateur peut alors conserver son accès et votre choix est enregistré dans les journaux d’audit.

1. L’administrateur de la révision d’accès peut exiger que vous motiviez votre décision dans la boîte de dialogue **Motif**. Même si aucun motif n’est requis. Vous pouvez toujours fournir un motif justifiant votre décision. Les informations que vous incluez seront disponibles pour d’autres réviseurs.

1. Une fois que vous avez spécifié l’action à entreprendre, cliquez sur **Enregistrer**.

    >[!NOTE]
    > Vous pouvez modifier votre réponse à tout moment avant que la révision d’accès soit terminée. Si vous souhaitez modifier votre réponse, sélectionnez la ligne et mettez à jour la réponse. Par exemple, vous pouvez approuver un utilisateur précédemment refusé ou refuser un utilisateur précédemment accepté.

    >[!IMPORTANT]
    > - Si un utilisateur se voit refuser l’accès, il n’est pas supprimé immédiatement. Il est supprimé une fois la période de révision terminée ou quand un administrateur interrompt la révision si l’option [Appliquer automatiquement](complete-access-review.md#apply-the-changes) est activée.
    > - S’il existe plusieurs réviseurs, la dernière réponse envoyée est enregistrée. Prenons un exemple où un administrateur désigne deux réviseurs : Alice et Bob. Alice ouvre la révision d’accès en premier et approuve la demande d’accès d’un utilisateur. Avant la fin de la période de révision, Bob ouvre la révision d’accès et refuse l’accès sur la demande précédemment approuvée par Alice. La dernière décision qui refuse l’accès est la réponse enregistrée.

### <a name="approve-or-deny-access-based-on-recommendations"></a>Approuver ou refuser l’accès selon les recommandations

Pour rendre les révisions d’accès plus faciles et plus rapides pour vous, nous fournissons également des recommandations que vous pouvez accepter en un seul clic. Les recommandations sont générées en fonction de l’activité de connexion de l’utilisateur.

1. Dans la barre bleue en bas de la page, cliquez sur **Accepter les recommandations**.

    ![Capture d’écran représentant la liste de révision d’accès ouverte dans laquelle le bouton « Accepter les recommandations » est sélectionné.](./media/perform-access-review/accept-recommendations.png)

    Vous obtenez un résumé des actions recommandées.

    ![Fenêtre qui affiche un résumé des actions recommandées](./media/perform-access-review/accept-recommendations-summary.png)

1. Cliquez sur **OK** pour accepter les recommandations.

## <a name="perform-access-review-using-my-access-new"></a>Effectuer une révision d’accès en utilisant Mon accès (nouveau)

Vous pouvez accéder à la nouvelle expérience de réviseur avec l’interface utilisateur mise à jour dans Mon accès de deux façons :

### <a name="my-apps-portal"></a>Portail Mes applications

1. Connectez-vous à Mes applications à l’adresse [https://myapps.microsoft.com](https://myapps.microsoft.com).

    ![Portail Mes applications répertoriant les applications auxquelles vous êtes autorisé à accéder](./media/perform-access-review/myapps-access-panel.png)

2. Cliquez sur la vignette **Révisions d’accès** pour afficher la liste des révisions d’accès en attente.

    > [!NOTE]
    > Si la vignette **Révisions d’accès** n’est pas visible, cela signifie qu’il n’y a pas de révisions d’accès à effectuer pour cette organisation et qu’aucune action n’est nécessaire pour l’instant.

![Liste des révisions d’accès en attente pour les applications et les groupes avec la bannière de nouvelle expérience disponible affichée pendant la durée de la préversion](./media/perform-access-review/banner.png)

3. Cliquez sur **Essayer** dans la bannière en haut de la page. Vous accédez alors à la nouvelle expérience Mon Accès.
  
### <a name="email"></a>E-mail

  >[!IMPORTANT]
> Il peut y avoir des retards dans la réception de l’e-mail et, dans certains cas, cela peut prendre jusqu’à 24 heures. Ajoutez azure-noreply@microsoft.com à votre liste des destinataires approuvés pour vous assurer de recevoir tous les e-mails.

   1. Recherchez un e-mail de Microsoft vous invitant à réviser les accès. Vous pouvez voir un exemple d’e-mail ci-dessous :

   ![Exemple d’e-mail de Microsoft pour la révision d’accès d’un groupe](./media/perform-access-review/access-review-email-preview.png)

   2. Cliquez sur le lien **Commencer une révision d'accès** pour ouvrir la révision d’accès.

>[!NOTE]
>Si ce clic vous amène au portail **Mes applications**, procédez de la manière décrite dans la section ci-dessus intitulée **portail Mes applications**.

### <a name="navigate-to-my-access-directly"></a>Accéder directement à Mon accès

Vous pouvez également afficher vos révisions d’accès en attente en ouvrant Mon accès dans votre navigateur.

1. Connectez-vous à Mon Accès à l’adresse https://myaccess.microsoft.com/.

2. Sélectionnez **Révisions d’accès** dans le menu de la barre latérale gauche pour afficher la liste des révisions d’accès en attente qui vous sont affectées.

   ![révisions d’accès dans le menu](./media/perform-access-review/access-review-menu.png)

### <a name="approve-or-deny-access-for-one-or-more-users"></a>Approuver ou refuser l’accès pour un ou plusieurs utilisateurs

Après avoir ouvert Mon accès sous Groupes et applications, vous voyez ceci :

- **Nom** Nom de la révision d’accès.
- **Échéance** Date d’échéance de la révision. Après cette date, des utilisateurs dont l’accès a été refusé pourraient être supprimés du groupe ou de l’application en cours de révision.
- **Ressource** Nom de la ressource objet de la révision.
- **Progression** Nombre d’utilisateurs révisés sur le nombre total d’utilisateurs entrant dans le cadre de la révision d’accès.

Cliquez sur le nom d’une révision d’accès pour commencer.

![Liste de révisions d’accès en attente pour les applications et les groupes](./media/perform-access-review/access-reviews-list-preview.png)

Une fois la révision d’accès ouverte, vous voyez la liste des utilisateurs auxquels elle s’étend. Si la requête vise à réviser votre propre accès, la page aura un aspect différent. Pour plus d’informations, consultez [Réviser son accès à des groupes ou à des applications](review-your-access.md).

Il y a deux manières d’approuver ou de refuser l’accès  :

- Vous pouvez approuver ou refuser l’accès manuellement pour un ou plusieurs utilisateurs.
- Vous pouvez accepter les recommandations du système.

#### <a name="manually-approve-or-deny-access-for-one-or-more-users"></a>Approuver ou refuser manuellement l’accès pour un ou plusieurs utilisateurs

1. Passez en revue la liste des utilisateurs pour décider s’il faut approuver ou refuser leur accès permanent.
2. Sélectionnez un ou plusieurs utilisateurs en cliquant sur le cercle en regard de leurs noms.
3. Sélectionnez **Approuver** ou **Refuser** dans la barre ci-dessus.
    - Si vous n’êtes pas sûr, vous pouvez cliquer sur **Je ne sais pas**. L’utilisateur peut alors conserver son accès et votre choix est enregistré dans les journaux d’audit. Il est important de garder à l’esprit que toutes les informations que vous fournissez sont disponibles pour d’autres réviseurs. Ceux-ci peuvent lire vos commentaires et en tenir compte quand ils révisent la demande.

    ![Révision d’accès ouverte répertoriant les utilisateurs à réviser](./media/perform-access-review/user-list-preview.png)

4. L’administrateur de la révision d’accès peut exiger que vous motiviez votre décision dans la boîte de dialogue **Motif**. Même si aucun motif n’est requis. Vous pouvez toujours fournir un motif justifiant votre décision. Les informations que vous incluez sont disponibles pour d’autres approbateurs de la révision.

5. Cliquez sur **Envoyer**.
    - Vous pouvez modifier votre réponse à tout moment jusqu'à ce que la révision d’accès soit terminée. Si vous souhaitez modifier votre réponse, sélectionnez la ligne et mettez à jour la réponse. Par exemple, vous pouvez approuver un utilisateur précédemment refusé ou refuser un utilisateur précédemment accepté.

 >[!IMPORTANT]
 > - Si un utilisateur se voit refuser l’accès, il n’est pas supprimé immédiatement. Il est supprimé une fois la période de révision terminée ou quand un administrateur interrompt la révision. 
 > - S’il existe plusieurs réviseurs, la dernière réponse envoyée est enregistrée. Prenons un exemple où un administrateur désigne deux réviseurs : Alice et Bob. Alice ouvre la révision d’accès en premier et approuve la demande d’accès d’un utilisateur. Avant la fin de la période de révision, Bob ouvre la révision d’accès et refuse l’accès sur la demande précédemment approuvée par Alice. La dernière décision qui refuse l’accès est la réponse enregistrée.

#### <a name="approve-or-deny-access-based-on-recommendations"></a>Approuver ou refuser l’accès selon les recommandations

Pour rendre les révisions d’accès plus faciles et plus rapides pour vous, nous fournissons également des recommandations que vous pouvez accepter en un seul clic. Les recommandations sont générées en fonction de l’activité de connexion de l’utilisateur.

1. Sélectionnez un ou plusieurs utilisateurs, puis cliquez sur **Accepter les recommandations**.

    ![Liste de révision d’accès ouverte montrant le bouton Accepter les recommandations](./media/perform-access-review/accept-recommendations-preview.png)

1. Cliquez sur **Envoyer** pour accepter les recommandations.

Pour accepter les recommandations pour tous les utilisateurs, assurez-vous qu’aucun d’eux n’est sélectionné, puis cliquez sur le bouton **Accepter les recommandations** dans la barre supérieure.

>[!NOTE]
>Lorsque vous acceptez les recommandations, les décisions précédentes ne sont pas modifiées.

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer une révision d’accès de groupes ou révisions d’accès des applications dans Azure AD](complete-access-review.md)
