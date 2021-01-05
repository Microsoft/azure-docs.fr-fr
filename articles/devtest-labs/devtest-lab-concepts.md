---
title: Concepts de DevTest Labs | Microsoft Docs
description: Découvrez les concepts de base de DevTest Labs et comment il peut faciliter la création, la gestion et la surveillance des machines virtuelles Azure
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: b184f835ee99d47ddb843e5b02136299c26bfcdc
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92327856"
---
# <a name="devtest-labs-concepts"></a>Concepts de DevTest Labs
## <a name="overview"></a>Vue d’ensemble
La liste suivante présente les définitions et concepts principaux de DevTest Labs :

## <a name="labs"></a>Laboratoires
Un laboratoire est l’infrastructure qui comprend un groupe de ressources, telles que des machines virtuelles, qui vous permet de mieux gérer ces ressources en spécifiant des limites et des quotas.

## <a name="virtual-machine"></a>Machine virtuelle
Une machine virtuelle Azure est l’un des différents types de [ressources informatiques évolutives et à la demande](/azure/architecture/guide/technology-choices/compute-decision-tree) proposés par Azure. Les machines virtuelles Azure vous apportent la souplesse de la virtualisation sans vous obliger à acheter le matériel qui exécute la machine virtuelle, ni à en assurer la maintenance. Toutefois, vous devez toujours assurer la maintenance de la machine virtuelle en effectuant des tâches comme la configuration, la mise à jour corrective et l’installation des logiciels qui s’exécutent dessus.

L’article [Vue d’ensemble des machines virtuelles Windows dans Azure](../virtual-machines/windows/overview.md) vous informe sur les points à prendre en compte avant de créer une machine virtuelle, sur sa création et sur sa gestion.

## <a name="claimable-vm"></a>Machine virtuelle revendicable
Une machine virtuelle Azure revendicable est une machine virtuelle qui peut être utilisée par n’importe quel utilisateur de laboratoire disposant d’autorisations. Un administrateur de laboratoire peut préparer des machines virtuelles avec des images de base et des artefacts spécifiques, et les enregistrer dans un pool partagé. Un utilisateur de laboratoire peut alors exiger une machine virtuelle fonctionnelle à partir du pool lorsqu’il a besoin d’une machine virtuelle avec cette configuration spécifique.

Une machine virtuelle revendicable n’est pas initialement affectée à un utilisateur spécifique, mais s’affiche dans la liste de tous les utilisateurs sous « Machines virtuelles revendicables ». Une fois qu’une machine virtuelle est revendiquée par un utilisateur, elle est placée dans la zone « Mes machines virtuelles » et n’est plus exigible par un autre utilisateur.

## <a name="environment"></a>Environnement
Dans DevTest Labs, un environnement fait référence à une collection de ressources Azure dans un laboratoire. [Ce billet de blog](./devtest-lab-faq.md#blog-post) explique comment créer des environnements à plusieurs machines virtuelles à partir de vos modèles Azure Resource Manager.

## <a name="base-images"></a>Images de base
Les images de base sont des images de machine virtuelle dont tous les outils et paramètres sont préinstallés et configurés pour créer rapidement une machine virtuelle. Vous pouvez approvisionner une machine virtuelle en sélectionnant une base existante, puis en ajoutant un artefact pour installer l’agent de test. Vous pouvez ensuite enregistrer la machine virtuelle approvisionnée comme base. Il est donc possible d’utiliser la base sans avoir à réinstaller l’agent de test pour chaque approvisionnement de votre machine virtuelle.

## <a name="artifacts"></a>Artefacts
Les artefacts sont utilisés pour déployer et configurer votre application après l’approvisionnement d’une machine virtuelle. Les artefacts peuvent être :

* des outils que vous voulez installer sur la machine virtuelle, tels que des agents, Fiddler, Visual Studio ;
* des actions que vous souhaitez exécuter sur la machine virtuelle, telles que le clonage d’un dépôt ;
* des applications que vous voulez tester.

Les artefacts sont des fichiers JSON [Azure Resource Manager](../azure-resource-manager/management/overview.md) qui contiennent des instructions pour effectuer le déploiement et appliquer la configuration.

## <a name="artifact-repositories"></a>Référentiels d’artefact
Les référentiels d’artefact sont des dépôts Git dans lesquels les artefacts sont archivés. Vous pouvez ajouter des référentiels d’artefacts à plusieurs laboratoires de votre organisation pour les réutiliser et les partager.

## <a name="formulas"></a>Formules
Par rapport aux images de base, les formules offrent en plus un mécanisme pour l’approvisionnement rapide de machines virtuelles. Dans DevTest Labs, une formule est une liste de valeurs de propriétés par défaut utilisée pour créer une machine virtuelle de laboratoire.
Avec les formules, les machines virtuelles ayant le même ensemble de propriétés, comme l’image de base, la taille de machine virtuelle, le réseau virtuel et les artefacts, peuvent être créées sans avoir à spécifier ces propriétés à chaque fois. Quand vous créez une machine virtuelle à partir d’une formule, les valeurs par défaut peuvent être utilisées telles quelles ou modifiées.

## <a name="policies"></a>Stratégies
Les stratégies vous aident à contrôler les coûts dans votre laboratoire. Par exemple, vous pouvez créer une stratégie pour arrêter automatiquement les machines virtuelles selon une planification définie.

## <a name="caps"></a>Limites
Les plafonds constituent un mécanisme permettant de réduire le gaspillage dans votre laboratoire. Par exemple, vous pouvez définir un plafond pour limiter le nombre de machines virtuelles qui peuvent être créées par un utilisateur ou dans un laboratoire.

## <a name="security-levels"></a>Niveaux de sécurité
L’accès à la sécurité est déterminé par le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Pour comprendre comment l’accès est déterminé, vous devez saisir les différences entre une autorisation, un rôle et une étendue tels que définis par Azure RBAC.

* Autorisation - Une autorisation est un accès défini pour une action spécifique (p. ex. accès en lecture à toutes les machines virtuelles).
* Rôle - un rôle est un jeu d’autorisations qui peuvent être regroupées et attribuées à un utilisateur. Par exemple, le rôle *propriétaire de l’abonnement* a accès à toutes les ressources au sein d’un abonnement.
* Étendue - Une étendue est un niveau dans la hiérarchie d’une ressource Azure, comme un groupe de ressources, un simple laboratoire ou l’ensemble de l’abonnement.

Dans l’étendue de DevTest Labs, il existe deux types de rôles pour définir des autorisations utilisateur : propriétaire de laboratoire et utilisateur de laboratoire.

* Propriétaire de laboratoire - un propriétaire de laboratoire a accès à toutes les ressources du laboratoire. Ainsi, le propriétaire d’un laboratoire peut modifier les stratégies, lire et écrire sur toutes les machines virtuelles, modifier le réseau virtuel et ainsi de suite.
* Utilisateur de laboratoire : un utilisateur de laboratoire peut afficher toutes les ressources de laboratoire, telles que les machines virtuelles, les stratégies et les réseaux virtuels, mais il ne peut pas modifier les stratégies ou les machines virtuelles créées par d’autres utilisateurs.

Pour découvrir comment créer des rôles personnalisés dans DevTest Labs, consultez l’article [Accorder des autorisations à des utilisateurs sur des stratégies de laboratoire spécifiques](devtest-lab-grant-user-permissions-to-specific-lab-policies.md).

Étant donné que les étendues sont hiérarchiques, lorsqu’un utilisateur dispose d’autorisations pour une certaine étendue, il reçoit automatiquement ces autorisations pour chaque niveau d’étendue inférieur englobé. Par exemple, si un utilisateur est affecté au rôle de propriétaire d’abonnement, il a accès à toutes les ressources dans un abonnement, notamment à toutes les machines virtuelles, tous les réseaux virtuels et tous les laboratoires. Ainsi, un propriétaire d’abonnement hérite automatiquement du rôle de propriétaire de laboratoire. Toutefois, l’inverse n’est pas vrai. Un propriétaire de laboratoire a accès à un laboratoire, qui est une étendue inférieure au niveau d’abonnement. Par conséquent, un propriétaire de laboratoire ne sera pas en mesure de voir les machines virtuelles ou réseaux virtuels ou toutes les ressources qui sont en dehors du laboratoire.

## <a name="azure-resource-manager-templates"></a>Modèles Microsoft Azure Resource Manager
Tous les concepts abordés dans cet article peuvent être configurés à l’aide de modèles Azure Resource Manager, qui vous permettent de définir l’infrastructure/la configuration de votre solution Windows Azure et de la déployer de façon répétée dans un état cohérent.

[Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md#template-format) décrit la structure d’un modèle Azure Resource Manager et les propriétés disponibles dans les différentes sections d’un modèle.

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Étapes suivantes
[Créer un laboratoire dans DevTest Labs](devtest-lab-create-lab.md)
