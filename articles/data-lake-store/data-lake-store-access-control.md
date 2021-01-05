---
title: Vue d’ensemble du contrôle d’accès dans Data Lake Storage Gen1 | Microsoft Docs
description: Découvrez les principes de base du modèle de contrôle d’accès d’Azure Data Lake Storage Gen1, qui dérive de HDFS.
services: data-lake-store
documentationcenter: ''
author: twooley
manager: mtillman
editor: cgronlun
ms.assetid: d16f8c09-c954-40d3-afab-c86ffa8c353d
ms.service: data-lake-store
ms.devlang: na
ms.topic: conceptual
ms.date: 03/26/2018
ms.author: twooley
ms.openlocfilehash: 48ff32655b107958a3e8e42dbd7de0f405a6fffa
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97094860"
---
# <a name="access-control-in-azure-data-lake-storage-gen1"></a>Contrôle d’accès dans Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 met en œuvre un modèle de contrôle d’accès dérivé de HDFS, lui-même issu du modèle de contrôle d’accès POSIX. Cet article présente les notions de base du modèle de contrôle d’accès pour Data Lake Storage Gen1. 

## <a name="access-control-lists-on-files-and-folders"></a>Listes de contrôle d’accès sur les fichiers et dossiers

Il existe deux types de listes de contrôle d’accès (ACL) : les **ACL d’accès** et les **ACL par défaut**.

* **ACL d’accès** : elles contrôlent l’accès à un objet. Les fichiers et les dossiers ont tous des ACL d’accès.

* **ACL par défaut** : « modèle » d’ACL associées à un dossier, qui déterminent les ACL d’accès pour tous les éléments enfants créés dans ce dossier. Les fichiers n’ont pas d’ACL par défaut.


Les ACL d’accès et les ACL par défaut ont la même structure.

> [!NOTE]
> La modification de l’ACL par défaut d’un parent n’affecte pas l’ACL d’accès ni l’ACL par défaut des éléments enfants qui existent déjà.
>
>

## <a name="permissions"></a>Autorisations

Les autorisations sur un objet de système de fichiers sont **Lecture**, **Écriture** et **Exécution**. Elles peuvent être utilisées sur les fichiers et les dossiers comme l’indique le tableau ci-dessous :

|            |    Fichier     |   Dossier |
|------------|-------------|----------|
| **Lecture (R)** | Permet de lire le contenu d’un fichier | Requiert les autorisations **Lecture** et **Exécution** pour répertorier le contenu du dossier|
| **Écriture (W)** | Permet d’écrire ou d’ajouter du contenu dans un fichier | Requiert les autorisations **Écriture** et **Exécution** pour créer des éléments enfants dans un dossier |
| **Exécution (X)** | Cela ne signifie rien dans le contexte de Data Lake Storage Gen1 | Requise pour parcourir les éléments enfants d’un dossier |

### <a name="short-forms-for-permissions"></a>Formes abrégées des autorisations

**RWX** correspond à **Lecture + Écriture + Exécution**. Il existe une forme numérique plus condensée dans laquelle **Lecture = 4**, **Écriture = 2** et **Exécution = 1**. Les autorisations sont représentées par la somme de ces chiffres. Voici quelques exemples.

| Forme numérique | Forme abrégée |      Signification     |
|--------------|------------|------------------------|
| 7            | `RWX`        | Lecture + Écriture + Exécution |
| 5            | `R-X`        | Lecture + Exécution         |
| 4            | `R--`        | Lire                   |
| 0            | `---`        | Aucune autorisation         |


### <a name="permissions-do-not-inherit"></a>Les autorisations ne se transmettent pas en héritage

Dans le modèle POSIX utilisé par Data Lake Storage Gen1, les autorisations d’un élément sont stockées sur l’élément lui-même. En d’autres termes, les autorisations d’un élément ne peuvent pas être héritées des éléments parents.

## <a name="common-scenarios-related-to-permissions"></a>Scénarios courants liés aux autorisations

Voici quelques scénarios courants pour vous aider à comprendre les autorisations nécessaires pour effectuer certaines opérations sur un compte Data Lake Storage Gen1.

| Opération | Object              |    /      | Seattle/   | Portland/   | Data.txt       |
|-----------|---------------------|-----------|------------|-------------|----------------|
| Lire      | Data.txt            |   `--X`   |   `--X`    |  `--X`      | `R--`          |
| Ajouter à | Data.txt            |   `--X`   |   `--X`    |  `--X`      | `-W-`          |
| DELETE    | Data.txt            |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| Créer    | Data.txt            |   `--X`   |   `--X`    |  `-WX`      | `---`          |
| List      | /                   |   `R-X`   |   `---`    |  `---`      | `---`          |
| List      | /Seattle/           |   `--X`   |   `R-X`    |  `---`      | `---`          |
| List      | /Seattle/Portland/  |   `--X`   |   `--X`    |  `R-X`      | `---`          |


> [!NOTE]
> Les autorisations d’écriture ne sont pas nécessaires pour supprimer le fichier, tant que les deux conditions précédentes sont remplies.
>
>


## <a name="users-and-identities"></a>Utilisateurs et identités

Chaque fichier et dossier dispose d’autorisations distinctes pour ces identités :

* L’utilisateur propriétaire
* Le groupe propriétaire
* Les utilisateurs nommés
* Les groupes nommés
* Tous les autres utilisateurs

Les identités des utilisateurs et des groupes sont des identités Azure Active Directory (Azure AD). Sauf mention contraire, dans le contexte de Data Lake Storage Gen1, un « utilisateur » peut désigner un utilisateur Azure AD ou un groupe de sécurité Azure AD.

### <a name="the-super-user"></a>Le super utilisateur

Un super utilisateur a plus de droits que tous les autres utilisateurs du compte Data Lake Storage Gen1. Un super utilisateur :

* possède les autorisations RWX sur **tous** les fichiers et dossiers ;
* peut modifier les autorisations sur n’importe quel fichier ou dossier ;
* peut modifier l’utilisateur ou le groupe propriétaire d’un fichier ou d’un dossier.

Tous les utilisateurs qui font partie du rôle **Propriétaires** pour un compte Data Lake Storage Gen1 sont automatiquement considérés comme des super utilisateurs.

### <a name="the-owning-user"></a>L’utilisateur propriétaire

L’utilisateur qui a créé l’élément est automatiquement l’utilisateur propriétaire de l’élément. Les utilisateurs propriétaires peuvent :

* modifier les autorisations de leurs fichiers ;
* modifier le groupe propriétaire d’un fichier détenu, tant que l’utilisateur propriétaire est également membre du groupe cible.

> [!NOTE]
> L’utilisateur propriétaire *ne peut pas* modifier l’utilisateur propriétaire d’un fichier ou d’un dossier. Seuls les super utilisateurs peuvent modifier l’utilisateur propriétaire d’un fichier ou d’un dossier.
>
>

### <a name="the-owning-group"></a>Le groupe propriétaire

**Contexte**

Dans les ACL POSIX, chaque utilisateur est associé à un « groupe principal ». Par exemple, l’utilisateur « Alice » peut appartenir au groupe « Finance ». Alice peut appartenir à plusieurs groupes, mais un groupe est toujours désigné comme son groupe principal. Dans POSIX, lorsqu’Alice crée un fichier, son groupe principal est défini comme groupe propriétaire de ce fichier (en l’occurrence, « finance »). Sinon, le groupe propriétaire se comporte comme pour les autorisations assignées à d’autres utilisateurs/groupes.

Comme il n’existe aucun « groupe principal », associé aux utilisateurs de Data Lake Storage Gen1, le groupe propriétaire est affecté comme indiqué ci-dessous.

**Affectation du groupe propriétaire pour un nouveau fichier ou dossier**

* **Cas 1** : le dossier racine « / ». Ce dossier est créé lors de la création d’un compte Data Lake Storage Gen1. Dans ce cas, le groupe propriétaire est défini sur un GUID composé uniquement de zéros.  Cette valeur n’autorise pas l’accès.  Il s’agit d’un espace réservé jusqu’à ce qu’un groupe soit affecté.
* **Cas 2** (tous les autres cas) : lorsqu’un élément est créé, le groupe propriétaire est copié à partir du dossier parent.

**Modification du groupe propriétaire**

Le groupe propriétaire peut être modifié par :
* un super utilisateur ;
* l’utilisateur propriétaire, si l’utilisateur propriétaire est également membre du groupe cible.

> [!NOTE]
> Le groupe propriétaire *ne peut pas* modifier les ACL d’un fichier ou d’un dossier.
>
> Pour les comptes créés en septembre 2018 ou avant, le groupe propriétaire a été défini en fonction de l’utilisateur ayant créé le compte, par exemple le dossier racine pour l’exemple **Cas 1**, ci-dessus.  Un seul compte d’utilisateur n’est pas valide pour accorder des autorisations via le groupe propriétaire, par conséquent aucune autorisations n’est accordée par ce paramètre par défaut. Vous pouvez assigner cette autorisation à un groupe d’utilisateurs valide.


## <a name="access-check-algorithm"></a>Algorithme de vérification des accès

Le pseudocode suivant représente l’algorithme de vérification des accès pour les comptes Data Lake Storage Gen1.

```
def access_check( user, desired_perms, path ) : 
  # access_check returns true if user has the desired permissions on the path, false otherwise
  # user is the identity that wants to perform an operation on path
  # desired_perms is a simple integer with values from 0 to 7 ( R=4, W=2, X=1). User desires these permissions
  # path is the file or folder
  # Note: the "sticky bit" is not illustrated in this algorithm
  
# Handle super users.
  if (is_superuser(user)) :
    return True

  # Handle the owning user. Note that mask IS NOT used.
  entry = get_acl_entry( path, OWNER )
  if (user == entry.identity)
      return ( (desired_perms & entry.permissions) == desired_perms )

  # Handle the named users. Note that mask IS used.
  entries = get_acl_entries( path, NAMED_USER )
  for entry in entries:
      if (user == entry.identity ) :
          mask = get_mask( path )
          return ( (desired_perms & entry.permmissions & mask) == desired_perms)

  # Handle named groups and owning group
  member_count = 0
  perms = 0
  entries = get_acl_entries( path, NAMED_GROUP | OWNING_GROUP )
  for entry in entries:
    if (user_is_member_of_group(user, entry.identity)) :
      member_count += 1
      perms | =  entry.permissions
  if (member_count>0) :
    return ((desired_perms & perms & mask ) == desired_perms)
 
  # Handle other
  perms = get_perms_for_other(path)
  mask = get_mask( path )
  return ( (desired_perms & perms & mask ) == desired_perms)
```

### <a name="the-mask"></a>Le masque

Comme illustré dans l’algorithme de vérification des accès, le masque limite l’accès pour les **utilisateurs nommés**, le **groupe propriétaire** et les **groupes nommés**.  

> [!NOTE]
> Pour un nouveau compte Data Lake Storage Gen1, le masque de l’ACL d’accès et du dossier racine (« / ») est défini par défaut sur RWX.
>
>

### <a name="the-sticky-bit"></a>Le sticky bit

Le sticky bit est une fonctionnalité avancée d’un système de fichiers POSIX. Dans le contexte de Data Lake Storage Gen1, il est peu probable que le sticky bit soit nécessaire. En résumé, si le sticky bit est activé sur un dossier, un élément enfant peut uniquement être supprimé ou renommé par l’utilisateur propriétaire de l’élément enfant.

Le sticky bit n’est pas affiché dans le portail Azure.

## <a name="default-permissions-on-new-files-and-folders"></a>Autorisations par défaut sur les nouveaux fichiers et dossiers

Lorsqu’un nouveau fichier ou dossier est créé dans un dossier existant, l’ACL par défaut sur le dossier parent détermine :

- L’ACL par défaut et l’ACL d’accès d’un dossier enfant
- L’ACL d’accès d’un fichier enfant (les fichiers n’ont pas d’ACL par défaut)

### <a name="umask"></a>umask

Quand vous créez un fichier ou dossier, l’umask est utilisé pour modifier la façon dont les ACL par défaut sont définies sur l’élément enfant. L’umask est une valeur 9 bits sur les dossiers parents qui contient une valeur RWX pour **l’utilisateur propriétaire**, le **groupe propriétaire** et **d’autres rôles**.

L’umask pour Azure Data Lake Storage Gen1 est une valeur constante définie sur 007. Cette valeur se traduit par

| Composant umask     | Forme numérique | Forme abrégée | Signification |
|---------------------|--------------|------------|---------|
| umask.owning_user   |    0         |   `---`      | Pour l’utilisateur propriétaire, copiez l’ACL par défaut du parent dans l’ACL d’accès de l’enfant | 
| umask.owning_group  |    0         |   `---`      | Pour le groupe propriétaire, copiez l’ACL par défaut du parent dans l’ACL d’accès de l’enfant | 
| umask.other         |    7         |   `RWX`      | Pour d’autres rôles, supprimez toutes les autorisations de l’ACL d’accès de l’enfant |

La valeur umask utilisée par Azure Data Lake Storage Gen1 signifie effectivement que la valeur pour d’autres rôles n’est jamais transmise par défaut sur les nouveaux enfants, quelle que soit l’indication de l’ACL par défaut. 

Le pseudocode suivant montre comment l’umask est appliqué lors de la création des ACL pour un élément enfant.

```
def set_default_acls_for_new_child(parent, child):
    child.acls = []
    for entry in parent.acls :
        new_entry = None
        if (entry.type == OWNING_USER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_user))
        elif (entry.type == OWNING_GROUP) :
            new_entry = entry.clone(perms = entry.perms & (~umask.owning_group))
        elif (entry.type == OTHER) :
            new_entry = entry.clone(perms = entry.perms & (~umask.other))
        else :
            new_entry = entry.clone(perms = entry.perms )
        child_acls.add( new_entry )
```

## <a name="common-questions-about-acls-in-data-lake-storage-gen1"></a>Questions les plus fréquentes sur les ACL dans Data Lake Storage Gen1

### <a name="do-i-have-to-enable-support-for-acls"></a>Dois-je activer la prise en charge des ACL ?

Non. Le contrôle d’accès via les ACL est toujours activé pour les comptes Data Lake Storage Gen1.

### <a name="which-permissions-are-required-to-recursively-delete-a-folder-and-its-contents"></a>Quelles sont les autorisations nécessaires pour supprimer de manière récursive un dossier et son contenu ?

* L’utilisateur doit disposer des autorisations **Écriture + Exécution** sur le dossier parent.
* L’utilisateur doit disposer des autorisations **Lecture + Écriture + Exécution** sur le dossier à supprimer et sur tous ses sous-dossiers.

> [!NOTE]
> L’autorisation Écriture n’est pas nécessaire pour supprimer des fichiers situés dans des dossiers. En outre, le dossier racine « / » ne peut **jamais** être supprimé.
>
>

### <a name="who-is-the-owner-of-a-file-or-folder"></a>Qui est le propriétaire d’un fichier ou d’un dossier ?

Le créateur d’un fichier ou d’un dossier en devient le propriétaire.

### <a name="which-group-is-set-as-the-owning-group-of-a-file-or-folder-at-creation"></a>Quel groupe est propriétaire d’un fichier ou d’un dossier lors de sa création ?

Le groupe propriétaire est copié à partir du groupe propriétaire du dossier dans lequel le fichier ou le dossier est créé.

### <a name="i-am-the-owning-user-of-a-file-but-i-dont-have-the-rwx-permissions-i-need-what-do-i-do"></a>Je suis l’utilisateur propriétaire d’un fichier, mais je n’ai pas les autorisations RWX dont j’ai besoin. Que faire ?

L’utilisateur propriétaire peut modifier les autorisations du fichier pour s’accorder les autorisations RWX dont il a besoin.

### <a name="when-i-look-at-acls-in-the-azure-portal-i-see-user-names-but-through-apis-i-see-guids-why-is-that"></a>Lorsque j’examine les ACL dans le portail Azure, je vois des noms d’utilisateur alors que, dans les API, je vois des GUID. Pourquoi ?

Dans les ACL, les entrées sont stockées sous forme de GUID qui correspondent aux utilisateurs dans Azure AD. Les API renvoient les GUID en l’état. Le portail Azure tente de simplifier l’utilisation des ACL en convertissant les GUID en noms conviviaux lorsque cela est possible.

### <a name="why-do-i-sometimes-see-guids-in-the-acls-when-im-using-the-azure-portal"></a>Pourquoi est-ce que je vois parfois des GUID dans les ACL lorsque j’utilise le portail ?

Un GUID apparaît lorsque l’utilisateur n’existe plus dans Azure AD. Cela se produit généralement lorsque l’utilisateur a quitté l’entreprise ou que son compte a été supprimé dans Azure AD. En outre, assurez-vous d’utiliser le bon ID pour définir les ACL (détails dans la question ci-dessous).

### <a name="when-using-service-principal-what-id-should-i-use-to-set-acls"></a>Lorsque j’utilise le principe du service, quel ID dois-je utiliser pour définir les ACL ?

Dans le portail Azure, accédez à **Azure Active Directory > Applications d’entreprise**, puis sélectionnez votre application. L’onglet **Vue d’ensemble** doit afficher un ID d’objet, et c’est ce dernier qui doit être utilisé lors de l’ajout d’ACL pour l’accès aux données (et non l’ID d’application).

### <a name="does-data-lake-storage-gen1-support-inheritance-of-acls"></a>Data Lake Storage Gen1 prend-il en charge l’héritage des ACL ?

Non, mais les ACL par défaut peuvent être utilisées pour définir les ACL des fichiers et dossiers enfants nouvellement créés sous le dossier parent.  

### <a name="where-can-i-learn-more-about-posix-access-control-model"></a>Comment en savoir plus sur le modèle de contrôle d’accès POSIX ?

* [Listes de contrôle d’accès (ACL) POSIX sur Linux](https://www.linux.com/news/posix-acls-linux)
* [HDFS Permission Guide (Guide des autorisations HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html)
* [Forum aux questions POSIX](https://www.opengroup.org/austin/papers/posix_faq.html)
* [POSIX 1003.1 2008](https://standards.ieee.org/findstds/standard/1003.1-2008.html)
* [POSIX 1003.1 2013](https://pubs.opengroup.org/onlinepubs/9699919799.2013edition/)
* [POSIX 1003.1 2016](https://pubs.opengroup.org/onlinepubs/9699919799.2016edition/)
* [ACL POSIX sous Ubuntu](https://help.ubuntu.com/community/FilePermissionsACLs)
* [ACL using access control lists on Linux (ACL utilisant des listes de contrôle d’accès sous Linux)](https://bencane.com/2012/05/27/acl-using-access-control-lists-on-linux/)

## <a name="see-also"></a>Voir aussi

* [Vue d’ensemble d’Azure Data Lake Storage Gen1](data-lake-store-overview.md)
