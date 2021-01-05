---
title: Communication pour les rôles dans les services cloud | Microsoft Docs
description: Dans Cloud Services, des points de terminaison (http, https, tcp, udp) peuvent être associés aux instances de rôle pour faciliter la communication avec l’extérieur ou entre instances de rôle.
services: cloud-services
documentationcenter: ''
author: tgore03
manager: carmonm
ms.service: cloud-services
ms.topic: article
ms.date: 12/14/2016
ms.author: tagore
ms.openlocfilehash: 094e08becf4f3a60c98d89bfae7e7c3a69b677f8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75386338"
---
# <a name="enable-communication-for-role-instances-in-azure"></a>Activer la communication pour les instances de rôle dans Azure
Les rôles de service cloud communiquent via des connexions internes et externes. Les connexions externes sont appelées **points de terminaison d’entrée** tandis que les connexions internes sont appelées **points de terminaison internes**. Cette rubrique explique comment modifier la [définition de service](cloud-services-model-and-package.md#csdef) pour créer des points de terminaison.

## <a name="input-endpoint"></a>Point de terminaison d’entrée
Le point de terminaison d’entrée est utilisé lorsque vous souhaitez exposer un port à l’extérieur. Vous spécifiez le type de protocole et le port du point de terminaison qui s’applique ensuite aux ports internes et externes du point de terminaison. Si vous le souhaitez, vous pouvez spécifier un autre port interne pour le point de terminaison avec l’attribut [localPort](/previous-versions/azure/reference/gg557552(v=azure.100)#inputendpoint) .

Le point de terminaison d’entrée peut utiliser les protocoles suivants : **http, https, tcp, udp**.

Pour créer un point de terminaison d’entrée, ajoutez l’élément enfant **InputEndpoint** à l’élément **Endpoints** d’un rôle web ou de travail.

```xml
<Endpoints>
  <InputEndpoint name="StandardWeb" protocol="http" port="80" localPort="80" />
</Endpoints> 
```

## <a name="instance-input-endpoint"></a>Point de terminaison d’entrée d’instance
Les points de terminaison d’entrée d’instance sont similaires aux points de terminaison d’entrées. Cependant, ils vous permettent de mapper des ports publics spécifiques pour chaque instance de rôle individuelle en utilisant le réacheminement de port sur l’équilibreur de charge. Vous pouvez spécifier un seul port public ou une plage de ports.

Le point de terminaison d’entrée d’instance peut utiliser uniquement le protocole **tcp** ou **udp**.

Pour créer un point de terminaison d’entrée d’instance, ajoutez l’élément enfant **InstanceInputEndpoint** à l’élément **Endpoints** d’un rôle web ou de travail.

```xml
<Endpoints>
  <InstanceInputEndpoint name="Endpoint2" protocol="tcp" localPort="10100">
    <AllocatePublicPortFrom>
      <FixedPortRange max="10109" min="10105" />
    </AllocatePublicPortFrom>
  </InstanceInputEndpoint>
</Endpoints>
```

## <a name="internal-endpoint"></a>Point de terminaison interne
Les points de terminaison internes sont disponibles pour la communication d’instance à instance. Le port est facultatif et, en cas d’omission, un port dynamique est affecté au point de terminaison. Une plage de ports peut être utilisée. Le nombre de points de terminaison internes est limité à cinq par rôle.

Le point de terminaison interne peut utiliser les protocoles suivants : **http, tcp, udp, any**.

Pour créer un point de terminaison d’entrée interne, ajoutez l’élément enfant **InternalEndpoint** à l’élément **Endpoints** d’un rôle web ou de travail.

```xml
<Endpoints>
  <InternalEndpoint name="Endpoint3" protocol="any" port="8999" />
</Endpoints> 
```

Vous pouvez également utiliser une plage de ports.

```xml
<Endpoints>
  <InternalEndpoint name="Endpoint3" protocol="any">
    <FixedPortRange max="8999" min="8995" />
  </InternalEndpoint>
</Endpoints>
```


## <a name="worker-roles-vs-web-roles"></a>Rôles de travail et Rôles web
Les points de terminaison présentent une légère différence lorsque vous travaillez avec des rôles de travail et des rôles Web. Avec le rôle Web, au moins un point de terminaison d’entrée doit utiliser le protocole **HTTP** .

```xml
<Endpoints>
  <InputEndpoint name="StandardWeb" protocol="http" port="80" localPort="80" />
  <!-- more endpoints may be declared after the first InputEndPoint -->
</Endpoints>
```

## <a name="using-the-net-sdk-to-access-an-endpoint"></a>Utilisation du Kit de développement logiciel (SDK) .NET pour accéder à un point de terminaison
La bibliothèque managée Azure fournit des méthodes permettant aux instances de rôle de communiquer au moment de l’exécution. À partir du code s’exécutant dans une instance de rôle, vous pouvez récupérer des informations sur l’existence d’autres instances de rôle et leurs points de terminaison, ainsi que des informations sur l’instance de rôle actuelle.

> [!NOTE]
> Vous pouvez uniquement récupérer des informations sur les instances de rôle s’exécutant dans votre service cloud et qui définissent au moins un point de terminaison interne. Vous ne pouvez pas obtenir de données sur les instances de rôle s’exécutant dans un autre service.
> 
> 

Vous pouvez utiliser la propriété [Instances](/previous-versions/azure/reference/ee741904(v=azure.100)) pour récupérer les instances d’un rôle. Utilisez d’abord la propriété [CurrentRoleInstance](/previous-versions/azure/reference/ee741907(v=azure.100)) pour renvoyer une référence à l’instance de rôle actuelle, puis la propriété [Role](/previous-versions/azure/reference/ee741918(v=azure.100)) pour retourner une référence au rôle lui-même.

Lorsque vous vous connectez à une instance de rôle par programme via le Kit de développement logiciel (SDK) .NET, il est relativement facile d’accéder aux informations de point de terminaison. Par exemple, une fois que vous êtes connecté à un environnement de rôle spécifique, vous pouvez obtenir le port d’un point de terminaison spécifique avec ce code :

```csharp
int port = RoleEnvironment.CurrentRoleInstance.InstanceEndpoints["StandardWeb"].IPEndpoint.Port;
```

La propriété **Instances** renvoie une collection d’objets **RoleInstance**. Cette collection contient toujours l’instance actuelle. Si le rôle ne définit pas de point de terminaison interne, la collection contient l’instance actuelle mais aucune autre instance. Lorsqu’aucun point de terminaison interne n’est défini pour le rôle, la collection contient toujours une seule instance de rôle. Si le rôle définit un point de terminaison interne, ses instances sont détectables lors de l’exécution et le nombre d’instances dans la collection correspond au nombre d’instances spécifiées pour le rôle dans le fichier de configuration de service.

> [!NOTE]
> La bibliothèque managée Azure ne fournit aucun moyen de déterminer l’intégrité des autres instances de rôle. Cependant, vous pouvez implémenter ces évaluations d’intégrité vous-même si votre service a besoin de cette fonctionnalité. Vous pouvez utiliser les [diagnostics Azure](cloud-services-dotnet-diagnostics.md) pour obtenir des informations sur l’exécution des instances de rôle.
> 
> 

Pour déterminer le numéro de port d’un point de terminaison interne sur une instance de rôle, vous pouvez utiliser la propriété [InstanceEndpoints](/previous-versions/azure/reference/ee741917(v=azure.100)) pour renvoyer un objet Dictionnaire qui contient les noms des points de terminaison et les adresses IP et ports correspondants. La propriété [IPEndpoint](/previous-versions/azure/reference/ee741919(v=azure.100)) renvoie l’adresse IP et le port pour un point de terminaison spécifié. La propriété **PublicIPEndpoint** renvoie le port pour un point de terminaison à charge équilibrée. La partie de la propriété **PublicIPEndpoint** relative à l’adresse IP n’est pas utilisée.

Voici un exemple d’itération d’instances de rôle.

```csharp
foreach (RoleInstance roleInst in RoleEnvironment.CurrentRoleInstance.Role.Instances)
{
    Trace.WriteLine("Instance ID: " + roleInst.Id);
    foreach (RoleInstanceEndpoint roleInstEndpoint in roleInst.InstanceEndpoints.Values)
    {
        Trace.WriteLine("Instance endpoint IP address and port: " + roleInstEndpoint.IPEndpoint);
    }
}
```

Voici un exemple de rôle de travail avec lequel le point de terminaison est exposé via la définition de service et commence à écouter les connexions.

> [!WARNING]
> Ce code ne fonctionne que pour un service déployé. Lorsqu’ils sont exécutés dans l’émulateur de calcul Azure, les éléments de configuration de service qui créent des points de terminaison de port directs (éléments**InstanceInputEndpoint** ) sont ignorés.
> 
> 

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Net;
using System.Net.Sockets;
using System.Threading;
using Microsoft.WindowsAzure;
using Microsoft.WindowsAzure.Diagnostics;
using Microsoft.WindowsAzure.ServiceRuntime;
using Microsoft.WindowsAzure.StorageClient;

namespace WorkerRole1
{
  public class WorkerRole : RoleEntryPoint
  {
    public override void Run()
    {
      try
      {
        // Initialize method-wide variables
        var epName = "Endpoint1";
        var roleInstance = RoleEnvironment.CurrentRoleInstance;

        // Identify direct communication port
        var myPublicEp = roleInstance.InstanceEndpoints[epName].PublicIPEndpoint;
        Trace.TraceInformation("IP:{0}, Port:{1}", myPublicEp.Address, myPublicEp.Port);

        // Identify public endpoint
        var myInternalEp = roleInstance.InstanceEndpoints[epName].IPEndpoint;

        // Create socket listener
        var listener = new Socket(
          myInternalEp.AddressFamily, SocketType.Stream, ProtocolType.Tcp);

        // Bind socket listener to internal endpoint and listen
        listener.Bind(myInternalEp);
        listener.Listen(10);
        Trace.TraceInformation("Listening on IP:{0},Port: {1}",
          myInternalEp.Address, myInternalEp.Port);

        while (true)
        {
          // Block the thread and wait for a client request
          Socket handler = listener.Accept();
          Trace.TraceInformation("Client request received.");

          // Define body of socket handler
          var handlerThread = new Thread(
            new ParameterizedThreadStart(h =>
            {
              var socket = h as Socket;
              Trace.TraceInformation("Local:{0} Remote{1}",
                socket.LocalEndPoint, socket.RemoteEndPoint);

              // Shut down and close socket
              socket.Shutdown(SocketShutdown.Both);
              socket.Close();
            }
          ));

          // Start socket handler on new thread
          handlerThread.Start(handler);
        }
      }
      catch (Exception e)
      {
        Trace.TraceError("Caught exception in run. Details: {0}", e);
      }
    }

    public override bool OnStart()
    {
      // Set the maximum number of concurrent connections 
      ServicePointManager.DefaultConnectionLimit = 12;

      // For information on handling configuration changes
      // see the MSDN topic at https://go.microsoft.com/fwlink/?LinkId=166357.
      return base.OnStart();
    }
  }
}
```

## <a name="network-traffic-rules-to-control-role-communication"></a>Règles de trafic réseau pour contrôler la communication entre les rôles
Après avoir défini les points de terminaison internes, vous pouvez ajouter des règles de trafic réseau (basées sur les points de terminaison que vous avez créés) pour contrôler la façon dont les instances de rôle peuvent communiquer entre elles. Le diagramme suivant montre quelques scénarios courants relatifs au contrôle de la communication entre les rôles :

![Scénarios de règles de trafic réseau](./media/cloud-services-enable-communication-role-instances/scenarios.png "Scénarios de règles de trafic réseau")

L’exemple de code suivant montre des définitions de rôles pour les rôles illustrés dans le diagramme précédent. Chaque définition de rôle inclut au moins un point de terminaison interne défini :

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <WebRole name="WebRole1" vmsize="Medium">
    <Sites>
      <Site name="Web">
        <Bindings>
          <Binding name="HttpIn" endpointName="HttpIn" />
        </Bindings>
      </Site>
    </Sites>
    <Endpoints>
      <InputEndpoint name="HttpIn" protocol="http" port="80" />
      <InternalEndpoint name="InternalTCP1" protocol="tcp" />
    </Endpoints>
  </WebRole>
  <WorkerRole name="WorkerRole1">
    <Endpoints>
      <InternalEndpoint name="InternalTCP2" protocol="tcp" />
    </Endpoints>
  </WorkerRole>
  <WorkerRole name="WorkerRole2">
    <Endpoints>
      <InternalEndpoint name="InternalTCP3" protocol="tcp" />
      <InternalEndpoint name="InternalTCP4" protocol="tcp" />
    </Endpoints>
  </WorkerRole>
</ServiceDefinition>
```

> [!NOTE]
> La communication entre les rôles peut être restreinte avec les points de terminaison internes des ports fixes et affectés automatiquement.
> 
> 

Par défaut, une fois un point de terminaison interne défini, la communication peut s’effectuer à partir de n’importe quel rôle vers le point de terminaison interne d’un rôle sans restriction. Pour restreindre la communication, vous devez ajouter un élément **NetworkTrafficRules** à l’élément **ServiceDefinition** dans le fichier de définition de service.

### <a name="scenario-1"></a>Scénario 1
Autoriser uniquement le trafic réseau de **WebRole1** à **WorkerRole1**.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo>
      <Destinations>
        <RoleEndpoint endpointName="InternalTCP2" roleName="WorkerRole1"/>
      </Destinations>
      <AllowAllTraffic/>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WebRole1"/>
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
</ServiceDefinition>
```

### <a name="scenario-2"></a>Scénario 2
Autoriser uniquement le trafic réseau de **WebRole1** à **WorkerRole1** et **WorkerRole2**.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo>
      <Destinations>
        <RoleEndpoint endpointName="InternalTCP2" roleName="WorkerRole1"/>
        <RoleEndpoint endpointName="InternalTCP3" roleName="WorkerRole2"/>
      </Destinations>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WebRole1"/>
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
</ServiceDefinition>
```

### <a name="scenario-3"></a>Scénario 3
Autoriser uniquement le trafic réseau de **WebRole1** à **WorkerRole1** et de **WorkerRole1** à **WorkerRole2**.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo>
      <Destinations>
        <RoleEndpoint endpointName="InternalTCP2" roleName="WorkerRole1"/>
      </Destinations>
      <AllowAllTraffic/>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WebRole1"/>
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo>
      <Destinations>
        <RoleEndpoint endpointName="InternalTCP3" roleName="WorkerRole2"/>
      </Destinations>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WorkerRole1"/>
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
</ServiceDefinition>
```

### <a name="scenario-4"></a>Scénario 4
Autoriser uniquement le trafic réseau de **WebRole1** à **WorkerRole1**, de **WebRole1** à **WorkerRole2** et de **WorkerRole1** à **WorkerRole2**.

```xml
<ServiceDefinition name="MyService" xmlns="http://schemas.microsoft.com/ServiceHosting/2008/10/ServiceDefinition">
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo>
      <Destinations>
        <RoleEndpoint endpointName="InternalTCP2" roleName="WorkerRole1"/>
      </Destinations>
      <AllowAllTraffic/>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WebRole1"/>
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo >
      <Destinations>
        <RoleEndpoint endpointName="InternalTCP3" roleName="WorkerRole2"/>
      </Destinations>
      <AllowAllTraffic/>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WorkerRole1"/>
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
  <NetworkTrafficRules>
    <OnlyAllowTrafficTo >
      <Destinations>
        <RoleEndpoint endpointName="InternalTCP4" roleName="WorkerRole2"/>
      </Destinations>
      <AllowAllTraffic/>
      <WhenSource matches="AnyRule">
        <FromRole roleName="WebRole1"/>
      </WhenSource>
    </OnlyAllowTrafficTo>
  </NetworkTrafficRules>
</ServiceDefinition>
```

Vous trouverez une référence de schéma XML pour les éléments ci-dessus [ici](/previous-versions/azure/reference/gg557551(v=azure.100)).

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur le [modèle](cloud-services-model-and-package.md)de service cloud.




