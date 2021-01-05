---
title: Configurer l’authentification multifacteur Azure pour Windows Virtual Desktop - Azure
description: Comment configurer l’authentification multifacteur Azure pour une sécurité accrue dans Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 12/10/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 00aba5d169a05eab25dcc63ca813955e71d09598
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092378"
---
# <a name="enable-azure-multifactor-authentication-for-windows-virtual-desktop"></a>Activer l’authentification multifacteur Azure pour Windows Virtual Desktop

>[!IMPORTANT]
> Si vous visitez cette page à partir de la documentation Windows Virtual Desktop (classique), veillez à [revenir à la documentation Windows Virtual Desktop (classique)](./virtual-desktop-fall-2019/tenant-setup-azure-active-directory.md) une fois que vous avez terminé.

Le client Windows pour Windows Virtual Desktop est une bonne option pour l’intégration de Windows Virtual Desktop à votre machine locale. Cependant, quand vous configurez votre compte Windows Virtual Desktop dans le client Windows, vous devez prendre certaines mesures pour garantir votre propre sécurité et celle de vos utilisateurs.

Lorsque vous vous connectez pour la première fois, le client demande votre nom d’utilisateur, votre mot de passe et votre authentification multifacteur Azure. Après cela, la prochaine fois que vous vous connectez, le client a mémorisé votre jeton pour votre application d’entreprise Azure Active Directory (AD). Lorsque vous sélectionnez **Mémoriser mes informations** dans l’invite d’informations d’identification pour l’hôte de session, vos utilisateurs peuvent se connecter après le redémarrage du client sans devoir entrer à nouveau leurs informations d’identification.

Bien que la mémorisation des informations d’identification soit pratique, elle peut également rendre moins sécurisés les déploiements sur des scénarios d’entreprise ou des appareils personnels. Pour protéger vos utilisateurs, vous pouvez faire en sorte que le client demande plus fréquemment des informations d’identification d’authentification multifacteur Azure. Cet article vous montre comment configurer la stratégie d’accès conditionnel pour Windows Virtual Desktop de façon à activer ce paramètre.

## <a name="prerequisites"></a>Prérequis

Voici ce dont vous avez besoin pour commencer :

- Attribuez aux utilisateurs une licence qui comprend Azure Active Directory Premium P1 ou P2.
- Un groupe Azure Active Directory avec les utilisateurs affectés en tant que membres du groupe.
- Activez l’authentification multifacteur Azure pour tous vos utilisateurs. Pour plus d’informations sur la façon de le faire, consultez [Comment exiger la vérification en deux étapes pour un utilisateur](../active-directory/authentication/howto-mfa-userstates.md#view-the-status-for-a-user).

> [!NOTE]
> Le paramètre suivant s’applique également au [client web Windows Virtual Desktop](https://rdweb.wvd.microsoft.com/arm/webclient/index.html).

## <a name="create-a-conditional-access-policy"></a>Créer une stratégie d’accès conditionnel

Voici comment créer une stratégie d’accès conditionnel qui exige l’authentification multifacteur lors de la connexion à Windows Virtual Desktop :

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
2. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
3. Sélectionnez **Nouvelle stratégie**.
4. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
5. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**.
6. Sous **Inclure**, sélectionnez **Sélectionner des utilisateurs et des groupes** > **Utilisateurs et groupes** > choisissez le groupe créé lors de la phase [Prérequis](#prerequisites).
7. Sélectionnez **Terminé**.
8. Sous **Applications ou actions cloud** > **Inclure**, sélectionnez **Sélectionner les applications**.
9. Sélectionnez l’une des applications suivantes en fonction de la version de Windows Virtual Desktop que vous utilisez.
   
   - Si vous utilisez Windows Virtual Desktop (classique), choisissez ces applications :
       
       - **Windows Virtual Desktop** (ID de l’application 5a0aa725-4958-4b0c-80a9-34562e23f3b7)
       - **Client Windows Virtual Desktop** (ID d’application fa4345a4-A730-4230-84a8-7d9651b86739), qui vous permet de définir des stratégies sur le client web
       
        Après cela, passez directement à l’étape 11.

   - Si vous utilisez Windows Virtual Desktop, choisissez plutôt l’application suivante :
       
       -  **Windows Virtual Desktop** (ID de l’application 9cdead84-a844-4324-93f2-b2e6bb768d07)
       
        Après cela, passez à l’étape 10.

   >[!IMPORTANT]
   > Ne sélectionnez pas l’application nommée Windows Virtual Desktop – Fournisseur Azure Resource Manager (50e95039-b200-4007-bc97-8d5790743a63). Cette application, qui ne sert qu’à récupérer le flux utilisateur, ne doit pas comporter d’authentification multifacteur.
   > 
   > Si vous utilisez Windows Virtual Desktop (classique) et si la stratégie d’accès conditionnel bloque tous les accès et exclut uniquement les ID d’applications Windows Virtual Desktop, vous pouvez résoudre ce problème en ajoutant l’ID d’application 9cdead84-a844-4324-93f2-b2e6bb768d07 à la stratégie. Si vous n’ajoutez pas cet ID d’application, la détection de flux des ressources Windows Virtual Desktop (classique) sera bloquée.

10. Accédez à **Conditions** > **Applications clientes**, puis spécifiez où vous souhaitez appliquer la stratégie :
    
    - Sélectionnez **Navigateur** si vous souhaitez que la stratégie s’applique au client web.
    - Sélectionnez **Applications mobiles et clients de bureau** si vous souhaitez appliquer la stratégie à d’autres clients.
    - Activez les deux cases à cocher si vous souhaitez appliquer la stratégie à tous les clients.
   
    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la page des applications clientes. L’utilisateur a activé la case à cocher Applications mobiles et clients de bureau.](media/select-apply.png)

11. Une fois que vous avez sélectionné votre application, choisissez **Sélectionner**, puis sélectionnez **Terminé**.

    > [!div class="mx-imgBorder"]
    > ![Capture d’écran de la page Applications ou actions cloud. Les applications Windows Virtual Desktop et Client Windows Virtual Desktop sont mises en surbrillance en rouge.](media/cloud-apps-enterprise.png)

    >[!NOTE]
    >Pour rechercher l'ID de l’application que vous souhaitez sélectionner, accédez à **Applications d’entreprise**, puis sélectionnez **Applications Microsoft** dans le menu déroulant correspondant au type d’application.

12. Sous **Contrôles d’accès** > **Accorder**, sélectionnez **Accorder l’accès**, **Exiger une authentification multifacteur**, puis **Sélectionner**.
13. Sous **Contrôles d’accès** > **Session**, sélectionnez **Fréquence de connexion**, définissez la valeur sur la durée entre les invites de votre choix, puis sélectionnez **Sélectionner**. Par exemple, si vous définissez la valeur sur **1** et l’unité sur **Heures**, vous aurez besoin de l’authentification multifacteur si une connexion est lancée une heure après la dernière.
14. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
15. Sélectionnez **Créer** pour activer votre stratégie.

>[!NOTE]
>Lorsque vous utilisez le client web pour vous connecter à Windows Virtual Desktop par le biais de votre navigateur, le journal répertorie l’ID d’application cliente en tant que a85cf173-4192-42F8-81fa-777a763e6e2c (client Windows Virtual Desktop). Cela est dû au fait que l’application cliente est liée en interne à l’ID d’application serveur où la stratégie d’accès conditionnel a été définie. 

## <a name="next-steps"></a>Étapes suivantes

- [Apprenez-en davantage sur les stratégies d’accès conditionnel](../active-directory/conditional-access/concept-conditional-access-policies.md)

- [Apprenez-en davantage sur la fréquence de connexion des utilisateurs](../active-directory/conditional-access/howto-conditional-access-session-lifetime.md#user-sign-in-frequency)
