---
title: Fournir une preuve de concept - Azure DevTest Labs | Microsoft Docs
description: Découvrez comment fournir une preuve de concept visant à bien intégrer Azure DevTest Labs dans un environnement d’entreprise.
ms.topic: article
ms.date: 06/2/2020
ms.openlocfilehash: 9c28cf9eebd8a39a2edce48e4fb8b96dc7608d80
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87288025"
---
# <a name="deliver-a-proof-of-concept"></a>Fournir une preuve de concept 

L’un des principaux scénarios d'Azure DevTest Labs consiste à permettre des environnements de développement et de test dans le cloud. Voici quelques exemples :

* Création de bureaux de développeur dans le cloud.
* Configuration d’environnements de test.
* Activation de l’accès aux machines virtuelles et à d’autres ressources Azure.
* Configuration d’une zone de bac à sable pour permettre aux développeurs d’apprendre et d’expérimenter.

DevTest Labs fournit également des rails de sécurité pour les stratégies et des contrôles des coûts permettant à l’entreprise de proposer un « libre-service Azure» aux développeurs qui adhèrent aux stratégies de l'entreprise en matière de sécurité, de réglementation et de conformité. 

Chaque entreprise a des exigences différentes en termes d'intégration d'Azure DevTest Labs à son environnement. Cet article décrit les étapes les plus courantes que les entreprises doivent effectuer pour assurer la réussite d’une preuve de concept. Une preuve de concept est la première étape d’un déploiement de bout en bout réussi. 

## <a name="getting-started"></a>Prise en main 

Pour commencer à fournir une preuve de concept. Il est important de prendre le temps de découvrir Azure et DevTest Labs.  Pour ce faire, voici quelques ressources utiles : 

* [Compréhension du portail Azure](https://azure.microsoft.com/features/azure-portal/)
* [Notions de base sur DevTest Labs](devtest-lab-overview.md)
* [Scénarios pris en charge par DevTest Labs](devtest-lab-guidance-get-started.md)
* [Documentation DevTest Labs pour les entreprises](devtest-lab-guidance-prescriptive-adoption.md)
* [Introduction à la mise en réseau Azure](../virtual-network/virtual-networks-overview.md)

## <a name="prerequisites"></a>Prérequis 

Pour mener à bien un pilote ou une preuve de concept avec DevTest Labs, il existe plusieurs prérequis : 

* **Abonnement Azure** : Les entreprises disposent souvent d’un [Contrat Entreprise](https://azure.microsoft.com/pricing/purchase-options/enterprise-agreement/) qui permet d’accéder à Azure, et elles peuvent utiliser un abonnement existant ou nouveau pour DevTest Labs. Les entreprises peuvent également utiliser un [abonnement Visual Studio](https://azure.microsoft.com/pricing/member-offers/visual-studio-subscriptions/) pendant la phase pilote (en profitant des crédits Azure gratuits). Si aucune de ces options n’est disponible, une entreprise peut créer et utiliser un [compte Azure gratuit](https://azure.microsoft.com/free/search/?&OCID=AID719825_SEM_g4lyBqgB&lnkd=Bing_Azure_Brand&msclkid=ecc4275a31b61375749e7a5322c20de8&dclid=CMGW5-m78-ICFaLt4QodmUwGtQ). Si un Contrat Entreprise est en place, l'utilisation d'un [ abonnement Enterprise Dev/Test](https://azure.microsoft.com/offers/ms-azr-0148p/) est une excellente option pour accéder aux systèmes d'exploitation clients Windows 10/Windows 8.1 et bénéficier de tarifs réduits pour les charges de travail de développement et de test. 
* **Locataire Azure Active Directory** : Pour activer la gestion des utilisateurs (par exemple, l’ajout d’utilisateurs ou de propriétaires de labo), ces utilisateurs doivent faire partie du [locataire Azure Active Directory](https://azure.microsoft.com/services/active-directory/) utilisé dans l’abonnement Azure pour le pilote. Les entreprises configurent souvent l’[identité hybride ](../active-directory/hybrid/whatis-hybrid-identity.md) pour permettre aux utilisateurs d’utiliser leur identité locale dans le cloud, mais ce n’est pas nécessaire pour le pilote DevTest Labs. 

## <a name="scoping-of-the-pilot"></a>Étendue du pilote 

Il est important de planifier un pilote avant de commencer l’implémentation. Savoir à l’avance que les ressources ne seront pas conservées indéfiniment permet de définir des attentes claires pour les utilisateurs du pilote. 

> [!IMPORTANT]
> Nous n’insisterons jamais assez sur l’importance de bien définir l’étendue du pilote et les attentes en amont.

Répondez aux questions clés suivantes avant de lancer le pilote : 

* Que souhaitez-vous apprendre et comment envisagez-vous la réussite du pilote ? 
* Quels scénarios ou charges de travail le pilote couvrira-t-il ? Il est important d'en limiter le nombre pour bien définir le pilote et le mener à bien rapidement. 
* Quelles ressources doivent être disponibles dans le lab ? Par exemple : images personnalisées, images de la Place de marché, stratégies, topologie de réseau. 
* Qui sont les utilisateurs et les équipes impliqués dans le pilote pour vérifier l’expérience ?  
* Quelle est la durée du pilote ? Choisissez une plage de temps adaptée à l’étendue planifiée, comme deux semaines ou un mois. 
* Une fois la phase pilote terminée, que deviendront les ressources allouées qui ont été utilisées pendant la phase pilote ? Envisagez-vous de supprimer les ressources du pilote ? Vous pouvez penser :
   
   « S’il est possible de planifier la suppression des machines virtuelles et labos au terme de la phase pilote, il est possible de configurer un abonnement unique pour le pilote et d’y effectuer notre travail tout en résolvant les questions de déploiement d’échelle en parallèle. » 

Il est courant de vouloir créer le pilote « parfait » afin qu’il représente de manière précise l’état final du service une fois celui-ci déployé dans l’entreprise. Ce n'est pas la meilleure solution. Plus vous visez la « perfection », plus votre charge de travail est importante *avant* de pouvoir utiliser le pilote. L'objectif du pilote consiste à prendre les bonnes décisions en matière d'extension et de déploiement du service final. 

Le pilote doit permettre de choisir les charges de travail et dépendances minimales nécessaires afin de déterminer si Azure DevTest Labs est adapté à votre entreprise. Pour garantir le bon déroulement du pilote dans les meilleurs délais, nous vous recommandons de choisir la charge de travail la plus simple dotée du plus petit nombre de dépendances possible. Si ce n’est pas possible, choisissez la charge de travail la plus représentative en termes de complexités potentielles pour permettre de répliquer la phase pilote lors de la phase de scale-out. 

L’exemple suivant illustre une preuve de concept bien définie. 

## <a name="example-proof-of-concept-plan"></a>Exemple : Plan de preuve de concept 

Cette section présente un exemple à suivre afin de définir une preuve de concept de pilote pour DevTest Labs. 

> [!TIP]
> Pour limiter le risque d’une défaillance de votre projet, nous vous recommandons vivement de ne pas ignorer l’exemple décrit dans cette section. 

### <a name="overview"></a>Vue d’ensemble 

Nous envisageons de développer un nouvel environnement dans Azure basé sur DevTest Labs dont les fournisseurs se serviront en tant qu’environnement isolé du réseau d’entreprise. Pour savoir si la solution répondra aux exigences, nous allons développer une preuve de concept afin de valider la solution de bout en bout. Nous avons inclus plusieurs fournisseurs pour expérimenter et vérifier l’expérience. 

### <a name="outcomes"></a>Résultats 

Lors de la génération d’une preuve de concept, nous nous concentrons d’abord sur les résultats (ce que nous essayons d’atteindre). Au terme de la preuve de concept, nous espérons obtenir ce qui suit : 

* Solution opérationnelle de bout en bout permettant aux fournisseurs d’utiliser des comptes invités dans Azure Active Directory (Azure AD) pour accéder à un environnement isolé dans Azure. L’environnement est doté des ressources nécessaires pour qu’ils soient productifs. 
* Tous les problèmes de blocage potentiels qui affectent une utilisation et une adoption à plus grande échelle sont énumérés et compris.
* Les personnes chargées de développer la preuve de concept ont une bonne compréhension du code. Elles comprennent également les éléments collatéraux impliqués et se montrent confiantes quant à une plus large adoption.

### <a name="open-questions-and-prerequisites"></a>Questions ouvertes et prérequis 

* Disposons-nous d’un abonnement que nous pouvons utiliser pour ce projet ? 
* Disposons-nous d’un locataire Azure AD et d’un administrateur général Azure AD identifié capable de fournir aide et conseils pour les questions liées à Azure AD ? 
* Disposons-nous d’un espace de collaboration pour les personnes travaillant sur le projet ? 

   * Code source et scripts (Azure Repos, par exemple) 
   * Documents (Microsoft Teams ou SharePoint, par exemple)  
   * Conversations (Microsoft Teams, par exemple) 
   * Éléments de travail (Azure Boards, par exemple) 
* Quelles sont les ressources requises pour les fournisseurs ? Cela englobe les applications disponibles sur le réseau, aussi bien localement sur les machines virtuelles que sur d’autres serveurs nécessaires. 
* Les machines virtuelles seront-elles jointes à un domaine dans Azure ? Si tel est le cas, s’agira-t-il d’Azure Active Directory Domain Services (Azure AD DS) ou d’autre chose ? 
* Avez-vous identifié l’équipe ou les fournisseurs comme cible de la preuve de concept ? Quels seront les clients de l’environnement ?
* Quelle région Azure utiliserons-nous pour la preuve de concept ? 
* Disposons-nous de la liste des services que les fournisseurs sont autorisés à utiliser via DevTest Labs en plus de IaaS (machines virtuelles) ? 
* Comment envisageons-nous d’entraîner les fournisseurs/utilisateurs sur l’utilisation du labo ? 

### <a name="components-of-the-proof-of-concept-solution"></a>Composants de la solution de preuve de concept 

Nous attendons de la solution les composants suivants : 

* Différentes équipes de fournisseurs utiliseront un ensemble de labos dans Azure.
* Les labos sont connectés à une infrastructure réseau qui prend en charge la configuration requise.
* Les fournisseurs ont accès aux labos par le biais d’Azure AD et d’attributions de rôles.
* Les fournisseurs disposent d’un moyen de se connecter à leurs ressources. Plus précisément, un VPN de site à site permet d’accéder directement aux machines virtuelles, sans adresses IP publiques.
* Un ensemble d’artefacts couvre les logiciels exigés dont les fournisseurs ont besoin sur les machines virtuelles.

## <a name="additional-planning-and-design-decisions"></a>Décisions supplémentaires relatives à la planification et à la conception 

Avant de publier une solution DevTest Labs complète, vous devez prendre plusieurs décisions importantes relatives à la planification et à la conception. Votre expérience en matière de preuve de concept peut vous y aider. Autres points à prendre en compte : 

* **Topologie de l’abonnement** : Les exigences au niveau de l’entreprise concernant les ressources dans Azure peuvent s’étendre au-delà des [quotas disponibles dans un seul abonnement](../azure-resource-manager/management/azure-subscription-service-limits.md). Cela nécessite plusieurs abonnements Azure et/ou demandes de service pour augmenter les limites de l’abonnement initial. Il est important de décider à l’avance comment distribuer les ressources entre les abonnements. Le [guide de décision concernant les abonnements](/azure/architecture/cloud-adoption/decision-guides/subscriptions/) est très utile car il est difficile de déplacer ultérieurement des ressources vers un autre abonnement. À titre d’exemple, un labo ne peut pas être déplacé vers un autre abonnement après avoir été créé.  
* **Topologie du réseau** : L’[infrastructure réseau par défaut](../app-service/networking-features.md) que DevTest Labs crée automatiquement peut ne pas répondre pleinement aux exigences et contraintes des utilisateurs de l’entreprise. Il est courant de rencontrer des [réseaux virtuels connectés à Azure ExpressRoute](/azure/architecture/reference-architectures/hybrid-networking/), des [réseaux en étoile](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) pour une connectivité entre les abonnements, et même un [routage forcé](../vpn-gateway/vpn-gateway-forced-tunneling-rm.md) pour garantir une connectivité locale uniquement. DevTest Labs permet de connecter des réseaux virtuels existants au labo pour que vous puissiez les utiliser quand vous créez des machines virtuelles dans ce labo. 
* **Accès à distance à des machines virtuelles** : De nombreuses options permettent d'accéder à distance aux machines virtuelles situées dans DevTest Labs. La plus simple consiste à utiliser des adresses IP publiques ou des adresses IP publiques partagées. Il s’agit de [paramètres disponibles dans le labo](devtest-lab-shared-ip.md). Si ces options ne suffisent pas, l’utilisation d’une passerelle d’accès à distance est également possible. Cette option est illustrée dans l’[architecture de référence d’entreprise DevTest Labs](devtest-lab-reference-architecture.md) et décrite en détail dans la [documentation relative à la passerelle Bureau à distance DevTest Labs](configure-lab-remote-desktop-gateway.md). Les entreprises peuvent également utiliser ExpressRoute ou un VPN de site à site pour connecter leur labo à leur réseau local. Cela permet les connexions Bureau à distance ou SSH directes aux machines virtuelles en fonction de leur adresse IP privée, sans aucune exposition à Internet. 
* **Gestion des autorisations** : Les deux autorisations clés couramment utilisées dans DevTest Labs sont [Propriétaire et Utilisateur de laboratoire](devtest-lab-add-devtest-user.md). Avant de déployer DevTest Labs à grande échelle, il est important de décider qui sera en charge de chaque niveau d’accès du lab. Souvent, le détenteur du budget (responsable d’équipe, par exemple) est le propriétaire du labo et les membres de l’équipe sont les utilisateurs du labo. Ce modèle permet à la personne (responsable d’équipe) en charge du budget d’ajuster les paramètres de stratégie et de gérer le budget de l’équipe.  

## <a name="completing-the-proof-of-concept"></a>Finalisation de la preuve de concept 

Une fois ces différents points abordés, le moment est venu de finaliser le pilote. Il est temps de recueillir les commentaires des utilisateurs, de déterminer si le pilote a réussi et de décider si l’organisation déploiera DevTest Labs à grande échelle dans l’entreprise. C’est également le moment idéal pour envisager le déploiement automatisé de DevTest Labs et des ressources associées afin de garantir une certaine cohérence tout au long du déploiement à grande échelle. 

## <a name="next-steps"></a>Étapes suivantes 

* [Documentation DevTest Labs pour les entreprises](devtest-lab-guidance-prescriptive-adoption.md)
* [Architecture de référence pour une entreprise](devtest-lab-reference-architecture.md)
* [Réalisation d’un scale-up de votre déploiement DevTest Labs](devtest-lab-guidance-orchestrate-implementation.md)
* [Orchestrer l’implémentation d’Azure DevTest Labs](devtest-lab-guidance-orchestrate-implementation.md)
