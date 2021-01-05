---
title: Partager des images de machine virtuelle avec les galeries partagées
description: Découvrez comment utiliser les galeries d’images partagées pour partager des images de machine virtuelle Linux au sein de votre organisation.
author: axayjo
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 10/14/2020
ms.author: akjosh
ms.reviewer: cynthn
ms.openlocfilehash: 4156308fa58ea1f02e6e3f2c5174839dc1cd381c
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96839671"
---
# <a name="shared-image-galleries-overview"></a>Vue d’ensemble des galeries d’images partagées

Shared Image Gallery est un service qui vous permet de structurer et d’organiser vos images. Les galeries d’images partagées proposent les éléments suivants :

- Réplication globale des images.
- la gestion de versions et le regroupement d’images pour faciliter la gestion ;
- des images hautement disponibles avec des comptes ZRS (Zone Redundant Storage) dans les régions qui prennent en charge les zones de disponibilité Azure Le stockage redondant interzone (ZRS) offre une meilleure résilience en cas de défaillances de zones.
- Prise en charge du stockage Premium (Premium_LRS).
- le partage entre différents abonnements, voire entre locataires Active Directory, à l’aide du contrôle d’accès en fonction du rôle (RBAC) ;
- la mise à l’échelle de vos déploiements avec des réplicas d’image dans chaque région.

Celle-ci vous permet de partager vos images avec différents utilisateurs, principaux de service ou groupes Active Directory au sein de votre organisation. Il est possible de répliquer des images partagées dans plusieurs régions, pour une mise à l’échelle plus rapide de vos déploiements.

Une image est une copie d’une machine virtuelle complète (dont tous les disques de données attachés) ou juste du disque de système d’exploitation, selon la façon dont elle est créée. Quand vous créez une machine virtuelle à partir de l’image, une copie des disques durs virtuels dans l’image est utilisée pour créer les disques de la nouvelle machine virtuelle. L’image reste dans le stockage et peut être utilisée sans limite pour créer des machines virtuelles.

Si vous avez un grand nombre d’images à gérer et que vous voulez qu’elles soient disponibles dans toute votre entreprise, vous pouvez utiliser une galerie d’images partagées comme référentiel. 

La fonctionnalité Galerie d’images partagées a plusieurs types de ressources :

| Ressource | Description|
|----------|------------|
| **Source d’image** | Il s’agit d’une ressource qui peut être utilisée pour créer une **version d’image** dans une galerie d’images. Une source d’image peut être une machine virtuelle Azure existante qui est [généralisée ou spécialisée](#generalized-and-specialized-images), une image managée, une capture instantanée, un disque dur virtuel ou une version d’image dans une autre galerie d’images. |
| **Galerie d’images** | Tout comme la Place de marché Azure, une **galerie d’images** est un dépôt permettant de gérer et partager des images, mais vous contrôlez les utilisateurs qui y ont accès. |
| **Définition d'image** | Les définitions d’image sont créées dans une galerie et contiennent des informations sur l’image et sur les exigences relatives à son utilisation en interne. Ces informations indiquent, par exemple, si l’image est Windows ou Linux, et comprennent les notes de publication et les exigences de mémoire maximale et minimale. Il s’agit d’une définition de type d’image. |
| **Version de l’image** | Une **version d’image** est ce qui vous permet de créer une machine virtuelle quand vous utilisez une galerie. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Tout comme une image managée, quand vous utilisez une **version d’image** pour créer une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois. |

<br>

![Graphique montrant comment vous pouvez avoir plusieurs versions d’une image dans la galerie](./media/shared-image-galleries/shared-image-gallery.png)

## <a name="image-definitions"></a>Définitions d’image

Une définition d’image est un regroupement logique des versions d’une image. La définition d’image contient des informations sur la raison pour laquelle l’image a été créée, le système d’exploitation concerné et d’autres informations sur l’utilisation de l’image. Une définition d’image est similaire à un plan, qui inclut l’ensemble des détails concernant la création d’une image spécifique. Vous ne déployez pas une machine virtuelle à partir d’une définition d’image, mais à partir des versions de l’image créées sur la base de la définition.

Il existe trois paramètres pour chaque définition d’image, qui sont utilisés les uns avec les autres : **Publisher**, **Offre** et **SKU**. Ils permettent de rechercher une définition d’image spécifique. Des versions d'image peuvent partager une ou deux de ces valeurs, mais pas les trois.  Par exemple, voici trois définitions d'image et leurs valeurs :

|Définition de l’image|Serveur de publication|Offre|Sku|
|---|---|---|---|
|myImage1|Contoso|Finances|Backend|
|myImage2|Contoso|Finances|Serveur frontal|
|myImage3|Test|Finances|Serveur frontal|

Ces trois définitions présentent des ensembles de valeurs uniques. Le format ressemble à celui qui est utilisé pour spécifier un éditeur (publisher), une offre et une SKU pour des [images Azure Marketplace](./windows/cli-ps-findimage.md) dans Azure PowerShell, afin d’obtenir la toute dernière version d’une image d’une Place de marché Microsoft Azure. Chaque définition d’image doit disposer d’un ensemble unique de ces valeurs.

Les paramètres suivants déterminent les types de versions d’images qu’elles peuvent contenir :

- État de système d’exploitation : vous pouvez définir l’état du système d’exploitation sur la valeur [Généralisée ou Spécialisée](#generalized-and-specialized-images). Ce champ doit obligatoirement être renseigné.
- Système d’exploitation : Windows ou Linux. Ce champ doit obligatoirement être renseigné.
-   Génération d’Hyper-V : spécifiez si l’image a été créée à partir d’un disque dur virtuel Hyper-V de génération 1 ou de [génération 2](generation-2.md). Par défaut, il s’agit de génération 1.


Voici d’autres paramètres qui peuvent être configurés sur votre définition d’image de sorte à faciliter le suivi de vos ressources :

- Description : elle vous permet de fournir des informations plus détaillées sur la raison pour laquelle la définition d’image a été créée. Par exemple, vous avez peut-être besoin d’une définition d’image pour le serveur principal sur lequel l’application est préinstallée.
- CLUF : il peut être utilisé pour pointer vers un contrat de licence utilisateur final spécifique à la définition d’image.
- Notes de publication et déclaration de confidentialité : stockez des notes de publication et des déclarations de confidentialité dans le stockage Azure et fournissez un URI permettant d’y accéder dans la définition d’image.
- Date de fin de vie : indiquez une date de fin de vie pour la définition d’image afin de pouvoir utiliser l’automatisation pour supprimer les anciennes définitions d’image.
- Étiquette : vous pouvez ajouter des étiquettes lorsque vous créez votre définition d’image. Pour en savoir plus sur les étiquettes, voir [Organisation des ressources Azure à l’aide d’étiquettes](../azure-resource-manager/management/tag-resources.md).
- Suggestions concernant la quantité maximale et minimale de processeurs virtuels et de mémoire : si votre image est associée à ces types de recommandation, vous pouvez indiquer ces informations dans votre définition d’image.
- Types de disque non autorisés : vous pouvez fournir des informations sur les besoins de votre machine virtuelle en termes de stockage. Par exemple, si l’image n’est pas adaptée aux disques durs standard, vous pouvez les ajouter à la liste de disques non autorisés.
- Informations sur le plan d’achat d’images de la Place de marché - `-PurchasePlanPublisher`, `-PurchasePlanName` et `-PurchasePlanProduct`. Pour en savoir plus sur les informations relatives au plan d’achat, consultez [Trouver des images dans la Place de marché Azure](./windows/cli-ps-findimage.md) et [Donner des informations sur le plan d’achat de la Place de marché Azure lors de la création d’images](marketplace-images.md).


## <a name="image-versions"></a>Versions d’images

Une **version d’image** est ce que vous utilisez pour créer une machine virtuelle. Vous pouvez avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous utilisez une **version d’image** pour créer une machine virtuelle, la version d’image est utilisée pour créer des disques pour la machine virtuelle. Les versions d’image peuvent être utilisées plusieurs fois.

Les propriétés d’une version d’image sont les suivantes :

- Numéro de version. Il est utilisé comme nom de la version d’image. Son format est toujours : MajorVersion.MinorVersion.Patch. Si vous spécifiez d’utiliser la **dernière** image lors de la création d’une machine virtuelle, la dernière image est choisie en fonction de la valeur la plus élevée de MajorVersion, de MinorVersion, puis de Patch. 
- Source. La source peut être une machine virtuelle, un disque managé, une capture instantanée, une image managée ou une autre version de l’image. 
- Exclure de la plus récente. Vous pouvez empêcher l’utilisation d’une version comme version la plus récente de l’image. 
- Date de fin de vie. Date à partir de laquelle des machines virtuelles ne peuvent pas être créées à partir de cette image.


## <a name="generalized-and-specialized-images"></a>Images généralisées et spécialisées

Il existe deux états de système d'exploitation pris en charge par Galerie d’images partagées. Généralement, les images nécessitent que la machine virtuelle utilisée pour créer l'image ait été généralisée avant de créer l'image. La généralisation est un processus qui supprime de la machine virtuelle les informations spécifiques à la machine et à l'utilisateur. Pour Windows, l’outil Sysprep est utilisé. Pour Linux, vous pouvez utiliser les paramètres [waagent](https://github.com/Azure/WALinuxAgent), `-deprovision` ou `-deprovision+user`.

Les machines virtuelles spécialisées n'ont pas été soumises à un processus de suppression des informations et des comptes spécifiques aux machines. Par ailleurs, les machines virtuelles créées à partir d'images spécialisées ne sont associées à aucun `osProfile`. Cela signifie que les images spécialisées ont certaines limites en plus de certains avantages.

- Les machines virtuelles et les groupes identiques créés à partir d’images spécialisées peuvent être opérationnels plus rapidement. Étant donné qu’ils sont créés à partir d’une source qui a déjà fait l’objet d’un premier démarrage, les machines virtuelles créées à partir de ces images démarrent plus rapidement.
- Les comptes qui pourraient être utilisés pour se connecter à la machine virtuelle peuvent également être utilisés sur n'importe quelle machine virtuelle créée en utilisant l'image spécialisée créée à partir de celle-ci.
- Les machines virtuelles porteront le **nom de l'ordinateur** de la machine virtuelle d’où est extraire l’image. Vous devriez renommer l’ordinateur pour éviter tout conflit.
- Le `osProfile` représente la façon dont certaines informations sensibles sont transmises à la machine virtuelle, en utilisant `secrets`. Cela peut entraîner des problèmes lors de l'utilisation de KeyVault, WinRM et d'autres fonctionnalités qui utilisent `secrets` dans le `osProfile`. Dans certains cas, vous pouvez utiliser des identités de service managées (MSI) pour contourner ces limitations.

## <a name="regional-support"></a>Prise en charge régionale

Toutes les régions publiques peuvent être choisies comme cibles. Cependant, pour que vous puissiez effectuer la réplication dans les régions Australie Centre et Australie Centre 2, votre abonnement doit être ajouté à la liste verte. Pour demander l’ajout d’un abonnement à la liste verte, accédez à : https://azure.microsoft.com/global-infrastructure/australia/contact/

## <a name="limits"></a>limites 

Certaines limites par abonnement ont été définies pour le déploiement de ressources à l’aide des galeries d’images partagées :
- 100 galeries d’images partagées par abonnement et par région
- 1 000 définitions d’images par abonnement et par région
- 10 000 versions d’image par abonnement et par région
- 10 réplicas de version d’image par abonnement et par région
- Tout disque attaché à l’image doit avoir une taille inférieure ou égale à 1 To

Pour en savoir plus, consultez les exemples figurant dans la section [Vérifier l’utilisation des ressources par rapport aux limites](../networking/check-usage-against-limits.md), qui indiquent comment vérifier l’utilisation actuelle.
 
## <a name="scaling"></a>Mise à l'échelle
La galerie d’images partagées vous permet de spécifier le nombre de réplicas qu’Azure doit conserver pour les images. De cette façon, dans les scénarios de déploiement multimachines virtuelles, les déploiements de machines virtuelles peuvent être répartis sur différents réplicas pour réduire le risque de limitation du traitement de création d’instances liée à la surcharge d’un seul réplica.

Avec la galerie d’images partagées, vous pouvez désormais déployer jusqu’à 1 000 instances de machines virtuelles dans un groupe de machines virtuelles identiques (à partir de 600 avec des images managées). Le réplicas d’images permettent d’optimiser les performances d’un déploiement, sa fiabilité et sa cohérence.   Vous pouvez définir un nombre de réplicas différents dans chaque région cible, selon les besoins de mise à l’échelle de la région. Comme chaque réplica est une copie complète de votre image, cela vous permet de mettre à l’échelle vos déploiements de façon linéaire, en fonction de chaque réplica supplémentaire. Nous le savons bien, les images ou régions ne sont jamais les mêmes, mais nous vous invitons à suivre les règles suivantes sur la création de réplicas dans une région :

- Pour les déploiements qui ne concernent pas des groupes de machines virtuelles identiques : pour chaque lot de 20 machines virtuelles que vous créez simultanément, nous vous recommandons de ne garder qu’un réplica. Par exemple, si vous créez 120 machines virtuelles simultanément à l’aide de la même image dans une région, nous vous conseillons de conserver au moins 6 réplicas de votre image. 
- Pour les déploiements de groupes de machines virtuelles identiques : pour chaque déploiement de groupe identique avec un maximum de 600 instances, nous vous recommandons de garder au moins un réplica. Ainsi, si vous créez 5 groupes identiques simultanément, chacun doté de 600 instances de VM utilisant la même image dans une seule région, nous vous suggérons de conserver au moins 5 réplicas de votre image. 

De plus, nous recommandons toujours de surapprovisionner le nombre de réplicas, à cause de facteurs tels que la taille des images, le contenu et le type de système d’exploitation.

![Graphique montrant comment mettre à l’échelle des images](./media/shared-image-galleries/scaling.png)

## <a name="make-your-images-highly-available"></a>Rendre votre image hautement disponible

[Azure Zone Redundant Storage (ZRS)](https://azure.microsoft.com/blog/azure-zone-redundant-storage-in-public-preview/) fournit une résilience élevée contre une défaillance de la Zone de disponibilité dans la région. Avec la disponibilité générale de la galerie d’images partagées, vous pouvez choisir de stocker vos images dans les comptes ZRS au sein de régions avec des Zones de disponibilité. 

Vous pouvez également choisir le type de compte de chaque région cible. Le type de compte de stockage par défaut est Standard_LRS, mais vous pouvez choisir Standard_ZRS pour les régions avec des Zones de disponibilité. Vérifiez la disponibilité régionale de ZRS [ici](../storage/common/storage-redundancy.md).

![Graphique affiche ZRS](./media/shared-image-galleries/zrs.png)

## <a name="replication"></a>Réplication
La galerie d’images partagées vous permet aussi de répliquer vos images sur d’autres régions Azure automatiquement. Chaque version de l’image partagée peut être répliquée sur différentes régions en fonction des besoins de votre organisation. Par exemple, vous pouvez répliquer la dernière image dans plusieurs régions tout en gardant les versions plus anciennes disponibles dans une seule région. De cette façon vous économisez sur les coûts de stockage des versions d’image partagée. 

Les régions sur lesquelles est répliquée une version d’image partagée peuvent être mises à jour après la création. La durée de réplication sur différentes régions dépend de la quantité de données copiées et du nombre de régions concernées. La réplication peut prendre plusieurs heures dans certains cas. Pendant le processus, vous pouvez voir l’état de la réplication par région. Une fois la réplication d’image effectuée dans une région, vous pouvez ensuite déployer une machine virtuelle ou un groupe identique à partir de cette version d’image dans la région.

![Graphique montrant comment répliquer des images](./media/shared-image-galleries/replication.png)

## <a name="access"></a>Accès

Tout comme la Galerie d’images partagées, la définition d’image et la version d’image sont des ressources, qui peuvent être partagées à l’aide des contrôle d’accès en fonction du rôle (RBAC) Azure natifs intégrés. Le contrôle d’accès en fonction du rôle (RBAC) vous permet de partager ces ressources avec d’autres utilisateurs, principaux de service et groupes. Vous pouvez même partager l’accès avec des personnes en dehors du locataire au sein duquel ils ont été créés. Un utilisateur disposant d’un accès à la version d’image partagée peut déployer une machine virtuelle ou un groupe de machines virtuelles identiques.  La matrice de partage suivante vous aide à comprendre les éléments auxquels l’utilisateur a accès :

| Partagé avec l’utilisateur     | Galerie d’images partagées | Définition de l’image | Version d’image |
|----------------------|----------------------|--------------|----------------------|
| Galerie d’images partagées | Oui                  | Oui          | Oui                  |
| Définition de l’image     | Non                   | Oui          | Oui                  |

Nous vous recommandons de partager les images au niveau de la galerie, afin de proposer une expérience optimale. Nous vous déconseillons de partager des versions d’images individuelles. Pour en savoir plus sur le contrôle d’accès en fonction du rôle (RBAC), consultez la section relative à la [gestion de l’accès aux ressources Azure à l’aide du contrôle d’accès en fonction du rôle (RBAC)](../role-based-access-control/role-assignments-portal.md).

Les images peuvent également être partagées à grande échelle, même entre les locataires, via l’inscription d’une application multilocataire. Pour en savoir plus sur le partage d’images entre les locataires, consultez « Partager des images de machine virtuelle de la galerie entre des locataires Azure » à l’aide d’[Azure CLI](./linux/share-images-across-tenants.md) ou de [PowerShell](./windows/share-images-across-tenants.md).

## <a name="billing"></a>Facturation
L’utilisation du service Galerie d’images partagées n’engendre aucuns frais supplémentaires. Vous êtes facturé pour les ressources suivantes :
-   Coûts du stockage de chaque réplica. Le coût de stockage est facturé en tant qu’instantané et calculé sur la base de la taille occupée par la version de l’image, du nombre de réplicas de la version de l’image et du nombre de régions dans lesquelles la version est répliquée. 
-   Coûts de sortie de réseau pour la réplication de la première version d’image de la région source vers les régions répliquées. Les réplicas suivants sont gérés au sein de la région, donc aucun coût supplémentaire n’est prévu. 

Par exemple, imaginez que vous ayez une image d’un disque de système d’exploitation de 127 Go, qui occupe seulement 10 Go de stockage, et un disque de données vide de 32 Go. La taille occupée par chaque image est seulement de 10 Go. L’image est répliquée dans trois régions et chaque région a deux réplicas. Il y aura six instantanés au total, chacun utilisant 10 Go. Le coût de stockage de chaque instantané vous sera facturé conformément à la taille occupée de 10 Go. Vous paierez des frais de sortie du réseau pour la copie du premier réplica vers les deux régions supplémentaires. Pour plus d’informations sur les tarifs des instantanés dans chaque région, consultez [Tarifs des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/). Pour plus d’informations sur la sortie du réseau, consultez [Détails de la tarification de la bande passante](https://azure.microsoft.com/pricing/details/bandwidth/).


## <a name="updating-resources"></a>Mise à jour des ressources

Une fois qu’elles sont créées, vous pouvez apporter des modifications aux ressources de la galerie d’images. Ceux-ci sont limités aux éléments suivants :
 
Galerie d’images partagées :
- Description

Définition d’image :
- Processeurs virtuels recommandés
- Mémoire recommandée
- Description
- Date de fin de vie

Version d’image :
- Nombre de réplicas régionaux
- Régions cibles
- Exclure de la plus récente
- Date de fin de vie

## <a name="sdk-support"></a>Prise en charge des Kits de développement logiciel (SDK)

Les SDK suivants prennent en charge la création de galeries d’images partagées :

- [.NET](/dotnet/api/overview/azure/virtualmachines/management?view=azure-dotnet)
- [Java](/java/azure/?view=azure-java-stable)
- [Node.JS](/javascript/api/@azure/arm-compute)
- [Python](/python/api/overview/azure/virtualmachines?view=azure-python)
- [Go](/azure/go/)

## <a name="templates"></a>Modèles

Vous pouvez créer la ressource de galerie d’images partagées à l’aide de modèles. Plusieurs modèles de démarrage rapide Azure sont disponibles : 

- [Créer une galerie d’images partagées](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Créer une définition d’image dans une galerie d’images partagées](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Créer une version d’image dans une galerie d’images partagées](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [Créer une machine virtuelle à partir d’une version d’image](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

## <a name="frequently-asked-questions"></a>Forum aux questions 

* [Comment lister toutes les ressources de galerie d’images partagées de différents abonnements ?](#how-can-i-list-all-the-shared-image-gallery-resources-across-subscriptions) 
* [Puis-je déplacer mon image existante vers la galerie d’images partagées ?](#can-i-move-my-existing-image-to-the-shared-image-gallery)
* [Puis-je créer une version d’image à partir d’un disque spécialisé ?](#can-i-create-an-image-version-from-a-specialized-disk)
* [Une fois la ressource de galerie d’images partagées créée, puis-je la déplacer vers un autre abonnement ?](#can-i-move-the-shared-image-gallery-resource-to-a-different-subscription-after-it-has-been-created)
* [Puis-je répliquer mes versions d’image entre des clouds tels qu’Azure China 21Vianet, Azure Allemagne et Azure Government ?](#can-i-replicate-my-image-versions-across-clouds-such-as-azure-china-21vianet-or-azure-germany-or-azure-government-cloud)
* [Puis-je répliquer mes versions d’image entre abonnements ?](#can-i-replicate-my-image-versions-across-subscriptions)
* [Puis-je partager des versions d’image entre locataires Azure AD ?](#can-i-share-image-versions-across-azure-ad-tenants)
* [Combien de temps faut-il pour répliquer des versions d’image sur les régions cibles ?](#how-long-does-it-take-to-replicate-image-versions-across-the-target-regions)
* [Quelle est la différence entre la région source et la région cible ?](#what-is-the-difference-between-source-region-and-target-region)
* [Comment spécifier la région source pendant la création de la version d’image ?](#how-do-i-specify-the-source-region-while-creating-the-image-version)
* [Comment spécifier le nombre de réplicas de version d’image à créer dans chaque région ?](#how-do-i-specify-the-number-of-image-version-replicas-to-be-created-in-each-region)
* [Puis-je créer la galerie d’images partagées à un emplacement différent de celui de la définition d’image et de la version d’image ?](#can-i-create-the-shared-image-gallery-in-a-different-location-than-the-one-for-the-image-definition-and-image-version)
* [Quel est le coût d’utilisation de la galerie d’images partagées ?](#what-are-the-charges-for-using-the-shared-image-gallery)
* [Quelle version d’API utiliser pour créer une galerie d’images partagées, une définition d’image et une version d’image ?](#what-api-version-should-i-use-to-create-shared-image-gallery-and-image-definition-and-image-version)
* [Quelle version d’API utiliser pour créer une machine virtuelle partagée ou un groupe de machines virtuelles identiques à partir de la version d’image ?](#what-api-version-should-i-use-to-create-shared-vm-or-virtual-machine-scale-set-out-of-the-image-version)
* [Puis-je mettre à jour mon groupe de machines virtuelles identiques créé à l’aide d’images managées pour utiliser des images Shared Image Gallery ?](#can-i-update-my-virtual-machine-scale-set-created-using-managed-image-to-use-shared-image-gallery-images)

### <a name="how-can-i-list-all-the-shared-image-gallery-resources-across-subscriptions"></a>Comment lister toutes les ressources de galerie d’images partagées de différents abonnements ?

Pour lister toutes les ressources de galerie d’images partagées de différents abonnements auxquels vous avez accès sur le portail Azure, suivez les étapes ci-dessous :

1. Ouvrez le [portail Azure](https://portal.azure.com).
1. Faites défiler la page vers le bas, puis sélectionnez **Toutes les ressources**.
1. Sélectionnez tous les abonnements pour lesquels vous voulez lister toutes les ressources.
1. Recherchez des ressources de type **Galerie d’images partagée**.
  
Pour lister toutes les ressources de galerie d’images partagées des différents abonnements sur lesquels vous avez des autorisations, utilisez la commande suivante dans Azure CLI :

```azurecli
   az account list -otsv --query "[].id" | xargs -n 1 az sig list --subscription
```

Pour plus d’informations, consultez **Gérer les ressources de la galerie** avec [Azure CLI](update-image-resources-cli.md) ou [PowerShell](update-image-resources-powershell.md).

### <a name="can-i-move-my-existing-image-to-the-shared-image-gallery"></a>Puis-je déplacer mon image existante vers la galerie d’images partagées ?
 
Oui. Il existe 3 scénarios basés sur les types d’images.

 Scénario 1 : Si vous avez une image managée, vous pouvez créer une définition et une version de cette image. Pour plus d’informations, consultez **Effectuer une migration à partir d’une image managée vers une version d’image** avec [Azure CLI](image-version-managed-image-cli.md) ou [PowerShell](image-version-managed-image-powershell.md).

 Scénario 2 : Si vous avez une image non managée, vous pouvez créer une image managée de l’image, puis créer une définition et une version de cette image. 

 Scénario 3 : Si vous avez un VHD dans votre système de fichiers local, vous devez le charger sur une image managée pour pouvoir ensuite créer une définition et une version de cette image.

- Si le VHD vient d’une machine virtuelle Windows, consultez [Charger un disque dur virtuel](./windows/upload-generalized-managed.md).
- Si le VHD est pour une machine virtuelle Linux, consultez [Charger un disque dur virtuel](./linux/upload-vhd.md#option-1-upload-a-vhd)

### <a name="can-i-create-an-image-version-from-a-specialized-disk"></a>Puis-je créer une version d’image à partir d’un disque spécialisé ?

Oui, vous pouvez créer une machine virtuelle à partir d’une image spécialisée en utilisant l’interface [CLI](vm-specialized-image-version-cli.md), [PowerShell](vm-specialized-image-version-powershell.md) ou l’API. 

### <a name="can-i-move-the-shared-image-gallery-resource-to-a-different-subscription-after-it-has-been-created"></a>Une fois la ressource de galerie d’images partagées créée, puis-je la déplacer vers un autre abonnement ?

Non, vous ne pouvez pas déplacer la ressource de galerie d’images partagées vers un autre abonnement. Vous pouvez répliquer les versions de l’image se trouvant dans la galerie vers d’autres régions ou copier une image d’une autre galerie à l’aide d’[Azure CLI](image-version-another-gallery-cli.md) ou de [PowerShell](image-version-another-gallery-powershell.md).

### <a name="can-i-replicate-my-image-versions-across-clouds-such-as-azure-china-21vianet-or-azure-germany-or-azure-government-cloud"></a>Puis-je répliquer mes versions d’image entre des clouds tels qu’Azure China 21Vianet, Azure Allemagne et Azure Government ?

Non, vous ne pouvez pas répliquer les versions d’image entre clouds.

### <a name="can-i-replicate-my-image-versions-across-subscriptions"></a>Puis-je répliquer mes versions d’image entre abonnements ?

Non, vous pouvez répliquer les versions d’image entre régions dans un abonnement et les utiliser dans d’autres abonnements au moyen de RBAC.

### <a name="can-i-share-image-versions-across-azure-ad-tenants"></a>Puis-je partager des versions d’image entre locataires Azure AD ? 

Oui, vous pouvez utiliser le contrôle d’accès en fonction du rôle (RBAC) pour partager des images avec des individus se trouvant sur différents locataires. Si vous voulez effectuer un partage à grande échelle, consultez la section relative au partage d’images de galerie sur plusieurs locataires Azure via [PowerShell](./windows/share-images-across-tenants.md) ou la [CLI](./linux/share-images-across-tenants.md).

### <a name="how-long-does-it-take-to-replicate-image-versions-across-the-target-regions"></a>Combien de temps faut-il pour répliquer des versions d’image sur les régions cibles ?

Le temps de réplication des versions d’image dépend entièrement de la taille de l’image et du nombre de régions cibles. Toutefois, une bonne pratique est de limiter la taille de l’image et de rapprocher les régions source et cible pour obtenir de meilleurs résultats. Vous pouvez vérifier l’état de la réplication à l’aide de l’indicateur -ReplicationStatus.

### <a name="what-is-the-difference-between-source-region-and-target-region"></a>Quelle est la différence entre la région source et la région cible ?

La région source est la région dans laquelle votre version d’image est créée et les régions cibles sont les régions dans lesquelles vous stockez une copie de votre version d’image. Pour chaque version d’image, vous pouvez avoir seulement une région source. Par ailleurs, vérifiez que vous passez l’emplacement de la région source comme l’une des régions cibles quand vous créez une version d’image.

### <a name="how-do-i-specify-the-source-region-while-creating-the-image-version"></a>Comment spécifier la région source pendant la création de la version d’image ?

Quand vous créez une version d’image, vous pouvez utiliser l’étiquette **--location** dans l’interface CLI et **-Location** dans PowerShell pour spécifier la région source. Vérifiez que l’image managée que vous utilisez comme image de base pour créer la version d’image est dans le même emplacement que celui dans lequel vous voulez créer la version d’image. Par ailleurs, vérifiez que vous passez l’emplacement de la région source comme l’une des régions cibles quand vous créez une version d’image.  

### <a name="how-do-i-specify-the-number-of-image-version-replicas-to-be-created-in-each-region"></a>Comment spécifier le nombre de réplicas de version d’image à créer dans chaque région ?

Deux méthodes vous permettent de spécifier le nombre de réplicas de version d’image à créer dans chaque région :
 
1. Le nombre de réplicas régionaux, qui spécifie le nombre de réplicas que vous voulez créer par région. 
2. Le nombre de réplicas communs, qui est le nombre par défaut par région si le nombre de réplicas régionaux n’est pas spécifié. 

Pour spécifier le nombre de réplicas régionaux, passez l’emplacement avec le nombre de réplicas à créer dans cette région : « USA Centre Sud=2». 

Si le nombre de réplicas régionaux n’est pas spécifié avec chaque emplacement, le nombre de réplicas par défaut est le nombre de réplicas communs que vous avez spécifié. 

Pour spécifier le nombre de réplicas communs dans l’interface CLI, utilisez l’argument **--replica-count** dans la commande `az sig image-version create`.

### <a name="can-i-create-the-shared-image-gallery-in-a-different-location-than-the-one-for-the-image-definition-and-image-version"></a>Puis-je créer la galerie d’images partagées à un emplacement différent de celui de la définition d’image et de la version d’image ?

Oui, vous pouvez. Cependant, nous vous encourageons à conserver au même endroit le groupe de ressources, la galerie d’images partagées, la définition d’image et la version d’image.

### <a name="what-are-the-charges-for-using-the-shared-image-gallery"></a>Quel est le coût d’utilisation de la galerie d’images partagées ?

L’utilisation du service Galerie d’images partagées n’engendre aucun coût, à l’exception des coûts de stockage des versions d’image et des coûts de sortie de réseau pour la réplication des versions d’image de la région source vers les régions cibles.

### <a name="what-api-version-should-i-use-to-create-shared-image-gallery-and-image-definition-and-image-version"></a>Quelle version d’API utiliser pour créer une galerie d’images partagées, une définition d’image et une version d’image ?

Pour utiliser des galeries d’images partagées, des définitions d’image et des versions d’image, nous vous recommandons d’utiliser la version d’API 2018-06-01. ZRS requiert la version 2019-03-01 ou plus.

### <a name="what-api-version-should-i-use-to-create-shared-vm-or-virtual-machine-scale-set-out-of-the-image-version"></a>Quelle version d’API utiliser pour créer une machine virtuelle partagée ou un groupe de machines virtuelles identiques à partir de la version d’image ?

Pour déployer une machine virtuelle ou un groupe de machines virtuelles identiques à partir d’une version d’image, nous vous recommandons d’utiliser la version d’API 2018-04-01 ou une version ultérieure.

### <a name="can-i-update-my-virtual-machine-scale-set-created-using-managed-image-to-use-shared-image-gallery-images"></a>Puis-je mettre à jour mon groupe de machines virtuelles identiques créé à l’aide d’images managées pour utiliser des images Shared Image Gallery ?

Oui, vous pouvez mettre à jour la référence d’image de groupe identique à partir d’une image managée vers une image de galerie d’images partagées, à condition que le type de système d’exploitation, la génération Hyper-V et la disposition du disque de données soient cohérents entre les images.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment déployer des images partagées avec [Azure CLI](shared-images-cli.md) et [PowerShell](shared-images-powershell.md).

