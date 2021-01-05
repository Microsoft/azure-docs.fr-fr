---
title: Évaluer un grand nombre de serveurs physiques pour la migration vers Azure avec Azure Migrate | Microsoft Docs
description: Décrit comment évaluer un grand nombre de serveurs physiques pour la migration vers Azure à l’aide du service Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: how-to
ms.date: 01/19/2020
ms.openlocfilehash: 45954a42613fb1716e3b66305c0485a3966b8e37
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753686"
---
# <a name="assess-large-numbers-of-physical-servers-for-migration-to-azure"></a>Évaluer un grand nombre de serveurs physiques pour la migration vers Azure

Cet article explique comment évaluer un grand nombre de serveurs physiques locaux pour les migrer vers Azure à l’aide de l’outil d’évaluation de serveur Azure Migrate.

[Azure Migrate](migrate-services-overview.md) fournit un hub d’outils qui vous permettent de découvrir, d’évaluer et de migrer des applications, une infrastructure et des charges de travail vers Microsoft Azure. Le hub comprend des outils Azure Migrate et des offres d’ISV (fournisseurs de logiciels indépendants) tiers. 


Dans cet article, vous apprendrez comment :
> [!div class="checklist"]
> * Planifier l’évaluation à grande échelle.
> * Configurer les autorisations Azure et préparer les serveurs physiques pour l’évaluation.
> * Créer un projet Azure Migrate et une évaluation.
> * Examiner l’évaluation au fur et à mesure que vous planifiez la migration.


> [!NOTE]
> Si vous voulez tester une preuve de concept afin d’évaluer quelques serveurs avant d’effectuer une évaluation à grande échelle, suivez notre [série de tutoriels](./tutorial-discover-physical.md).

## <a name="plan-for-assessment"></a>Planifier l’évaluation

Lorsque vous planifiez l’évaluation d’un grand nombre de serveurs physiques, quelques points sont à prendre en compte :

- **Planification de projets Azure Migrate** : Découvrez comment déployer les projets Azure Migrate. Par exemple, si vos centres de données sont situés dans des zones géographiques différentes, ou si vous avez besoin de stocker des métadonnées de découverte, d’évaluation ou de migration dans une autre zone géographique, vous pouvez avoir besoin de plusieurs projets.
- **Planifier des appliances** : Azure Migrate utilise une appliance Azure Migrate locale, déployée sur un ordinateur Windows, afin de découvrir en permanence des serveurs pour l’évaluation et la migration. L’appliance surveille les changements d’environnement tels que l’ajout de machines virtuelles, de disques ou d’adaptateurs réseau. Elle envoie également à Azure les métadonnées et les données de performance concernant ces opérations. Vous devez déterminer le nombre d’appliances à déployer.


## <a name="planning-limits"></a>Limites de planification
 
Utilisez les limites résumées dans ce tableau pour la planification.

**Planification** | **Limites**
--- | --- 
**Projets Azure Migrate** | Évaluez jusqu’à 35 000 serveurs par projet.
**Appliance Azure Migrate** | Une appliance peut découvrir jusqu’à 1000 serveurs.<br/> Une appliance ne peut être associée qu’à un seul projet Azure Migrate.<br/> Un nombre quelconque d’appliances peut être associé à un même projet Azure Migrate. <br/><br/> 
**Groupe** | Vous pouvez ajouter jusqu’à 35 000 serveurs dans un groupe unique.
**Évaluation d’Azure Migrate** | Vous pouvez évaluer jusqu’à 35 000 serveurs par évaluation.


## <a name="other-planning-considerations"></a>Autres considérations en matière de planification

- Pour démarrer la découverte à partir de l’appliance, vous devez sélectionner chaque serveur physique. 

## <a name="prepare-for-assessment"></a>Préparer pour l’évaluation

Préparez les serveurs physiques et Azure à une évaluation de serveur. 

1. Vérifiez les [spécifications et limitations de la prise en charge des serveurs physiques](migrate-support-matrix-physical.md).
2. Configurez des autorisations pour permettre à votre compte Azure d’interagir avec Azure Migrate.
3. Préparez les serveurs physiques.

Suivez les instructions de [ce tutoriel](./tutorial-discover-physical.md) pour configurer ces paramètres.

## <a name="create-a-project"></a>Création d’un projet

Conformément à vos exigences de planification, procédez comme suit :

1. Créer un projet Azure Migrate.
2. Ajoutez l’outil Évaluation de serveur Azure Migrate aux projets.

[En savoir plus](./create-manage-projects.md)

## <a name="create-and-review-an-assessment"></a>Créer et examiner une évaluation

1. Créez des évaluations pour les serveurs physiques.
1. Passez en revue les évaluations en préparation de la planification de la migration.

[En savoir](tutorial-assess-physical.md) plus sur la création et l’examen des évaluations.
    

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous découvrirez comment :
 
> [!div class="checklist"] 
> * Planifier la mise à l’échelle des évaluations Azure Migrate pour les serveurs physiques.
> * Préparer les serveurs physiques et Azure à une évaluation.
> * Créer un projet Azure Migrate et effectuer des évaluations.
> * Passer en revue les évaluations en préparation de la migration.

Maintenant, [découvrez comment](concepts-assessment-calculation.md) les évaluations sont calculées, et comment les [modifier](how-to-modify-assessment.md).