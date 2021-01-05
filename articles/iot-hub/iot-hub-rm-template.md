---
title: Créer un IoT Hub Azure à l’aide d’un modèle (.NET) | Microsoft Docs
description: Comment utiliser un modèle Azure Resource Manager pour créer un IoT Hub avec un programme C#.
author: robinsh
manager: philmea
ms.author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: csharp
ms.topic: conceptual
ms.date: 08/08/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: db4b676e65d36a9476fd72b66cc8ccfa38af4d85
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92144497"
---
# <a name="create-an-iot-hub-using-azure-resource-manager-template-net"></a>Créer un IoT Hub avec un modèle Azure Resource Manager (.NET)

[!INCLUDE [iot-hub-resource-manager-selector](../../includes/iot-hub-resource-manager-selector.md)]

Vous pouvez utiliser Azure Resource Manager pour créer et gérer des hubs Azure IoT de façon programmée. Ce didacticiel vous montre comment utiliser un modèle Azure Resource Manager pour créer un IoT Hub à partir d’un programme C#.

> [!NOTE]
> Azure a deux modèles de déploiement différents pour créer et utiliser des ressources :  [Azure Resource Manager et classique](../azure-resource-manager/management/deployment-models.md).  Cet article traite de l’utilisation du modèle de déploiement Azure Resource Manager.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Visual Studio.
* Un compte Azure actif. <br/>Si vous ne possédez pas de compte, vous pouvez créer un [compte gratuit][lnk-free-trial] en quelques minutes.
* Un [compte Stockage Azure][lnk-storage-account] dans lequel vous pouvez stocker vos fichiers de modèles Azure Resource Manager.
* [Azure PowerShell 1.0][lnk-powershell-install] ou version ultérieure.

[!INCLUDE [iot-hub-prepare-resource-manager](../../includes/iot-hub-prepare-resource-manager.md)]

## <a name="prepare-your-visual-studio-project"></a>Préparer votre projet Visual Studio

1. Dans Visual Studio, créez un projet Visual C# Bureau classique Windows en utilisant le modèle de projet **Application console (.NET Framework)** . Nommez ce projet **CreateIoTHub**.

2. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre projet, puis cliquez sur **GÉRER LES PACKAGES NUGET**.

3. Dans le gestionnaire de packages NuGet, activez **Inclure la version préliminaire** et sur la page **Parcourir**, recherchez **Microsoft.Azure.Management.ResourceManager**. Sélectionnez le package et cliquez sur **Installer**. Sous **Réviser les modifications**, cliquez sur **OK**, puis sur **J’accepte** pour accepter les licences.

4. Dans le gestionnaire de packages NuGet, recherchez **Microsoft.IdentityModel.Clients.ActiveDirectory**.  Cliquez sur **Installer**, dans **Réviser les modifications**, cliquez sur **OK**, puis cliquez sur **J’accepte** pour accepter la licence.

5. Dans Program.cs, remplacez les instructions **using** existantes par le code suivant :

    ```csharp
    using System;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.ResourceManager.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Microsoft.Rest;
    ```

6. Dans Program.cs, ajoutez les variables statiques suivantes en remplaçant les valeurs des espaces réservés. Vous avez noté les éléments **ApplicationId**, **SubscriptionId**, **TenantId** et **Password** précédemment dans ce didacticiel. **Nom de votre compte de stockage Azure** est le nom du compte de stockage Azure où vous stockez vos fichiers de modèle Azure Resource Manager. **Nom du groupe de ressources** est le nom du groupe de ressources que vous utiliserez pour créer le hub IoT. Il peut s’agir d’un groupe de ressources existant ou nouveau. **Nom du déploiement** est le nom du déploiement, par exemple **Déploiement_01**.

    ```csharp
    static string applicationId = "{Your ApplicationId}";
    static string subscriptionId = "{Your SubscriptionId}";
    static string tenantId = "{Your TenantId}";
    static string password = "{Your application Password}";
    static string storageAddress = "https://{Your storage account name}.blob.core.windows.net";
    static string rgName = "{Resource group name}";
    static string deploymentName = "{Deployment name}";
    ```

[!INCLUDE [iot-hub-get-access-token](../../includes/iot-hub-get-access-token.md)]

## <a name="submit-a-template-to-create-an-iot-hub"></a>Envoyer un modèle pour créer un hub IoT

Utilisez un fichier de paramètres et un modèle JSON pour créer un IoT Hub dans votre groupe de ressources. Vous pouvez également utiliser un modèle Azure Resource Manager pour apporter des modifications à un IoT Hub existant.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre projet, cliquez sur **Ajouter**, puis sur **Nouvel élément**. Ajoutez un fichier JSON appelé **template.json** à votre projet.

2. Pour ajouter un IoT Hub standard à la région **USA Est**, remplacez le contenu de **template.json** par la définition de ressource suivante. Pour obtenir la liste des régions qui prennent en charge IoT Hub, voir [Statut Azure][lnk-status] :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "hubName": {
          "type": "string"
        }
      },
      "resources": [
      {
        "apiVersion": "2016-02-03",
        "type": "Microsoft.Devices/IotHubs",
        "name": "[parameters('hubName')]",
        "location": "East US",
        "sku": {
          "name": "S1",
          "tier": "Standard",
          "capacity": 1
        },
        "properties": {
          "location": "East US"
        }
      }
      ],
      "outputs": {
        "hubKeys": {
          "value": "[listKeys(resourceId('Microsoft.Devices/IotHubs', parameters('hubName')), '2016-02-03')]",
          "type": "object"
        }
      }
    }
    ```

3. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur votre projet, cliquez sur **Ajouter**, puis sur **Nouvel élément**. Ajoutez un fichier JSON appelé **parameters.json** à votre projet.

4. Remplacez le contenu de **parameters.json** avec les informations des paramètres ci-dessous qui définissent le nom du nouvel IoT Hub sur **{vos initiales}mynewiothub**. Le nom du IoT Hub doit être globalement unique et inclure votre nom ou vos initiales :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "hubName": { "value": "mynewiothub" }
      }
    }
    ```
   [!INCLUDE [iot-hub-pii-note-naming-hub](../../includes/iot-hub-pii-note-naming-hub.md)]

5. Dans l’**Explorateur de serveurs**, connectez-vous à votre abonnement Azure et, dans votre compte de stockage Azure, créez un conteneur appelé **modèles**. Dans le panneau **Propriétés**, définissez les autorisations **Accès public en lecture** pour le conteneur **modèles** sur **Blob**.

6. Dans l’**Explorateur de serveurs**, cliquez avec le bouton droit sur le conteneur **modèles**, puis cliquez sur **Afficher le conteneur d’objets blob**. Cliquez sur le bouton **Télécharger l’objet blob**, sélectionnez les fichiers **parameters.json** et **templates.json**, puis cliquez sur **Ouvrir** pour télécharger les fichiers JSON dans le conteneur **modèles**. Les URL des objets blob contenant les données JSON sont :

    ```csharp
    https://{Your storage account name}.blob.core.windows.net/templates/parameters.json
    https://{Your storage account name}.blob.core.windows.net/templates/template.json
    ```
7. Ajoutez la méthode suivante au fichier Program.cs :

    ```csharp
    static void CreateIoTHub(ResourceManagementClient client)
    {

    }
    ```

8. Ajoutez le code suivant à la méthode **CreateIoTHub** pour envoyer le fichier de modèle et le fichier de paramètres à Azure Resource Manager :

    ```csharp
    var createResponse = client.Deployments.CreateOrUpdate(
        rgName,
        deploymentName,
        new Deployment()
        {
          Properties = new DeploymentProperties
          {
            Mode = DeploymentMode.Incremental,
            TemplateLink = new TemplateLink
            {
              Uri = storageAddress + "/templates/template.json"
            },
            ParametersLink = new ParametersLink
            {
              Uri = storageAddress + "/templates/parameters.json"
            }
          }
        });
    ```

9. Ajoutez le code suivant à la méthode **CreateIoTHub** qui affiche l’état et les clés du nouvel IoT Hub :

    ```csharp
    string state = createResponse.Properties.ProvisioningState;
    Console.WriteLine("Deployment state: {0}", state);

    if (state != "Succeeded")
    {
      Console.WriteLine("Failed to create iothub");
    }
    Console.WriteLine(createResponse.Properties.Outputs);
    ```

## <a name="complete-and-run-the-application"></a>Terminer et exécuter l’application

Vous pouvez maintenant terminer l’application en appelant la méthode **CreateIoTHub** avant sa génération et son exécution.

1. Ajoutez le code suivant à la fin de la méthode **Main** :

    ```csharp
    CreateIoTHub(client);
    Console.ReadLine();
    ```

2. Cliquez sur **Build**, puis sur **Générer la solution**. Corrigez les éventuelles erreurs.

3. Cliquez sur **Déboguer**, puis **Démarrer le débogage** pour exécuter l’application. Le déploiement peut prendre plusieurs minutes.

4. Pour vérifier que votre application a bien ajouté le nouvel IoT Hub, accédez au [portail Azure][lnk-azure-portal] et affichez votre liste de ressources. Vous pouvez également utiliser la cmdlet PowerShell **Get-AzResource**.

> [!NOTE]
> Cet exemple d’application ajoute un IoT Hub S1 Standard qui vous est facturé. Lorsque vous avez terminé, vous pouvez supprimer le hub IoT via le [portail Azure][lnk-azure-portal] ou à l’aide de la cmdlet PowerShell **Remove-AzResource**.

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez déployé un IoT Hub à l’aide d’un modèle Azure Resource Manager avec un programme C#, vous pouvez aller encore plus loin :

* Découvrez les capacités de [l’API REST du fournisseur de ressources IoT Hub][lnk-rest-api].
* Pour plus d’informations sur les capacités d’Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager][lnk-azure-rm-overview].
* Pour connaître la syntaxe JSON et les propriétés à utiliser dans les modèles, consultez [Types de ressources Microsoft.Devices](/azure/templates/microsoft.devices/iothub-allversions).

Pour en savoir plus sur le développement pour IoT Hub, consultez les articles suivants :

* [Présentation du SDK C][lnk-c-sdk]
* [Kits de développement logiciel (SDK) Azure IoT][lnk-sdks]

Pour explorer davantage les capacités de IoT Hub, consultez :

* [Déploiement d’une IA sur des appareils de périmètre avec Azure IoT Edge][lnk-iotedge]

<!-- Links -->
[lnk-free-trial]: https://azure.microsoft.com/pricing/free-trial/
[lnk-azure-portal]: https://portal.azure.com/
[lnk-status]: https://azure.microsoft.com/status/
[lnk-powershell-install]: /powershell/azure/install-Az-ps
[lnk-rest-api]: /rest/api/iothub/iothubresource
[lnk-azure-rm-overview]: ../azure-resource-manager/management/overview.md
[lnk-storage-account]:../storage/common/storage-create-storage-account.md

[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-iotedge]: ../iot-edge/quickstart-linux.md