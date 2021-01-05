---
title: Tutoriel - Provisionner un appareil X.509 simulé sur Azure IoT Hub à l’aide de Java et des groupes d’inscription
description: Dans ce tutoriel, créez et provisionnez un appareil X.509 simulé à l’aide du kit SDK des appareils et services Java ainsi que des groupes d’inscription pour le service IoT Hub Device Provisioning
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.devlang: java
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 4cfbfe3e3e3ba620d8292767012c9bb866d8a878
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94968092"
---
# <a name="tutorial-create-and-provision-a-simulated-x509-device-using-java-device-and-service-sdk-and-group-enrollments-for-iot-hub-device-provisioning-service"></a>Tutoriel : Créer et approvisionner un appareil X.509 simulé pour le service IoT Hub Device Provisioning à l’aide du Kit de développement logiciel (SDK) pour services et appareils Java et de groupes d’inscription

Ces étapes indiquent comment simuler un appareil X.509 sur votre ordinateur de développement exécutant le système d’exploitation Windows et comment utiliser un exemple de code pour connecter cet appareil simulé au service Device Provisioning et à votre IoT Hub à l’aide de groupes d’inscription. 

Avant de continuer, veillez à réaliser les étapes décrites dans la section [Configuration du service IoT Hub Device Provisioning avec le portail Azure](./quick-setup-auto-provision.md).


## <a name="prerequisites"></a>Prérequis

1. Assurez-vous que le [Java SE Development Kit 8](/azure/developer/java/fundamentals/java-jdk-long-term-support) est bien installé sur votre ordinateur.

1. Téléchargez et installez [Maven](https://maven.apache.org/install.html).

1. Assurez-vous que l’élément `git` est installé sur votre machine et est ajouté aux variables d’environnement accessibles à la fenêtre de commande. Consultez la section relative aux [outils clients de Software Freedom Conservancy](https://git-scm.com/download/) pour accéder à la dernière version des outils `git` à installer, qui inclut **Git Bash**, l’application de ligne de commande que vous pouvez utiliser pour interagir avec votre référentiel Git local. 

1. Utilisez la [Vue d’ensemble des certificats](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/CACertificateOverview.md) suivante pour créer vos certificats de test.

    > [!NOTE]
    > Cette étape nécessite [OpenSSL](https://www.openssl.org/), qui peut être généré et installé à partir d’une source ou téléchargé et installé à partir d’un [tiers](https://wiki.openssl.org/index.php/Binaries), par exemple [celui-ci](https://sourceforge.net/projects/openssl/). Si vous avez déjà créé vos certificats _racine_, _intermédiaires_ et de _périphérique_, vous pouvez ignorer cette étape.
    >

    1. Exécutez les deux premières étapes pour créer vos certificats _racines_ et _intermédiaires_.

    1. Connectez-vous au portail Azure, cliquez sur le bouton **Toutes les ressources** dans le menu de gauche et ouvrez votre service de provisionnement.

        1. Dans le panneau de résumé du service Device Provisioning, sélectionnez **Certificats**, puis cliquez sur le bouton **Ajouter** dans la partie supérieure.

        1. Sous **Ajouter un certificat**, entrez les informations suivantes :
            - Entrez un nom de certificat unique.
            - Sélectionnez le fichier **_RootCA.pem_** que vous avez créé.
            - Cela fait, cliquez sur le bouton **Enregistrer**.

           ![Ajouter un certificat](./media/tutorial-group-enrollments/add-certificate.png)

        1. Sélectionnez le certificat que vous venez de créer :
            - Cliquez sur **Générer le code de vérification**. Copiez le code généré.
            - Exécutez l’étape de vérification. Entrez le _code de vérification_ ou cliquez avec le bouton droit et collez-le dans la fenêtre PowerShell en cours d’exécution.  Appuyez sur **Entrée**.
            - Sélectionnez le fichier **_verifyCert4.pem_** que vous venez de créer dans le portail Azure. Cliquez sur **Vérifier**.

              ![Valider le certificat](./media/tutorial-group-enrollments/validate-certificate.png)

    1. Terminez en exécutant les étapes de création de vos certificats de périphérique et de nettoyage des ressources.

       > [!NOTE]
       > Lorsque vous créez des certificats de périphérique, veillez à utiliser uniquement des caractères alphanumériques minuscules et des traits d’union dans le nom de votre appareil.
       >


## <a name="create-a-device-enrollment-entry"></a>Créer une entrée d’inscription d’appareil

1. Ouvrez une invite de commandes. Clonez le référentiel GitHub pour les exemples de code du Kit de développement logiciel (SDK) Java :

    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-java.git --recursive
    ```

1. Dans le code source téléchargé, accédez au dossier d’exemples **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_**. Ouvrez le fichier **_/src/main/java/samples/com/microsoft/azure/sdk/iot/ServiceEnrollmentGroupSample.java_** dans l’éditeur de votre choix, puis ajoutez les informations suivantes :

    1. Ajoutez `[Provisioning Connection String]` pour votre service d’approvisionnement. Pour cela, procédez comme suit depuis le portail :

        1. Accédez au service d’approvisionnement dans le [portail Azure](https://portal.azure.com).

        1. Ouvrez les **Stratégies d’accès partagé**, puis sélectionnez une stratégie qui a pour autorisation *EnrollmentWrite*.

        1. Copiez la **chaîne de connexion de la clé primaire**.

            ![Comment obtenir la chaîne de connexion d’approvisionnement à partir du portail](./media/tutorial-group-enrollments/provisioning-string.png)  

        1. Dans l’exemple de fichier de code **_ServiceEnrollmentGroupSample.java_**, remplacez `[Provisioning Connection String]` par la **chaîne de connexion de la clé primaire**.

            ```java
            private static final String PROVISIONING_CONNECTION_STRING = "[Provisioning Connection String]";
            ```

    1. Ouvrez votre fichier de certificat de signature intermédiaire dans un éditeur de texte. Mettez à jour la valeur `PUBLIC_KEY_CERTIFICATE_STRING` avec la valeur de votre certificat de signature intermédiaire.

        Si vous avez généré vos certificats d’appareil avec l’interpréteur de commandes Bash, *./certs/azure-iot-test-only.intermediate.cert.pem* contient la clé de certificat intermédiaire. Si vos certificats ont été générés avec PowerShell, *./Intermediate1.pem* est votre fichier de certificat intermédiaire.

        ```java
        private static final String PUBLIC_KEY_CERTIFICATE_STRING =
                "-----BEGIN CERTIFICATE-----\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
                "-----END CERTIFICATE-----\n";
        ```

    1. Accédez au hub IoT associé à votre service d’approvisionnement dans le [portail Azure](https://portal.azure.com). Ouvrez l’onglet **Vue d’ensemble** du hub, puis copiez le **nom d’hôte**. Assignez ce **nom d’hôte** au paramètre *IOTHUB_HOST_NAME*.

        ```java
        private static final String IOTHUB_HOST_NAME = "[Host name].azure-devices.net";
        ```

    1. Étudiez l’exemple de code. Il crée, met à jour, interroge et supprime une inscription d’un groupe d’appareils X.509. Pour vérifier la validité de l’inscription sur le portail, commentez temporairement les lignes suivantes de code à la fin du fichier _ServiceEnrollmentGroupSample.java_ :

        ```java
        // ************************************** Delete info of enrollmentGroup ***************************************
        System.out.println("\nDelete the enrollmentGroup...");
        provisioningServiceClient.deleteEnrollmentGroup(enrollmentGroupId);
        ```

    1. Enregistrez le fichier _ServiceEnrollmentGroupSample.java_.

1. Ouvrez une fenêtre de commande, puis accédez au dossier **_azure-iot-sdk-java/provisioning/provisioning-samples/service-enrollment-group-sample_**.

1. Générez l’exemple de code à l’aide de cette commande :

    ```cmd\sh
    mvn install -DskipTests
    ```

1. Dans la fenêtre de commande, exécutez l’exemple à l’aide de ces commandes :

    ```cmd\sh
    cd target
    java -jar ./service-enrollment-group-sample-{version}-with-deps.jar
    ```

1. Observez la fenêtre de sortie pour savoir si l’inscription a abouti.

    ![Inscription réussie](./media/tutorial-group-enrollments/enrollment.png) 

1. Accédez au service d’approvisionnement dans le portail Azure. Cliquez sur **Gérer les inscriptions**. Notez que le groupe d’appareils X.509 apparaît sous l’onglet **Groupes d’inscription**, avec un *NOM DE GROUPE* généré automatiquement.

## <a name="simulate-the-device"></a>Simuler l’appareil

1. Dans le panneau de résumé du service Device Provisioning, sélectionnez **Vue d’ensemble**, puis notez les valeurs _ID Scope_ (Étendue d’ID) et _Provisioning Service Global Endpoint_ (Point de terminaison global du service d’approvisionnement).

    ![Informations sur le service](./media/tutorial-group-enrollments/extract-dps-endpoints.png)

1. Ouvrez une invite de commandes. Accédez à l’exemple de dossier du projet.

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-samples/provisioning-X509-sample
    ```

1. Modifiez `/src/main/java/samples/com/microsoft/azure/sdk/iot/ProvisioningX509Sample.java` pour inclure votre _Étendue de l’ID_ et votre _Point de terminaison global du service de provisionnement_ que vous avez notés précédemment.

    ```java
    private static final String idScope = "[Your ID scope here]";
    private static final String globalEndpoint = "[Your Provisioning Service Global Endpoint here]";
    private static final ProvisioningDeviceClientTransportProtocol PROVISIONING_DEVICE_CLIENT_TRANSPORT_PROTOCOL = ProvisioningDeviceClientTransportProtocol.HTTPS;
    private static final int MAX_TIME_TO_WAIT_FOR_REGISTRATION = 10000; // in milli seconds
    private static final String leafPublicPem = "<Your Public PEM Certificate here>";
    private static final String leafPrivateKey = "<Your Private PEM Key here>";
    ```

1. Mettez à jour les variables `leafPublicPem` et `leafPrivateKey` avec vos certificats d’appareils publics et privés.

    Si vous avez généré vos certificats d’appareils avec PowerShell, les fichiers mydevice* contiennent la clé publique, la clé privée et le PFX de l’appareil.

    Si vous avez généré vos certificats d’appareils avec l’interpréteur de commandes Bash, ./certs/new-device.cert.pem contient la clé publique. La clé privée de l’appareil se trouve dans le fichier ./private/new-device.key.pem.

    Ouvrez votre fichier de clé publique et mettez à jour la variable `leafPublicPem` ayant cette valeur. Copiez le texte de _---BEGIN PRIVATE KEY---_ dans _---END PRIVATE KEY---_ .

    ```java
    private static final String leafPublicPem = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
    ```

    Ouvrez votre fichier de clé privée et mettez à jour la variable `leafPrivatePem` ayant cette valeur. Copiez le texte de _-----BEGIN RSA PRIVATE KEY-----_ dans _-----END RSA PRIVATE KEY-----_ .

    ```java
    private static final String leafPrivateKey = "-----BEGIN RSA PRIVATE KEY-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END RSA PRIVATE KEY-----\n";
    ```

1. Ajoutez une nouvelle variable juste en dessous de `leafPrivateKey` pour votre certificat intermédiaire. Nommez cette nouvelle variable `intermediateKey`. Donnez-lui la valeur de votre certificat de signature intermédiaire.

    Si vous avez généré vos certificats d’appareil avec l’interpréteur de commandes Bash, *./certs/azure-iot-test-only.intermediate.cert.pem* contient la clé de certificat intermédiaire. Si vos certificats ont été générés avec PowerShell, *./Intermediate1.pem* est votre fichier de certificat intermédiaire.

    ```java
    private static final String intermediateKey = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
    ```

1. Dans la fonction `main`, ajoutez `intermediateKey` à la collection `signerCertificates` avant l’initialisation de `securityProviderX509`.

    ```java
    public static void main(String[] args) throws Exception
    {
        ...

        try
        {
            ProvisioningStatus provisioningStatus = new ProvisioningStatus();

            // Add intermediate certificate as part of the certificate key chain.
            signerCertificates.add(intermediateKey);

            SecurityProvider securityProviderX509 = new SecurityProviderX509Cert(leafPublicPem, leafPrivateKey, signerCertificates);
    ```

1. Enregistrez les changements apportés, puis générez l’exemple. Accédez au dossier cible et exécutez le fichier jar créé.

    ```cmd/sh
    mvn clean install
    cd target
    java -jar ./provisioning-x509-sample-{version}-with-deps.jar
    ```

    ![Inscription réussie](./media/tutorial-group-enrollments/registration.png)

1. Dans le portail, accédez au IoT Hub lié à votre service d’approvisionnement, ouvrez le panneau **Device Explorer**. En cas de réussite de l’approvisionnement de l’appareil simulé X.509 sur le Hub, son ID de périphérique s’affiche sur le panneau **Device Explorer**, avec un *ÉTAT* **activé**. Notez que vous devrez peut-être cliquer sur le bouton **Actualiser** en haut, si vous avez déjà ouvert le panneau avant d’exécuter l’exemple d’application de l’appareil. 

    ![L’appareil est inscrit avec le hub IoT](./media/tutorial-group-enrollments/hub-registration.png) 


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de continuer à manipuler et explorer l’exemple de client d’appareil, ne nettoyez pas les ressources créées lors de ce démarrage rapide. Sinon, procédez aux étapes suivantes pour supprimer toutes les ressources créées lors de ce démarrage rapide.

1. Fermez la fenêtre de sortie de l’exemple de client d’appareil sur votre machine.
1. Dans le menu de gauche du portail Azure, cliquez sur **Toutes les ressources**, puis sélectionnez votre service Device Provisioning. Ouvrez le panneau **Gérer les inscriptions** pour votre service, puis cliquez sur l’onglet **Inscriptions individuelles**. Sélectionnez l’*ID D’INSCRIPTION* de l’appareil inscrit dans ce démarrage rapide, puis cliquez sur le bouton **Supprimer** dans la partie supérieure. 
1. À partir du menu de gauche, dans le portail Azure, cliquez sur **Toutes les ressources**, puis sélectionnez votre IoT Hub. Ouvrez le panneau **IoT Devices** (Appareils IoT) pour votre hub, sélectionnez *l’ID D’APPAREIL* de l’appareil inscrit dans ce démarrage rapide, puis cliquez sur le bouton **Supprimer** dans la partie supérieure.


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé un appareil X.509 simulé sur un ordinateur Windows. Vous l’avez également approvisionné vers votre hub IoT à l’aide du service Azure IoT Hub Device Provisioning et de groupes d’inscription. Pour en savoir plus sur votre appareil X.509, passez aux concepts d’appareil. 

> [!div class="nextstepaction"]
> [Concepts du service IoT Hub Device Provisioning](concepts-service.md)