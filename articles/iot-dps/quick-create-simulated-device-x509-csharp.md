---
title: 'Démarrage rapide : Provisionner un appareil X.509 simulé auprès d’Azure IoT Hub à l’aide de C#'
description: 'Démarrage rapide : Créer et provisionner un appareil X.509 simulé auprès du service Azure IoT Hub Device Provisioning (DPS) à l’aide du SDK d’appareil C#. Ce démarrage rapide utilise des inscriptions individuelles.'
author: wesmc7777
ms.author: wesmc
ms.date: 11/08/2018
ms.topic: quickstart
ms.service: iot-dps
services: iot-dps
ms.devlang: csharp
ms.custom: mvc
ms.openlocfilehash: 27bb1c97fa082f15642ab9eff6b0bdba357068a2
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91323972"
---
# <a name="quickstart-create-and-provision-a-simulated-x509-device-using-c-device-sdk-for-iot-hub-device-provisioning-service"></a>Démarrage rapide : Créer et approvisionner un appareil X.509 simulé auprès du service IoT Hub Device Provisioning à l’aide du C# Device SDK

[!INCLUDE [iot-dps-selector-quick-create-simulated-device-x509](../../includes/iot-dps-selector-quick-create-simulated-device-x509.md)]

Ces étapes expliquent comment utiliser les [échantillons Azure IoT pour C#](https://github.com/Azure-Samples/azure-iot-samples-csharp) afin de simuler un appareil X.509 sur une machine de développement exécutant le système d'exploitation Windows. L'échantillon connecte également l'appareil simulé à un IoT Hub à l'aide de l'instance de Device Provisioning Service.

Si vous ne connaissez pas le processus de provisionnement automatique, veuillez consulter la vue d’ensemble de l’[approvisionnement](about-iot-dps.md#provisioning-process). Vérifiez également que vous avez suivi la procédure décrite dans [Configurer le service d’approvisionnement d’appareil IoT Hub avec le portail Azure](./quick-setup-auto-provision.md) avant de continuer. 

Le service Azure IoT Device Provisioning prend en charge deux types d’inscriptions :
- [Groupes d’inscription](concepts-service.md#enrollment-group) : utilisés pour inscrire plusieurs appareils connexes.
- [Inscriptions individuelles](concepts-service.md#individual-enrollment) : utilisées pour inscrire un seul appareil.

Cet article présente les inscriptions individuelles.

[!INCLUDE [IoT Device Provisioning Service basic](../../includes/iot-dps-basic.md)]

<a id="setupdevbox"></a>
## <a name="prepare-the-development-environment"></a>Préparer l’environnement de développement 

1. Vérifiez que le kit [SDK .NET Core 2.1 ou version ultérieure](https://www.microsoft.com/net/download/windows) est installé sur votre machine. 

1. Assurez-vous que l’élément `git` est installé sur votre machine et est ajouté aux variables d’environnement accessibles à la fenêtre de commande. Consultez la section relative aux [outils clients de Software Freedom Conservancy](https://git-scm.com/download/) pour accéder à la dernière version des outils `git` à installer, qui inclut **Git Bash**, l’application de ligne de commande que vous pouvez utiliser pour interagir avec votre référentiel Git local. 

1. Ouvrez une invite de commandes ou Git Bash. Clonez les échantillons Azure IoT pour le référentiel C# GitHub :
    
    ```cmd
    git clone https://github.com/Azure-Samples/azure-iot-samples-csharp.git
    ```

## <a name="create-a-self-signed-x509-device-certificate-and-individual-enrollment-entry"></a>Créer un certificat d’appareil X.509 auto-signé et une entrée d’inscription individuelle

Dans cette section, vous allez utiliser un certificat X.509 auto-signé pour lequel il est important de garder à l’esprit les éléments suivants :

* Les certificats auto-signés sont destinés aux tests uniquement et ne doivent pas être utilisés en production.
* La date d’expiration par défaut d’un certificat auto-signé est d’un an.

Vous allez utiliser l’exemple de code [Exemple de client d'appareil d'approvisionnement - X.509 Attestation](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/device/X509Sample)pour créer le certificat à utiliser avec l’entrée d’inscription individuelle de l’appareil simulé.


1. Dans une invite de commandes, accédez au répertoire du projet contenant l’exemple d’approvisionnement d’appareil X.509.

    ```cmd
    cd .\azure-iot-samples-csharp\provisioning\Samples\device\X509Sample
    ```

2. Cet exemple de code est configuré pour utiliser des certificats X.509 stockés dans un fichier PKCS12 protégé par mot de passe (certificate.pfx). En outre, vous avez besoin d’un fichier de certificat de clé publique (certificate.cer) pour créer par la suite, dans ce guide de démarrage rapide, une inscription individuelle. Pour générer un certificat auto-signé et ses fichiers .cer et .pfx associés, exécutez la commande suivante :

    ```cmd
    powershell .\GenerateTestCertificate.ps1
    ```

3. Le script vous invite à saisir un mot de passe PFX. N’oubliez pas ce mot de passe. Il est indispensable lorsque vous exécutez l’exemple.

    ![ Saisir le mot de passe](./media/quick-create-simulated-device-x509-csharp/generate-certificate.png)  


4. Connectez-vous au portail Azure, sélectionnez **Toutes les ressources** dans le menu de gauche et ouvrez votre service de provisionnement.

5. Dans le menu du service Device Provisioning, sélectionnez **Gérer les inscriptions**. Sélectionnez l’onglet **Inscriptions individuelles**, puis le bouton **Ajouter une inscription individuelle** dans la partie supérieure. 

6. Dans le volet **Ajouter une inscription**, entrez les informations suivantes :
   - Sélectionnez **X.509** comme *mécanisme* d’attestation d’identité.
   - Sous *Fichier .pem ou .cer du certificat principal*, choisissez *Sélectionner un fichier* pour sélectionner le fichier de certificat **certificate.cer** créé au cours des étapes précédentes.
   - Ne renseignez pas le champ **ID de l’appareil**. Votre appareil va être approvisionné. Son ID est défini sur le nom commun (CN) dans le certificat X.509, soit **iothubx509device1**. Ce nom sera également utilisé pour l’ID d’inscription de l’entrée d’inscription individuelle. 
   - Si vous le souhaitez, vous pouvez fournir les informations suivantes :
       - Sélectionnez un hub IoT lié à votre service d’approvisionnement.
       - Mettez à jour l’**état du jumeau d’appareil initial** à l’aide de la configuration initiale de votre choix pour l’appareil.
   - Cela fait, appuyez sur le bouton **Enregistrer**. 

     [![Ajouter une inscription individuelle pour l’attestation X.509 dans le portail](./media/quick-create-simulated-device-x509-csharp/device-enrollment.png)](./media/quick-create-simulated-device-x509-csharp/device-enrollment.png#lightbox)
    
   Lorsque l’opération aboutit, l’entrée d’inscription de votre appareil X.509 apparaît en tant que **iothubx509device1** sous la colonne *ID d’inscription* de l’onglet *Inscriptions individuelles*. 

## <a name="provision-the-simulated-device"></a>Approvisionner l’appareil simulé

1. Dans le panneau **Vue d’ensemble** de votre service de provisionnement, notez la valeur **_Étendue de l’ID_** .

    ![Extraction des informations de point de terminaison du service Device Provisioning à partir du panneau du Portail](./media/quick-create-simulated-device-x509-csharp/copy-scope.png) 


2. Tapez la commande suivante pour générer et exécuter l’exemple d’approvisionnement d’appareil X.509. Remplacez la valeur `<IDScope>` par l’étendue de l’ID de votre service d’approvisionnement. 

    ```cmd
    dotnet run <IDScope>
    ```

3. Lorsque vous y êtes invité, entrez le mot de passe du fichier PFX que vous avez créé précédemment. Notez les messages qui simulent le démarrage et la connexion de l’appareil au service d’approvisionnement d’appareil pour obtenir des informations concernant votre IoT Hub. 

    ![Exemple de sortie d’appareil](./media/quick-create-simulated-device-x509-csharp/sample-output.png) 

4. Vérifiez que l’appareil a bien été approvisionné. En cas de réussite du provisionnement de l’appareil simulé auprès du hub IoT lié à votre service de provisionnement, l’ID d’appareil s’affiche sur le panneau **Appareils IoT** du hub. 

    ![L’appareil est inscrit avec le hub IoT](./media/quick-create-simulated-device-x509-csharp/registration.png) 

    Si vous avez modifié la valeur par défaut de l’*état du jumeau d’appareil initial* dans l’entrée d’inscription de votre appareil, l’état du jumeau souhaité peut être extrait du hub et agir en conséquence. Pour en savoir plus, consultez [Comprendre et utiliser les jumeaux d’appareil IoT Hub](../iot-hub/iot-hub-devguide-device-twins.md)


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de manipuler et d’explorer davantage l’exemple de client d’appareil, ne nettoyez pas les ressources créées dans ce guide de démarrage rapide. Sinon, effectuez les étapes suivantes pour supprimer toutes les ressources créées par ce guide.

1. Fermez la fenêtre de sortie de l’exemple de client d’appareil sur votre machine.
1. Fermez la fenêtre du simulateur TPM sur votre machine.
1. Dans le menu de gauche du portail Azure, sélectionnez **Toutes les ressources**, puis votre service Device Provisioning. Dans la partie supérieure du panneau **Vue d’ensemble**, appuyez sur **Supprimer** en haut du volet.  
1. À partir du menu de gauche, dans le portail Azure, sélectionnez **Toutes les ressources**, puis votre hub IoT. Dans la partie supérieure du panneau **Vue d’ensemble**, appuyez sur **Supprimer** en haut du volet.  

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un appareil X.509 simulé sur un ordinateur Windows et l’avez provisionné auprès de votre hub IoT à l’aide du service Azure IoT Hub Device Provisioning sur le portail. Pour apprendre à inscrire un appareil X.509 programmatiquement, passez au guide de démarrage rapide correspondant. 

> [!div class="nextstepaction"]
> [Démarrage rapide Azure - Inscrire des appareils X.509 auprès du service Azure IoT Hub Device Provisioning](quick-enroll-device-x509-csharp.md)
