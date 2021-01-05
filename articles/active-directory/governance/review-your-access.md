---
title: Réviser votre accès aux groupes et aux applications dans les révisions d’accès - Azure AD
description: Découvrez comment réviser votre propre accès à des groupes ou applications dans les révisions d’accès Azure Active Directory.
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
ms.openlocfilehash: b3fef2f85ca7e7b4034c8582477796d49446ea44
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746777"
---
# <a name="review-access-for-yourself-to-groups-or-applications-in-azure-ad-access-reviews"></a>Réviser votre accès à des groupes ou applications dans Azure AD

Azure Active Directory (Azure AD) simplifie la manière dont les entreprises gèrent l’accès aux applications ou groupes dans Azure AD et d’autres services en ligne Microsoft, avec une fonction appelée révision d’accès Azure AD.

Cet article décrit comment réviser votre propre accès à un groupe ou une application.

## <a name="review-your-access-using-my-apps"></a>Passer en revue votre accès à l’aide de Mes applications

La première étape pour effectuer une révision d’accès consiste à rechercher et ouvrir la révision d’accès.

>[!IMPORTANT]
> Il peut y avoir des retards dans la réception de l’e-mail et, dans certains cas, cela peut prendre jusqu’à 24 heures. Ajoutez azure-noreply@microsoft.com à votre liste de destinataires approuvés pour vous assurer de recevoir tous les e-mails.

1. Recherchez un e-mail de Microsoft vous invitant à réviser les accès. Voici un exemple d’e-mail vous invitant à réviser votre accès à un groupe.

    ![Exemple d’e-mail de Microsoft pour la révision d’accès à un groupe](./media/review-your-access/access-review-email.png)

1. Cliquez sur le lien **Réviser l’accès** pour ouvrir la révision d’accès.

Si vous n’avez pas reçu l’e-mail, vous trouverez les révisions d’accès en attente en procédant comme suit.

1. Connectez-vous au portail Mes applications à l’adresse [https://myapps.microsoft.com](https://myapps.microsoft.com).

    ![Portail Mes applications répertoriant les applications auxquelles vous êtes autorisé à accéder](./media/review-your-access/myapps-access-panel.png)

1. En haut à droite de la page, cliquez sur le symbole d’utilisateur, qui affiche votre nom et organisation par défaut. Si plusieurs organisations sont listées, sélectionnez l’organisation qui a demandé une révision d’accès.

1. Sur le côté droit de la page, cliquez sur la vignette **Révisions d’accès** pour voir la liste des révisions d’accès en attente.

    Si la vignette n’est pas visible, cela signifie qu’il n’y a pas de révisions d’accès à effectuer pour cette organisation et qu’aucune action n’est nécessaire pour l’instant.

    ![Liste de révisions d’accès en attente pour vos applications et groupes](./media/review-your-access/access-reviews-list.png)

1. Cliquez sur le lien **Commencer la révision** associé à la révision d’accès que vous souhaitez effectuer.

### <a name="perform-the-access-review"></a>Effectuer la révision d’accès

Une fois que vous avez ouvert la révision d’accès, vous pouvez voir votre accès.

1. Révisez votre accès et déterminez si vous en avez toujours besoin.

    Si la requête vise à réviser l’accès d’autres utilisateurs, la page aura un aspect différent. Pour plus d’informations, consultez [Réviser l’accès à des groupes ou à des applications](perform-access-review.md).

    ![Capture d’écran montrant une révision d’accès ouverte qui vous demande si vous avez toujours besoin d’accéder à un groupe.](./media/review-your-access/perform-access-review.png)

1. Cliquez sur **Oui** pour conserver votre accès ou sur **Non** pour supprimer votre accès.

1. Si vous cliquez sur **Oui**, vous devrez peut-être spécifier une justification dans la zone **Raison**.

    ![Capture d’écran montrant une révision d’accès terminée qui vous demande si vous avez toujours besoin d’accéder à un groupe, avec l’option « Oui » sélectionnée.](./media/review-your-access/perform-access-review-submit.png)

1. Cliquez sur **Envoyer**.

    Votre sélection est soumise et vous revenez au portail Mes applications.

    Si vous souhaitez modifier votre réponse, rouvrez la page des révisions d’accès et mettez à jour votre réponse. Vous pouvez modifier votre réponse à tout moment jusqu'à ce que la révision d’accès soit terminée.

    > [!NOTE]
    > Si vous avez indiqué que vous n’avez plus besoin de l’accès, votre accès n’est pas supprimé immédiatement. Vous êtes supprimé une fois la révision terminée ou lorsqu’un administrateur interrompt la révision.

## <a name="review-your-own-access-using-my-access-new"></a>Passer en revue votre propre accès en utilisant Mon Accès (nouveau)

Vous pouvez essayer la nouvelle expérience avec l’interface utilisateur mise à jour dans Mon Accès de deux façons :

### <a name="my-apps-portal"></a>Portail Mes applications

1. Connectez-vous au portail Mes applications à l’adresse [https://myapps.microsoft.com](https://myapps.microsoft.com).

    ![Portail Mes applications répertoriant les applications auxquelles vous êtes autorisé à accéder](./media/review-your-access/myapps-access-panel.png)

2. Cliquez sur la vignette **Révisions d’accès** pour afficher la liste des révisions d’accès en attente.

    > [!NOTE]
    > Si la vignette **Révisions d’accès** n’est pas visible, cela signifie qu’il n’y a pas de révisions d’accès à effectuer pour cette organisation et qu’aucune action n’est nécessaire pour l’instant.

3. Cliquez sur **Essayez !** dans la bannière en haut de la page pour accéder à la nouvelle expérience Mon Accès.

    ![Liste des révisions d’accès en attente pour les applications et les groupes avec la bannière de nouvelle expérience disponible affichée pendant la durée de la préversion](./media/review-your-access/banner-your-access.png)

4. Continuer dans la section **Effectuer la révision d’accès**

### <a name="email"></a>E-mail

>[!IMPORTANT]
> Il peut y avoir des retards dans la réception de l’e-mail et, dans certains cas, cela peut prendre jusqu’à 24 heures. Ajoutez azure-noreply@microsoft.com à votre liste de destinataires approuvés pour vous assurer de recevoir tous les e-mails.

1. Recherchez un e-mail de Microsoft vous invitant à réviser les accès. Vous pouvez voir un exemple d’e-mail ci-dessous :

 ![Exemple d’e-mail de Microsoft pour la révision d’accès d’un groupe](./media/review-your-access/access-review-email-preview.png)

2. Cliquez sur le lien **Réviser l’accès** pour ouvrir la révision d’accès.

3. Continuer dans la section **Effectuer la révision d’accès**

>[!NOTE]
>Si ce clic vous amène au portail **Mes applications**, procédez de la manière décrite dans la section ci-dessus intitulée **portail Mes applications**.

### <a name="directly-at-my-access"></a>Directement à Mon Accès

Vous pouvez également afficher vos révisions d’accès en attente en ouvrant Mon Accès dans votre navigateur.

1. Connectez-vous à Mon Accès à l’adresse https://myaccess.microsoft.com/.

2. Sélectionnez **Révisions d’accès** dans le menu de la barre latérale gauche pour afficher la liste des révisions d’accès en attente qui vous sont affectées.

   ![révisions d’accès dans le menu](./media/review-your-access/access-review-menu.png)

### <a name="perform-the-access-review"></a>Effectuer la révision d’accès

1. Sous Groupes et applications, vous pouvez voir :
    
    - **Nom** Nom de la révision d’accès.
    - **Échéance** Date d’échéance de la révision. Après cette date, des utilisateurs dont l’accès a été refusé pourraient être supprimés du groupe ou de l’application en cours de révision.
    - **Ressource** Nom de la ressource objet de la révision.
    - **Progression** Nombre d’utilisateurs révisés sur le nombre total d’utilisateurs entrant dans le cadre de la révision d’accès.
    
2. Cliquez sur le nom d’une révision d’accès pour commencer.

   ![Liste de révisions d’accès en attente pour les applications et les groupes](./media/review-your-access/access-reviews-list-preview.png)

3. Révisez votre accès et déterminez si vous en avez toujours besoin.

    Si la requête vise à réviser l’accès d’autres utilisateurs, la page aura un aspect différent. Pour plus d’informations, consultez [Réviser l’accès à des groupes ou à des applications](perform-access-review.md).

    ![Ouvrir la révision d’accès en vous demandant si vous avez toujours besoin de l’accès à un groupe](./media/review-your-access/review-access-preview.png)

1. Sélectionnez **Oui** pour conserver votre accès ou **Non** pour le supprimer.

1. Si vous cliquez sur **Oui**, vous devrez peut-être spécifier une justification dans la zone **Raison**.

    ![Révision d’accès déterminant si vous avez toujours besoin de l’accès à un groupe terminée](./media/review-your-access/review-access-yes-preview.png)

1. Cliquez sur **Envoyer**.

    Votre sélection est soumise et vous revenez au portail Mon Accès.

    Si vous souhaitez modifier votre réponse, rouvrez la page de révisions d’accès et mettez à jour votre réponse. Vous pouvez modifier votre réponse à tout moment jusqu'à ce que la révision d’accès soit terminée.

    > [!NOTE]
    > Si vous avez indiqué que vous n’avez plus besoin de l’accès, votre accès n’est pas supprimé immédiatement. Vous êtes supprimé une fois la révision terminée ou lorsqu’un administrateur interrompt la révision.

## <a name="next-steps"></a>Étapes suivantes

- [Effectuer une révision d’accès de groupes ou révisions d’accès des applications dans Azure AD](complete-access-review.md)
