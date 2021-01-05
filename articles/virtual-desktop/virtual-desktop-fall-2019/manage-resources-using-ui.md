---
title: Déployer un outil de gestion avec un modèle Azure Resource Manager - Azure
description: Comment installer un outil d’interface utilisateur à l’aide d’un modèle Azure Resource Manager pour gérer les ressources de Windows Virtual Desktop (classique).
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: f96365bcdf64d19dc0b894f2f1230233b3137bc7
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842649"
---
# <a name="deploy-a-windows-virtual-desktop-classic-management-tool-with-an-azure-resource-manager-template"></a>Déployer un outil de gestion Windows Virtual Desktop (classique) à l’aide d’un modèle Azure Resource Manager

>[!IMPORTANT]
>Ce contenu s’applique à Windows Virtual Desktop (classique), qui ne prend pas en charge les objets Windows Virtual Desktop Azure Resource Manager.

Les instructions de cet article vous indiquent comme déployer l’interface utilisateur avec un modèle Azure Resource Manager.

## <a name="important-considerations"></a>Points importants à prendre en compte

Étant donné que l’application nécessite un consentement pour interagir avec Windows Virtual Desktop, cet outil ne prend pas en charge les scénarios B2B (Business-to-Business). Chaque abonnement du locataire Azure Active Directory (AAD) nécessite son propre déploiement de l’outil de gestion.

Cet outil de gestion est un exemple. Microsoft fournira des mises à jour importantes pour la qualité et la sécurité. Le [code source est disponible sur GitHub](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy). Les clients et les partenaires sont encouragés à personnaliser l’outil pour l’adapter à leurs besoins métier.

Les navigateurs suivants sont compatibles avec l’outil de gestion :
- Google Chrome version 68 ou ultérieure
- Microsoft Edge version 40.15063 ou ultérieure
- Mozilla Firefox version 52.0 ou ultérieure
- Safari version 10 ou ultérieure (macOS uniquement)

## <a name="what-you-need-to-deploy-the-management-tool"></a>Ce dont vous avez besoin pour déployer l’outil de gestion

Avant de déployer l’outil de gestion, vous avez besoin d’un utilisateur Azure Active Directory (Azure AD) pour créer une inscription d’application et déployer l’interface utilisateur de gestion. Cet utilisateur doit remplir les conditions suivantes :

- Azure AD Multi-Factor Authentication (MFA) doit être désactivé.
- Il doit avoir l’autorisation de créer des ressources dans votre abonnement Azure.
- Il doit avoir l’autorisation de créer une application Azure AD. Suivez ces étapes pour vérifier si votre utilisateur dispose des autorisations nécessaires en suivant les instructions fournies dans [Autorisations nécessaires](../../active-directory/develop/howto-create-service-principal-portal.md#permissions-required-for-registering-an-app).

Une fois que vous avez déployé et configuré l’outil de gestion, nous vous recommandons de demander à un utilisateur de lancer l’interface utilisateur de gestion pour vérifier que tout fonctionne correctement. L’utilisateur qui lance l’interface utilisateur de gestion doit avoir une attribution de rôle qui lui permet de voir ou de modifier le locataire Windows Virtual Desktop.

## <a name="deploy-the-management-tool"></a>Déployer l’outil de gestion

Avant de commencer, vérifiez que les applications serveur et client ont le consentement nécessaire en vous rendant sur la [page de consentement de Windows Virtual Desktop](https://rdweb.wvd.microsoft.com) pour l’annuaire Azure Active Directory (AAD) représenté.

Suivez ces instructions pour déployer le modèle Azure Resource Management :

1. Accédez à la [page Azure RDS-Templates de GitHub ](https://github.com/Azure/RDS-Templates/tree/master/wvd-templates/wvd-management-ux/deploy).
2. Déployez le modèle sur Azure.
    - Si vous déployez sur un abonnement Entreprise, faites défiler vers le bas et sélectionnez **Déployer sur Azure**.
    - Si vous déployez sur un abonnement de fournisseur de solutions cloud, suivez ces instructions pour déployer sur Azure :
        1. Faites défiler vers le bas et cliquez sur **Déployer sur Azure**, puis sélectionnez **Copier l’adresse du lien**.
        2. Ouvrez un éditeur de texte tel que le bloc-notes et collez le lien.
        3. Juste après <https://portal.azure.com/> et avant le dièse (#), entrez un arobase (@) suivi du nom de domaine du locataire. Voici un exemple du format : <https://portal.azure.com/@Contoso.onmicrosoft.com#create/>.
        4. Connectez-vous au portail Azure en tant qu’utilisateur avec des autorisations d’administrateur/contributeur pour l’abonnement du fournisseur de solutions cloud.
        5. Collez le lien que vous avez copié dans l’éditeur de texte dans la barre d’adresse.
3. Quand vous entrez les paramètres, procédez comme suit :
    - Pour le paramètre **isServicePrincipal**, sélectionnez **false**.
    - Pour les informations d’identification, entrez vos informations d’identification Azure AD avec l’authentification multifacteur désactivée. Ces informations d’identification seront utilisées pour créer l’application Azure AD et les ressources Azure. Pour plus d’informations, consultez [Ce dont vous avez besoin pour déployer l’outil de gestion](#what-you-need-to-deploy-the-management-tool).
    - Pour **applicationName**, utilisez un nom unique pour l’application qui sera inscrite dans votre annuaire Azure Active Directory. Ce nom sera également utilisé pour l’URL de l’application web. Par exemple, vous pouvez utiliser un nom comme « Apr3UX ».
4. Après avoir spécifié les paramètres, acceptez les conditions générales, puis sélectionnez **Acheter**.

## <a name="provide-consent-for-the-management-tool"></a>Donner son consentement pour l’outil de gestion

Après avoir exécuté le modèle Azure Resource Manager de GitHub, vous trouverez un groupe de ressources contenant deux services d’application ainsi qu’un plan App Service dans le portail Azure.

Avant de vous connecter et d’utiliser l’outil de gestion, vous devez donner un consentement pour la nouvelle application Azure AD qui est associée à l’outil de gestion. En fournissant ce consentement, vous autorisez l’outil de gestion à effectuer des appels de gestion de Windows Virtual Desktop pour le compte de l’utilisateur qui est actuellement connecté à l’outil.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant les autorisations fournies quand vous donnez un consentement à l’outil de gestion de l’interface utilisateur.](../media/management-ui-delegated-permissions.png)

Pour déterminer quel utilisateur vous pouvez utiliser pour vous connecter à l’outil, accédez à votre [page des paramètres utilisateur d’Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/) et prenez note de la valeur de **Les utilisateurs peuvent autoriser les applications à accéder aux données de l’entreprise en leur nom**.

> [!div class="mx-imgBorder"]
> ![Capture d’écran montrant si les utilisateurs peuvent accorder leur consentement à des applications seulement pour leur utilisateur.](../media/management-ui-user-consent-allowed.png)

- Si la valeur est définie sur **Oui**, vous pouvez vous connecter avec n’importe quel compte d’utilisateur de l’annuaire Azure Active Directory et donner un consentement seulement pour cet utilisateur. Cependant, si vous vous connectez ultérieurement à l’outil de gestion avec un autre compte d’utilisateur, vous devez donner à nouveau le même consentement.
- Si la valeur est définie sur **Non**, vous devez vous connecter en tant qu’administrateur général dans l’annuaire Azure Active Directory et donner le consentement de l’administrateur pour tous les utilisateurs de l’annuaire. Aucun autre utilisateur ne recevra une invite de consentement.


Une fois que vous avez décidé quel utilisateur vous allez utiliser pour donner le consentement, suivez ces instructions pour donner ce consentement à l’outil :

1. Accédez à vos ressources Azure, sélectionnez la ressource Azure App Services avec le nom fourni dans le modèle (par exemple Apr3UX), puis accédez à l’URL associée, par exemple `https://rdmimgmtweb-210520190304.azurewebsites.net`.
2. Connectez-vous en utilisant le compte d’utilisateur Azure Active Directory approprié.
3. Si vous vous êtes authentifié avec un administrateur général, vous pouvez maintenant cochez la case **Consentement pour le compte de votre organisation**. Sélectionnez **Accepter** pour donner votre consentement.

   > [!div class="mx-imgBorder"]
   > ![Capture d’écran montrant la page de consentement complète que l’utilisateur ou l’administrateur voit.](../media/management-ui-consent-page.png)

Vous accédez maintenant à l’outil de gestion.

## <a name="use-the-management-tool"></a>Utiliser l’outil de gestion

Après avoir donné un consentement pour l’organisation ou pour un utilisateur spécifié, vous pouvez accéder à tout moment à l’outil de gestion.

Suivez ces instructions pour lancer l’outil :

1. Sélectionnez la ressource Azure App Services avec le nom fourni dans le modèle (par exemple Apr3UX), puis accédez à l’URL associée, par exemple `https://rdmimgmtweb-210520190304.azurewebsites.net`.
2. Connectez-vous avec vos informations d’identification Windows Virtual Desktop.
3. Quand vous êtes invité à choisir un groupe de locataires, sélectionnez **Groupe de locataires par défaut** dans la liste déroulante.
4. Quand vous sélectionnez **Groupe de locataires par défaut**, un menu doit s’afficher du côté gauche de votre fenêtre. Dans ce menu, recherchez le nom de votre groupe de locataires et sélectionnez-le.

  > [!NOTE]
  > Si vous avez un groupe de locataires personnalisé, entrez le nom manuellement au lieu de choisir dans la liste déroulante.

## <a name="report-issues"></a>Signaler des problèmes

Si vous rencontrez des problèmes avec l’outil de gestion ou d’autres outils Windows Virtual Desktop, suivez les instructions de [Modèles Azure Resource Manager pour les services Bureau à distance](https://github.com/Azure/RDS-Templates/blob/master/README.md) pour les signaler sur GitHub.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment déployer et vous connecter à l’outil de gestion, vous pouvez découvrir comment utiliser l’aide d’Azure Service pour superviser les problèmes des services et les alertes d’intégrité. Pour plus d’informations, consultez notre [Tutoriel de configuration des alertes de service](set-up-service-alerts-2019.md).