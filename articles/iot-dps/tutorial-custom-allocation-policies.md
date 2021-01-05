---
title: Tutoriel sur l’utilisation des stratégies d’allocation personnalisées avec le service Azure IoT Hub Device Provisioning Service (DPS)
description: Tutoriel sur l’utilisation des stratégies d’allocation personnalisées avec le service Azure IoT Hub Device Provisioning (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 09/23/2020
ms.topic: tutorial
ms.service: iot-dps
services: iot-dps
ms.custom: mvc
ms.openlocfilehash: 4cab1765a387bbae61c9c242a8e7a1ca881ea1f5
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966647"
---
# <a name="tutorial-use-custom-allocation-policies-with-device-provisioning-service-dps"></a>Tutoriel : Utiliser des stratégies d’allocation personnalisées avec le service DPS (Device Provisioning Service)

Une stratégie d’allocation personnalisée vous permet de mieux contrôler le processus d’assignation des appareils à un hub IoT. Cette assignation s’effectue par l’utilisation d’un code personnalisé dans une [Fonction Azure](../azure-functions/functions-overview.md) qui s’exécute pendant le provisionnement. Le service Device provisioning appelle votre code de fonction Azure en fournissant toutes les informations appropriées sur l’appareil et l’inscription. Votre code de fonction est exécuté et retourne les informations sur les hubs IoT utilisés pour le provisionnement de l’appareil.

Avec les stratégies d’allocation personnalisées, vous définissez vos propres stratégies d’allocation quand les stratégies fournies par le service Device Provisioning ne sont pas adaptées aux besoins de votre scénario.

Par exemple, vous souhaitez peut-être examiner le certificat utilisé par un appareil pendant le provisionnement et assigner l’appareil à un hub IoT en fonction d’une propriété du certificat. Ou, vous avez peut-être une base de données contenant des informations sur vos appareils et souhaitez interroger la base de données pour déterminer à quel hub IoT un appareil doit être assigné.

Cet article présente un groupe d’inscription avec une stratégie d’allocation personnalisée qui utilise une Fonction Azure écrite en C# pour provisionner des appareils Toaster à l’aide de clés symétriques. Tout appareil non reconnu comme appareil Toaster n’est pas provisionné sur un hub IoT.

Ces appareils demanderont un provisionnement à l’aide d’un exemple de code de provisionnement inclus dans le [SDK Azure IoT pour C](https://github.com/Azure/azure-iot-sdk-c).


Dans ce tutoriel, vous allez effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une application Azure Functions pour héberger la fonction d’allocation personnalisée
> * Créer une inscription de groupe en ajoutant du code Azure Functions pour la stratégie d’allocation personnalisée
> * Créer des clés d’appareil pour deux appareils
> * Configurer l’environnement de développement pour l’exemple de code d’appareil à partir du [SDK Azure IoT pour C](https://github.com/Azure/azure-iot-sdk-c)
> * Exécuter les appareils et vérifier qu’ils sont provisionnés selon la stratégie d’allocation personnalisée


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Cet article suppose que vous avez suivi la procédure décrite dans [Configurer le service IoT Hub Device Provisioning avec le portail Azure](./quick-setup-auto-provision.md) pour créer votre hub IoT et votre instance DPS.

* Dernière version de [Git](https://git-scm.com/download/) installée.

* Pour un environnement de développement Windows, [Visual Studio](https://visualstudio.microsoft.com/vs/) 2019 avec la charge de travail [« Desktop Development with C++ »](/cpp/ide/using-the-visual-studio-ide-for-cpp-desktop-development) activée est nécessaire. Visual Studio 2015 et Visual Studio 2017 sont également pris en charge.

* Pour Linux ou macOS, consultez la rubrique appropriée de la section [Préparer votre environnement de développement](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) dans la documentation du [SDK Azure IoT C](https://github.com/Azure/azure-iot-sdk-c).


[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-custom-allocation-function"></a>Créer la fonction d’allocation personnalisée

Dans cette section, vous allez créer une fonction Azure qui implémente votre stratégie d’allocation personnalisée. Cette fonction détermine si un appareil doit être inscrit sur votre hub IoT, selon que son ID d’inscription contient le préfixe de chaîne **contoso-toaster**.

1. Connectez-vous au [portail Azure](https://portal.azure.com). Dans votre page d’accueil, sélectionnez **+ Créer une ressource**.

2. Dans la zone de recherche *Rechercher dans la Place de marché*, tapez « Application de fonction ». Dans la liste déroulante, sélectionnez **Application de fonction**, puis sélectionnez **Créer**.

3. Dans la page de création **Application de fonction**, sous l’onglet **Options de base**, entrez les paramètres suivants pour votre nouvelle application de fonction et sélectionnez **Vérifier + créer** :

    **Abonnement**: Si vous avez plusieurs abonnements et que l’abonnement souhaité n’est pas sélectionné, sélectionnez celui que vous souhaitez utiliser.

    **Groupe de ressources** : Ce champ vous permet de créer un groupe de ressources ou de sélectionner un groupe déjà existant pour contenir l’application de fonction. Choisissez le même groupe de ressources qui contient l’IoT Hub que vous avez créé pour le test précédemment, par exemple, **TestResources**. En plaçant toutes les ressources associées dans un même groupe, vous pouvez les gérer toutes ensemble.

    **Nom de l’application de fonction** : entrez un nom unique pour votre application de fonction. Cet exemple utilise **contoso-function-app**.

    **Publier** : Vérifiez que **Code** est sélectionné.

    **Pile d’exécution** : Sélectionnez **.NET Core** dans la liste déroulante.

    **Région** : Sélectionnez la même région que votre groupe de ressources. Cet exemple utilise la région **USA Ouest**.

    > [!NOTE]
    > Par défaut, Application Insights est activé. Application Insights n’est pas nécessaire pour cet article, mais il peut vous aider à comprendre et à examiner les problèmes que vous rencontrez avec l’allocation personnalisée. Si vous préférez, vous pouvez désactiver les Application Insights en sélectionnant l’onglet **Surveillance**, puis en sélectionnant **Non** pour **Activer Application Insights**.

    ![Créer une application de fonction Azure pour héberger la fonction d’allocation personnalisée](./media/tutorial-custom-allocation-policies/create-function-app.png)

4. Dans la page **Résumé**, sélectionnez **Créer** pour créer l’application de fonction. Le déploiement peut prendre plusieurs minutes. Une fois l’opération terminée, sélectionnez **Accéder à la ressource**.

5. Dans le volet gauche, sous **Fonctions**, cliquez sur **Fonctions** puis sur **+ Ajouter** pour ajouter une nouvelle fonction.

6. Sur la page des modèles, sélectionnez la vignette **Déclencheur HTTP** , puis **Créer une fonction**. Une fonction nommée **HttpTrigger1** est créée, et le portail affiche la page de page de présentation de votre fonction.

7. Cliquez sur **Code + Test** pour votre nouvelle fonction. Le portail affiche le contenu du fichier de code **run.csx** . 

8. Remplacez le code de la fonction **HttpTrigger1** par le code suivant, puis sélectionnez **Enregistrer**. Votre code d’allocation personnalisé est prêt à être utilisé.

    ```csharp
    #r "Newtonsoft.Json"

    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        // Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);

        log.LogInformation("Request.Body:...");
        log.LogInformation(requestBody);

        // Get registration ID of the device
        string regId = data?.deviceRuntimeContext?.registrationId;

        string message = "Uncaught error";
        bool fail = false;
        ResponseObj obj = new ResponseObj();

        if (regId == null)
        {
            message = "Registration ID not provided for the device.";
            log.LogInformation("Registration ID : NULL");
            fail = true;
        }
        else
        {
            string[] hubs = data?.linkedHubs.ToObject<string[]>();

            // Must have hubs selected on the enrollment
            if (hubs == null)
            {
                message = "No hub group defined for the enrollment.";
                log.LogInformation("linkedHubs : NULL");
                fail = true;
            }
            else
            {
                // This is a Contoso Toaster 
                if (regId.Contains("contoso-toaster"))
                {
                    //Log IoT hubs configured for the enrollment
                    foreach(string hubString in hubs)
                    {
                        log.LogInformation("linkedHub : " + hubString);
                    }

                    obj.iotHubHostName = hubs[0];
                    log.LogInformation("Selected hub : " + obj.iotHubHostName);
                }
                else
                {
                    fail = true;
                    message = "Unrecognized device registration.";
                    log.LogInformation("Unknown device registration");
                }
            }
        }

        log.LogInformation("\nResponse");
        log.LogInformation((obj.iotHubHostName != null) ? JsonConvert.SerializeObject(obj) : message);

        return (fail)
            ? new BadRequestObjectResult(message) 
            : (ActionResult)new OkObjectResult(obj);
    }

    public class ResponseObj
    {
        public string iotHubHostName {get; set;}
    }
    ```

9. Juste en dessous du bas du fichier de code **run.csx**, cliquez sur **Journaux** pour surveiller la journalisation à partir de la fonction d’allocation personnalisée. 


## <a name="create-the-enrollment"></a>Créer l’inscription

Dans cette section, vous allez créer un groupe d’inscriptions qui utilise la stratégie d’allocation personnalisée. Par souci de simplicité, cet article utilise [l’attestation de clé symétrique](concepts-symmetric-key-attestation.md) avec l’inscription. Pour sécuriser votre solution, utilisez plutôt [l’attestation de certificat X.509](concepts-x509-attestation.md) avec une chaîne d’approbation.

1. Toujours dans le [Portail Azure](https://portal.azure.com), ouvrez votre service d’approvisionnement.

2. Sélectionnez **Gérer les inscriptions** dans le volet de gauche, puis sélectionnez le bouton **Ajouter un groupe d’inscriptions** en haut de la page.

3. Dans **Ajouter un groupe d’inscriptions**, entrez les informations du tableau ci-dessous, puis cliquez sur le bouton **Enregistrer**.

    | Champ | Description et/ou valeur suggérée |
    | :---- | :----------------------------- |
    | **Nom du groupe** | Entrez **contoso-custom-allocated-devices** |
    | **Type d’attestation** | Sélectionnez **Clé symétrique** |
    | **Générer automatiquement des clés** | cette case est normalement déjà cochée. |
    | **Sélectionner le mode d’affectation des appareils aux hubs** | Sélectionnez **Personnalisé (utiliser la fonction Azure)** |
    | **Sélectionner les hubs IoT auxquels ce groupe peut être attribué** | Sélectionnez le hub IoT que vous avez créé précédemment lorsque vous avez terminé le guide de démarrage rapide. |
    | **Sélectionner une fonction Azure** | Sélectionnez l’abonnement qui contient l’application de fonction que vous avez créée. Sélectionnez ensuite **contoso-function-app** et **HttpTrigger1** pour la fonction. |

    ![Ajouter un groupe d’inscription d’allocation personnalisée pour l’attestation de clé symétrique](./media/tutorial-custom-allocation-policies/create-custom-allocation-enrollment.png)

4. Après avoir enregistré l’inscription, rouvrez-la et notez la valeur de la **Clé primaire**. Vous devez d’abord enregistrer l’inscription pour permettre la génération des clés. Cette clé symétrique primaire sera utilisée pour générer des clés d’appareil uniques pour les appareils qui tentent un provisionnement ultérieurement. 

## <a name="derive-unique-device-keys"></a>Dériver les clés d’appareil uniques

Les appareils n’utilisent pas la clé symétrique primaire directement. Au lieu de cela, vous utilisez la clé primaire pour dériver une clé d’appareil pour chaque appareil. Dans cette section, vous créez deux clés d’appareil uniques. Une clé sera utilisée pour l’appareil simulé Toaster. L’autre clé sera utilisée pour l’appareil simulé Heat pump. Les clés générées permettront aux deux appareils de tenter une inscription. Un seul ID d’inscription d’appareil aura un suffixe valide devant être accepté par l’exemple de code de stratégie d’allocation personnalisée. En conséquence, l’un est accepté et l’autre rejeté

Pour dériver la clé d’appareil, utilisez la clé symétrique que vous avez notée précédemment pour calculer le code [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) de l’ID d’inscription de l’appareil pour chaque appareil, puis convertissez le résultat au format Base64. Pour plus d’informations sur la création de clés d’appareil dérivées avec des groupes d’inscription, consultez la section sur les inscriptions de groupes dans [Attestation de clé symétrique](concepts-symmetric-key-attestation.md).

Pour l’exemple de cet article, utilisez les ID d’inscription des deux appareils suivants avec le code ci-dessous pour calculer une clé d’appareil pour les deux appareils :

* **contoso-toaster-007**
* **contoso-heatpump-088**

Remplacez la valeur de la variable **KEY** par la **clé primaire** que vous avez notée précédemment après la création de votre groupe d’inscription. La valeur de clé et la sortie affichées avec le code ci-dessous ne sont qu’un exemple.

#### <a name="powershell"></a>[PowerShell](#tab/powershell)

Si vous utilisez une station de travail Windows, utilisez PowerShell pour générer votre clé d’appareil dérivée, comme indiqué dans l’exemple suivant.

```powershell
$KEY='oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA=='

$REG_ID1='contoso-toaster-007'
$REG_ID2='contoso-heatpump-088'

$hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
$hmacsha256.key = [Convert]::FromBase64String($key)
$sig1 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID1))
$sig2 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID2))
$derivedkey1 = [Convert]::ToBase64String($sig1)
$derivedkey2 = [Convert]::ToBase64String($sig2)

echo "`n`n$REG_ID1 : $derivedkey1`n$REG_ID2 : $derivedkey2`n`n"
```

```powershell
contoso-toaster-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
contoso-heatpump-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

#### <a name="bash"></a>[Bash](#tab/bash)

Si vous utilisez une station de travail Linux, vous pouvez utiliser openssl pour générer vos clés d’appareil dérivées, comme indiqué dans l’exemple Bash suivant.

```bash
KEY=oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA==

REG_ID1=contoso-toaster-007
REG_ID2=contoso-heatpump-088

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
devkey1=$(echo -n $REG_ID1 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)
devkey2=$(echo -n $REG_ID2 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)

echo -e $"\n\n$REG_ID1 : $devkey1\n$REG_ID2 : $devkey2\n\n"
```

```bash
contoso-toaster-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
contoso-heatpump-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

---


## <a name="prepare-an-azure-iot-c-sdk-development-environment"></a>Préparer un environnement de développement pour le SDK Azure IoT pour C

Ces appareils demanderont un provisionnement à l’aide d’un exemple de code de provisionnement inclus dans le [SDK Azure IoT pour C](https://github.com/Azure/azure-iot-sdk-c).

Dans cette section, vous préparez l’environnement de développement utilisé pour générer le [SDK Azure IoT pour C](https://github.com/Azure/azure-iot-sdk-c). Le SDK inclut l’exemple de code pour l’appareil simulé. Cet appareil simulé tente le provisionnement durant la séquence de démarrage de l’appareil.

Cette section s’applique à une station de travail Windows. Pour obtenir un exemple sur Linux, consultez la section sur la configuration des machines virtuelles dans [Guide pratique du provisionnement pour la multilocation](how-to-provision-multitenant.md).

1. Téléchargez le [système de génération CMake](https://cmake.org/download/).

    Il est important que les composants requis Visual Studio (Visual Studio et la charge de travail « Développement Desktop en C++ ») soient installés sur votre machine, **avant** de commencer l’installation de l’élément `CMake`. Une fois les composants requis en place et le téléchargement effectué, installez le système de génération de CMake.

2. Recherchez le nom de balise de la [version la plus récente](https://github.com/Azure/azure-iot-sdk-c/releases/latest) du kit de développement logiciel (SDK).

3. Ouvrez une invite de commandes ou l’interpréteur de commandes Git Bash. Exécutez les commandes suivantes pour cloner la dernière version du dépôt GitHub du [Azure IoT C SDK](https://github.com/Azure/azure-iot-sdk-c). Utilisez l’étiquette obtenue à l’étape précédente comme valeur pour le paramètre `-b` :

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Attendez-vous à ce que cette opération prenne plusieurs minutes.

4. Créez un sous-répertoire `cmake` dans le répertoire racine du référentiel Git et accédez à ce dossier. Exécutez les commandes suivantes à partir du répertoire `azure-iot-sdk-c` :

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Exécutez la commande suivante qui génère une version du SDK propre à votre plateforme cliente de développement. Une solution Visual Studio pour l’appareil simulé est générée dans le répertoire `cmake`. 

    ```cmd
    cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    ```

    Si `cmake` ne trouve pas votre compilateur C++, vous risquez de rencontrer des erreurs de build lors de l’exécution de la commande. Si cela se produit, essayez d’exécuter la commande dans [l’invite de commandes de Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs).

    Une fois la génération terminée, les dernières lignes de sortie doivent ressembler à la sortie suivante :

    ```cmd/sh
    $ cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

## <a name="simulate-the-devices"></a>Simuler les appareils

Dans cette section, vous allez mettre à jour un exemple d’approvisionnement nommé **prov\_dev\_client\_sample** provenant du SDK Azure IoT pour C que vous avez configuré précédemment.

Cet exemple de code simule une séquence de démarrage d’un appareil qui envoie la demande de provisionnement à votre instance du service Device Provisioning. La séquence de démarrage entraîne la reconnaissance de l’appareil Toaster et son assignation au hub IoT défini par la stratégie d’allocation personnalisée.

1. Dans le portail Azure, sélectionnez l’onglet **Vue d’ensemble** de votre service Device Provisioning et notez les valeurs de **_Étendue de l’ID_**.

    ![Extraction des informations de point de terminaison du service Device Provisioning à partir du panneau du Portail](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. Dans Visual Studio, ouvrez le fichier solution **azure_iot_sdks.sln** généré précédemment en exécutant CMake. Le fichier solution doit être à l’emplacement suivant :

    ```
    azure-iot-sdk-c\cmake\azure_iot_sdks.sln
    ```

3. Dans la fenêtre *Explorateur de solutions* de Visual Studio, accédez au dossier **Provision\_Samples**. Développez l’exemple de projet nommé **prov\_dev\_client\_sample**. Développez **Fichiers sources**, puis ouvrez **prov\_dev\_client\_sample.c**.

4. Recherchez la constante `id_scope` et remplacez la valeur par la valeur **Étendue de l’ID** que vous avez copiée précédemment. 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

5. Recherchez la définition de la fonction `main()` dans le même fichier. Vérifiez que la variable `hsm_type` a la valeur `SECURE_DEVICE_TYPE_SYMMETRIC_KEY`, comme indiqué ci-dessous :

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

6. Dans la fonction `main()`, recherchez l’appel à `Prov_Device_Register_Device()`. Juste avant cet appel, ajoutez les lignes de code suivantes qui utilisent [`Prov_Device_Set_Provisioning_Payload()`](/azure/iot-hub/iot-c-sdk-ref/prov-device-client-h/prov-device-set-provisioning-payload) pour passer une charge utile JSON personnalisée pendant le provisionnement. Cela permet de fournir plus d’informations à vos fonctions d’allocation personnalisées. Cela peut également être utilisé pour passer le type d’appareil au lieu d’examiner l’ID d’inscription.

    ```c
    // An example custom payload
    const char* custom_json_payload = "{\"MyDeviceFirmwareVersion\":\"12.0.2.5\",\"MyDeviceProvisioningVersion\":\"1.0.0.0\"}";

    prov_device_result = Prov_Device_Set_Provisioning_Payload(prov_device_handle, custom_json_payload);
    if (prov_device_result != PROV_DEVICE_RESULT_OK)
    {
        (void)printf("\r\nFailure setting provisioning payload: %s\r\n", MU_ENUM_TO_STRING(PROV_DEVICE_RESULT, prov_device_result));
    }
    ```

7. Cliquez avec le bouton droit sur le projet **prov\_dev\_client\_sample** et sélectionnez **Définir comme projet de démarrage**.

### <a name="simulate-the-contoso-toaster-device"></a>Simuler l’appareil Toaster de Contoso

1. Pour simuler le toaster, recherchez l’appel à `prov_dev_set_symmetric_key_info()` dans **prov\_dev\_client\_sample.c**, qui est placé en commentaire.

    ```c
    // Set the symmetric key if using they auth type
    //prov_dev_set_symmetric_key_info("<symm_registration_id>", "<symmetric_Key>");
    ```

    Supprimez les marques de commentaire pour l’appel de fonction et remplacez les valeurs des espaces réservés (y compris les crochets) par l’ID d’inscription du Toaster et la clé d’appareil dérivée que vous avez générée précédemment. La valeur **JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=** est donnée seulement à titre d’exemple.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-toaster-007", "JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=");
    ```

    Enregistrez le fichier.

2. Dans le menu Visual Studio, sélectionnez **Déboguer** > **Exécuter sans débogage** pour exécuter la solution. Dans l’invite pour régénérer le projet, sélectionnez **Oui** pour régénérer le projet avant de l’exécuter.

    Le texte suivant est un exemple de journalisation de la sortie générée par le code de la fonction d’allocation personnalisée en cours d’exécution pour l’appareil Toaster. Notez qu’un hub est correctement sélectionné pour un appareil Toaster. Notez également le membre `payload` qui contient le contenu JSON personnalisé que vous avez ajouté au code. Il peut s’utiliser pour votre code dans `deviceRuntimeContext`.

    Cette journalisation est disponible en cliquant sur **Journaux** sous le code de fonction dans le portail :

    ```cmd
    2020-09-23T11:44:37.505 [Information] Executing 'Functions.HttpTrigger1' (Reason='This function was programmatically called via the host APIs.', Id=4596d45e-086f-4e86-929b-4a02814eee40)
    2020-09-23T11:44:41.380 [Information] C# HTTP trigger function processed a request.
    2020-09-23T11:44:41.381 [Information] Request.Body:...
    2020-09-23T11:44:41.381 [Information] {"enrollmentGroup":{"enrollmentGroupId":"contoso-custom-allocated-devices","attestation":{"type":"symmetricKey"},"capabilities":{"iotEdge":false},"etag":"\"e8002126-0000-0100-0000-5f6b2a570000\"","provisioningStatus":"enabled","reprovisionPolicy":{"updateHubAssignment":true,"migrateDeviceData":true},"createdDateTimeUtc":"2020-09-23T10:58:31.62286Z","lastUpdatedDateTimeUtc":"2020-09-23T10:58:31.62286Z","allocationPolicy":"custom","iotHubs":["contoso-toasters-hub-1098.azure-devices.net"],"customAllocationDefinition":{"webhookUrl":"https://contoso-function-app.azurewebsites.net/api/HttpTrigger1?****","apiVersion":"2019-03-31"}},"deviceRuntimeContext":{"registrationId":"contoso-toaster-007","symmetricKey":{},"payload":{"MyDeviceFirmwareVersion":"12.0.2.5","MyDeviceProvisioningVersion":"1.0.0.0"}},"linkedHubs":["contoso-toasters-hub-1098.azure-devices.net"]}
    2020-09-23T11:44:41.687 [Information] linkedHub : contoso-toasters-hub-1098.azure-devices.net
    2020-09-23T11:44:41.688 [Information] Selected hub : contoso-toasters-hub-1098.azure-devices.net
    2020-09-23T11:44:41.688 [Information] Response
    2020-09-23T11:44:41.688 [Information] {"iotHubHostName":"contoso-toasters-hub-1098.azure-devices.net"}
    2020-09-23T11:44:41.689 [Information] Executed 'Functions.HttpTrigger1' (Succeeded, Id=4596d45e-086f-4e86-929b-4a02814eee40, Duration=4347ms)    
    ```

    L’exemple de sortie d’appareil suivant illustre le démarrage réussi de l’appareil Toaster simulé et sa connexion à l’instance du service de provisionnement à assigner au hub IoT Toasters à l’aide de la stratégie d’allocation personnalisée :

    ```cmd
    Provisioning API Version: 1.3.6

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-toasters-hub-1098.azure-devices.net, deviceId: contoso-toaster-007

    Press enter key to exit:
    ```

### <a name="simulate-the-contoso-heat-pump-device"></a>Simuler l’appareil Heat pump de Contoso

1. Pour simuler l’appareil pompe à chaleur, mettez à jour l’appel à `prov_dev_set_symmetric_key_info()` dans **prov\_dev\_client\_sample.c** à nouveau avec l’ID d’inscription et la clé d’appareil dérivée de la pompe à chaleur générée précédemment. La valeur **6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=** est donnée seulement à titre d’exemple.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("contoso-heatpump-088", "6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=");
    ```

    Enregistrez le fichier.

2. Dans le menu Visual Studio, sélectionnez **Déboguer** > **Exécuter sans débogage** pour exécuter la solution. Dans l’invite pour régénérer le projet, sélectionnez **Oui** pour régénérer le projet avant de l’exécuter.

    Le texte suivant est un exemple de journalisation de la sortie générée par le code de la fonction d’allocation personnalisée en cours d’exécution pour l’appareil Heat Pump. La stratégie d’allocation personnalisée rejette cette inscription avec une erreur HTTP 400 Bad Request. Notez le membre `payload` qui contient le contenu JSON personnalisé que vous avez ajouté au code. Il peut s’utiliser pour votre code dans `deviceRuntimeContext`.

    Cette journalisation est disponible en cliquant sur **Journaux** sous le code de fonction dans le portail :

    ```cmd
    2020-09-23T11:50:23.652 [Information] Executing 'Functions.HttpTrigger1' (Reason='This function was programmatically called via the host APIs.', Id=2fa77f10-42f8-43fe-88d9-a8c01d4d3f68)
    2020-09-23T11:50:23.653 [Information] C# HTTP trigger function processed a request.
    2020-09-23T11:50:23.654 [Information] Request.Body:...
    2020-09-23T11:50:23.654 [Information] {"enrollmentGroup":{"enrollmentGroupId":"contoso-custom-allocated-devices","attestation":{"type":"symmetricKey"},"capabilities":{"iotEdge":false},"etag":"\"e8002126-0000-0100-0000-5f6b2a570000\"","provisioningStatus":"enabled","reprovisionPolicy":{"updateHubAssignment":true,"migrateDeviceData":true},"createdDateTimeUtc":"2020-09-23T10:58:31.62286Z","lastUpdatedDateTimeUtc":"2020-09-23T10:58:31.62286Z","allocationPolicy":"custom","iotHubs":["contoso-toasters-hub-1098.azure-devices.net"],"customAllocationDefinition":{"webhookUrl":"https://contoso-function-app.azurewebsites.net/api/HttpTrigger1?****","apiVersion":"2019-03-31"}},"deviceRuntimeContext":{"registrationId":"contoso-heatpump-088","symmetricKey":{},"payload":{"MyDeviceFirmwareVersion":"12.0.2.5","MyDeviceProvisioningVersion":"1.0.0.0"}},"linkedHubs":["contoso-toasters-hub-1098.azure-devices.net"]}
    2020-09-23T11:50:23.654 [Information] Unknown device registration
    2020-09-23T11:50:23.654 [Information] Response
    2020-09-23T11:50:23.654 [Information] Unrecognized device registration.
    2020-09-23T11:50:23.655 [Information] Executed 'Functions.HttpTrigger1' (Succeeded, Id=2fa77f10-42f8-43fe-88d9-a8c01d4d3f68, Duration=11ms)
    ```

    L’exemple de sortie d’appareil suivant illustre le démarrage de l’appareil Heat Pump simulé et sa connexion à l’instance du service de provisionnement pour tenter une inscription à un hub IoT à l’aide de la stratégie d’allocation personnalisée. Cette opération échoue avec l’erreur (`Custom allocation failed with status code: 400`), car la stratégie d’allocation personnalisée a été conçue pour autoriser uniquement les appareils Toaster :


    ```cmd
    Provisioning API Version: 1.3.7
    
    Registering Device
    
    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_device_ll_client.c Func:_prov_transport_process_json_reply Line:658 Provisioning Failure: OperationId: 4.eb89f3e8407a3711.2525bd34-02e9-4e91-a9c0-4dbc4ad5de66 - Date: 2020-09-23T17:05:58.2363145Z - Msg: Custom allocation failed with status code: 400
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_transport_mqtt_common.c Func:_prov_transport_common_mqtt_dowork Line:1014 Unable to process registration reply.
    Error: Time:Wed Sep 23 13:06:01 2020 File:d:\testing\azure-iot-sdk-c\provisioning_client\src\prov_device_ll_client.c Func:_on_transport_registration_data Line:770 Failure retrieving data from the provisioning service
    
    Failure registering device: PROV_DEVICE_RESULT_DEV_AUTH_ERROR
    Press enter key to exit:    
    ```
    
## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous envisagez de continuer à utiliser les ressources créées dans cet article, vous pouvez les conserver. Sinon, effectuez les étapes suivantes pour supprimer toutes les ressources qui ont été créées dans le cadre de cet article, et ainsi éviter des frais inutiles.

Les étapes décrites supposent que vous avez créé toutes les ressources utilisées dans cet article dans le même groupe de ressources nommé **contoso-us-resource-group**, comme nous vous l’avions demandé.

> [!IMPORTANT]
> La suppression d’un groupe de ressources est irréversible. Le groupe de ressources et toutes les ressources qu’il contient sont supprimés définitivement. Veillez à ne pas supprimer accidentellement les mauvaises ressources ou le mauvais groupe de ressources. Si vous avez créé l’IoT Hub à l’intérieur d’un groupe de ressources existant qui concerne des ressources que vous souhaitez conserver, supprimer uniquement la ressource de l’ IoT Hub au lieu de supprimer le groupe de ressources.
>

Pour supprimer le groupe de ressources par nom :

1. Connectez-vous au [portail Azure](https://portal.azure.com), puis sélectionnez **Groupes de ressources**.

2. Dans la zone de texte **Filtrer par nom...**, tapez le nom du groupe de ressources contenant vos ressources (ici, **contoso-us-resource-group**). 

3. À droite de votre groupe de ressources dans la liste des résultats, sélectionnez **...**, puis **Supprimer le groupe de ressources**.

4. Il vous sera demandé de confirmer la suppression du groupe de ressources. Saisissez à nouveau le nom de votre groupe de ressources pour confirmer, puis sélectionnez **Supprimer**. Après quelques instants, le groupe de ressources et toutes les ressources qu’il contient sont supprimés.

## <a name="next-steps"></a>Étapes suivantes

* Pour obtenir un exemple plus détaillé de stratégie d’allocation personnalisée, consultez [Comment utiliser des stratégies d’allocation personnalisées](how-to-use-custom-allocation-policies.md).
* Pour en savoir plus sur le reprovisionnement, consultez [Concepts du reprovisionnement d’appareils IoT Hub](concepts-device-reprovision.md).
* Pour en savoir plus sur le déprovisionnement, consultez [Guide pratique pour déprovisionner des appareils auparavant provisionnés automatiquement](how-to-unprovision-devices.md).