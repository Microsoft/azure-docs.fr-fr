---
title: Schéma NetworkConfiguration d’Azure Cloud Services | Microsoft Docs
description: Découvrez les éléments enfants de l’élément NetworkConfiguration du fichier de configuration de service, qui spécifie les valeurs de réseau virtuel et DNS.
ms.custom: ''
ms.date: 12/07/2016
services: cloud-services
ms.service: cloud-services
ms.topic: reference
caps.latest.revision: 28
author: tgore03
ms.author: tagore
ms.openlocfilehash: a3f62bb33332e818cc40c5195a1cc2f667f819b4
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92072575"
---
# <a name="azure-cloud-services-config-networkconfiguration-schema"></a>Schéma NetworkConfiguration de configuration d’Azure Cloud Services

L’élément `NetworkConfiguration` du fichier de configuration de service spécifie les valeurs du réseau virtuel et DNS. Ces paramètres sont facultatifs pour les services cloud.

Vous pouvez consulter les ressources suivantes pour en savoir plus sur les réseaux virtuels et les schémas associés :

- [Schéma de configuration de services cloud (classique)](schema-cscfg-file.md)
- [Schéma de définition de services cloud (classique)](schema-csdef-file.md)
- [Créer un réseau virtuel (classique)](/previous-versions/azure/virtual-network/virtual-networks-create-vnet-classic-pportal)

## <a name="networkconfiguration-element"></a>Élément NetworkConfiguration
L’exemple suivant illustre l’élément `NetworkConfiguration` et ses éléments enfants.

```xml
<ServiceConfiguration>
  <NetworkConfiguration>
    <AccessControls>
      <AccessControl name="aclName1">
        <Rule order="<rule-order>" action="<rule-action>" remoteSubnet="<subnet-address>" description="rule-description"/>
      </AccessControl>
    </AccessControls>
    <EndpointAcls>
      <EndpointAcl role="<role-name>" endpoint="<endpoint-name>" accessControl="<acl-name>"/>
    </EndpointAcls>
    <Dns>
      <DnsServers>
        <DnsServer name="<server-name>" IPAddress="<server-address>" />
      </DnsServers>
    </Dns>
    <VirtualNetworkSite name="Group <RG-VNet> <VNet-name>"/>
    <AddressAssignments>
      <InstanceAddress roleName="<role-name>">
        <Subnets>
          <Subnet name="<subnet-name>"/>
        </Subnets>
      </InstanceAddress>
      <ReservedIPs>
        <ReservedIP name="<reserved-ip-name>"/>
      </ReservedIPs>
    </AddressAssignments>
  </NetworkConfiguration>
</ServiceConfiguration>
```

La table suivante décrit les éléments enfants de l’élément `NetworkConfiguration`.

| Élément       | Description |
| ------------- | ----------- |
| AccessControl | facultatif. Spécifie les règles d’accès aux points de terminaison dans un service cloud. Le nom du contrôle d’accès est défini par une chaîne pour l’attribut `name`. L’élément `AccessControl` contient un ou plusieurs éléments `Rule`. Il est possible de définir plusieurs éléments `AccessControl`.|
| Règle | facultatif. Spécifie l’action à réaliser pour une plage de sous-réseau spécifiée d’adresses IP. L’ordre de la règle est défini par une valeur de chaîne pour l’attribut `order`. Plus le numéro de règle est petit, plus la priorité est élevée. Par exemple, vous pouvez spécifier des règles avec des numéros d’ordre de 100, 200 et 300. La règle avec le numéro d’ordre 100 a priorité sur la règle avec le numéro d’ordre 200.<br /><br /> L’action de la règle est définie par une chaîne pour l’attribut `action`. Les valeurs possibles sont les suivantes :<br /><br /> -   `permit` : indique que seuls les paquets de la plage de sous-réseau spécifiée peuvent communiquer avec le point de terminaison.<br />-   `deny` : indique que l’accès est refusé aux points de terminaison dans la plage de sous-réseau spécifiée.<br /><br /> La plage de sous-réseau d’adresses IP affectées par la règle est définie par une chaîne pour l’attribut `remoteSubnet`. La description de la règle est définie par une chaîne pour l’attribut `description`.|
| EndpointAcl | facultatif. Spécifie l’affectation de règles de contrôle d’accès à un point de terminaison. Le nom du rôle qui contient le point de terminaison est défini par une chaîne pour l’attribut `role`. Le nom du point de terminaison est défini par une chaîne pour l’attribut `endpoint`. Le nom de l’ensemble de règles `AccessControl` à appliquer au point de terminaison est défini dans une chaîne pour l’attribut `accessControl`. Il est possible de définir plusieurs éléments `EndpointAcl`.|
| DnsServer | facultatif. Spécifie les paramètres d’un serveur DNS. Vous pouvez spécifier des paramètres pour des serveurs DNS sans réseau virtuel. Le nom du serveur DNS est défini par une chaîne pour l’attribut `name`. L’adresse IP du serveur DNS est définie par une chaîne pour l’attribut `IPAddress`. L'adresse IP doit être une adresse IPv4 valide.|
| VirtualNetworkSite | facultatif. Spécifie le nom du site de réseau virtuel dans lequel vous souhaitez déployer votre service cloud. Ce paramètre ne crée pas de site de réseau virtuel. Il fait référence à un site qui a été défini précédemment dans le fichier réseau pour votre réseau virtuel. Un service cloud ne peut faire partie que d’un seul réseau virtuel. Si vous ne spécifiez pas ce paramètre, le service cloud ne sera pas déployé sur un réseau virtuel. Le nom du site de réseau virtuel est défini par une chaîne pour l’attribut `name`.|
| InstanceAddress | facultatif. Spécifie l’association d’un rôle à un sous-réseau ou à un ensemble de sous-réseaux du réseau virtuel. Quand vous associez un nom de rôle à une adresse d’instance, vous pouvez spécifier les sous-réseaux auxquels vous souhaitez associer ce rôle. L’élément `InstanceAddress` contient un élément Subnets. Le nom du rôle associé aux sous-réseaux est défini par une chaîne pour l’attribut `roleName`.|
| Subnet | facultatif. Spécifie le sous-réseau qui correspond au nom du sous-réseau dans le fichier de configuration réseau. Le nom du sous-réseau est défini par une chaîne pour l’attribut `name`.|
| ReservedIP | facultatif. Spécifie l’adresse IP réservée à associer au déploiement. Vous devez utiliser l’action Créer une adresse IP réservée pour créer l’adresse IP réservée. Chaque déploiement dans un service cloud peut être associé à une adresse IP réservée. Le nom de l’adresse IP réservée est défini par une chaîne pour l’attribut `name`.|

## <a name="see-also"></a>Voir aussi
[Schéma de configuration de services cloud (classique)](schema-cscfg-file.md)