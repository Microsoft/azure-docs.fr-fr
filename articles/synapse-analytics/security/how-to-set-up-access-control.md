---
title: Guide pratique pour configurer le contrôle d’accès pour votre espace de travail Synapse
description: Cet article explique comment contrôler l’accès à un espace de travail Synapse à l’aide des rôles Azure, des rôles Synapse, des autorisations SQL et des autorisations Git.
services: synapse-analytics
author: billgib
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 12/03/2020
ms.author: billgib
ms.reviewer: jrasnick
ms.openlocfilehash: 62c30356017b5ea5d93351e6f22b8b7b0c22718c
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109264"
---
# <a name="how-to-set-up-access-control-for-your-synapse-workspace"></a>Guide pratique pour configurer le contrôle d’accès pour votre espace de travail Synapse 

Cet article explique comment contrôler l’accès à un espace de travail Synapse à l’aide des rôles Azure, des rôles Synapse, des autorisations SQL et des autorisations Git.   

Dans ce guide, vous allez configurer un espace de travail et configurer un système de contrôle d’accès de base adapté à de nombreux projets Synapse.  Vous découvrirez ensuite des options avancées pour un contrôle plus fin si vous en avez besoin.  

Le contrôle d’accès Synapse peut être simplifié à l’aide de groupes de sécurité alignés sur les rôles et les personnages de votre organisation.  Il vous suffit d’ajouter et de supprimer des utilisateurs des groupes de sécurité pour gérer l’accès.

Avant de commencer cette procédure pas à pas, consultez la [Vue d’ensemble du contrôle d’accès Synapse](./synapse-workspace-access-control-overview.md) pour vous familiariser avec les mécanismes de contrôle d’accès utilisés par synapse.   

## <a name="access-control-mechanisms"></a>Mécanismes de contrôle d’accès

> [!NOTE]
> L’approche adoptée dans ce guide consiste à créer plusieurs groupes de sécurité, puis à attribuer des rôles à ces groupes. Une fois les groupes configurés, il vous suffit de gérer l’appartenance aux groupes de sécurité pour contrôler l’accès à l’espace de travail.

Pour sécuriser un espace de travail Synapse, vous devez suivre un modèle de configuration des éléments suivants :

- Des **groupes de sécurité**, pour regrouper les utilisateurs ayant des exigences d’accès similaires
- Des **rôles Azure**, pour contrôler qui peut créer et gérer des pools SQL, des pools Apache Spark et des runtimes d’intégration, et accéder au stockage ADLS Gen2
- Des **rôles Synapse**, pour contrôler l’accès aux artefacts de code publiés, l’utilisation des ressources de calcul Apache Spark et les runtimes d’intégration 
- Des **autorisations SQL**, pour contrôler l’accès des plans de données et d’administration aux pools SQL 
- Des **autorisations Git**, pour contrôler qui peut accéder aux artefacts de code dans le contrôle de code source si vous configurez la prise en charge de Git pour l’espace de travail 
 
## <a name="steps-to-secure-a-synapse-workspace"></a>Procédure de sécurisation d’un espace de travail Synapse

Ce document utilise des noms standard pour simplifier les instructions. Remplacez-les par les noms de votre choix.

|Paramètre | Nom standard | Description |
| :------ | :-------------- | :---------- |
| **Espace de travail Synapse** | `workspace1` |  Nom de l’espace de travail Synapse. |
| **Compte ADLSGEN2** | `storage1` | Compte ADLS à utiliser avec votre espace de travail. |
| **Conteneur** | `container1` | Conteneur dans STG1 que l’espace de travail utilisera par défaut. |
| **Locataire Active directory** | `contoso` | Nom du locataire Azure Active actif.|
||||

## <a name="step-1-set-up-security-groups"></a>ÉTAPE 1 : Configurer des groupes de sécurité

>[!Note] 
>Au cours de la préversion, nous vous recommandons de créer des groupes de sécurité mappés aux rôles Synapse **Administrateur Synapse SQL** et **Administrateur Apache Spark Synapse**.  Avec l’introduction de nouveaux rôles RBAC et étendues Synapse plus précis, nous vous recommandons désormais d’utiliser ces nouvelles fonctionnalités pour contrôler l’accès à votre espace de travail.  Ces nouveaux rôles et étendues offrent une plus grande flexibilité en matière de configuration et reconnaissent le fait que les développeurs utilisent souvent une combinaison de SQL et Spark pour créer des applications d’analytique et qu’ils peuvent devoir accéder à des ressources spécifiques plutôt qu’à l’espace de travail entier. [En savoir plus](./synapse-workspace-synapse-rbac.md) sur RBAC Synapse.

Créez les groupes de sécurité suivants pour votre espace de travail :

- **`workspace1_SynapseAdministrators`** , pour les utilisateurs qui ont besoin d’un contrôle total sur l’espace de travail.  Ajoutez-vous à ce groupe de sécurité, tout au moins au début
- **`workspace1_SynapseContributors`** , pour les développeurs qui doivent développer, déboguer et publier du code sur le service   
- **`workspace1_SynapseComputeOperators`** , pour les utilisateurs qui doivent gérer et superviser des pools Apache Spark et des runtimes d’intégration
- **`workspace1_SynapseCredentialUsers`** , pour les utilisateurs qui doivent déboguer et exécuter des pipelines d’orchestration à l’aide des informations d’identification MSI (Managed Service Identity) de l’espace de travail et annuler des exécutions de pipeline   

Vous assignerez bientôt des rôles Synapse à ces groupes au niveau de l’étendue de l’espace de travail.  

Créez également ce groupe de sécurité : 
- **`workspace1_SQLAdmins`** , groupe pour les utilisateurs qui ont besoin d’une autorité d’administration SQL Active Directory dans les pools SQL de l’espace de travail. 

Vous utiliserez le groupe `workspace1_SQLAdmins` lorsque vous configurerez des autorisations SQL dans des pools SQL lors de leur création. 

Pour une configuration de base, ces cinq groupes suffisent. Plus tard, vous pourrez ajouter des groupes de sécurité pour gérer les utilisateurs qui ont besoin d’un accès plus spécialisé ou pour accorder aux utilisateurs un accès uniquement à des ressources spécifiques.

> [!NOTE]
>- Découvrez comment créer un groupe de sécurité réseau en consultant [cet article](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-create-azure-portal).
>- Découvrez comment ajouter un groupe de sécurité à partir d’un autre groupe de sécurité en consultant [cet article](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-groups-membership-azure-portal).

>[!Tip]
>Les utilisateurs Synapse peuvent utiliser Azure Active Directory dans le portail Azure pour consulter leurs appartenances aux groupes afin de déterminer les rôles qui leur ont été accordés.

## <a name="step-2-prepare-your-adls-gen2-storage-account"></a>ÉTAPE 2 : Préparer votre compte de stockage ADLS Gen2

Un espace de travail Synapse utilise un conteneur de stockage par défaut pour les besoins suivants :
  - Stockage des fichiers de données de stockage pour les tables Spark.
  - Journaux d’exécution pour les travaux Spark.
  - Gestion des bibliothèques que vous choisissez d’installer.

Identifiez les informations suivantes relatives à votre stockage :

- Le compte ADLS Gen2 à utiliser pour votre espace de travail. Il est nommé `storage1` dans le présent document. `storage1` est considéré comme le compte de stockage « principal » pour votre espace de travail
- Le conteneur à l’intérieur de `workspace1` que votre espace de travail Synapse utilisera par défaut. Il est nommé `container1` dans le présent document 

- Dans le portail Azure, attribuez les rôles Azure suivants sur `container1` aux groupes de sécurité 

  - Attribuez le rôle **Contributeur aux données Blob du stockage** à `workspace1_SynapseAdmins` 
  - Attribuez le rôle **Contributeur aux données Blob du stockage** à `workspace1_SynapseContributors`
  - Attribuez le rôle **Contributeur aux données Blob du stockage** à `workspace1_SynapseComputeOperators`

## <a name="step-3-create-and-configure-your-synapse-workspace"></a>ÉTAPE 3 : Créer et configurer votre espace de travail Synapse

Dans le portail Azure, créez un espace de travail Synapse :

- Sélectionnez votre abonnement
- Sélectionnez ou créez un groupe de ressources pour lequel vous disposez du rôle **Propriétaire** Azure.
- Nommez l’espace de travail `workspace1`.
- Choisissez `storage1` comme compte de stockage.
- Choisissez `container1` comme conteneur utilisé en tant que « système de fichiers ».
- Ouvrez WS1 dans Synapse Studio.
- Accédez à **Gérer** > **Contrôle d’accès** et attribuez des rôles Synapse à l’*étendue de l’espace de travail* aux groupes de sécurité comme suit :
  - Attribuez le rôle **Administrateur Synapse** à `workspace1_SynapseAdministrators`. 
  - Attribuez le rôle **Contributeur Synapse** à `workspace1_SynapseContributors`. 
  - Attribuez le rôle **Opérateur de calcul Synapse** à `workspace1_SynapseComputeOperators`.

## <a name="step-4-grant-the-workspace-msi-access-to-the-default-storage-container"></a>ÉTAPE 4 : Accorder à l’espace de travail un accès MSI au conteneur de stockage par défaut

Pour exécuter des pipelines et effectuer des tâches système, Synapse exige que l’identité de service managée (MSI) de l’espace de travail ait accès à `container1` dans le compte ADLS Gen2 par défaut.

- Ouvrez le portail Azure
- Recherchez le compte de stockage, `storage1`, puis `container1`.
- À l’aide de **Contrôle d’accès (IAM)** , vérifiez que le rôle **Contributeur aux données Blob du stockage** est attribué à l’identité MSI de l’espace de travail.
  - Si ce n’est pas le cas, attribuez-le.
  - Le MSI porte le même nom que l’espace de travail. Dans cet article, il s’agirait de `workspace1`.

## <a name="step-5-grant-synapse-administrators-the-azure-contributor-role-on-the-workspace"></a>ÉTAPE 5 : Accorder aux administrateurs de Synapse le rôle Contributeur Azure sur l’espace de travail 

Pour créer des pools SQL, des pools Apache Spark et des runtimes d’intégration, les utilisateurs doivent avoir au moins un accès Contributeur Azure à l’espace de travail. Le rôle Contributeur permet également à ces utilisateurs de gérer les ressources, y compris la suspension et la mise à l’échelle.

- Ouvrez le portail Azure
- Recherchez l’espace de travail, `workspace1`.
- Attribuez le rôle **Contributeur Azure** sur `workspace1` à `workspace1_SynapseAdministrators`. 

## <a name="step-6-assign-sql-active-directory-admin-role"></a>ÉTAPE 6 : Attribuer à SQL le rôle d’Administrateur Active Directory

Le créateur de la station de travail est configuré automatiquement comme administrateur SQL Active Directory pour l’espace de travail.  Ce rôle ne peut être accordé qu’à un utilisateur ou un groupe unique. Dans cette étape, vous allez affecter l’administrateur SQL Active Directory de l’espace de travail au groupe de sécurité `workspace1_SQLAdmins`.  L’attribution de ce rôle donne à ce groupe un accès administrateur hautement privilégié à tous les pools SQL et à toutes les bases de données de l’espace de travail.   

- Ouvrez le portail Azure
- Accédez à `workspace1`.
- Sous **Paramètres**, sélectionnez **Administrateur SQL Active Directory**.
- Sélectionnez **Définir l’administrateur**, puis choisissez **`workspace1_SQLAdmins`** .

>[!Note]
>L’étape 6 est facultative.  Vous pouvez choisir d’accorder au groupe `workspace1_SQLAdmins` un rôle moins privilégié. Pour attribuer le rôle `db_owner` ou d’autres rôles SQL, vous devez exécuter des scripts sur chaque base de données SQL. 

## <a name="step-7-grant-access-to-sql-pools"></a>ÉTAPE 7 : Accorder l’accès aux pools SQL

Par défaut, tous les utilisateurs disposant du rôle Administrateur Synapse disposent aussi du rôle `db_owner` SQL sur le pool SQL serverless, « Intégré », et toutes ses bases de données.

L’accès aux pools SQL pour d’autres utilisateurs et pour l’identité MSI de l’espace de travail est contrôlé à l’aide d’autorisations SQL.  L’attribution d’autorisations SQL nécessite l’exécution de scripts SQL sur chaque bases de données SQL après la création.  Il existe trois cas qui exigent l’exécution de ces scripts :
1. Octroi à d’autres utilisateurs de l’accès au pool SQL serverless, « Intégré », et à ses bases de données
2. Octroi à tout utilisateur de l’accès à des bases de données de pools dédiés
3. Octroi à l’identité MSI de l’espace de travail de l’accès à une base de données de pool SQL afin de permettre aux pipelines qui nécessitent un accès au pool SQL de s’exécuter correctement

Vous trouverez ci-dessous des exemples de scripts SQL.

Pour accorder l’accès à une base de données de pool SQL dédié, les scripts peuvent être exécutés par le créateur de l’espace de travail ou par n’importe quel membre du groupe `workspace1_SQLAdmins`.  

Pour accorder l’accès au pool SQL serverless, « Intégré », les scripts peuvent être exécutés par n’importe quel membre du groupe `workspace1_SQLAdmins` ou du groupe `workspace1_SynapseAdministrators`. 

> [!TIP]
> Les étapes ci-dessous doivent être exécutées pour **chaque** pool SQL afin d’accorder à l’utilisateur un accès à toutes les bases de données SQL, sauf dans la section [Autorisation au niveau de l’espace de travail](#workspace-scoped-permission) où vous pouvez attribuer un rôle sysadmin au niveau de l’espace de travail à l’utilisateur.

### <a name="step-71-serverless-sql-pool-built-in"></a>ÉTAPE 7.1 : Pool SQL serverless, Intégré

Dans cette section, il existe des exemples de script montrant comment accorder à un utilisateur l’autorisation d’accéder à une base de données particulière ou à toutes les bases de données dans le pool SQL serverless, « Intégré ».

> [!NOTE]
> Dans les exemples de script, remplacez *alias* par l’alias de l’utilisateur ou du groupe auquel l’accès est accordé, et *domain* par le domaine d’entreprise que vous utilisez.

#### <a name="database-scoped-permission"></a>Autorisation au niveau de la base de données

Pour accorder à un utilisateur l’accès à une base de données SQL serverless **unique**, suivez les étapes de cet exemple :

1. Créer des informations de connexion (LOGIN)

    ```sql
    use master
    go
    CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
    go
    ```

2. Créer un utilisateur (USER)

    ```sql
    use yourdb -- Use your database name
    go
    CREATE USER alias FROM LOGIN [alias@domain.com];
    ```

3. Ajouter USER aux membres du rôle spécifié

    ```sql
    use yourdb -- Use your database name
    go
    alter role db_owner Add member alias -- Type USER name from step 2
    ```

#### <a name="workspace-scoped-permission"></a>Autorisation au niveau de l’espace de travail

Pour accorder un accès complet à **tous** les pools SQL serverless de l’espace de travail, utilisez le script dans cet exemple :

```sql
use master
go
CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
ALTER SERVER ROLE sysadmin ADD MEMBER [alias@domain.com];
```

### <a name="step-72-dedicated-sql-pools"></a>ÉTAPE 7.2 : Pools SQL dédiés

Pour accorder l’accès à une **seule** base de données de pool SQL dédié, effectuez les étapes suivantes dans l’éditeur de script Synapse SQL :

1. Créez l’utilisateur dans la base de données en exécutant la commande suivante sur la base de données cible, sélectionnée à l’aide de la liste déroulante *Se connecter à* :

    ```sql
    --Create user in the database
    CREATE USER [<alias@domain.com>] FROM EXTERNAL PROVIDER;
    ```

2. Accordez à l’utilisateur un rôle pour accéder à la base de données :

    ```sql
    --Grant role to the user in the database
    EXEC sp_addrolemember 'db_owner', '<alias@domain.com>';
    ```

> [!IMPORTANT]
> *db_datareader* et *db_datawriter* peuvent fonctionner pour les autorisations en lecture/écriture si vous ne souhaitez pas accorder l’autorisation *db_owner*.
> Pour qu’un utilisateur Spark puisse lire et écrire directement à partir de Spark dans ou à partir d’un pool SQL, l’autorisation *db_owner* est nécessaire.

Après avoir créé les utilisateurs, exécutez des requêtes pour vérifier que le pool SQL serverless peut interroger le compte de stockage.

### <a name="step-73-sql-access-control-for-synapse-pipeline-runs"></a>ÉTAPE 7.3 : Contrôle d’accès SQL pour les exécutions de pipelines Synapse

### <a name="workspace-managed-identity"></a>Identité managée de l’espace de travail

> [!IMPORTANT]
> Pour exécuter des pipelines comprenant des jeux de données ou des activités qui référencent un pool SQL, l’identité de l’espace de travail doit pouvoir accéder au pool SQL.

Exécutez les commandes suivantes sur chaque pool SQL pour autoriser l’identité système managée de l’espace de travail à exécuter des pipelines sur les bases de données de pools SQL :  

>[!note]
>Dans les scripts ci-dessous, pour une base de données de pool SQL dédié, databasename correspond au nom du pool.  Pour une base de données dans le pool SQL serverless « Intégré », databasename correspond au nom de la base de données.

```sql
--Create a SQL user for the workspace MSI in database
CREATE USER [<workspacename>] FROM EXTERNAL PROVIDER;

--Granting permission to the identity
GRANT CONTROL ON DATABASE::<databasename> TO <workspacename>;
```

Cette autorisation peut être supprimée en exécutant le script suivant sur le même pool SQL :

```sql
--Revoke permission granted to the workspace MSI
REVOKE CONTROL ON DATABASE::<databasename> TO <workspacename>;

--Delete the workspace MSI user in the database
DROP USER [<workspacename>];
```

## <a name="step-8-add-users-to-security-groups"></a>ÉTAPE 8 : Ajouter des utilisateurs à des groupes de sécurité

La configuration initiale de votre système de contrôle d’accès est terminée.

Pour gérer l’accès, vous pouvez ajouter et supprimer des utilisateurs aux groupes de sécurité que vous avez configurés.  Vous pouvez affecter manuellement des utilisateurs à des rôles Synapse, mais cela n’a pas pour effet de configurer leurs autorisations de manière cohérente. Au lieu de cela, limitez-vous à ajouter ou supprimer des utilisateurs dans les groupes de sécurité.

## <a name="step-9-network-security"></a>ÉTAPE 9 : Sécurité réseau

En guise d’ultime étape pour sécuriser votre espace de travail, vous devez sécuriser l’accès réseau à l’aide de :
- [Pare-feu d’espace de travail](./synapse-workspace-ip-firewall.md)
- [Réseau virtuel managé](./synapse-workspace-managed-vnet.md) 
- [Points de terminaison privés](./synapse-workspace-managed-private-endpoints.md)
- [Liaison privée](../../azure-sql/database/private-endpoint-overview.md)

## <a name="step-10-completion"></a>ÉTAPE 10 : Completion

Votre espace de travail est maintenant entièrement configuré et sécurisé.

## <a name="supporting-more-advanced-scenarios"></a>Prise en charge de scénarios plus avancés

Ce guide est axé sur la configuration d’un système de contrôle d’accès de base. Vous pouvez prendre en charge des scénarios plus avancés en créant des groupes de sécurité supplémentaires et en attribuant à ces groupes des rôles plus précis à des étendues plus spécifiques. Prenez les cas suivants :

**Activez la prise en charge de Git** pour l’espace de travail pour des scénarios de développement plus avancés, notamment CI/CD.  En mode Git, les autorisations Git déterminent si un utilisateur peut valider les modifications apportées à sa branche de travail.  La publication sur le service a lieu uniquement à partir de la branche de collaboration.  Il peut être judicieux de créer un groupe de sécurité pour les développeurs qui doivent développer et déboguer des mises à jour dans une branche de travail mais qui n’ont pas besoin de publier des modifications dans le service actif.

**Limitez l’accès des développeurs** à des ressources spécifiques.  Créez des groupes de sécurité plus fins pour les développeurs qui ont besoin d’accéder uniquement à des ressources spécifiques.  Attribuez à ces groupes des rôles Synapse appropriés qui sont limités à des pools Spark, des runtimes d’intégration ou des informations d’identification spécifiques.

**Empêchez les opérateurs d’accéder aux artefacts de code**.  Créez des groupes de sécurité pour les opérateurs qui doivent superviser l’état opérationnel des ressources de calcul Synapse et afficher les journaux, mais qui n’ont pas besoin d’accéder au code ou de publier des mises à jour du service. Attribuez à ces groupes le rôle d’Opérateur de calcul, étendu à des pools Spark et des runtimes d’intégration spécifiques.  

## <a name="next-steps"></a>Étapes suivantes

Découvrez [comment gérer les attributions de rôles RBAC Synapse](./how-to-manage-synapse-rbac-role-assignments.md) et créez un [espace de travail Synapse](../quickstart-create-workspace.md).
