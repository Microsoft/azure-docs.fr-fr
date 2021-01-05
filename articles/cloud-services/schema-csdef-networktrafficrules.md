---
title: Déf. Azure Cloud Services Schéma NetworkTrafficRules | Microsoft Docs
description: En savoir plus sur NetworkTrafficRules, qui limite les rôles qui peuvent accéder aux points de terminaison internes d’un rôle. Il se combine avec les rôles dans un fichier de définition de service.
ms.custom: ''
ms.date: 04/14/2015
services: cloud-services
ms.reviewer: ''
ms.service: cloud-services
ms.suite: ''
ms.tgt_pltfrm: ''
ms.topic: reference
ms.assetid: 351b369f-365e-46c1-82ce-03fc3655cc88
caps.latest.revision: 17
author: tgore03
ms.author: tagore
ms.openlocfilehash: e53c10395ec3168e656633cc43fb2d01902209fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79534726"
---
# <a name="azure-cloud-services-definition-networktrafficrules-schema"></a>Schéma NetworkTrafficRules de définition des services cloud Azure
Le nœud `NetworkTrafficRules` est un élément facultatif dans le fichier de définition de service, qui spécifie comment les rôles communiquent entre eux. Il limite les rôles qui peuvent accéder aux points de terminaison internes du rôle spécifique. Le `NetworkTrafficRules` n’est pas un élément autonome ; il est associé à deux ou plusieurs rôles dans un fichier de définition de service.

L’extension par défaut du fichier de définition de service est .csdef.

> [!NOTE]
>  Le nœud `NetworkTrafficRules` n’est disponible que par le biais du kit SDK Azure version 1.3 ou supérieure.

## <a name="basic-service-definition-schema-for-the-network-traffic-rules"></a>Schéma de définition de service de base pour les règles de trafic réseau
Le format de base d’un fichier de définition de service contenant des définitions de trafic réseau se présente comme suit.

```xml
<ServiceDefinition …>
   <NetworkTrafficRules>
      <OnlyAllowTrafficTo>
         <Destinations>
            <RoleEndpoint endpointName="<name-of-the-endpoint>" roleName="<name-of-the-role-containing-the-endpoint>"/>
         </Destinations>
         <AllowAllTraffic/>
         <WhenSource matches="[AnyRule]">
            <FromRole roleName="<name-of-the-role-to-allow-traffic-from>"/>
         </WhenSource>
      </OnlyAllowTrafficTo>
   </NetworkTrafficRules>
</ServiceDefinition>
```

## <a name="schema-elements"></a>Éléments du schéma
Le nœud `NetworkTrafficRules` du fichier de définition de service inclut ces éléments, ils sont détaillés dans les sections suivantes de cette rubrique :

[Élément NetworkTrafficRules](#NetworkTrafficRules)

[Élément OnlyAllowTrafficTo](#OnlyAllowTrafficTo)

[Élément Destinations](#Destinations)

[Élément RoleEndpoint](#RoleEndpoint)

Élément AllowAllTraffic

[Élément WhenSource](#WhenSource)

[Élément FromRole](#FromRole)

##  <a name="networktrafficrules-element"></a><a name="NetworkTrafficRules"></a> Élément NetworkTrafficRules
L’élément `NetworkTrafficRules` spécifie les rôles qui peuvent communiquer avec tel point de terminaison sur un autre rôle. Un service peut contenir une définition `NetworkTrafficRules`.

##  <a name="onlyallowtrafficto-element"></a><a name="OnlyAllowTrafficTo"></a> Élément OnlyAllowTrafficTo
L’élément `OnlyAllowTrafficTo` décrit une collection de points de terminaison de destination et les rôles qui peuvent communiquer avec eux. Vous pouvez spécifier plusieurs `OnlyAllowTrafficTo` nœuds.

##  <a name="destinations-element"></a><a name="Destinations"></a> Élément Destinations
L’élément `Destinations` décrit une collection de RoleEndpoint qui peuvent communiquer avec eux.

##  <a name="roleendpoint-element"></a><a name="RoleEndpoint"></a> Élément RoleEndpoint
L’élément `RoleEndpoint` décrit un point de terminaison sur un rôle qui permet de communiquer avec lui. Vous pouvez spécifier plusieurs éléments `RoleEndpoint` s’il existe plus d’un point de terminaison sur le rôle.

| Attribut      | Type     | Description |
| -------------- | -------- | ----------- |
| `endpointName` | `string` | Obligatoire. Le nom du point de terminaison vers lequel autoriser le trafic.|
| `roleName`     | `string` | Obligatoire. Le nom du rôle web vers lequel autoriser la communication.|

## <a name="allowalltraffic-element"></a>Élément AllowAllTraffic
L’élément `AllowAllTraffic` est une règle qui autorise tous les rôles à communiquer avec les points de terminaison définis dans le nœud `Destinations`.

##  <a name="whensource-element"></a><a name="WhenSource"></a> Élément WhenSource
L’élément `WhenSource` décrit une collection de rôles qui peuvent communiquer avec les points de terminaison définis dans le nœud `Destinations`.

| Attribut | Type     | Description |
| --------- | -------- | ----------- |
| `matches` | `string` | Obligatoire. Spécifie la règle à appliquer lorsque vous autorisez les communications. Actuellement, la seule valeur possible est `AnyRule`.|
  
##  <a name="fromrole-element"></a><a name="FromRole"></a> Élément FromRole
L’élément `FromRole` spécifie les rôles qui peuvent communiquer avec les points de terminaison définis dans le nœud `Destinations`. Vous pouvez spécifier plusieurs éléments `FromRole` s’il existe plus d’un rôle capable de communiquer avec les points de terminaison.

| Attribut  | Type     | Description |
| ---------- | -------- | ----------- |
| `roleName` | `string` | Obligatoire. Nom du rôle à partir duquel autoriser la communication.|

## <a name="see-also"></a>Voir aussi
[Schéma de définition de services cloud (classique)](schema-csdef-file.md)




