---
title: Scénarios courants d’utilisation d’Azure DevTest Labs
description: Cet article décrit les principaux scénarios d’utilisation d’Azure DevTest Labs et deux cheminements généraux pour commencer à utiliser le service dans votre organisation.
ms.topic: article
ms.date: 06/20/2020
ms.reviewer: christianreddington,anthdela,juselph
ms.openlocfilehash: 1adb9b009a250942bab7e9b0e3b4b3e52840341c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85481593"
---
# <a name="popular-scenarios-for-using-azure-devtest-labs"></a>Scénarios courants d’utilisation d’Azure DevTest Labs
Selon les besoins d’une entreprise, DevTest Labs peut être configuré pour répondre à différentes exigences.  Cet article présente les scénarios courants. Chaque scénario expose les avantages apportés par l’utilisation de DevTest Labs et les ressources à utiliser pour implémenter ces scénarios.  

- Postes de travail de développeur
- Environnements de test
- Sessions de formation, ateliers pratiques et hackathons
- Investigations en mode bac à sable
- Laboratoires de salle de classe

## <a name="developer-desktops"></a>Postes de travail de développeur
Les développeurs ont souvent des exigences distinctes concernant les ordinateurs de développement pour différents projets. Avec DevTest Labs, les développeurs peuvent accéder à des machines virtuelles à la demande, qui sont configurées pour s’adapter à leurs scénarios les plus courants. DevTest Labs permet de bénéficier des avantages suivants :

- Les organisations peuvent fournir des machines courantes, garantissant la cohérence entre les équipes.
- Les développeurs peuvent provisionner rapidement leurs machines de développement à la demande ou [demander une machine préconfigurée existante](devtest-lab-add-claimable-vm.md).
- Les développeurs peuvent provisionner des ressources en libre-service sans avoir besoin d’autorisations de niveau abonnement.
- Les informaticiens ou les administrateurs peuvent [prédéfinir la topologie réseau](devtest-lab-configure-vnet.md), et les développeurs peuvent l’utiliser directement, de manière simple et intuitive, sans accès spécial.
- Les développeurs peuvent facilement [personnaliser](devtest-lab-add-vm.md#add-an-existing-artifact-to-a-vm) leurs machines de développement selon leurs besoins.
- Les administrateurs peuvent contrôler les coûts en s’assurant que :
    - Les développeurs [ne peuvent pas obtenir plus de machines virtuelles](devtest-lab-set-lab-policy.md#set-virtual-machines-per-user) que nécessaire pour le développement.
    - [Les machines virtuelles sont arrêtées](devtest-lab-set-lab-policy.md#set-auto-shutdown) quand elles ne sont pas utilisées.
    - Seul [un sous-ensemble des tailles d’instance de machine virtuelle est autorisé](devtest-lab-set-lab-policy.md#set-allowed-virtual-machine-sizes) pour les labos spécifiques.
    - [Gestion des coûts cibles et des notifications](devtest-lab-configure-cost-management.md) pour chaque labo.

Pour plus d’informations, consultez [Utiliser Azure DevTest Labs pour les développeurs](devtest-lab-developer-lab.md). 

## <a name="test-environments"></a>Environnements de test
La création et la gestion des environnements de test dans une entreprise peuvent nécessiter un effort considérable. Avec DevTest Labs, les environnements de test peuvent être facilement créés, mis à jour ou dupliqués. Il permet aux équipes d’accéder à un environnement entièrement configuré quand c’est nécessaire. Dans ce scénario, DevTest Labs offre les avantages suivants :

- Les organisations peuvent fournir des environnements de test garantissant la cohérence entre les équipes.
- Les testeurs peuvent tester la dernière version de leur application en provisionnant rapidement des environnements Windows et Linux grâce à des modèles réutilisables.
- Les administrateurs peuvent connecter le labo à Azure DevOps pour permettre des scénarios DevOps.
- Les propriétaires de labo peuvent contrôler les coûts en garantissant que :
    - [Les machines virtuelles sont arrêtées](devtest-lab-set-lab-policy.md#set-auto-shutdown) quand elles ne sont pas utilisées.
    - Seul [un sous-ensemble des tailles d’instance de machine virtuelle est autorisé](devtest-lab-set-lab-policy.md#set-allowed-virtual-machine-sizes) pour les labos spécifiques.
    - [Gestion des coûts cibles et des notifications](devtest-lab-configure-cost-management.md) pour chaque labo.

Pour plus d’informations, consultez [Utiliser Azure DevTest Labs pour les environnements de test de machine virtuelle et PaaS](devtest-lab-test-env.md).

## <a name="sandboxed-investigations"></a>Investigations en mode bac à sable
Les développeurs examinent souvent différentes technologies ou des conceptions d’infrastructure. Par défaut, tous les environnements créés avec DevTest Labs sont créés dans leur propre groupe de ressources. L’utilisateur de DevTest Labs obtient seulement un accès en lecture à ces ressources. Cependant, pour les développeurs qui ont besoin de davantage de contrôle, un paramètre de laboratoire peut être mis à jour pour donner des [droits de contributeur](https://azure.microsoft.com/updates/azure-devtest-labs-view-and-set-access-rights-to-an-environment-rg/) à l’utilisateur d’origine de DevTest Labs pour chaque environnement qu’il crée.  Avec DevTest Labs, les développeurs peuvent recevoir automatiquement l’autorisation de contributeur pour les environnements qu’ils créent dans le labo.  Ce scénario permet aux développeurs d’ajouter et/ou de modifier les ressources Azure dont ils ont besoin pour leurs environnements de développement ou de test. La page [Coût par ressource](devtest-lab-configure-cost-management.md#view-cost-by-resource) permet aux propriétaires de labo de suivre le coût de chaque environnement utilisé pour les investigations.

Pour plus d’informations, consultez [Définir les droits d’accès à un groupe de ressources d’environnement](https://aka.ms/dtl-sandbox).

## <a name="trainings-hands-on-labs-and-hackathons"></a>Formations, ateliers pratiques et hackathons 
Un labo dans Azure DevTest Labs est un conteneur de choix pour les activités temporaires comme les ateliers, les ateliers pratiques, les formations ou les hackathons.  Ce service vous permet de créer un laboratoire où vous pouvez fournir des modèles personnalisés utilisables par chaque participant pour créer des environnements de formation identiques et isolés. Dans ce scénario, DevTest Labs offre les avantages suivants :

- Des [stratégies](devtest-lab-set-lab-policy.md) garantissent que les participants obtiennent seulement le nombre de ressources dont ils ont besoin, comme des machines virtuelles.
- Les machines préconfigurées et créées sont [demandées](devtest-lab-add-claimable-vm.md) en une seule action du participant.
- Les labos sont partagés avec les participants via l’accès à une [URL du labo](devtest-lab-faq.md#how-do-i-share-a-direct-link-to-my-lab).
- Des [dates d’expiration](devtest-lab-add-vm.md#steps-to-add-a-vm-to-a-lab-in-azure-devtest-labs) sur les machines virtuelles garantissent que les machines sont supprimées une fois qu’elles ne sont plus nécessaires.
- Il est facile de [supprimer un labo](devtest-lab-delete-lab-vm.md#delete-a-lab) et toutes les [ressources associées](devtest-lab-faq.md#how-do-i-automate-the-process-of-deleting-all-the-vms-in-my-lab) quand la formation terminée.

Pour plus d’informations, consultez [Utiliser Azure DevTest Labs à des fins de formation](devtest-lab-training-lab.md).  

## <a name="proof-of-concept-vs-scaled-deployment"></a>Déploiements de type « preuve de concept » et déploiements « à l’échelle »
Pour explorer DevTest Labs, vous pouvez utiliser deux types de cheminements : un déploiement de type « preuve de concept » et un déploiement de type « à l’échelle ».  

Un **déploiement à l’échelle** consiste en des semaines/mois d’examen et de planification dans le but de déployer DevTest Labs dans toute l’entreprise pour des centaines voire des milliers de développeurs.

Un déploiement de type **preuve de concept** est centré sur le travail d’une seule équipe pour établir une valeur au niveau de l’organisation. Si un déploiement à l’échelle semble tentant, l’approche a tendance à échouer plus souvent que l’option de preuve de concept. Par conséquent, nous vous recommandons de commencer doucement. Apprenez de la première équipe, répétez la même approche avec deux ou trois équipes supplémentaires, puis planifiez un déploiement à l’échelle en vous appuyant sur les connaissances acquises. Pour que l’option de preuve de concept réussisse, nous vous recommandons de choisir une ou deux équipes et d’identifier leurs scénarios (environnement de test ou environnement de développement), de documenter leurs cas d’utilisation actuels et de déployer DevTest Labs.

## <a name="next-steps"></a>Étapes suivantes
Consultez les articles suivants :

- [Concepts de DevTest Labs](devtest-lab-concepts.md)
- [FAQ DevTest Labs](devtest-lab-faq.md)

