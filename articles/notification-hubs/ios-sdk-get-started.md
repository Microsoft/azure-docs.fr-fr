---
title: Envoyer des notifications Push vers iOS avec Azure Notification Hubs et le SDK iOS
description: Dans ce tutoriel, vous allez découvrir comment utiliser Azure Notification Hubs et Apple Push Notification Service pour envoyer des notifications Push à des appareils iOS.
author: sethmanheim
ms.author: sethm
ms.date: 10/30/2020
ms.topic: tutorial
ms.service: notification-hubs
ms.reviewer: thsomasu
ms.lastreviewed: 06/01/2020
ms.openlocfilehash: c920c9b3b28df7f5bf3bf169ef88ab967f23649e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001318"
---
# <a name="tutorial-send-push-notifications-to-ios-apps-using-azure-notification-hubs"></a>Tutoriel : Envoyer des notifications Push vers des applications iOS avec Azure Notification Hubs

Ce tutoriel montre comment configurer Azure Notification Hubs et des informations d’identification pour envoyer des notifications Push à un appareil iOS par le biais d’[Apple Push Notification Service (APNS)](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1). 

L’exécution de ce tutoriel est un prérequis pour les tutoriels iOS Swift et Objective C suivants. Il décrit les étapes ci-dessous :

- Génération du fichier de demande de signature de certificat
- Inscription de votre application pour les notifications Push
- Création d’un profil de mise en service pour l’application
- Création d’un hub de notification
- Configuration du hub de notification avec des informations APNS

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin d'un compte Azure actif. Si vous ne possédez pas de compte, vous pouvez créer un compte d’évaluation gratuit en quelques minutes. Pour plus d’informations, consultez [Essai gratuit Azure](https://azure.microsoft.com/free/).

Vous devez également disposer des éléments suivants :

- Un compte de [développeur Apple](https://developer.apple.com/) actif
- Un Mac exécutant [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), ainsi qu’un certificat de développeur valide installé dans votre trousseau
- Un iPhone ou un iPad exécutant iOS version 10 ou ultérieure
- Votre appareil physique inscrit auprès du [portail Apple](https://developer.apple.com/) et associé à votre certificat

Veillez à lire la [vue d’ensemble d’Azure Notification Hubs](notification-hubs-push-notification-overview.md) si vous n’êtes pas familiarisé avec le service.

> [!NOTE]
> Le hub de notification sera configuré pour utiliser le mode d’authentification Bac à sable uniquement. Vous ne devez pas utiliser ce mode d’authentification pour les charges de travail de production.

## <a name="generate-the-certificate-signing-request-file"></a>Générer le fichier de demande de signature de certificat

Le service de notification Push Apple (APNS) utilise des certificats pour authentifier vos notifications Push. Suivez ces instructions pour créer le certificat push nécessaire pour envoyer et recevoir des notifications. Pour plus d’informations sur ces concepts, consultez la documentation officielle [d’Apple Push Notification Service](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html).

Générez le fichier de demande de signature de certificat (CSR, Certificate Signing Request) utilisé par Apple pour générer un certificat Push signé :

1. Sur votre Mac, exécutez l’outil Trousseaux d’accès. Vous pouvez l’ouvrir à partir du dossier **Utilitaires** ou du dossier **Autre** sur Launchpad.

2. Sélectionnez **Trousseaux d’accès**, développez **Assistant de certification**, puis cliquez sur **Demander un certificat à une autorité de certification**.

   :::image type="content" source="media/ios-sdk-get-started/image1.png" alt-text="Capture d’écran qui met en évidence l’option de menu Request a Certificate from a Certificate Authority.":::

   > [!NOTE]
   > Par défaut, Trousseaux d’accès sélectionne le premier élément de la liste. Cela peut poser problème si vous êtes dans la catégorie **Certificats** et que **Apple Worldwide Developer Relations Certification Authority** (Autorité de certification des relations des développeurs dans le monde entier) ne figure pas comme premier élément dans la liste. Veillez à disposer d’un élément non-clé, ou que la clé **Apple Worldwide Developer Relations Certification Authority** est sélectionnée, avant de générer la demande de signature de certificat (CSR).

3. Sélectionnez votre **adresse e-mail d’utilisateur**, entrez votre **nom commun**, veillez à spécifier **Enregistré sur le disque**, puis sélectionnez **Continuer**. Laissez le champ **Adresse de messagerie d’autorité de certification** vide, car il n’est pas requis.

   :::image type="content" source="media/ios-sdk-get-started/image2.png" alt-text="Informations requises sur le certificat":::

4. Entrez un nom pour le fichier CSR dans **Enregistrer en tant que**, sélectionnez l’emplacement dans **Où**, puis sélectionnez **Enregistrer**.

   :::image type="content" source="media/ios-sdk-get-started/image3.png" alt-text="Choisir le nom de fichier":::

   Cette action enregistre le fichier CSR à l’emplacement sélectionné. L’emplacement par défaut est **Bureau**. Notez l’emplacement choisi pour ce fichier.

Ensuite, inscrivez votre application auprès d’Apple, activez les notifications Push, puis téléchargez le fichier de demande de signature de certificat exporté pour créer un certificat Push.

## <a name="register-your-app-for-push-notifications"></a>Inscription de votre application pour les notifications Push

Pour envoyer des notifications Push vers une application iOS, inscrivez votre application auprès d’Apple ainsi que pour les notifications Push.

1. Si vous n’avez pas encore inscrit votre application, accédez au [portail de provisionnement iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456) dans le Centre pour développeurs Apple. Connectez-vous au portail avec votre ID Apple, puis sélectionnez **Identifiants**. Sélectionnez ensuite **+** pour inscrire une nouvelle application.

   :::image type="content" source="media/ios-sdk-get-started/image4.png" alt-text="Page App IDs":::

2. Dans l’écran **Register a New Identifier** (Inscrire un nouvel identifiant), sélectionnez la case d’option **App IDs** (ID d’application). Sélectionnez **Continuer**.

   :::image type="content" source="media/ios-sdk-get-started/image5.png" alt-text="Page Register new ID":::

3. Mettez à jour les trois valeurs suivantes pour votre nouvelle application, puis sélectionnez **Continue** :

   - **Description** : tapez un nom descriptif pour votre application.
   - **ID de l’offre groupée** : entrez un identifiant de bundle au format **Organization Identifier.Product Name** comme indiqué dans le [Guide de distribution d’application](https://help.apple.com/xcode/mac/current/#/dev91fe7130a). Les valeurs **Organization Identifier** et **Product Name** doivent correspondre à l’identificateur d’organisation et au nom de produit que vous utiliserez pour créer le projet Xcode. Dans la capture d’écran ci-dessous, la valeur **NotificationHubs** est utilisée comme identificateur d’organisation, tandis que la valeur **GetStarted** correspond au nom du produit. Vérifiez que la valeur **Bundle Identifier** correspond à celle de votre projet Xcode, afin que Xcode utilise le profil de publication correct.

      :::image type="content" source="media/ios-sdk-get-started/image6.png" alt-text="Inscrire un ID d’application":::

   - **Notifications Push** : cochez l’option **Notifications Push** dans la section **Capabilities** (Fonctionnalités).

      :::image type="content" source="media/ios-sdk-get-started/image7.png" alt-text="Inscrire un nouvel ID d’application":::

      L’ID de votre application est généré et vous êtes invité à confirmer les informations. Sélectionnez **Continue** (Continuer), puis **Register** (Inscrire) pour confirmer le nouvel ID d’application.

      :::image type="content" source="media/ios-sdk-get-started/image8.png" alt-text="Confirmer le nouvel ID d’application":::

      Après avoir sélectionné **Register** (Inscrire), vous voyez le nouvel ID d’application affiché comme élément de ligne dans la page **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils).

4. Dans la page **Certificates, Identifiers & Profiles**, sous **Identifiers** (Identifiants), recherchez l’élément de ligne ID d’application que vous venez de créer, puis sélectionnez sa ligne pour afficher l’écran **Edit your App ID Configuration** (Modifier votre configuration d’ID d’application).

## <a name="create-a-certificate-for-notification-hubs"></a>Créer un certificat pour Notification Hubs

> [!NOTE]
> Avec la version d’iOS 13, vous ne pouvez recevoir que des notifications en mode silencieux à l’aide de l’authentification basée sur un jeton. Si vous utilisez l’authentification basée sur les certificats pour vos informations d’identification APNS, vous devez passer à l’utilisation de l’authentification basée sur les jetons.

Un certificat est nécessaire pour permettre au hub de notification de fonctionner avec **APNS**. Cette opération peut être effectuée de deux manières :

- Créez un fichier **.p12** qui peut être chargé directement sur Notification Hubs.

- Créez un certificat **.p8** qui peut être utilisé pour l’[authentification basée sur un jeton](notification-hubs-push-notification-http2-token-authentication.md) (l’approche la plus récente).

La deuxième option présente un certain nombre d’avantages par rapport à l’utilisation de certificats, comme indiqué dans[Authentification basée sur un jeton (HTTP/2) pour APNS](notification-hubs-push-notification-http2-token-authentication.md). Toutefois, des étapes sont pour les deux approches.

### <a name="option-1-create-a-p12-push-certificate-that-can-be-uploaded-directly-to-notification-hubs"></a>Option 1 : Créer un certificat Push .p12 qui peut être chargé directement sur Notification Hubs

1. Faites défiler la page jusqu’à l’option cochée **Push Notifications** (Notifications Push), puis sélectionnez **Configure** (Configurer) pour créer le certificat.

   :::image type="content" source="media/ios-sdk-get-started/image9.png" alt-text="ID d’application":::

2. La fenêtre **Apple Push Notification service SSL Certificates** (Certificats SSL du service Apple Push Notification) s’affiche. Sélectionnez le bouton **Create Certificate** dans la section **Development SSL Certificate**.

   :::image type="content" source="media/ios-sdk-get-started/image10.png" alt-text="Créer un certificat":::

   L’écran **Create a new Certificate** (Créer un certificat) s’affiche.

   > [!NOTE]
   > Ce didacticiel utilise un certificat de développement. Le même processus est utilisé lors de l’inscription d’un certificat de production. Vérifiez que vous utilisez le même type de certificat que lorsque vous envoyez des notifications.

3. Sélectionnez **Choose File**, accédez à l’emplacement où vous avez enregistré le fichier de demande de signature de certificat créé lors de la première tâche, puis double-cliquez sur le nom du certificat pour le charger. Sélectionnez **Continuer**.

4. Une fois que le portail a créé un certificat, sélectionnez le bouton **Télécharger**. Enregistrez le certificat et notez son emplacement pour le retrouver facilement.

   :::image type="content" source="media/ios-sdk-get-started/image11.png" alt-text="Télécharger le certificat":::

   Le certificat est téléchargé et enregistré dans votre dossier **Downloads**.

   :::image type="content" source="media/ios-sdk-get-started/image12.png" alt-text="Rechercher le fichier de certificat":::

   Par défaut, le certificat de développement téléchargé se nomme **aps_development.cer**.

5. Double-cliquez sur le certificat Push téléchargé **aps\_development.cer**. Cette opération installe le nouveau certificat dans le Trousseau, comme indiqué dans l’image suivante :

   :::image type="content" source="media/ios-sdk-get-started/image13.png" alt-text="Trousseaux d’accès":::

   Bien que le nom de votre certificat puisse être différent, il portera toutefois le préfixe **Apple Development iOS Push Notification Services**.

6. Dans Trousseau d’accès, cliquez avec le bouton droit sur le certificat push que vous avez créé dans la catégorie **Certificats** . Sélectionnez **Exporter**, nommez le fichier, sélectionnez le format **.p12**, puis sélectionnez **Enregistrer**.

   :::image type="content" source="media/ios-sdk-get-started/image14.png" alt-text="Exporter le certificat":::

   Vous pouvez choisir de protéger le certificat par un mot de passe, mais cela est facultatif. Cliquez sur **OK** si vous souhaitez ignorer l’étape de création du mot de passe. Notez le nom du fichier et l’emplacement du certificat .p12 exporté. Ces informations sont nécessaires pour activer l’authentification avec APNS.

   > [!NOTE]
   > Le nom et l’emplacement de votre fichier .p12 peuvent être différents de ceux montrés dans ce tutoriel.

### <a name="option-2-create-a-p8-certificate-that-can-be-used-for-token-based-authentication"></a>Option n°2 : Créer un certificat .p8 qui peut être utilisé pour l’authentification basée sur un jeton

1. Prenez note des détails suivants :

   - **Préfixe d’ID d’application** (il s’agit d’un **ID d’équipe**)
   - **ID de bundle**

2. De retour dans **Certificats, identificateurs et profils**, cliquez sur **Clés**. Si vous disposez déjà d’une clé configurée pour **APNS**, vous pouvez réutiliser le certificat .p8 que vous avez téléchargé juste après l’avoir créée. Dans ce cas, vous pouvez ignorer les étapes 3 à 5.

3. Cliquez sur le bouton **+** (ou sur le bouton **Créer une clé**) pour créer une clé.

4. Fournissez une valeur appropriée pour **Nom de clé**, activez l’option **Apple Push Notification Service (APNS)** , puis cliquez sur **Continuer**, puis sur **Inscrire** dans l’écran suivant.

5. Cliquez sur **Télécharger**, puis déplacez le fichier **.p8** (préfixé avec `AuthKey_` ) vers un répertoire local sécurisé, puis cliquez sur **Terminé**.

   > [!IMPORTANT]
   > Veillez à conserver votre fichier .p8 dans un emplacement sécurisé (et à enregistrer une sauvegarde). Une fois votre clé téléchargée, elle ne peut pas être retéléchargée ; la copie du serveur est supprimée.

6. Dans **Clés**, cliquez sur la clé que vous venez de créer (ou une clé existante si vous avez choisi de l’utiliser à la place).

7. Notez la valeur de l’**ID de clé**.

8. Ouvrez votre certificat .p8 dans l’application appropriée de votre choix, par exemple [Visual Studio Code](https://code.visualstudio.com/), puis notez la valeur de la clé. Il s’agit de la valeur comprise entre **-----BEGIN PRIVATE KEY-----** et **-----END PRIVATE KEY-----** .

   ```p8
   -----BEGIN PRIVATE KEY-----
   <key_value>
   -----END PRIVATE KEY-----
   ```

   Il s’agit de la valeur de jeton qui sera utilisée pour configurer Notification Hubs.

À la fin de ces étapes, vous devez disposer des informations suivantes, dont vous vous servirez dans [Configurer votre hub de notification avec des informations APNS](#configure-the-notification-hub-with-apns-information) :

- **ID de l’équipe** (voir l’étape 1)
- **ID de bundle** (voir l’étape 1)
- **ID de clé** (voir l’étape 7)
- **Valeur de jeton** (la valeur de clé .p8 ; voir l’étape 8)

## <a name="create-a-provisioning-profile"></a>Créer un profil de provisionnement

1. Revenez au [portail de provisionnement iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456), sélectionnez **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils), sélectionnez **Profiles** (Profils) dans le menu de gauche, puis sélectionnez **+** pour créer un profil. L’écran **Register a New Provisioning Profile** (Inscrire un nouveau profil de provisionnement) s’affiche.

2. Sélectionnez **iOS App Development** (Développement d’application iOS) sous **Development** (Développement) comme type de profil de provisionnement, puis sélectionnez **Continue** (Continuer).

   :::image type="content" source="media/ios-sdk-get-started/image15.png" alt-text="Liste de profils de provisionnement":::

3. Ensuite, dans la liste déroulante **App ID**, sélectionnez l’ID d’application que vous avez créé, puis sélectionnez **Continue**.

   :::image type="content" source="media/ios-sdk-get-started/image16.png" alt-text="Sélection d’un ID d’application":::

4. Dans la fenêtre **Select certificates** (Sélectionner des certificats), sélectionnez le certificat de développement utilisé pour la signature de code, puis sélectionnez **Continuer**. Ce certificat n’est pas le certificat Push que vous avez créé. S’il n’en existe pas, vous devez le créer. Si un certificat existe, passez à l’étape suivante. Pour créer un certificat de développement s’il n’en existe pas déjà :

   1. Si vous voyez **No Certificates are available** (Aucun certificat n’est disponible), sélectionnez **Create Certificate** (Créer un certificat).
   2. Dans la section **Software** (Logiciels), sélectionnez **Apple Development** (Développement Apple). Sélectionnez **Continuer**.
   3. Dans l’écran **Create a new Certificate** (Créer un certificat), sélectionnez **Choose File** (Choisir un fichier).
   4. Accédez au fichier de **demande de signature du certificat** que vous avez créé précédemment, sélectionnez-le, puis sélectionnez **Ouvrir**.
   5. Sélectionnez **Continuer**.
   6. Téléchargez le certificat de développement et notez son emplacement pour le retrouver facilement.

5. Revenez à la page **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils), sélectionnez **Profiles** dans le menu de gauche, puis sélectionnez **+** pour créer un profil. L’écran **Register a New Provisioning Profile** (Inscrire un nouveau profil de provisionnement) s’affiche.

6. Dans la fenêtre **Select certificates** (Sélectionner des certificats), sélectionnez le certificat de développement que vous venez de créer. Sélectionnez **Continuer**.

7. Sélectionnez les appareils à utiliser pour le test, puis sélectionnez **Continue**.

8. Pour finir, choisissez un nom pour le profil dans **Provisioning Profile Name**, puis sélectionnez **Generate**.

   :::image type="content" source="media/ios-sdk-get-started/image17.png" alt-text="Choisir un nom de profil de provisionnement":::

9. Une fois le profil de provisionnement créé, sélectionnez **Télécharger**. Notez son emplacement pour le retrouver facilement.

10. Accédez à l’emplacement du profil de provisionnement, puis double-cliquez sur le profil pour l’installer sur votre machine de développement Xcode.

## <a name="create-a-notification-hub"></a>Création d’un hub de notifications

Dans cette section, vous créez un hub de notification et configurez l’authentification avec APNS à l’aide du certificat Push .p12 ou de l’authentification basée sur un jeton. Si vous souhaitez utiliser un hub de notification que vous avez déjà créé, vous pouvez passer directement à l’étape 5.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).

2. Sélectionnez **Tous les services** dans le menu de gauche, puis sélectionnez **Notification Hubs** dans la section **Mobile**. Sélectionnez l’icône d’étoile en regard du nom du service pour l’ajouter à la section **FAVORIS** dans le menu de gauche. Après avoir ajouté **Notification Hubs** aux **FAVORIS**, sélectionnez-le.

   :::image type="content" source="media/ios-sdk-get-started/image18.png" alt-text="Portail Azure":::

3. Dans la page **Notification Hubs**, sélectionnez **Ajouter** dans la barre d’outils.

   :::image type="content" source="media/ios-sdk-get-started/image19.png" alt-text="Bouton de barre d’outils Ajouter":::

4. Dans la page **Notification Hubs**, effectuez les étapes suivantes :

   1. Entrez un nom dans **Notification Hub**.
   2. Entrez un nom dans **Créer un espace de noms**. Un espace de noms contient un ou plusieurs hubs de notification.
   3. Sélectionnez une valeur dans la liste déroulante **Emplacement**. Cette valeur spécifie l’emplacement où vous voulez créer le hub de notification.
   4. Sélectionnez un groupe de ressources existant dans **Groupe de ressources**, ou créez-en un.
   5. Sélectionnez **Créer**.

   :::image type="content" source="media/ios-sdk-get-started/image20.png" alt-text="Définir des propriétés":::

5. Sélectionnez **Notifications** (icône de cloche), puis **Accéder à la ressource**. Vous pouvez également actualiser la liste dans la page **Notification Hubs** et sélectionner votre hub.

   :::image type="content" source="media/ios-sdk-get-started/image21.png" alt-text="Notifications du portail":::

6. Sélectionnez **Stratégies d’accès** dans la liste. Notez que les deux chaînes de connexion sont disponibles pour vous. Vous en aurez besoin pour gérer les notifications Push.

   > [!IMPORTANT]
   > N’utilisez pas la stratégie **DefaultFullSharedAccessSignature** dans votre application. Elle est censée être utilisée uniquement dans votre back-end.

   :::image type="content" source="media/ios-sdk-get-started/image22.png" alt-text="Chaînes de connexion":::

## <a name="configure-the-notification-hub-with-apns-information"></a>Configurer le hub de notification avec des informations APNS

Sous **Services de notification**, sélectionnez **Apple (APNS)** , puis suivez les étapes appropriées en fonction de l’approche que vous avez choisie dans la section [Création d’un certificat pour les hubs de notification](#create-a-certificate-for-notification-hubs).

> [!NOTE]
> Utilisez **Production** comme **Mode Application** uniquement si vous souhaitez envoyer des notifications Push aux utilisateurs ayant acheté votre application dans le Store.

### <a name="option-1-use-a-p12-push-certificate"></a>Option 1 : Utiliser un certificat Push .p12

1. Sélectionnez **Certificate**.

2. Sélectionnez l’icône du fichier.

3. Sélectionnez le fichier .p12 exporté précédemment, puis sélectionnez **Ouvrir**.

4. S’il y a lieu, entrez le mot de passe correct.

5. Sélectionnez le mode **Bac à sable**.

   :::image type="content" source="media/ios-sdk-get-started/image23.png" alt-text="Configurer":::

6. Sélectionnez **Enregistrer**.

### <a name="option-2-use-token-based-authentication"></a>Option n°2 : Utiliser l’authentification basée sur un jeton

1. Sélectionnez **Jeton**.

2. Entrez les valeurs suivantes déjà en votre possession :

   - **ID de clé**
   - **ID de bundle**
   - **ID de l’équipe**
   - **Jeton**

3. Choisissez **Bac à sable**.

4. Sélectionnez **Enregistrer**.

Vous avez maintenant configuré votre hub de notification avec APNS. Vous disposez aussi des chaînes de connexion nécessaires pour inscrire votre application et envoyer des notifications Push.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé et configuré un hub de notification dans Azure et vous l’avez configuré pour autoriser l’envoi de notifications à votre application par le biais d’Apple Push Notification Service (APNS). Maintenant, nous allons créer un exemple d’application iOS et intégrer le SDK Azure Notifications Hubs afin de lui permettre de recevoir des notifications Push envoyées par le biais du portail Azure. Passez au tutoriel suivant en fonction du langage de votre choix :

- [Tutoriel : Envoyer des notifications Push vers des applications iOS avec Azure Notification Hubs](ios-sdk-300.md)
