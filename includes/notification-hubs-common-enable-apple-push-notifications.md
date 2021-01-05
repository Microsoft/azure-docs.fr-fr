---
author: mikeparker104
ms.author: miparker
ms.date: 06/02/2020
ms.service: notification-hubs
ms.topic: include
ms.openlocfilehash: 72065ce602c6d29255a3500e26d8c6f36a19ebed
ms.sourcegitcommit: 971a3a63cf7da95f19808964ea9a2ccb60990f64
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 06/19/2020
ms.locfileid: "85095312"
---
### <a name="register-your-ios-app-for-push-notifications"></a>Inscrire votre application iOS pour les notifications Push

Pour envoyer des notifications Push vers une application iOS, inscrivez votre application auprès d’Apple ainsi que pour les notifications Push.  

1. Si vous n’avez pas encore inscrit votre application, accédez au [portail de provisionnement iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456) dans le Centre pour développeurs Apple. Connectez-vous au portail avec votre identifiant Apple, accédez à **Certificates, Identifiers & Profiles**, puis sélectionnez **Identifiers**. Cliquez sur **+** pour inscrire une nouvelle application.

    ![Page d’ID d’application Portail d’approvisionnement iOS](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-ios-appids.png)

1. Dans l’écran **Register a New Identifier** (Inscrire un nouvel identifiant), sélectionnez la case d’option **App IDs** (ID d’application). Sélectionnez **Continuer**.

    ![Portail de provisionnement iOS - Page d’inscription d’un nouvel ID d’application](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-ios-appids-new.png)

1. Mettez à jour les trois valeurs suivantes pour votre nouvelle application, puis sélectionnez **Continue** :

   * **Description** : tapez un nom descriptif pour votre application.

   * **ID de l’offre groupée** : entrez un identifiant de bundle au format **com.<identificateur_organisation>.<nom_produit>** comme indiqué dans le [Guide de distribution d’application](https://help.apple.com/xcode/mac/current/#/dev91fe7130a). Dans la capture d’écran ci-dessous, la valeur `mobcat` est utilisée comme identificateur d’organisation, tandis que la valeur **PushDemo** correspond au nom du produit.

      ![Portail de provisionnement iOS - Page d’inscription d’un ID d’application](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-new-appid-bundle.png)

   * **Notifications Push** : cochez l’option **Notifications Push** dans la section **Capabilities** (Fonctionnalités).

      ![Formulaire pour inscrire un nouvel ID d’application](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-new-appid-push.png)

      L’ID de votre application est généré et vous êtes invité à confirmer les informations. Sélectionnez **Continue** (Continuer), puis **Register** (Inscrire) pour confirmer le nouvel ID d’application.

      ![Confirmer le nouvel ID d’application](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-new-appid-register.png)

      Après avoir sélectionné **Register** (Inscrire), vous voyez le nouvel ID d’application affiché comme élément de ligne dans la page **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils).

1. Dans la page **Certificates, Identifiers & Profiles**, sous **Identifiers**, localisez l’élément de ligne de l’ID d’application que vous avez créé. Sélectionnez ensuite sa ligne pour afficher l’écran **Modifier votre configuration d’ID d’application**.

#### <a name="creating-a-certificate-for-notification-hubs"></a>Création d’un certificat pour Notification Hubs

Un certificat est nécessaire pour permettre au hub de notification de fonctionner avec les **services de notification Push Apple (APNS)** et peut être fourni de deux manières :

1. [Création d’un certificat Push p12 qui peut être chargé directement sur le hub de notification](#option-1-creating-a-p12-push-certificate-that-can-be-uploaded-directly-to-notification-hub) (*l’approche d’origine*)

1. [Création d’un certificat p8 qui peut être utilisé pour l’authentification basée sur un jeton](#option-2-creating-a-p8-certificate-that-can-be-used-for-token-based-authentication) (*l’approche la plus récente et recommandée*)

L’approche la plus récente présente un certain nombre d’avantages, comme indiqué dans [Authentification basée sur un jeton (HTTP/2) pour APNS](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-push-notification-http2-token-authentification). Moins d’étapes sont nécessaires, mais elles sont également obligatoires pour des scénarios spécifiques. Toutefois, des étapes ont été fournies pour les deux approches, l’une comme l’autre pouvant être utilisée dans le cadre de ce tutoriel.

##### <a name="option-1-creating-a-p12-push-certificate-that-can-be-uploaded-directly-to-notification-hub"></a>OPTION 1 : Création d’un certificat Push p12 qui peut être chargé directement sur le hub de notification

1. Sur votre Mac, exécutez l’outil Trousseaux d’accès. Vous pouvez l’ouvrir à partir du dossier **Utilitaires** ou du dossier **Autre** sur Launchpad.

1. Sélectionnez **Trousseaux d’accès**, développez **Assistant de certification**, puis cliquez sur **Demander un certificat à une autorité de certification**.

    ![Utiliser l’accès au trousseau pour demander un nouveau certificat](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-request-cert-from-ca.png)

   > [!NOTE]
   > Par défaut, Trousseaux d’accès sélectionne le premier élément de la liste. Cela peut poser problème si vous êtes dans la catégorie **Certificats** et que **Apple Worldwide Developer Relations Certification Authority** (Autorité de certification des relations des développeurs dans le monde entier) ne figure pas comme premier élément dans la liste. Veillez à disposer d’un élément non-clé, ou que la clé **Apple Worldwide Developer Relations Certification Authority** est sélectionnée, avant de générer la demande de signature de certificat (CSR).

1. Sélectionnez votre **adresse e-mail d’utilisateur**, entrez votre **nom commun**, veillez à spécifier **Enregistré sur le disque**, puis sélectionnez **Continuer**. Laissez le champ **Adresse de messagerie d’autorité de certification** vide, car il n’est pas requis.

    ![Informations attendues sur le certificat](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-csr-info.png)

1. Entrez un nom pour le **fichier de demande de signature de certificat** dans **Enregistrer sous**, sélectionnez l’emplacement dans **Où**, puis sélectionnez **Enregistrer**.

    ![Choisir un nom de fichier pour le certificat](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-save-csr.png)

    Cette action enregistre le **fichier CSR** à l’emplacement sélectionné. L’emplacement par défaut est **Bureau**. Notez l’emplacement choisi pour ce fichier.

1. De retour dans la page **Certificates, Identifiers & Profiles** du [portail de provisionnement iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456), faites défiler jusqu’à l’option activée **Push Notifications**, puis sélectionnez **Configure** pour créer le certificat.

    ![Page de modification d’ID d’application](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-edit-appid.png)

1. La fenêtre **Apple Push Notification service TLS/SSL Certificates** (Certificats TLS/SSL du service Apple Push Notification) s’affiche. Sélectionnez le bouton **Create Certificate** (Créer un certificat) sous la section **Development TLS/SSL Certificate** (Certificat de développement TLS/SSL).

    ![Créer un certificat pour le bouton d’ID d’application](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-appid-create-cert.png)

    L’écran **Create a new Certificate** (Créer un certificat) s’affiche.

    > [!NOTE]
    > Ce didacticiel utilise un certificat de développement. Le même processus est utilisé lors de l’inscription d’un certificat de production. Assurez-vous simplement que vous utilisez le même type de certificat quand vous envoyez des notifications.

1. Sélectionnez **Choose File** (Choisir un fichier), accédez à l’emplacement où vous avez enregistré le **fichier de demande de signature de certificat**, puis double-cliquez sur le nom du certificat à charger. Sélectionnez **Continuer**.

1. Une fois que le portail a créé un certificat, sélectionnez le bouton **Télécharger**. Enregistrez le certificat et notez son emplacement pour le retrouver facilement.

    ![Page de téléchargement du certificat généré](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-appid-download-cert.png)

    Le certificat est téléchargé et enregistré sur votre ordinateur dans le dossier **Téléchargements**.

    ![Recherchez le fichier de certificat dans le dossier Téléchargements](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-cert-downloaded.png)

    > [!NOTE]
    > Par défaut, le certificat de développement téléchargé se nomme **aps_development.cer**.

1. Double-cliquez sur le certificat Push téléchargé **aps_development.cer**. Cette opération installe le nouveau certificat dans le Trousseau, comme indiqué dans l’image suivante :

    ![Liste de certificats d’accès Keychain montrant le nouveau certificat](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-cert-in-keychain.png)

    > [!NOTE]
    > Bien que le nom de votre certificat puisse être différent, il portera le préfixe **Apple Development iOS Push Notification Services** et se verra associer l’identificateur de bundle approprié.

1. Dans Trousseau d’accès, utilisez **Ctrl** + **clic** sur le certificat push que vous avez créé dans la catégorie **Certificats**. Sélectionnez **Exporter**, nommez le fichier, sélectionnez le format **p12**, puis sélectionnez **Enregistrer**.

    ![Exportation du certificat au format p12](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-export-cert-p12.png)

    Vous pouvez choisir de protéger le certificat par un mot de passe, mais ce dernier est facultatif. Cliquez sur **OK** si vous souhaitez ignorer l’étape de création du mot de passe. Notez le nom du fichier et l’emplacement du certificat p12 exporté. Ces informations sont nécessaires pour activer l’authentification avec APNs.

    > [!NOTE]
    > Le nom et l’emplacement de votre fichier p12 peuvent être différents de ceux montrés dans ce tutoriel.

##### <a name="option-2-creating-a-p8-certificate-that-can-be-used-for-token-based-authentication"></a>OPTION 2 : Création d’un certificat p8 qui peut être utilisé pour l’authentification basée sur un jeton

1. Prenez note des détails suivants :

    * **Préfixe d’ID d’application** (**ID d’équipe**)
    * **ID de bundle**

1. De retour dans **Certificats, identificateurs et profils**, cliquez sur **Clés**.

   > [!NOTE]
   > Si vous disposez déjà d’une clé configurée pour **APNS**, vous pouvez réutiliser le certificat p8 que vous avez téléchargé juste après l’avoir créée. Dans ce cas, vous pouvez ignorer les étapes **3** à **5**.

1. Cliquez sur le bouton **+** (ou sur le bouton **Créer une clé**) pour créer une clé.
1. Fournissez une valeur appropriée pour **Nom de clé**, activez l’option **Apple Push Notification Service (APNS)** , puis cliquez sur **Continuer**, puis sur **Inscrire** dans l’écran suivant.
1. Cliquez sur **Télécharger**, puis déplacez le fichier **p8** (préfixé avec *AuthKey_* ) vers un répertoire local sécurisé, puis cliquez sur **Terminé**.

   > [!NOTE]
   > Veillez à conserver votre fichier p8 dans un emplacement sécurisé (et à enregistrer une sauvegarde). Une fois votre clé téléchargée, elle ne peut pas être retéléchargée, car la copie du serveur est supprimée.
  
1. Dans **Clés**, cliquez sur la clé que vous avez créée (ou une clé existante si vous avez choisi de l’utiliser à la place).
1. Notez la valeur de l’**ID de clé**.
1. Ouvrez votre certificat .p8 dans l’application appropriée de votre choix, par exemple [**Visual Studio Code**](https://code.visualstudio.com). Notez la valeur de la clé (comprise entre **-----BEGIN PRIVATE KEY-----** et **-----END PRIVATE KEY-----** ).

    -----BEGIN PRIVATE KEY-----  
    <valeur_clé>  
    -----END PRIVATE KEY-----

    > [!NOTE]
    > Il s’agit de la **valeur de jeton** qui sera utilisée pour configurer le **hub de notification**.

À la fin de ces étapes, vous devez disposer des informations suivantes, dont vous vous servirez dans [Configurer votre hub de notification avec des informations APNS](#configure-your-notification-hub-with-apns-information) :

* **ID de l’équipe** (voir l’étape 1)
* **ID de bundle** (voir l’étape 1)
* **ID de clé** (voir l’étape 7)
* **Valeur de jeton** (valeur de clé p8 obtenue à l’étape 8)

### <a name="create-a-provisioning-profile-for-the-app"></a>Création d’un profil d’approvisionnement pour l’application

1. Revenez au [portail de provisionnement iOS](https://go.microsoft.com/fwlink/p/?LinkId=272456), sélectionnez **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils), sélectionnez **Profiles** (Profils) dans le menu de gauche, puis sélectionnez **+** pour créer un profil. L’écran **Register a New Provisioning Profile** (Inscrire un nouveau profil de provisionnement) s’affiche.

1. Sélectionnez **iOS App Development** (Développement d’application iOS) sous **Development** (Développement) comme type de profil de provisionnement, puis sélectionnez **Continue** (Continuer).

    ![Liste du profil d'approvisionnement](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-new-provisioning-profile.png)

1. Ensuite, dans la liste déroulante **App ID**, sélectionnez l’ID d’application que vous avez créé, puis sélectionnez **Continue**.

    ![Sélectionner l’ID de l’application](./media/notification-hubs-common-enable-apple-push-notifications/notification-hubs-select-appid-for-provisioning.png)

1. Dans la fenêtre **Select certificates** (Sélectionner des certificats), sélectionnez le certificat de développement utilisé pour la signature de code, puis sélectionnez **Continuer**.

    > [!NOTE]
    > Ce certificat n’est pas le certificat Push que vous avez créé à l’[étape précédente](#creating-a-certificate-for-notification-hubs). Il s’agit de votre certificat de développement. S’il n’en existe pas, vous devez le créer, car il s’agit d’un [prérequis](#prerequisites) pour ce tutoriel. Vous pouvez créer des certificats de développeur dans le [portail des développeurs Apple](https://developer.apple.com), par le biais de [Xcode](https://developer.apple.com/library/archive/documentation/ToolsLanguages/Conceptual/YourFirstAppStoreSubmission/ProvisionYourDevicesforDevelopment/ProvisionYourDevicesforDevelopment.html) ou dans [Visual Studio](https://docs.microsoft.com/xamarin/ios/get-started/installation/device-provisioning/).

1. Revenez à la page **Certificates, Identifiers & Profiles** (Certificats, identifiants et profils), sélectionnez **Profiles** dans le menu de gauche, puis sélectionnez **+** pour créer un profil. L’écran **Register a New Provisioning Profile** (Inscrire un nouveau profil de provisionnement) s’affiche.

1. Dans la fenêtre **Select certificates** (Sélectionner des certificats), sélectionnez le certificat de développement que vous avez créé. Sélectionnez **Continuer**.

1. Sélectionnez les appareils à utiliser pour le test, puis sélectionnez **Continue**.

1. Pour finir, choisissez un nom pour le profil dans **Provisioning Profile Name** (Nom du profil de provisionnement), puis sélectionnez **Generate** (Générer).

    ![Choisir un nom du profil d'approvisionnement](./media/notification-hubs-enable-apple-push-notifications/notification-hubs-provisioning-name-profile.png)

1. Une fois le profil de provisionnement créé, sélectionnez **Télécharger**. Notez son emplacement pour le retrouver facilement.

1. Accédez à l’emplacement du profil de provisionnement, puis double-cliquez sur le profil pour l’installer sur votre machine de développement.
