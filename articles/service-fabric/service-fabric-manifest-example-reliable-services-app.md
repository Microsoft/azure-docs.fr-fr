---
title: Exemples de manifeste de l’application Reliable Services
description: Découvrez comment configurer les paramètres des manifestes d’applications et de services pour une application Service Fabric de services fiables.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/11/2018
ms.author: pepogors
ms.openlocfilehash: f40e54f5260f827f0b18c833d23d1f57b5ebc3a3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84701096"
---
# <a name="reliable-services-application-and-service-manifest-examples"></a>Exemples de manifestes d’application de services fiables et de service
Voici quelques exemples de manifestes d’applications et de services pour une application Service Fabric avec un serveur web ASP.NET Core frontal et un serveur backend avec état. Ces exemples ont pour but de vous montrer les paramètres disponibles et leur utilisation. Ces manifestes d’applications et de services sont basés sur les manifestes du [guide de démarrage rapide Service Fabric .NET](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/).

Les fonctionnalités sont les suivantes :

|Manifeste|Fonctionnalités|
|---|---|
|[Manifeste d’application](#application-manifest)| [Gouvernance des ressources](service-fabric-resource-governance.md), [Exécuter un service en tant que compte administrateur local](service-fabric-application-runas-security.md), [Application d’une stratégie par défaut à tous les packages de code de service](service-fabric-application-runas-security.md#apply-a-default-policy-to-all-service-code-packages), [Créer des principaux d’utilisateur et de groupe](service-fabric-application-runas-security.md), partager un package de données entre des instances de service, [Écraser des points de terminaison de service](service-fabric-service-manifest-resources.md#overriding-endpoints-in-servicemanifestxml)| 
|Manifeste de service FrontEndService| [Exécuter un script au démarrage du service](service-fabric-run-script-at-service-startup.md), [Définir un point de terminaison HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest) | 
|Manifeste de service BackEndService| [Déclarer un package de configuration](service-fabric-application-and-service-manifests.md), [Déclarer un package de données](service-fabric-application-and-service-manifests.md), [Configurer un point de terminaison](service-fabric-service-manifest-resources.md)| 

Pour en savoir plus sur les éléments XML spécifiques, consultez les articles [Éléments de manifeste de l’application](#application-manifest-elements), [Éléments de manifeste VotingWeb](#votingweb-service-manifest-elements) et [Éléments de manifeste VotingData](#votingdata-service-manifest-elements).

## <a name="application-manifest"></a>Manifeste d’application

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
    <Parameter Name="CpuCores" DefaultValue="2" />
    <Parameter Name="Memory" DefaultValue="4084" />
    <Parameter Name="BlockIOWeight" DefaultValue="200" />
    <Parameter Name="MaximumIOBandwidth" DefaultValue="1024" />
    <Parameter Name="MemoryReservationInMB" DefaultValue="1024" />
    <Parameter Name="MemorySwapInMB" DefaultValue="4084"/>
    <Parameter Name="Port" DefaultValue="8081" />
    <Parameter Name="Protocol" DefaultValue="tcp" />
    <Parameter Name="Type" DefaultValue="internal" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <!-- Override endpoints declared in the service manifest. -->
    <ResourceOverrides>
      <Endpoints>
        <Endpoint Name="DataEndpoint" Port="[Port]" Protocol="[Protocol]" Type="[Type]" />
      </Endpoints>
    </ResourceOverrides>

    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      
      <!-- Set resource governance at the service package level. -->
      <ServicePackageResourceGovernancePolicy CpuCores="[CpuCores]" MemoryInMB="[Memory]"/>

      <!-- Set resource governance at the code package level. -->
      <ResourceGovernancePolicy CodePackageRef="Code" CpuPercent="10" MemoryInMB="[Memory]" BlockIOWeight="[BlockIOWeight]" 
                                MaximumIOBandwidth="[MaximumIOBandwidth]" MaximumIOps="[MaximumIOps]" MemoryReservationInMB="[MemoryReservationInMB]" 
                                MemorySwapInMB="[MemorySwapInMB]"/>

      <!-- Share the data package across multiple instances of the VotingData service-->
      <PackageSharingPolicy PackageRef="Data"/>

      <!-- Give read rights on the "DataEndpoint" endpoint to the Customer2 account.-->
      <SecurityAccessPolicy GrantRights="Read" PrincipalRef="Customer2" ResourceRef="DataEndpoint" ResourceType="Endpoint"/>         
    </Policies>
  </ServiceManifestImport>
  
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    
    <!-- Policies to be applied to the imported service manifest. -->
    <Policies>
      <!-- Run the setup entry point (defined in the imported service manifest) as the SetupAdminUser account 
      (declared in the following Principals section). -->
      <RunAsPolicy CodePackageRef="Code" UserRef="SetupAdminUser" EntryPointType="Setup" />
      
    </Policies>

  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <!-- Define users and groups required to run the services and access resources.  Principals are used in the Policies section(s). -->
  <Principals>
    <!-- Declare a set of groups as security principals, which can be referenced in policies. Groups are useful if there are multiple users 
    for different service entry points and they need to have certain common privileges that are available at the group level. -->
    <Groups>
      <!-- Create a group that has administrator privileges. -->
      <Group Name="LocalAdminGroup">
        <Membership>
          <SystemGroup Name="Administrators" />
        </Membership>
      </Group>
    </Groups>
    <Users>
      <!-- Declare a user and add the user to the Administrators system group. The SetupAdminUser account is used to run the 
      setup entry point of the VotingWebPkg code package (described in the preceding Policies section).-->
      <User Name="SetupAdminUser">
        <MemberOf>
          <SystemGroup Name="Administrators" />
        </MemberOf>
      </User>
      <!-- Create a user. Local user accounts are created on the machines where the application is deployed. By default, these accounts 
      do not have the same names as those specified here. Instead, they are dynamically generated and have random passwords. -->
      <User Name="Customer1" >
        <MemberOf>
          <!-- Add the user to the local administrators group.-->
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <!-- Create a user as a local user with the specified account name and password. Local user accounts are created on the machines 
      where the application is deployed. -->
      <User Name="Customer2" AccountType="LocalUser" AccountName="Customer1" Password="MyPassword">
        <MemberOf>
          <!-- Add the user to the local administrators group.-->
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <!-- Create a user as NetworkService. -->
      <User Name="MyDefaultAccount" AccountType="NetworkService" />      
    </Users>
  </Principals>
  <!-- Policies applied at the application level. -->
  <Policies>
    <!-- Specify a default user account for all code packages that don’t have a specific RunAsPolicy defined in 
    the ServiceManifestImport section(s). -->
    <DefaultRunAsPolicy UserRef="MyDefaultAccount" />
    
  </Policies>
</ApplicationManifest>

```

## <a name="votingweb-service-manifest"></a>Manifeste de service VotingWeb

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType. 
         This name must match the string used in RegisterServiceType call in Program.cs. -->
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <!-- A privileged entry point that by default runs with the same credentials as Service Fabric (typically the NetworkService account) before 
    any other entry point. Use the setup entry point to set system environment variables, give the account running the service (NETWORK SERVICE, by default) 
    access to a certificate's private key, or perform other setup tasks. In the application manifest, you can change the security permissions to run the startup script 
    under a local system account or an administrator account.  -->
    <SetupEntryPoint>
      <ExeHost>
        <!-- The setup script to run. -->
        <Program>Setup.bat</Program>
        <!-- Pass arguments to the script when it runs.-->
        <Arguments>MyValue</Arguments>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
        <!-- Warning! Do not use console redirection in a production application, only use it for local development and debugging. Redirects console output from the startup
        script to an output file in the application folder called "log" on the cluster node where the application is deployed and run. Also set the number of output files
        to retain and the maximum file size (in KB). -->
        <ConsoleRedirection FileRetentionCount="10" FileMaxSizeInKb="20480"/>
      </ExeHost>
    </SetupEntryPoint>
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- Configure a HTTPS endpoint on port 443. This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Protocol="https" Name="EndpointHttps" Type="Input" Port="443" />
    </Endpoints>
  </Resources>
</ServiceManifest>

```

## <a name="votingdata-service-manifest"></a>Manifeste de service VotingData

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingDataPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType. 
         This name must match the string used in RegisterServiceType call in Program.cs. -->
    <StatefulServiceType ServiceTypeName="VotingDataType"  HasPersistedState="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingData.exe</Program>
        <!-- The working directory for the process in the code package on the node where the application is deployed. CodePackage sets the working directory to be 
        the root of the code package regardless of where the EXE is defined in the code package directory. This is where the processes can write the data. Writing data 
        in the code package or code base is not recommended as those folders could be shared between different application instances and may get deleted.-->
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Declares a folder, named by the Name attribute, under PackageRoot that contains a Settings.xml file. This file contains sections of user-defined, 
  key-value pair settings that the process can read back at run time. During an upgrade, if only the ConfigPackage version has changed, 
  then the running process is not restarted. Instead, a callback notifies the process that configuration settings have changed so they can be reloaded dynamically. -->
  <ConfigPackage Name="Config" Version="1.0.0" />
  
  <!-- Declares a folder, named by the Name attribute, under PackageRoot which contains static data files to be consumed by the process at run time. -->
  <DataPackage Name="Data" Version="1.0.0"/>

  <Resources>
    <Endpoints>
      <!-- Define an internal (used for intra-application communication) TCP endpoint. Since no port is specified, one is created and assigned dynamically
           to the service.-->
      <Endpoint Name="DataEndpoint" Protocol="tcp" Type="Internal" />
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

### <a name="resourceoverrides-element"></a>Élément ResourceOverrides
Spécifie les substitutions de ressources pour les points de terminaison déclarés dans les ressources du manifeste de service. Pour plus d’informations, consultez l’article [Élément ResourceOverrides](service-fabric-service-model-schema-elements.md#ResourceOverridesElementResourceOverridesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="endpoints-element"></a>Élément Endpoints
Le(s) point(s) de terminaison à remplacer. Pour plus d’informations, consultez l’article [Élément Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourceOverridesTypecomplexType).

### <a name="endpoint-element"></a>Élément Endpoint
Le point de terminaison à remplacer, qui est déclaré dans le manifeste de service. Pour plus d’informations, consultez l’article [Élément Endpoint](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).

### <a name="policies-element"></a>Élément Policies
Décrit les stratégies (liaison de point de terminaison, partage de package, exécuter en tant que, et accès et sécurité) à appliquer sur le manifeste de service importé. Pour plus d’informations, consultez l’article [Élément Policies](service-fabric-service-model-schema-elements.md#PoliciesElementServiceManifestImportPoliciesTypeComplexTypeDefinedInServiceManifestImportelement).

### <a name="servicepackageresourcegovernancepolicy-element"></a>Élément ServicePackageResourceGovernancePolicy
Définit la stratégie de gouvernance des ressources qui est appliquée au niveau du package de service entier. Pour plus d’informations, consultez l’article [Élément ServicePackageResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ServicePackageResourceGovernancePolicyElementServicePackageResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInServicePackageTypecomplexType).

### <a name="resourcegovernancepolicy-element"></a>Élément ResourceGovernancePolicy
Spécifie les limites de ressources pour un package de code. Pour plus d’informations, consultez l’article [Élément ResourceGovernancePolicy](service-fabric-service-model-schema-elements.md#ResourceGovernancePolicyElementResourceGovernancePolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelementDefinedInDigestedEndpointelement).

### <a name="packagesharingpolicy-element"></a>Élément PackageSharingPolicy
Indique si un package de code, de configuration ou de données doit être partagé entre les instances de service du même type de service. Pour plus d’informations, consultez l’article [Élément PackageSharingPolicy](service-fabric-service-model-schema-elements.md#PackageSharingPolicyElementPackageSharingPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexType).

### <a name="securityaccesspolicy-element"></a>Élément SecurityAccessPolicy
Accorde des autorisations d’accès à une entité sur une ressource (par exemple un point de terminaison) définie dans un manifeste de service. En règle générale, il est très utile de contrôler et de limiter l’accès des services à différentes ressources afin de réduire les risques de sécurité. Cela est particulièrement important quand l’application est générée à partir d’une collection de services d’une place de marché qui sont développés par plusieurs développeurs. Pour plus d’informations, consultez l’article [Élément SecurityAccessPolicy](service-fabric-service-model-schema-elements.md#SecurityAccessPolicyElementSecurityAccessPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInSecurityAccessPolicieselementDefinedInDigestedEndpointelement).

### <a name="runaspolicy-element"></a>Élément RunAsPolicy
Spécifie le compte d’utilisateur local ou de système local sous lequel s’exécutera un package de code de service. Les comptes de domaine sont pris en charge sur les déploiements de Windows Server où Azure Active Directory est disponible. Par défaut, les applications s’exécutent sous le compte qui exécute le processus Fabric.exe. Les applications peuvent également s’exécuter en tant qu’autres comptes, qui doivent être déclarés dans la section Principals. Si vous appliquez une stratégie RunAs à un service et que le manifeste de service déclare des ressources de point de terminaison avec le protocole HTTP, vous devez aussi spécifier une stratégie SecurityAccessPolicy pour vous assurer que les ports affectés à ces points de terminaison sont bien au format ACL pour le compte utilisateur RunAs sous lequel le service s’exécute. Pour un point de terminaison HTTPS, vous devez également définir un EndpointBindingPolicy pour indiquer le nom du certificat à retourner au client. Pour plus d’informations, consultez l’article [Élément RunAsPolicy](service-fabric-service-model-schema-elements.md#RunAsPolicyElementRunAsPolicyTypeComplexTypeDefinedInServiceManifestImportPoliciesTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="defaultservices-element"></a>Élément DefaultServices
Déclare des instances de service qui sont créées automatiquement chaque fois qu’une application est instanciée par rapport à ce type d’application. Pour plus d’informations, consultez l’article [Élément DefaultServices](service-fabric-service-model-schema-elements.md#DefaultServicesElementDefaultServicesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInApplicationInstanceTypecomplexType).

### <a name="service-element"></a>Élément Service
Déclare un service devant être créé automatiquement quand l’application est instanciée. Pour plus d’informations, consultez l’article [Élément Service](service-fabric-service-model-schema-elements.md#ServiceElementanonymouscomplexTypeComplexTypeDefinedInDefaultServicesTypecomplexType).

### <a name="statefulservice-element"></a>Élément StatefulService
Définit un service avec état. Pour plus d’informations, consultez l’article [Élément StatefulService](service-fabric-service-model-schema-elements.md#StatefulServiceElementStatefulServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement).

### <a name="statelessservice-element"></a>Élément StatelessService
Définit un service sans état. Pour plus d’informations, consultez l’article [Élément StatelessService](service-fabric-service-model-schema-elements.md#StatelessServiceElementStatelessServiceTypeComplexTypeDefinedInServiceTemplatesTypecomplexTypeDefinedInServiceelement).

### <a name="principals-element"></a>Élément Principals
Décrit les entités de sécurité (utilisateurs, groupes) nécessaires pour que cette application exécute des services et ressources sécurisés. Les entités sont référencées dans les sections sur les stratégies. Pour plus d’informations, consultez l’article [Élément Principals](service-fabric-service-model-schema-elements.md#PrincipalsElementSecurityPrincipalsTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInEnvironmentTypecomplexType).

### <a name="groups-element"></a>Élément Groups
Déclare un ensemble de groupes en tant qu’entités de sécurité pouvant être référencées dans des stratégies. Les groupes sont utiles s’il existe plusieurs utilisateurs pour des points d’entrée de service différents et s’ils doivent disposer de certains privilèges courants disponibles au niveau du groupe. Pour plus d’informations, consultez l’article [Élément Groups](service-fabric-service-model-schema-elements.md#GroupsElementanonymouscomplexTypeComplexTypeDefinedInSecurityPrincipalsTypecomplexType).

### <a name="group-element"></a>Élément Group
Déclare un groupe en tant qu’entité de sécurité pouvant être référencée dans des stratégies. Pour plus d’informations, consultez l’article [Élément Group](service-fabric-service-model-schema-elements.md#GroupElementanonymouscomplexTypeComplexTypeDefinedInGroupselement).

### <a name="membership-element"></a>Élément Membership
 Pour plus d’informations, consultez l’article [Élément Membership](service-fabric-service-model-schema-elements.md#MembershipElementanonymouscomplexTypeComplexTypeDefinedInGroupelement).

### <a name="systemgroup-element"></a>Élément SystemGroup
 Pour plus d’informations, consultez l’article [Élément SystemGroup](service-fabric-service-model-schema-elements.md#SystemGroupElementanonymouscomplexTypeComplexTypeDefinedInMembershipelement).

### <a name="users-element"></a>Élément Users
Déclare un ensemble d’utilisateurs en tant qu’entités de sécurité pouvant être référencées dans des stratégies. Pour plus d’informations, consultez l’article [Élément Users](service-fabric-service-model-schema-elements.md#UsersElementanonymouscomplexTypeComplexTypeDefinedInSecurityPrincipalsTypecomplexType).

### <a name="user-element"></a>Élément User
Déclare un utilisateur en tant qu’entité de sécurité pouvant être référencée dans des stratégies. Pour plus d’informations, consultez l’article [Élément User](service-fabric-service-model-schema-elements.md#UserElementanonymouscomplexTypeComplexTypeDefinedInUserselement).

### <a name="memberof-element"></a>Élément MemberOf
Comme des utilisateurs peuvent être ajoutés à tout groupe d’appartenance existant, il peut donc hériter de l’ensemble des propriétés et des paramètres de sécurité de ce groupe d’appartenance. Le groupe d’appartenance peut être utilisé pour sécuriser les ressources externes qui doivent être accessibles à différents services ou au même service (sur une autre machine). Pour plus d’informations, consultez l’article [Élément MemberOf](service-fabric-service-model-schema-elements.md#MemberOfElementanonymouscomplexTypeComplexTypeDefinedInUserelement).

### <a name="systemgroup-element"></a>Élément SystemGroup
Groupe système auquel ajouter l’utilisateur.  Le groupe système doit être défini dans la section Groups. Pour plus d’informations, consultez l’article [Élément SystemGroup](service-fabric-service-model-schema-elements.md#SystemGroupElementanonymouscomplexTypeComplexTypeDefinedInMemberOfelement).

### <a name="group-element"></a>Élément Group
Groupe auquel ajouter l’utilisateur.  Le groupe doit être défini dans la section Groups. Pour plus d’informations, consultez l’article [Élément Group](service-fabric-service-model-schema-elements.md#GroupElementanonymouscomplexTypeComplexTypeDefinedInMemberOfelement).

### <a name="policies-element"></a>Élément Policies
Décrit les stratégies (collecte de journaux, « exécuter en tant que » par défaut, intégrité et sécurité d’accès) à appliquer au niveau de l’application. Pour plus d’informations, consultez l’article [Élément Policies](service-fabric-service-model-schema-elements.md#PoliciesElementApplicationPoliciesTypeComplexTypeDefinedInApplicationManifestTypecomplexTypeDefinedInEnvironmentTypecomplexType).

### <a name="defaultrunaspolicy-element"></a>Élément DefaultRunAsPolicy
Spécifiez un compte d’utilisateur par défaut pour tous les packages de code de service qui n’ont pas de stratégie RunAsPolicy spécifique définie dans la section ServiceManifestImport. Pour plus d’informations, consultez l’article [Élément DefaultRunAsPolicy](service-fabric-service-model-schema-elements.md#DefaultRunAsPolicyElementanonymouscomplexTypeComplexTypeDefinedInApplicationPoliciesTypecomplexType).




## <a name="votingweb-service-manifest-elements"></a>Éléments de manifeste de service VotingWeb
### <a name="servicemanifest-element"></a>Élément ServiceManifest
Décrit de façon déclarative le type et la version du service. Il répertorie les packages de code, de configuration et de données pouvant être mis à niveau indépendamment et qui constituent ensemble un package de service pour prendre en charge un ou plusieurs types de service. Les ressources, paramètres de diagnostics et métadonnées de service, telles que le type de service, les propriétés d’intégrité et les métriques d’équilibrage de charge, sont également spécifiés. Pour plus d’informations, consultez l’article [Élément ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Élément ServiceTypes
Définit les types de service pris en charge par un CodePackage dans ce manifeste. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les types de service sont déclarés au niveau du manifeste et non au niveau du package de code. Pour plus d’informations, consultez l’article [Élément ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statelessservicetype-element"></a>Élément StatelessServiceType
Décrit un type de service sans état. Pour plus d’informations, consultez l’article [Élément StatelessServiceType](service-fabric-service-model-schema-elements.md#StatelessServiceTypeElementStatelessServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Élément CodePackage
Décrit un package de code qui prend en charge un type de service défini. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les processus qui en résultent sont censés inscrire les types de service pris en charge au moment de l'exécution. S’il existe plusieurs packages de code, ils sont tous activés chaque fois que le système recherche l’un des types de service déclarés. Pour plus d’informations, consultez l’article [Élément CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="setupentrypoint-element"></a>Élément SetupEntryPoint
Point d’entrée privilégié qui s’exécute par défaut avec les mêmes informations d’identification que Service Fabric (généralement le compte NETWORKSERVICE) avant tout autre point d’entrée. Le fichier exécutable spécifié par EntryPoint est généralement l’hôte de service à exécution longue. La présence d’un point d’entrée de configuration distinct évite d’avoir à exécuter l’hôte de service avec des privilèges élevés pendant de longues périodes de temps. Pour plus d’informations, consultez l’article [Élément SetupEntryPoint](service-fabric-service-model-schema-elements.md#SetupEntryPointElementanonymouscomplexTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="exehost-element"></a>Élément ExeHost
 Pour plus d’informations, consultez l’article [Élément ExeHost](service-fabric-service-model-schema-elements.md#ExeHostElementExeHostEntryPointTypeComplexTypeDefinedInSetupEntryPointelement).

### <a name="program-element"></a>Élément Program
Nom du fichier exécutable.  Par exemple, « MySetup.bat » ou « MyServiceHost.exe ». Pour plus d’informations, consultez l’article [Élément Program](service-fabric-service-model-schema-elements.md#ProgramElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="arguments-element"></a>Élément Arguments
 Pour plus d’informations, consultez l’article [Élément Arguments](service-fabric-service-model-schema-elements.md#ArgumentsElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="workingfolder-element"></a>Élément WorkingFolder
Répertoire de travail pour le processus dans le package de code sur le nœud de cluster où l’application est déployée. Vous pouvez spécifier trois valeurs : Work (valeur par défaut), CodePackage ou CodeBase. CodeBase indique que le répertoire de travail correspond au répertoire dans lequel le fichier EXE est défini dans le package de code. CodePackage définit le répertoire de travail pour en faire la racine du package de code, peu importe l’emplacement de définition du fichier EXE dans le répertoire de package de code. Work définit le répertoire de travail dans un dossier unique créé sur le nœud.  Ce dossier est le même pour l’intégralité de l’instance de l’application. Par défaut, le répertoire de travail de tous les processus de l’application est défini sur le dossier de travail de l’application. C’est là que les processus peuvent écrire les données. Il est déconseillé d’écrire les données dans le package de code ou la base de code, car ces dossiers peuvent être partagés entre différentes instances d’application et sont susceptibles d’être supprimés. Pour plus d’informations, consultez l’article [Élément WorkingFolder](service-fabric-service-model-schema-elements.md#WorkingFolderElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="consoleredirection-element"></a>Élément ConsoleRedirection

> [!WARNING]
> N’utilisez pas la redirection de console dans une application de production. Utilisez cette fonctionnalité uniquement pour le développement local et le débogage. La sortie de la console est redirigée du script de démarrage vers un fichier de sortie dans le dossier d’application « log », sur le nœud de cluster où l’application est déployée et exécutée. Pour plus d’informations, consultez l’article [Élément ConsoleRedirection](service-fabric-service-model-schema-elements.md#ConsoleRedirectionElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="entrypoint-element"></a>Élément EntryPoint
Le fichier exécutable spécifié par EntryPoint est généralement l’hôte de service à exécution longue. La présence d’un point d’entrée de configuration distinct évite d’avoir à exécuter l’hôte de service avec des privilèges élevés pendant de longues périodes de temps. Le fichier exécutable spécifié par EntryPoint est exécuté une fois que SetupEntryPoint se termine correctement. Le processus résultant fait l’objet d’une surveillance et est redémarré (à partir de SetupEntryPoint) en cas d’interruption ou de défaillance. Pour plus d’informations, consultez l’article [Élément EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="exehost-element"></a>Élément ExeHost
 Pour plus d’informations, consultez l’article [Élément ExeHost](service-fabric-service-model-schema-elements.md#ExeHostElementanonymouscomplexTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="configpackage-element"></a>Élément ConfigPackage
Déclare un dossier, nommé par l’attribut de nom et placé sous PackageRoot, contient un fichier Settings.xml. Ce fichier contient des sections de paramètres clé-valeur définis par l'utilisateur que le processus peut lire pendant l'exécution. Le processus en cours d’exécution n’est pas redémarré pendant la mise à niveau si seule la version de ConfigPackage a changé. Au lieu de cela, un rappel indique au processus que les paramètres de configuration ont été modifiés afin qu'ils puissent être rechargés dynamiquement. Pour plus d’informations, consultez l’article [Élément ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="resources-element"></a>Élément Resources
Décrit les ressources utilisées par ce service, qui peuvent être déclarées sans modifier le code compilé et changées quand le service est déployé. L’accès à ces ressources est contrôlé par le biais des sections Principals et Policies du manifeste d’application. Pour plus d’informations, consultez l’article [Élément Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Élément Endpoints
Définit des points de terminaison pour le service. Pour plus d’informations, consultez l’article [Élément Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Élément Endpoint
Le point de terminaison à remplacer, qui est déclaré dans le manifeste de service. Pour plus d’informations, consultez l’article [Élément Endpoint](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).



## <a name="votingdata-service-manifest-elements"></a>Éléments de manifeste de service VotingData
### <a name="servicemanifest-element"></a>Élément ServiceManifest
Décrit de façon déclarative le type et la version du service. Il répertorie les packages de code, de configuration et de données pouvant être mis à niveau indépendamment et qui constituent ensemble un package de service pour prendre en charge un ou plusieurs types de service. Les ressources, paramètres de diagnostics et métadonnées de service, telles que le type de service, les propriétés d’intégrité et les métriques d’équilibrage de charge, sont également spécifiés. Pour plus d’informations, consultez l’article [Élément ServiceManifest](service-fabric-service-model-schema-elements.md#ServiceManifestElementServiceManifestTypeComplexType).

### <a name="servicetypes-element"></a>Élément ServiceTypes
Définit les types de service pris en charge par un CodePackage dans ce manifeste. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les types de service sont déclarés au niveau du manifeste et non au niveau du package de code. Pour plus d’informations, consultez l’article [Élément ServiceTypes](service-fabric-service-model-schema-elements.md#ServiceTypesElementServiceAndServiceGroupTypesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="statefulservicetype-element"></a>Élément StatefulServiceType
Décrit un type de services avec état. Pour plus d’informations, consultez l’article [Élément StatefulServiceType](service-fabric-service-model-schema-elements.md#StatefulServiceTypeElementStatefulServiceTypeTypeComplexTypeDefinedInServiceAndServiceGroupTypesTypecomplexTypeDefinedInServiceTypesTypecomplexType).

### <a name="codepackage-element"></a>Élément CodePackage
Décrit un package de code qui prend en charge un type de service défini. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les processus qui en résultent sont censés inscrire les types de service pris en charge au moment de l'exécution. S’il existe plusieurs packages de code, ils sont tous activés chaque fois que le système recherche l’un des types de service déclarés. Pour plus d’informations, consultez l’article [Élément CodePackage](service-fabric-service-model-schema-elements.md#CodePackageElementCodePackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedCodePackageelement).

### <a name="entrypoint-element"></a>Élément EntryPoint
Le fichier exécutable spécifié par EntryPoint est généralement l’hôte de service à exécution longue. La présence d’un point d’entrée de configuration distinct évite d’avoir à exécuter l’hôte de service avec des privilèges élevés pendant de longues périodes de temps. Le fichier exécutable spécifié par EntryPoint est exécuté une fois que SetupEntryPoint se termine correctement. Le processus résultant fait l’objet d’une surveillance et est redémarré (à partir de SetupEntryPoint) en cas d’interruption ou de défaillance. Pour plus d’informations, consultez l’article [Élément EntryPoint](service-fabric-service-model-schema-elements.md#EntryPointElementEntryPointDescriptionTypeComplexTypeDefinedInCodePackageTypecomplexType).

### <a name="exehost-element"></a>Élément ExeHost
 Pour plus d’informations, consultez l’article [Élément ExeHost](service-fabric-service-model-schema-elements.md#ExeHostElementanonymouscomplexTypeComplexTypeDefinedInEntryPointDescriptionTypecomplexType).

### <a name="program-element"></a>Élément Program
Nom du fichier exécutable.  Par exemple, « MySetup.bat » ou « MyServiceHost.exe ». Pour plus d’informations, consultez l’article [Élément Program](service-fabric-service-model-schema-elements.md#ProgramElementxs:stringComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="workingfolder-element"></a>Élément WorkingFolder
Répertoire de travail pour le processus dans le package de code sur le nœud de cluster où l’application est déployée. Vous pouvez spécifier trois valeurs : Work (valeur par défaut), CodePackage ou CodeBase. CodeBase indique que le répertoire de travail correspond au répertoire dans lequel le fichier EXE est défini dans le package de code. CodePackage définit le répertoire de travail pour en faire la racine du package de code, peu importe l’emplacement de définition du fichier EXE dans le répertoire de package de code. Work définit le répertoire de travail dans un dossier unique créé sur le nœud.  Ce dossier est le même pour l’intégralité de l’instance de l’application. Par défaut, le répertoire de travail de tous les processus de l’application est défini sur le dossier de travail de l’application. C’est là que les processus peuvent écrire les données. Il est déconseillé d’écrire les données dans le package de code ou la base de code, car ces dossiers peuvent être partagés entre différentes instances d’application et sont susceptibles d’être supprimés. Pour plus d’informations, consultez l’article [Élément WorkingFolder](service-fabric-service-model-schema-elements.md#WorkingFolderElementanonymouscomplexTypeComplexTypeDefinedInExeHostEntryPointTypecomplexType).

### <a name="configpackage-element"></a>Élément ConfigPackage
Déclare un dossier, nommé par l’attribut de nom et placé sous PackageRoot, contient un fichier Settings.xml. Ce fichier contient des sections de paramètres clé-valeur définis par l'utilisateur que le processus peut lire pendant l'exécution. Le processus en cours d’exécution n’est pas redémarré pendant la mise à niveau si seule la version de ConfigPackage a changé. Au lieu de cela, un rappel indique au processus que les paramètres de configuration ont été modifiés afin qu'ils puissent être rechargés dynamiquement. Pour plus d’informations, consultez l’article [Élément ConfigPackage](service-fabric-service-model-schema-elements.md#ConfigPackageElementConfigPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedConfigPackageelement).

### <a name="datapackage-element"></a>Élément DataPackage
Déclare un dossier, nommé par l’attribut de nom et placé sous PackageRoot, qui contient des fichiers de données statiques destinées à être consommées par le processus pendant l’exécution. Service Fabric recycle tous les fichiers EXE et DLLHOST spécifiés dans les packages hôtes et de prise en charge quand l’un des packages de données répertoriés dans le manifeste de service est mis à niveau. Pour plus d’informations, consultez l’article [Élément DataPackage](service-fabric-service-model-schema-elements.md#DataPackageElementDataPackageTypeComplexTypeDefinedInServiceManifestTypecomplexTypeDefinedInDigestedDataPackageelement).

### <a name="resources-element"></a>Élément Resources
Décrit les ressources utilisées par ce service, qui peuvent être déclarées sans modifier le code compilé et changées quand le service est déployé. L’accès à ces ressources est contrôlé par le biais des sections Principals et Policies du manifeste d’application. Pour plus d’informations, consultez l’article [Élément Resources](service-fabric-service-model-schema-elements.md#ResourcesElementResourcesTypeComplexTypeDefinedInServiceManifestTypecomplexType).

### <a name="endpoints-element"></a>Élément Endpoints
Définit des points de terminaison pour le service. Pour plus d’informations, consultez l’article [Élément Endpoints](service-fabric-service-model-schema-elements.md#EndpointsElementanonymouscomplexTypeComplexTypeDefinedInResourcesTypecomplexType).

### <a name="endpoint-element"></a>Élément Endpoint
Le point de terminaison à remplacer, qui est déclaré dans le manifeste de service. Pour plus d’informations, consultez l’article [Élément Endpoint](service-fabric-service-model-schema-elements.md#EndpointElementEndpointOverrideTypeComplexTypeDefinedInEndpointselement).

