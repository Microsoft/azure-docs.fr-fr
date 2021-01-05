---
title: Probes (diagnostics) Azure Service Fabric
description: Explique comment modéliser un diagnostic probe liveness dans Azure Service Fabric à l’aide de fichiers manifestes d’application et de service.
ms.topic: conceptual
author: tugup
ms.author: tugup
ms.date: 3/12/2020
ms.openlocfilehash: 07a1b836ca7ea79244e303f54654dfcaa6e5fcb9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82137584"
---
# <a name="liveness-probe"></a>Probe liveness
À partir de la version 7.1, Azure Service Fabric prend en charge un mécanisme de diagnostic probe liveness pour les applications [conteneurisées][containers-introduction-link]. Un probe liveness permet de signaler l’activité d’une application conteneurisée, qui redémarrera si elle ne répond pas rapidement.
Cet article fournit une vue d’ensemble de la manière de définir un probe liveness à l’aide de fichiers manifeste.

Avant de poursuivre la lecture de cet article, familiarisez-vous avec le [modèle d’application Service Fabric][application-model-link] et le [modèle d’hébergement Service Fabric][hosting-model-link].

> [!NOTE]
> Probe liveness est pris en charge uniquement pour les conteneurs en mode de mise en réseau NAT.

## <a name="semantics"></a>Sémantique
Vous ne pouvez spécifier qu’un seul probe liveness par conteneur, et vous pouvez contrôler son comportement à l’aide des champs suivants :

* `initialDelaySeconds`: délai initial, en secondes, avant le début de l’exécution du diagnostic probe après que le conteneur a démarré. La valeur prise en charge est **int**. La valeur par défaut est 0 et la valeur minimale est 0.

* `timeoutSeconds`: délai, en secondes, après lequel nous considérons le diagnostic probe comme ayant échoué s’il ne s’est pas terminé avec succès. La valeur prise en charge est **int**. La valeur par défaut est 1 et la valeur minimale est 1.

* `periodSeconds`: période, en secondes, pour spécifier la fréquence du diagnostic. La valeur prise en charge est **int**. La valeur par défaut est 10 et la valeur minimale est 1.

* `failureThreshold`: quand cette valeur est atteinte, le conteneur redémarre. La valeur prise en charge est **int**. La valeur par défaut est 3 et la valeur minimale est 1.

* `successThreshold`: en cas d’échec, pour que le diagnostic soit considéré comme réussi, il doit s’exécuter correctement pour cette valeur. La valeur prise en charge est **int**. La valeur par défaut est 1 et la valeur minimale est 1.

Il peut y avoir, au plus, un diagnostic sur un conteneur à tout moment. Si le diagnostic ne se termine pas avant l’expiration du délai défini dans **timeoutSeconds**, patientez et comptez le temps jusqu’au **FailureThreshold**. 

En outre, ServiceFabric déclenche les [rapports d’intégrité][health-introduction-link] du diagnostic probe suivants sur **DeployedServicePackage** :

* `OK`: le diagnostic réussit pour la valeur définie dans **successThreshold**.

* `Error`: **failureCount** ==  **failureThreshold** avant le redémarrage du conteneur.

* `Warning`: 
    * le diagnostic échoue et **failureCount** < **failureThreshold**. Ce rapport d’intégrité reste jusqu’à ce que **failureCount** atteigne la valeur définie dans **failureThreshold** ou **successThreshold**.
    * En cas de réussite après un échec, l’avertissement reste, mais avec les réussites consécutives mises à jour.

## <a name="specifying-a-liveness-probe"></a>Spécification d’un diagnostic probe liveness

Vous pouvez spécifier un diagnostic probe liveness dans le fichier ApplicationManifest.xml sous **ServiceManifestImport**.

Le diagnostic peut concerner l’un des éléments suivants :

* HTTP
* TCP
* Exec 

### <a name="http-probe"></a>Diagnostic HTTP

Pour un diagnostic HTTP, Service Fabric envoie une requête HTTP au port et au chemin que vous spécifiez. Un code de retour supérieur ou égal à 200, et inférieur à 400, indique une réussite.

Voici un exemple qui montre comment spécifier un diagnostic HTTP :

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <HttpGet Path="/" Port="8081" Scheme="http">
              <HttpHeader Name="Foo" Value="Val"/>
              <HttpHeader Name="Bar" Value="val1"/>
            </HttpGet>
          </Probe>
        </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

Le diagnostic HTTP a des propriétés supplémentaires que vous pouvez définir :

* `path`: chemin à utiliser dans la requête HTTP.

* `port`: port à utiliser pour les diagnostics. Cette propriété est obligatoire. La plage est comprise entre 1 et 65 535.

* `scheme`: schéma à utiliser pour la connexion au package de code. Si cette propriété a la valeur HTTPS, la vérification du certificat est ignorée. Le paramètre par défaut est HTTP.

* `httpHeader`: en-têtes à définir dans la requête. Vous pouvez spécifier plusieurs en-têtes.

* `host`: adresse IP de l’hôte à laquelle se connecter.

### <a name="tcp-probe"></a>Diagnostic TCP

Pour un diagnostic TCP, Service Fabric essaie d’ouvrir un socket sur le conteneur à l’aide du port spécifié. S’il peut établir une connexion, le diagnostic est considéré comme ayant réussi. Voici un exemple montrant comment spécifier un diagnostic qui utilise un socket TCP :

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <TcpSocket Port="8081"/>
          </Probe>
        </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

### <a name="exec-probe"></a>Diagnostic Exec

Ce diagnostic émet une commande **exec** dans le conteneur et attend que la commande se termine.

> [!NOTE]
> La commande **exec** prend une chaîne séparée par des virgules. La commande de l’exemple suivant fonctionne pour un conteneur Linux.
> Si vous essayez de diagnostiquer un conteneur Windows, utilisez **cmd**.

```xml
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateless1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <CodePackagePolicy CodePackageRef="Code">
        <Probes>
          <Probe Type="Liveness" FailureThreshold="5" SuccessThreshold="2" InitialDelaySeconds="10" PeriodSeconds="30" TimeoutSeconds="20">
            <Exec>
              <Command>ping,-c,2,localhost</Command>
            </Exec>
          </Probe>        
       </Probes>
      </CodePackagePolicy>
    </Policies>
  </ServiceManifestImport>
```

## <a name="next-steps"></a>Étapes suivantes
Pour accéder à des informations connexes, consultez l’article suivant :
* [Service Fabric et conteneurs][containers-introduction-link]

<!-- Links -->
[containers-introduction-link]: service-fabric-containers-overview.md
[health-introduction-link]: service-fabric-health-introduction.md
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md

