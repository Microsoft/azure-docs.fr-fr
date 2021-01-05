---
title: Activer le studio Azure Machine Learning dans un réseau virtuel
titleSuffix: Azure Machine Learning
description: Découvrez comment configurer Azure Machine Learning studio permet d’accéder aux données stockées au sein d’un réseau virtuel.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.reviewer: larryfr
ms.author: aashishb
author: aashishb
ms.date: 10/21/2020
ms.custom: contperf-fy20q4, tracking-python
ms.openlocfilehash: 3f128b7ee7fa8f690c2097a5d27e274ec1eb2a8a
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97559537"
---
# <a name="use-azure-machine-learning-studio-in-an-azure-virtual-network"></a>Utiliser le studio Azure Machine Learning dans un réseau virtuel Azure

Dans cet article, vous allez apprendre à utiliser le studio Azure Machine Learning dans un réseau virtuel. Le studio inclut des fonctionnalités telles que AutoML, le concepteur et l’étiquetage des données. Pour pouvoir utiliser ces fonctionnalités dans un réseau virtuel, vous devez suivre les étapes décrites dans cet article.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> - Accordez au studio l’accès aux données stockées au sein d’un réseau virtuel.
> - Accédez au studio à partir d’une ressource à l’intérieur d’un réseau virtuel.
> - Découvrez l'impact du studio sur la sécurité du stockage.

Cet article est le cinquième d’une série de cinq qui vous guide à travers le processus de sécurisation d’un workflow Azure Machine Learning. Nous vous recommandons vivement de lire les parties précédentes pour configurer un environnement de réseau virtuel.

Consultez les autres articles de cette série :

[1. Présentation du réseau virtuel](how-to-network-security-overview.md) > [2. Sécurisation de l’espace de travail](how-to-secure-workspace-vnet.md) > [3. Sécurisation de l’environnement d’entraînement](how-to-secure-training-vnet.md) > [4. Sécurisation de l’environnement d’inférence](how-to-secure-inferencing-vnet.md) > **5. Activation de la fonctionnalité de studio**


> [!IMPORTANT]
> Si votre espace de travail se trouve dans un __cloud souverain__, comme Azure Government ou Azure China 21Vianet, les notebooks intégrés _ne prennent pas_ en charge l’utilisation du stockage qui se trouve dans un réseau virtuel. Cela étant, vous pouvez utiliser des notebooks Jupyter à partir d’une instance de calcul. Pour plus d’informations, consultez la section [Accéder aux données d’un notebook d’instance de capacité de calcul](how-to-secure-training-vnet.md#access-data-in-a-compute-instance-notebook).


## <a name="prerequisites"></a>Prérequis

+ Lisez [Vue d’ensemble de la sécurité réseau](how-to-network-security-overview.md) pour comprendre l’architecture des réseaux virtuels et les scénarios associés.

+ Un réseau virtuel et un sous-réseau préexistants à utiliser.

+ Un [espace de travail Azure Machine Learning existant avec Liaison privée activée](how-to-secure-workspace-vnet.md#secure-the-workspace-with-private-endpoint).

+ Un [compte de stockage Azure existant a ajouté votre réseau virtuel](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-service-endpoints).

## <a name="configure-data-access-in-the-studio"></a>Configurer l’accès aux données dans le studio

Certaines des fonctionnalités du studio sont désactivées par défaut dans un réseau virtuel. Pour réactiver ces fonctionnalités, vous devez activer l’identité managée pour les comptes de stockage que vous prévoyez d’utiliser dans le studio. 

Les opérations suivantes sont désactivées par défaut dans un réseau virtuel :

* Aperçu des données dans Studio
* Visualisation des données dans le concepteur
* Déploiement d’un modèle dans le concepteur ([compte de stockage par défaut](#enable-managed-identity-authentication-for-default-storage-accounts))
* Envoi d’une expérience AutoML ([compte de stockage par défaut](#enable-managed-identity-authentication-for-default-storage-accounts))
* Démarrage d’un projet d’étiquetage

Studio prend en charge la lecture de données à partir des types de magasins de données suivants sur un réseau virtuel :

* Objets blob Azure
* Azure Data Lake Storage Gen1
* Azure Data Lake Storage Gen2
* Azure SQL Database

### <a name="configure-datastores-to-use-workspace-managed-identity"></a>Configurer des magasins de données pour utiliser l’identité managée de l’espace de travail

Après avoir ajouté un compte de stockage Azure à votre réseau virtuel avec un [point de terminaison de service](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-service-endpoints) ou un [point de terminaison privé](how-to-secure-workspace-vnet.md#secure-azure-storage-accounts-with-private-endpoints), vous devez configurer votre magasin de données afin qu’il utilise l’authentification via une [identité managée](../active-directory/managed-identities-azure-resources/overview.md). Ainsi, le studio peut accéder aux données de votre compte de stockage.

Azure Machine Learning utilise des [magasins de données](concept-data.md#datastores) pour se connecter aux comptes de stockage. Procédez comme suit pour configurer un magasin de données afin d’utiliser l’identité managée.

1. Dans Studio, sélectionnez __Magasins de données__.

1. Pour mettre à jour un magasin de données existant, sélectionnez-le et sélectionnez __Mettre à jour les informations d’identification__.

    Pour créer un magasin de données, sélectionnez __+ Nouveau magasin de données__.

1. Dans les paramètres de magasin de données, sélectionnez __Oui__ pour __Utiliser l’identité managée d’espace de travail pour afficher un aperçu des données et les profiler dans Azure Machine Learning Studio__.

    ![Capture d’écran montrant comment activer l’identité managée de l’espace de travail](./media/how-to-enable-studio-virtual-network/enable-managed-identity.png)

Ces étapes ajoutent l’identité managée de l’espace de travail en tant que __Lecteur__ au service de stockage à l’aide du contrôle d’accès en fonction du rôle Azure (Azure RBAC). L’accès __Lecteur__ permet à l’espace de travail de récupérer les paramètres du pare-feu et de s’assurer que les données ne quittent pas le réseau virtuel. La prise en compte des modifications peut prendre jusqu’à 10 minutes.

### <a name="enable-managed-identity-authentication-for-default-storage-accounts"></a>Activer l’authentification via une identité managée pour les comptes de stockage par défaut

Chaque espace de travail Azure Machine Learning dispose de deux comptes de stockage par défaut : un compte de stockage BLOB par défaut et un compte de magasin de fichiers par défaut, qui sont définis lors de la création de votre espace de travail. Vous pouvez également définir de nouvelles valeurs par défaut dans la page de gestion **Magasin de données**.

![Capture d’écran montrant où se trouvent les magasins de données par défaut](./media/how-to-enable-studio-virtual-network/default-datastores.png)

Le tableau suivant décrit les raisons pour lesquelles vous devez activer l’authentification via l’identité managée pour les comptes de stockage par défaut de votre espace de travail.

|Compte de stockage  | Notes  |
|---------|---------|
|Stockage blob par défaut de l’espace de travail| Stocke les ressources de modèle à partir du concepteur. Vous devez activer l’authentification via une identité managée sur ce compte de stockage pour déployer des modèles dans le concepteur. <br> <br> Vous pouvez visualiser et exécuter un pipeline de concepteur s’il utilise un magasin de données non défini par défaut qui a été configuré pour utiliser l’identité managée. Toutefois, si vous essayez de déployer un modèle entraîné sans que l’identité managée soit activée sur le magasin de données par défaut, le déploiement échoue, quelles que soient les autres magasins de données en cours d’utilisation.|
|Magasin de fichiers par défaut de l’espace de travail| Stocke les ressources d’expérimentation AutoML. Vous devez activer l’authentification via une identité managée sur ce compte de stockage pour soumettre des expériences AutoML. |

> [!WARNING]
> Il existe un problème connu où le magasin de fichiers par défaut ne crée pas automatiquement le dossier `azureml-filestore`, qui est nécessaire pour soumettre des expériences AutoML. Cela se produit lorsque les utilisateurs importent un magasin de fichiers existant et le définissent par défaut lors de la création de l’espace de travail.
> 
> Pour éviter ce problème, vous avez deux options : 1) Utilisez le magasin de fichiers par défaut qui est automatiquement créé pour vous lors de la création de votre espace de travail. 2) Pour importer votre propre magasin de fichiers, assurez-vous que le magasin de fichiers est en dehors du réseau virtuel pendant la création de l’espace de travail. Une fois l’espace de travail créé, ajoutez le compte de stockage au réseau virtuel.
>
> Pour résoudre ce problème, supprimez le compte de magasin de fichiers du réseau virtuel, puis rajoutez-le au réseau virtuel.


### <a name="grant-workspace-managed-identity-__reader__-access-to-storage-private-link"></a>Accorder à l’identité managée de l’espace de travail un accès __Lecteur__ à la liaison privée de stockage

Si votre compte de stockage Azure utilise un point de terminaison privé, vous devez accorder à l’identité managée de l’espace de travail l’accès **Lecteur** à la liaison privée. Pour plus d’informations, voir le rôle intégré [Lecteur](../role-based-access-control/built-in-roles.md#reader). 

Si votre compte de stockage utilise un point de terminaison de service, vous pouvez ignorer cette étape.

## <a name="access-the-studio-from-a-resource-inside-the-vnet"></a>Accéder au studio à partir d’une ressource au sein d’un réseau virtuel

Si vous accédez à Studio à partir d’une ressource au sein d’un réseau virtuel (par exemple une instance de calcul ou une machine virtuelle), vous devez autoriser le trafic sortant du réseau virtuel vers Studio. 

Par exemple, si vous utilisez des groupes de sécurité réseau pour limiter le trafic sortant, ajoutez une règle à une destination d’__étiquette de service__ __AzureFrontDoor.Frontend__.

## <a name="technical-notes-for-managed-identity"></a>Notes techniques pour l’identité managée

L’utilisation de l’identité managée pour accéder aux services de stockage a un impact sur les considérations en matière de sécurité. Cette section décrit les modifications pour chaque type de compte de stockage. 

Ces considérations sont propres au __type de compte de stockage__ auquel vous accédez.

### <a name="azure-blob-storage"></a>Stockage Blob Azure

Pour __Stockage Blob Azure__, l’identité managée de l’espace de travail est également ajoutée en tant que [Lecteur des données Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader) afin qu’elle puisse lire les données à partir du stockage d’objets Blob.

### <a name="azure-data-lake-storage-gen2-access-control"></a>Contrôle d’accès Azure Data Lake Storage Gen2

Vous pouvez utiliser à la fois des listes de contrôle d’accès (ACL, access-control list) de style POSIX et le contrôle Azure RBAC pour contrôler l’accès aux données au sein d’un réseau virtuel.

Pour utiliser Azure RBAC, ajoutez l’identité managée de l’espace de travail au rôle [Lecteur de données Blob](../role-based-access-control/built-in-roles.md#storage-blob-data-reader). Pour plus d'informations, consultez [Contrôle d'accès en fonction du rôle Azure](../storage/blobs/data-lake-storage-access-control-model.md#role-based-access-control).

Pour utiliser des listes de contrôle d’accès, vous pouvez accorder l’accès à l’identité managée de l’espace de travail comme tout autre principal de sécurité. Pour plus d’informations, consultez [Listes de contrôle d’accès sur les fichiers et répertoires](../storage/blobs/data-lake-storage-access-control.md#access-control-lists-on-files-and-directories).

### <a name="azure-data-lake-storage-gen1-access-control"></a>Contrôle d’accès Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 prend uniquement en charge les listes de contrôle d’accès de type POSIX. Vous pouvez accorder à l’identité managée de l’espace de travail l’accès aux ressources comme n’importe quel autre principal de sécurité. Pour plus d’informations, consultez [Contrôle d’accès dans Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-access-control.md).

### <a name="azure-sql-database-contained-user"></a>Utilisateur autonome Azure SQL Database

Pour accéder aux données stockées dans Azure SQL Database à l’aide d’une identité managée, vous devez créer un utilisateur autonome SQL mappé à l’identité managée. Pour plus d’informations sur la création d’un utilisateur à partir d’un fournisseur externe, consultez [Créer des utilisateurs à relation contenant-contenu mappés à des identités Azure AD](../azure-sql/database/authentication-aad-configure.md#create-contained-users-mapped-to-azure-ad-identities).

Après avoir créé un utilisateur autonome SQL, vous devez lui accorder des autorisations à l’aide de la [commande T-SQL GRANT](/sql/t-sql/statements/grant-object-permissions-transact-sql).

### <a name="azure-machine-learning-designer-intermediate-module-output"></a>Sortie du module intermédiaire du concepteur Azure Machine Learning

Vous pouvez spécifier l’emplacement de sortie de n’importe quel module dans le concepteur. Vous pouvez ainsi stocker les jeux de données intermédiaires dans un emplacement distinct à des fins de sécurité, de journalisation ou d’audit. Pour spécifier la sortie :

1. Sélectionnez le module dont vous souhaitez spécifier la sortie.
1. Dans le volet Paramètres du module qui s’affiche à droite, sélectionnez **Paramètres de sortie**.
1. Spécifiez le magasin de données que vous souhaitez utiliser pour chaque sortie de module.
 
Assurez-vous que vous avez accès aux comptes de stockage intermédiaires de votre réseau virtuel. Dans le cas contraire, le pipeline échoue.

Vous devez également [activer l’authentification via une identité managée](#configure-datastores-to-use-workspace-managed-identity) pour les comptes de stockage intermédiaires afin de visualiser les données de sortie.

## <a name="next-steps"></a>Étapes suivantes

Cet article est une partie facultative d’une série en quatre parties sur les réseaux virtuels. Consultez les autres articles pour découvrir comment sécuriser un réseau virtuel :

* [Partie 1 : Vue d’ensemble des réseaux virtuels](how-to-network-security-overview.md)
* [Partie 2 : Sécuriser les ressources d’espace de travail](how-to-secure-workspace-vnet.md)
* [Partie 3 : Sécuriser l’environnement d’entraînement](how-to-secure-training-vnet.md)
* [Partie 4 : Sécuriser l’environnement d’inférence](how-to-secure-inferencing-vnet.md)