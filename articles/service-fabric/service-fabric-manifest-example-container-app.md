---
title: Exemples de manifestes d’application conteneur Azure Service Fabric
description: Découvrez comment configurer les paramètres des manifestes d’applications et de services pour une application Service Fabric multiconteneur.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/08/2018
ms.author: pepogors
ms.openlocfilehash: 5efdbd17db20e69ace33b379ddbb99b2c4a20e69
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84701164"
---
# <a name="multi-container-application-and-service-manifest-examples"></a>Exemples de manifestes d’application pour plusieurs conteneurs et de service
Le document suivant fournit des exemples de manifestes d’applications et de services pour une application Service Fabric multiconteneur. Ces exemples ont pour but de vous montrer les paramètres disponibles et leur utilisation. Ces manifestes d’applications et de services sont basés sur les manifestes de l’[exemple de conteneur Windows Server 2016](https://github.com/Azure-Samples/service-fabric-containers/tree/master/Windows).

Les fonctionnalités sont les suivantes :

|Manifeste|Fonctionnalités|
|---|---|
|[Manifeste d’application](#application-manifest)| [Écraser des variables d’environnement](service-fabric-get-started-containers.md#configure-and-set-environment-variables), [Configuration du mappage des ports de conteneur à l’hôte](service-fabric-get-started-containers.md#configure-container-port-to-host-port-mapping-and-container-to-container-discovery), [Configurer l’authentification au registre de conteneurs](service-fabric-get-started-containers.md#configure-container-repository-authentication), [Gouvernance des ressources](service-fabric-resource-governance.md), [Définir le mode d’isolation](service-fabric-get-started-containers.md#configure-isolation-mode), [Spécifier les images conteneur spécifiques au build du système d’exploitation](service-fabric-get-started-containers.md#specify-os-build-specific-container-images)| 
|[Manifeste de service FrontEndService](#frontendservice-service-manifest)| [Définir des variables d’environnement](service-fabric-get-started-containers.md#configure-and-set-environment-variables), [Configurer un point de terminaison](service-fabric-get-started-containers.md#configure-communication), transmettre des commandes au conteneur, [Importer un certificat dans un conteneur](service-fabric-securing-containers.md)| 
|[Manifeste de service BackEndService](#backendservice-service-manifest)|[Définir des variables d’environnement](service-fabric-get-started-containers.md#configure-and-set-environment-variables), [Configurer un point de terminaison](service-fabric-get-started-containers.md#configure-communication), [Configurer le pilote de volume](service-fabric-containers-volume-logging-drivers.md)| 

Pour en savoir plus sur les éléments XML spécifiques, consultez les articles [Éléments de manifeste de l’application](#application-manifest-elements), [Éléments de manifeste de service FrontEndService](#frontendservice-service-manifest-elements) et [Éléments de manifeste de service BackEndService](#backendservice-service-manifest-elements).

## <a name="application-manifest"></a>Manifeste d’application

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="Container.ApplicationType"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="BackEndService_InstanceCount" DefaultValue="-1" />
    <Parameter Name="FrontEndService_InstanceCount" DefaultValue="-1" />
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="BlockIOWeight" DefaultValue="200" />
    <Parameter Name="MaximumIOBandwidth" DefaultValue="1024" />
    <Parameter Name="MemoryReservationInMB" DefaultValue="1024" />
    <Parameter Name="MemorySwapInMB" DefaultValue="4084"/>
    <Parameter Name="MaximumIOps" DefaultValue="20"/>
    <Parameter Name="MemoryFront" DefaultValue="4084" />
    <Parameter Name="MemoryBack" DefaultValue="2048" />
    <Parameter Name="CertThumbprint" DefaultValue=""/>
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="BackEndServicePkg" ServiceManifestVersion="1.0.0" />    
    
    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[MemoryFront]"/>

      <!-- Set resource governance at the code package level. -->
      <ResourceGovernancePolicy CodePackageRef="Code" CpuPercent="10" MemoryInMB="[MemoryFront]" BlockIOWeight="[BlockIOWeight]" MaximumIOBandwidth="[MaximumIOBandwidth]" MaximumIOps="[MaximumIOps]" MemoryReservationInMB="[MemoryReservationInMB]" MemorySwapInMB="[MemorySwapInMB]"/>
      
      <!-- Policies for activating container hosts. -->
      <ContainerHostPolicies CodePackageRef="Code" Isolation="process">
        
        <!-- Credentials for the repository hosting the container image.-->
        <RepositoryCredentials AccountName="sfsamples" Password="ENCRYPTED-PASSWORD" PasswordEncrypted="true"/>
        
        <!-- This binds the port the container is listening on (8905 in this sample) to an endpoint resource named "BackEndServiceTypeEndpoint", which is defined in the service manifest.  -->
        <PortBinding ContainerPort="8905" EndpointRef="BackEndServiceTypeEndpoint"/>
        
        <!-- Configure the Azure Files volume plugin.  Bind the source folder on the host VM or a remote share to the destination folder within the running container. -->
        <Volume Source="azfiles" Destination="c:\VolumeTest\Data" Driver="sfazurefile">
          <!-- Driver options to be passed to driver. The Azure Files volume plugin supports the following driver options:
            shareName (the Azure Files file share that provides the volume for the container), storageAccountName (the Azure storage account
            that contains the Azure Files file share), storageAccountKey (Access key for the Azure storage account that contains the Azure Files file share).
            These three driver options are required. -->
          <DriverOption Name="shareName" Value="" />
          <DriverOption Name="storageAccountName" Value="MY-STORAGE-ACCOUNT-NAME" />
          <DriverOption Name="storageAccountKey" Value="MY-STORAGE-ACCOUNT-KEY" />
        </Volume>
        
        <!-- Windows Server containers may not be compatible across different versions of the OS.  You can specify multiple OS images per container and tag 
        them with the build versions of the OS. Get the build version of the OS by running "winver" at a Windows command prompt. -->
        <ImageOverrides>
          <!-- If the underlying OS is build version 16299 (Windows Server version 1709), Service Fabric picks the container image tagged with Os="16299". -->
          <Image Name="sfsamples.azurecr.io/sfsamples/servicefabricbackendservice_1709" Os="16299" />
          
          <!-- An untagged container image is assumed to work across all versions of the OS and overrides the image specified in the service manifest. -->
          <Image Name="sfsamples.azurecr.io/sfsamples/servicefabricbackendservice_default" />          
        </ImageOverrides>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>

  <!-- Policies to be applied to the imported service manifest. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="FrontEndServicePkg" ServiceManifestVersion="1.0.0" />
    
    <!-- This enables you to provide different values for environment variables when creating a FrontEndService
         Theses environment variables are declared in the FrontEndServiceType service manifest-->
    <EnvironmentOverrides CodePackageRef="Code">
      <EnvironmentVariable Name="BackendServiceName" Value="Container.Application/BackEndService"/>
      <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
      <EnvironmentVariable Name="IsContainer" Value="true"/>
    </EnvironmentOverrides>
    
    <!-- This policy maps the  port of the container (80) to the endpoint declared in the service, 
         FrontEndServiceTypeEndpoint which is exposed as port 80 on the host-->    
    <Policies>

      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[MemoryBack]"/>

      <!-- Policies for activating container hosts. -->
      <ContainerHostPolicies CodePackageRef="Code" Isolation="process">

        <!-- Credentials for the repository hosting the container image.-->
        <RepositoryCredentials AccountName="sfsamples" Password="ENCRYPTED-PASSWORD" PasswordEncrypted="true"/>

        <!-- Binds an endpoint resource (declared in the service manifest) to the exposed container port. -->
        <PortBinding ContainerPort="80" EndpointRef="FrontEndServiceTypeEndpoint"/>

        <!-- Import a certificate into the container.  The certificate must be installed in the LocalMachine store of all the cluster nodes.
          When the application starts, the runtime reads the certificate and generates a PFX file and password (on Windows) or a PEM file (on Linux).
          The PFX file and password are accessible in the container using the Certificates_ServicePackageName_CodePackageName_CertName_PFX and 
          Certificates_ServicePackageName_CodePackageName_CertName_Password environment variables. The PEM file is accessible in the container using the 
          Certificates_ServicePackageName_CodePackageName_CertName_PEM and Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey environment variables.-->
        <CertificateRef Name="MyCert1" X509StoreName="My" X509FindValue="[CertThumbprint]" />

        <!-- If the certificate is already in PFX or PEM form, you can create a data package inside your application and reference that certificate here. -->
        <CertificateRef Name="MyCert2" DataPackageRef="Data" DataPackageVersion="1.0.0" RelativePath="MyCert2.PFX" Password="ENCRYPTED-PASSWORD" IsPasswordEncrypted="true"/>
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>
  
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
        
    <Service Name="FrontEndService" >
      <StatelessService ServiceTypeName="FrontEndServiceType" InstanceCount="[FrontEndService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
        <Service Name="BackEndService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="BackEndServiceType" InstanceCount="[BackEndService_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

## <a name="frontendservice-service-manifest"></a>Manifeste de service FrontEndService

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="FrontEndServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="FrontEndServiceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ContainerHost>
        <!--The repo and image on https://hub.docker.com or Azure Container Registry. -->
        <ImageName>sfsamples.azurecr.io/sfsamples/servicefabricfrontendservice:v1</ImageName>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container or exe.  These variables are overridden in the application manifest. -->
    <EnvironmentVariables>
      <EnvironmentVariable Name="BackendServiceName" Value=""/>
      <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
      <EnvironmentVariable Name="IsContainer" Value=""/>
    </EnvironmentVariables>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />
  
  <!-- Data package is the contents of the Data directory under PackageRoot that contains an 
       independently-updateable and versioned static data that's consumed by the process at runtime. -->
  <DataPackage Name="Data" Version="1.0.0"/>

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. For a guest executable is used to register with the NamingService at its REST endpoint
           with http scheme and port 80 -->
      <Endpoint Name="FrontEndServiceTypeEndpoint" UriScheme="http" Port="80"/>
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="backendservice-service-manifest"></a>Manifeste de service BackEndService

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="BackEndServicePkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         The UseImplicitHost attribute indicates this is a guest service. -->
    <StatelessServiceType ServiceTypeName="BackEndServiceType" UseImplicitHost="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ContainerHost>
        <!--The repo and image on https://hub.docker.com or Azure Container Registry. -->
        <ImageName>sfsamples.azurecr.io/sfsamples/servicefabricbackendservice:v1</ImageName>
        
        <!-- Pass comma delimited commands to your container. -->
        <Commands> dotnet, myproc.dll, 5 </Commands>
      </ContainerHost>
    </EntryPoint>
    <!-- Pass environment variables to your container. These variables are overridden in the application manifest. -->
    <EnvironmentVariables>
      <EnvironmentVariable Name="IsContainer" Value="true"/>
    </EnvironmentVariables>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the host port on which to 
           listen. For a guest executable is used to register with the NamingService at its REST endpoint
           with http scheme. In this case since no port is specified, one is created and assigned dynamically
           to the service. This dynamically assigned host port is mapped to the container port (8905 in this sample),
            which was specified in the application manifest.-->
      <Endpoint Name="BackEndServiceTypeEndpoint" UriScheme="http" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="application-manifest-elements"></a>Éléments de manifeste de l’application
### <a name="applicationmanifest-element"></a>Élément ApplicationManifest
Décrit de façon déclarative la version et le type d’application. Un ou plusieurs manifestes des services constitutifs sont référencés pour composer un type d’application. Les paramètres de configuration des services constitutifs peuvent être substitués à l’aide de paramètres d’application paramétrables. Les services, modèles de service, entités, stratégies, configuration de diagnostics et certificats par défaut peuvent également être déclarés au niveau de l’application. Pour plus d’informations, consultez l’article [Élément ApplicationManifest](service-fabric-service-model-schema-elements.md#ApplicationManifestElementApplicationManifestTypeComplexType).

### <a name="parameters-element"></a>Élément Parameters
Déclare les paramètres utilisés dans ce manifeste d’application. La valeur de ces paramètres peut être fournie quand l’application est instanciée, et peut être utilisée pour substituer les paramètres de configuration de service ou d’application. Pour plus d’informations, consultez l’article [Élément Parameters](service-fabric-service-model-schema-elements.md#ParametersElementanonymouscomplexTypeComplexTypeDefinedInApplicationManifestTypecomplexType).

### <a name="parameter-element"></a>Élément Parameter
Paramètre d’application à utiliser dans ce manifeste. La valeur du paramètre peut être modifiée lors de l’instanciation de l’application ou, si aucune valeur n’est fournie, la valeur par défaut est utilisée. Pour plus d’informations, consultez l’article [Élément Parameter](service-fabric-service-model-schema-elements.md#ParameterElementanonymouscomplexTypeComplexTypeDefinedInParameterselement).

### <a name="servicemanifestimport-element"></a>Élément ServiceManifestImport
Importe un manifeste de service créé par le développeur de service. Un manifeste de service doit être importé pour chaque service constitutif de l’application. Des substitutions et des stratégies de configuration peuvent être déclarées pour le manifeste de service. Pour plus d’informations, consultez l’article [Élément ServiceManifestImport](service-fabric-service-model-schema-elements.md#ServiceManifestImportElementanonymouscomplexTypeComplexTypeDefinedInApplicationManifestTypecomplexType).

### <a name="servicemanifestref-element"></a>Élément ServiceManifestRef
Importe le manifeste de service par référence. Actuellement, le fichier manifeste de service (ServiceManifest.xml) doit être présent dans le package de build. Pour plus d’informations, consultez l’article [Élément ServiceManifestRef](service-fabric-service-model-schema-elements.md#ServiceManifestRefElementServiceManifestRefTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="policies-element"></a>Élément Policies
Décrit les stratégies (liaison de point de terminaison, partage de package, exécuter en tant que, et accès et sécurité) à appliquer sur le manifeste de service importé. Pour plus d’informations, consultez l’article [Élément Policies](service-fabric-service-model-schema-elements.md#PoliciesElementServiceManifestImportPoliciesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="servicepackageresourcegovernancepolicy-element"></a>Élément ServicePackageResourceGovernancePolicy
Définit la stratégie de gouvernance des ressources qui est appliquée au niveau du package de service entier. Pour plus d’informations, consultez l’article [Élément ServicePackageResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ServicePackageResourceGovernancePolicyElementServicePackageResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInServicePackageTypecomplexType).

### <a name="resourcegovernancepolicy-element"></a>Élément ResourceGovernancePolicy
Spécifie les limites de ressources d’un package de code. Pour plus d’informations, consultez l’article [Élément ResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ResourceGovernancePolicyElementResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelementDefinedInDigestedEndpointelement).

### <a name="containerhostpolicies-element"></a>Élément ContainerHostPolicies
Spécifie des stratégies pour l’activation des hôtes de conteneur. Pour plus d’informations, consultez l’article [Élément ContainerHostPolicies](service-fabric-service-model-schema-elements.md#ContainerHostPoliciesElementContainerHostPoliciesTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="repositorycredentials-element"></a>Élément RepositoryCredentials
Informations d’identification pour le référentiel d’images de conteneur à partir duquel extraire des images. Pour plus d’informations, consultez l’article [Élément RepositoryCredentials](service-fabric-service-model-schema-elements.md#RepositoryCredentialsElementRepositoryCredentialsTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType)

### <a name="portbinding-element"></a>Élément PortBinding
Spécifie la ressource de point de terminaison à utiliser pour lier le port du conteneur exposé. Pour plus d’informations, consultez l’article [Élément ContainerHostPolicies](service-fabric-service-model-schema-elements.md#PortBindingElementPortBindingTypeComplexTypeDefinedInServicePackageContainerPolicyTypecomplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="volume-element"></a>Élément Volume
Spécifie le volume à lier au conteneur. Pour plus d’informations, consultez l’article [Élément Volume](service-fabric-service-model-schema-elements.md#VolumeElementContainerVolumeTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="driveroption-element"></a>Élément DriverOption
Options de pilote à passer au pilote. Pour plus d’informations, consultez l’article [Élément DriverOption](service-fabric-service-model-schema-elements.md#DriverOptionElementDriverOptionTypeComplexTypeDefinedInContainerLoggingDriverTypecomplexTypeDefinedInContainerVolumeTypecomplexType).

### <a name="imageoverrides-element"></a>Élément ImageOverrides
Les conteneurs Windows Server peuvent ne pas être compatibles entre les différentes versions du système d’exploitation.  Vous pouvez spécifier plusieurs images du système d’exploitation par conteneur et les marquer avec les versions de build du système d’exploitation. Obtenez le numéro de build du système d’exploitation en exécutant « winver » dans une invite de commandes Windows. Si la version de build du système d’exploitation sous-jacent est 16299 (Windows Server version 1709), Service Fabric choisit l’image de conteneur marquée Os="16299". Une image de conteneur non marquée est censée fonctionner sur toutes les versions du système d’exploitation et remplace l’image spécifiée dans le manifeste de service. Pour plus d’informations, consultez l’article [Élément ImageOverrides](service-fabric-service-model-schema-elements.md#ImageOverridesElementImageOverridesTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="image-element"></a>Élément Image
Image de conteneur correspondant au numéro de build du système d’exploitation à lancer. Si l’attribut du système d’exploitation n’est pas spécifié, l’image de conteneur est censée fonctionner sur toutes les versions du système d’exploitation et remplace l’image spécifiée dans le manifeste de service. Pour plus d’informations, consultez l’article [Élément Image](service-fabric-service-model-schema-elements.md#ImageElementImageTypeComplexTypeDefinedInImageOverridesTypecomplexType).

### <a name="environmentoverrides-element"></a>Élément EnvironmentOverrides
 Pour plus d’informations, consultez l’article [Élément EnvironmentOverrides](service-fabric-service-model-schema-elements.md#EnvironmentOverridesElementEnvironmentOverridesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="environmentvariable-element"></a>Élément EnvironmentVariable
Variable d’environnement. Pour plus d’informations, consultez l’article [Élément EnvironmentVariable](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableOverrideTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexType).

### <a name="certificateref-element"></a>Élément CertificateRef
Spécifie des informations sur un certificat X509 qui doit être exposé à l’environnement de conteneur. Le certificat doit être installé dans le magasin LocalMachine de tous les nœuds du cluster.
Lors du démarrage de l’application, le runtime lit le certificat et génère un fichier PFX ainsi qu’un mot de passe (Windows) ou un fichier PEM (Linux).
Le fichier PFX et le mot de passe sont accessibles dans le conteneur en utilisant les variables d’environnement Certificates_ServicePackageName_CodePackageName_CertName_PFX et Certificates_ServicePackageName_CodePackageName_CertName_Password. Le fichier PEM est accessible dans le conteneur en utilisant les variables d’environnement Certificates_ServicePackageName_CodePackageName_CertName_PEM et Certificates_ServicePackageName_CodePackageName_CertName_PrivateKey. Pour plus d’informations, consultez l’article [Élément CertificateRef](service-fabric-service-model-schema-elements.md#CertificateRefElementContainerCertificateTypeComplexTypeDefinedInContainerHostPoliciesTypecomplexType).

### <a name="defaultservices-element"></a>Élément DefaultServices
Déclare des instances de service qui sont créées automatiquement chaque fois qu’une application est instanciée par rapport à ce type d’application. Pour plus d’informations, consultez l’article [Élément DefaultServices](service-fabric-service-model-schema-elements.md#DefaultServicesElementDefaultServicesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInApplicationInstanceTypecomplexType).

### <a name="service-element"></a>Élément Service
Déclare un service devant être créé automatiquement quand l’application est instanciée. Pour plus d’informations, consultez l’article [Élément Service](service-fabric-service-model-schema-elements.md#ServiceElementanonymouscomplexTypeComplexTypeDefinedInDefaultServicesTypecomplexType).

### <a name="statelessservice-element"></a>Élément StatelessService
Définit un service sans état. Pour plus d’informations, consultez l’article [Élément StatelessService](service-fabric-service-model-schema-elements.md#StatelessServiceElementStatelessServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement).


## <a name="frontendservice-service-manifest-elements"></a>Éléments de manifeste de service FrontEndService
### <a name="servicemanifest-element"></a>Élément ServiceManifest
Décrit de façon déclarative le type et la version du service. Il répertorie les packages de code, de configuration et de données pouvant être mis à niveau indépendamment et qui constituent ensemble un package de service pour prendre en charge un ou plusieurs types de service. Les ressources, paramètres de diagnostics et métadonnées de service, telles que le type de service, les propriétés d’intégrité et les métriques d’équilibrage de charge, sont également spécifiés. Pour plus d’informations, consultez l’article [Élément ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Élément ServiceTypes
Définit les types de service pris en charge par un CodePackage dans ce manifeste. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les types de service sont déclarés au niveau du manifeste et non au niveau du package de code. Pour plus d’informations, consultez l’article [Élément ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statelessservicetype-element"></a>Élément StatelessServiceType
Décrit un type de service sans état. Pour plus d’informations, consultez l’article [Élément StatelessServiceType](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Élément CodePackage
Décrit un package de code qui prend en charge un type de service défini. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les processus qui en résultent sont censés inscrire les types de service pris en charge au moment de l'exécution. S’il existe plusieurs packages de code, ils sont tous activés chaque fois que le système recherche l’un des types de service déclarés. Pour plus d’informations, consultez l’article [Élément CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="entrypoint-element"></a>Élément EntryPoint
Le fichier exécutable spécifié par EntryPoint est généralement l’hôte de service à exécution longue. La présence d’un point d’entrée de configuration distinct évite d’avoir à exécuter l’hôte de service avec des privilèges élevés pendant de longues périodes de temps. Le fichier exécutable spécifié par EntryPoint est exécuté une fois que SetupEntryPoint se termine correctement. Le processus résultant fait l’objet d’une surveillance et est redémarré (à partir de SetupEntryPoint) en cas d’interruption ou de défaillance. Pour plus d’informations, consultez l’article [Élément EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="containerhost-element"></a>Élément ContainerHost
 Pour plus d’informations, consultez l’article [Élément ContainerHost](service-fabric-service-model-schema-elements.md#ContainerHostElementContainerHostEntryPointTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="imagename-element"></a>Élément ImageName
Référentiel et image sur [ https://hub.docker.com ](https://hub.docker.com) ou Azure Container Registry. Pour plus d’informations, consultez l’article [Élément ImageName](service-fabric-service-model-schema-elements.md#ImageNameElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType).

### <a name="environmentvariables-element"></a>Élément EnvironmentVariables
Passez les variables d'environnement à votre conteneur ou fichier exe.  Pour plus d’informations, consultez l’article [Élément EnvironmentVariables](service-fabric-service-model-schema-elements.md#EnvironmentVariablesElementEnvironmentVariablesTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="environmentvariable-element"></a>Élément EnvironmentVariable
Variable d’environnement. Pour plus d’informations, consultez l’article [Élément EnvironmentVariable](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableOverrideTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexType).

### <a name="configpackage-element"></a>Élément ConfigPackage
Déclare un dossier, nommé par l’attribut Name, qui contient un fichier Settings.xml. Ce fichier contient des sections de paramètres clé-valeur définis par l'utilisateur que le processus peut lire pendant l'exécution. Le processus en cours d’exécution n’est pas redémarré pendant la mise à niveau si seule la version de ConfigPackage a changé. Au lieu de cela, un rappel indique au processus que les paramètres de configuration ont été modifiés afin qu'ils puissent être rechargés dynamiquement. Pour plus d’informations, consultez l’article [Élément ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="datapackage-element"></a>Élément DataPackage
Déclare un dossier, nommé par l’attribut Name, qui contient des fichiers de données statiques. Service Fabric recycle tous les fichiers EXE et DLLHOST spécifiés dans les packages hôtes et de prise en charge quand l’un des packages de données répertoriés dans le manifeste de service est mis à niveau. Pour plus d’informations, consultez l’article [Élément DataPackage](service-fabric-service-model-schema-elements.md#DataPackageElementDataPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedDataPackageelement).

### <a name="resources-element"></a>Élément Resources
Décrit les ressources utilisées par ce service, qui peuvent être déclarées sans modifier le code compilé et changées quand le service est déployé. L’accès à ces ressources est contrôlé par le biais des sections Principals et Policies du manifeste d’application. Pour plus d’informations, consultez l’article [Élément Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Élément Endpoints
Définit des points de terminaison pour le service. Pour plus d’informations, consultez l’article [Élément Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Élément Endpoint
Pour plus d’informations, consultez l’article [Élément Endpoint](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).


## <a name="backendservice-service-manifest-elements"></a>Éléments de manifeste de service BackEndService
### <a name="servicemanifest-element"></a>Élément ServiceManifest
Décrit de façon déclarative le type et la version du service. Il répertorie les packages de code, de configuration et de données pouvant être mis à niveau indépendamment et qui constituent ensemble un package de service pour prendre en charge un ou plusieurs types de service. Les ressources, paramètres de diagnostics et métadonnées de service, telles que le type de service, les propriétés d’intégrité et les métriques d’équilibrage de charge, sont également spécifiés. Pour plus d’informations, consultez l’article [Élément ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Élément ServiceTypes
Définit les types de service pris en charge par un CodePackage dans ce manifeste. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les types de service sont déclarés au niveau du manifeste et non au niveau du package de code. Pour plus d’informations, consultez l’article [Élément ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statelessservicetype-element"></a>Élément StatelessServiceType
Décrit un type de service sans état. Pour plus d’informations, consultez l’article [Élément StatelessServiceType](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Élément CodePackage
Décrit un package de code qui prend en charge un type de service défini. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les processus qui en résultent sont censés inscrire les types de service pris en charge au moment de l'exécution. S’il existe plusieurs packages de code, ils sont tous activés chaque fois que le système recherche l’un des types de service déclarés. Pour plus d’informations, consultez l’article [Élément CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="entrypoint-element"></a>Élément EntryPoint
Le fichier exécutable spécifié par EntryPoint est généralement l’hôte de service à exécution longue. La présence d’un point d’entrée de configuration distinct évite d’avoir à exécuter l’hôte de service avec des privilèges élevés pendant de longues périodes de temps. Le fichier exécutable spécifié par EntryPoint est exécuté une fois que SetupEntryPoint se termine correctement. Le processus résultant fait l’objet d’une surveillance et est redémarré (à partir de SetupEntryPoint) en cas d’interruption ou de défaillance. Pour plus d’informations, consultez l’article [Élément EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="containerhost-element"></a>Élément ContainerHost
Pour plus d’informations, consultez l’article [Élément ContainerHost](service-fabric-service-model-schema-elements.md#ContainerHostElementContainerHostEntryPointTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="imagename-element"></a>Élément ImageName
Référentiel et image sur [ https://hub.docker.com ](https://hub.docker.com) ou Azure Container Registry. Pour plus d’informations, consultez l’article [Élément ImageName](service-fabric-service-model-schema-elements.md#ImageNameElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType).

### <a name="commands-element"></a>Élément Commands
Passez une liste de commandes séparées par des virgules au conteneur. Pour plus d’informations, consultez l’article [Élément Commands](service-fabric-service-model-schema-elements.md#CommandsElementxs:stringComplexTypeDefinedInContainerHostEntryPointTypecomplexType).

### <a name="environmentvariables-element"></a>Élément EnvironmentVariables
Passez les variables d'environnement à votre conteneur ou fichier exe.  Pour plus d’informations, consultez l’article [Élément EnvironmentVariables](service-fabric-service-model-schema-elements.md#EnvironmentVariablesElementEnvironmentVariablesTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="environmentvariable-element"></a>Élément EnvironmentVariable
Variable d’environnement. Pour plus d’informations, consultez l’article [Élément EnvironmentVariable](service-fabric-service-model-schema-elements.md#EnvironmentVariableElementEnvironmentVariableOverrideTypeComplexTypeDefinedInEnvironmentOverridesTypecomplexType).

### <a name="configpackage-element"></a>Élément ConfigPackage
Déclare un dossier, nommé par l’attribut Name, qui contient un fichier Settings.xml. Ce fichier contient des sections de paramètres clé-valeur définis par l'utilisateur que le processus peut lire pendant l'exécution. Le processus en cours d’exécution n’est pas redémarré pendant la mise à niveau si seule la version de ConfigPackage a changé. Au lieu de cela, un rappel indique au processus que les paramètres de configuration ont été modifiés afin qu'ils puissent être rechargés dynamiquement. Pour plus d’informations, consultez l’article [Élément ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="resources-element"></a>Élément Resources
Décrit les ressources utilisées par ce service, qui peuvent être déclarées sans modifier le code compilé et changées quand le service est déployé. L’accès à ces ressources est contrôlé par le biais des sections Principals et Policies du manifeste d’application. Pour plus d’informations, consultez l’article [Élément Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Élément Endpoints
Définit des points de terminaison pour le service. Pour plus d’informations, consultez l’article [Élément Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Élément Endpoint
 Pour plus d’informations, consultez l’article [Élément Endpoint](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).

