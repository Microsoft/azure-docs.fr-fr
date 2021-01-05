---
title: Conteneurs serverless dans Azure
description: Le service Azure Container Instances offre le moyen le plus simple et le plus rapide d’exécuter des conteneurs isolés dans Azure, sans avoir à gérer des machines virtuelles ni à adopter un orchestrateur de plus haut niveau.
ms.topic: overview
ms.date: 08/10/2020
ms.custom: seodec18, mvc
ms.openlocfilehash: bd68fab380754eca38eebf3fd52634508f282cf6
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "88121661"
---
# <a name="what-is-azure-container-instances"></a>Présentation d’Azure Container Instances

Les conteneurs deviennent le meilleur moyen pour mettre en package, déployer et gérer des applications cloud. Azure Container Instances propose la façon la plus simple et rapide d’exécuter un conteneur dans Azure, sans avoir à gérer des machines virtuelles et sans avoir à adopter un service de niveau supérieur.

Azure Container Instances est une excellente solution pour les scénarios qui peuvent fonctionner dans des conteneurs isolés, y compris les applications basiques, automatiser des tâches et créer des travaux. Pour les scénarios où vous devez disposer d’orchestration de conteneur complète (détection de service dans plusieurs conteneurs, mise à l’échelle automatique et mises à niveau d’application coordonnées), nous vous recommandons le service [Azure Kubernetes Service (AKS)](../aks/index.yml).

## <a name="fast-startup-times"></a>Temps de démarrage rapide

Les conteneurs offrent des avantages de démarrage conséquents sur les machines virtuelles. Azure Container Instances peut démarrer des conteneurs dans Azure en quelques secondes, sans avoir à configurer ni gérer des machines virtuelles.

Récupérez les images conteneur Linux ou Windows à partir de Docker Hub, d’un [registre de conteneurs Azure](../container-registry/index.yml) privé ou d’un autre registre Docker cloud. Azure Container Instances met en cache plusieurs images de systèmes d’exploitation de base courants, accélérant ainsi le déploiement de vos images d’application personnalisées.

> [!NOTE]
> Actuellement, vous ne pouvez pas déployer une image à partir d’un registre local vers Azure Container Instances.

## <a name="container-access"></a>Accès au conteneur

Azure Container Instances permet d’exposer les groupes de votre conteneur directement sur Internet avec une adresse IP publique et un nom de domaine complet. Lorsque vous créez une instance de conteneur, vous pouvez spécifier une étiquette de nom DNS personnalisée pour que votre application soit accessible à l’emplacement *customlabel*.*azureregion*.azurecontainer.io.

Azure Container Instances prend également en charge l’exécution d’une commande dans un conteneur en cours d’exécution en fournissant un interpréteur de commandes interactif qui facilite le développement et le débogage des applications. L’accès s’effectue via HTTPS, en utilisant TLS pour sécuriser les connexions clientes.

> [!IMPORTANT]
> À partir du 13 janvier 2020, Azure Container Instances exigera l’utilisation de TLS 1.2 pour toutes les connexions sécurisées établies à partir des serveurs et des applications. TLS 1.0 et 1.1 ne seront plus pris en charge.

## <a name="compliant-deployments"></a>Déploiements conformes

### <a name="hypervisor-level-security"></a>Sécurité au niveau de l’hyperviseur

D’un point de vue historique, les conteneurs ont offert l’isolation de dépendance d’application et la gouvernance des ressources, mais n’ont pas été considérés suffisamment renforcés pour une utilisation de plusieurs locataires hostile. Azure Container Instances garantie que votre application se retrouve aussi isolée dans un conteneur que dans une machine virtuelle.

### <a name="customer-data"></a>Données client

Le service ACI stocke le minimum de données client nécessaires au bon fonctionnement de vos groupes de conteneurs. Le stockage de données client dans une seule région n'est actuellement disponible que dans la région Asie Sud-Est (Singapour) de la zone géographique Asie-Pacifique. Pour toutes les autres régions, les données client sont stockées dans [Zone géographique](https://azure.microsoft.com/global-infrastructure/geographies/). Pour en savoir plus, contactez le Support Azure.

## <a name="custom-sizes"></a>Tailles personnalisées

Les conteneurs sont généralement optimisés pour n’exécuter qu’une application, mais les besoins précis de ces applications peuvent grandement varier. Azure Container Instances permet une utilisation optimale en autorisant les spécifications exactes des cœurs d’unité centrale et de la mémoire. Vous payez pour ce dont vous avez besoin et vous êtes facturé immédiatement, pour pouvoir ajuster vos dépenses selon vos besoins.

Pour les travaux nécessitant beaucoup de ressources système, comme le machine learning, Azure Container Instances peut programmer des conteneurs Linux pour utiliser des [ressources GPU](container-instances-gpu.md) (préversion) NVIDIA Tesla.

## <a name="persistent-storage"></a>Stockage persistant

Pour récupérer et rendre persistants des états avec Azure Container Instances, nous proposons le [montage direct des partages Azure Files](./container-instances-volume-azure-files.md) sauvegardés par le service Stockage Azure.

## <a name="linux-and-windows-containers"></a>Conteneurs Windows et Linux

Azure Container Instances peut programmer des conteneurs Windows et Linux avec la même API. Spécifiez simplement le type de système d’exploitation lorsque vous créez vos [groupes de conteneurs](container-instances-container-groups.md).

Certaines fonctionnalités sont actuellement restreintes aux conteneurs Linux :

* Plusieurs conteneurs par groupe de conteneurs
* Montage de volume ([Azure Files](container-instances-volume-azure-files.md), [emptyDir](container-instances-volume-emptydir.md), [GitRepo](container-instances-volume-gitrepo.md), [secret](container-instances-volume-secret.md))
* [Métriques d’utilisation des ressources](container-instances-monitor.md) avec Azure Monitor
* [Déploiement d’un réseau virtuel](container-instances-vnet.md)
* [Ressources GPU](container-instances-gpu.md) (préversion)

Pour les déploiements de conteneurs Windows, utilisez des images basées sur des [images de base Windows](container-instances-faq.md#what-windows-base-os-images-are-supported) communes.

> [!NOTE]
> L’utilisation d’images basées sur Windows Server 2019 dans Azure Container Instances est disponible en version préliminaire.

## <a name="co-scheduled-groups"></a>Groupes co-planifiés

Azure Container Instances prend en charge la planification de [groupes de plusieurs conteneurs](container-instances-container-groups.md) qui partagent un même hôte, réseau local, stockage et cycle de vie. Vous pouvez ainsi combiner votre conteneur d’application principal avec d’autres conteneurs à rôle d’assistance, comme les side-cars de journalisation.

## <a name="virtual-network-deployment"></a>Déploiement d’un réseau virtuel

Azure Container Instances permet le [déploiement d’instances de conteneur dans un réseau virtuel Azure](container-instances-vnet.md). Quand elles sont déployées dans un sous-réseau au sein de votre réseau virtuel, les instances de conteneur peuvent communiquer de manière sécurisée avec d’autres ressources dans le réseau virtuel, y compris celles qui sont en local (par le biais d’une [passerelle VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md) ou d’[ExpressRoute](../expressroute/expressroute-introduction.md)).

## <a name="next-steps"></a>Étapes suivantes

Essayez de déployer un conteneur dans Azure avec une seule commande à l’aide de notre guide de démarrage rapide :

> [!div class="nextstepaction"]
> [Guide de démarrage rapide : créer son premier conteneur dans Azure Container Instances](container-instances-quickstart.md)

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
