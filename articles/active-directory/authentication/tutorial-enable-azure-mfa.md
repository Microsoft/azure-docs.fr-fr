---
title: Activer Azure AD Multi-Factor Authentication
description: Dans ce tutoriel, vous allez apprendre à activer Azure AD Multi-Factor Authentication pour un groupe d’utilisateurs, et à tester l’invite du facteur secondaire pendant un événement de connexion.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: justinha
author: justinha
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 32228e90e3cfc064cd5be9cd0655f321ab3e2809
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741148"
---
# <a name="tutorial-secure-user-sign-in-events-with-azure-ad-multi-factor-authentication"></a>Tutoriel : Événements de connexion utilisateur sécurisée avec Azure AD Multi-Factor Authentication

L’authentification MFA (Azure Multi-Factor Authentication) est un processus dans lequel un utilisateur est invité, au cours d’un événement de connexion, à utiliser des formes d’identification supplémentaires. Il peut s’agir en ce qui le concerne d’entrer un code sur son téléphone portable ou de scanner son empreinte digitale. Quand vous exigez une deuxième forme d’authentification, la sécurité est accrue, car ce facteur supplémentaire n’est pas un élément facile à obtenir ou à dupliquer par un attaquant.

Azure AD Multi-Factor Authentication et les stratégies d’accès conditionnel offrent la flexibilité nécessaire qui permet l’authentification MFA des utilisateurs pendant des événements de connexion spécifiques.

> [!IMPORTANT]
> Ce tutoriel montre aux administrateurs comment activer Azure AD Multi-Factor Authentication.
>
> Si votre équipe informatique n’a pas activé la possibilité d’utiliser Azure AD Multi-Factor Authentication, ou si vous rencontrez des problèmes lors de la connexion, contactez votre support technique pour obtenir de l’aide.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une stratégie d’accès conditionnel permettant d’activer AD Azure Multi-Factor Authentication pour un groupe d’utilisateurs
> * Configurer les conditions de stratégie qui demandent l’authentification MFA
> * Tester le processus MFA en tant qu’utilisateur

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un locataire Azure AD actif avec au moins un abonnement Azure AD Premium P1 ou une licence d’évaluation activée.
    * Si nécessaire, [créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un compte avec des privilèges d’*administrateur général*.
* Un utilisateur non-administrateur avec un mot de passe que vous connaissez, par exemple *testuser*. Vous testez l’expérience Azure AD Multi-Factor Authentication de l’utilisateur final à l’aide de ce compte dans ce tutoriel.
    * Si vous devez créer un utilisateur, consultez [Démarrage rapide : Ajouter de nouveaux utilisateurs à Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).
* Un groupe dont l’utilisateur non-administrateur est membre, par exemple *MFA-Test-Group*. Vous activez Azure AD Multi-Factor Authentication pour ce groupe dans ce tutoriel.
    * Si vous devez créer un groupe, consultez la procédure à suivre dans [Créer un groupe et ajouter des membres dans Azure Active Directory](../fundamentals/active-directory-groups-create-azure-portal.md).

## <a name="create-a-conditional-access-policy"></a>Créer une stratégie d’accès conditionnel

Il est recommandé de se servir des stratégies d’accès conditionnel pour activer et utiliser Microsoft Azure AD Multi-Factor Authentication. L’accès conditionnel vous permet de créer et de définir des stratégies qui réagissent aux événements de connexion en demandant des actions supplémentaires avant d’autoriser un utilisateur à accéder à une application ou à un service.

![Diagramme de vue d’ensemble du fonctionnement de l’accès conditionnel pour sécuriser le processus de connexion](media/tutorial-enable-azure-mfa/conditional-access-overview.png)

Les stratégies d’accès conditionnel peuvent être granulaires et spécifiques, avec pour objectif de permettre aux utilisateurs d’être productifs partout et à tout moment, tout en protégeant votre organisation. Dans ce tutoriel, nous allons créer une stratégie d’accès conditionnel de base pour demander l’authentification MFA lorsqu’un utilisateur se connecte au portail Azure. Dans un prochain tutoriel de cette série, vous configurerez Azure AD Multi-Factor Authentication à l’aide d’une stratégie d’accès conditionnel basée sur le risque.

Tout d’abord, créez une stratégie d’accès conditionnel et affectez votre groupe test d’utilisateurs comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte disposant d’autorisations d’*administrateur général*.
1. Recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Sécurité** dans le menu de gauche.
1. Sélectionnez **Accès conditionnel**, puis choisissez **+ Nouvelle stratégie**.
1. Entrez le nom de la stratégie, par exemple *Pilote MFA*.
1. Sous **Affectations**, choisissez **Utilisateurs et groupes**, puis activez la case d’option **Sélectionner des utilisateurs et des groupes**.
1. Cochez la case **Utilisateurs et groupes**, puis choisissez **Sélectionner** pour parcourir les utilisateurs et les groupes Azure AD disponibles.
1. Recherchez et sélectionnez votre groupe Azure AD, par exemple *MFA-Test-Group*, puis choisissez **Sélectionner**.

    [ ![Sélectionner votre groupe Azure AD à utiliser avec la stratégie d’accès conditionnel](media/tutorial-enable-azure-mfa/select-group-for-conditional-access-cropped.png) ](media/tutorial-enable-azure-mfa/select-group-for-conditional-access.png#lightbox)

1. Pour appliquer la stratégie d’accès conditionnel au groupe, sélectionnez **Terminé**.

## <a name="configure-the-conditions-for-multi-factor-authentication"></a>Configurer les conditions pour l’authentification multifacteur

La stratégie d’accès conditionnel étant créée et un groupe test d’utilisateurs attribué, définissez maintenant les actions ou les applications cloud qui déclenchent la stratégie. Ces actions ou applications cloud représentent les scénarios pour lesquels vous décidez de demander un processus supplémentaire, par exemple pour exiger l’authentification MFA. Ainsi, vous pouvez décider que l’accès à une application financière ou l’utilisation d’outils de gestion nécessite une invite de vérification supplémentaire.

Pour ce tutoriel, configurez la stratégie d’accès conditionnel afin d’exiger l’authentification MFA lorsqu’un utilisateur se connecte au portail Azure.

1. Sélectionnez **Applications ou actions cloud**. Vous pouvez choisir d’appliquer la stratégie d’accès conditionnel à *Toutes les applications Cloud* ou *Sélectionner des applications*. Pour assurer la flexibilité, vous pouvez également exclure certaines applications de la stratégie.

    Pour ce tutoriel, dans la page *Inclure*, choisissez la case d’option **Sélectionner des applications**.

1. Choisissez **Sélectionner**, puis parcourez la liste des événements de connexion disponibles qui peuvent être utilisés.

    Pour ce tutoriel, choisissez **Gestion Microsoft Azure** afin que la stratégie s’applique aux événements de connexion sur le portail Azure.

1. Pour l’appliquer aux applications sélectionnées, choisissez **Sélectionner**, puis **Terminé**.

    ![Sélectionner l’application Gestion Microsoft Azure à inclure dans la stratégie d’accès conditionnel](media/tutorial-enable-azure-mfa/select-azure-management-app.png)

Les contrôles d’accès vous permettent de définir les conditions à remplir pour qu’un utilisateur dispose d’un accès, par exemple le besoin d’une application cliente approuvée ou l’utilisation d’un appareil joint à Azure AD Hybride. Dans ce tutoriel, configurez les contrôles d’accès pour demander l’authentification MFA lors d’un événement de connexion sur le portail Azure.

1. Sous *Contrôle d’accès*, choisissez **Accorder**, puis assurez-vous que la case d’option **Accorder l’accès** est sélectionnée.
1. Cochez la case **Exiger une authentification multifacteur**, puis choisissez **Sélectionner**.

Les stratégies d’accès conditionnel peuvent être définies sur *Rapport seul* si vous souhaitez voir comment la configuration affecte les utilisateurs, ou sur *Désactivée* si vous ne voulez pas utiliser la stratégie pour le moment. Étant donné qu’un groupe test d’utilisateurs était ciblé pour ce tutoriel, activez la stratégie, puis testez Azure AD Multi-Factor Authentication.

1. Basculez *Activer la stratégie* sur **Activé**.
1. Pour appliquer la stratégie d’accès conditionnel, sélectionnez **Créer**.

## <a name="test-azure-ad-multi-factor-authentication"></a>Tester Azure AD Multi-Factor Authentication

Nous allons voir à présent votre stratégie d’accès conditionnel et Azure AD Multi-Factor Authentication en action. Tout d’abord, connectez-vous de la façon suivante à une ressource qui ne nécessite pas d’authentification MFA :

1. Ouvrez une nouvelle fenêtre de navigateur en mode de navigation privée ou InPrivate, et accédez à [https://account.activedirectory.windowsazure.com](https://account.activedirectory.windowsazure.com).
1. Connectez-vous avec votre utilisateur test non-administrateur, par exemple *testuser*. Il n’y a aucune invite vous demandant de procéder à l’authentification MFA.
1. Fermez la fenêtre du navigateur.

À présent, connectez-vous au portail Azure. Étant donné que le portail Azure était configuré dans la stratégie d’accès conditionnel pour exiger une vérification supplémentaire, vous obtenez une invite Azure AD Multi-Factor Authentication.

1. Ouvrez une nouvelle fenêtre de navigateur dans InPrivate ou en mode de navigation privée, et accédez à [https://portal.azure.com](https://portal.azure.com).
1. Connectez-vous avec votre utilisateur test non-administrateur, par exemple *testuser*. Il vous est demandé de vous inscrire et d’utiliser Azure AD Multi-Factor Authentication. Suivez les invites pour aller au bout du processus et vérifiez que vous vous connectez correctement au portail Azure.

    ![Suivre les invites du navigateur, puis l’invite de votre authentification multifacteur inscrite pour vous connecter](media/tutorial-enable-azure-mfa/azure-multi-factor-authentication-browser-prompt.png)

1. Fermez la fenêtre du navigateur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne souhaitez plus utiliser la stratégie d’accès conditionnel pour activer le service Azure AD Multi-Factor Authentication configuré dans le cadre de ce tutoriel, supprimez la stratégie en suivant ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Sécurité** dans le menu de gauche.
1. Sélectionnez **Accès conditionnel**, puis choisissez la stratégie que vous avez créée, par exemple *Pilote MFA*.
1. Choisissez **Supprimer**, puis confirmez que vous souhaitez supprimer la stratégie.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez activé Azure AD Multi-Factor Authentication au moyen de stratégies d’accès conditionnel pour un groupe sélectionné d’utilisateurs. Vous avez appris à :

> [!div class="checklist"]
> * Créer une stratégie d’accès conditionnel permettant d’activer Azure AD Multi-Factor Authentication pour un groupe d’utilisateurs Azure AD
> * Configurer les conditions de stratégie qui demandent l’authentification MFA
> * Tester le processus MFA en tant qu’utilisateur

> [!div class="nextstepaction"]
> [Activer la réécriture du mot de passe pour la réinitialisation de mot de passe en libre-service (SSPR)](./tutorial-enable-sspr-writeback.md)