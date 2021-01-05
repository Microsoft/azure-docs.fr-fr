---
title: 'Démarrage rapide : Provisionner un appareil X.509 simulé auprès d’Azure IoT Hub à l’aide de Java'
description: 'Démarrage rapide Azure : Créer et provisionner un appareil X.509 simulé à l’aide du SDK d’appareil Java pour le service IoT Hub Device Provisioning (DPS). Ce guide de démarrage rapide utilise des inscriptions individuelles.'
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.devlang: java
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 2ed458ee0267a65bb7551940fc522509dc310311
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966613"
---
# <a name="quickstart-create-and-provision-a-simulated-x509-device-using-java-device-sdk-for-iot-hub-device-provisioning-service"></a>Démarrage rapide : Créer et approvisionner un appareil X.509 simulé auprès du service IoT Hub Device Provisioning à l’aide du Java Device SDK

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-x509](../../includes/iot-dps-selector-quick-create-simulated-device-x509.md)]

Dans ce guide de démarrage rapide, vous allez créer un appareil X.509 simulé sur un ordinateur Windows. Vous allez utiliser l’exemple de code Java de l’appareil pour connecter cet appareil simulé à votre hub IoT en utilisant une inscription individuelle auprès du service de provisionnement des appareils (DPS).

## <a name="prerequisites"></a>Prérequis

- Connaissance des concepts liés à l’[approvisionnement](about-iot-dps.md#provisioning-process).
- Exécution des étapes décrites dans [Configurer le service IoT Hub Device Provisioning avec le portail Azure](./quick-setup-auto-provision.md).
- Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Java SE Development Kit 8](/azure/developer/java/fundamentals/java-jdk-long-term-support).
- [Maven](https://maven.apache.org/install.html).
- [Git](https://git-scm.com/download/).

## <a name="prepare-the-environment"></a>Préparer l’environnement 

1. Assurez-vous que le [Java SE Development Kit 8](/azure/developer/java/fundamentals/java-jdk-long-term-support) est bien installé sur votre ordinateur.

2. Téléchargez et installez [Maven](https://maven.apache.org/install.html).

3. Assurez-vous que Git est installé sur votre machine et ajouté aux variables d’environnement accessibles à la fenêtre de commande. Consultez la section relative aux [outils clients de Software Freedom Conservancy](https://git-scm.com/download/) pour accéder à la dernière version des outils `git` à installer, qui inclut **Git Bash**, l’application de ligne de commande que vous pouvez utiliser pour interagir avec votre référentiel Git local. 

4. Ouvrez une invite de commandes. Clonez le référentiel GitHub pour l’exemple de code de simulation d’appareil :
    
    ```cmd/sh
    git clone https://github.com/Azure/azure-iot-sdk-java.git --recursive
    ```
5. Accédez au répertoire `azure-iot-sdk-`java racine et créez le projet pour télécharger tous les packages nécessaires.
   
   ```cmd/sh
   cd azure-iot-sdk-java
   mvn install -DskipTests=true
   ```
6. Accédez au projet de générateur de certificats et générez le projet. 

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-tools/provisioning-x509-cert-generator
    mvn clean install
    ```

## <a name="create-a-self-signed-x509-device-certificate-and-individual-enrollment-entry"></a>Créer un certificat d’appareil X.509 auto-signé et une entrée d’inscription individuelle

Dans cette section, vous allez utiliser un certificat X.509 auto-signé pour lequel il est important de garder à l’esprit les éléments suivants :

* Les certificats auto-signés sont destinés aux tests uniquement et ne doivent pas être utilisés en production.
* La date d’expiration par défaut d’un certificat auto-signé est d’un an.

Vous allez utiliser l’exemple de code du [Kit de développement logiciel (SDK) Azure IoT pour Java](https://github.com/Azure/azure-iot-sdk-java.git) pour créer le certificat à utiliser avec l’entrée d’inscription individuelle de l’appareil simulé.

Le service Azure IoT Device Provisioning prend en charge deux types d’inscriptions :

- [Groupes d’inscription](concepts-service.md#enrollment-group) : utilisés pour inscrire plusieurs appareils connexes.
- [Inscriptions individuelles](concepts-service.md#individual-enrollment) : utilisées pour inscrire un seul appareil.

Cet article présente les inscriptions individuelles.

1. À l’aide de l’invite de commandes des étapes précédentes, accédez au dossier `target`, puis exécutez le fichier .jar créé à l’étape précédente.

    ```cmd/sh
    cd target
    java -jar ./provisioning-x509-cert-generator-{version}-with-deps.jar
    ```

2. Entrez **N** pour _Do you want to input common name_ (Voulez-vous saisir le nom commun ?). Copiez dans le Presse-papiers la sortie de `Client Cert`, de *-----BEGIN CERTIFICATE-----* à *-----END CERTIFICATE-----* .

   ![Générateur de certificats individuels](./media/java-quick-create-simulated-device-x509/individual.png)

3. Créez un fichier nommé **_X509individual.pem_** sur votre ordinateur Windows, ouvrez-le dans l’éditeur de votre choix, puis copiez le contenu du Presse-papiers dans ce fichier. Enregistrez le fichier et fermez votre éditeur.

4. À l’invite de commandes, entrez **N** pour _Do you want to input Verification Code_ (Voulez-vous entrer le code de validation ?), puis gardez la sortie du programme ouverte pour vous en servir plus tard dans ce guide de démarrage rapide. Vous copierez plus tard les valeurs `Client Cert` et `Client Cert Private Key` pour les utiliser dans la section suivante.

5. Connectez-vous au [portail Azure](https://portal.azure.com), sélectionnez le bouton **Toutes les ressources** dans le menu de gauche et ouvrez votre instance de service Device Provisioning.

6. Dans le menu du service Device Provisioning, sélectionnez **Gérer les inscriptions**. Sélectionnez l’onglet **Inscriptions individuelles**, puis le bouton **Ajouter une inscription individuelle** dans la partie supérieure. 

7. Dans le volet **Ajouter une inscription**, entrez les informations suivantes :
   - Sélectionnez **X.509** comme *mécanisme* d’attestation d’identité.
   - Sous le *fichier .pem ou .cer du certificat principal*, choisissez *Select a file* (Sélectionner un fichier) pour sélectionner le fichier de certificat **X509individual.pem** créé au cours des étapes précédentes.  
   - Si vous le souhaitez, vous pouvez fournir les informations suivantes :
     - Sélectionnez un hub IoT lié à votre service d’approvisionnement.
     - Entrez un ID d’appareil unique. Veillez à éviter les données sensibles lorsque vous affectez un nom à votre appareil. 
     - Mettez à jour l’**état du jumeau d’appareil initial** à l’aide de la configuration initiale de votre choix pour l’appareil.
     - Cela fait, appuyez sur le bouton **Enregistrer**. 

     [![Ajouter une inscription individuelle pour l’attestation X.509 dans le portail](./media/java-quick-create-simulated-device-x509/device-enrollment.png)](./media/how-to-manage-enrollments/individual-enrollment.png#lightbox)

     Lorsque l’inscription aboutit, votre appareil X.509 apparaît en tant que **microsoftriotcore** sous la colonne *ID d’inscription* de l’onglet *Inscriptions individuelles*. 



## <a name="simulate-the-device"></a>Simuler l’appareil

1. À partir du menu Service Device Provisioning, sélectionnez **Vue d’ensemble**, puis notez les valeurs des champs _Étendue de l’ID_ et _Point de terminaison global du service de provisionnement_.

    ![Informations sur le service](./media/java-quick-create-simulated-device-x509/extract-dps-endpoints.png)

2. Ouvrez une invite de commandes. Accédez à l’exemple de dossier de projet du référentiel du Kit de développement logiciel (SDK) Java.

    ```cmd/sh
    cd azure-iot-sdk-java/provisioning/provisioning-samples/provisioning-X509-sample
    ```

3. Entrez les informations relatives à l’identité X.509 et au service d’approvisionnement dans votre code. Elles sont utilisées lors de l’approvisionnement automatique, pour l’attestation de l’appareil simulé, avant l’inscription de l’appareil :

   - Modifiez le fichier `/src/main/java/samples/com/microsoft/azure/sdk/iot/ProvisioningX509Sample.java` pour inclure les valeurs _Étendue de l’ID_ et _Provisioning Service Global Endpoint_ (Point de terminaison global du service d’approvisionnement) notées précédemment. Insérez également les valeurs _Client Cert_ (Certificat client) et _Client Cert Private Key_ (Clé privée du certificat client) déjà utilisées.

      ```java
      private static final String idScope = "[Your ID scope here]";
      private static final String globalEndpoint = "[Your Provisioning Service Global Endpoint here]";
      private static final ProvisioningDeviceClientTransportProtocol PROVISIONING_DEVICE_CLIENT_TRANSPORT_PROTOCOL = ProvisioningDeviceClientTransportProtocol.HTTPS;
      private static final String leafPublicPem = "<Your Public PEM Certificate here>";
      private static final String leafPrivateKey = "<Your Private PEM Key here>";
      ```

   - Utilisez le format suivant lorsque vous copiez/collez votre certificat et votre clé privée :
        
      ```java
      private static final String leafPublicPem = "-----BEGIN CERTIFICATE-----\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "+XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
        "-----END CERTIFICATE-----\n";
      private static final String leafPrivateKey = "-----BEGIN PRIVATE KEY-----\n" +
            "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
            "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\n" +
            "XXXXXXXXXX\n" +
            "-----END PRIVATE KEY-----\n";
      ```

4. Générez l’exemple. Accédez au dossier `target` et exécutez le fichier .jar créé.

    ```cmd/sh
    mvn clean install
    cd target
    java -jar ./provisioning-x509-sample-{version}-with-deps.jar
    ```

5. Dans le portail Azure, accédez au IoT Hub lié à votre service d’approvisionnement et ouvrez le panneau **Device Explorer**. En cas de réussite de l’approvisionnement de l’appareil simulé X.509 sur le Hub, son ID d’appareil s’affiche sur le panneau **Device Explorer**, avec un *ÉTAT* **activé**.  Notez que vous devrez peut-être appuyer sur le bouton **Actualiser** dans la partie supérieure si vous avez déjà ouvert le panneau avant d’exécuter l’exemple d’application de l’appareil. 

    ![L’appareil est inscrit avec le hub IoT](./media/java-quick-create-simulated-device-x509/hubregistration.png) 

> [!NOTE]
> Si vous avez modifié la valeur par défaut de l’*état du jumeau d’appareil initial* dans l’entrée d’inscription de votre appareil, l’état du jumeau souhaité peut être extrait du hub et agir en conséquence. Pour en savoir plus, consultez [Comprendre et utiliser les jumeaux d’appareil IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md).
>


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de manipuler et d’explorer davantage l’exemple de client d’appareil, ne nettoyez pas les ressources créées dans ce guide de démarrage rapide. Sinon, effectuez les étapes suivantes pour supprimer toutes les ressources créées par ce guide.

1. Fermez la fenêtre de sortie de l’exemple de client d’appareil sur votre machine.
2. Dans le menu de gauche du portail Azure, sélectionnez **Toutes les ressources**, puis votre service Device Provisioning. Ouvrez le panneau **Gérer les inscriptions** pour votre service, puis sélectionnez l’onglet **Inscriptions individuelles**. Cochez la case à côté de l’*ID D’INSCRIPTION* de l’appareil que vous avez inscrit dans ce guide de démarrage rapide, puis appuyez sur le bouton **Supprimer** dans la partie supérieure du volet. 
3. À partir du menu de gauche, dans le portail Azure, sélectionnez **Toutes les ressources**, puis votre hub IoT. Ouvrez le panneau **Appareils IoT** pour votre hub, cochez la case en regard de l’*ID D’APPAREIL* de l’appareil que vous avez inscrit dans ce guide de démarrage rapide, puis appuyez sur le bouton **Supprimer** dans la partie supérieure du volet.


## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un appareil X.509 simulé sur votre machine Windows. Vous avez configuré son inscription dans votre instance Azure IoT Hub Device Provisioning Service, puis approvisionné automatiquement l’appareil sur votre hub IoT. Pour apprendre à inscrire un appareil X.509 programmatiquement, passez au guide de démarrage rapide correspondant. 

> [!div class="nextstepaction"]
> [Démarrage rapide Azure - Inscrire des appareils X.509 auprès du service Azure IoT Hub Device Provisioning](quick-enroll-device-x509-java.md)