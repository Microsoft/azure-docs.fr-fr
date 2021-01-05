---
title: Gouvernance de l’infrastructure Azure DevTest Labs - Ressource
description: Cet article traite de l’alignement et de la gestion des ressources pour Azure DevTest Labs au sein de votre organisation.
ms.topic: article
ms.date: 06/26/2020
ms.reviewer: christianreddington,anthdela,juselph
ms.openlocfilehash: 1e470da5cd317d49f0d0734caa11eed6630d3f32
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85480913"
---
# <a name="governance-of-azure-devtest-labs-infrastructure---resources"></a>Gouvernance de l’infrastructure Azure DevTest Labs – Ressources
Cet article traite de l’alignement et de la gestion des ressources pour DevTest Labs au sein de votre organisation. 

## <a name="align-within-an-azure-subscription"></a>Alignement au sein d’un abonnement Azure 

### <a name="question"></a>Question
Comment aligner des ressources DevTest Labs au sein d’un abonnement Azure ?

### <a name="answer"></a>Réponse
Avant qu’une organisation utilise Azure pour le développement général d’applications, les planificateurs informatiques doivent examiner comment intégrer la fonctionnalité au portefeuille global des services. L’examen doit prendre en compte les problématiques suivantes :

- Comment mesurer les coûts associés au cycle de vie du développement d’applications ?
- Comment l’organisation aligne-t-elle les offres de service proposées sur la stratégie de sécurité de l’entreprise ? 
- La segmentation est-elle nécessaire pour séparer les environnements de développement et de production ? 
- Quels contrôles sont introduits pour faciliter à long terme la gestion, la stabilité et la croissance ?

La **première pratique recommandée** est d’examiner la taxonomie Azure des organisations. Cette taxonomie souligne les divisions entre les abonnements de développement et de production. Dans le diagramme suivant, la taxonomie suggérée permet une séparation logique entre l’environnement de développement/test et celui de production. Grâce à cette approche, une organisation peut introduire des codes de facturation, qui lui permettront d’effectuer un suivi des coûts associés à chaque environnement séparément. Pour plus d’informations, consultez la section sur la [gouvernance normative de l’abonnement](/azure/architecture/cloud-adoption/appendix/azure-scaffold). En outre, vous pouvez utiliser les [balises Azure](../azure-resource-manager/management/tag-resources.md) pour organiser les ressources à des fins de suivi et de facturation.

La **deuxième pratique recommandée** consiste à activer l’abonnement DevTest au sein du portail Azure Enterprise. L’activation permet à une organisation d’exécuter des systèmes d’exploitation clients qui ne sont en principe pas disponibles dans un abonnement Azure Enterprise. Utilisez ensuite des logiciels d’entreprise où vous payez uniquement pour le calcul sans vous soucier des licences. Cela garantit que la facturation pour les services désignés (y compris les images de galerie dans les systèmes IaaS telles que Microsoft SQL Server) est basée sur la consommation uniquement. Pour accéder à plus d’informations sur l’abonnement Azure DevTest, [cliquez ici](https://azure.microsoft.com/offers/ms-azr-0148p/) pour les clients Contrat Entreprise (EA) et [cliquez ici](https://azure.microsoft.com/offers/ms-azr-0023p/) pour les clients Go.

![Alignement des ressources avec des abonnements](./media/devtest-lab-guidance-governance/resource-alignment-with-subscriptions.png)

Ce modèle offre à une organisation la flexibilité nécessaire pour déployer Azure DevTest Labs à grande échelle. Une organisation peut prendre en charge des centaines de laboratoires pour différentes entités commerciales, avec 100 à 1 000 machines virtuelles s’exécutant en parallèle. Le modèle promeut la notion d’une solution de laboratoire d’entreprise centralisée, qui permet de partager les mêmes principes relatifs à la gestion de la configuration et aux contrôles de sécurité.

Ce modèle garantit également que l’organisation n’épuise pas ses limites de ressources associées à son abonnement Azure. Pour plus d’informations sur l’abonnement et les limites du service, consultez [Abonnement Azure et limites, quotas et contraintes du service](../azure-resource-manager/management/azure-subscription-service-limits.md). Le processus d’approvisionnement de DevTest Labs peut utiliser un grand nombre de groupes de ressources. Vous pouvez demander l’augmentation des limites en effectuant une demande de support dans l’abonnement Azure DevTest. Les ressources au sein de l’abonnement de production ne sont pas modifiées à mesure que l’utilisation de l’abonnement de développement augmente. Pour plus d’informations sur le scaling de DevTest Labs, consultez la section [Scaling des quotas et des limites dans DevTest Labs](devtest-lab-scale-lab.md).

Une limite de niveau abonnement courante qui doit être prise en compte est la façon dont les affectations de plage d’adresses IP réseau sont allouées pour prendre en charge les abonnements de développement et de production. Ces affectations doivent expliquer la croissance au fil du temps (en partant du principe qu’il existe une connectivité locale ou une autre topologie de mise en réseau qui oblige l’entreprise à gérer sa pile de mise en réseau au lieu de définir par défaut sur la mise en œuvre d’Azure). La pratique recommandée est d’avoir plusieurs réseaux virtuels qui disposent d’un grand préfixe d’adresse IP. Ce grand préfixe est affecté à plusieurs sous-réseaux volumineux et divisé entre eux. Cette méthode est mieux que d’avoir plusieurs réseaux virtuels avec de petits sous-réseaux. Par exemple, avec 10 abonnements, vous pouvez définir 10 réseaux virtuels (un pour chaque abonnement). Les laboratoires qui ne nécessitent pas d’isolation peuvent partager le même sous-réseau de réseau virtuel de l’abonnement.

## <a name="maintain-naming-conventions"></a>Gérer les conventions d’affectation de noms

### <a name="question"></a>Question
Comment gérer une convention d’affectation de noms au sein de mon environnement DevTest Labs ?

### <a name="answer"></a>Réponse
Vous souhaitez peut-être appliquer les conventions d’affectation de noms actuelles aux opérations d’Azure, et les rendre cohérentes au sein de l’environnement DevTest Labs.

Lors du déploiement de DevTest Labs, nous vous recommandons de disposer de stratégies de démarrage spécifiques. Pour déployer ces stratégies, vous utilisez un script central et des modèles JSON qui permettent d’assurer la cohérence. Les stratégies d’affectation de noms peuvent être implémentées par le biais des stratégies Azure appliquées au niveau de l’abonnement. Pour accéder à des exemples JSON pour Azure Policy, consultez la section [Exemples Azure Policy](../governance/policy/samples/index.md).

## <a name="number-of-users-per-lab-and-labs-per-organization"></a>Nombre d’utilisateurs par laboratoire et nombre de laboratoires par organisation

### <a name="question"></a>Question 
Comment déterminer le taux d’utilisateurs par laboratoire et le nombre total de laboratoires nécessaires dans une organisation ?

### <a name="answer"></a>Réponse
Nous recommandons que les unités commerciales et les groupes de développement associés au même projet de développement soient rattachés au même laboratoire. Cela permet d’appliquer les mêmes types de stratégies, d’images et de stratégies d’arrêt aux deux groupes. 

Vous serez peut-être amené à prendre en compte des limites géographiques. Par exemple, les développeurs du nord-est des États-Unis (US) peuvent utiliser un laboratoire configuré dans USA Est2. Par ailleurs, les développeurs de Dallas (Texas) et Denver (Colorado) peuvent être redirigés afin d’utiliser une ressource de USA Centre Sud. S’il existe un travail collaboratif avec un tiers externe, les développeurs peuvent être affectés à un laboratoire qui n’est pas utilisé par les développeurs internes. 

Vous pouvez également utiliser un labo pour un projet spécifique au sein d’Azure DevOps Projects. Vous appliquez ensuite la sécurité via un groupe Azure Active Directory spécifié, qui autorise l’accès aux deux ensembles de ressources. Le réseau virtuel attribué au laboratoire peut constituer une autre limite permettant de consolider les utilisateurs.

## <a name="deletion-of-resources"></a>Suppression des ressources

### <a name="question"></a>Question
Comment empêcher la suppression des ressources au sein d’un laboratoire ?

### <a name="answer"></a>Réponse
Nous vous recommandons de définir des autorisations appropriées au niveau du laboratoire afin que seuls les utilisateurs autorisés puissent supprimer des ressources ou modifier les stratégies de laboratoire. Les développeurs doivent être placés dans le groupe **DevTest Labs Users** (Utilisateurs DevTest Labs). Le développeur en chef ou le responsable de l’infrastructure doit correspondre au titre de **DevTest Labs Owner** (Propriétaire DevTest Labs). Nous vous conseillons de ne pas avoir plus de deux propriétaires de laboratoire. Cette stratégie s’étend au référentiel de code pour éviter toute corruption. Les utilisateurs du laboratoire sont autorisés à se servir des ressources, mais ne peuvent pas mettre à jour des stratégies de laboratoire. Consultez l’article suivant qui répertorie les rôles et les droits dont chaque groupe intégré dispose au sein d’un laboratoire : [Ajouter des propriétaires et des utilisateurs dans Azure DevTest Labs](devtest-lab-add-devtest-user.md).

## <a name="move-lab-to-another-resource-group"></a>Déplacer le laboratoire vers un autre groupe de ressources 

### <a name="question"></a>Question
Est-il possible de déplacer un laboratoire dans un autre groupe de ressources ?

### <a name="answer"></a>Réponse
Oui. Depuis la page d’accueil du laboratoire, accédez à la page Groupe de ressources. Sélectionnez **Déplacer** dans la barre d’outils. Ensuite, sélectionnez le laboratoire vous souhaitez déplacer vers un autre groupe de ressources. Lorsque vous créez un laboratoire, un groupe de ressources est automatiquement créé pour vous. Toutefois, vous souhaiterez peut-être déplacer le laboratoire vers un autre groupe de ressources qui respecte les conventions d’affectation de noms d’entreprise. 

## <a name="next-steps"></a>Étapes suivantes
Voir la section [Manage cost and ownership](devtest-lab-guidance-governance-cost-ownership.md) (Gérer les coûts et la propriété).
