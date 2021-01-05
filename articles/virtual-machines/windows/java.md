---
title: Créer et gérer une machine virtuelle Azure en Java
description: Utilisez Java et Azure Resource Manager pour déployer une machine virtuelle et toutes ses ressources de prise en charge.
services: virtual-machines-windows
author: cynthn
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.topic: how-to
ms.date: 07/17/2017
ms.custom: devx-track-java
ms.author: cynthn
ms.openlocfilehash: f571dc5f4f3aee2405d4968b050bd2dd92a709db
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012746"
---
# <a name="create-and-manage-windows-vms-in-azure-using-java"></a>Créer et gérer des machines virtuelles Windows dans Azure à l’aide de Java

Une [machine virtuelle Azure](overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) a besoin de plusieurs ressources de prise en charge Azure. Cet article traite de la création, de la gestion et de la suppression de ressources de machines virtuelles avec Java. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Création d’un projet Maven
> * Ajout de dépendances
> * Créer des informations d’identification
> * Créer des ressources
> * Effectuer d’autres tâches de gestion
> * Supprimer des ressources
> * Exécution de l'application

Ces étapes prennent environ 20 minutes.

## <a name="create-a-maven-project"></a>Création d’un projet Maven

1. Si ce n’est pas déjà fait, installez [Java](/azure/developer/java/fundamentals/java-jdk-long-term-support).
2. Installez [Maven](https://maven.apache.org/download.cgi).
3. Créez un dossier et le projet :
    
    ```
    mkdir java-azure-test
    cd java-azure-test
    
    mvn archetype:generate -DgroupId=com.fabrikam -DartifactId=testAzureApp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
    ```

## <a name="add-dependencies"></a>Ajout de dépendances

1. Sous le dossier `testAzureApp`, ouvrez le fichier `pom.xml` et ajoutez la configuration de build au &lt;projet&gt; pour activer la génération de votre application :

    ```xml
    <build>
      <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>exec-maven-plugin</artifactId>
            <configuration>
                <mainClass>com.fabrikam.testAzureApp.App</mainClass>
            </configuration>
        </plugin>
      </plugins>
    </build>
    ```

2. Ajoutez les dépendances nécessaires pour accéder au kit Azure Java SDK.

    ```xml
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-compute</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-resources</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.microsoft.azure</groupId>
      <artifactId>azure-mgmt-network</artifactId>
      <version>1.1.0</version>
    </dependency>
    <dependency>
      <groupId>com.squareup.okio</groupId>
      <artifactId>okio</artifactId>
      <version>1.13.0</version>
    </dependency>
    <dependency>
      <groupId>com.nimbusds</groupId>
      <artifactId>nimbus-jose-jwt</artifactId>
      <version>3.6</version>
    </dependency>
    <dependency>
      <groupId>net.minidev</groupId>
      <artifactId>json-smart</artifactId>
      <version>1.0.6.3</version>
    </dependency>
    <dependency>
      <groupId>javax.mail</groupId>
      <artifactId>mail</artifactId>
      <version>1.4.5</version>
    </dependency>
    ```

3. Enregistrez le fichier .

## <a name="create-credentials"></a>Créer des informations d’identification

Avant de commencer cette étape, assurez-vous que vous avez accès à un [principal de service Active Directory](../../active-directory/develop/howto-create-service-principal-portal.md). Vous devez également enregistrer l’ID d’application, la clé d’authentification et l’ID de client dont vous aurez besoin dans une étape ultérieure.

### <a name="create-the-authorization-file"></a>Créer le fichier d’autorisation

1. Créez un fichier sous le nom `azureauth.properties` et ajoutez-y ces propriétés :

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

2. Enregistrez le fichier .
3. Définissez une variable d’environnement sous le nom AZURE_AUTH_LOCATION dans votre shell avec le chemin complet du fichier d’authentification.

### <a name="create-the-management-client"></a>Créer le client de gestion

1. Ouvrez le fichier `App.java` situé sous `src\main\java\com\fabrikam` et vérifiez que cette instruction package se trouve en haut :

    ```java
    package com.fabrikam.testAzureApp;
    ```

2. Sous l’instruction package, ajoutez ces instructions import :
   
    ```java
    import com.microsoft.azure.management.Azure;
    import com.microsoft.azure.management.compute.AvailabilitySet;
    import com.microsoft.azure.management.compute.AvailabilitySetSkuTypes;
    import com.microsoft.azure.management.compute.CachingTypes;
    import com.microsoft.azure.management.compute.InstanceViewStatus;
    import com.microsoft.azure.management.compute.DiskInstanceView;
    import com.microsoft.azure.management.compute.VirtualMachine;
    import com.microsoft.azure.management.compute.VirtualMachineSizeTypes;
    import com.microsoft.azure.management.network.PublicIPAddress;
    import com.microsoft.azure.management.network.Network;
    import com.microsoft.azure.management.network.NetworkInterface;
    import com.microsoft.azure.management.resources.ResourceGroup;
    import com.microsoft.azure.management.resources.fluentcore.arm.Region;
    import com.microsoft.azure.management.resources.fluentcore.model.Creatable;
    import com.microsoft.rest.LogLevel;
    import java.io.File;
    import java.util.Scanner;
    ```

2. Pour créer les informations d’identification Active Directory dont vous avez besoin pour effectuer des demandes, ajoutez ce code à la méthode main de la classe App :
   
    ```java
    try {
        final File credFile = new File(System.getenv("AZURE_AUTH_LOCATION"));
        Azure azure = Azure.configure()
            .withLogLevel(LogLevel.BASIC)
            .authenticate(credFile)
            .withDefaultSubscription();
    } catch (Exception e) {
        System.out.println(e.getMessage());
        e.printStackTrace();
    }

    ```

## <a name="create-resources"></a>Créer des ressources

### <a name="create-the-resource-group"></a>Créer le groupe de ressources

Toutes les ressources doivent être contenues dans un [groupe de ressources](../../azure-resource-manager/management/overview.md).

Pour spécifier les valeurs de l’application et créer le groupe de ressources, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Creating resource group...");
ResourceGroup resourceGroup = azure.resourceGroups()
    .define("myResourceGroup")
    .withRegion(Region.US_EAST)
    .create();
```

### <a name="create-the-availability-set"></a>Créer le groupe à haute disponibilité

Les [groupes à haute disponibilité](tutorial-availability-sets.md) facilitent la maintenance des machines virtuelles utilisées par votre application.

Pour créer le groupe à haute disponibilité, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Creating availability set...");
AvailabilitySet availabilitySet = azure.availabilitySets()
    .define("myAvailabilitySet")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withSku(AvailabilitySetSkuTypes.MANAGED)
    .create();
```
### <a name="create-the-public-ip-address"></a>Créer une adresse IP publique

Une [adresse IP publique](../../virtual-network/public-ip-addresses.md) est nécessaire pour communiquer avec la machine virtuelle.

Pour créer l’adresse IP publique de la machine virtuelle, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Creating public IP address...");
PublicIPAddress publicIPAddress = azure.publicIPAddresses()
    .define("myPublicIP")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withDynamicIP()
    .create();
```

### <a name="create-the-virtual-network"></a>Créer un réseau virtuel

Un machine virtuelle doit figurer dans un sous-réseau d’un [réseau virtuel](../../virtual-network/virtual-networks-overview.md).

Pour créer un sous-réseau et un réseau virtuel, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Creating virtual network...");
Network network = azure.networks()
    .define("myVN")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withAddressSpace("10.0.0.0/16")
    .withSubnet("mySubnet","10.0.0.0/24")
    .create();
```

### <a name="create-the-network-interface"></a>Créer l’interface réseau

Une machine virtuelle a besoin d’une interface réseau pour communiquer sur le réseau virtuel.

Pour créer une interface réseau, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Creating network interface...");
NetworkInterface networkInterface = azure.networkInterfaces()
    .define("myNIC")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetwork(network)
    .withSubnet("mySubnet")
    .withPrimaryPrivateIPAddressDynamic()
    .withExistingPrimaryPublicIPAddress(publicIPAddress)
    .create();
```

### <a name="create-the-virtual-machine"></a>Créer la machine virtuelle

Maintenant que vous avez créé l’ensemble des ressources de prise en charge, vous pouvez créer une machine virtuelle.

Pour créer la machine virtuelle, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Creating virtual machine...");
VirtualMachine virtualMachine = azure.virtualMachines()
    .define("myVM")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetworkInterface(networkInterface)
    .withLatestWindowsImage("MicrosoftWindowsServer", "WindowsServer", "2012-R2-Datacenter")
    .withAdminUsername("azureuser")
    .withAdminPassword("Azure12345678")
    .withComputerName("myVM")
    .withExistingAvailabilitySet(availabilitySet)
    .withSize("Standard_DS1")
    .create();
Scanner input = new Scanner(System.in);
System.out.println("Press enter to get information about the VM...");
input.nextLine();
```

> [!NOTE]
> Ce didacticiel crée une machine virtuelle exécutant une version du système d’exploitation Windows Server. Pour en savoir plus sur la sélection d’autres images, consultez [Parcourir et sélectionner des images de machine virtuelle Azure avec Windows PowerShell et l’interface CLI Azure](../linux/cli-ps-findimage.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
> 
>

Si vous souhaitez utiliser un disque existant au lieu d’une image marketplace, utilisez ce code : 

```java
ManagedDisk managedDisk = azure.disks.define("myosdisk")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withWindowsFromVhd("https://mystorage.blob.core.windows.net/vhds/myosdisk.vhd")
    .withSizeInGB(128)
    .withSku(DiskSkuTypes.PremiumLRS)
    .create();

azure.virtualMachines.define("myVM")
    .withRegion(Region.US_EAST)
    .withExistingResourceGroup("myResourceGroup")
    .withExistingPrimaryNetworkInterface(networkInterface)
    .withSpecializedOSDisk(managedDisk, OperatingSystemTypes.Windows)
    .withExistingAvailabilitySet(availabilitySet)
    .withSize(VirtualMachineSizeTypes.StandardDS1)
    .create();
```

## <a name="perform-management-tasks"></a>Effectuer d’autres tâches de gestion

Pendant le cycle de vie d’une machine virtuelle, vous souhaiterez exécuter des tâches de gestion telles que le démarrage, l’arrêt ou la suppression d’une machine virtuelle. En outre, vous souhaiterez peut-être créer du code pour automatiser les tâches répétitives ou complexes.

Quand vous avez besoin d’effectuer une opération avec la machine virtuelle, vous devez obtenir une instance de celle-ci. Ajoutez ce code au bloc try de la méthode main :

```java
VirtualMachine vm = azure.virtualMachines().getByResourceGroup("myResourceGroup", "myVM");
```

### <a name="get-information-about-the-vm"></a>Obtenir des informations sur la machine virtuelle

Pour obtenir des informations sur la machine virtuelle, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("hardwareProfile");
System.out.println("    vmSize: " + vm.size());
System.out.println("storageProfile");
System.out.println("  imageReference");
System.out.println("    publisher: " + vm.storageProfile().imageReference().publisher());
System.out.println("    offer: " + vm.storageProfile().imageReference().offer());
System.out.println("    sku: " + vm.storageProfile().imageReference().sku());
System.out.println("    version: " + vm.storageProfile().imageReference().version());
System.out.println("  osDisk");
System.out.println("    osType: " + vm.storageProfile().osDisk().osType());
System.out.println("    name: " + vm.storageProfile().osDisk().name());
System.out.println("    createOption: " + vm.storageProfile().osDisk().createOption());
System.out.println("    caching: " + vm.storageProfile().osDisk().caching());
System.out.println("osProfile");
System.out.println("    computerName: " + vm.osProfile().computerName());
System.out.println("    adminUserName: " + vm.osProfile().adminUsername());
System.out.println("    provisionVMAgent: " + vm.osProfile().windowsConfiguration().provisionVMAgent());
System.out.println("    enableAutomaticUpdates: " + vm.osProfile().windowsConfiguration().enableAutomaticUpdates());
System.out.println("networkProfile");
System.out.println("    networkInterface: " + vm.primaryNetworkInterfaceId());
System.out.println("vmAgent");
System.out.println("  vmAgentVersion: " + vm.instanceView().vmAgent().vmAgentVersion());
System.out.println("    statuses");
for(InstanceViewStatus status : vm.instanceView().vmAgent().statuses()) {
    System.out.println("    code: " + status.code());
    System.out.println("    displayStatus: " + status.displayStatus());
    System.out.println("    message: " + status.message());
    System.out.println("    time: " + status.time());
}
System.out.println("disks");
for(DiskInstanceView disk : vm.instanceView().disks()) {
    System.out.println("  name: " + disk.name());
    System.out.println("  statuses");
    for(InstanceViewStatus status : disk.statuses()) {
        System.out.println("    code: " + status.code());
        System.out.println("    displayStatus: " + status.displayStatus());
        System.out.println("    time: " + status.time());
    }
}
System.out.println("VM general status");
System.out.println("  provisioningStatus: " + vm.provisioningState());
System.out.println("  id: " + vm.id());
System.out.println("  name: " + vm.name());
System.out.println("  type: " + vm.type());
System.out.println("VM instance status");
for(InstanceViewStatus status : vm.instanceView().statuses()) {
    System.out.println("  code: " + status.code());
    System.out.println("  displayStatus: " + status.displayStatus());
}
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="stop-the-vm"></a>Arrêtez la machine virtuelle.

Vous pouvez arrêter une machine virtuelle et conserver tous ses paramètres, mais continuer à être facturé, ou arrêter une machine virtuelle et la libérer. Lorsqu’une machine virtuelle est libérée, toutes les ressources qui lui sont associées sont également libérées et la facturation de la machine virtuelle prend fin.

Pour arrêter la machine virtuelle sans la libérer, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Stopping vm...");
vm.powerOff();
System.out.println("Press enter to continue...");
input.nextLine();
```

Si vous souhaitez libérer la machine virtuelle, modifiez l’appel de mise hors tension avec ce code :

```java
vm.deallocate();
```

### <a name="start-the-vm"></a>Démarrer la machine virtuelle

Pour démarrer la machine virtuelle, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Starting vm...");
vm.start();
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="resize-the-vm"></a>Redimensionner la machine virtuelle

De nombreux aspects du déploiement doivent être pris en considération lors du choix d’une taille pour votre machine virtuelle. Pour plus d’informations, voir [Tailles des machines virtuelles](../sizes.md).  

Pour modifier la taille de la machine virtuelle, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Resizing vm...");
vm.update()
    .withSize(VirtualMachineSizeTypes.STANDARD_DS2)
    .apply();
System.out.println("Press enter to continue...");
input.nextLine();
```

### <a name="add-a-data-disk-to-the-vm"></a>Ajouter un disque de données à la machine virtuelle

Pour ajouter à la machine virtuelle un disque de données d’une taille de 2 Go, avec un numéro d’unité logique de 0 et un type de mise en cache Lecture/écriture, ajoutez ce code au bloc try de la méthode main :

```java
System.out.println("Adding data disk...");
vm.update()
    .withNewDataDisk(2, 0, CachingTypes.READ_WRITE)
    .apply();
System.out.println("Press enter to delete resources...");
input.nextLine();
```

## <a name="delete-resources"></a>Supprimer des ressources

Étant donné que les ressources utilisées dans Microsoft Azure vous sont facturées, il est toujours conseillé de supprimer les ressources qui ne sont plus nécessaires. Si vous souhaitez supprimer les machines virtuelles et l’ensemble des ressources de prise en charge, il vous suffit de supprimer le groupe de ressources.

1. Pour supprimer le groupe de ressources, ajoutez ce code au bloc try de la méthode main :
   
    ```java
    System.out.println("Deleting resources...");
    azure.resourceGroups().deleteByName("myResourceGroup");
    ```

2. Enregistrez le fichier App.java.

## <a name="run-the-application"></a>Exécution de l'application

L’exécution complète de cette application console devrait durer cinq minutes environ.

1. Pour exécuter l’application, utilisez cette commande Maven :

    ```
    mvn compile exec:java
    ```

2. Avant d’appuyer sur **Entrée** pour démarrer la suppression des ressources, prenez quelques minutes pour vérifier que les ressources dans le portail Azure ont bien été créées. Cliquez sur l’état du déploiement pour afficher des informations sur le déploiement.


## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur l’utilisation des [bibliothèques Azure pour Java](/java/azure/java-sdk-azure-overview).