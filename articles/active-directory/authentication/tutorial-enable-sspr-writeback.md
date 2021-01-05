---
title: Activer la réécriture du mot de passe Azure Active Directory
description: Dans ce tutoriel, vous apprenez à activer la réécriture de la réinitialisation du mot de passe en libre-service Azure AD au moyen d’Azure AD Connect pour synchroniser les modifications dans un environnement Active Directory Domain Services local.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: justinha
author: justinha
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.custom: contperf-fy20q4
ms.openlocfilehash: 444ca19732921b336cae32a9b1eb5755a08e4bd3
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028051"
---
# <a name="tutorial-enable-azure-active-directory-self-service-password-reset-writeback-to-an-on-premises-environment"></a>Tutoriel : Activer la réécriture de la réinitialisation du mot de passe en libre-service Azure Active Directory dans un environnement local

Avec SSPR Azure AD, la réinitialisation du mot de passe en libre-service Azure Active Directory, les utilisateurs peuvent mettre à jour leur mot de passe ou déverrouiller leur compte en utilisant un navigateur web. Dans un environnement hybride, où Azure AD est connecté à un environnement Active Directory Domain Services (AD DS) local, ce scénario peut entraîner une différence entre les mots de passe des deux annuaires.

La réécriture du mot de passe peut être utilisée pour synchroniser les changements de mot de passe dans Azure AD sur votre environnement AD DS local. Azure AD Connect fournit un mécanisme sécurisé qui renvoie ces changements de mot de passe à un annuaire local existant d’Azure AD.

> [!IMPORTANT]
> Ce tutoriel montre à un administrateur comment réactiver la réinitialisation de mot de passe en libre-service dans un environnement local. Si vous êtes un utilisateur final déjà inscrit pour la réinitialisation de mot de passe en libre-service et que vous devez récupérer votre compte, accédez à https://aka.ms/sspr.
>
> Si votre équipe informatique n’a pas activé la réinitialisation de votre propre mot de passe, contactez votre support technique pour obtenir une assistance supplémentaire.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer les autorisations nécessaires pour la réécriture du mot de passe
> * Activer l’option de réécriture du mot de passe dans Azure AD Connect
> * Activer la réécriture du mot de passe dans SSPR Azure AD

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un locataire Azure AD actif avec au moins un abonnement Azure AD Premium P1 ou une licence d’évaluation activée.
    * Si nécessaire, [créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
    * Pour plus d’informations, consultez [Conditions de licence pour la réinitialisation de mot de passe en libre-service Azure AD](concept-sspr-licensing.md).
* Un compte avec des privilèges d’*administrateur général*.
* Azure AD configuré pour la réinitialisation du mot de passe en libre-service.
    * Si nécessaire, [suivez le tutoriel précédent pour activer la réinitialisation de mot de passe en libre-service Azure AD](tutorial-enable-sspr.md).
* Un environnement AD DS local existant, configuré avec une version actuelle d’Azure AD Connect.
    * Si nécessaire, configurez Azure AD Connect en utilisant la configuration [Rapide](../hybrid/how-to-connect-install-express.md) ou [Personnalisée](../hybrid/how-to-connect-install-custom.md).
    * Pour utiliser la réécriture du mot de passe, vos contrôleurs de domaine doivent exécuter Windows Server 2012 ou une version ultérieure.

## <a name="configure-account-permissions-for-azure-ad-connect"></a>Configurer les autorisations de compte pour Azure AD Connect

Azure AD Connect vous permet de synchroniser utilisateurs, groupes et informations d’identification entre un environnement AD DS local et Azure AD. Généralement, vous installez Azure AD Connect sur un ordinateur Windows Server 2012 ou ultérieur joint au domaine AD DS local.

Pour fonctionner correctement avec la réécriture SSPR, le compte spécifié dans Azure AD Connect doit disposer des autorisations et options appropriées définies. Si vous ne savez pas quel compte est actuellement utilisé, ouvrez Azure AD Connect et sélectionnez l’option **Afficher la configuration actuelle**. Le compte auquel vous devez ajouter des autorisations est listé sous **Annuaires synchronisés**. Les autorisations et options suivantes doivent être définies sur le compte :

* **Réinitialiser le mot de passe**
* **Autorisations en écriture** sur `lockoutTime`
* **Autorisations en écriture** sur `pwdLastSet`
* **Droits étendus** pour « Ne pas faire expirer le mot de passe » sur l’objet racine de *chaque domaine* de cette forêt, s’il n’est pas déjà défini.

Si vous n’attribuez pas ces autorisations, la réécriture peut sembler être configurée correctement, mais les utilisateurs rencontrent des erreurs quand ils gèrent leurs mots de passe locaux à partir du cloud. Les autorisations doivent être appliquées à **Cet objet et tous ceux descendants** pour que l’option « Ne pas faire expirer le mot de passe » soit proposée.  

> [!TIP]
>
> Si les mots de passe de certains comptes d’utilisateur ne sont pas réécrits dans l’annuaire local, vérifiez que l’héritage n’est pas désactivé pour le compte dans l’environnement local AD DS. Les autorisations d’écriture pour les mots de passe doivent être appliquées aux objets descendants pour que la fonctionnalité fonctionne correctement.

Afin de configurer les autorisations appropriées pour l’écriture différée de mot de passe, procédez comme suit :

1. Dans votre environnement AD DS local, ouvrez **Utilisateurs et ordinateurs Active Directory** avec un compte disposant des autorisations *d’administrateur de domaine* appropriées.
1. Dans le menu **Affichage**, assurez-vous que l’option **Fonctionnalités avancées** est activée.
1. Dans le volet gauche, cliquez avec le bouton droit sur l’objet représentant la racine du domaine, puis sélectionnez **Propriétés** > **Sécurité** > **Avancée**.
1. Dans l’onglet **Autorisations**, sélectionnez **Ajouter**.
1. Pour **Principal**, sélectionnez le compte auquel les autorisations doivent être appliquées (compte utilisé par Azure AD Connect).
1. Dans la liste déroulante **S’applique à**, sélectionnez **Objets utilisateur descendants**.
1. Sous *Autorisations*, cochez la case correspondant à l’option suivante :
    * **Réinitialiser le mot de passe**
1. Sous *Propriétés*, cochez les cases correspondant aux options suivantes. Faites défiler la liste pour rechercher ces options, qui peuvent être déjà définies par défaut :
    * **Écrire lockoutTime**
    * **Écrire pwdLastSet**

    [ ![Définir les autorisations appropriées dans Utilisateurs et ordinateurs Active Directory pour le compte utilisé par Azure AD Connect](media/tutorial-enable-sspr-writeback/set-ad-ds-permissions-cropped.png) ](media/tutorial-enable-sspr-writeback/set-ad-ds-permissions.png#lightbox)

1. Lorsque vous êtes prêt, sélectionnez **Appliquer / OK** pour appliquer les changements et fermer les boîtes de dialogue ouvertes.

Lorsque vous mettez à jour des autorisations, la réplication de ces autorisations pour tous les objets de l’annuaire peut durer une heure ou plus.

Les stratégies de mot de passe dans l’environnement AD DS local peuvent empêcher le traitement correct des réinitialisations de mot de passe. Pour que la réécriture du mot de passe fonctionne le plus efficacement possible, la stratégie de groupe pour *Âge minimal du mot de passe* doit être définie sur 0. Ce paramètre se trouve sous **Configuration ordinateur > Stratégies > Paramètres Windows > Paramètres de sécurité > Stratégies de compte** dans `gpedit.msc`.

Si vous mettez à jour la stratégie de groupe, attendez la réplication de la stratégie mise à jour ou utilisez la commande `gpupdate /force`.

> [!Note]
> Pour que les mots de passe soient changés immédiatement, la réécriture du mot de passe doit être définie sur 0. Toutefois, si les utilisateurs adhèrent aux stratégies locales et que le paramètre *Âge minimal du mot de passe* est défini sur une valeur supérieure à zéro, la réécriture du mot de passe fonctionne toujours une fois les stratégies locales évaluées.

## <a name="enable-password-writeback-in-azure-ad-connect"></a>Activer la réécriture du mot de passe dans Azure AD Connect

Une des options de configuration dans Azure AD Connect concerne la réécriture du mot de passe. Lorsque cette option est activée, les événements de changement de mot de passe amènent Azure AD Connect à synchroniser les informations d’identification mises à jour dans l’environnement AD DS local.

Pour activer la réécriture SSPR, commencez par activer l’option de réécriture dans Azure AD Connect. À partir de votre serveur Azure AD Connect, effectuez les étapes suivantes :

1. Connectez-vous à votre serveur Azure AD Connect et démarrez l’Assistant de configuration **Azure AD Connect**.
1. Sur la page d’**accueil**, sélectionnez **Configurer**.
1. Sur la page **Tâches supplémentaires**, sélectionnez **Personnalisation des options de synchronisation**, puis cliquez sur **Suivant**.
1. Dans la page **Connexion à Azure AD**, entrez les informations d’identification d’administrateur général pour votre locataire Azure et sélectionnez **Suivant**.
1. Sur les pages **Connexion des annuaires** et **Filtrage par domaine/unité d’organisation**, sélectionnez **Suivant**.
1. Sur la page **Fonctionnalités facultatives**, cochez la case située à côté de l’option **Écriture différée de mot de passe**, puis sélectionnez **Suivant**.

    ![Configurer Azure AD Connect pour la réécriture du mot de passe](media/tutorial-enable-sspr-writeback/enable-password-writeback.png)

1. Sur la page **Prêt à configurer**, sélectionnez **Configurer** et attendez la fin du processus.
1. Lorsque la configuration prend fin, sélectionnez **Quitter**.

## <a name="enable-password-writeback-for-sspr"></a>Activer la réécriture du mot de passe pour SSPR

La réécriture du mot de passe étant activée dans Azure AD Connect, configurez à présent SSPR Azure AD pour la réécriture. Lorsque vous autorisez l’utilisation par SSPR de la réécriture du mot de passe, les utilisateurs qui changent ou réinitialisent leur mot de passe bénéficient également de la synchronisation du mot de passe mis à jour dans l’environnement AD DS local.

Pour activer la réécriture du mot de passe dans SSPR, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général.
1. Recherchez et sélectionnez **Azure Active Directory**, puis sélectionnez **Réinitialisation de mot de passe** et choisissez **Intégration locale**.
1. Définissez l’option **Réécrire des mots de passe dans votre annuaire local ?** sur *Oui*.
1. Définissez l’option **Voulez-vous autoriser les utilisateurs à déverrouiller les comptes sans réinitialiser leur mot de passe ?** sur *Oui*.

    ![Activer la réinitialisation de mot de passe en libre-service Azure AD pour la réécriture du mot de passe](media/tutorial-enable-sspr-writeback/enable-sspr-writeback.png)

1. Quand vous êtes prêt, sélectionnez **Enregistrer**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous décidez de ne plus utiliser la fonctionnalité de réécriture de SSPR que vous avez configurée dans ce tutoriel, effectuez les étapes suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez et sélectionnez **Azure Active Directory**, puis sélectionnez **Réinitialisation de mot de passe** et choisissez **Intégration locale**.
1. Définissez l’option **Réécrire des mots de passe dans votre annuaire local ?** sur *Non*.
1. Définissez l’option **Voulez-vous autoriser les utilisateurs à déverrouiller les comptes sans réinitialiser leur mot de passe ?** sur *Non*.

Si vous ne souhaitez plus utiliser de fonctionnalités de mot de passe, effectuez les étapes suivantes à partir de votre serveur Azure AD Connect :

1. Connectez-vous à votre serveur Azure AD Connect et démarrez l’Assistant de configuration **Azure AD Connect**.
1. Sur la page d’**accueil**, sélectionnez **Configurer**.
1. Sur la page **Tâches supplémentaires**, sélectionnez **Personnalisation des options de synchronisation**, puis cliquez sur **Suivant**.
1. Dans la page **Connexion à Azure AD**, entrez les informations d’identification d’administrateur général pour votre locataire Azure et sélectionnez **Suivant**.
1. Sur les pages **Connexion des annuaires** et **Filtrage par domaine/unité d’organisation**, sélectionnez **Suivant**.
1. Dans la page **Fonctionnalités facultatives**, décochez la case située en regard de l’option **Réécriture du mot de passe**, puis sélectionnez **Suivant**.
1. Sur la page **Prêt à configurer**, sélectionnez **Configurer** et attendez la fin du processus.
1. Lorsque la configuration prend fin, sélectionnez **Quitter**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez activé la réécriture SSPR Azure AD dans un environnement AD DS local. Vous avez appris à :

> [!div class="checklist"]
> * Configurer les autorisations nécessaires pour la réécriture du mot de passe
> * Activer l’option de réécriture du mot de passe dans Azure AD Connect
> * Activer la réécriture du mot de passe dans SSPR Azure AD

> [!div class="nextstepaction"]
> [Évaluer les risques lors de la connexion](tutorial-risk-based-sspr-mfa.md)
