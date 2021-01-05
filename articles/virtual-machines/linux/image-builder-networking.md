---
title: Options de mise en réseau du service Azure Image Builder
description: Comprendre les options de mise en réseau lors du déploiement du service Azure VM Image Builder
author: danielsollondon
ms.author: danis
ms.date: 08/10/2020
ms.topic: article
ms.service: virtual-machines
ms.subservice: imaging
ms.openlocfilehash: 531c423e42338b72b41c54466d5bfe8a89cd3c45
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91969635"
---
# <a name="azure-image-builder-service-networking-options"></a>Options de mise en réseau du service Azure Image Builder

Vous devez choisir de déployer Azure Image Builder avec ou sans réseau virtuel existant.

## <a name="deploy-without-specifying-an-existing-vnet"></a>Déployer sans spécifier de réseau virtuel existant

Si vous ne spécifiez pas de réseau virtuel existant, Azure Image Builder crée un réseau virtuel et un sous-réseau dans le groupe de ressources de mise en lots. Une ressource IP publique est utilisée avec un groupe de sécurité réseau pour limiter le trafic entrant vers le service Azure Image Builder. L’adresse IP publique facilite le canal pour les commandes d’Azure image Builder pendant la génération de l’image. Une fois la génération terminée, la machine virtuelle, l’adresse IP publique, les disques et le réseau virtuel sont supprimés. Pour utiliser cette option, ne spécifiez pas de propriétés de réseau virtuel.

## <a name="deploy-using-an-existing-vnet"></a>Déployer à l’aide d’un réseau virtuel existant

Si vous spécifiez un réseau virtuel et un sous-réseau, Azure image Builder déploie la machine virtuelle de build sur le réseau virtuel que vous avez choisi. Vous pouvez accéder aux ressources accessibles sur votre réseau virtuel. Vous pouvez également créer un réseau virtuel en silo qui n’est connecté à aucun autre réseau virtuel. Si vous spécifiez un réseau virtuel, Azure Image Builder n’utilise pas d’adresse IP publique. La communication à partir du service Azure Image Builder avec la machine virtuelle de build utilise la technologie Azure Private Link.

Pour plus d’informations, consultez l’un des exemples suivants :

* [Utiliser Azure Image Builder pour les machines virtuelles Windows autorisant l’accès à un réseau virtuel Azure existant](../windows/image-builder-vnet.md)
* [Utiliser Azure Image Builder pour les machines virtuelles Linux autorisant l’accès à un réseau virtuel Azure existant](image-builder-vnet.md)

### <a name="what-is-azure-private-link"></a>Qu’est-ce que Liaison privée Azure ?

Azure Private Link fournit une connectivité privée entre un réseau virtuel et la plateforme Azure en tant que service (PaaS), appartenant à un client ou à des services partenaires Microsoft. Il simplifie l’architecture réseau et sécurise la connexion entre les points de terminaison dans Azure en éliminant l’exposition des données à l’internet public. Pour plus d'informations, consultez la [documentation Liaison privée](../../private-link/index.yml).

### <a name="required-permissions-for-an-existing-vnet"></a>Autorisations requises pour un réseau virtuel existant

Azure Image Builder requiert des autorisations spécifiques pour utiliser un réseau virtuel existant. Pour plus d’informations, consultez [Configurer les autorisations de service Azure Image Builder à l’aide d’Azure CLI](image-builder-permissions-cli.md) ou [Configurer les autorisations du service Azure Image Builder à l’aide de PowerShell](image-builder-permissions-powershell.md).

### <a name="what-is-deployed-during-an-image-build"></a>Qu’est-ce qui est déployé pendant une génération d’image ?

Si vous utilisez un réseau virtuel existant, Azure Image Builder déploie une machine virtuelle supplémentaire (machine virtuelle proxy) et Azure Load Balancer (ALB) connecté à la Liaison privée. Le trafic du service AIB passe par la liaison privée vers ALB. ALB communique avec la machine virtuelle proxy à l’aide du port 60001 pour le système d’exploitation Linux ou 60000 pour le système d’exploitation Windows. Le proxy transfère les commandes à la machine virtuelle de build à l’aide du port 22 pour le système d’exploitation Linux ou 5986 pour le système d’exploitation Windows.

> [!NOTE]
> Le réseau virtuel doit être dans la même région que celle du service Azure Image Builder.
> 

### <a name="why-deploy-a-proxy-vm"></a>Pourquoi déployer une machine virtuelle proxy ?

Lorsqu’une machine virtuelle sans adresse IP publique se trouve derrière un Load Balancer interne, elle n’a pas accès à Internet. Azure Load Balancer utilisé pour le réseau virtuel est interne. La machine virtuelle proxy autorise l’accès à Internet pour la machine virtuelle de build pendant les builds. Les groupes de sécurité réseau associés peuvent être utilisés pour limiter l’accès à la machine virtuelle de build.

La taille de la machine virtuelle proxy déployée est A1_v2 standard, en plus de la machine virtuelle de build. La machine virtuelle proxy est utilisée pour envoyer des commandes entre le service Azure image Builder et la machine virtuelle de build. Les propriétés de la machine virtuelle proxy ne peuvent pas être modifiées, y compris la taille ou le système d’exploitation. Vous ne pouvez pas modifier les propriétés de la machine virtuelle proxy pour les builds d’images Windows et Linux.

### <a name="image-template-parameters-to-support-vnet"></a>Paramètres du modèle d’image pour la prise en charge du réseau virtuel
```json
"VirtualNetworkConfig": {
        "name": "",
        "subnetName": "",
        "resourceGroupName": ""
        },
```

| Paramètre | Description |
|---------|---------|
| name | (Facultatif) Nom d’un réseau virtuel préexistant. |
| subnetName | Nom du sous-réseau au sein du réseau virtuel spécifié. Doit être spécifié si et seulement si le *nom* est spécifié. |
| resourceGroupName | Nom du groupe de ressources contenant le réseau virtuel spécifié. Doit être spécifié si et seulement si le *nom* est spécifié. |

Le service Liaison privée requiert une adresse IP du réseau virtuel et du sous-réseau spécifiés. Actuellement, Azure ne prend pas en charge les stratégies réseau sur ces adresses IP. Par conséquent, les stratégies réseau doivent être désactivées sur le sous-réseau. Pour plus d'informations, consultez la [documentation Liaison privée](../../private-link/index.yml).

### <a name="checklist-for-using-your-vnet"></a>Liste de contrôle pour l’utilisation de votre réseau virtuel

1. Autoriser Azure Load Balancer (ALB) à communiquer avec la machine virtuelle proxy dans un groupe de sécurité réseau
    * [Exemple AZ CLI](image-builder-vnet.md#add-network-security-group-rule)
    * [Exemple PowerShell](../windows/image-builder-vnet.md#add-network-security-group-rule)
2. Désactiver la stratégie de service privé sur le sous-réseau
    * [Exemple AZ CLI](image-builder-vnet.md#disable-private-service-policy-on-subnet)
    * [Exemple PowerShell](../windows/image-builder-vnet.md#disable-private-service-policy-on-subnet)
3. Autoriser Azure Image Builder à créer un ALB et à ajouter des machines virtuelles au réseau virtuel
    * [Exemple AZ CLI](image-builder-permissions-cli.md#existing-vnet-azure-role-example)
    * [Exemple PowerShell](image-builder-permissions-powershell.md#permission-to-customize-images-on-your-vnets)
4. Autoriser Azure Image Builder à lire/écrire des images sources et à créer des images
    * [Exemple AZ CLI](image-builder-permissions-cli.md#custom-image-azure-role-example)
    * [Exemple PowerShell](image-builder-permissions-powershell.md#custom-image-azure-role-example)
5. Veillez à utiliser le réseau virtuel dans la même région que celle du service Azure Image Builder.


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Vue d’ensemble d’Azure Image Builder](image-builder-overview.md).