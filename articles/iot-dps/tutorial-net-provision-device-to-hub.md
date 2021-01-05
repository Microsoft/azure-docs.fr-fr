---
title: Tutoriel - Provisionner un appareil à l’aide du service Azure IoT Hub Device Provisioning (.NET)
description: Ce tutoriel montre comment provisionner votre appareil sur un seul hub IoT à l’aide du service Azure IoT Hub Device Provisioning avec .NET.
author: wesmc7777
ms.author: wesmc
ms.date: 11/12/2019
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.devlang: csharp
ms.custom: mvc, devx-track-csharp
ms.openlocfilehash: f9a14ee6ee3e10b36d64ec11fc23807efe2bfaf2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966562"
---
# <a name="tutorial-enroll-the-device-to-an-iot-hub-using-the-azure-iot-hub-provisioning-service-client-net"></a>Tutoriel : Inscrire l’appareil à un hub IoT avec le client du service IoT Hub Provisioning (.NET)

Dans le didacticiel précédent, vous avez appris à configurer un appareil pour vous connecter à votre service Device Provisioning. Dans ce didacticiel, vous allez apprendre à utiliser ce service pour approvisionner votre appareil sur un seul hub IoT à l’aide d’une **_inscription individuelle_** et des **_groupes d’inscriptions_** . Ce didacticiel vous explique les procédures suivantes :

> [!div class="checklist"]
> * Inscrire l’appareil
> * Démarrer l’appareil
> * Vérifier que l’appareil est enregistré

## <a name="prerequisites"></a>Prérequis

Avant de continuer, assurez-vous de configurer votre appareil et son *module de sécurité matériel* comme indiqué dans le didacticiel [Configurer un appareil à approvisionner à l’aide du service IoT Hub Device Provisioning](./tutorial-set-up-device.md).

* Visual Studio

> [!NOTE]
> Visual Studio n’est pas nécessaire. L’installation de [.NET](https://www.microsoft.com/net) suffit, et les développeurs peuvent utiliser leur éditeur préféré sur Windows ou Linux.  

Ce didacticiel simule la période du processus de fabrication de matériel ou juste après, quand les informations de l’appareil sont ajoutées au service d’approvisionnement. Ce code est généralement exécuté sur un PC ou un appareil d’usine qui peut exécuter du code .NET et ne doit pas être ajouté aux appareils eux-mêmes.


## <a name="enroll-the-device"></a>Inscrire l’appareil

Cette étape implique l’ajout des artefacts de sécurité uniques de l’appareil au service Device Provisioning. Ces artefacts de sécurité sont les suivants :

- Pour les appareils TPM :
    - La *paire de clés de type EK* qui est unique pour chaque simulation ou processeur TPM. Pour plus d’informations, consultez [Comprendre la paire de clés de type EK (Endorsement Key) du module de plateforme sécurisée](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc770443(v=ws.11)).
    - *L’ID d’enregistrement* qui est utilisé pour identifier un appareil dans l’espace de noms ou l’étendue. Cet ID peut être ou non le même que l’ID de l’appareil. L’ID est obligatoire pour chaque appareil. Pour les appareils basés sur TPM, l’ID d’enregistrement peut être dérivé du module TPM lui-même, par exemple un hachage SHA-256 de la paire de clés de type EK TPM.

- Pour les appareils basés sur X.509 :
    - Le [certificat X.509 délivré à l’appareil](/windows/win32/seccertenroll/about-x-509-public-key-certificates), sous la forme d’un fichier *.pem* ou *.cer*. Pour une inscription individuelle, vous devez utiliser le *certificat feuille* de votre système X.509, tandis que pour des groupes d’inscriptions, vous devez utiliser le *certificat racine* ou un *certificat du signataire* équivalent.
    - *L’ID d’enregistrement* qui est utilisé pour identifier un appareil dans l’espace de noms ou l’étendue. Cet ID peut être ou non le même que l’ID de l’appareil. L’ID est obligatoire pour chaque appareil. Pour les appareils basés sur X.509, l’ID d’inscription est dérivé du nom commun du certificat. Pour plus d’informations sur ces exigences, consultez [Concepts d’appareil du service IoT Hub Device Provisioning](./concepts-service.md).

Il existe deux façons d’inscrire l’appareil auprès du service Device Provisioning :

- **Inscription individuelle** : représente une entrée d’un seul appareil pouvant être enregistré auprès du service Device Provisioning. Les inscriptions individuelles peuvent utiliser des certificats x509 ou des jetons SAP (dans un module TPM réel ou virtuel) comme mécanismes d’attestation. Nous vous recommandons d’utiliser des inscriptions individuelles pour les appareils qui nécessitent une configuration initiale unique ou pour les appareils qui peuvent uniquement utiliser des jetons SAP par le biais du Module de plateforme sécurisée (TPM) comme mécanisme d’attestation. Dans le cas d’inscriptions individuelles, vous pouvez spécifier l’ID de l’appareil IoT Hub souhaité.

- **Groupe d’inscriptions** : représente un groupe d’appareils qui partagent un mécanisme d’attestation spécifique. Nous recommandons d’utiliser un groupe d’inscriptions pour un grand nombre d’appareils qui partagent une configuration initiale souhaitée ou pour des appareils destinés au même locataire. Les groupes d’inscription sont X.509 uniquement, et partagent tous un certificat de signature dans leur chaîne de certificats X.509.

### <a name="enroll-the-device-using-individual-enrollments"></a>Inscrire l’appareil à l’aide des inscriptions individuelles

1. Dans Visual Studio, créez un projet d’application de console Visual C# à l’aide du modèle de projet **d’application de console**. Nommez le projet **DeviceProvisioning**.
    
1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **DeviceProvisioning**, puis cliquez sur **Gérer les packages NuGet...** .

1. Dans la fenêtre **Gestionnaire de package NuGet**, sélectionnez **Parcourir**, puis recherchez **microsoft.azure.devices.provisioning.service**. Sélectionnez l’entrée et cliquez sur **Installer** pour installer le package **Microsoft.Azure.Devices.Provisioning.Service**, puis acceptez les conditions d’utilisation. Cette procédure télécharge, installe et ajoute une référence au package NuGet [Azure IoT Device Provisioning Service SDK](https://www.nuget.org/packages/Microsoft.Azure.Devices.Provisioning.Service/) et ses dépendances.

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :
   
    ```csharp
    using Microsoft.Azure.Devices.Provisioning.Service;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur d’espace réservé par la chaîne de connexion du service Device Provisioning que vous avez notée à la section précédente.
   
    ```csharp
    static readonly string ServiceConnectionString = "{Device Provisioning Service connection string}";

    private const string SampleRegistrationId = "sample-individual-csharp";
    private const string SampleTpmEndorsementKey =
            "AToAAQALAAMAsgAgg3GXZ0SEs/gakMyNRqXXJP1S124GUgtk8qHaGzMUaaoABgCAAEMAEAgAAAAAAAEAxsj2gUS" +
            "cTk1UjuioeTlfGYZrrimExB+bScH75adUMRIi2UOMxG1kw4y+9RW/IVoMl4e620VxZad0ARX2gUqVjYO7KPVt3d" +
            "yKhZS3dkcvfBisBhP1XH9B33VqHG9SHnbnQXdBUaCgKAfxome8UmBKfe+naTsE5fkvjb/do3/dD6l4sGBwFCnKR" +
            "dln4XpM03zLpoHFao8zOwt8l/uP3qUIxmCYv9A7m69Ms+5/pCkTu/rK4mRDsfhZ0QLfbzVI6zQFOKF/rwsfBtFe" +
            "WlWtcuJMKlXdD8TXWElTzgh7JS4qhFzreL0c1mI0GCj+Aws0usZh7dLIVPnlgZcBhgy1SSDQMQ==";
    private const string OptionalDeviceId = "myCSharpDevice";
    private const ProvisioningStatus OptionalProvisioningStatus = ProvisioningStatus.Enabled;
    ```

1. Ajoutez le code suivant pour implémenter l’inscription de l’appareil :

    ```csharp
    static async Task SetRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetRegistrationData");

        Attestation attestation = new TpmAttestation(SampleTpmEndorsementKey);

        IndividualEnrollment individualEnrollment = new IndividualEnrollment(SampleRegistrationId, attestation);

        individualEnrollment.DeviceId = OptionalDeviceId;
        individualEnrollment.ProvisioningStatus = OptionalProvisioningStatus;

        Console.WriteLine("\nAdding new individualEnrollment...");
        var serviceClient = ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString);

        IndividualEnrollment individualEnrollmentResult =
            await serviceClient.CreateOrUpdateIndividualEnrollmentAsync(individualEnrollment).ConfigureAwait(false);

        Console.WriteLine("\nIndividualEnrollment created with success.");
        Console.WriteLine(individualEnrollmentResult);
    }
    ```

1. Enfin, ajoutez le code suivant à la méthode **Main** pour ouvrir la connexion à votre hub IoT et commencer l’inscription :
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Provisioning example");

        SetRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    Console.ReadLine();
    ```
        
1. Dans l’Explorateur de solutions de Visual Studio, cliquez avec le bouton droit sur votre solution, puis sur **Définir les projets de démarrage...** . Sélectionnez **Projet de démarrage unique**, puis sélectionnez le projet **DeviceProvisioning** dans le menu déroulant.  

1. Exécutez l’application pour l’appareil .NET **DeviceProvisioning**. Elle doit configurer l’approvisionnement de l’appareil : 

    ![Exécution de l’inscription individuelle](./media/tutorial-net-provision-device-to-hub/individual.png)

Une fois l’appareil correctement inscrit, il doit apparaître comme suit dans le portail :

   ![Inscription réussie dans le portail](./media/tutorial-net-provision-device-to-hub/individual-portal.png)

### <a name="enroll-the-device-using-enrollment-groups"></a>Inscrire l’appareil à l’aide des groupes d’inscriptions

> [!NOTE]
> L’exemple de groupe d’inscriptions nécessite un certificat X.509.

1. Dans l’Explorateur de solutions Visual Studio, ouvrez le projet **DeviceProvisioning** créé ci-dessus. 

1. Ajoutez les instructions `using` suivantes en haut du fichier **Program.cs** :
    
    ```csharp
    using System.Security.Cryptography.X509Certificates;
    ```

1. Ajoutez les champs suivants à la classe **Program** . Remplacez la valeur d’espace réservé par l’emplacement du certificat X509.
   
    ```csharp
    private const string X509RootCertPathVar = "{X509 Certificate Location}";
    private const string SampleEnrollmentGroupId = "sample-group-csharp";
    ```

1. Ajoutez le code suivant à **Program.cs** pour implémenter l’inscription du groupe :

    ```csharp
    public static async Task SetGroupRegistrationDataAsync()
    {
        Console.WriteLine("Starting SetGroupRegistrationData");

        using (ProvisioningServiceClient provisioningServiceClient =
                ProvisioningServiceClient.CreateFromConnectionString(ServiceConnectionString))
        {
            Console.WriteLine("\nCreating a new enrollmentGroup...");

            var certificate = new X509Certificate2(X509RootCertPathVar);

            Attestation attestation = X509Attestation.CreateFromRootCertificates(certificate);

            EnrollmentGroup enrollmentGroup = new EnrollmentGroup(SampleEnrollmentGroupId, attestation);

            Console.WriteLine(enrollmentGroup);
            Console.WriteLine("\nAdding new enrollmentGroup...");

            EnrollmentGroup enrollmentGroupResult =
                await provisioningServiceClient.CreateOrUpdateEnrollmentGroupAsync(enrollmentGroup).ConfigureAwait(false);

            Console.WriteLine("\nEnrollmentGroup created with success.");
            Console.WriteLine(enrollmentGroupResult);
        }
    }
    ```

1. Enfin, remplacez le code suivant par la méthode **Main** pour ouvrir la connexion à votre hub IoT et commencer l’inscription du groupe :
   
    ```csharp
    try
    {
        Console.WriteLine("IoT Device Group Provisioning example");

        SetGroupRegistrationDataAsync().GetAwaiter().GetResult();
            
        Console.WriteLine("Done, hit enter to exit.");
        Console.ReadLine();
    }
    catch (Exception ex)
    {
        Console.WriteLine();
        Console.WriteLine("Error in sample: {0}", ex.Message);
    }
    ```

1. Exécutez l’application pour l’appareil .NET **DeviceProvisioning**. Elle doit configurer l’approvisionnement du groupe pour l’appareil : 

    ![Exécution de l’inscription du groupe](./media/tutorial-net-provision-device-to-hub/group.png)

    Une fois le groupe d’appareils correctement inscrit, il doit apparaître comme suit dans le portail :

   ![Inscription du groupe réussie dans le portail](./media/tutorial-net-provision-device-to-hub/group-portal.png)


## <a name="start-the-device"></a>Démarrer l’appareil

À ce stade, la configuration suivante est prête pour l’enregistrement de l’appareil :

1. Votre appareil ou groupe d’appareils est inscrit auprès de votre service Device Provisioning. 
2. votre appareil est prêt avec la sécurité configurée et accessible via l’application à l’aide du kit de développement logiciel (SDK) client du service Device Provisioning.

Démarrez l’appareil pour autoriser votre application cliente à démarrer l’enregistrement auprès de votre service Device Provisioning.  


## <a name="verify-the-device-is-registered"></a>Vérifier que l’appareil est enregistré

Une fois que votre appareil démarre, les actions suivantes doivent se produire. Consultez l’article relatif à [l’approvisionnement d’un exemple d’appareil client](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/provisioning/Samples/device) pour plus d’informations. 

1. L’appareil envoie une demande d’enregistrement à votre service Device Provisioning.
2. Pour les appareils TPM, le service Device Provisioning envoie une demande d’enregistrement à laquelle répond votre appareil. 
3. Une fois l’inscription réussie, le service Device Provisioning envoie l’URI du hub IoT, l’ID de l’appareil et la clé chiffrée à l’appareil. 
4. L’application cliente IoT Hub sur l’appareil peut alors se connecter à votre hub. 
5. Une fois la connexion au hub établie, l’appareil doit apparaître dans **Device Explorer** dans le hub IoT. 

    ![Connexion réussie au hub dans le portail](./media/tutorial-net-provision-device-to-hub/hub-connect-success.png)

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Inscrire l’appareil
> * Démarrer l’appareil
> * Vérifier que l’appareil est enregistré

Passez au didacticiel suivant pour apprendre à approvisionner plusieurs appareils sur des hubs à charge équilibrée. 

> [!div class="nextstepaction"]
> [Approvisionner des appareils sur des hubs IoT à charge équilibrée](./tutorial-provision-multiple-hubs.md)