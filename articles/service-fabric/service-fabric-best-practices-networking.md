---
title: Meilleures pratiques de mise en réseau Azure Service Fabric
description: Considérations relatives aux règles et à la conception pour la gestion de la connectivité réseau à l’aide d’Azure Service Fabric.
author: chrpap
ms.topic: conceptual
ms.date: 01/23/2019
ms.author: chrpap
ms.openlocfilehash: b8db69792b31fd82646757423e669e39e8539d06
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91630700"
---
# <a name="networking"></a>Mise en réseau

Lorsque vous créez et gérez des clusters Azure Service Fabric, vous assurez une connectivité réseau à vos nœuds et à vos applications. Les ressources de mise en réseau comprennent les plages d'adresses IP, les réseaux virtuels, les équilibreurs de charge et les groupes de sécurité réseau. Dans cet article, vous découvrirez les meilleures pratiques liées à ces ressources.

Consultez [Modèles de mise en réseau de Service Fabric](./service-fabric-patterns-networking.md) Azure pour savoir comment créer des clusters utilisant les fonctionnalités suivantes : réseau virtuel ou sous-réseau existant, adresse IP publique statique, équilibreur de charge interne uniquement, ou équilibreur de charge interne et externe.

## <a name="infrastructure-networking"></a>Mise en réseau de l’infrastructure
Optimisez les performances de votre machine virtuelle avec la mise en réseau accélérée, en déclarant la propriété *enableAcceleratedNetworking* dans votre modèle Resource Manager. L’extrait suivant fait partie d’un groupe de machines virtuelles identiques NetworkInterfaceConfigurations qui permet la mise en réseau accélérée :

```json
"networkInterfaceConfigurations": [
  {
    "name": "[concat(variables('nicName'), '-0')]",
    "properties": {
      "enableAcceleratedNetworking": true,
      "ipConfigurations": [
        {
        <snip>
        }
      ],
      "primary": true
    }
  }
]
```
Un cluster Service Fabric peut être configuré sur [Linux avec mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-cli.md), et [Windows avec mise en réseau accélérée](../virtual-network/create-vm-accelerated-networking-powershell.md).

La mise en réseau accélérée est prise en charge pour les références SKU de la série de machines virtuelles Azure : D/DSv2, D/DSv3, E/ESv3, F/FS, FSv2 et Ms/Mms. La mise en réseau accélérée a été testée avec succès à l’aide de la référence SKU Standard_DS8_v3 pour un cluster Service Fabric Windows le 23/01/2019 et à l’aide de la référence SKU Standard_DS12_v2 pour un cluster Service Fabric Linux le 29/01/2019.

Pour activer la mise en réseau accélérée sur un cluster Service Fabric existant, vous devez d’abord [mettre à l'échelle ce cluster en ajoutant un groupe de machines virtuelles identiques](./virtual-machine-scale-set-scale-node-type-scale-out.md), afin d'effectuer ce qui suit :
1. Approvisionner un type de nœud avec mise en réseau accélérée activée
2. Migrer vos services et leur état vers le type de nœud approvisionné avec mise en réseau accélérée activée

La mise à l’échelle de l’infrastructure est requise pour activer la mise en réseau accélérée sur un cluster existant. En effet, l'activation de la mise en réseau accélérée peut entraîner un temps d’arrêt, car elle implique que toutes les machines virtuelles d'un groupe à haute disponibilité soient [arrêtées et libérées avant d'activer la mise en réseau accélérée sur une carte réseau existante](../virtual-network/create-vm-accelerated-networking-cli.md#enable-accelerated-networking-on-existing-vms).

## <a name="cluster-networking"></a>Mise en réseau de cluster

* Pour déployer des clusters Service Fabric dans un réseau virtuel existant, suivez la procédure décrite dans [Modèles de mise en réseau de Service Fabric](./service-fabric-patterns-networking.md).

* Les groupes de sécurité réseau (NSG) sont recommandés pour les types de nœuds afin de limiter le trafic entrant et sortant de leur cluster. Assurez-vous que les ports nécessaires sont ouverts dans les groupes de sécurité réseau. 

* Le type de nœud principal, qui contient les services système Service Fabric n'est pas tenu d’être exposé via l’équilibreur de charge externe et peut être exposé par un [équilibreur de charge interne](./service-fabric-patterns-networking.md#internal-only-load-balancer).

* Utilisez une [adresse IP publique statique](./service-fabric-patterns-networking.md#static-public-ip-address-1) pour votre cluster.

## <a name="network-security-rules"></a>Règles de sécurité du réseau

Les règles de base sont le minimum pour un verrouillage de sécurité d’un cluster Service Fabric géré par Azure. Ne pas ouvrir les ports suivants ou ne pas approuver l’adresse IP/l’URL empêche le bon fonctionnement du cluster et risque de ne pas être pris en charge. Avec cette règle définie, il est strictement nécessaire d’utiliser des [mises à niveau automatiques de l’image du système d’exploitation](../virtual-machine-scale-sets/virtual-machine-scale-sets-automatic-upgrade.md). Autrement, des ports supplémentaires doivent être ouverts.

### <a name="inbound"></a>Entrant 
|Priority   |Nom               |Port        |Protocol  |Source             |Destination       |Action   
|---        |---                |---         |---       |---                |---               |---
|3900       |Azure              |19080       |TCP       |Internet           |VirtualNetwork    |Allow
|3910       |Client             |19000       |TCP       |Internet           |VirtualNetwork    |Allow
|3920       |Cluster            |1025-1027   |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3930       |Éphémère          |49152-65534 |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3940       |Application        |20000-30000 |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3950       |SMB                |445         |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3960       |RDP                |3389-3488   |TCP       |Internet           |VirtualNetwork    |Deny
|3970       |SSH                |22          |TCP       |Internet           |VirtualNetwork    |Deny
|3980       |Point de terminaison personnalisé    |80          |TCP       |Internet           |VirtualNetwork    |Allow
|4100       |Bloquer le trafic entrant      |443         |Quelconque       |Quelconque                |Quelconque               |Allow

Plus d’informations sur les règles de sécurité de trafic entrant :

* **Azure**. Service Fabric Explorer utilise ce port pour parcourir et gérer votre cluster. Le fournisseur de ressources Service Fabric l’utilise également pour demander des informations sur votre cluster afin de les afficher dans le portail de gestion Azure. Si ce port n’est pas accessible à partir du fournisseur de ressources Service Fabric, vous voyez s’afficher un message tel que « Nœuds introuvables » ou « UpgradeServiceNotReachable » dans le portail Azure, et votre liste de nœuds et d’applications est vide. Cela signifie que, si vous souhaitez avoir une certaine visibilité de votre cluster dans le portail de gestion Azure, votre équilibreur de charge doit exposer une adresse IP publique et votre groupe de sécurité réseau doit autoriser le trafic 19080 entrant.  

* **Client**. Point de terminaison de connexion client pour des API telles que REST, PowerShell et CLI. 

* **Cluster**. Utilisé pour la communication entre nœuds. Ne doit jamais être bloqué.

* **Éphémère**. Service Fabric utilise une partie de ces ports comme ports d’application et le reste est disponible pour le système d’exploitation. Le service mappe également cette plage à la plage existante dans le système d’exploitation. Vous pouvez donc utiliser en toutes circonstances les plages spécifiées dans l’exemple. Assurez-vous que la différence entre les ports de début et de fin est d’au moins 255. Vous pouvez rencontrer des conflits si cette différence est trop faible, étant donné que cette plage est partagée avec le système d’exploitation. Pour afficher la plage de ports dynamiques configurée, exécutez la commande *netsh int ipv4 show dynamic port tcp*. Ces ports ne sont pas nécessaires pour les clusters Linux.

* **Application**. La plage des ports d’application doit suffire à couvrir les exigences en matière de points de terminaison de vos applications. Cette plage doit être exclusive à partir de la plage de ports dynamiques de la machine, c’est-à-dire la plage ephemeralPorts comme défini dans la configuration. Service Fabric utilise ces ports chaque fois que des nouveaux ports sont nécessaires, et prend également en charge l’ouverture du pare-feu pour ces ports sur les nœuds.

* **SMB**. Le protocole SMB est utilisé par le service ImageStore pour deux scénarios. Ce port est nécessaire pour télécharger les packages à partir de l’ImageStore par les nœuds, ainsi que pour les répliquer entre les réplicas. 

* **RDP**. Facultatif, si le protocole RDP est requis à partir d’Internet ou de VirtualNetwork pour des scénarios JumpBox. 

* **SSH**. Facultatif, si le protocole SSH est requis à partir d’Internet ou de VirtualNetwork pour des scénarios JumpBox.

* **Point de terminaison personnalisé**. Exemple destiné à votre application pour activer un point de terminaison accessible via Internet.

### <a name="outbound"></a>Sortant

|Priority   |Nom               |Port        |Protocol  |Source             |Destination       |Action   
|---        |---                |---         |---       |---                |---               |---
|3900       |Réseau            |Quelconque         |TCP       |VirtualNetwork     |VirtualNetwork    |Allow
|3910       |Fournisseur de ressources  |443         |TCP       |VirtualNetwork     |ServiceFabric     |Allow
|3920       |Mettre à niveau            |443         |TCP       |VirtualNetwork     |Internet          |Allow
|3950       |Bloquer le trafic sortant     |Quelconque         |Quelconque       |Quelconque                |Quelconque               |Deny

Pour plus d’informations sur les règles de sécurité du trafic sortant, procédez comme suit :

* **Réseau**. Canal de communication pour les sous-réseaux et vers un autre réseau virtuel.

* **Fournisseur de ressources**. Connexion établie par l’UpgradeService pour exécuter tous les déploiements ARM opérés par le fournisseur de ressources Service Fabric.

* **Mise à niveau**. Service de mise à niveau utilisant l’adresse download.microsoft.com pour obtenir les bits. Nécessaire pour configurer et réimager, ainsi que pour les mises à niveau du runtime. Le service fonctionne avec des adresses IP dynamiques. Dans le scénario d’un équilibreur de charge « interne uniquement », un équilibreur de charge externe supplémentaire doit être ajouté au modèle avec une règle autorisant le trafic sortant pour le port 443. Si vous le souhaitez, vous pouvez bloquer ce port après une installation réussie mais, dans ce cas, le package de mise à niveau doit être distribué aux nœuds, ou le port doit être ouvert pendant une brève période. Une mise à niveau manuelle sera ensuite nécessaire.

Utilisez le Pare-feu Azure avec le [journal de flux NSG](../network-watcher/network-watcher-nsg-flow-logging-overview.md) et l’[analyse du trafic](../network-watcher/traffic-analytics.md) pour suivre des problèmes de verrouillage de sécurité. Le modèle ARM [Service Fabric avec NSG](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG) est un bon exemple pour commencer. 


## <a name="application-networking"></a>Mise en réseau de l’application

* Pour exécuter des charges de travail de conteneur Windows, utilisez le [mode de mise en réseau ouvert](./service-fabric-networking-modes.md#set-up-open-networking-mode) afin de simplifier la communication de service à service.

* Utilisez un proxy inversé tel que [Traefik](https://docs.traefik.io/v1.6/configuration/backends/servicefabric/) ou le [proxy inversé Service Fabric](./service-fabric-reverseproxy.md) pour exposer les ports d’application courants tels que 80 ou 443.

* Pour les conteneurs Windows hébergés sur des machines isolées sous « air gap » qui ne peuvent pas extraire des couches de base du stockage cloud Azure, remplacez le comportement de couche étrangère, en utilisant l’indicateur [--allow-nondistributable-artifacts](/virtualization/windowscontainers/about/faq#how-do-i-make-my-container-images-available-on-air-gapped-machines) dans le démon Docker.

## <a name="next-steps"></a>Étapes suivantes

* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Windows Server : [Création de clusters Service Fabric pour Windows Server](service-fabric-cluster-creation-for-windows-server.md)
* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Linux : [Créer un cluster Linux](service-fabric-cluster-creation-via-portal.md)
* En savoir plus sur les [options de prise en charge de Service Fabric](service-fabric-support.md)

[NSGSetup]: ./media/service-fabric-best-practices/service-fabric-nsg-rules.png
