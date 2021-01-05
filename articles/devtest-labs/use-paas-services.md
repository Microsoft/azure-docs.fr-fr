---
title: Utiliser des services PaaS (Platform-as-a-Service) dans Azure DevTest Labs
description: Découvrez comment utiliser des services PaaS (Platform-as-a-Service) dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: eec37527386098174906dc2737d7b763241da3f2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85478737"
---
# <a name="use-platform-as-a-service-paas-services-in-azure-devtest-labs"></a>Utiliser des services PaaS (Platform-as-a-Service) dans Azure DevTest Labs
PaaS est pris en charge dans DevTest Labs via les environnements. Dans DevTest Labs, les environnements sont pris en charge par les modèles préconfigurés Azure Resource Manager dans un référentiel Git. Les environnements peuvent contenir des ressources PaaS et IaaS. Ainsi, vous pouvez créer des systèmes complexes pouvant inclure des ressources Azure telles que des machines virtuelles, bases de données, réseaux virtuels et applications web, qui sont personnalisés pour fonctionner ensemble. Ces modèles permettent un déploiement cohérent et améliorent la gestion des environnements à l’aide du contrôle de code source. 

Correctement configuré, un système permet les scénarios suivants : 

- Développeurs ayant plusieurs environnements indépendants
- Test sur différentes configurations de manière asynchrone
- Intégration à des pipelines de préproduction et de production sans modifications des modèles
- Les machines et environnements de développement au sein du même lab améliorent la gestion et les rapports sur les coûts.  

Le fournisseur de ressources DevTest Labs crée des ressources pour le compte de l'utilisateur du lab, et aucune autorisation supplémentaire n'est à attribuer à cet utilisateur pour utiliser les ressources PaaS. L’image suivante illustre un cluster Service Fabric en tant qu’un environnement dans le lab.

![Cluster Service Fabric en tant qu'environnement](./media/create-environment-service-fabric-cluster/cluster-created.png)

Pour plus d'informations sur la configuration des environnements, consultez [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md). DevTest Labs dispose d'un référentiel public de modèles Azure Resource Manager, que vous pouvez utiliser pour créer des environnements sans devoir vous connecter à une source GitHub externe par vous-même. Pour plus d’informations sur les environnements publics, consultez [Configurer et utiliser des environnements publics dans Azure DevTest Labs](devtest-lab-configure-use-public-environments.md).

Au sein des grandes organisations, les équipes de développement fournissent généralement des environnements tels que les environnements de test personnalisés/isolés. Certains environnements peuvent être utilisés par toutes les équipes d'une unité commerciale ou d'une division. Le groupe informatique peut vouloir mettre ses environnements à la disposition de toutes les équipes de l'organisation.  

## <a name="customizations"></a>Personnalisations

#### <a name="sandbox"></a>Bac à sable 
Le propriétaire du lab peut personnaliser des environnements lab pour faire évoluer le rôle de l’utilisateur de **lecteur** à **contributeur** au sein du groupe de ressources. Cette fonctionnalité est disponible dans **Paramètres du lab**, sous **Configuration et stratégies**. Ce changement de rôle permet à l’utilisateur d'ajouter ou de supprimer des ressources au sein de l'environnement. Si vous souhaitez davantage restreindre l’accès, utilisez les stratégies Azure. Cette fonctionnalité vous permet de personnaliser les ressources ou la configuration sans accès au niveau de l’abonnement.

#### <a name="custom-tokens"></a>Jetons personnalisés
Certaines informations de lab personnalisé se trouvent en dehors du groupe de ressources et sont spécifiques aux environnements auxquels le modèle peut accéder. En voici quelques-unes : 

- Identification du réseau du lab
- Location
- Compte de stockage dans lequel sont stockés des modèles Resource Manager. 
 
#### <a name="lab-virtual-network"></a>Réseau virtuel du lab
L'article [Connexion d'environnements au réseau virtuel du lab](connect-environment-lab-virtual-network.md) explique comment modifier votre modèle Resource Manager pour utiliser le jeton `$(LabSubnetId)`. Lorsqu’un environnement est créé, le jeton `$(LabSubnetId)` est remplacé par la première marque de sous-réseau où l'option **use in virtual machine create** est définie sur **true**. Elle permet à l'environnement d'utiliser les réseaux précédemment créés. Pour utiliser les même modèles Resource Manager dans les environnements de test, préproduction et production, utilisez `$(LabSubnetId)` en tant que valeur par défaut dans un paramètre de modèle Resource Manager. 

#### <a name="environment-storage-account"></a>Compte de stockage d’environnement
DevTest Labs prend en charge l’utilisation de [modèles Resource Manager imbriqués](../azure-resource-manager/templates/linked-templates.md). L'article [[Déployer des modèles Azure Resource Manager imbriqués pour les environnements de test](deploy-nested-template-environments.md) explique comment utiliser `_artifactsLocation` et les jetons `_artifactsLocationSasToken` pour créer un URI vers un modèle Resource Manager dans le même dossier ou dans un dossier imbriqué du modèle principal. Pour plus d’informations sur ces deux jetons, consultez la section **Artefacts de déploiement** du [Guide des meilleures pratiques - Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/blob/master/1-CONTRIBUTION-GUIDE/best-practices.md).

## <a name="user-experience"></a>Expérience utilisateur

## <a name="developer"></a>Développeur
Les développeurs utilisent le même flux de travail pour créer une machine virtuelle que créer un environnement spécifique. Ils sélectionnent l’environnement par rapport à l'image de machine et entrent les informations requises par le modèle. Chaque développeur disposant d'un environnement permet le déploiement des modifications et le débogage amélioré de la boucle interne. L’environnement peut être créé à tout moment à l’aide du dernier modèle.  Cette fonctionnalité permet la destruction puis la recréation des environnements afin de réduire les temps d'arrêt nécessaires à la création manuelle du système ou à sa restauration après un test des défaillances.  

### <a name="testing"></a>Test
Les environnements DevTest Labs permettent de tester indépendamment du code et des configurations spécifiques de manière asynchrone. En règle générale, l'environnement est configuré avec le code issu de la requête de tirage (pull request) individuelle et lance n'importe quel test automatisé. Une fois le test automatisé terminé, un test manuel peut être exécuté par rapport à l'environnement spécifique. Ce processus est généralement suivi dans le cadre du pipeline CI/CD. 

## <a name="management-experience"></a>Expérience de gestion

### <a name="cost-tracking"></a>Suivi des coûts
La fonctionnalité de suivi des coûts inclut des ressources Azure au sein des différents environnements dans le cadre de la tendance des coûts globaux. Le coût par ressources ne répartit pas les différentes ressources dans l'environnement mais affiche l'environnement sous forme de coût unique.

### <a name="security"></a>Sécurité
Correctement configuré, un abonnement Azure avec DevTest Labs peut [limiter l’accès aux ressources Azure par le biais du lab](devtest-lab-add-devtest-user.md). Avec les environnements, un propriétaire de lab peut permettre aux utilisateurs d’accéder aux ressources PaaS avec des configurations approuvées, sans autoriser l'accès aux autres ressources Azure. Dans un scénario où les utilisateurs de lab personnalisent les environnements, le propriétaire du lab peut autoriser un accès contributeur. L'accès contributeur permet à l'utilisateur du lab d'ajouter ou de supprimer des ressources Azure uniquement au sein du groupe de ressources managées. Il facilite le suivi et la gestion sans accès contributeur à l'abonnement.

### <a name="automation"></a>Automatisation
L'automatisation est un composant clé d'un écosystème efficace à grande échelle. L'automatisation est nécessaire pour assurer la gestion et le suivi de plusieurs environnements dans différents abonnements et labs.

### <a name="cicd-pipeline"></a>Pipeline CI/CD
Dans DevTest Labs, les services PaaS contribuent à améliorer le pipeline CI/CD moyennant des déploiements concentrés avec accès contrôlé par le lab.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur les environnements, consultez les articles suivants : 

- 
- [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md)
- [Configurer et utiliser des environnements publics dans Azure DevTest Labs](devtest-lab-configure-use-public-environments.md)
- [Créer un environnement avec un cluster Service Fabric autonome dans Azure DevTest Labs](create-environment-service-fabric-cluster.md)
- [Connecter un environnement au réseau virtuel de votre lab dans Azure DevTest Labs](connect-environment-lab-virtual-network.md)
- [Intégrer des environnements à vos pipelines CI/CD Azure DevOps](integrate-environments-devops-pipeline.md)
 





