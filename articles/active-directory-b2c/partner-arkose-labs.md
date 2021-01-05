---
title: Arkose Labs avec Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Découvrez comment intégrer l’authentification Azure AD B2C avec Arkose Labs pour vous protéger contre les attaques de bot, les attaques de prise de contrôle de compte et les ouvertures de comptes frauduleuses.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 06/08/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 2c7eea87101a36edb0d77026489ea351b601158b
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97584593"
---
# <a name="tutorial-for-configuring-arkose-labs-with-azure-active-directory-b2c"></a>Tutoriel pour la configuration d’Arkose Labs avec Azure Active Directory B2C

Dans ce tutoriel, découvrez comment intégrer l’authentification Azure AD B2C avec Arkose Labs. Arkose Labs aide les organisations à lutter contre les attaques de bot, les attaques de prise de contrôle de compte et les ouvertures de comptes frauduleuses.  

## <a name="prerequisites"></a>Prérequis

Pour commencer, vous avez besoin des éléments suivants :

* Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).
* [Un locataire Azure AD B2C](tutorial-create-tenant.md) qui est lié à votre abonnement Azure.

## <a name="scenario-description"></a>Description du scénario

Le diagramme suivant décrit comment Arkose Labs s’intègre à Azure AD B2C.

![Diagramme montrant l’architecture d’Arkose Labs](media/partner-arkose-labs/arkose-architecture-diagram.png)

| Étape  | Description |
|---|---|
|1     | Un utilisateur se connecte avec un compte créé précédemment. Quand l’utilisateur sélectionne Envoyer, un défi Arkose Labs Enforcement s’affiche. Une fois que l’utilisateur a terminé le défi, l’état est envoyé à Arkose Labs pour générer un jeton.        |
|2     |  Arkose Labs renvoie le jeton à Azure AD B2C.       |
|3     |  Avant l’envoi du formulaire de connexion, le jeton est envoyé à Arkose Labs à des fins de vérification.       |
|4     |  Arkose renvoie un résultat de réussite ou d’échec relatif au défi.       |
|5     |  Si le défi est réussi, un formulaire de connexion est envoyé à Azure AD B2C qui termine l’authentification.       |
|   |   |

## <a name="onboard-with-arkose-labs"></a>Intégration avec Arkose Labs

1. Commencez par contacter [Arkose Labs](https://www.arkoselabs.com/book-a-demo/) et créer un compte.

2. Une fois votre compte créé, accédez à https://dashboard.arkoselabs.com/login.

3. Dans le tableau de bord, accédez aux paramètres du site pour rechercher votre clé publique et votre clé privée. Ces informations seront nécessaires ultérieurement pour configurer Azure AD B2C.

## <a name="integrate-with-azure-ad-b2c"></a>Intégrer avec Azure AD B2C

### <a name="part-1--create-blob-storage-to-store-the-custom-html"></a>Partie 1 – Créer un stockage Blob pour stocker le code HTML personnalisé

Pour créer un compte de stockage, procédez comme suit :  

1. Connectez-vous au portail Azure.

2. Veillez à bien utiliser l’annuaire qui contient votre abonnement Azure. Sélectionnez le filtre  **Annuaire + abonnement** dans le menu supérieur et sélectionnez l’annuaire qui contient votre abonnement. Cet annuaire est différent de celui qui contient votre locataire Azure B2C.

3. Choisissez **Tous les services** dans le coin supérieur gauche du portail Azure, puis recherchez et sélectionnez  **Comptes de stockage**.

4. Sélectionnez **Ajouter**.

5. Sous  **Groupe de ressources**, sélectionnez  **Créer**, entrez un nom pour le nouveau groupe de ressources, puis sélectionnez **OK**.

6. Nommez le compte de stockage. Le nom que vous choisissez doit être unique dans Azure, et contenir entre 3 et 24 caractères, uniquement des lettres minuscules et des chiffres.

7. Sélectionnez l’emplacement du compte de stockage ou acceptez l’emplacement par défaut.

8. Acceptez toutes les autres valeurs par défaut, sélectionnez   **Vérifier + créer** > **Créer**.

9. Une fois le compte de stockage créé, sélectionnez  **Accéder à la ressource**.

#### <a name="create-a-container"></a>Créez un conteneur.

1. Dans la page de vue d’ensemble du compte de stockage, sélectionnez   **Blobs**.

2. Sélectionnez   **Conteneur**, entrez un nom pour le conteneur, choisissez   **Blob (accès en lecture anonyme pour les objets blob uniquement)** , puis sélectionnez **OK**.

#### <a name="enable-cross-origin-resource-sharing-cors"></a>Activer CORS (Partage des ressources cross-origin)

Le code Azure AD B2C dans un navigateur utilise une approche moderne et standard pour charger le contenu personnalisé à partir d’une URL que vous spécifiez dans un flux d’utilisateur. CORS permet de demander des ressources restreintes dans une page web à partir d’autres domaines.

1. Dans le menu, sélectionnez   **CORS**.

2. Dans   **Origines autorisées**, entrez  `https://your-tenant-name.b2clogin.com`. Remplacez « your-tenant-name » par le nom de votre locataire Azure AD B2C. Par exemple :  `https://fabrikam.b2clogin.com`. Utilisez des minuscules quand vous entrez le nom de votre locataire.

3. Pour  **Méthodes autorisées**, sélectionnez  **GET**, **PUT** et  **OPTIONS**.

4. Pour **En-têtes autorisés**, saisissez un astérisque (*).

5. Pour  **En-têtes exposés**, saisissez un astérisque (*).

6. Pour **Âge maximal**, tapez 200.

   ![Inscription et connexion à Arkose Labs](media/partner-arkose-labs/signup-signin-arkose.png)

7. Sélectionnez **Enregistrer**.

### <a name="part-2--set-up-a-back-end-server"></a>Partie 2 – Configurer un serveur back-end

Téléchargez Git Bash et procédez comme suit :

1. Suivez les instructions pour [créer une application web](../app-service/quickstart-php.md) jusqu’au message « Félicitations ! Vous avez déployé votre première application en PHP sur App Service ».

2. Ouvrez votre dossier local et renommez le fichier **index.php** en **verify-token.php**.

3. Ouvrez le fichier verify-token.php récemment renommé et :

   a. Remplacez le contenu par le contenu du fichier verify-token.php figurant dans le [référentiel GitHub](https://github.com/ArkoseLabs/Azure-AD-B2C).

   b. Remplacez <private_key> à la ligne 3 par votre clé privée obtenue à partir du tableau de bord Arkose Labs.

4. Dans la fenêtre du terminal local, validez vos modifications dans Git, puis envoyez les modifications de code à Azure en tapant ce qui suit dans Git Bash :

   ``git commit -am "updated output"``

   ``git push azure main``  

### <a name="part-3---final-setup"></a>Partie 3 –  Configuration finale

#### <a name="store-the-custom-html"></a>Stocker le code HTML personnalisé

1. Ouvrez le fichier index.html stocké dans le [référentiel GitHub](https://github.com/ArkoseLabs/Azure-AD-B2C).

2. Remplacez toutes les instances de `<tenantname>` par le nom de votre locataire b2C (en d’autres termes, `<tenantname>.b2clogin.com`). Il doit y avoir quatre instances.

3. Remplacez `<appname>` par le nom de l’application que vous avez créée dans la Partie 2, étape 1.

   ![Capture d’écran montrant l’interface Azure CLI Arkose Labs](media/partner-arkose-labs/arkose-azure-cli.png)

4. Remplacez `<public_key>` à la ligne 64 par la clé publique que vous avez obtenue à partir du tableau de bord Arkose Labs.

5. Chargez le fichier index.html dans le stockage Blob créé ci-dessus.

6. Accédez à **Stockage** > **Conteneur** > **Charger**.

#### <a name="set-up-azure-ad-b2c"></a>Configurer Azure AD B2C

> [!NOTE]
> Si vous n’en avez pas, [créez un locataire Azure AD B2C](tutorial-create-tenant.md) qui est lié à votre abonnement Azure.

1. Créez un flux utilisateur basé sur [ces](tutorial-create-user-flows.md) informations. Arrêtez quand vous atteignez la section **Tester le flux utilisateur**.

2. Activez JavaScript dans votre [flux utilisateur](javascript-and-page-layout.md).

3. Dans la même page du flux utilisateur, activez l’URL de page personnalisée : Accédez à **Flux utilisateur** > **mise en page** > **utiliser un contenu de page personnalisée** = **oui** > **insérer une URL de page personnalisée**.
Cette URL de page personnalisée est obtenue à partir de l’emplacement du fichier index.html dans le stockage Blob  

   ![Capture d’écran montrant l’URL du stockage Arkose Labs](media/partner-arkose-labs/arkose-storage-url.png)

## <a name="test-the-user-flow"></a>Tester le flux utilisateur

1. Ouvrez le locataire Azure AD B2C, puis sous **Stratégies**, sélectionnez **Flux utilisateur**.

2. Sélectionnez votre flux utilisateur précédemment créé.

3. Sélectionnez **Exécuter le flux utilisateur** puis sélectionnez les paramètres :

   a. **Application** : sélectionnez l’application enregistrée (l’exemple concerne JWT).

   b. **URL de réponse** : sélectionnez l’URL de redirection.

   c. Sélectionnez **Exécuter le flux utilisateur**.

4. Suivez le processus d’inscription et créez un compte.

5. Déconnectez-vous.

6. Suivez le processus de connexion.

7. Un puzzle Arkose Labs s’affiche lorsque vous sélectionnez **continuer**.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](custom-policy-get-started.md?tabs=applications)