---
title: En savoir plus sur Azure Image Builder (préversion)
description: Découvrez Azure Image Builder pour les machines virtuelles Azure.
author: danielsollondon
ms.author: danis
ms.date: 05/02/2019
ms.topic: conceptual
ms.service: virtual-machines
ms.subservice: imaging
ms.reviewer: cynthn
ms.openlocfilehash: fb596352011bcce3130d22b7277444bc45679f4c
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96839692"
---
# <a name="preview-azure-image-builder-overview"></a>Aperçu : Vue d’ensemble du Générateur d’images Azure

Des images de machine virtuelle standardisées permettent aux organisations de migrer vers le cloud en veillant à la cohérence des déploiements. Les images incluent généralement des paramètres de sécurité et de configuration prédéfinis et les logiciels nécessaires. La mise en place de votre propre pipeline d’acquisition d’image nécessite du temps, une infrastructure et une configuration. En revanche, le Générateur d’images de machine virtuelle Azure génère une simple configuration décrivant votre image et l’envoie au service qui la produit et la distribue.
 
Le Générateur d’images de machine virtuelle Azure vous permet de partir d’une image de la Place de marché Azure basée sur Windows ou Linux, d’images personnalisées existantes ou d’une image ISO Red Hat Enterprise Linux (RHEL), et d’y ajouter vos propres personnalisations. Le Générateur d’images s’appuyant sur [HashiCorp Packer](https://packer.io/), vous pouvez également importer vos scripts fournisseurs de shell Packer existants. Vous pouvez également spécifier où vous souhaitez que vos images soient hébergées dans la [Galerie d’images partagées Azure](shared-image-galleries.md), en tant qu’images managées ou disque dur virtuel.

> [!IMPORTANT]
> Le Générateur d’images Azure est actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="preview-features"></a>Fonctionnalités de préversion

La préversion prend en charge les fonctionnalités suivantes :

- Création d’images de base finales, qui incluent vos configurations minimales de sécurité et d’entreprise, et permettent aux services de les personnaliser en fonction de leurs besoins.
- En corrigeant des images existantes, le Générateur d’images vous permet de corriger en permanence des images personnalisées.
- Connectez Image Builder à vos réseaux virtuels existants, afin de pouvoir vous connecter à des serveurs de configuration existants (DSC, Chef, Puppet, etc.), à des partages de fichiers ou à tout autre serveur/service routable.
- L’intégration avec la Galerie d’images partagées Azure vous permet de distribuer, de contrôler les versions, et de mettre à l’échelle des images globalement, et offre un système de gestion des images.
- L’intégration avec des pipelines de génération d’images existants vous permet d’appeler simplement le Générateur d’images à partir de votre pipeline ou d’utiliser la simple tâche Azure DevOps de prévisualisation du Générateur d’images.
- Migrez un pipeline de personnalisation d’image existant vers Azure. Utilisez vos scripts, commandes et processus existants pour personnaliser des images.
- Création d’images au format VHD pour prendre en charge Azure Stack.
 

## <a name="regions"></a>Régions
Le service Générateur d’images Azure sera disponible en préversion dans les régions suivantes. Des images peuvent être distribuées en dehors de ces régions.
- USA Est
- USA Est 2
- Centre-USA Ouest
- USA Ouest
- USA Ouest 2
- Europe Nord
- Europe Ouest

## <a name="os-support"></a>Prise en charge du système d’exploitation
Le Générateur d’images Azure prendra en charge les images de système d’exploitation de base de la Place de marché Azure :
- Ubuntu 18.04
- Ubuntu 16.04
- RHEL 7.6, 7.7
- CentOS 7.6, 7.7
- SLES 12 SP4
- SLES 15, SLES 15 SP1
- Windows 10 RS5 Enterprise/Enterprise multisession/Professional
- Windows 2016
- Windows 2019

Les fichiers ISO RHEL ne sont plus pris en charge.

## <a name="how-it-works"></a>Fonctionnement

Le Générateur d’images Azure est un service Azure entièrement managé accessible à un fournisseur de ressources Azure. Le processus du Générateur d’images Azure comporte trois parties principales, le sourçage, la personnalisation et la distribution, représentés dans un modèle. Le diagramme ci-dessous présente les composants avec certaines de leurs propriétés. 
 

**Processus du Générateur d’images** 

![Schéma conceptuel du processus du Générateur d’images Azure](./media/image-builder-overview/image-builder-process.png)

1. Créez le modèle d’image en tant que fichier .json. Ce fichier .json contient des informations sur la source, les personnalisations et la distribution de l’image. Plusieurs exemples son disponibles dans le [référentiel GitHub du Générateur d’images Azure](https://github.com/danielsollondon/azvmimagebuilder/tree/master/quickquickstarts).
1. Envoyez-le au service. Cela a pour effet de créer un artefact de modèle d’image dans le groupe de ressources que vous spécifiez. En arrière-plan, le Générateur d’images télécharge l’image ou l’ISO source et les scripts en fonction des besoins. Celles-ci sont stockées dans un groupe de ressources distinct créé automatiquement dans votre abonnement, au format suivant : IT_\<DestinationResourceGroup>_\<TemplateName>. 
1. Une fois le modèle d’image créé, vous pouvez générer l’image. Dans l’image d’arrière-plan, le générateur utilise le modèle et les fichiers sources pour créer une machine virtuelle (taille par défaut : Standard_D1_v2), le réseau, l’IP publique, le groupe de sécurité réseau et le stockage dans le groupe de ressources IT_\<DestinationResourceGroup>_\<TemplateName>.
1. Dans le cadre de la création d’image, le Générateur d’images distribue l’image en fonction du modèle, puis supprime les ressources supplémentaires dans le groupe de ressources IT_\<DestinationResourceGroup>_\<TemplateName> créé pour le processus.


## <a name="permissions"></a>Autorisations
Lorsque vous vous inscrivez à AIB, cela permet au service AIB de créer, de gérer et de supprimer un groupe de ressources intermédiaire (IT_ *), et d’y ajouter des ressources requise pour la génération d’image. Cette opération est effectuée par un nom de principal du service AIB rendu disponible dans votre abonnement lors d’une inscription réussie.

Pour permettre à Azure VM Image Builder de distribuer des images aux images gérées ou à une galerie d’images partagées, vous devez créer une identité attribuée par l’utilisateur Azure qui dispose des autorisations nécessaires pour lire et écrire des images. Si vous accédez à Azure Storage, cela nécessite des autorisations pour lire des conteneurs privés.

Au départ, vous devez suivre la documentation [créer une identité gérée attribuée par l’utilisateur Azure](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-cli.md) pour savoir comment créer une identité.

Une fois que vous disposez de l'identité, vous devez lui accorder des autorisations. Pour ce faire, vous pouvez utiliser une définition de rôle personnalisée Azure, puis attribuer l'identité gérée attribuée par l'utilisateur pour utiliser la définition de rôle personnalisée.

Les autorisations sont expliquées plus en détail [ici](https://github.com/danielsollondon/azvmimagebuilder/blob/master/aibPermissions.md#azure-vm-image-builder-permissions-explained-and-requirements), et les exemples montrent comment cela est implémenté.

> [!Note]
> Précédemment avec AIB, vous utiliseriez le SPN AIB et lui accordiez des autorisations d’accès aux groupes de ressources d’image. Nous nous éloignons de ce modèle afin de permettre des fonctionnalités futures. À compter du 26 mai 2020, image Builder n’acceptera plus les modèles qui n’ont pas d’identité attribuée par l’utilisateur, et les modèles existants devront être renvoyés au service avec une [identité d’utilisateur](./linux/image-builder-json.md). Les exemples ici montrent déjà comment vous pouvez créer une identité attribuée par l’utilisateur et l’ajouter à un modèle. Pour plus d’informations, consultez cette [documentation](https://github.com/danielsollondon/azvmimagebuilder#service-updates-and-latest-release-information) concernant ce changement et les mises à jour de publications.

## <a name="costs"></a>Coûts
Nous vous facturons des coûts de calcul, de mise en réseau et de stockage lors de la création, de la génération et du stockage d’images avec le Générateur d’images Azure. Ces coûts sont similaires à ceux associés à la création manuelle d’images personnalisées. Pour les ressources, vous êtes facturé conformément aux tarifs Azure qui vous sont appliqués. 

Pendant le processus de création d’image, les fichiers sont téléchargés et stockés dans le groupe de ressources `IT_<DestinationResourceGroup>_<TemplateName>`, ce qui implique des coûts de stockage modiques. Si vous ne souhaitez pas conserver ces ressources, supprimez le **modèle d’image** une fois l’image générée.
 
Le Générateur d’images crée une machine virtuelle de taille D1v2, ainsi que le stockage et la mise en réseau nécessaires pour la machine virtuelle. Ces ressources durent pendant le temps d’exécution du processus de génération, puis sont supprimées une fois l’image créée par le Générateur d’images. 
 
Le Générateur d’images Azure distribue l’image aux régions choisies, ce qui peut occasionner des frais de sortie de réseau.

## <a name="hyper-v-generation"></a>Hyper-V Génération
Le générateur Image Builder prend en charge uniquement en mode natif la création d’images Hyper-V de génération 1 (Gen1) dans Azure Shared Image Gallery (SIG) ou Managed Image. Si vous souhaitez créer des images de génération 2 (Gen2), vous devez utiliser une image source Gen2 et la distribuer sur le disque dur virtuel. Après quoi, vous devez créer une image managée à partir du disque dur virtuel et l’injecter dans le SIG en tant qu’image Gen2.
 
## <a name="next-steps"></a>Étapes suivantes 
 
Pour essayer le Générateur d’images Azure, voir les articles sur la génération d’images [Linux](./linux/image-builder.md) ou [Windows](./windows/image-builder.md).

