---
title: Gérer Azure Data Lake Analytics à l’aide du portail Azure
description: Cet article décrit comment utiliser le portail Azure pour gérer des comptes, des sources de données, des utilisateurs et des travaux Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 12/05/2016
ms.openlocfilehash: c1057478d9ae7d63bef9477a0b1297fa2c507ffa
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92218423"
---
# <a name="manage-azure-data-lake-analytics-using-the-azure-portal"></a>Gérer Azure Data Lake Analytics à l’aide du portail Azure
[!INCLUDE [manage-selector](../../includes/data-lake-analytics-selector-manage.md)]

Cet article décrit comment gérer des comptes, des sources de données, des utilisateurs et des travaux Azure Data Lake Analytics à l’aide du portail Azure.


<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-data-lake-analytics-accounts"></a>Gérer les comptes Data Lake Analytics

### <a name="create-an-account"></a>Créer un compte

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Cliquez sur **Créer une ressource** > **Intelligence + analyse**  > **Data Lake Analytics** .
3. Sélectionnez des valeurs pour les éléments suivants : 
   1. **Name**  : nom du compte Data Lake Analytics.
   2. **Abonnement** : abonnement Azure utilisé pour le compte.
   3. **Groupe de ressources** : groupe de ressources Azure dans lequel créer le compte. 
   4. **Emplacement** : centre de données Azure pour le compte Data Lake Analytics. 
   5. **Data Lake Store** : Store par défaut à utiliser pour le compte Data Lake Analytics. Le compte Azure Data Lake Store et le compte Data Lake Analytics doivent se trouver dans le même emplacement.
4. Cliquez sur **Créer** . 

### <a name="delete-a-data-lake-analytics-account"></a>Supprimer un compte Data Lake Analytics

Avant de supprimer un compte Data Lake Analytics, vous devez supprimer le compte Data Lake Store dépendant.

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Cliquez sur **Supprimer** .
3. Entez le nom du compte.
4. Cliquez sur **Supprimer** .

<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-data-sources"></a>Gérer les sources de données

Data Lake Analytics prend en charge les sources de données suivantes :

* Data Lake Store
* Stockage Azure

Vous pouvez utiliser l’Explorateur de données pour parcourir les sources de données et effectuer des opérations de gestion des fichiers de base. 

### <a name="add-a-data-source"></a>Ajouter une source de données

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Cliquez sur **Sources de données** .
3. Cliquez sur **Ajouter une source de données** .
    
   * Pour ajouter un compte Data Lake Store, vous avez besoin du nom du compte et de l’accès au compte pour pouvoir l’interroger.
   * Pour ajouter du stockage Blob Azure, vous avez besoin du compte de stockage et de la clé du compte. Pour les trouver, accédez au compte de stockage dans le portail.

## <a name="set-up-firewall-rules"></a>Configurer des règles de pare-feu

Vous pouvez utiliser Data Lake Analytics pour mieux verrouiller l’accès à votre compte Data Lake Analytics au niveau du réseau. Vous pouvez activer un pare-feu, spécifier une adresse IP ou définir une plage d’adresses IP pour vos clients approuvés. Une fois ces mesures activées, seuls les clients possédant des adresses IP dans la plage définie peuvent se connecter au Store.

Si d’autres services Azure, comme Azure Data Factory ou des machines virtuelles, se connectent au compte Data Lake Analytics, vérifiez que l’option **Autoriser les services Azure** est **activée** . 

### <a name="set-up-a-firewall-rule"></a>Configurer une règle de pare-feu

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Dans le menu de gauche, cliquez sur **Pare-feu** .

## <a name="add-a-new-user"></a>Ajouter un nouvel utilisateur

Vous pouvez utiliser **l’Assistant Ajout d’un utilisateur** pour configurer facilement de nouveaux utilisateurs Data Lake.

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Sur la gauche, sous **Prise en main** , cliquez sur **Assistant Ajout d’un utilisateur** .
3. Sélectionnez un utilisateur, puis cliquez sur **Enregistrer** .
4. Sélectionnez un rôle, puis cliquez sur **Enregistrer** . Pour configurer un nouveau développeur qui utilisera Azure Data Lake, sélectionnez le rôle **Développeur Data Lake Analytics** .
5. Sélectionnez les listes de contrôle d’accès (ACL) pour les bases de données U-SQL. Lorsque vous êtes satisfait de vos choix, cliquez sur **Sélectionner** .
6. Sélectionnez les listes de contrôle d’accès pour les fichiers. Pour le Store par défaut, ne modifiez pas les listes de contrôle d’accès pour le dossier racine « / » et pour le dossier /system. Cliquez sur **Sélectionner** .
7. Passez en revue toutes vos sélections, puis cliquez sur **Exécuter** .
8. Une fois l’Assistant exécuté, cliquez sur **Terminé** .

## <a name="manage-azure-role-based-access-control"></a>Gérer le contrôle d’accès en fonction du rôle Azure

Comme d’autres services Azure, vous pouvez utiliser le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour contrôler la façon dont les utilisateurs interagissent avec le service.

Les rôles Azure standard ont les fonctionnalités suivantes :
* **Propriétaire** : peut envoyer des travaux, surveiller des travaux, annuler des travaux de n’importe quel utilisateur et configurer le compte.
* **Collaborateur** : peut envoyer des travaux, surveiller des travaux, annuler des travaux de n’importe quel utilisateur et configurer le compte.
* **Lecteur** : peut surveiller les travaux.

Utilisez le rôle Développeur Data Lake Analytics pour permettre aux développeurs U-SQL d’utiliser le service Data Lake Analytics. Vous pouvez utiliser le rôle Développeur Data Lake Analytics pour les tâches suivantes :
* Envoyer des travaux.
* Surveiller l’état du travail et la progression des travaux soumis par les utilisateurs.
* Afficher les scripts U-SQL à partir des travaux soumis par les utilisateurs.
* Annuler uniquement vos propres travaux.

### <a name="add-users-or-security-groups-to-a-data-lake-analytics-account"></a>Ajouter des utilisateurs ou des groupes de sécurité à un compte Data Lake Analytics

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Cliquez sur **Contrôle d’accès (IAM)**  > **Ajouter une attribution de rôle** .
3. Sélectionnez un rôle
4. Ajoutez un utilisateur.
5. Cliquez sur **OK** .

>[!NOTE]
>Si un utilisateur ou un groupe de sécurité doit envoyer des travaux, ils doivent également être autorisés sur le compte du Store. Pour plus d’informations, consultez [Sécuriser les données stockées dans Data Lake Store](../data-lake-store/data-lake-store-secure-data.md).
>

<!-- ################################ -->
<!-- ################################ -->

## <a name="manage-jobs"></a>Gestion des travaux

### <a name="submit-a-job"></a>Soumettre un travail

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.

2. Cliquez sur **Nouveau travail** . Pour chaque travail, configurez les éléments suivants :

    1. **Nom du travail** : Nom du travail.
    2. **Priorité** : les nombres inférieurs ont une priorité supérieure. Si deux travaux sont en file d’attente, celui dont la valeur de la priorité est la plus faible s’exécute en premier.
    3. **Parallélisme** : nombre maximal de processus de calcul à réserver pour ce travail.

3. Cliquez sur **Envoyer le travail** .

### <a name="monitor-jobs"></a>Surveiller des travaux

1. Dans le portail Azure, accédez à votre compte Data Lake Analytics.
2. Cliquez sur **Afficher tous les travaux** . Une liste de tous les travaux actifs et récemment terminés dans le compte s’affiche.
3. Le cas échéant, cliquez sur **Filtrer** pour rechercher les travaux par **Intervalle de temps** , **Nom du travail** et **Auteur** . 

### <a name="monitoring-pipeline-jobs"></a>Surveillance des tâches de pipeline
Les tâches qui font partie d’un pipeline fonctionnent ensemble, généralement séquentiellement, pour réaliser un scénario spécifique. Par exemple, vous pouvez avoir un pipeline qui nettoie, extrait, transforme, regroupe l’utilisation des informations du client. Les tâches de pipeline sont identifiées à l’aide de la propriété « Pipeline » lorsque la tâche a été soumise. Les tâches planifiées à l’aide du fichier de définition d’application V2 auront automatiquement cette propriété remplie. 

Pour afficher une liste des tâches U-SQL qui font partie de pipelines : 

1. Dans le portail Azure, accédez à vos comptes Data Lake Analytics.
2. Cliquez sur **Informations sur les tâches** . L’onglet « Toutes les tâches » s’affiche par défaut avec la liste des tâches en cours d’exécution, en file d’attente et terminées.
3. Cliquez sur l’onglet **Tâches de pipeline** . Une liste des tâches de pipeline s’affiche, ainsi que des statistiques agrégées pour chaque pipeline.

### <a name="monitoring-recurring-jobs"></a>Surveillance des tâches récurrentes
Une tâche récurrente a la même logique métier, mais utilise les données d’entrée différente chaque fois qu’elle est exécutée. Dans l’idéal, les tâches récurrentes doivent toujours réussir et ont une exécution relativement stable ; La surveillance de ces comportements permet de garantir que la tâche est intègre. Les tâches récurrentes sont identifiées à l’aide de la propriété « Récurrence ». Les tâches planifiées à l’aide du fichier de définition d’application V2 auront automatiquement cette propriété remplie.

Pour afficher une liste des tâches U-SQL récurrentes : 

1. Dans le portail Azure, accédez à vos comptes Data Lake Analytics.
2. Cliquez sur **Informations sur les tâches** . L’onglet « Toutes les tâches » s’affiche par défaut avec la liste des tâches en cours d’exécution, en file d’attente et terminées.
3. Cliquez sur l’onglet **Tâches récurrentes** . Une liste des tâches récurrentes s’affiche, ainsi que des statistiques agrégées pour chaque tâche récurrente.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation d’Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Gestion d’Azure Data Lake Analytics à l’aide d’Azure PowerShell](data-lake-analytics-manage-use-powershell.md)
* [Gérer Azure Data Lake Analytics à l’aide de stratégies](data-lake-analytics-account-policies.md)
