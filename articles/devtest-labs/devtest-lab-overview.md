---
title: À propos d’Azure DevTest Labs | Microsoft Docs
description: Découvrez comment DevTest Labs peut faciliter la création, la gestion et la surveillance des machines virtuelles Azure
ms.topic: article
ms.date: 06/20/2020
ms.openlocfilehash: b7e6493e5fb1c99dfa7962ed464dca3ebc6b0c7e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85480199"
---
# <a name="about-azure-devtest-labs"></a>À propos d’Azure DevTest Labs
Azure DevTest Labs permet aux développeurs au sein d’équipes de gérer eux-mêmes efficacement les machines virtuelles et les ressources PaaS sans attendre les approbations.

DevTest Labs crée des labos consistant en bases préconfigurées ou en modèles Azure Resource Manager. Ils ont l’ensemble des outils et logiciels nécessaires que vous pouvez utiliser pour créer des environnements. Vous pouvez créer des environnements en quelques minutes, et non pas en plusieurs heures ou plusieurs jours.

Avec DevTest Labs, vous pouvez tester les dernières versions de vos applications en effectuant les tâches suivantes :

- Provisionner rapidement des environnements Windows et Linux en utilisant des modèles et des artefacts réutilisables.
- DevTest Labs facilite l’intégration de votre pipeline de déploiement pour approvisionner des environnements à la demande.
- Effectuez un scale-up de votre test de charge en provisionnant plusieurs agents de test et créez des environnements préprovisionnés pour des formations et des démonstrations.

> [!VIDEO https://channel9.msdn.com/Blogs/Azure/What-is-Azure-DevTest-Labs/player]

## <a name="capabilities"></a>Fonctionnalités
Dev/Test Labs fournit les fonctionnalités suivantes pour les développeurs travaillant avec des machines virtuelles :

- Créer rapidement des machines virtuelles en suivant moins de cinq étapes simples.
- Choisir parmi une liste de bases de machines virtuelles qui sont configurées, approuvées et autorisées par le responsable d’équipe ou le département informatique central.
- Créer des machines virtuelles à partir d’images personnalisées précréées où tous les logiciels et outils sont déjà installés. 
- Créer des machines virtuelles à partir de formules qui sont des images personnalisées combinées avec les builds les plus récentes des logiciels qui sont installés quand les machines virtuelles sont créées. 
- Installer des artefacts qui sont des extensions déployées sur des machines virtuelles après leur provisionnement.
- Définir des planifications d’arrêt automatique et de démarrage automatique sur des machines virtuelles.
- Revendiquer une machine virtuelle précréée sans passer par le processus de création.

Dev/Test Labs fournit les fonctionnalités suivantes pour les développeurs travaillant avec des environnements PaaS :

- Utiliser Resource Manager pour créer rapidement des environnements PaaS en suivant moins de trois étapes simples.
- Choisir parmi une liste de modèles Resource Manager qui sont configurés et autorisés par le responsable d’équipe ou le département informatique central.
- Créer un groupe de ressources vide (bac à sable) en utilisant un modèle Resource Manager pour explorer Azure dans le contexte d’un labo.

DevTest Labs permet également au département informatique central de contrôler les pertes, d’optimiser les coûts sur les ressources et de respecter les budgets en effectuant les tâches suivantes :  

- Définition de planifications d’arrêt automatique et de démarrage automatique sur des machines virtuelles.
- Définition de stratégies quant au nombre de machines virtuelles que les utilisateurs peuvent créer.
- Définition de stratégies quant aux tailles de machines virtuelles et aux images de la galerie que les utilisateurs peuvent choisir.
- Suivi des coûts et définition de cibles sur les labos.
- Notification des coûts élevés prévus pour des labos, de façon à pouvoir prendre les actions nécessaires.

DevTest Labs offre les avantages suivants lors de la création, de la configuration et de la gestion d’environnements dans le cloud.

## <a name="cost-control-and-governance"></a>Contrôle des coûts et gouvernance
DevTest Labs facilite le contrôle des coûts en vous permettant d’effectuer les tâches suivantes :

- [Définir des stratégies pour vos labos](devtest-lab-set-lab-policy.md), comme le nombre de machines virtuelles par utilisateur ou par labo. 
- Créer des [stratégies pour arrêter et démarrer automatiquement](devtest-lab-set-lab-policy.md) des machines virtuelles.
- Suivre les coûts sur les machines virtuelles et les ressources PaaS créées au sein des labos de façon à respecter [votre budget](devtest-lab-configure-cost-management.md).
- Rester dans le contexte de vos labos, de façon à ne pas créer de ressources en dehors de ceux-ci.

## <a name="quickly-get-to-ready-to-test"></a>Vos applications sont prêtes pour le test en un clin d’œil
DevTest Labs vous permet de créer des environnements préprovisionnés équipés avec tout ce dont votre équipe a besoin pour commencer à développer et à tester des applications. [Revendiquez simplement les environnements](devtest-lab-add-claimable-vm.md) où la dernière build satisfaisante de votre application est installée et commencez à travailler. Vous pouvez aussi utiliser des conteneurs pour créer encore plus rapidement et plus facilement des environnements.

## <a name="create-once-use-everywhere"></a>Créez, réutilisez
Capturez et partagez des [modèles d’environnement](devtest-lab-create-environment-from-arm.md) PaaS et des [artefacts](add-artifact-repository.md) au sein de votre équipe ou organisation, tout cela dans le contrôle du code source, pour créer facilement des environnements de développement et de test.

## <a name="worry-free-self-service"></a>Libre-service convivial
DevTest Labs permet à vos développeurs et à vos testeurs de [créer des machines virtuelles IaaS](devtest-lab-add-vm.md) et des [ressources PaaS](devtest-lab-create-environment-from-arm.md) rapidement et facilement en utilisant un ensemble de ressources préconfigurées.

## <a name="use-iaas-and-paas-resources"></a>Utiliser des ressources IaaS et PaaS 
Les développeurs peuvent également créer des ressources PaaS, comme des clusters Azure Service Fabric, la fonctionnalité Web Apps d’Azure App Service et des batteries de serveurs SharePoint, en utilisant des modèles Resource Manager. Pour bien démarrer avec PaaS dans les labos, utilisez les modèles du [référentiel d’environnements publics](devtest-lab-configure-use-public-environments.md) ou [connectez le labo à votre propre dépôt Git](devtest-lab-create-environment-from-arm.md#configure-your-own-template-repositories). Vous pouvez aussi suivre les coûts sur ces ressources de façon à respecter votre budget.

## <a name="integrate-with-your-existing-toolchain"></a>Intégrer à votre chaîne d’outils existante
Utilisez des plug-ins prêts à l’emploi ou l’API pour provisionner des environnement de développement/test directement à partir de votre [outil d’intégration continue](devtest-lab-integrate-ci-cd.md) préféré, d’un environnement de développement intégré ou d’un pipeline de mise en production automatisé. Vous pouvez également utiliser l’outil en ligne de commande complet.

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :

- Pour en savoir plus sur DevTest Labs, consultez [Concepts de DevTest Labs](devtest-lab-concepts.md).
- Pour une démonstration avec des instructions pas à pas, consultez [Tutoriel : Configurer un labo avec Azure DevTest Labs](tutorial-create-custom-lab.md).