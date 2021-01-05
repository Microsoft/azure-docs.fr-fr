---
title: Orchestrer l’implémentation d’Azure DevTest Labs
description: Cet article fournit des conseils pour orchestrer l’implémentation d’Azure DevTest Labs dans votre organisation.
ms.topic: article
ms.date: 06/26/2020
ms.reviewer: christianreddington,anthdela,juselph
ms.openlocfilehash: 1958e818f014b7419a1a33e9453fbad460dfc159
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92330612"
---
# <a name="orchestrate-the-implementation-of-azure-devtest-labs"></a>Orchestrer l’implémentation d’Azure DevTest Labs
Cet article présente une approche recommandée pour un déploiement et une implémentation rapides d’Azure DevTest Labs. L’illustration suivante met l’accent sur l’ensemble du processus sous forme de conseils prescriptifs tout en observant la flexibilité pour prendre en charge différents scénarios et différentes exigences du secteur.

![Étapes d’implémentation d’Azure DevTest Labs](./media/devtest-lab-guidance-orchestrate-implementation/implementation-steps.png)

## <a name="assumptions"></a>Hypothèses
Cet article suppose que les éléments suivants sont en place avant d’implémenter un pilote DevTest Labs :

- **Abonnement Azure** : l’équipe pilote a accès au déploiement des ressources dans un abonnement Azure. Si les charges de travail sont uniquement des charges de développement et de test, il est recommandé de sélectionner l’offre Enterprise DevTest pour obtenir des images disponibles supplémentaires et des tarifs inférieurs sur les machines virtuelles Windows.
- **Accès local** : si nécessaire, l’accès local a déjà été configuré. Il peut être établi via une connexion VPN de site à site ou ExpressRoute. Établir la connectivité via Express Route prend généralement plusieurs semaines, il est recommandé qu’ExpressRoute soit déjà en place avant de démarrer le projet.
- **Équipes pilotes** : les équipes de projet de développement initial qui utilisent DevTest Labs ont été identifiées, ainsi que les activités de développement ou de test applicables et des exigences/objectifs ont été établis pour ces équipes.

## <a name="milestone-1-establish-initial-network-topology-and-design"></a>Phase 1 : établir la conception et la topologie de réseau initiales
La première chose à faire lors du déploiement d’une solution Azure DevTest Labs est d’établir la connectivité planifiée pour les machines virtuelles. Voici les procédures à suivre :

1. Définir **les plages d’adresses IP initiales** qui sont assignées à l’abonnement DevTest dans Azure. Cette étape nécessite de prévoir l’utilisation attendue en nombre de machines virtuelles afin de pouvoir fournir un bloc assez grand pour une expansion future.
2. Identifier les **méthodes d’accès souhaité** dans les DevTest Labs (par exemple, l’accès externe / interne). Un point clé de cette étape consiste à déterminer si les machines virtuelles ont des adresses IP publiques (autrement dit, accessibles directement à partir d’internet).
3. Identifier et établir des **méthodes de connectivité** en local et avec le reste de l’environnement cloud Azure. Si le routage forcé avec ExpressRoute est activé, il est probable que les machines virtuelles nécessitent des configurations de proxy appropriées pour traverser le pare-feu d’entreprise.
4. Si les machines virtuelles doivent être **jointes à un domaine** , déterminer si elles rejoignent un domaine basé sur le cloud (Services d’annuaire AAD par exemple) ou un domaine local. Pour les VM en local, déterminer quelle unité d’organisation (UO) dans Active Directory est jointe par les machines virtuelles. En outre, vérifier que les utilisateurs ont accès aux jonctions (ou établir un compte de service qui a la possibilité de créer des enregistrements machine dans le domaine)

## <a name="milestone-2-deploy-the-pilot-lab"></a>Phase 2 : déployer le laboratoire pilote
Une fois la topologie du réseau en place, le premier laboratoire/pilote peut être créé en suivant les étapes suivantes :

1. Créer un environnement DevTest Labs initial.
2. Déterminer les images et les tailles de machine virtuelle autorisées pour une utilisation avec le laboratoire. Décider si les images personnalisées peuvent être téléchargées dans Azure pour une utilisation avec DevTest Labs.
3. Sécurisez l’accès au laboratoire en créant un contrôle d’accès en fonction du rôle Azure initial (Azure RBAC) pour le laboratoire (propriétaires de laboratoires et utilisateurs de laboratoire). Nous recommandons d’utiliser des comptes active directory synchronisés avec Azure Active Directory pour l’identité avec DevTest Labs.
4. Configurer DevTest pour utiliser des stratégies comme les planifications, la gestion des coûts, les machines virtuelles revendicables, les images personnalisées ou les formules.
5. Établir un référentiel en ligne tel que Azure Repos/Git.
6. Décider de l’utilisation de référentiels publics ou privés ou d’une combinaison des deux. Organiser les modèles JSON pour les déploiements et le maintien en état à long terme.
7. Si besoin, créer des artefacts personnalisés. Cette étape est facultative. 

## <a name="milestone-3-documentation-support-learn-and-improve"></a>Phase 3 : documentation, support, apprentissage et amélioration
Les équipes pilotes initiales peuvent nécessiter une prise en charge approfondie pour débuter. Basez-vous sur leurs expériences pour vous assurer que la documentation et le support appropriés sont en place pour un lancement continu d’Azure DevTest Labs.

1. Présenter les nouvelles ressources DevTest Labs (démonstrations, documentation) aux équipes pilotes
2. Selon les expériences des équipes pilote, planifier et fournir une documentation en fonction des besoins
3. Formaliser le processus pour l’intégration des nouvelles équipes (création et configuration des laboratoires, accès, etc.)
4. Selon l’adoption initiale, vérifier que la prévision d’origine de l’espace d’adressage IP est toujours raisonnable et juste
5. S’assurer que les révisions de conformité et de sécurité appropriées ont été conduites

## <a name="next-steps"></a>Étapes suivantes
Voir l’article suivant de cette série : [Gouvernance de l’infrastructure Azure DevTest Labs](devtest-lab-guidance-governance-resources.md)
