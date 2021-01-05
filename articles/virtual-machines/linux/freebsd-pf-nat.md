---
title: Utilisation du filtre de paquets de FreeBSD pour créer un pare-feu dans Azure
description: Découvrez comment déployer un pare-feu NAT à l’aide de FreeBSD PF dans Azure.
author: KylieLiang
ms.service: virtual-machines-linux
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/20/2017
ms.author: kyliel
ms.openlocfilehash: 6a20708c5564075c24eb031a39292b020a2ecc00
ms.sourcegitcommit: 5dbea4631b46d9dde345f14a9b601d980df84897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91371318"
---
# <a name="how-to-use-freebsds-packet-filter-to-create-a-secure-firewall-in-azure"></a>Comment utiliser le filtre de paquets de FreeBSD pour créer un pare-feu sécurisé dans Azure
Cet article explique comment déployer un pare-feu NAT à l’aide du filtre de paquets FreeBSD via le modèle Azure Resource Manager pour un scénario de serveur web courant.

## <a name="what-is-pf"></a>Qu’est-ce que PF ?
PF (filtre de paquets, également écrit pf) est un filtre de paquets avec état sous licence BSD, un élément central des logiciels de pare-feu. PF a évolué rapidement et offre désormais plusieurs avantages par rapport aux autres pare-feu disponibles. La traduction d’adresses réseau (NAT) est intégrée à PF depuis ses débuts. Le planificateur de paquets et la gestion de file d’attente active ont ensuite été intégrés à PF, en intégrant l’ALTQ et en le rendant configurable via la configuration de PF. D’autres fonctionnalités comme pfsync et le CARP pour le basculement et la redondance, authpf pour l’authentification de session et ftp-proxy pour faciliter la protection du difficile protocole FTP sont venues enrichir PF par la suite. En bref, PF est un pare-feu puissant et riche en fonctionnalités. 

## <a name="get-started"></a>Bien démarrer
Si vous souhaitez configurer un pare-feu sécurisé dans le cloud pour vos serveurs web, n’attendons pas une seconde de plus. Vous pouvez également appliquer les scripts utilisés dans ce modèle Azure Resource Manager pour configurer la topologie de votre réseau.
Le modèle Azure Resource Manager configure une machine virtuelle FreeBSD qui effectue la traduction d’adresses réseau/la redirection à l’aide de PF et deux machines virtuelles FreeBSD avec le serveur web Nginx installé et configuré. Outre l’exécution de NAT pour le trafic de sortie des deux serveurs web, la machine virtuelle NAT/de redirection intercepte les requêtes HTTP et les redirige vers les deux serveurs web de manière alternée. Le réseau virtuel utilise l’espace d’adresses IP non routables privé 10.0.0.2/24 et vous pouvez modifier les paramètres du modèle. Le modèle Azure Resource Manager définit également une table de routage pour le réseau virtuel entier. Il s’agit d’une collection d’itinéraires individuels permettant de remplacer les itinéraires par défaut d’Azure basés sur l’adresse IP de destination. 

![Schéma représentant une adresse IP publique sur une instance NAT qui effectue une redirection à l’aide de la méthode de tourniquet (Round Robin) sur deux machines virtuelles principales qui hébergent des serveurs web nginx.](./media/freebsd-pf-nat/pf_topology.jpg)
    
### <a name="deploy-through-azure-cli"></a>Déploiement avec l’interface de ligne de commande Azure
Vous devez avoir la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et être connecté à un compte Azure avec [az login](/cli/azure/reference-index). Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé `myResourceGroup` à l’emplacement `West US`.

```azurecli
az group create --name myResourceGroup --location westus
```

Ensuite, déployez le modèle pf-freebsd-setup avec la commande [az group deployment create](/cli/azure/group/deployment). Téléchargez azuredeploy.parameters.json sous le même chemin d’accès et définissez vos propres valeurs de ressource, comme `adminPassword`, `networkPrefix` et `domainNamePrefix`. 

```azurecli
az group deployment create --resource-group myResourceGroup --name myDeploymentName \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/pf-freebsd-setup/azuredeploy.json \
    --parameters '@azuredeploy.parameters.json' --verbose
```

Après environ cinq minutes, vous obtenez les informations de `"provisioningState": "Succeeded"`. Vous pouvez ensuite vous connecter par SSH à la machine virtuelle principale (NAT) ou accéder à un serveur web Nginx dans un navigateur à l’aide de l’adresse IP publique ou du nom de domaine complet de la machine virtuelle principale (NAT). L’exemple suivant répertorie le nom complet et l’adresse IP publique affectés à la machine virtuelle principale (NAT) dans le groupe de ressources `myResourceGroup`. 

```azurecli
az network public-ip list --resource-group myResourceGroup
```
    
## <a name="next-steps"></a>Étapes suivantes
Voulez-vous configurer votre propre NAT dans Azure ? L’Open Source, gratuit mais puissant ? Alors PF est un bon choix. En utilisant le modèle pf-freebsd-setup, il vous suffit de cinq minutes pour configurer un pare-feu NAT avec équilibrage de charge de tourniquet à l’aide de PF de FreeBSD dans Azure pour un scénario de serveur web courant. 

Si vous souhaitez en savoir plus sur l’offre de FreeBSD dans Azure, reportez-vous à [Présentation de FreeBSD sur Azure](freebsd-intro-on-azure.md).

Si vous souhaitez en savoir plus sur PF, reportez-vous au [Manuel de FreeBSD](https://www.freebsd.org/doc/handbook/firewalls-pf.html) ou au [Guide de l’utilisateur de PF](https://www.freebsd.org/doc/handbook/firewalls-pf.html).
