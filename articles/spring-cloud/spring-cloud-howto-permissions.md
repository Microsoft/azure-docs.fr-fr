---
title: 'Guide pratique : Utilisation d’autorisations dans Azure Spring Cloud'
description: Cet article explique comment créer des rôles personnalisés pour les autorisations dans Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/04/2020
ms.custom: devx-track-java
ms.openlocfilehash: 0c0b3bd81e0f73a7879382e28516378bd722bc17
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96498640"
---
# <a name="how-to-use-permissions-in-azure-spring-cloud"></a>Comment utiliser des autorisations dans Azure Spring Cloud
Cet article explique comment créer des rôles personnalisés qui délèguent des autorisations aux ressources Azure Spring Cloud. Les rôles personnalisés étendent les [rôles intégrés Azure](../role-based-access-control/built-in-roles.md) avec diverses autorisations.

Nous allons implémenter les rôles personnalisés suivants :

* **Rôle Développeur** : 
    * Déployer
    * Test
    * Redémarrer les applications
    * Peut appliquer et modifier les configurations des applications dans le référentiel git
    * Peut récupérer le flux de journal
* **Ops – Ingénierie de fiabilité des sites** : 
    * Redémarrer les applications
    * Récupérer les flux de journaux
    * Impossible de modifier les applications ou les configurations
* **Rôle Azure Pipelines/Jenkins/Github Actions** :
    * Peut effectuer des opérations de création, lecture, mise à jour, suppression
    * Peut créer et configurer tout ce qui se trouve dans Azure Spring Cloud et des applications dans l’instance de service : Azure Pipelines, Jenkins ou GitHub Actions, à l’aide de modèles Resource Manager ou Terraform

## <a name="define-developer-role"></a>Définir le rôle Développeur

Le rôle Développeur comprend les autorisations nécessaires pour redémarrer les applications et afficher leurs flux de journaux, mais ne peut pas apporter de modifications aux applications ni à la configuration.

### <a name="navigate-subscription-and-resource-group-access-control-iam"></a>Parcourir le contrôle d’accès (IAM) des abonnements et des groupes de ressources

Procédez comme suit pour commencer à définir un rôle.

1. Dans le portail Azure, ouvrez l’abonnement et le groupe de ressources dans lesquels vous souhaitez que le rôle personnalisé soit attribuable.
2. Ouvrez **Contrôle d’accès (IAM)** .
3. Cliquez sur **+ Ajouter**.
4. Cliquez sur **Ajouter un rôle personnalisé**.
5. Cliquez sur **Suivant**.

   ![Créer un rôle personnalisé](media/spring-cloud-permissions/create-custom-role.png)

6. Cliquez sur **Ajouter des autorisations**.

   ![Début de l’ajout d’autorisations](media/spring-cloud-permissions/add-permissions.png)

### <a name="search-for-azure-spring-cloud-permissions"></a>Rechercher des autorisations Azure Spring Cloud
7. Dans la zone de recherche, recherchez *Microsoft.app*.
Sélectionnez *Microsoft Azure Spring Cloud*.

   ![Sélectionner Azure Spring Cloud](media/spring-cloud-permissions/spring-cloud-permissions.png)

8. Sélectionnez les autorisations pour le rôle Développeur :

Dans **Microsoft.AppPlatform/Spring**, sélectionnez :
* Écriture : Créer ou mettre à jour une instance de service Azure Spring Cloud
* Lecture : Récupérer une instance de service Azure Spring Cloud
* Autre : Répertorier les clés de test de l’instance de service Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps**, sélectionnez :
* Lecture : Lire l’application Microsoft Azure Spring Cloud
* Autre : Récupérer l’URL de chargement des ressources de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/bindings**, sélectionnez :
* Lecture : Lire la liaison de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/deployments**, sélectionnez :
* Écriture : Écrire le déploiement de l’application Microsoft Azure Spring Cloud
* Lecture : Lire le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Démarrer le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Arrêter le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Redémarrer le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Récupérer l’URL du fichier journal du déploiement de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/domains**, sélectionnez :
* Lecture : Lire le domaine personnalisé de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/certificates**, sélectionnez :
* Lecture : Lire le certificat Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/locations/operationResults/Spring**, sélectionnez :
* Lecture : Lire le résultat de l’opération

Dans **Microsoft.AppPlatform/locations/operationStatus/operationId**, sélectionnez :
* Lecture : Lire l’état de l’opération

    [ ![Créer des autorisations Développeur](media/spring-cloud-permissions/developer-permissions-box.png) ](media/spring-cloud-permissions/developer-permissions-box.png#lightbox)

9. Cliquez sur **Add**.

10. Passez en revue les autorisations.

11. Cliquez sur **Vérifier et créer**.

## <a name="define-devops-engineer-role"></a>Définir le rôle Ingénieur DevOps
Cette procédure définit un rôle disposant des autorisations nécessaires pour déployer, tester et redémarrer les applications Azure Spring Cloud.

1. Répétez la procédure pour accéder au contrôle d’accès (IAM) des abonnements, des groupes de ressources et des accès.
2. Sélectionnez les autorisations pour le rôle Ingénieur DevOps :

Dans **Microsoft.AppPlatform/Spring**, sélectionnez :
* Écriture : Créer ou mettre à jour une instance de service Azure Spring Cloud
* Supprimer : Supprimer une instance de service Azure Spring Cloud
* Lecture : Récupérer une instance de service Azure Spring Cloud
* Autre : Activer un point de terminaison de test de l’instance de service Azure Spring Cloud
* Autre : Désactiver un point de terminaison de test de l’instance de service Azure Spring Cloud
* Autre : Répertorier les clés de test de l’instance de service Azure Spring Cloud
* Autre : Régénérer une clé de test de l’instance de service Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps**, sélectionnez :
* Écriture : Écrire l’application Microsoft Azure Spring Cloud
* Supprimer : Supprimer l’application Microsoft Azure Spring Cloud
* Lecture : Lire l’application Microsoft Azure Spring Cloud
* Autre : Récupérer l’URL de chargement des ressources de l’application Microsoft Azure Spring Cloud
* Autre : Valider le domaine personnalisé de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/bindings**, sélectionnez :
* Écriture : Écrire la liaison de l’application Microsoft Azure Spring Cloud
* Supprimer : Supprimer la liaison de l’application Microsoft Azure Spring Cloud
* Lecture : Lire la liaison de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/deployments**, sélectionnez :
* Écriture : Écrire le déploiement de l’application Microsoft Azure Spring Cloud
* Supprimer : Supprimer le déploiement de l’application Azure Spring Cloud
* Lecture : Lire le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Démarrer le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Arrêter le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Redémarrer le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Récupérer l’URL du fichier journal du déploiement de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/deployments/skus**, sélectionnez :
* Lecture : Répertorier les SKU disponibles pour le déploiement d’application

Dans **Microsoft.AppPlatform/locations**, sélectionnez :
* Autre : Vérifier la disponibilité du nom

Dans Microsoft.AppPlatform/locations/operationResults/Spring, sélectionnez : Lecture : Lire le résultat de l’opération

Dans **Microsoft.AppPlatform/locations/operationStatus/operationId**, sélectionnez :
* Lecture : Lire l’état de l’opération

Dans **Microsoft.AppPlatform/skus**, sélectionnez :
* Lecture : Répertorier les SKU disponibles

   [ ![Autorisations Dev/Op](media/spring-cloud-permissions/dev-ops-permissions.png) ](media/spring-cloud-permissions/dev-ops-permissions.png#lightbox)

3. Cliquez sur **Add**.

4. Passez en revue les autorisations.

5. Cliquez sur **Vérifier et créer**.

## <a name="define-ops---site-reliability-engineering-role"></a>Définir le rôle Ops – Ingénierie de fiabilité des sites
Cette procédure définit un rôle disposant des autorisations nécessaires pour déployer, tester et redémarrer les applications Azure Spring Cloud.

1. Répétez la procédure pour accéder au contrôle d’accès (IAM) des abonnements, des groupes de ressources et des accès.

2. Sélectionnez les autorisations pour le rôle Ops – Ingénierie de fiabilité des sites :

Dans **Microsoft.AppPlatform/Spring**, sélectionnez :
* Lecture : Récupérer une instance de service Azure Spring Cloud
* Autre : Répertorier les clés de test de l’instance de service Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps**, sélectionnez :
* Lecture : Lire l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/apps/deployments**, sélectionnez :
* Lecture : Lire le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Démarrer le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Arrêter le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Redémarrer le déploiement de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/locations/operationResults/Spring**, sélectionnez :
* Lecture : Lire le résultat de l’opération

Dans **Microsoft.AppPlatform/locations/operationStatus/operationId**, sélectionnez :
* Lecture : Lire l’état de l’opération

   [ ![Autorisations Ops/SRE](media/spring-cloud-permissions/ops-sre-permissions.png) ](media/spring-cloud-permissions/ops-sre-permissions.png#lightbox)

3. Cliquez sur **Add**.

4. Passez en revue les autorisations.

5. Cliquez sur **Vérifier et créer**.

## <a name="define-azure-pipelinesprovisioning-role"></a>Définir le rôle Azure Pipelines/Approvisionnement
Ce rôle Jenkins/Github Actions peut créer et configurer tout ce qui se trouve dans Azure Spring Cloud et des applications avec une instance de service. Ce rôle est destiné à libérer ou à déployer du code.

1. Répétez la procédure pour accéder au contrôle d’accès (IAM) des abonnements, des groupes de ressources et des accès.

2. Ouvrez les options **Autorisations**.

3. Sélectionnez les autorisations pour le rôle Azure Pipelines/Approvisionnement :
  
Dans **Microsoft.AppPlatform/Spring**, sélectionnez :
* Écriture : Créer ou mettre à jour une instance de service Azure Spring Cloud
* Supprimer : Supprimer une instance de service Azure Spring Cloud
* Lecture : Récupérer une instance de service Azure Spring Cloud
* Autre : Activer un point de terminaison de test de l’instance de service Azure Spring Cloud
* Autre : Désactiver un point de terminaison de test de l’instance de service Azure Spring Cloud
* Autre : Répertorier les clés de test de l’instance de service Azure Spring Cloud
* Autre : Régénérer une clé de test de l’instance de service Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps**, sélectionnez :
* Écriture : Écrire l’application Microsoft Azure Spring Cloud
* Supprimer : Supprimer l’application Microsoft Azure Spring Cloud
* Lecture : Lire l’application Microsoft Azure Spring Cloud
* Autre : Récupérer l’URL de chargement des ressources de l’application Microsoft Azure Spring Cloud
* Autre : Valider le domaine personnalisé de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/bindings**, sélectionnez :
* Écriture : Écrire la liaison de l’application Microsoft Azure Spring Cloud
* Supprimer : Supprimer la liaison de l’application Microsoft Azure Spring Cloud
* Lecture : Lire la liaison de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/Spring/apps/deployments**, sélectionnez :
* Écriture : Écrire le déploiement de l’application Microsoft Azure Spring Cloud
* Supprimer : Supprimer le déploiement de l’application Azure Spring Cloud
* Lecture : Lire le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Démarrer le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Arrêter le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Redémarrer le déploiement de l’application Microsoft Azure Spring Cloud
* Autre : Récupérer l’URL du fichier journal du déploiement de l’application Microsoft Azure Spring Cloud

Dans **Microsoft.AppPlatform/skus**, sélectionnez :
* Lecture : Répertorier les SKU disponibles

Dans **Microsoft.AppPlatform/locations**, sélectionnez :
* Autre : Vérifier la disponibilité du nom

Dans **Microsoft.AppPlatform/locations/operationResults/Spring**, sélectionnez :
* Lecture : Lire le résultat de l’opération

Dans **Microsoft.AppPlatform/locations/operationStatus/operationId**, sélectionnez :
* Lecture : Lire l’état de l’opération

Dans **Microsoft.AppPlatform/skus**, sélectionnez :
* Lecture : Répertorier les SKU disponibles

   [ ![Autorisations Pipelines](media/spring-cloud-permissions/pipelines-permissions-box.png) ](media/spring-cloud-permissions/pipelines-permissions-box.png#lightbox)  

4. Cliquez sur **Add**.

5. Passez en revue les autorisations.

6. Cliquez sur **Vérifier et créer**.


## <a name="see-also"></a>Voir aussi
* [Créer ou mettre à jour des rôles Azure personnalisés en utilisant le portail Azure](../role-based-access-control/custom-roles-portal.md)

Pour plus d’informations sur les trois méthodes qui permettent de définir une autorisation personnalisée, consultez :
* [Cloner un rôle](../role-based-access-control/custom-roles-portal.md#clone-a-role)
* [Commencer à partir de zéro](../role-based-access-control/custom-roles-portal.md#start-from-scratch)
* [Démarrer à partir d’un JSON](../role-based-access-control/custom-roles-portal.md#start-from-json)