---
title: Rôles et autorisations pour Azure Data Factory
description: Cet article décrit les rôles et autorisations requis pour créer des fabriques de données et travailler avec des ressources enfants.
ms.date: 11/5/2018
ms.topic: conceptual
ms.service: data-factory
services: data-factory
documentationcenter: ''
ms.workload: data-services
author: dcstwh
ms.author: weetok
manager: anandsub
ms.openlocfilehash: 70be8d8be48f2b1e1cc275c06e4abff09e3e62f6
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498572"
---
# <a name="roles-and-permissions-for-azure-data-factory"></a>Rôles et autorisations pour Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]


Cet article décrit les rôles requis pour créer et gérer des ressources Azure Data Factory ainsi que les autorisations accordées par ces rôles.

## <a name="roles-and-requirements"></a>Rôles et conditions requises

Pour créer des instances Data Factory, le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être membre des rôles *Contributeur* ou *Propriétaire*, ou *administrateur* de l’abonnement Azure. Pour visualiser les autorisations dont vous disposez dans l’abonnement, sélectionnez votre nom d’utilisateur dans le coin supérieur droit du Portail Azure, puis sélectionnez **Autorisations**. Si vous avez accès à plusieurs abonnements, sélectionnez l’abonnement approprié. 

Les exigences applicables à la création et à la gestion des ressources enfants pour Data Factory (jeux de données, services liés, pipelines, déclencheurs et runtimes d’intégration) sont les suivantes :
- Pour créer et gérer des ressources enfants dans le portail Azure, vous devez appartenir au rôle **Contributeur de Data Factory** au niveau du **groupe de ressources** ou à un niveau supérieur.
- Pour créer et gérer des ressources enfants à l’aide de PowerShell ou du Kit de développement logiciel (SDK), le rôle **Contributeur** au niveau du groupe de ressources ou à un niveau supérieur est suffisant.

Pour découvrir des exemples d’instructions concernant l’ajout d’un utilisateur à un rôle, consultez l’article décrivant comment [ajouter des rôles](../cost-management-billing/manage/add-change-subscription-administrator.md).

## <a name="set-up-permissions"></a>Définir des autorisations

Après avoir créé une fabrique de données, vous allez sans doute vouloir permettre à d’autres utilisateurs de l’utiliser. Pour accorder cet accès à d’autres utilisateurs, vous devez les ajouter au rôle **Contributeur de Data Factory**, lequel est intégré au **groupe de ressources** qui contient la fabrique de données.

### <a name="scope-of-the-data-factory-contributor-role"></a>Portée du rôle Contributeur de Data Factory

L’appartenance au rôle **Contributeur de Data Factory** permet aux utilisateurs d’effectuer les opérations suivantes :
- Créer, modifier et supprimer des fabriques de données et des ressources enfants, y compris des jeux de données, des services liés, des pipelines, des déclencheurs et des runtimes d’intégration.
- Déployer des modèles Resource Manager. Le déploiement Resource Manager est la méthode de déploiement utilisée par Data Factory dans le portail Microsoft Azure.
- Gérer les alertes App Insights pour une fabrique de données.
- Créer des tickets de support.

Pour plus d’informations sur ce rôle, voir [Rôle Contributeur de Data Factory](../role-based-access-control/built-in-roles.md#data-factory-contributor).

### <a name="resource-manager-template-deployment"></a>Déploiement de modèle Resource Manager

Le rôle **Contributeur de Data Factory**, au niveau du groupe de ressources ou au-delà, permet aux utilisateurs de déployer des modèles Resource Manager. Par conséquent, les membres de ce rôle peuvent utiliser des modèles Resource Manager pour déployer des fabriques de données et leurs ressources enfants, y compris des jeux de données, des services liés, des pipelines, des déclencheurs et des runtimes d’intégration. L’appartenance à ce rôle ne permet pas à l’utilisateur de créer d’autres ressources.

Les autorisations pour les référentiels Azure et GitHub sont indépendantes des autorisations Data Factory. Par conséquent, un utilisateur disposant des autorisations de référentiel et uniquement membre du rôle Lecteur peut modifier les ressources enfant Data Factory et apporter des modifications au référentiel, mais il ne peut pas publier ces modifications.

> [!IMPORTANT]
> Le déploiement du modèle Resource Manager avec le rôle **Contributeur de Data Factory** ne permet pas d’accroître vos autorisations. Par exemple, si vous déployez un modèle qui crée une machine virtuelle Azure et si vous n’êtes pas autorisé à créer des machines virtuelles, le déploiement échoue et renvoie une erreur d’autorisation.

### <a name="custom-scenarios-and-custom-roles"></a>Scénarios et rôles personnalisés

Parfois, il peut se révéler utile d’accorder différents niveaux d’accès selon les utilisateurs de la fabrique de données. Par exemple :
- Vous avez peut-être besoin d’un groupe dans lequel les utilisateurs disposent uniquement d’autorisations sur une fabrique de données spécifique.
- Ou bien, vous avez peut-être besoin d’un groupe dans lequel les utilisateurs peuvent uniquement surveiller une ou plusieurs fabriques de données, mais pas les modifier.

Pour ces scénarios personnalisés, vous devez créer des rôles personnalisés et les assigner aux utilisateurs. Pour plus d’informations sur les rôles personnalisés, voir [Rôles personnalisés dans Azure](..//role-based-access-control/custom-roles.md).

Voici quelques exemples qui illustrent l’intérêt d’utiliser des rôles personnalisés :

- Vous permettez à un utilisateur de créer, modifier ou supprimer une fabrique de données dans un groupe de ressources à partir du portail Microsoft Azure.

  Vous attribuez le rôle intégré **Contributeur de Data Factory** à l’utilisateur au niveau du groupe de ressources. Si vous souhaitez autoriser l’accès à une fabrique de données dans un abonnement, attribuez le rôle au niveau de l’abonnement.

- Vous laissez un utilisateur afficher (lire) et surveiller une fabrique de données, mais vous ne l’autorisez pas à l’éditer ou à la modifier.

  Vous attribuez le rôle **lecteur** à l’utilisateur dans la ressource de fabrique de données.

- Vous permettez à un utilisateur de modifier une seule fabrique de données dans le portail Microsoft Azure.

  Ce scénario nécessite deux attributions de rôles.

  1. Vous attribuez le rôle **Contributeur** au niveau de la fabrique de données.
  2. Vous créez un rôle personnalisé avec l'autorisation **Microsoft.Resources/deployments/** . Vous attribuez ce rôle personnalisé à l’utilisateur au niveau du groupe de ressources.

- Vous permettez à un utilisateur de tester la connexion dans un service lié ou d’afficher un aperçu des données dans un jeu de données

    Créez un rôle personnalisé avec des autorisations pour les actions suivantes : **Microsoft.DataFactory/factories/getFeatureValue/read** et **Microsoft.DataFactory/factories/getDataPlaneAccess/action**. Attribuez ce rôle personnalisé à l’utilisateur dans la ressource de fabrique de données.

- Vous permettez à un utilisateur de mettre à jour une fabrique de données à partir de PowerShell ou du SDK, mais pas dans le portail Microsoft Azure.

  Vous attribuez le rôle **Contributeur** à l’utilisateur pour la ressource de fabrique de données. Ce rôle permet à l’utilisateur de voir les ressources dans le portail Azure, mais pas d’accéder aux boutons **Publier** et **Publier tout**.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les rôles dans Azure : [Comprendre les définitions de rôles](../role-based-access-control/role-definitions.md)

- En savoir plus sur le rôle **Contributeur de Data Factory** : [Rôle Contributeur de Data Factory](../role-based-access-control/built-in-roles.md#data-factory-contributor).
