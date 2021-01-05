---
title: Spécification de points de terminaison du service Service Fabric
description: Comment décrire les ressources du point de terminaison dans un manifeste de service, y compris comment configurer des points de terminaison HTTPS
ms.topic: conceptual
ms.date: 09/16/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: 0ed5a4aa8993f52d42b97288cd143e6114ff36ff
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033304"
---
# <a name="specify-resources-in-a-service-manifest"></a>Spécifier des ressources dans un manifeste de service
## <a name="overview"></a>Vue d’ensemble
Les applications et services Service Fabric sont définis et créés dans différentes versions à l’aide à l’aide de fichiers manifeste. Pour obtenir une vue d’ensemble plus général de ServiceManifest.xml et d’ApplicationManifest.xml, consultez [Manifestes des services et applications Service Fabric](service-fabric-application-and-service-manifests.md).

Le manifeste de service met les ressources à la disposition du service à déclarer, ou à modifier, sans changer le code compilé. Service Fabric prend en charge la configuration des ressources des points de terminaison du service. L’accès aux ressources spécifiées dans le manifeste de service peut être contrôlé par le biais de la valeur SecurityGroup dans le manifeste de l’application. La déclaration des ressources permet de les modifier au moment du déploiement. Ainsi, le service n’a pas besoin d’introduire un nouveau mécanisme de configuration. La définition de schéma pour le fichier ServiceManifest.xml est installée avec le SDK et les outils Service Fabric sous *C:\Program Files\Microsoft SDKs\Service Fabric\schemas\ServiceFabricServiceModel.xsd*, et est présentée dans la [documentation sur le schéma ServiceFabricServiceModel.xsd](service-fabric-service-model-schema.md).

## <a name="endpoints"></a>Points de terminaison
Lorsqu’une ressource de point de terminaison est définie dans le manifeste de service, Service Fabric alloue les ports de la plage de ports d’application réservés lorsqu’un port n’est pas spécifié de manière explicite. Examinez, par exemple, le point de terminaison *ServiceEndpoint1* spécifié dans l’extrait de code du manifeste fourni après ce paragraphe. En outre, les services peuvent également demander un port spécifique d’une ressource. Les réplicas de service exécutés sur des nœuds de cluster différents peuvent être alloués à des numéros de ports différents, tandis que les réplicas d’un service exécutés sur le même nœud partagent le port. Les réplicas de service peuvent alors utiliser ces ports pour la réplication et pour écouter les demandes du client.

Lors de l’activation d’un service qui spécifie un point de terminaison HTTPS, Service Fabric définit l’entrée de contrôle d’accès pour le port, lie le certificat de serveur spécifié au port et accorde également à l’identité que le service exécute sous la forme d’autorisations sur la clé privée du certificat. Le flux d’activation est appelé chaque fois que Service Fabric démarre, ou lorsque la déclaration de certificat de l’application est modifiée via une mise à niveau. Les changements/renouvellements sont également supervisés dans le certificat de point de terminaison, et les autorisations sont réappliquées régulièrement si nécessaire.

À l’arrêt du service, Service Fabric nettoie l’entrée de contrôle d’accès du point de terminaison et supprime la liaison de certificat. Toutefois, les autorisations appliquées à la clé privée du certificat ne sont pas nettoyées.

> [!WARNING] 
> De par leur conception, les ports statiques ne doivent pas chevaucher la plage de ports d’application spécifiée dans ClusterManifest. Si vous spécifiez un port statique, affectez-le hors de la plage de ports d’application ; sinon, des conflits de port se produiront. Avec la version 6.5CU2, nous émettons un **avertissement d’intégrité** quand nous détectons un tel conflit, mais nous laissons le déploiement continuer de façon synchronisée avec le comportement de la version 6.5 expédiée. Toutefois, nous pouvons empêcher le déploiement de l’application à partir des versions majeures suivantes.
>
> Avec la version 7.0, nous émettrons un **Avertissement d’intégrité** en cas de détection d’une utilisation de la plage de ports d’application supérieure à HostingConfig::ApplicationPortExhaustThresholdPercentage (par défaut, 80 %).
>

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
    <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
    <Endpoint Name="ServiceEndpoint3" Protocol="https"/>
  </Endpoints>
</Resources>
```

S’il existe plusieurs packages de code dans un package de service unique, le package de code doit également être référencé dans la section **Points de terminaison**.  Par exemple, si **ServiceEndpoint2a** et **ServiceEndpoint2b** sont des points de terminaison provenant du même package de service faisant référence à des packages de code différents, le package de code correspondant à chaque point de terminaison est clarifié comme suit :

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="ServiceEndpoint2a" Protocol="http" Port="802" CodePackageRef="Code1"/>
    <Endpoint Name="ServiceEndpoint2b" Protocol="http" Port="801" CodePackageRef="Code2"/>
  </Endpoints>
</Resources>
```

Reportez-vous à [Configuration de services Reliable Services avec état](service-fabric-reliable-services-configuration.md) pour en savoir plus sur le référencement des points de terminaison à partir du fichier des paramètres de package de configuration (settings.xml).

## <a name="example-specifying-an-http-endpoint-for-your-service"></a>Exemple : spécification d’un point de terminaison HTTP pour votre service
Le manifeste de service suivant définit une seule ressource de point de terminaison TCP et deux ressources de point de terminaison HTTP dans l’élément &lt;Ressources&gt;.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="Stateful1Pkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         This name must match the string used in the RegisterServiceType call in Program.cs. -->
    <StatefulServiceType ServiceTypeName="Stateful1Type" HasPersistedState="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>Stateful1.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an
       independently updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port number on which to
           listen. Note that if your service is partitioned, this port is shared with
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
      <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
      <Endpoint Name="ServiceEndpoint3" Protocol="https"/>
      <Endpoint Name="ServiceEndpoint4" Protocol="https" Port="14023"/>

      <!-- This endpoint is used by the replicator for replicating the state of your service.
           This endpoint is configured through the ReplicatorSettings config section in the Settings.xml
           file under the ConfigPackage. -->
      <Endpoint Name="ReplicatorEndpoint" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## <a name="example-specifying-an-https-endpoint-for-your-service"></a>Exemple : spécification d’un point de terminaison HTTPS pour votre service
Le protocole HTTPS assure l'authentification du serveur et est également utilisé pour le chiffrement des communications client-serveur. Pour activer HTTPS sur votre service Service Fabric, spécifiez le protocole dans la section *Ressources -> Points de terminaison -> Point de terminaison* du manifeste de service, comme indiqué plus haut pour le point de terminaison *ServiceEndpoint3*.

> [!NOTE]
> Il est impossible de modifier le protocole d’un service lors de la mise à niveau de l’application. En effet, il s’agit d’une modification avec rupture.
> 

> [!WARNING] 
> Lorsque vous utilisez HTTPS, n’utilisez pas le même port et le même certificat pour les différentes instances de service (indépendantes de l’application) déployées sur le même nœud. La mise à niveau de deux services différents utilisant le même port dans différentes instances d’application aboutit à un échec. Pour plus d’informations, consultez [Mise à niveau de plusieurs applications avec des points de terminaison HTTPS](service-fabric-application-upgrade.md#upgrading-multiple-applications-with-https-endpoints).
>

Voici un exemple de fichier ApplicationManifest illustrant la configuration requise pour un point de terminaison HTTPS. Le certificat de serveur/point de terminaison peut être déclaré par empreinte numérique ou par nom commun de sujet, et une valeur doit être fournie. EndpointRef est une référence à EndpointResource dans ServiceManifest, dont le protocole doit être défini sur 'https'. Vous pouvez ajouter plusieurs EndpointCertificate.  

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="Application1Type"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
    <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
       should match the Name and Version attributes of the ServiceManifest element defined in the
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <EndpointBindingPolicy CertificateRef="SslCertByTP" EndpointRef="ServiceEndpoint3"/>
      <EndpointBindingPolicy CertificateRef="SslCertByCN" EndpointRef="ServiceEndpoint4"/>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types when an instance of this
         application type is created. You can also create one or more instances of service type by using the
         Service Fabric PowerShell module.

         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="Stateful1">
      <StatefulService ServiceTypeName="Stateful1Type" TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]" MinReplicaSetSize="[Stateful1_ ]">
        <UniformInt64Partition PartitionCount="[Stateful1_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
      </StatefulService>
    </Service>
  </DefaultServices>
  <Certificates>
    <EndpointCertificate Name="SslCertByTP" X509FindValue="FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0" X509StoreName="MY" />  
    <EndpointCertificate Name="SslCertByCN" X509FindType="FindBySubjectName" X509FindValue="ServiceFabric-EndpointCertificateBinding-Test" X509StoreName="MY" />  
  </Certificates>
</ApplicationManifest>
```

Pour les clusters Linux, le **MY** stocke par défaut dans le dossier **/var/lib/sfcerts**.

Pour obtenir un exemple d’application complète qui utilise un point de terminaison HTTPS, consultez [Ajouter un point de terminaison HTTPS à un service front-end d’API web ASP.NET Core à l’aide de Kestrel](./service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest).

## <a name="port-acling-for-http-endpoints"></a>Port liste ACL pour les points de terminaison HTTP
Service Fabric effectue automatiquement des points de terminaison HTTP(S) ACL spécifiés par défaut. Il n’effectue **pas** de mise en liste ACL automatique si un point de terminaison n’a aucun élément [SecurityAccessPolicy](service-fabric-assign-policy-to-endpoint.md) qui lui est associé et que Service Fabric est configuré pour s’exécuter à l’aide d’un compte disposant de privilèges d’administrateur.

## <a name="overriding-endpoints-in-servicemanifestxml"></a>Écraser des points de terminaison dans ServiceManifest.xml

Dans ApplicationManifest, ajoutez une section ResourceOverrides qui sera une section sœur de la section ConfigOverrides. Vous pouvez y spécifier le remplacement de la section Endpoints dans la section des ressources spécifiée dans le manifeste de service. Le remplacement des points de terminaison est pris en charge dans le runtime 5.7.217/SDK 2.7.217 et versions supérieures.

Pour pouvoir remplacer EndPoint dans ServiceManifest à l’aide d’ApplicationParameters, changez ApplicationManifest comme suit :

Dans la section ServiceManifestImport, ajoutez une nouvelle section « ResourceOverrides ».

```xml
<ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <ResourceOverrides>
      <Endpoints>
        <Endpoint Name="ServiceEndpoint" Port="[Port]" Protocol="[Protocol]" Type="[Type]" />
        <Endpoint Name="ServiceEndpoint1" Port="[Port1]" Protocol="[Protocol1] "/>
      </Endpoints>
    </ResourceOverrides>
        <Policies>
           <EndpointBindingPolicy CertificateRef="SslCertByTP" EndpointRef="ServiceEndpoint"/>
        </Policies>
  </ServiceManifestImport>
```

Dans Parameters, ajoutez le code ci-dessous :

```xml
  <Parameters>
    <Parameter Name="Port" DefaultValue="" />
    <Parameter Name="Protocol" DefaultValue="" />
    <Parameter Name="Type" DefaultValue="" />
    <Parameter Name="Port1" DefaultValue="" />
    <Parameter Name="Protocol1" DefaultValue="" />
  </Parameters>
```

Lors du déploiement de l’application, vous pouvez passer ces valeurs comme ApplicationParameters.  Par exemple :

```powershell
PS C:\> New-ServiceFabricApplication -ApplicationName fabric:/myapp -ApplicationTypeName "AppType" -ApplicationTypeVersion "1.0.0" -ApplicationParameter @{Port='1001'; Protocol='https'; Type='Input'; Port1='2001'; Protocol='http'}
```

Remarque : Si les valeurs fournies pour un élément ApplicationParameter sont vides, la valeur par défaut fournie dans ServiceManifest pour le EndPointName correspondant est réactivée.

Par exemple :

Si, dans ServiceManifest, vous avez spécifié :

```xml
  <Resources>
    <Endpoints>
      <Endpoint Name="ServiceEndpoint1" Protocol="tcp"/>
    </Endpoints>
  </Resources>
```

Supposons que la valeur Port1 et Protocol1 des paramètres d’application est Null ou vide. Le port est déterminé par ServiceFabric et Protocol (protocole) a la valeur TCP.

Supposons que vous spécifiez une valeur incorrecte. Par exemple, pour Port, vous avez spécifié la valeur de chaîne « Foo » au lieu d’un entier.  La commande New-ServiceFabricApplication échoue avec l’erreur suivante : `The override parameter with name 'ServiceEndpoint1' attribute 'Port1' in section 'ResourceOverrides' is invalid. The value specified is 'Foo' and required is 'int'.`

## <a name="next-steps"></a>Étapes suivantes

Cet article a expliqué comment définir des points de terminaison dans le manifeste de service de Service Fabric. Pour obtenir des exemples plus détaillés, consultez :

> [!div class="nextstepaction"]
> [Exemples de manifestes d’application et de service](service-fabric-manifest-examples.md)

Pour obtenir une procédure pas à pas de l’empaquetage et du déploiement d’une application existante sur un cluster Service Fabric, consultez :

> [!div class="nextstepaction"]
> [Empaqueter et déployer un fichier exécutable existant sur Service Fabric](service-fabric-deploy-existing-app.md)