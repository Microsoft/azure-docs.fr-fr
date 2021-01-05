---
title: Déploiements Gestionnaire des ressources et classiques
description: Décrit les différences entre le modèle de déploiement de Resource Manager et le modèle de déploiement classique (ou de gestion des services).
ms.topic: conceptual
ms.date: 02/06/2020
ms.openlocfilehash: da260db84de0cc8bd092eb95491b7cc38df93a12
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185842"
---
# <a name="azure-resource-manager-vs-classic-deployment-understand-deployment-models-and-the-state-of-your-resources"></a>Déploiement Azure Resource Manager et déploiement Classic : comprendre les modèles de déploiement et l’état de vos ressources

> [!NOTE]
> Les informations fournies dans cet article sont uniquement utilisées lorsque vous effectuez une migration d’un déploiement classique à un déploiement Azure Resource Manager.

Dans cet article, vous découvrez les modèles de déploiement classique et Azure Resource Manager. Resource Manager et les modèles de déploiement classique représentent deux façons différentes de déployer et de gérer vos solutions Azure. Vous les utilisez avec deux ensembles différents d’API et les ressources déployées peuvent contenir des différences importantes. Les deux modèles ne sont pas compatibles entre eux. Cet article décrit ces différences.

Pour simplifier le déploiement et la gestion des ressources, Microsoft recommande l’utilisation de Resource Manager pour les nouvelles ressources. Microsoft recommande, si possible, un nouveau déploiement des ressources existantes via Resource Manager.

Si vous découvrez Resource Manager, nous vous recommandons de commencer par passer en revue la terminologie définie dans l’article [Vue d’ensemble d’Azure Resource Manager](overview.md).

## <a name="history-of-the-deployment-models"></a>Historique des modèles de déploiement

À l’origine, Azure fournissait uniquement le modèle de déploiement classique. Dans ce modèle, chaque ressource existait indépendamment. Il n’existait aucun moyen de regrouper des ressources associées. Au lieu de cela, il fallait suivre manuellement les ressources qui composaient la solution ou l’application manuellement, et veiller à les gérer selon une approche coordonnée. Pour déployer une solution, il fallait soit créer chaque ressource individuellement via le portail, soit créer un script qui déployait toutes les ressources dans le bon ordre. Pour supprimer une solution, il fallait supprimer chaque ressource individuellement. Il était difficile d’appliquer et de mettre à jour des stratégies de contrôle d’accès pour des ressources associées. Enfin, il n’était pas possible d’appliquer des balises aux ressources afin de les étiqueter à l’aide de termes facilitant la supervision des ressources et la gestion de la facturation.

En 2014, Azure a introduit le modèle de déploiement Resource Manager, et avec celui-ci le concept de groupe de ressources. Un groupe de ressources est un conteneur de ressources qui partagent un cycle de vie commun. Le modèle de déploiement Resource Manager présente plusieurs avantages :

* Vous pouvez déployer, gérer et surveiller tous les services de votre solution comme un groupe, plutôt que de gérer ces services individuellement.
* Vous pouvez déployer votre solution à plusieurs reprises tout au long de son cycle de vie et avoir ainsi l’assurance que vos ressources présentent un état cohérent lors de leur déploiement.
* Vous pouvez appliquer le contrôle d’accès à toutes les ressources de votre groupe de ressources, et ces stratégies sont appliquées automatiquement lorsque de nouvelles ressources sont ajoutées au groupe de ressources.
* Vous pouvez appliquer des balises aux ressources pour organiser logiquement toutes les ressources de votre abonnement.
* Vous pouvez utiliser JavaScript Objet Notation (JSON) pour définir l’infrastructure de votre solution. Le fichier JSON est connu sous le nom de modèle Resource Manager.
* Vous pouvez définir les dépendances entre les ressources afin de les déployer dans le bon ordre.

Quand Resource Manager a été ajouté, toutes les ressources ont été rétroactivement ajoutées aux groupes de ressources par défaut. Si vous créez une ressource via un déploiement classique maintenant, la ressource est automatiquement créée dans un groupe de ressources par défaut pour ce service, même si vous n’avez pas spécifié ce groupe de ressources au moment du déploiement. Toutefois, le simple fait d’exister au sein d’un groupe de ressources ne signifie pas que la ressource a été convertie au modèle Resource Manager.

## <a name="understand-support-for-the-models"></a>Présentation de la prise en charge des modèles

Il existe trois scénarios à connaître :

1. Azure Cloud Services ne prend pas en charge le modèle de déploiement Resource Manager.
2. Les machines virtuelles, les comptes de stockage et les réseaux virtuels prennent en charge les modèles de déploiement classique et Resource Manager.
3. Tous les autres services Azure prennent en charge Resource Manager.

Pour les machines virtuelles, les comptes de stockage et les réseaux virtuels, si la ressource a été créée via un déploiement classique, vous devez continuer à l’utiliser via des opérations classiques. Si la machine virtuelle, le compte de stockage ou le réseau virtuel a été créé via un déploiement Resource Manager, vous devez continuer à utiliser des opérations de Resource Manager. Cette distinction peut être déroutante lorsque votre abonnement contient un mélange de ressources créées via un déploiement Resource Manager et un déploiement classique. Cette combinaison de ressources peut générer des résultats inattendus, car les ressources ne prennent pas en charge les mêmes opérations.

Dans certains cas, une commande Resource Manager peut récupérer des informations sur une ressource créée via un déploiement classique ou effectuer une tâche d’administration, telle que le déplacement d’une ressource classique vers un autre groupe de ressources, Ces exemples ne doivent cependant pas donner l’impression que le type prend en charge les opérations de Resource Manager. Par exemple, supposons que vous disposiez d’un groupe de ressources qui contient une machine virtuelle, créée via un déploiement classique. Si vous exécutez la commande PowerShell Resource Manager suivante :

```powershell
Get-AzResource -ResourceGroupName ExampleGroup -ResourceType Microsoft.ClassicCompute/virtualMachines
```

Elle renvoie la machine virtuelle :

```powershell
Name              : ExampleClassicVM
ResourceId        : /subscriptions/{guid}/resourceGroups/ExampleGroup/providers/Microsoft.ClassicCompute/virtualMachines/ExampleClassicVM
ResourceName      : ExampleClassicVM
ResourceType      : Microsoft.ClassicCompute/virtualMachines
ResourceGroupName : ExampleGroup
Location          : westus
SubscriptionId    : {guid}
```

Toutefois, l’applet de commande Resource Manager **Get-AzVM** retourne uniquement les machines virtuelles déployées par le biais de Resource Manager. La commande suivante ne renvoie pas la machine virtuelle créée via un déploiement classique.

```powershell
Get-AzVM -ResourceGroupName ExampleGroup
```

Seules les ressources créées via Resource Manager prennent en charge les balises. Vous ne pouvez pas appliquer de balises à des ressources classiques.

## <a name="changes-for-compute-network-and-storage"></a>Modifications en termes de calcul, de réseau et de stockage

Le diagramme suivant affiche les ressources de calcul, de réseau et de stockage déployées via Resource Manager.

![Architecture Resource Manager](./media/deployment-models/arm_arch3.png)

SRP : Fournisseur de ressources de stockage, CRP : Fournisseur de ressources de calcul, NRP : Fournisseur de ressources réseau

Notez les relations suivantes entre les ressources :

* Toutes les ressources existent au sein d’un groupe de ressources.
* La machine virtuelle dépend d’un compte de stockage spécifique défini dans le fournisseur de ressources de stockage pour stocker ses disques dans le Stockage Blob (obligatoire).
* La machine virtuelle fait référence à une carte d’interface réseau spécifique définie dans le fournisseur de ressources réseau (obligatoire) et à un groupe à haute disponibilité défini dans le fournisseur de ressources de calcul (facultatif).
* La carte d’interface réseau fait référence à l’adresse IP attribuée à la machine virtuelle (obligatoire), au sous-réseau du réseau virtuel de la machine virtuelle (obligatoire) et à un groupe de sécurité réseau (facultatif).
* Le sous-réseau présent au sein d’un réseau virtuel fait référence à un groupe de sécurité réseau (facultatif).
* L’instance d’équilibreur de charge fait référence au pool principal d’adresses IP qui inclut la carte d’interface réseau d’une machine virtuelle (facultatif) et fait référence à l’adresse IP publique ou privée d’un équilibreur de charge (facultatif).

Voici les composants et leurs relations concernant le déploiement classique :

![architecture classique](./media/deployment-models/arm_arch1.png)

La solution classique pour l’hébergement d’une machine virtuelle comprend les éléments suivants :

* Un service cloud nécessaire qui agit en tant que conteneur pour héberger des machines virtuelles (calcul). Les machines virtuelles sont automatiquement approvisionnées avec une carte d’interface réseau et une adresse IP attribuée par Azure. En outre, le service cloud présente une instance d’équilibrage de charge externe, une adresse IP publique et des points de terminaison par défaut permettant un accès à distance et un trafic PowerShell distant pour les machines virtuelles Windows et un trafic Secure Shell (SSH) pour les machines virtuelles Linux.
* Un compte de stockage obligatoire qui stocke les disques durs virtuels d’une machine virtuelle, y compris le système d’exploitation ainsi que les disques de données temporaires et supplémentaires (stockage).
* Un réseau virtuel facultatif qui agit en tant que conteneur supplémentaire dans lequel vous pouvez créer une structure de sous-réseaux et choisir le sous-réseau sur lequel se trouve la machine virtuelle (réseau).

Le tableau suivant décrit les modifications dans l’interaction des fournisseurs de ressources de calcul, de réseau et de stockage :

| Élément | Classique | Gestionnaire de ressources |
| --- | --- | --- |
| Service cloud pour machines virtuelles |Le service cloud était un conteneur pour les machines virtuelles nécessitant la disponibilité de la plateforme et l’équilibrage de charge. |Le service cloud n’est plus un objet nécessaire à la création des machines virtuelles avec le nouveau modèle. |
| Virtual Network |Un réseau virtuel est facultatif pour la machine virtuelle. S’il est inclus, le réseau virtuel ne peut pas être déployé avec Resource Manager. |La machine virtuelle nécessite un réseau virtuel déployé avec Resource Manager. |
| Comptes de stockage |La machine virtuelle nécessite un compte de stockage qui stocke les disques durs virtuels pour le système d’exploitation, ainsi que les disques de données temporaires et supplémentaires. |La machine virtuelle nécessite un compte de stockage pour stocker ses disques dans le Stockage Blob. |
| Groupes à haute disponibilité |La disponibilité de la plateforme était indiquée en configurant le même « AvailabilitySetName » sur les machines virtuelles. Le nombre maximal de domaines d’erreur était de 2. |Le groupe à haute disponibilité est une ressource exposée par le fournisseur Microsoft.Compute. Les machines virtuelles qui nécessitent une haute disponibilité doivent faire partie du groupe à haute disponibilité. Le nombre maximal de domaines d’erreur est maintenant de 3. |
| Groupe d'affinités |Les groupes d’affinités étaient nécessaires pour créer des réseaux virtuels. Toutefois, avec l’introduction des réseaux virtuels régionaux, ils ne sont plus nécessaires. |Pour simplifier, le concept de groupe d’affinités n’existe pas dans les API exposées par le Gestionnaire de ressources Azure. |
| Équilibrage de la charge. |La création d’un service cloud fournit un équilibrage de la charge implicite pour les machines virtuelles déployées. |L’équilibrage de la charge est une ressource exposée par le fournisseur Microsoft.Network. La principale interface réseau des machines virtuelles dont la charge doit être équilibrée doit faire référence à l’équilibrage de la charge. Ces éléments d’équilibrage de la charge peuvent être internes ou externes. Une instance d’équilibreur de charge fait référence au pool principal d’adresses IP qui comporte la carte d’interface réseau d’une machine virtuelle (facultatif) et fait référence à l’adresse IP publique ou privée d’un équilibreur de charge (facultative). |
| Adresse IP virtuelle |Cloud Services obtient une adresse IP virtuelle par défaut lorsqu’une machine virtuelle est ajoutée à un service cloud. L’adresse IP virtuelle est l’adresse associée à l’équilibrage de charge implicite. |L’adresse IP est une ressource exposée par le fournisseur Microsoft.Network. L’adresse IP publique peut être statique (réservée) ou dynamique. Les adresses IP publiques dynamiques peuvent être attribuées à un équilibreur de charge. Les adresses IP publiques peuvent être sécurisées à l’aide de groupes de sécurité. |
| Adresses IP réservées |Vous pouvez réserver une adresse IP dans Azure et l’associer à un service cloud pour vous assurer que l’adresse IP est permanente. |L’adresse IP publique peut être créée en mode « Statique ». Elle offre les mêmes fonctionnalités qu’une « Adresse IP réservée ». |
| Adresse IP publique par machine virtuelle |Les adresses IP publiques peuvent également être associées directement à une machine virtuelle. |L’adresse IP est une ressource exposée par le fournisseur Microsoft.Network. L’adresse IP publique peut être statique (réservée) ou dynamique. |
| Points de terminaison |Les points de terminaison d’entrée doivent être configurés sur la machine virtuelle pour pouvoir ouvrir la connectivité pour certains ports. Un des modes les plus communs de connexion aux machines virtuelles se fait en définissant des points de terminaison d’entrée. |Les règles NAT entrantes peuvent être configurées sur l’équilibrage de la charge pour obtenir les mêmes fonctionnalités d’activation des points de terminaison sur certains ports spécifiques pour se connecter aux machines virtuelles. |
| Nom DNS |Les services cloud obtiennent un nom DNS global unique et implicite. Par exemple : `mycoffeeshop.cloudapp.net`. |Les noms DNS sont des paramètres facultatifs qui peuvent être spécifiés sur une ressource d’adresse IP publique. Le nom de domaine complet se présente au format suivant : `<domainlabel>.<region>.cloudapp.azure.com`. |
| Interfaces réseau |L’interface réseau principale et secondaire et ses propriétés ont été définies en tant que configuration du réseau d’une machine virtuelle. |L’interface réseau est une ressource exposée par le fournisseur Microsoft.Network. Le cycle de vie de l’interface réseau n’est pas lié à une machine virtuelle. L’interface réseau fait référence à l’adresse IP attribuée à la machine virtuelle (obligatoire), au sous-réseau du réseau virtuel de la machine virtuelle (obligatoire) et à un groupe de sécurité réseau (facultatif). |

Pour en savoir plus sur la connexion de réseaux virtuels à partir de modèles de déploiement différents, consultez [Connecter des réseaux virtuels utilisant des modèles de déploiement différents dans le portail](../../vpn-gateway/vpn-gateway-connect-different-deployment-models-portal.md).

## <a name="migrate-from-classic-to-resource-manager"></a>Migration à partir de déploiements classiques vers des déploiements Resource Manager

Si vous êtes prêt à migrer vos ressources d’un déploiement classique vers un déploiement Resource Manager, consultez les articles suivants :

1. [Étude technique approfondie de la migration prise en charge par la plateforme de ressources Classic vers Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-deep-dive.md)
2. [Migration prise en charge par la plateforme de ressources IaaS Classic vers Azure Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md)
3. [Migration de ressources IaaS d’un environnement Classic vers Azure Resource Manager à l’aide d’Azure PowerShell](../../virtual-machines/migration-classic-resource-manager-ps.md)
4. [Migration de ressources IaaS d’un environnement Classic vers Azure Resource Manager à l’aide de l’interface de ligne de commande Azure](../../virtual-machines/migration-classic-resource-manager-cli.md)

## <a name="frequently-asked-questions"></a>Forum aux questions

**Puis-je créer une machine virtuelle à l’aide de Resource Manager et la déployer dans un réseau virtuel créé à l’aide d’un déploiement classique ?**

Cette configuration n’est pas prise en charge. Vous ne pouvez pas utiliser Resource Manager pour déployer une machine virtuelle dans un réseau virtuel créé à l’aide d’un déploiement classique.

**Puis-je créer une machine virtuelle à l’aide de Resource Manager à partir d’une image utilisateur créée à l’aide du modèle de déploiement classique ?**

Cette configuration n’est pas prise en charge. Toutefois, vous pouvez copier les fichiers de disque dur virtuel à partir d’un compte de stockage créé à l’aide du modèle de déploiement classique, et les ajouter à un nouveau compte créé à l’aide de Resource Manager.

**Quel est l’impact sur le quota pour mon abonnement ?**

Les quotas pour les machines virtuelles, les réseaux virtuels et les comptes de stockage créés via Azure Resource Manager sont distincts des autres quotas. Chaque abonnement obtient des quotas pour créer les ressources avec les nouvelles API. Plus d’informations sur les quotas supplémentaires sont disponibles [ici](../../azure-resource-manager/management/azure-subscription-service-limits.md).

**Puis-je continuer à utiliser mes scripts automatisés pour approvisionner des machines virtuelles, des réseaux virtuels et des comptes de stockage avec les API Resource Manager ?**

Toutes les automatisations et tous les scripts que vous avez créés continueront à fonctionner pour les machines virtuelles et les réseaux virtuels existants créés en mode Gestion des services Azure. Cependant, les scripts doivent être mis à jour afin qu’ils utilisent le nouveau schéma pour créer les mêmes ressources via le mode Resource Manager.

**Où puis-je trouver des exemples de modèles Azure Resource Manager ?**

Vous trouverez un ensemble complet de modèles de démarrage sur [Modèles de démarrage rapide Azure Resource Manager](https://azure.microsoft.com/documentation/templates/).

## <a name="next-steps"></a>Étapes suivantes

* Pour connaître les commandes permettant de déployer un modèle, consultez [Déploiement d’une application avec un modèle Azure Resource Manager](../templates/deploy-powershell.md).