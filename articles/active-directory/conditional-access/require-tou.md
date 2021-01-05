---
title: 'Accès conditionnel : Exiger l’acceptation des conditions d’utilisation - Azure Active Directory'
description: Dans ce guide de démarrage rapide, vous découvrirez comment exiger que vos conditions d’utilisation soient acceptées avant d’accorder l’accès à une sélection d’applications cloud grâce à l’accès conditionnel Azure Active Directory.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: quickstart
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8ab484e8caaffaf57f19f1fcd1e65f4b8e723f86
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93077895"
---
# <a name="quickstart-require-terms-of-use-to-be-accepted-before-accessing-cloud-apps"></a>Démarrage rapide : Exiger l’acceptation des conditions d’utilisation avant d’accorder l’accès à des applications cloud

Si vous souhaitez obtenir le consentement des utilisateurs avant qu’ils ne puissent accéder à certaines applications cloud de votre environnement, vous pouvez demander à ce qu’ils acceptent vos conditions d’utilisation. L’accès conditionnel Azure Active Directory (Azure AD) vous offre les avantages suivants :

- Une méthode simple pour configurer les conditions d’utilisation
- La possibilité d’exiger l’acceptation de vos conditions d’utilisation à l’aide d’une stratégie d’accès conditionnel  

Ce démarrage rapide vous montre comment configurer une [stratégie d’accès conditionnel Azure AD](./overview.md) qui vous permet de demander aux utilisateurs d’accepter vos conditions d’utilisation pour accéder à une certaine application cloud de votre environnement.

:::image type="content" source="./media/require-tou/5555.png" alt-text="Capture d’écran du portail Azure. Un volet qui définit une stratégie nommée Exiger des conditions d’utilisation pour Isabella est visible." border="false":::

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour suivre le scénario décrit dans ce démarrage rapide, vous avez besoin de ce qui suit :

- **Accès à l’édition Azure AD Premium**  : l’accès conditionnel Azure AD est une fonctionnalité d’Azure AD Premium.
- **Un compte d’essai nommé Isabella Simonsen**  : si vous ignorez comment créer un compte d’essai, voir [Ajouter des utilisateurs basés sur le cloud](../fundamentals/add-users-azure-active-directory.md#add-a-new-user).

## <a name="test-your-sign-in"></a>Tester la connexion

L’objectif de cette étape consiste à obtenir une impression de l’expérience de connexion sans stratégie d’accès conditionnel.

**Pour tester la connexion :**

1. [Connectez-vous](https://portal.azure.com/) à votre portail Azure en tant que Isabella Simonsen.
1. Déconnectez-vous.

## <a name="create-your-terms-of-use"></a>Créer vos conditions d’utilisation

Cette section explique comment configurer des conditions d’utilisation. Lorsque vous créez des conditions d’utilisation, vous sélectionnez une valeur pour **Appliquer avec des modèles de stratégie d’accès conditionnel**. Sélectionnez **Stratégie personnalisée** pour ouvrir la boîte de dialogue permettant de créer une nouvelle stratégie d’accès conditionnel. Cette étape est possible dès que vos conditions d’utilisation sont créées.

**Pour créer vos conditions d’utilisation :**

1. Dans Microsoft Word, créez un nouveau document.
1. Saisissez **My terms of use** , puis enregistrez le document sur votre ordinateur sous le nom **mytou.pdf**.
1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Dans la barre de navigation gauche du portail Azure, cliquez sur **Azure Active Directory**.

   ![Azure Active Directory](./media/require-tou/02.png)

1. Sur la page **Azure Active Directory** , dans la section **Sécurité** , cliquez sur **Accès conditionnel**.

   ![Accès conditionnel](./media/require-tou/03.png)

1. Dans la section **Gérer** , cliquez sur **Conditions d’utilisation**.

   :::image type="content" source="./media/require-tou/04.png" alt-text="Capture d’écran de la section Gérer de la page Azure Active Directory. L’élément Conditions d’utilisation est mis en surbrillance." border="false":::

1. Dans le menu supérieur, cliquez sur **Nouvelles conditions**.

   :::image type="content" source="./media/require-tou/05.png" alt-text="Capture d’écran d’un menu dans la page Azure Active Directory. L’élément Nouvelles conditions est mis en évidence." border="false":::

1. Sur la page **Nouvelles conditions d’utilisation** :

   :::image type="content" source="./media/require-tou/112.png" alt-text="Capture d’écran de la page Nouvelles conditions d’utilisation mettant en évidence le nom, le nom complet, le document, la langue, l’accès conditionnel et la bascule d’extension des conditions d’utilisation." border="false":::

   1. Dans la zone de texte **Nom** , saisissez **My TOU**.
   1. Dans la zone de texte **Nom d’affichage** saisissez **My TOU**.
   1. Chargez vos conditions d’utilisation sous forme de fichier PDF.
   1. Dans la zone **langue** , sélectionnez **anglais**.
   1. Activez l’option permettant de **demander aux utilisateurs de développer les conditions d’utilisation** **.**
   1. Pour l’option **Appliquer avec des modèles de stratégie d’accès conditionnel** , sélectionnez **Stratégie personnalisée**.
   1. Cliquez sur **Créer**.

## <a name="create-your-conditional-access-policy"></a>Créer votre stratégie d’accès conditionnel

Cette section montre comment créer la stratégie d’accès conditionnel requise. Le scénario de ce démarrage rapide utilise ce qui suit :

- le portail Azure en tant qu’espace réservé pour une application cloud qui exige que vos conditions d’utilisation soient acceptées ; 
- Votre exemple d’utilisateur pour tester la stratégie d’accès conditionnel.  

Dans votre stratégie, définissez :

| Paramètre | Valeur |
| --- | --- |
| Utilisateurs et groupes | Isabella Simonsen |
| Applications cloud | Gestion Microsoft Azure |
| Accorder l'accès | My TOU |

:::image type="content" source="./media/require-tou/1234.png" alt-text="Capture d’écran d’un volet du portail Azure qui définit une stratégie. Les flèches indiquent que la stratégie accorde l’accès à Mes conditions d’utilisation et comprend un utilisateur et une application." border="false":::

**Pour configurer votre stratégie d’accès conditionnel, effectuez les étapes suivantes :**

1. Sur la page **Nouveau** , dans la zone de texte **Nom** , saisissez **Require TOU for Isabella**.

   ![Nom](./media/require-tou/71.png)

1. Dans la section **Affectation** , cliquez sur **Utilisateurs et groupes**.

   :::image type="content" source="./media/require-tou/06.png" alt-text="Capture d’écran de la section Affectations d’un volet du portail Azure qui définit une stratégie. L’élément Utilisateurs et groupes est visible, avec la valeur Aucun sélectionnée." border="false":::

1. Sur la page **Utilisateurs et groupes** :

   :::image type="content" source="./media/require-tou/24.png" alt-text="Capture d’écran de l’onglet Inclure de la page Utilisateurs et groupes. L’option Sélectionner des utilisateurs et des groupes est sélectionnée, de même que l’option Utilisateurs et groupes. Sélectionner est mis en surbrillance." border="false":::

   1. Cliquez sur **Sélectionner des utilisateurs et des groupes** , puis choisissez **des utilisateurs et des groupes**.
   1. Cliquez sur **Sélectionner**.
   1. Dans la page **Sélectionner** , sélectionnez **Isabella Simonsen** , puis cliquez sur **Sélectionner**.
   1. Dans la page **Utilisateurs et groupes** , cliquez sur **Terminé**.
1. Cliquez sur **Applications cloud**.

   :::image type="content" source="./media/require-tou/08.png" alt-text="Capture d’écran de la section Affectations d’un volet du portail Azure qui définit une stratégie. L’élément Applications cloud est visible, avec la valeur Aucun sélectionnée." border="false":::

1. Sur la page **Applications cloud** :

   ![Sélection des applications cloud](./media/require-tou/26.png)

   1. Cliquez sur **Sélectionner les applications**.
   1. Cliquez sur **Sélectionner**.
   1. Dans la page **Sélectionner** , choisissez **Gestion Microsoft Azure** , puis cliquez sur **Sélectionner**.
   1. Dans la page **Applications cloud** , cliquez sur **Terminé**.
1. Dans la section **Contrôles d’accès** , cliquez sur **Accorder**.

   ![Contrôles d’accès](./media/require-tou/10.png)

1. Sur la page des **octrois** :

   ![Accorder](./media/require-tou/111.png)

   1. Sélectionner **Accorder l’accès**.
   1. Sélectionnez **My TOU**.
   1. Cliquez sur **Sélectionner**.
1. Dans la section **Activer la stratégie** , cliquez sur **Activée**.

   ![Activer la stratégie](./media/require-tou/18.png)

1. Cliquez sur **Créer**.

## <a name="evaluate-a-simulated-sign-in"></a>Évaluer une connexion simulée

À présent que vous avez configuré votre stratégie d’accès conditionnel, vous souhaitez probablement savoir s’il fonctionne comme prévu. Dans un premier temps, utilisez l’outil de stratégie d’accès conditionnel What If pour simuler une connexion de votre utilisateur de test. La simulation évalue l’impact cette connexion sur vos stratégies et génère un rapport de simulation.  

Pour initialiser l’outil d’évaluation de stratégie **What If** , définissez ce qui suit :

- **Isabella Simonsen** en tant qu’utilisateur.
- **Gestion Microsoft Azure** en tant qu’application cloud.

Un clic sur **What If** a pour effet de créer un rapport de simulation indiquant ce qui suit :

- **Require TOU for Isabella** sous **Stratégies qui vont s’appliquer**
- **My TOU** en tant que **contrôles des octrois**.

![Outil de stratégie What If](./media/require-tou/79.png)

**Pour évaluer votre stratégie d’accès conditionnel :**

1. Dans la page [Accès conditionnel - Stratégies](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ConditionalAccessBlade/Policies), dans le menu en haut, cliquez sur **What If**.  

   ![What If](./media/require-tou/14.png)

1. Cliquez sur **Utilisateurs** , sélectionnez **Isabella Simonsen** , puis cliquez sur **Sélectionner**.

   ![Utilisateur](./media/require-tou/15.png)

1. Pour sélectionner une application cloud :

   :::image type="content" source="./media/require-tou/16.png" alt-text="Capture d’écran de la section Applications cloud. Le texte indique qu’une application est sélectionnée." border="false":::

   1. Cliquez sur **Applications cloud**.
   1. Dans la page **Applications cloud** , cliquez sur **Sélectionner les applications**.
   1. Cliquez sur **Sélectionner**.
   1. Dans la page **Sélectionner** , choisissez **Gestion Microsoft Azure** , puis cliquez sur **Sélectionner**.
   1. Dans la page Applications cloud, cliquez sur **Terminé**.
1. Cliquez sur **What If**.

## <a name="test-your-conditional-access-policy"></a>Tester votre stratégie d’accès conditionnel

Dans la section précédente, vous avez appris à évaluer une connexion simulée. En plus d’une simulation, vous devez tester votre stratégie d’accès conditionnel pour vous assurer qu’elle fonctionne comme prévu.

Pour tester votre stratégie, essayez de vous connecter à votre [portail Azure](https://portal.azure.com) à l’aide de votre compte de test **Isabella Simonsen**. Une boîte de dialogue doit s’afficher, exigeant que vous acceptiez vos conditions d’utilisation.

:::image type="content" source="./media/require-tou/57.png" alt-text="Capture d’écran d’une boîte de dialogue intitulée Conditions d’utilisation de la fonction de protection de la sécurité des identités, avec les boutons Refuser et Accepter et un bouton intitulé Mes conditions d’utilisation." border="false":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez l’utilisateur de test et la stratégie d’accès conditionnel :

- Si vous ignorez comment supprimer un utilisateur Azure AD, voir [Supprimer des utilisateurs d’Azure AD](../fundamentals/add-users-azure-active-directory.md#delete-a-user).
- Pour supprimer votre stratégie, sélectionnez-la, puis cliquez sur **Supprimer** dans la barre d’outils Accès rapide.

    :::image type="content" source="./media/require-tou/33.png" alt-text="Capture d’écran montrant une stratégie nommée Exiger l’authentification MFA pour l’accès au portail Azure. Le menu contextuel est visible, avec l’option Supprimer en évidence." border="false":::

- Pour supprimer vos conditions d’utilisation, sélectionnez-les, puis cliquez sur **Supprimer les conditions** dans la barre d’outils supérieure.

    :::image type="content" source="./media/require-tou/29.png" alt-text="Capture d’écran montrant une partie d’un tableau répertoriant les documents des conditions d’utilisation. Le document Mes conditions d’utilisation est visible. Dans le menu, l’option Supprimer les conditions est mise en évidence." border="false":::

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Exiger une authentification multifacteur pour certaines applications spécifiques](../authentication/tutorial-enable-azure-mfa.md)