---
title: Déployer une machine virtuelle à l’aide de C# et d’un modèle Resource Manager
description: Apprenez à utiliser C# et un modèle Resource Manager pour déployer une machine virtuelle Azure.
author: cynthn
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: how-to
ms.date: 07/14/2017
ms.author: cynthn
ms.custom: devx-track-csharp
ms.openlocfilehash: 779a09532790ea272d8c95ac28f8c152216efc5a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008648"
---
# <a name="deploy-an-azure-virtual-machine-using-c-and-a-resource-manager-template"></a>Déployer une machine virtuelle Azure à l’aide de C# et d’un modèle Resource Manager

Cet article explique la procédure de déploiement d’un modèle Azure Resource Manager à l’aide de C#. Le modèle que vous créez déploie une machine virtuelle unique exécutant Windows Server dans un nouveau réseau virtuel avec un seul sous-réseau.

Pour obtenir une description détaillée de la ressource de machine virtuelle, consultez [Virtual machines in an Azure Resource Manager template (Machines virtuelles dans un modèle Azure Resource Manager)](template-description.md). Pour plus d’informations sur toutes les ressources d’un modèle, consultez [Guide de création d’un modèle Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

Ces étapes prennent environ 10 minutes.

## <a name="create-a-visual-studio-project"></a>Créer un projet Visual Studio

Lors de cette étape, assurez-vous que Visual Studio est installé et que vous créez une application console utilisée pour déployer le modèle.

1. Si vous ne l’avez pas déjà fait, installez [Visual Studio](/visualstudio/install/install-visual-studio). Sélectionnez **Développement .NET Desktop** dans la page Charges de travail, puis cliquez sur **Installer**. Dans le résumé, vous pouvez voir que les **Outils de développement .NET Framework 4 - 4.6** sont automatiquement sélectionnés. Si vous avez déjà installé Visual Studio, vous pouvez ajouter la charge de travail .NET en utilisant le lanceur Visual Studio.
2. Dans Visual Studio, cliquez sur **Fichier** > **Nouveau** > **Projet**.
3. Dans **Modèles** > **Visual C#** , sélectionnez **Application console (.NET Framework)** , entrez le nom de projet *myDotnetProject*, sélectionnez l’emplacement du projet, puis cliquez sur **OK**.

## <a name="install-the-packages"></a>Installer les packages

Les packages NuGet sont le moyen le plus simple pour installer les bibliothèques dont vous avez besoin pour terminer ces étapes. Pour obtenir les bibliothèques dont vous avez besoin dans Visual Studio, suivez ces étapes :

1. Cliquez sur **Outils** > **Gestionnaire de package NuGet**, puis sur **Console du gestionnaire de package**.
2. Dans la console, tapez ces commandes :

    ```powershell
    Install-Package Microsoft.Azure.Management.Fluent
    Install-Package WindowsAzure.Storage
    ```

## <a name="create-the-files"></a>Créer les fichiers

Lors de cette étape, vous créez un fichier de modèle qui déploie les ressources et un fichier de paramètres qui fournit les valeurs des paramètres au modèle. Vous créez également un fichier d’autorisation qui est utilisé pour effectuer des opérations Azure Resource Manager.

### <a name="create-the-template-file"></a>Créer le fichier de modèle

1. Dans l’Explorateur de solutions, cliquez sur *myDotnetProject* > **Ajouter** > **Nouvel élément**, puis sélectionnez **Fichier texte** dans *Éléments Visual C#* . Nommez le fichier *CreateVMTemplate.json*, puis cliquez sur **Ajouter**.
2. Ajoutez ce code JSON au fichier que vous avez créé :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "adminUsername": { "type": "string" },
        "adminPassword": { "type": "securestring" }
      },
      "variables": {
        "vnetID": "[resourceId('Microsoft.Network/virtualNetworks','myVNet')]", 
        "subnetRef": "[concat(variables('vnetID'),'/subnets/mySubnet')]", 
      },
      "resources": [
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/publicIPAddresses",
          "name": "myPublicIPAddress",
          "location": "[resourceGroup().location]",
          "properties": {
            "publicIPAllocationMethod": "Dynamic",
            "dnsSettings": {
              "domainNameLabel": "myresourcegroupdns1"
            }
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/virtualNetworks",
          "name": "myVNet",
          "location": "[resourceGroup().location]",
          "properties": {
            "addressSpace": { "addressPrefixes": [ "10.0.0.0/16" ] },
            "subnets": [
              {
                "name": "mySubnet",
                "properties": { "addressPrefix": "10.0.0.0/24" }
              }
            ]
          }
        },
        {
          "apiVersion": "2016-03-30",
          "type": "Microsoft.Network/networkInterfaces",
          "name": "myNic",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Network/publicIPAddresses/', 'myPublicIPAddress')]",
            "[resourceId('Microsoft.Network/virtualNetworks/', 'myVNet')]"
          ],
          "properties": {
            "ipConfigurations": [
              {
                "name": "ipconfig1",
                "properties": {
                  "privateIPAllocationMethod": "Dynamic",
                  "publicIPAddress": { "id": "[resourceId('Microsoft.Network/publicIPAddresses','myPublicIPAddress')]" },
                  "subnet": { "id": "[variables('subnetRef')]" }
                }
              }
            ]
          }
        },
        {
          "apiVersion": "2016-04-30-preview",
          "type": "Microsoft.Compute/virtualMachines",
          "name": "myVM",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Network/networkInterfaces/', 'myNic')]"
          ],
          "properties": {
            "hardwareProfile": { "vmSize": "Standard_DS1" },
            "osProfile": {
              "computerName": "myVM",
              "adminUsername": "[parameters('adminUsername')]",
              "adminPassword": "[parameters('adminPassword')]"
            },
            "storageProfile": {
              "imageReference": {
                "publisher": "MicrosoftWindowsServer",
                "offer": "WindowsServer",
                "sku": "2012-R2-Datacenter",
                "version": "latest"
              },
              "osDisk": {
                "name": "myManagedOSDisk",
                "caching": "ReadWrite",
                "createOption": "FromImage"
              }
            },
            "networkProfile": {
              "networkInterfaces": [
                {
                  "id": "[resourceId('Microsoft.Network/networkInterfaces','myNic')]"
                }
              ]
            }
          }
        }
      ]
    }
    ```

3. Enregistrez le fichier CreateVMTemplate.json.

### <a name="create-the-parameters-file"></a>Créer le fichier de paramètres

Pour spécifier des valeurs pour les paramètres de ressource dans le modèle, vous créez un fichier de paramètres qui contient les valeurs.

1. Dans l’Explorateur de solutions, cliquez sur *myDotnetProject* > **Ajouter** > **Nouvel élément**, puis sélectionnez **Fichier texte** dans *Éléments Visual C#* . Nommez le fichier *Parameters.json*, puis cliquez sur **Ajouter**.
2. Ajoutez ce code JSON au fichier que vous avez créé :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "adminUserName": { "value": "azureuser" },
        "adminPassword": { "value": "Azure12345678" }
      }
    }
    ```

4. Enregistrez le fichier Parameters.json.

### <a name="create-the-authorization-file"></a>Créer le fichier d’autorisation

Avant de commencer cette étape, vérifiez que vous avez accès à un [principal de service Active Directory](../../active-directory/develop/howto-authenticate-service-principal-powershell.md). Dans le principal de service, vous obtenez le jeton d'authentification des demandes pour Azure Resource Manager. Vous devez également enregistrer l’ID d’application, la clé d’authentification et l’ID de locataire dont vous avez besoin dans le fichier d’autorisation.

1. Dans l’Explorateur de solutions, cliquez sur *myDotnetProject* > **Ajouter** > **Nouvel élément**, puis sélectionnez **Fichier texte** dans *Éléments Visual C#* . Nommez le fichier *azureauth.properties*, puis cliquez sur **Ajouter**.
2. Ajoutez ces propriétés d’autorisation :

    ```
    subscription=<subscription-id>
    client=<application-id>
    key=<authentication-key>
    tenant=<tenant-id>
    managementURI=https://management.core.windows.net/
    baseURL=https://management.azure.com/
    authURL=https://login.windows.net/
    graphURL=https://graph.microsoft.com/
    ```

    Remplacez **&lt;subscription-id&gt;** par votre identificateur d’abonnement, **&lt;application-id&gt;** par l’identificateur d’application Active Directory, **&lt;authentication-key&gt;** par la clé d’application et **&lt;tenant-id&gt;** par l’identificateur du locataire.

3. Enregistrez le fichier azureauth.properties.
4. Définissez une variable d’environnement dans Windows nommée AZURE_AUTH_LOCATION avec le chemin complet au fichier d’autorisation que vous avez créé. Vous pouvez par exemple utiliser la commande PowerShell suivante :

    ```powershell
    [Environment]::SetEnvironmentVariable("AZURE_AUTH_LOCATION", "C:\Visual Studio 2019\Projects\myDotnetProject\myDotnetProject\azureauth.properties", "User")
    ```

    

## <a name="create-the-management-client"></a>Créer le client de gestion

1. Ouvrez le fichier Program.cs du projet que vous avez créé. Ensuite, ajoutez ces instructions using aux instructions existantes en haut du fichier :

    ```csharp
    using Microsoft.Azure.Management.Compute.Fluent;
    using Microsoft.Azure.Management.Compute.Fluent.Models;
    using Microsoft.Azure.Management.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent;
    using Microsoft.Azure.Management.ResourceManager.Fluent.Core;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```

2. Pour créer le client de gestion, ajoutez ce code à la méthode Main :

    ```csharp
    var credentials = SdkContext.AzureCredentialsFactory
        .FromFile(Environment.GetEnvironmentVariable("AZURE_AUTH_LOCATION"));

    var azure = Azure
        .Configure()
        .WithLogLevel(HttpLoggingDelegatingHandler.Level.Basic)
        .Authenticate(credentials)
        .WithDefaultSubscription();
    ```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Pour spécifier des valeurs pour l’application, ajoutez du code à la méthode Main :

```csharp
var groupName = "myResourceGroup";
var location = Region.USWest;

var resourceGroup = azure.ResourceGroups.Define(groupName)
    .WithRegion(location)
    .Create();
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

Le modèle et les paramètres sont déployés à partir d’un compte de stockage dans Azure. Lors de cette étape, vous créez le compte et chargez les fichiers. 

Pour créer le compte, ajoutez ce code à la méthode Main :

```csharp
string storageAccountName = SdkContext.RandomResourceName("st", 10);

Console.WriteLine("Creating storage account...");
var storage = azure.StorageAccounts.Define(storageAccountName)
    .WithRegion(Region.USWest)
    .WithExistingResourceGroup(resourceGroup)
    .Create();

var storageKeys = storage.GetKeys();
string storageConnectionString = "DefaultEndpointsProtocol=https;"
    + "AccountName=" + storage.Name
    + ";AccountKey=" + storageKeys[0].Value
    + ";EndpointSuffix=core.windows.net";

var account = CloudStorageAccount.Parse(storageConnectionString);
var serviceClient = account.CreateCloudBlobClient();

Console.WriteLine("Creating container...");
var container = serviceClient.GetContainerReference("templates");
container.CreateIfNotExistsAsync().Wait();
var containerPermissions = new BlobContainerPermissions()
    { PublicAccess = BlobContainerPublicAccessType.Container };
container.SetPermissionsAsync(containerPermissions).Wait();

Console.WriteLine("Uploading template file...");
var templateblob = container.GetBlockBlobReference("CreateVMTemplate.json");
templateblob.UploadFromFileAsync("..\\..\\CreateVMTemplate.json").Result();

Console.WriteLine("Uploading parameters file...");
var paramblob = container.GetBlockBlobReference("Parameters.json");
paramblob.UploadFromFileAsync("..\\..\\Parameters.json").Result();
```

## <a name="deploy-the-template"></a>Déployer le modèle

Déployez le modèle et les paramètres du compte de stockage que vous avez créé. 

Pour déployer le modèle, ajoutez ce code à la méthode Main :

```csharp
var templatePath = "https://" + storageAccountName + ".blob.core.windows.net/templates/CreateVMTemplate.json";
var paramPath = "https://" + storageAccountName + ".blob.core.windows.net/templates/Parameters.json";
var deployment = azure.Deployments.Define("myDeployment")
    .WithExistingResourceGroup(groupName)
    .WithTemplateLink(templatePath, "1.0.0.0")
    .WithParametersLink(paramPath, "1.0.0.0")
    .WithMode(Microsoft.Azure.Management.ResourceManager.Fluent.Models.DeploymentMode.Incremental)
    .Create();
Console.WriteLine("Press enter to delete the resource group...");
Console.ReadLine();
```

## <a name="delete-the-resources"></a>Supprimer les ressources

Étant donné que les ressources utilisées dans Microsoft Azure vous sont facturées, il est toujours conseillé de supprimer les ressources qui ne sont plus nécessaires. Vous n’avez pas besoin de supprimer séparément les ressources d’un groupe de ressources. Supprimez le groupe de ressources pour supprimer automatiquement toutes ses ressources. 

Pour supprimer le groupe de ressources, ajoutez ce code à la méthode Main :

```csharp
azure.ResourceGroups.DeleteByName(groupName);
```

## <a name="run-the-application"></a>Exécution de l'application

L’exécution complète de cette application console devrait durer cinq minutes environ. 

1. Pour exécuter l’application console, cliquez sur **Démarrer**.

2. Avant d’appuyer sur **Entrée** pour démarrer la suppression des ressources, prenez quelques minutes pour vérifier que les ressources dans le portail Azure ont bien été créées. Cliquez sur l’état du déploiement pour afficher des informations sur le déploiement.

## <a name="next-steps"></a>Étapes suivantes

* Si vous rencontrez des problèmes lors du déploiement, nous vous conseillons de consulter la section [Résolution des erreurs courantes dans un déploiement Azure avec Azure Resource Manager](../../azure-resource-manager/templates/common-deployment-errors.md).
* Découvrez comment déployer une machine virtuelle et ses ressources de soutien en consultant [Déployer une machine virtuelle Azure à l’aide de C#](../../azure-resource-manager/templates/deploy-rest.md).