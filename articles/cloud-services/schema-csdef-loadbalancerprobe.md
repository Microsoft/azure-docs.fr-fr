---
title: Déf. Azure Cloud Services Schéma LoadBalancerProbe | Microsoft Docs
description: Le schéma LoadBalancerProbe défini par le client est un probe d’intégrité des points de terminaison dans les instances de rôle. Il se combine avec des rôles de travail ou web dans un fichier de définition de service.
ms.custom: ''
ms.date: 04/14/2015
services: cloud-services
ms.service: cloud-services
ms.topic: reference
caps.latest.revision: 14
author: georgewallace
ms.author: tagore
ms.openlocfilehash: 6d0e84b6724d9df4162d4be3e06a9952087a53a6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79537344"
---
# <a name="azure-cloud-services-definition-loadbalancerprobe-schema"></a>Schéma LoadBalancerProbe de définition Azure Cloud Services
La sonde d’équilibreur de charge est une sonde d’intégrité définie par le client qui vérifie les points de terminaison UDP et les points de terminaison dans les instances de rôle. Le `LoadBalancerProbe` n’est pas un élément autonome ; il est associé au rôle web ou au rôle de travail dans un fichier de définition de service. Un `LoadBalancerProbe` peut être utilisé par plusieurs rôles.

L’extension par défaut du fichier de définition de service est .csdef.

## <a name="the-function-of-a-load-balancer-probe"></a>Fonction d’une sonde d’équilibreur de charge
Azure Load Balancer est chargé du routage du trafic entrant vers vos instances de rôle. L’équilibreur de charge détermine quelles instances peuvent recevoir du trafic en sondant régulièrement chaque instance afin de déterminer son intégrité. L’équilibreur de charge sonde chaque instance plusieurs fois par minute. Il existe deux options pour fournir l’intégrité de l’instance à l’équilibreur de charge : la sonde d’équilibreur de charge par défaut ou une sonde d’équilibreur de charge personnalisée, qui est implémentée en définissant LoadBalancerProbe dans le fichier .csdef.

La sonde d’équilibreur de charge par défaut utilise l’agent invité de la machine virtuelle, qui écoute et répond avec une réponse HTTP 200 OK uniquement si l’instance est prête (c.-à-d., si l’état de l’instance n’est pas Occupé, Recyclage, Arrêté, etc.). Si l’agent invité ne répond pas avec HTTP 200 OK, Azure Load Balancer marque l’instance comme ne répondant pas et arrête d’envoyer du trafic vers celle-ci. Azure Load Balancer continue à effectuer des tests ping de l’instance, et si l’agent invité répond avec HTTP 200, Azure Load Balancer envoie de nouveau le trafic vers cette instance. Lorsque vous utilisez un rôle web, le code de votre site web s’exécute généralement dans w3wp.exe, qui n’est pas surveillé par l’agent Azure Fabric ou l’agent invité, ce qui signifie que les échecs dans w3wp.exe (par exemple, les réponses HTTP 500) ne sont pas signalés à l’agent invité et l’équilibreur de charge ne sait pas qu’il doit placer cette instance hors service.

La sonde d’équilibreur de charge personnalisée remplace la sonde de l’agent invité par défaut et vous permet de créer votre propre logique personnalisée pour déterminer l’intégrité de l’instance de rôle. L’équilibreur de charge sonde régulièrement votre point de terminaison (par défaut toutes les 15 secondes) et l’instance est considérée en rotation si elle répond avec TCP ACK ou HTTP 200 dans le délai imparti (31 secondes par défaut). Il peut être utile d’implémenter votre propre logique pour supprimer des instances de la rotation de l’équilibreur de charge, par exemple en renvoyant un état autre que 200 si l’instance représente plus de 90 % de l’UC. Pour les rôles web qui utilisent w3wp.exe, cela signifie également que vous obtenez une analyse automatique de votre site web, dans la mesure où les défaillances dans votre code de site web renvoient un état autre que 200 à la sonde d’équilibreur de charge. Si vous ne définissez pas de LoadBalancerProbe dans le fichier .csdef, alors le comportement par défaut de l’équilibreur de charge (comme décrit précédemment) est utilisé.

Si vous utilisez une sonde d’équilibreur de charge personnalisée, vous devez vous assurer que votre logique prend en compte la méthode RoleEnvironment.OnStop. Lorsque vous utilisez la sonde d’équilibreur de charge par défaut, l’instance est extraite de la rotation avant qu’OnStop ne soit appelé, mais une sonde d’équilibreur de charge personnalisée peut continuer à renvoyer une réponse 200 OK pendant l’événement OnStop. Si vous utilisez l’événement OnStop pour nettoyer le cache, arrêter le service ou apporter toute autre modification qui peut affecter le comportement runtime de votre service, vous devez vous assurer que votre logique de sonde d’équilibreur de charge personnalisée supprime l’instance de la rotation.

## <a name="basic-service-definition-schema-for-a-load-balancer-probe"></a>Schéma de définition de service de base pour une sonde d’équilibreur de charge
 Le format de base d’un fichier de définition de service contenant une sonde d’équilibreur de charge est le suivant.

```xml
<ServiceDefinition …>
   <LoadBalancerProbes>
      <LoadBalancerProbe name="<load-balancer-probe-name>" protocol="[http|tcp]" path="<uri-for-checking-health-status-of-vm>" port="<port-number>" intervalInSeconds="<interval-in-seconds>" timeoutInSeconds="<timeout-in-seconds>"/>
   </LoadBalancerProbes>
</ServiceDefinition>
```

## <a name="schema-elements"></a>Éléments du schéma
L’élément `LoadBalancerProbes` du fichier de définition de service inclut les éléments suivants :

- [Élément LoadBalancerProbes](#LoadBalancerProbes)
- [Élément LoadBalancerProbe](#LoadBalancerProbe)

##  <a name="loadbalancerprobes-element"></a><a name="LoadBalancerProbes"></a> Élément LoadBalancerProbes
L’élément `LoadBalancerProbes` décrit la collection de sondes d’équilibreur de charge. Cet élément est l’élément parent de l’[élément LoadBalancerProbe](#LoadBalancerProbe). 

##  <a name="loadbalancerprobe-element"></a><a name="LoadBalancerProbe"></a> Élément LoadBalancerProbe
L’élément `LoadBalancerProbe` définit la sonde d’intégrité pour un modèle. Vous pouvez définir plusieurs sondes d’équilibreur de charge. 

Le tableau suivant décrit les attributs de l’élément `LoadBalancerProbe` :

|Attribut|Type|Description|
| ------------------- | -------- | -----------------|
| `name`              | `string` | Obligatoire. Le nom de la sonde d’équilibreur de charge. Le nom doit être unique.|
| `protocol`          | `string` | Obligatoire. Spécifie le protocole du point de terminaison. Les valeurs possibles sont `http` ou `tcp`. Si `tcp` est spécifié, une réponse ACK est requise pour que la sonde réussisse. Si `http` est spécifié, une réponse 200 OK de l’URI spécifiée est requise pour que la sonde réussisse.|
| `path`              | `string` | L’URI utilisée pour demander l’état d’intégrité de la machine virtuelle. `path` est nécessaire si `protocol` est défini sur `http`. Dans le cas contraire, il n’est pas autorisé.<br /><br /> Aucune valeur par défaut.|
| `port`              | `integer` | facultatif. Le port de communication de la sonde. Il est facultatif pour tous les points de terminaison, car le même port sera ensuite utilisé pour la sonde. Vous pouvez également configurer un port différent pour leur détection. Les valeurs possibles sont comprises entre 1 et 65535, ces deux valeurs étant incluses.<br /><br /> La valeur par défaut est définie par le point de terminaison.|
| `intervalInSeconds` | `integer` | facultatif. L’intervalle, en secondes, de fréquence de détection de l’état d’intégrité du point de terminaison. En règle générale, l’intervalle est légèrement inférieur à la moitié du délai d’expiration alloué (en secondes), ce qui permet d’utiliser deux sondes complètes avant de mettre l’instance hors rotation.<br /><br /> La valeur par défaut est 15, la valeur minimum 5.|
| `timeoutInSeconds`  | `integer` | facultatif. Le délai d’expiration, en secondes, appliqué à la sonde lorsqu’aucune réponse n’entraîne l’arrêt du trafic vers le point de terminaison. Cette valeur permet d’extraire des points de terminaison de la rotation plus rapidement ou plus lentement que le nombre de fois habituel dans Azure (valeurs par défaut).<br /><br /> La valeur par défaut est 31, la valeur minimum 11.|

## <a name="see-also"></a>Voir aussi
[Schéma de définition de services cloud (classique)](schema-csdef-file.md)