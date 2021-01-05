---
title: Configurer des listes personnalisées pour la protection par mot de passe Azure Active Directory
description: Dans ce tutoriel, vous allez apprendre à configurer des listes personnalisées de mots de passe interdits pour Azure Active Directory afin de limiter les mots couramment utilisés au sein de votre environnement.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: tutorial
ms.date: 07/13/2020
ms.author: justinha
author: justinha
ms.reviewer: rogoya
ms.collection: M365-identity-device-management
ms.openlocfilehash: 89abdcf4c76c92c2ed967594527d323b2cd8be32
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741182"
---
# <a name="tutorial-configure-custom-banned-passwords-for-azure-active-directory-password-protection"></a>Tutoriel : Configurer des mots de passe interdits personnalisés pour la protection par mot de passe Azure Active Directory

Les utilisateurs créent souvent des mots de passe basés sur des mots locaux courants, par exemple une école, une équipe de sport ou une personne célèbre. Ces mots de passe sont faciles à deviner et offrent une faible protection contre les attaques par dictionnaire. Si vous souhaitez instaurer des mots de passe forts dans votre organisation, la liste personnalisée de mots de passe interdits Azure AD (Azure Active Directory) vous permet d’ajouter des chaînes spécifiques à évaluer et à bloquer. Toute demande de changement de mot de passe est refusée s’il existe une correspondance dans la liste personnalisée de mots de passe interdits.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Activer les mots de passe interdits personnalisés
> * Ajouter des entrées à la liste personnalisée de mots de passe interdits
> * Tester les changements de mot de passe avec un mot de passe interdit

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce tutoriel, vous avez besoin des ressources et des privilèges suivants :

* Un locataire Azure AD actif avec au moins un abonnement Azure AD Premium P1 ou une licence d’évaluation activée.
    * Si nécessaire, [créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un compte avec des privilèges d’*administrateur général*.
* Un utilisateur non-administrateur avec un mot de passe que vous connaissez, par exemple *testuser*. Vous devez utiliser ce compte pour tester un événement de changement de mot de passe dans ce tutoriel.
    * Si vous devez créer un utilisateur, consultez [Démarrage rapide : Ajouter de nouveaux utilisateurs à Azure Active Directory](../fundamentals/add-users-azure-active-directory.md).
    * Pour permettre le test de l’opération de changement de mot de passe à l’aide d’un mot de passe interdit, le locataire Azure AD doit être [configuré pour la réinitialisation de mot de passe en libre-service](tutorial-enable-sspr.md).

## <a name="what-are-banned-password-lists"></a>Que sont les listes de mots de passe interdits ?

Azure AD inclut une liste globale de mots de passe interdits. Le contenu de la liste globale de mots de passe interdits n’est basé sur aucune source de données externe. En fait, la liste globale de mots de passe interdits est basée sur les résultats en continu de l’analyse et de la télémétrie de sécurité Azure AD. Quand un utilisateur ou un administrateur tente de changer ou de réinitialiser ses informations d’identification, le mot de passe souhaité est comparé à la liste des mots de passe interdits. La demande de changement de mot de passe n’aboutit pas, s’il existe une correspondance dans la liste globale de mots de passe interdits. Vous ne pouvez pas modifier cette liste globale de mots de passe interdits par défaut.

Pour affiner votre choix des mots de passe autorisés, vous pouvez également définir une liste personnalisée de mots de passe interdits. La liste personnalisée de mots de passe interdits fonctionne parallèlement à la liste globale de mots de passe interdits pour permettre l’instauration de mots de passe forts dans votre organisation. Vous pouvez ajouter des termes spécifiques à l’organisation dans la liste personnalisée de mots de passe interdits. En voici quelques exemples :

* Noms de marques
* Noms de produits
* Lieux, par exemple le siège social de l’entreprise
* Termes internes spécifiques à l’entreprise
* Abréviations ayant une signification spécifique dans l’entreprise

Quand un utilisateur tente de réinitialiser un mot de passe en le remplaçant par un élément qui figure dans la liste globale ou personnalisée de mots de passe interdits, il voit s’afficher l’un des messages d’erreur suivants :

* *Malheureusement, votre mot de passe contient un mot, une expression ou un modèle qui le rend facile à deviner. Réessayez avec un autre mot de passe.*
* *Malheureusement, vous ne pouvez pas utiliser ce mot de passe, car il contient des mots ou des caractères qui ont été bloqués par votre administrateur. Réessayez avec un autre mot de passe.*

La liste personnalisée de mots de passe interdits est limitée à un maximum de 1 000 termes. Elle n’est pas conçue pour bloquer de longues listes de mots de passe. Pour optimiser les avantages de la liste personnalisée de mots de passe interdits, consultez les [concepts de la liste personnalisée de mots de passe interdits](concept-password-ban-bad.md#custom-banned-password-list) et la [vue d’ensemble de l’algorithme d’évaluation de mots de passe](concept-password-ban-bad.md#how-are-passwords-evaluated).

## <a name="configure-custom-banned-passwords"></a>Configurer les mots de passe interdits personnalisés

Activons la liste personnalisée de mots de passe interdits, et ajoutons quelques entrées. Vous pouvez ajouter des entrées à la liste personnalisée de mots de passe interdits à tout moment.

Pour activer la liste personnalisée de mots de passe interdits et y ajouter des entrées, effectuez ce qui suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte disposant d’autorisations d’*administrateur général*.
1. Recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Sécurité** dans le menu de gauche.
1. Sous le titre de menu **Gérer**, sélectionnez **Méthodes d’authentification**, puis **Protection par mot de passe**.
1. Affectez à l’option **Appliquer la liste personnalisée** la valeur *Oui*.
1. Ajoutez des chaînes à la **liste personnalisée de mots de passe interdits**, à raison d’une chaîne par ligne. Les considérations et limitations suivantes s’appliquent à la liste personnalisée de mots de passe interdits :

    * La liste de mots de passe interdits personnalisée peut contenir jusqu’à 1 000 termes.
    * La liste de mots de passe interdits personnalisée respecte la casse.
    * La liste personnalisée de mots de passe interdits prend en compte la substitution de caractères courante, par exemple « o » et « 0 », ou « a » et « \@ ».
    * La longueur de chaîne minimale est de quatre caractères, et la longueur maximale est de 16 caractères.

    Spécifiez vos propres mots de passe personnalisés à interdire, comme indiqué dans l’exemple suivant

    [ ![Modifier la liste de mots de passe interdits personnalisée sous Méthodes d’authentification sur le portail Azure](media/tutorial-configure-custom-password-protection/enable-configure-custom-banned-passwords-cropped.png) ](media/tutorial-configure-custom-password-protection/enable-configure-custom-banned-passwords.png#lightbox)

1. Gardez l’option **Activer la protection par mot de passe sur Windows Server Active Directory** avec la valeur *Non*.
1. Pour activer les mots de passe interdits personnalisés et vos entrées, sélectionnez **Enregistrer**.

L’application de la mise à jour de la liste de mots de passe interdits personnalisée peut prendre plusieurs heures.

Dans le cas d’un environnement hybride, vous pouvez également [déployer la protection par mot de passe Azure AD sur un environnement local](howto-password-ban-bad-on-premises-deploy.md). Les mêmes listes globales et personnalisées de mots de passe interdits sont utilisées pour les demandes de changement de mot de passe cloud et en local.

## <a name="test-custom-banned-password-list"></a>Tester la liste personnalisée de mots de passe interdits

Pour voir la liste personnalisée de mots de passe interdits en action, essayez de remplacer le mot de passe par une variante de celui que vous avez ajouté au cours de la section précédente. Quand Azure AD tente de traiter le changement de mot de passe, ce dernier est mis en correspondance avec une entrée de la liste personnalisée de mots de passe interdits. Dans ce cas, l’utilisateur voit s’afficher une erreur.

> [!NOTE]
> Pour qu’un utilisateur puisse réinitialiser son mot de passe dans le portail web, le locataire Azure AD doit être [configuré pour la réinitialisation de mot de passe en libre-service](tutorial-enable-sspr.md). Si nécessaire, l’utilisateur peut [s’inscrire à la réinitialisation de mot de passe en libre-service (SSPR)https://aka.ms/ssprsetup](https://aka.ms/ssprsetup).

1. Accédez à la page **Mes applications** sur [https://myapps.microsoft.com](https://myapps.microsoft.com).
1. En haut à droite, sélectionnez votre nom, puis choisissez **Profil** dans le menu déroulant.

    ![Sélectionner un profil](media/tutorial-configure-custom-password-protection/myapps-profile.png)

1. Dans la page **Profil**, sélectionnez **Changer le mot de passe**.
1. Dans la page **Changer le mot de passe**, entrez le mot de passe existant (ancien). Entrez et confirmez un nouveau mot de passe figurant dans la liste personnalisée de mots de passe interdits que vous avez définie dans la section précédente, puis sélectionnez **Soumettre**.
1. Un message d’erreur vous indique que le mot de passe a été bloqué par l’administrateur, comme indiqué dans l’exemple suivant :

    ![Message d’erreur qui s’affiche quand vous essayez d’utiliser un mot de passe faisant partie de la liste personnalisée de mots de passe interdits](media/tutorial-configure-custom-password-protection/password-change-error.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne souhaitez plus utiliser la liste personnalisée de mots de passe interdits que vous avez configurée dans le cadre de ce tutoriel, effectuez ce qui suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Sécurité** dans le menu de gauche.
1. Sous le titre de menu **Gérer**, sélectionnez **Méthodes d’authentification**, puis **Protection par mot de passe**.
1. Affectez à l’option **Appliquer la liste personnalisée** la valeur *Non*.
1. Pour mettre à jour la configuration personnalisée des mots de passe interdits, sélectionnez **Enregistrer**.

## <a name="next-steps"></a>Étapes suivantes

Au cours de ce tutoriel, vous avez activé et configuré des listes personnalisées de mots de passe interdits dans le cadre de la protection par mot de passe pour Azure AD. Vous avez appris à :

> [!div class="checklist"]
> * Activer les mots de passe interdits personnalisés
> * Ajouter des entrées à la liste personnalisée de mots de passe interdits
> * Tester les changements de mot de passe avec un mot de passe interdit

> [!div class="nextstepaction"]
> [Activer Azure AD Multi-Factor Authentication en fonction des risques](./tutorial-enable-azure-mfa.md)