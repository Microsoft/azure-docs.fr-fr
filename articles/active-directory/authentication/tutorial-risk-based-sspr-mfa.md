---
title: Protection de la connexion des utilisateurs basée sur les risques dans Azure Active Directory
description: Dans ce tutoriel, vous allez apprendre à activer Azure Identity Protection pour protéger les utilisateurs lorsqu’un comportement de connexion risqué est détecté sur leur compte.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: justinha
author: justinha
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 97aabfa39954aa6ba937166eb54c05ac4874ea7e
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741336"
---
# <a name="tutorial-use-risk-detections-for-user-sign-ins-to-trigger-azure-ad-multi-factor-authentication-or-password-changes"></a>Tutoriel : Utiliser les détections de risques pour les connexions utilisateur afin de déclencher Azure AD Multi-Factor Authentication ou le changement du mot de passe

Pour protéger vos utilisateurs, vous pouvez configurer des stratégies reposant sur des risques dans Azure Active Directory (Azure AD) qui répondent automatiquement aux comportements à risque. Les stratégies d’Azure AD Identity Protection peuvent bloquer automatiquement une tentative de connexion ou nécessiter une action supplémentaire, par exemple exiger une modification de mot de passe ou demander Azure AD Multi-Factor Authentication. Ces stratégies fonctionnent avec les stratégies d’accès conditionnel Azure AD existantes en tant que couche supplémentaire de protection pour votre organisation. Les utilisateurs ne déclencheront peut-être jamais un comportement risqué dans l’une de ces stratégies, mais votre organisation est protégée si une tentative de compromission de sécurité se produit.

> [!IMPORTANT]
> Ce tutoriel montre aux administrateurs comment activer Azure AD Multi-Factor Authentication en fonction des risques.
>
> Si votre équipe informatique n’a pas activé la possibilité d’utiliser Azure AD Multi-Factor Authentication, ou si vous rencontrez des problèmes lors de la connexion, contactez votre support technique pour obtenir de l’aide.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Comprendre les stratégies disponibles pour Azure AD Identity Protection
> * Activer une inscription Azure AD Multi-Factor Authentication
> * Activer les modifications de mot de passe en fonction des risques
> * Activer l’authentification multifacteur en fonction des risques
> * Tester les stratégies basées sur les risques pour les tentatives de connexion de l’utilisateur

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un locataire Azure AD actif avec au moins un abonnement Azure AD Premium P2 ou une licence d’évaluation activée.
    * Si nécessaire, [créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un compte avec des privilèges d’*Administrateur général*.
* Azure AD configuré pour la réinitialisation de mot de passe en libre-service et Azure AD Multi-Factor Authentication
    * Si nécessaire, [suivez le tutoriel pour activer la réinitialisation de mot de passe en libre-service Azure AD](tutorial-enable-sspr.md).
    * Si nécessaire, [suivez le tutoriel pour activer Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md).

## <a name="overview-of-azure-ad-identity-protection"></a>Vue d’ensemble d’Azure AD Identity Protection

Chaque jour, Microsoft collecte et analyse des milliards de signaux anonymes dans le cadre de tentatives de connexion d’utilisateurs. Ces signaux aident à créer des modèles de comportement de connexion utilisateur légitime et à identifier les tentatives de connexion risquée potentielles. Azure AD Identity Protection peut examiner les tentatives de connexion des utilisateurs et prendre des mesures supplémentaires en cas de comportement suspect :

Certaines des actions suivantes peuvent déclencher la détection de risque d’Azure AD Identity Protection :

* Utilisateurs avec des informations d’identification volées.
* Connexions depuis des adresses IP anonymes.
* Voyage impossible vers des emplacements inhabituels.
* Connexions depuis des appareils infectés.
* Connexions depuis des adresses IP avec des activités suspectes.
* Connexions depuis des emplacements non connus.

Les trois stratégies suivantes sont disponibles dans Azure AD Identity Protection pour protéger les utilisateurs et répondre à une activité suspecte. Vous pouvez choisir d’activer ou de désactiver l’application de la stratégie, sélectionner les utilisateurs ou les groupes auxquels appliquer la stratégie et décider si vous souhaitez bloquer l’accès à la connexion ou demander une action supplémentaire.

* Stratégie de risque d’utilisateur
    * Identifie les comptes d’utilisateur qui peuvent avoir des informations d’identification compromises et y répond. Peut inviter l’utilisateur à créer un nouveau mot de passe.
* Stratégie en matière de risque à la connexion
    * Identifie et répond aux tentatives de connexion suspectes. Peut inviter l’utilisateur à fournir des formes supplémentaires de vérification à l’aide d’Azure AD Multi-Factor Authentication.
* Stratégie d'inscription MFA
    * Vérifiez que les utilisateurs se sont inscrits à Azure AD Multi-Factor Authentication. Si une stratégie de connexion à risque vous invite à utiliser l’authentification multifacteur, l’utilisateur doit déjà être inscrit pour Azure AD Multi-Factor Authentication.

Lorsque vous activez un utilisateur de stratégie ou une stratégie de risque de connexion, vous pouvez également choisir le seuil pour le niveau de risque : *Bas et supérieur*, *Moyen et supérieur* ou *Élevé*. Cette flexibilité vous permet de décider de la rigueur que vous souhaitez appliquer aux contrôles pour les événements de connexion suspects.

Vous trouverez plus d’informations sur Azure AD Identity Protection dans [What is Azure AD Identity Protection?](../identity-protection/overview-identity-protection.md) (Qu’est-ce qu’Azure AD Identity Protection ?)

## <a name="enable-mfa-registration-policy"></a>Activer la stratégie d’inscription MFA

Azure AD Identity Protection comprend une stratégie par défaut qui peut aider les utilisateurs à s’inscrire auprès d’Azure AD Multi-Factor Authentication. Si vous utilisez des stratégies supplémentaires pour protéger les événements de connexion, les utilisateurs doivent déjà avoir été inscrits pour l’authentification multifacteur. Lorsque vous activez cette stratégie, les utilisateurs n’ont pas besoin d’effectuer une authentification MFA à chaque événement de connexion. La stratégie vérifie uniquement l’état d’inscription d’un utilisateur et lui demande de se préinscrire si nécessaire.

Il est recommandé d’activer la stratégie d’inscription de l’authentification multifacteur pour les utilisateurs qui doivent être activés pour des stratégies Azure AD Identity Protection supplémentaires. Pour activer cette stratégie, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général.
1. Recherchez et sélectionnez **Azure Active Directory**, sélectionnez **Sécurité**, puis, sous l’en-tête de menu *Protéger*, choisissez **Identity Protection**.
1. Sélectionnez la **Stratégie d’inscription MFA** dans le menu de gauche.
1. Par défaut, la stratégie s’applique à *Tous les utilisateurs*. Si vous le souhaitez, sélectionnez **Affectations**, puis choisissez les utilisateurs ou les groupes auxquels appliquer la stratégie.
1. Sous *Contrôles*, sélectionnez **Accès**. Assurez-vous que l’option *Exiger l’inscription Azure AD MFA* est cochée, puis choisissez **Sélectionner**.
1. Définissez **Appliquer la stratégie** sur *Activé*, puis sélectionnez **Enregistrer**.

    ![Capture d’écran montrant comment obliger les utilisateurs à s’inscrire à l’authentification multifacteur dans le portail Azure](./media/tutorial-risk-based-sspr-mfa/enable-mfa-registration.png)

## <a name="enable-user-risk-policy-for-password-change"></a>Activer la stratégie de risque d’utilisateur pour la modification du mot de passe

Microsoft travaille avec des chercheurs, les forces de l’ordre, différentes équipes de sécurité chez Microsoft et autres sources de confiance pour trouver des paires nom d’utilisateur/mot de passe. Lorsqu’une de ces paires correspond à un compte dans votre environnement, une modification de mot de passe en fonction des risques peut être demandée. Cette stratégie et cette action nécessitent que l’utilisateur mette à jour son mot de passe avant de pouvoir se connecter, pour s’assurer que les informations d’identification précédemment exposées ne fonctionnent plus.

Pour activer cette stratégie, procédez comme suit :

1. Sélectionnez la **Stratégie de risque utilisateur** dans le menu de gauche.
1. Par défaut, la stratégie s’applique à *Tous les utilisateurs*. Si vous le souhaitez, sélectionnez **Affectations**, puis choisissez les utilisateurs ou les groupes auxquels appliquer la stratégie.
1. Sous *Conditions*, choisissez **Sélectionner les conditions > Sélectionner un niveau de risque**, puis choisissez *Moyen et supérieur*.
1. Choisissez **Sélectionner**, puis **Terminé**.
1. Sous *Accès*, sélectionnez **Accès**. Assurez-vous que l’option pour **Autoriser l’accès** et *Exiger la modification du mot de passe* est cochée, puis choisissez **Sélectionner**.
1. Définissez **Appliquer la stratégie** sur *Activé*, puis sélectionnez **Enregistrer**.

    ![Capture d’écran montrant comment activer la stratégie de risque utilisateur dans le portail Azure](./media/tutorial-risk-based-sspr-mfa/enable-user-risk-policy.png)

## <a name="enable-sign-in-risk-policy-for-mfa"></a>Activer la stratégie de connexion à risque pour l’authentification multifacteur

La plupart des utilisateurs ont un comportement normal qui peut être suivi. Lorsqu’ils sortent de cette norme, il peut être risqué de leur permettre de se connecter normalement. Au lieu de cela, vous souhaiterez peut-être bloquer cet utilisateur ou lui demander d’effectuer une authentification multifacteur. Si l’utilisateur réussit l’authentification multifacteur, vous pouvez considérer cela comme une tentative de connexion valide et accorder l’accès à l’application ou au service.

Pour activer cette stratégie, procédez comme suit :

1. Sélectionnez la **Stratégie de connexion à risque** dans le menu de gauche.
1. Par défaut, la stratégie s’applique à *Tous les utilisateurs*. Si vous le souhaitez, sélectionnez **Affectations**, puis choisissez les utilisateurs ou les groupes auxquels appliquer la stratégie.
1. Sous *Conditions*, choisissez **Sélectionner les conditions > Sélectionner un niveau de risque**, puis choisissez *Moyen et supérieur*.
1. Choisissez **Sélectionner**, puis **Terminé**.
1. Sous *Accès*, choisissez **Sélectionner un contrôle**. Assurez-vous que l’option pour **Autoriser l’accès** et *Exiger l’authentification multifacteur* est cochée, puis choisissez **Sélectionner**.
1. Définissez **Appliquer la stratégie** sur *Activé*, puis sélectionnez **Enregistrer**.

    ![Capture d’écran montrant comment activer la stratégie de connexion à risque dans le portail Azure](./media/tutorial-risk-based-sspr-mfa/enable-sign-in-risk-policy.png)

## <a name="test-risky-sign-events"></a>Tester les événements de connexion à risque

La plupart des événements de connexion utilisateur ne déclenchent pas les stratégies basées sur les risques configurées dans les étapes précédentes. Un utilisateur peut ne jamais voir une invite pour une authentification multifacteur supplémentaire ou réinitialiser son mot de passe. Si ses informations d’identification restent sécurisées et que son comportement est cohérent, les événements de connexion réussissent.

Pour tester les stratégies Azure AD Identity Protection créées au cours des étapes précédentes, vous avez besoin d’un moyen de simuler un comportement risqué ou des attaques potentielles. Les étapes à suivre pour effectuer ces tests varient en fonction de la stratégie Azure AD Identity Protection que vous souhaitez valider. Pour plus d’informations sur les scénarios et la démarche à suivre, consultez [Simuler des détections de risques dans Azure AD Identity Protection](../identity-protection/howto-identity-protection-simulate-risk.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez terminé le test et souhaitez désactiver les stratégies reposant sur des risques, retournez dans chaque stratégie à désactiver et définissez l’option **Appliquer la stratégie** sur *Désactivé*.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez activé des stratégies utilisateur basées sur les risques pour Azure AD Identity Protection. Vous avez appris à :

> [!div class="checklist"]
> * Comprendre les stratégies disponibles pour Azure AD Identity Protection
> * Activer une inscription Azure AD Multi-Factor Authentication
> * Activer les modifications de mot de passe en fonction des risques
> * Activer l’authentification multifacteur en fonction des risques
> * Tester les stratégies basées sur les risques pour les tentatives de connexion de l’utilisateur

> [!div class="nextstepaction"]
> [En savoir plus sur Azure AD Identity Protection](../identity-protection/overview-identity-protection.md)
