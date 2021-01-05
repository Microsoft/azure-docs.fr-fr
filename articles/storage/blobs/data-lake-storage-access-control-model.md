---
title: Modèle de contrôle d’accès pour Azure Data Lake Storage Gen2 | Microsoft Docs
description: Découvrez comment configurer l’accès au niveau du conteneur, du répertoire et du fichier dans les comptes dotés d’un espace de noms hiérarchique.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: conceptual
ms.date: 11/10/2020
ms.author: normesta
ms.openlocfilehash: 65d1ef76ffae113a4b526eec75301abbfea751e7
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017710"
---
# <a name="access-control-model-in-azure-data-lake-storage-gen2"></a>Modèle de contrôle d’accès dans Azure Data Lake Storage Gen2

Data Lake Storage Gen2 prend en charge les mécanismes d’autorisation suivants :

- Autorisation de clé partagée
- Autorisation de signature d’accès partagé (SAP)
- Contrôle d’accès en fonction du rôle (Azure RBAC)
- Listes de contrôle d’accès (ACL)

[L’autorisation Shared Key et SAS](#shared-key-and-shared-access-signature-sas-authorization) accorde l’accès à un utilisateur (ou une application) sans qu’il n’ait besoin d’une identité dans Azure Active Directory (Azure AD). Avec ces deux formes d’authentification, Azure RBAC et les ACL n’ont aucun effet.

Azure RBAC et ACL nécessitent l’utilisation d’une identité dans Azure AD de l’utilisateur (ou de l’application).  Le RBAC Azure vous permet d’octroyer un accès « grossiste » aux données du compte de stockage, par exemple l’accès en lecture ou en écriture à **toutes** les des données dans un compte de stockage, tandis que les listes de contrôle d’accès vous permettent d’accorder un accès « affiné », tel que l’accès en écriture à un répertoire ou à un fichier spécifique.  

Cet article se concentre sur le RBAC et les ACL Azure et sur la façon dont le système les évalue pour prendre des décisions d’autorisation pour les ressources de compte de stockage.

<a id="role-based-access-control"></a>

## <a name="role-based-access-control-azure-rbac"></a>Contrôle d’accès en fonction du rôle (Azure RBAC)

Azure RBAC utilise des attributions de rôles pour appliquer des ensembles d’autorisations aux [principaux de sécurité](../../role-based-access-control/overview.md#security-principal). Un principal de sécurité est un objet qui représente un utilisateur, un groupe, un principal de service ou une identité managée défini(e) dans Azure Active Directory (AD). Un jeu d’autorisations peut accorder à un principal de sécurité un niveau d’accès à « granularité grossière », tel qu’un accès en lecture ou en écriture à **toutes les** données dans un compte de stockage ou **toutes les** données dans un conteneur. 

Les rôles suivants permettent à un principal de sécurité d’accéder aux données dans un compte de stockage. 

|Rôle|Description|
|--|--|
| [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) | Accès complet aux conteneurs de stockage d’objets BLOB et aux données. Cet accès permet au principal de sécurité de définir le propriétaire d’un élément, et de modifier les listes de contrôle d’accès de tous les éléments. |
| [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) | Lire, écrire et supprimer des conteneurs et objets blob du stockage Azure. Cet accès ne permet pas au principal de sécurité de définir la propriété d’un élément, mais il peut modifier la liste de contrôle d’accès des éléments appartenant au principal de sécurité. |
| [Lecteur des données blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) | Lire et répertorier les conteneurs de stockage Blob et les objets Blob. |

Les rôles tels que [Propriétaire](../../role-based-access-control/built-in-roles.md#owner), [Contributeur](../../role-based-access-control/built-in-roles.md#contributor), [Lecteur](../../role-based-access-control/built-in-roles.md#reader)et [Contributeur de compte de stockage](../../role-based-access-control/built-in-roles.md#storage-account-contributor) permettent à un principal de sécurité de gérer un compte de stockage, mais ne fournissent pas l’accès aux données dans ce compte. Toutefois, ces rôles (à l’exclusion de **Lecteur**) peuvent obtenir l’accès aux clés de stockage, qui peuvent être utilisées dans différents outils clients pour accéder aux données.

## <a name="access-control-lists-acls"></a>Listes ACL

Les listes de contrôle d’accès vous permettent d’appliquer un niveau d’accès à « granularité plus fine » aux répertoires et aux fichiers. Une *liste de contrôle d’accès* est une construction d’autorisation qui contient une série d’*entrées ACL*. Chaque entrée de liste de contrôle d’accès associe le principal de sécurité à un niveau d’accès.  Pour plus d’informations, consultez [les listes de contrôle d’accès (ACL) dans Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).

## <a name="how-permissions-are-evaluated"></a>Évaluation des autorisations

Au cours de l’autorisation basée sur les entités de sécurité, les autorisations sont évaluées dans l’ordre suivant.

:un:&nbsp;&nbsp; Les attributions de rôles Azure sont évaluées en premier et ont priorité sur les attributions de listes de contrôle d’accès.

:deux:&nbsp;&nbsp; Si l’opération est entièrement autorisée en fonction de l’attribution de rôle Azure, les listes de contrôle d’accès ne sont pas évaluées du tout.

:trois:&nbsp;&nbsp; Si l’opération n’est pas entièrement autorisée, les listes de contrôle d’accès sont évaluées.

> [!div class="mx-imgBorder"]
> ![flux d’autorisation Data Lake Storage](./media/control-access-permissions-data-lake-storage/data-lake-storage-permissions-flow.png)

En raison de la façon dont les autorisations d’accès sont évaluées par le système, vous **ne pouvez pas** utiliser une liste de contrôle d’accès pour **limiter** l’accès qui a déjà été accordé par une attribution de rôle. Cela est dû au fait que le système évalue d’abord les attributions de rôles Azure, et si l’attribution accorde une autorisation d’accès suffisante, les ACL sont ignorées. 

Le diagramme suivant illustre le flux d’autorisation pour trois opérations courantes : lister le contenu des répertoires, lire un fichier et écrire un fichier.

> [!div class="mx-imgBorder"]
> ![exemple de flux d’autorisation Data Lake Storage](./media/control-access-permissions-data-lake-storage/data-lake-storage-permissions-example.png)

## <a name="permissions-table-combining-azure-rbac-and-acl"></a>Tableau des autorisations : Combinaison d’Azure RBAC et ACL

Le tableau suivant vous montre comment combiner des rôles Azure et des entrées de liste de contrôle d’accès afin qu’un principal de sécurité puisse effectuer les opérations indiquées dans la colonne **Opération**. Ce tableau présente une colonne qui illustre chaque niveau d’une hiérarchie de répertoires fictifs. Il existe une colonne pour le répertoire racine du conteneur (`/`), un sous-répertoire nommé **Oregon**, un sous-répertoire du répertoire Oregon nommé **Portland** et un fichier texte dans le répertoire Portland nommé **Data. txt**. Dans ces colonnes s’affichent des représentations sous[forme abrégée](data-lake-storage-access-control.md#short-forms-for-permissions) de l’entrée ACL requise pour accorder des autorisations. **N/A** (_pas applicable_) apparaît dans la colonne si aucune entrée de liste de contrôle d’accès n’est requise pour effectuer l’opération.

|    Opération             | Rôle Azure affecté               |    /        | Oregon/     | Portland/ | Data.txt |             
|--------------------------|----------------------------------|-------------|-------------|-----------|----------|
| Lire Data.txt            |   Propriétaire des données Blob du stockage        | N/A      | N/A      | N/A       | N/A    |  
|                          |   Contributeur aux données Blob du stockage  | N/A      | N/A      | N/A       | N/A    |
|                          |   Lecteur des données blob du stockage       | N/A      | N/A      | N/A       | N/A    |
|                          |   None                           | `--X`    | `--X`    | `--X`     | `R--`  |
| Ajouter à Data.txt       |   Propriétaire des données Blob du stockage        | N/A      | N/A      | N/A       | N/A    |
|                          |   Contributeur aux données Blob du stockage  | N/A      | N/A      | N/A       | N/A    |
|                          |   Lecteur des données blob du stockage       | `--X`    | `--X`    | `--X`     | `-W-`  |
|                          |   None                           | `--X`    | `--X`    | `--X`     | `RW-`  |
| Supprimer Data.txt          |   Propriétaire des données Blob du stockage        | N/A      | N/A      | N/A       | N/A    |
|                          |   Contributeur aux données Blob du stockage  | N/A      | N/A      | N/A       | N/A    |
|                          |   Lecteur des données blob du stockage       | `--X`    | `--X`    | `-WX`     | N/A    |
|                          |   None                           | `--X`    | `--X`    | `-WX`     | N/A    |
| Créer Data.txt          |   Propriétaire des données Blob du stockage        | N/A      | N/A      | N/A       | N/A    |
|                          |   Contributeur aux données Blob du stockage  | N/A      | N/A      | N/A       | N/A    |
|                          |   Lecteur des données blob du stockage       | `--X`    | `--X`    | `-WX`     | N/A    |
|                          |   None                           | `--X`    | `--X`    | `-WX`     | N/A    |
| Répertorier /                   |   Propriétaire des données Blob du stockage        | N/A      | N/A      | N/A       | N/A    |
|                          |   Contributeur aux données Blob du stockage  | N/A      | N/A      | N/A       | N/A    |
|                          |   Lecteur des données blob du stockage       | N/A      | N/A      | N/A       | N/A    |
|                          |   None                           | `R-X`    | N/A      | N/A       | N/A    |
| Répertorier /Oregon/            |   Propriétaire des données Blob du stockage        | N/A      | N/A      | N/A       | N/A    |
|                          |   Contributeur aux données Blob du stockage  | N/A      | N/A      | N/A       | N/A    |
|                          |   Lecteur des données blob du stockage       | N/A      | N/A      | N/A       | N/A    |
|                          |   None                           | `--X`    | `R-X`    | N/A       | N/A    |
| Répertorier /Oregon/Portland/   |   Propriétaire des données Blob du stockage        | N/A      | N/A      | N/A       | N/A    |
|                          |   Contributeur aux données Blob du stockage  | N/A      | N/A      | N/A       | N/A    |
|                          |   Lecteur des données blob du stockage       | N/A      | N/A      | N/A       | N/A    |
|                          |   None                           | `--X`    | `--X`    | `R-X`     | N/A    |


> [!NOTE] 
> Pour afficher le contenu d’un conteneur dans l’Explorateur Stockage Azure, les principaux de sécurité doivent [se connecter à l’Explorateur Stockage à l’aide de Azure AD](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#add-a-resource-via-azure-ad)et (au minimum) disposer d’un accès en lecture (R--) au dossier racine (`\`) d’un conteneur. Ce niveau d’autorisation leur donne la possibilité de répertorier le contenu du dossier racine. Si vous ne souhaitez pas que le contenu du dossier racine soit visible, vous pouvez lui attribuer le rôle [Lecteur](../../role-based-access-control/built-in-roles.md#reader). Avec ce rôle, ils peuvent répertorier les conteneurs dans le compte, mais pas le contenu du conteneur. Vous pouvez ensuite accorder l’accès à des répertoires et des fichiers spécifiques à l’aide de listes de contrôle d’accès.   

## <a name="security-groups"></a>Groupes de sécurité

[!INCLUDE [Security groups](../../../includes/azure-storage-data-lake-groups.md)]

## <a name="limits-on-azure-role-assignments-and-acl-entries"></a>Limites sur les affectations de rôle Azure et les entrées ACL

En utilisant des groupes, vous êtes moins susceptible de dépasser le nombre maximal d’attributions de rôles par abonnement et le nombre maximal d’entrées ACL par fichier ou par répertoire. Le tableau suivant décrit ces limites.

[!INCLUDE [Security groups](../../../includes/azure-storage-data-lake-rbac-acl-limits.md)] 

## <a name="shared-key-and-shared-access-signature-sas-authorization"></a>Autorisation de clé partagée et de signature d’accès partagé (SAP)

Azure Data Lake Storage Gen2 prend également en charge les méthodes [Shared Key](/rest/api/storageservices/authorize-with-shared-key) et [SAS](../common/storage-sas-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) pour l’authentification. Une caractéristique de ces méthodes d’authentification est qu’aucune identité n’est associée à l’appelant. Par conséquent, aucune permission basée sur une autorisation de principal de sécurité ne peut être accordée.

Dans le cas d’une clé partagée, l’appelant obtient effectivement l’accès « super utilisateur », ce qui signifie un accès complet à toutes les opérations sur toutes les ressources, notamment les données, la définition du propriétaire et la modification des ACL.

Les jetons SAP incluent les autorisations accordées dans le jeton. Les autorisations incluses dans le jeton SAP sont appliquées à toutes les décisions d’autorisation, mais aucune vérification ACL supplémentaire n’est effectuée.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur les listes de contrôle d’accès, consultez [les listes de contrôle d’accès (ACL) dans Azure Data Lake Storage Gen2](data-lake-storage-access-control.md).