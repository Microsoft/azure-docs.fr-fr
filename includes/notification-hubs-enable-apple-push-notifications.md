---
title: Fichier include
description: Fichier include
services: notification-hubs
author: sethmanheim
ms.service: notification-hubs
ms.topic: include
ms.date: 02/10/2020
ms.author: sethm
ms.custom: include file
ms.openlocfilehash: 7b5034f2163e8478d7ddb7b9271402b094a809d7
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95560826"
---
## <a name="generate-the-certificate-signing-request-file"></a>Générer le fichier de demande de signature de certificat

Le service de notification Push Apple (APNs) utilise des certificats pour authentifier vos notifications Push. Suivez ces instructions pour créer le certificat push nécessaire pour envoyer et recevoir des notifications. Pour plus d’informations sur ces concepts, consultez la documentation officielle [d’Apple Push Notification Service](https://developer.apple.com/library/archive/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html).

Générez le fichier de demande de signature de certificat (CSR, Certificate Signing Request) utilisé par Apple pour générer un certificat Push signé.

1. Sur votre Mac, exécutez l’outil Trousseaux d’accès. Vous pouvez l’ouvrir à partir du dossier **Utilitaires** ou du dossier **Autre** sur Launchpad.

1. Sélectionnez **Trousseaux d’accès**, développez **Assistant de certification**, puis cliquez sur **Demander un certificat à une autorité de certification**.

    ![Utiliser l’accès au trousseau pour demander un nouveau certificat](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-request-cert-from-ca.png)

   > [!NOTE]
   > Par défaut, Trousseaux d’accès sélectionne le premier élément de la liste. Cela peut poser problème si vous êtes dans la catégorie **Certificats** et que **Apple Worldwide Developer Relations Certification Authority** (Autorité de certification des relations des développeurs dans le monde entier) ne figure pas comme premier élément dans la liste. Veillez à disposer d’un élément non-clé, ou que la clé **Apple Worldwide Developer Relations Certification Authority** est sélectionnée, avant de générer la demande de signature de certificat (CSR).

1. Sélectionnez votre **adresse e-mail d’utilisateur**, entrez votre **nom commun**, veillez à spécifier **Enregistré sur le disque**, puis sélectionnez **Continuer**. Laissez le champ **Adresse de messagerie d’autorité de certification** vide, car il n’est pas requis.

    ![Informations requises sur le certificat](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-csr-info.png)

1. Entrez un nom pour le fichier CSR dans **Enregistrer en tant que**, sélectionnez l’emplacement dans **Où**, puis sélectionnez **Enregistrer**.

    ![Choisir un nom de fichier pour le certificat](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-save-csr.png)

    Cette action enregistre le fichier CSR à l’emplacement sélectionné. L’emplacement par défaut est **Bureau**. Notez l’emplacement choisi pour ce fichier.

Ensuite, inscrivez votre application auprès d’Apple, activez les notifications Push, puis téléchargez le fichier de demande de signature de certificat exporté pour créer un certificat Push.

## <a name="register-your-app-for-push-notifications"></a>Inscription de votre application pour les notifications Push

Pour envoyer des notifications Push vers une application iOS, inscrivez votre application auprès d’Apple ainsi que pour les notifications Push.  

1. Si vous n’avez pas encore inscrit votre application, accédez au [portail de provisionnement iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456) dans le Centre pour développeurs Apple. Connectez-vous au portail avec votre ID Apple, puis sélectionnez **Identifiants**. Sélectionnez ensuite **+** pour inscrire une nouvelle application.

    ![Page d’ID d’application Portail d’approvisionnement iOS](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-ios-appids.png)

2. Dans l’écran **Register a New Identifier** (Inscrire un nouvel identifiant), sélectionnez la case d’option **App IDs** (ID d’application). Sélectionnez **Continuer**.

    ![Portail de provisionnement iOS - Page d’inscription d’un nouvel ID d’application](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-ios-appids-new.png)

3. Mettez à jour les trois valeurs suivantes pour votre nouvelle application, puis sélectionnez **Continue** :

   * **Description** : tapez un nom descriptif pour votre application.

   * **ID de l’offre groupée** : entrez un identifiant de bundle au format **Organization Identifier.Product Name** comme indiqué dans le [Guide de distribution d’application](https://help.apple.com/xcode/mac/current/#/dev91fe7130a). Les valeurs *Organization Identifier* et *Product Name* doivent correspondre à l’identificateur d’organisation et au nom de produit que vous utiliserez pour créer le projet Xcode. Dans la capture d’écran ci-dessous, la valeur **NotificationHubs** est utilisée comme identificateur d’organisation, tandis que la valeur **GetStarted** correspond au nom du produit. Vérifiez que la valeur **Bundle Identifier** correspond à celle de votre projet Xcode, afin que Xcode utilise le profil de publication correct.

      ![Portail de provisionnement iOS - Page d’inscription d’un ID d’application](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-new-appid-bundle.png)

   * **Notifications Push** : cochez l’option **Notifications Push** dans la section **Capabilities** (Fonctionnalités).

      ![Formulaire pour inscrire un nouvel ID d’application](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-new-appid-push.png)

      L’ID de votre application est généré et vous êtes invité à confirmer les informations. Sélectionnez **Continue** (Continuer), puis **Register** (Inscrire) pour confirmer le nouvel ID d’application.

      ![Confirmer le nouvel ID d’application](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-new-appid-register.png)

      Après avoir sélectionné **Register** (Inscrire), vous voyez le nouvel ID d’application affiché comme élément de ligne dans la page **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils).

4. Dans la page **Certificates, Identifiers & Profiles**, sous **Identifiers** (Identifiants), recherchez l’élément de ligne ID d’application que vous venez de créer, puis sélectionnez sa ligne pour afficher l’écran **Edit your App ID Configuration** (Modifier votre configuration d’ID d’application).

## <a name="creating-a-certificate-for-notification-hubs"></a>Création d’un certificat pour les hubs de notification
Un certificat est nécessaire pour permettre au hub de notification de fonctionner avec **APNS**. Cette opération peut être effectuée de deux manières :

1. Créez un certificat **.p12** qui peut être chargé directement sur le hub de notification.  
2. Créez un certificat **.p8** qui peut être utilisé pour l’[authentification basée sur un jeton](../articles/notification-hubs/notification-hubs-push-notification-http2-token-authentication.md) (*l’approche la plus récente*).

L’approche la plus récente présente un certain nombre d’avantages (par rapport à l’utilisation de certificats), comme indiqué dans [Authentification basée sur un jeton (HTTP/2) pour APNS](../articles/notification-hubs/notification-hubs-push-notification-http2-token-authentication.md). Toutefois, des étapes ont été fournies pour les deux approches. 

### <a name="option-1-creating-a-p12-push-certificate-that-can-be-uploaded-directly-to-notification-hub"></a>OPTION 1 : Création d’un certificat Push .p12 qui peut être chargé directement sur le hub de notification

1. Faites défiler la page jusqu’à l’option cochée **Push Notifications** (Notifications Push), puis sélectionnez **Configure** (Configurer) pour créer le certificat.

    ![Page de modification d’ID d’application](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-edit-appid.png)

2. La fenêtre **Apple Push Notification service SSL Certificates** (Certificats SSL du service Apple Push Notification) s’affiche. Sélectionnez le bouton **Create Certificate** (Créer un certificat) sous la section **Development SSL Certificate** (Certificat de développement SSL).

    ![Créer un certificat pour le bouton d’ID d’application](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-appid-create-cert.png)

    L’écran **Create a new Certificate** (Créer un certificat) s’affiche.

    > [!NOTE]
    > Ce tutoriel utilise un certificat de développement dont votre application se sert pour générer un jeton d’appareil unique. Le même processus est utilisé lors de l’inscription d’un certificat de production. Assurez-vous simplement que vous utilisez le même type de certificat quand vous envoyez des notifications.

3. Sélectionnez **Choose File** (Choisir un fichier), accédez à l’emplacement où vous avez enregistré le fichier de demande de signature de certificat créé lors de la première tâche, puis double-cliquez sur le nom du certificat à charger. Sélectionnez **Continuer**.

4. Une fois que le portail a créé un certificat, sélectionnez le bouton **Télécharger**. Enregistrez le certificat et notez son emplacement pour le retrouver facilement.

    ![Page de téléchargement du certificat généré](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-appid-download-cert.png)

    Le certificat est téléchargé et enregistré sur votre ordinateur dans le dossier **Téléchargements**.

    ![Recherchez le fichier de certificat dans le dossier Téléchargements](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-cert-downloaded.png)

    > [!NOTE]
    > Par défaut, le certificat de développement téléchargé se nomme **aps_development.cer**.

5. Double-cliquez sur le certificat Push téléchargé **aps_development.cer**. Cette opération installe le nouveau certificat dans le Trousseau, comme indiqué dans l’image suivante :

    ![Liste de certificats d’accès Keychain montrant le nouveau certificat](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-cert-in-keychain.png)

    > [!NOTE]
    > Bien que le nom de votre certificat puisse être différent, il portera toutefois le préfixe **Apple Development iOS Push Notification Services**.

6. Dans Trousseau d’accès, cliquez avec le bouton droit sur le certificat push que vous avez créé dans la catégorie **Certificats** . Sélectionnez **Exporter**, nommez le fichier, sélectionnez le format **.p12**, puis sélectionnez **Enregistrer**.

    ![Exportation du certificat au format p12](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-export-cert-p12.png)

    Vous pouvez choisir de protéger le certificat par un mot de passe, mais cela est facultatif. Cliquez sur **OK** si vous souhaitez ignorer l’étape de création du mot de passe. Notez le nom du fichier et l’emplacement du certificat .p12 exporté. Ces informations sont nécessaires pour activer l’authentification avec APNs.

    > [!NOTE]
    > Le nom et l’emplacement de votre fichier .p12 peuvent être différents de ceux montrés dans ce tutoriel.

### <a name="option-2-creating-a-p8-certificate-that-can-be-used-for-token-based-authentication"></a>OPTION 2 : Création d’un certificat .p8 qui peut être utilisé pour l’authentification basée sur un jeton

1. Prenez note des détails suivants :

    - **Préfixe d’ID d’application** (il s’agit d’un **ID d’équipe**)
    - **ID de bundle**
    
2. De retour dans **Certificats, identificateurs et profils**, cliquez sur **Clés**.

   > [!NOTE]
   > Si vous disposez déjà d’une clé configurée pour **APNS**, vous pouvez réutiliser le certificat .p8 que vous avez téléchargé juste après l’avoir créée. Dans ce cas, vous pouvez ignorer les étapes **3** à **5**.

3. Cliquez sur le bouton **+** (ou sur le bouton **Créer une clé**) pour créer une clé.
4. Fournissez une valeur appropriée pour **Nom de clé**, activez l’option **Apple Push Notification Service (APNs)** , puis cliquez sur **Continuer**, puis sur **Inscrire** dans l’écran suivant.
5. Cliquez sur **Télécharger**, puis déplacez le fichier **.p8** (préfixé avec *AuthKey_* ) vers un répertoire local sécurisé, puis cliquez sur **Terminé**.

   > [!NOTE] 
   > Veillez à conserver votre fichier .p8 dans un emplacement sécurisé (et à enregistrer une sauvegarde). Une fois votre clé téléchargée, elle ne peut pas être retéléchargée, car la copie du serveur est supprimée.
  
6. Dans **Clés**, cliquez sur la clé que vous venez de créer (ou une clé existante si vous avez choisi de l’utiliser à la place).
7. Notez la valeur de l’**ID de clé**.
8. Ouvrez votre certificat .p8 dans l’application appropriée de votre choix, par exemple [**Visual Studio Code**](https://code.visualstudio.com), puis notez la valeur de la clé. Il s’agit de la valeur comprise entre **-----BEGIN PRIVATE KEY-----** et **-----END PRIVATE KEY-----** .

    ```
    -----BEGIN PRIVATE KEY-----
    <key_value>
    -----END PRIVATE KEY-----
    ```

    > [!NOTE]
    > Il s’agit de la **valeur de jeton** qui sera utilisée pour configurer le **hub de notification**. 

À la fin de ces étapes, vous devez disposer des informations suivantes, dont vous vous servirez dans [Configurer votre hub de notification avec des informations APNs](#configure-your-notification-hub-with-apns-information) :

- **ID de l’équipe** (voir l’étape 1)
- **ID de bundle** (voir l’étape 1)
- **ID de clé** (voir l’étape 7)
- **Valeur de jeton**, c’est-à-dire, la valeur de clé .p8 (voir l’étape 8)

## <a name="create-a-provisioning-profile-for-the-app"></a>Création d’un profil d’approvisionnement pour l’application

1. Revenez au [portail de provisionnement iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456), sélectionnez **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils), sélectionnez **Profiles** (Profils) dans le menu de gauche, puis sélectionnez **+** pour créer un profil. L’écran **Register a New Provisioning Profile** (Inscrire un nouveau profil de provisionnement) s’affiche.

1. Sélectionnez **iOS App Development** (Développement d’application iOS) sous **Development** (Développement) comme type de profil de provisionnement, puis sélectionnez **Continue** (Continuer).

    ![Liste du profil d'approvisionnement](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-new-provisioning-profile.png)

1. Ensuite, dans la liste déroulante **App ID**, sélectionnez l’ID d’application que vous avez créé, puis sélectionnez **Continue**.

    ![Sélectionner l’ID de l’application](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-select-appid-for-provisioning.png)

1. Dans la fenêtre **Select certificates** (Sélectionner des certificats), sélectionnez le certificat de développement utilisé pour la signature de code, puis sélectionnez **Continuer**. Ce certificat n’est pas le certificat Push que vous avez créé. S’il n’en existe pas, vous devez le créer. Si un certificat existe, passez à l’étape suivante. Pour créer un certificat de développement s’il n’en existe pas déjà :

    1. Si vous voyez **No Certificates are available** (Aucun certificat n’est disponible), sélectionnez **Create Certificate** (Créer un certificat).
    2. Dans la section **Software** (Logiciels), sélectionnez **Apple Development** (Développement Apple). Sélectionnez **Continuer**.
    3. Dans l’écran **Create a new Certificate** (Créer un certificat), sélectionnez **Choose File** (Choisir un fichier).
    4. Accédez au fichier de **demande de signature du certificat** que vous avez créé précédemment, sélectionnez-le, puis sélectionnez **Ouvrir**.
    5. Sélectionnez **Continuer**.
    6. Téléchargez le certificat de développement et notez son emplacement pour le retrouver facilement.

1. Revenez à la page **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils), sélectionnez **Profiles** dans le menu de gauche, puis sélectionnez **+** pour créer un profil. L’écran **Register a New Provisioning Profile** (Inscrire un nouveau profil de provisionnement) s’affiche.

1. Dans la fenêtre **Select certificates** (Sélectionner des certificats), sélectionnez le certificat de développement que vous venez de créer. Sélectionnez **Continuer**.

1. Sélectionnez les appareils à utiliser pour le test, puis sélectionnez **Continue**.

1. Pour finir, choisissez un nom pour le profil dans **Provisioning Profile Name** (Nom du profil de provisionnement), puis sélectionnez **Generate** (Générer).

    ![Choisir un nom du profil d'approvisionnement](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-provisioning-name-profile.png)

1. Une fois le profil de provisionnement créé, sélectionnez **Télécharger**. Notez son emplacement pour le retrouver facilement.

1. Accédez à l’emplacement du profil de provisionnement, puis double-cliquez sur le profil pour l’installer sur votre machine de développement Xcode.

## <a name="create-a-notification-hub"></a>Création d’un hub de notifications

Dans cette section, vous créez un hub de notification et configurez l’authentification avec APNs à l’aide du certificat Push .p12 ou de l’authentification basée sur un jeton. Si vous souhaitez utiliser un hub de notification que vous avez déjà créé, vous pouvez passer directement à l’étape 5.

[!INCLUDE [notification-hubs-portal-create-new-hub](notification-hubs-portal-create-new-hub.md)]

## <a name="configure-your-notification-hub-with-apns-information"></a>Configurer votre hub de notification avec des informations APNs

Sous **Services de notification**, sélectionnez **Apple (APNS)** , puis suivez les étapes appropriées en fonction de l’approche que vous avez choisie dans la section [Création d’un certificat pour les hubs de notification](#creating-a-certificate-for-notification-hubs).  

> [!NOTE]
> Si vous créez votre application avec l’App Store ou un profil de distribution ad hoc, utilisez la **production** pour le **mode d’application**. Ceci permettra à votre appareil d’envoyer des notifications Push aux utilisateurs qui ont acheté votre application sur le Store.

### <a name="option-1-using-a-p12-push-certificate"></a>OPTION 1 : Utilisation d’un certificat Push .p12

1. Sélectionnez **Certificate**.

1. Sélectionnez l’icône du fichier.

1. Sélectionnez le fichier .p12 exporté précédemment, puis sélectionnez **Ouvrir**.

1. S’il y a lieu, entrez le mot de passe correct.

1. Sélectionnez le mode **Bac à sable**.

    ![Configurer la certification APNS dans le portail Azure](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-apple-config-cert.png)

1. Sélectionnez **Enregistrer**.

### <a name="option-2-using-token-based-authentication"></a>OPTION 2 : Utilisation de l’authentification basée sur un jeton

1. Sélectionnez **Jeton**.
1. Entrez les valeurs suivantes déjà en votre possession :

    - **ID de clé**
    - **ID de bundle**
    - **ID de l’équipe**
    - **Jeton** 

1. Choisissez **Bac à sable**.
1. Sélectionnez **Enregistrer**. 

Vous avez maintenant configuré votre hub de notification avec APNs. Vous disposez aussi des chaînes de connexion pour inscrire votre application et envoyer des notifications Push.