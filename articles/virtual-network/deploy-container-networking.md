---
title: Déployer le réseau de conteneurs du réseau virtuel Azure | Microsoft Docs
description: Découvrez comment déployer le plug-in d’interface réseau de conteneur de réseau virtuel Azure pour les clusters Kubernetes.
services: virtual-network
documentationcenter: na
author: aanandr
manager: NarayanAnnamalai
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 9/18/2018
ms.author: aanandr
ms.custom: ''
ms.openlocfilehash: 09a0574666441138c143932e843080e8745f1b40
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87289593"
---
# <a name="deploy-the-azure-virtual-network-container-network-interface-plug-in"></a>Déployer le plug-in CLI Réseau virtuel Azure

Le plug-in CLI Réseau virtuel Azure s’installe sur une machine virtuelle Azure et apporte des fonctionnalités de réseau virtuel au pods Kubernetes et aux conteneurs Docker. Pour en savoir plus sur ce plug-in, consultez [Permettre aux conteneurs d’utiliser les fonctionnalités Réseau virtuel Azure](container-networking-overview.md). Ce plug-in peut de plus être utilisé avec Azure Kubernetes Service (AKS) en choisissant l’option [Réseau avancé](../aks/networking-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json), ce qui place automatiquement les conteneurs AKS dans un réseau virtuel.

## <a name="deploy-plug-in-for-acs-engine-kubernetes-cluster"></a>Déployer le plug-in pour le cluster Kubernetes ACS-Engine

ACS-Engine déploie un cluster Kubernetes avec un modèle Azure Resource Manager. La configuration du cluster est spécifiée dans un fichier JSON qui est passé à l’outil lors de la génération du modèle. Pour en savoir plus sur la liste complète des paramètres de cluster pris en charge et leur description, consultez [Moteur de Microsoft Azure Container Service - Définition du cluster](https://github.com/Azure/acs-engine/blob/master/docs/clusterdefinition.md). Ce plug-in est le plug-in du réseau par défaut des clusters créés à l’aide d’ACS-Engine. Les paramètres de configuration réseau suivants sont importants lors de la configuration du plug-in :

  | Paramètre                              | Description                                                                                                           |
  |--------------------------------------|------------------------------------------------------------------------------------------------------                 |
  | firstConsecutiveStaticIP             | Adresse IP qui est allouée au nœud principal. Ce paramètre est obligatoire.                                     |
  | clusterSubnet under kubernetesConfig | CIDR du sous-réseau de réseau virtuel sur lequel le cluster est déployé et à partir duquel les adresses IP sont allouées aux pods   |
  | vnetSubnetId sous masterProfile     | Spécifie l’ID de ressource Azure Resource Manager du sous-réseau sur lequel le cluster doit être déployé.                    |
  | vnetCidr                             | CIDR du réseau virtuel sur lequel le cluster est déployé.                                                             |
  | max-Pods sous kubeletConfig         | Nombre maximal de pods sur chaque machine virtuelle agent. Pour le plug-in, la valeur par défaut est 30. Vous pouvez spécifier jusqu’à 250 pods  |

### <a name="example-configuration"></a>Exemple de configuration

L’exemple de fichier json ci-dessous s’applique à un cluster présentant les propriétés suivantes :
-   1 nœud principal et 2 nœuds agent 
-   Est déployé sur un sous-réseau nommé *KubeClusterSubnet* (10.0.0.0/20) et sur lequel résident le nœud principal et les nœuds agent.

```json
{
  "apiVersion": "vlabs",
  "properties": {
    "orchestratorProfile": {
      "orchestratorType": "Kubernetes",
      "kubernetesConfig": {
        "clusterSubnet": "10.0.0.0/20" --> Subnet allocated for the cluster
      }
    },
    "masterProfile": {
      "count": 1,
      "dnsPrefix": "ACSKubeMaster",
      "vmSize": "Standard_A2",
      "vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<Vnet Name>/subnets/KubeClusterSubnet",
      "firstConsecutiveStaticIP": "10.0.1.50", --> IP address allocated to the Master node
"vnetCidr": "10.0.0.0/16" --> Virtual network address space
    },
    "agentPoolProfiles": [
      {
        "name": "k8sagentpoo1",
        "count": 2,
        "vmSize": "Standard_A2_v2",
"vnetSubnetId": "/subscriptions/<subscription ID>/resourceGroups/<Resource Group Name>/providers/Microsoft.Network/virtualNetworks/<VNet Name>/subnets/KubeClusterSubnet",
        "availabilityProfile": "AvailabilitySet"
      }
    ],
    "linuxProfile": {
      "adminUsername": "KubeServerAdmin",
      "ssh": {
        "publicKeys": [
          {…}
        ]
      }
    },
    "servicePrincipalProfile": {
      "clientId": "dd438987-aa12-4754-b47d-375811889714",
      "secret": "azure123"
    }
  }
}
```

## <a name="deploy-plug-in-for-a-kubernetes-cluster"></a>Déployer le plug-in pour un cluster Kubernetes

Procédez comme suit pour installer le plug-in sur chaque machine virtuelle Azure dans un cluster Kubernetes :

1. [Téléchargez et installez le plug-in](#download-and-install-the-plug-in).
2. Préallouez un pool d’adresses IP de réseau virtuel sur chaque machine virtuelle à partir duquel des adresses IP seront attribuées aux pods. Chaque machine virtuelle Azure est fournie avec une adresse IP privée de réseau virtuel principale sur chaque interface réseau. Le pool d’adresses IP pour les pods est ajouté en tant qu’adresses secondaires (*ipconfigs*) sur l’interface réseau de machine virtuelle, à l’aide d’une des options suivantes :

   - **CLI** :  [Attribuer plusieurs adresses IP à l’aide d’Azure CLI](virtual-network-multiple-ip-addresses-cli.md)
   - **PowerShell** :  [Attribuer plusieurs adresses IP à l’aide de PowerShell](virtual-network-multiple-ip-addresses-powershell.md)
   - **Portail** :  [Attribuer plusieurs adresses IP à l’aide du portail Azure](virtual-network-multiple-ip-addresses-portal.md)
   - **Modèle Azure Resource Manager** :  [Attribuer plusieurs adresses IP à l’aide de modèles](virtual-network-multiple-ip-addresses-template.md)

   Veillez à ajouter suffisamment d’adresses IP pour tous les pods que vous envisagez d’ajouter à la machine virtuelle.

3. Sélectionnez le plug-in qui fournit le réseau de votre cluster en passant à Kubelet l’option de ligne de commande `–network-plugin=cni` lors de la création du cluster. Par défaut, Kubernetes recherche le plug-in et le fichier de configuration dans les répertoires où ils sont déjà installés.
4. Si vous souhaitez que vos pods accèdent à internet, ajoutez la règle *iptables* suivante sur vos machines virtuelles Linux afin de traduire les adresses sources du trafic internet. Dans l’exemple suivant, la plage IP spécifiée est 10.0.0.0/8.

   ```bash
   iptables -t nat -A POSTROUTING -m iprange ! --dst-range 168.63.129.16 -m
   addrtype ! --dst-type local ! -d 10.0.0.0/8 -j MASQUERADE
   ```

   Les règles traduisent les adresses du trafic qui n’est pas destiné aux plages d’adresses IP spécifiées. Il est supposé que tout le trafic en dehors des plages précédentes correspond à du trafic internet. Vous pouvez choisir de spécifier les plages IP du réseau virtuel de la machine virtuelle, celles des réseaux virtuels appairés et celles des réseaux locaux.

   Les machines virtuelles Windows traduisent automatiquement les adresses sources du trafic dont la destination se trouve en dehors du sous-réseau auquel appartient la machine virtuelle. Il n’est pas possible de spécifier des plages d’adresses IP personnalisées.

Après avoir effectué les étapes précédentes, les pods ajoutés aux machines virtuelles agent Kubernetes reçoivent automatiquement des adresses IP privées à partir du réseau virtuel.

## <a name="deploy-plug-in-for-docker-containers"></a>Déployer le plug-in pour les conteneurs Docker

1. [Téléchargez et installez le plug-in](#download-and-install-the-plug-in).
2. Créez les conteneurs Docker en utilisant la commande suivante :

   ```
   ./docker-run.sh \<container-name\> \<container-namespace\> \<image\>
   ```

Les conteneurs commencent automatiquement à recevoir des adresses IP provenant du pool alloué. Si vous souhaitez équilibrer la charge du trafic vers les conteneurs Docker, ces derniers doivent être placés derrière un équilibreur de charge logiciel et vous devez configurer une sonde d’équilibreur de charge, de la même façon que vous créez une stratégie et des sondes pour une machine virtuelle.

### <a name="cni-network-configuration-file"></a>Fichier de configuration réseau CNI

Le fichier de configuration réseau CNI est décrit au format JSON. Par défaut, il est présent dans `/etc/cni/net.d` pour Linux et dans `c:\cni\netconf` pour Windows. Ce fichier spécifie la configuration du plug-in et il est différent pour Windows et Linux. Le fichier json qui suit est un exemple de fichier de configuration Linux et est suivi d’une explication pour certains paramètres clés. Aucune modification de ce fichier n’est nécessaire :

```json
{
       "cniVersion":"0.3.0",
       "name":"azure",
       "plugins":[
          {
             "type":"azure-vnet",
             "mode":"bridge",
             "bridge":"azure0",
             "ipam":{
                "type":"azure-vnet-ipam"
             }
          },
          {
             "type":"portmap",
             "capabilities":{
                "portMappings":true
             },
             "snat":true
          }
       ]
}
```

#### <a name="settings-explanation"></a>Explication des paramètres

- **cniVersion** : Les plug-ins CNI Réseau virtuel Azure prennent en charge les versions 0.3.0 et 0.3.1 de la  [spécification CNI](https://github.com/containernetworking/cni/blob/master/SPEC.md).
- **nom** : Nom du réseau. Cette propriété peut être définie sur une valeur unique.
- **type** : Nom du plug-in réseau. Définissez-le sur *azure-vnet*.
- **mode** : Mode de fonctionnement. Ce champ est facultatif. Le seul mode pris en charge est « bridge ». Pour plus d’informations, consultez  [Modes de fonctionnement](https://github.com/Azure/azure-container-networking/blob/master/docs/network.md).
- **bridge** : Nom du pont utilisé pour connecter les conteneurs à un réseau virtuel. Ce champ est facultatif. Si ce paramètres est omis, le plug-in choisit automatiquement un nom unique, en fonction de l’index d’interface principal.
- **ipam type** : Nom du plug-in IPAM. Définissez-le toujours sur *azure-vnet-ipam*.

## <a name="download-and-install-the-plug-in"></a>Télécharger et installer le plug-in

Téléchargez le plug-in depuis [GitHub](https://github.com/Azure/azure-container-networking/releases). Téléchargez la dernière version pour la plateforme que vous utilisez :

- **Linux** : [azure-vnet-cni-linux-amd64-\<version no.\>.tgz](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-linux-amd64-v1.0.12-rc3.tgz)
- **Windows** : [azure-vnet-cni-windows-amd64-\<version no.\>.zip](https://github.com/Azure/azure-container-networking/releases/download/v1.0.12-rc3/azure-vnet-cni-windows-amd64-v1.0.12-rc3.zip)

Copiez le script d’installation pour [Linux](https://github.com/Azure/azure-container-networking/blob/master/scripts/install-cni-plugin.sh) ou [Windows](https://github.com/Azure/azure-container-networking/blob/master/scripts/Install-CniPlugin.ps1) sur votre ordinateur. Enregistrez le script dans un répertoire `scripts` sur votre ordinateur et nommez le fichier `install-cni-plugin.sh` pour Linux, ou `install-cni-plugin.ps1` pour Windows. Pour installer le plug-in, exécutez le script approprié pour votre plateforme, en spécifiant la version du plug-in que vous utilisez. Par exemple, vous pouvez spécifier *v1.0.12-rc3* :

   ```bash
   \$scripts/install-cni-plugin.sh [version]
   ```

   ```powershell
   scripts\\ install-cni-plugin.ps1 [version]
   ```

Le script installe le plug-in sous `/opt/cni/bin` pour Linux et `c:\cni\bin` pour Windows. Le plug-in installé est fourni avec un fichier de configuration réseau simple qui fonctionne après l’installation. Il n’a pas besoin d’être mis à jour. Pour en savoir plus sur les paramètres du fichier, consultez [Fichier de configuration réseau CNI](#cni-network-configuration-file).