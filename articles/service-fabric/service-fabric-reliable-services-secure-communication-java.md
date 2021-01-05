---
title: Sécuriser des communications de service à distance avec Java
description: Découvrez comment sécuriser les communications à distance pour des services fiables Java s’exécutant dans un cluster Azure Service Fabric.
author: PavanKunapareddyMSFT
ms.topic: conceptual
ms.date: 06/30/2017
ms.custom: devx-track-java
ms.author: pakunapa
ms.openlocfilehash: 01a64fbcfef9f56abb0e1aa6cf7f5d821dd3763b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87325710"
---
# <a name="secure-service-remoting-communications-in-a-java-service"></a>Sécuriser les communications à distance des services dans un service Java
> [!div class="op_single_selector"]
> * [C# sur Windows](service-fabric-reliable-services-secure-communication.md)
> * [Java sur Linux](service-fabric-reliable-services-secure-communication-java.md)
>
>

La sécurité est un des aspects les plus importants de la communication. L’infrastructure d’application Reliable Services fournit quelques piles et outils de communication prédéfinis afin d’améliorer la sécurité. Cet article explique comment améliorer la sécurité quand vous utilisez la communication à distance dans un service Java. Il s’appuie sur un [exemple](service-fabric-reliable-services-communication-remoting-java.md) existant qui décrit comment configurer la communication à distance pour les services fiables écrits en Java. 

Pour sécuriser un service lors de l’utilisation de la communication à distance avec des services Java, effectuez les opérations suivantes :

1. Créez une interface, `HelloWorldStateless`, qui définit les méthodes disponibles pour l'appel de procédure distante sur votre service. Votre service utilisera `FabricTransportServiceRemotingListener`, qui est déclaré dans le package `microsoft.serviceFabric.services.remoting.fabricTransport.runtime`. Il s'agit d'une implémentation `CommunicationListener` qui fournit des fonctionnalités de communication à distance.

    ```java
    public interface HelloWorldStateless extends Service {
        CompletableFuture<String> getHelloWorld();
    }

    class HelloWorldStatelessImpl extends StatelessService implements HelloWorldStateless {
        @Override
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
        return listeners;
        }

        public CompletableFuture<String> getHelloWorld() {
            return CompletableFuture.completedFuture("Hello World!");
        }
    }
    ```
2. Ajoutez des paramètres de l’écouteur et des informations d’identification de sécurité.

    Vérifiez que le certificat que vous souhaitez utiliser pour sécuriser les communications de votre service est installé sur tous les nœuds du cluster. Pour les services exécutés sous Linux, le certificat doit être disponible comme un fichier au format PEM ; un fichier `.pem` qui contient le certificat et une clé privée ou un fichier `.crt` qui contient le certificat et un fichier `.key` qui contient la clé privée. Pour en savoir plus, consultez [Emplacement et format des certificats X.509 sur les nœuds Linux](./service-fabric-configure-certificates-linux.md#location-and-format-of-x509-certificates-on-linux-nodes).
    
    Vous pouvez fournir les paramètres de l’écouteur et les informations d’identification de sécurité de deux manières :

   1. À l'aide d’un [package de configuration](service-fabric-application-and-service-manifests.md):

       Ajoutez une section `TransportSettings` nommée dans le fichier settings.xml.

       ```xml
       <!--Section name should always end with "TransportSettings".-->
       <!--Here we are using a prefix "HelloWorldStateless".-->
        <Section Name="HelloWorldStatelessTransportSettings">
            <Parameter Name="MaxMessageSize" Value="10000000" />
            <Parameter Name="SecurityCredentialsType" Value="X509_2" />
            <Parameter Name="CertificatePath" Value="/path/to/cert/BD1C71E248B8C6834C151174DECDBDC02DE1D954.crt" />
            <Parameter Name="CertificateProtectionLevel" Value="EncryptandSign" />
            <Parameter Name="CertificateRemoteThumbprints" Value="BD1C71E248B8C6834C151174DECDBDC02DE1D954" />
        </Section>

       ```

       Dans ce cas, la méthode `createServiceInstanceListeners` se présente comme suit :

       ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this, FabricTransportRemotingListenerSettings.loadFrom(HelloWorldStatelessTransportSettings));
            }));
            return listeners;
        }
       ```

        Si vous ajoutez une section `TransportSettings` dans le fichier settings.xml sans aucun préfixe, `FabricTransportListenerSettings` charge par défaut tous les paramètres de cette section.

        ```xml
        <!--"TransportSettings" section without any prefix.-->
        <Section Name="TransportSettings">
            ...
        </Section>
        ```
        Dans ce cas, la méthode `CreateServiceInstanceListeners` se présente comme suit :

        ```java
        protected List<ServiceInstanceListener> createServiceInstanceListeners() {
            ArrayList<ServiceInstanceListener> listeners = new ArrayList<>();
            listeners.add(new ServiceInstanceListener((context) -> {
                return new FabricTransportServiceRemotingListener(context,this);
            }));
            return listeners;
        }
       ```
3. Lorsque vous appelez des méthodes sur un service sécurisé à l’aide de la pile de communication à distance, au lieu d’utiliser la classe `microsoft.serviceFabric.services.remoting.client.ServiceProxyBase` pour créer un proxy de service, utilisez `microsoft.serviceFabric.services.remoting.client.FabricServiceProxyFactory`.

    Si le code client s’exécute dans le cadre d’un service, vous pouvez charger `FabricTransportSettings` à partir du fichier settings.xml. Créez une section TransportSettings similaire au code de service, comme illustré précédemment. Apportez les modifications suivantes au code du client :

    ```java

    FabricServiceProxyFactory serviceProxyFactory = new FabricServiceProxyFactory(c -> {
            return new FabricTransportServiceRemotingClientFactory(FabricTransportRemotingSettings.loadFrom("TransportPrefixTransportSettings"), null, null, null, null);
        }, null)

    HelloWorldStateless client = serviceProxyFactory.createServiceProxy(HelloWorldStateless.class,
        new URI("fabric:/MyApplication/MyHelloWorldService"));

    CompletableFuture<String> message = client.getHelloWorld();

    ```
