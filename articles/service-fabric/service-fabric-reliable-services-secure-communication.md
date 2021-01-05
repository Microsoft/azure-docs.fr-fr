---
title: Sécuriser des communications de service à distance avec C#
description: Découvrez comment sécuriser des communications de service à distance pour des services fiables C# s’exécutant dans un cluster Azure Service Fabric.
author: suchiagicha
ms.topic: conceptual
ms.date: 04/20/2017
ms.author: pepogors
ms.openlocfilehash: ba68df53f1f21b9ff360772fe1a60c93c8df74d3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86252213"
---
# <a name="secure-service-remoting-communications-in-a-c-service"></a>Sécuriser les communications à distance des services dans un service C#
> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-services-secure-communication.md)
> * [Java sur Linux](service-fabric-reliable-services-secure-communication-java.md)
>
>

La sécurité est un des aspects les plus importants de la communication. L’infrastructure d’application Reliable Services fournit quelques piles et outils de communication prédéfinis afin d’améliorer la sécurité. Cet article explique comment améliorer la sécurité quand vous utilisez la communication à distance dans un service C#. Il s’appuie sur un [exemple](service-fabric-reliable-services-communication-remoting.md) existant qui décrit comment configurer la communication à distance pour les services fiables écrits en C#. 

Pour sécuriser un service lors de l’utilisation de la communication à distance avec des services C#, effectuez les opérations suivantes :

1. Créez une interface, `IHelloWorldStateful`, qui définit les méthodes disponibles pour l'appel de procédure distante sur votre service. Votre service utilisera `FabricTransportServiceRemotingListener`, qui est déclaré dans l’espace de noms `Microsoft.ServiceFabric.Services.Remoting.FabricTransport.Runtime`. Il s'agit d'une implémentation `ICommunicationListener` qui fournit des fonctionnalités de communication à distance.

    ```csharp
    public interface IHelloWorldStateful : IService
    {
        Task<string> GetHelloWorld();
    }

    internal class HelloWorldStateful : StatefulService, IHelloWorldStateful
    {
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]{
                    new ServiceReplicaListener(
                        (context) => new FabricTransportServiceRemotingListener(context,this))};
        }

        public Task<string> GetHelloWorld()
        {
            return Task.FromResult("Hello World!");
        }
    }
    ```
2. Ajoutez des paramètres de l’écouteur et des informations d’identification de sécurité.

    Vérifiez que le certificat que vous souhaitez utiliser pour sécuriser les communications de votre service est installé sur tous les nœuds du cluster. 
    
    > [!NOTE]
    > Des nœuds Linux, le certificat doit être présent sous forme de fichiers au format PEM dans le répertoire */var/lib/sfcerts*. Pour en savoir plus, consultez [Emplacement et format des certificats X.509 sur les nœuds Linux](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes). 

    Vous pouvez fournir les paramètres de l’écouteur et les informations d’identification de sécurité de deux manières :

   1. Directement dans le code de service :

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           FabricTransportRemotingListenerSettings  listenerSettings = new FabricTransportRemotingListenerSettings
           {
               MaxMessageSize = 10000000,
               SecurityCredentials = GetSecurityCredentials()
           };
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(context,this,listenerSettings))
           };
       }

       private static SecurityCredentials GetSecurityCredentials()
       {
           // Provide certificate details.
           var x509Credentials = new X509Credentials
           {
               FindType = X509FindType.FindByThumbprint,
               FindValue = "4FEF3950642138446CC364A396E1E881DB76B48C",
               StoreLocation = StoreLocation.LocalMachine,
               StoreName = "My",
               ProtectionLevel = ProtectionLevel.EncryptAndSign
           };
           x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
           x509Credentials.RemoteCertThumbprints.Add("9FEF3950642138446CC364A396E1E881DB76B483");
           return x509Credentials;
       }
       ```
   2. À l'aide d’un [package de configuration](service-fabric-application-and-service-manifests.md):

       Ajoutez une section `TransportSettings` nommée dans le fichier settings.xml.

       ```xml
       <Section Name="HelloWorldStatefulTransportSettings">
           <Parameter Name="MaxMessageSize" Value="10000000" />
           <Parameter Name="SecurityCredentialsType" Value="X509" />
           <Parameter Name="CertificateFindType" Value="FindByThumbprint" />
           <Parameter Name="CertificateFindValue" Value="4FEF3950642138446CC364A396E1E881DB76B48C" />
           <Parameter Name="CertificateRemoteThumbprints" Value="9FEF3950642138446CC364A396E1E881DB76B483" />
           <Parameter Name="CertificateStoreLocation" Value="LocalMachine" />
           <Parameter Name="CertificateStoreName" Value="My" />
           <Parameter Name="CertificateProtectionLevel" Value="EncryptAndSign" />
           <Parameter Name="CertificateRemoteCommonNames" Value="ServiceFabric-Test-Cert" />
       </Section>
       ```

       Dans ce cas, la méthode `CreateServiceReplicaListeners` se présente comme suit :

       ```csharp
       protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
       {
           return new[]
           {
               new ServiceReplicaListener(
                   (context) => new FabricTransportServiceRemotingListener(
                       context,this,FabricTransportRemotingListenerSettings .LoadFrom("HelloWorldStatefulTransportSettings")))
           };
       }
       ```

        Si vous ajoutez une section `TransportSettings` dans le fichier settings.xml, `FabricTransportRemotingListenerSettings` charge par défaut tous les paramètres de cette section.

        ```xml
        <!--"TransportSettings" section .-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        Dans ce cas, la méthode `CreateServiceReplicaListeners` se présente comme suit :

        ```csharp
        protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
        {
            return new[]
            {
                return new[]{
                        new ServiceReplicaListener(
                            (context) => new FabricTransportServiceRemotingListener(context,this))};
            };
        }
        ```
3. Lorsque vous appelez des méthodes sur un service sécurisé à l’aide de la pile de communication à distance, au lieu d’utiliser la classe `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxy` pour créer un proxy de service, utilisez `Microsoft.ServiceFabric.Services.Remoting.Client.ServiceProxyFactory`. Transmettez `FabricTransportRemotingSettings`, qui contient `SecurityCredentials`.

    ```csharp

    var x509Credentials = new X509Credentials
    {
        FindType = X509FindType.FindByThumbprint,
        FindValue = "9FEF3950642138446CC364A396E1E881DB76B483",
        StoreLocation = StoreLocation.LocalMachine,
        StoreName = "My",
        ProtectionLevel = ProtectionLevel.EncryptAndSign
    };
    x509Credentials.RemoteCommonNames.Add("ServiceFabric-Test-Cert");
    x509Credentials.RemoteCertThumbprints.Add("4FEF3950642138446CC364A396E1E881DB76B48C");

    FabricTransportRemotingSettings transportSettings = new FabricTransportRemotingSettings
    {
        SecurityCredentials = x509Credentials,
    };

    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(transportSettings));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    Si le code client s’exécute dans le cadre d’un service, vous pouvez charger `FabricTransportRemotingSettings` à partir du fichier settings.xml. Créez une section HelloWorldClientTransportSettings similaire au code de service, comme illustré précédemment. Apportez les modifications suivantes au code du client :

    ```csharp
    ServiceProxyFactory serviceProxyFactory = new ServiceProxyFactory(
        (c) => new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.LoadFrom("HelloWorldClientTransportSettings")));

    IHelloWorldStateful client = serviceProxyFactory.CreateServiceProxy<IHelloWorldStateful>(
        new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```

    Si le client n’est pas exécuté dans le cadre d’un service, vous pouvez créer un fichier client_name.settings.xml dans le même emplacement que le fichier client_name.exe. Créez ensuite une section TransportSettings dans ce fichier.

    Comme pour le service, si vous ajoutez une section `TransportSettings` dans le fichier client settings.xml/client_name.settings.xml, `FabricTransportRemotingSettings` charge par défaut tous les paramètres de cette section.

    Dans ce cas, le code précédent est encore davantage simplifié :  

    ```csharp

    IHelloWorldStateful client = ServiceProxy.Create<IHelloWorldStateful>(
                 new Uri("fabric:/MyApplication/MyHelloWorldService"));

    string message = await client.GetHelloWorld();

    ```


En guise de prochaine étape, lisez [API web avec OWIN dans Reliable Services](./service-fabric-reliable-services-communication-aspnetcore.md).
