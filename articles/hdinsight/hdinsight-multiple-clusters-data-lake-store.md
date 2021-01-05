---
title: Plusieurs clusters HDInsight et un compte Azure Data Lake Storage
description: Découvrez comment utiliser plusieurs clusters HDInsight avec un seul compte Data Lake Storage
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/18/2019
ms.openlocfilehash: df28374d0f124ceb46d2f97d55218d428275deca
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92533085"
---
# <a name="use-multiple-hdinsight-clusters-with-an-azure-data-lake-storage-account"></a>Utiliser plusieurs clusters HDInsight avec un compte Azure Data Lake Storage

À partir de HDInsight version 3.5, vous pouvez créer des clusters HDInsight avec des comptes Azure Data Lake Storage comme système de fichiers par défaut.
Data Lake Storage prend en charge le stockage illimité, ce qui le rend idéal non seulement pour héberger de grandes quantités de données, mais également pour héberger plusieurs clusters HDInsight qui partagent un même compte Data Lake Storage. Pour plus d’informations sur la création d’un cluster HDInsight avec Azure Data Lake Storage comme stockage, consultez [Démarrage rapide : Configurer des clusters dans HDInsight](./hdinsight-hadoop-provision-linux-clusters.md).

Cet article fournit des recommandations à l’administrateur Data Lake Storage pour configurer un compte Data Lake Storage unique et partagé qui peut être utilisé sur plusieurs clusters HDInsight **actifs** . Ces recommandations s’appliquent à l’hébergement de plusieurs clusters Apache Hadoop sécurisés ou non sécurisés sur un compte Data Lake Storage partagé.

## <a name="data-lake-storage-file-and-folder-level-acls"></a>Listes ACL de niveau fichier et dossier Data Lake Storage

Le reste de cet article part du principe que vous avez une bonne connaissance des listes ACL de niveau fichier et dossier dans l’application Azure Data Lake Storage, qui sont décrites en détail dans [Contrôle d’accès dans Azure Data Lake Storage](../data-lake-store/data-lake-store-access-control.md).

## <a name="data-lake-storage-setup-for-multiple-hdinsight-clusters"></a>Configuration de Data Lake Storage pour plusieurs clusters HDInsight

Prenons une hiérarchie de dossiers sur deux niveaux pour expliquer les recommandations quant à l’utilisation de plusieurs clusters HDInsight avec un compte Data Lake Storage. Supposez que vous avez un compte Data Lake Storage avec la structure de dossiers **/clusters/finance** . Avec cette structure, tous les clusters requis par le service financier peuvent utiliser /clusters/finance comme emplacement de stockage. À l’avenir, si une autre organisation (par exemple le service marketing) souhaite créer des clusters HDInsight avec le même compte Data Lake Storage, elle pourrait créer /clusters/marketing. Pour l’instant, nous allons simplement utiliser **/clusters/finance** .

Pour que cette structure de dossiers soit bien utilisée par les clusters HDInsight, l’administrateur Data Lake Storage doit attribuer les autorisations appropriées, comme décrit dans le tableau. Les autorisations indiquées dans le tableau correspondent aux ACL d’accès et non aux ACL par défaut.

|Dossier  |Autorisations  |Utilisateur propriétaire  |groupe propriétaire  | Utilisateur nommé | Autorisations de l’utilisateur nommé | Groupe nommé | Autorisations du groupe nommé |
|---------|---------|---------|---------|---------|---------|---------|---------|
|/ | rwxr-x--x  |admin |admin  |Principal du service |--x  |FINGRP   |r-x         |
|/clusters | rwxr-x--x |admin |admin |Principal du service |--x  |FINGRP |r-x         |
|/clusters/finance | rwxr-x--t |admin |FINGRP  |Principal du service |rwx  |-  |-     |

Dans le tableau,

- **admin** est le créateur et l’administrateur du compte Data Lake Storage.
- **Principal de service** est le principal de service Azure Active Directory (AAD) associé au compte.
- **FINGRP** est un groupe d’utilisateurs créé dans AAD qui contient les utilisateurs du service financier.

Pour obtenir des instructions sur la création d’une application AAD (qui crée également un principal de service), consultez [Créer une application AAD](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal). Pour obtenir des instructions sur la création d’un groupe d’utilisateurs dans AAD, consultez [Gestion des groupes dans Azure Active Directory](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Quelques points importants à prendre en compte.

- La structure de dossiers sur deux niveaux ( **/clusters/finance/** ) doit être créée et configurée avec les autorisations appropriées par l’administrateur Data Lake Storage **avant** d’utiliser le compte de stockage pour les clusters. Cette structure n’est pas créée automatiquement lors de la création des clusters.
- L’exemple ci-dessus recommande de définir le groupe propriétaire de **/clusters/finance** en tant que **FINGRP** et d’octroyer à FINGRP l’accès **r-x** à l’intégralité de la hiérarchie des dossiers à partir de la racine. Cela permet de s’assurer que les membres de FINGRP peuvent accéder à la structure de dossiers à partir de la racine.
- Dans le cas où plusieurs principaux de service AAD peuvent créer des clusters sous **/clusters/finance** , le sticky-bit (lorsque défini sur le dossier **finance** ) permet de s’assurer que les dossiers créés par un principal de service ne peuvent pas être supprimés par les autres.
- Une fois que la structure de dossiers et les autorisations sont en place, le processus de création de clusters HDInsight crée un emplacement de stockage spécifique aux clusters sous **/clusters/finance/** . Par exemple, l’emplacement de stockage d’un cluster nommé fincluster01 pourrait être **/clusters/finance/fincluster01** . La propriété et les autorisations pour les dossiers créés par le cluster HDInsight sont indiquées dans le tableau présenté ici.

    |Dossier  |Autorisations  |Utilisateur propriétaire  |groupe propriétaire  | Utilisateur nommé | Autorisations de l’utilisateur nommé | Groupe nommé | Autorisations du groupe nommé |
    |---------|---------|---------|---------|---------|---------|---------|---------|
    |/clusters/finance/fincluster01 | rwxr-x---  |Principal de service |FINGRP  |- |-  |-   |-  |

## <a name="recommendations-for-job-input-and-output-data"></a>Recommandations pour les données d’entrée et de sortie de tâche

Nous vous recommandons de stocker les données d’entrée et données de sortie d’une tâche en dehors de **/clusters** . Ainsi, même si le dossier spécifique aux clusters est supprimé pour récupérer de l’espace de stockage, les données d’entrée et de sortie du travail restent disponibles pour une utilisation ultérieure. Dans ce cas, assurez-vous que la hiérarchie des dossiers servant à stocker les données d’entrée et de sortie de la tâche permet un niveau d’accès approprié pour le principal de service.

## <a name="limit-on-clusters-sharing-a-single-storage-account"></a>Limite sur les clusters partageant un même compte de stockage

La limite du nombre de clusters pouvant partager un même compte Data Lake Storage dépend de la charge de travail exécutée sur ces clusters. En cas de nombre trop important de clusters ou de charges de travail très lourdes sur les clusters partageant un compte de stockage, l’entrée ou la sortie du compte de stockage risque d’être limitée.

## <a name="support-for-default-acls"></a>Prise en charge des ACL par défaut

Lorsque vous créez un principal de service avec un accès utilisateur nommé (comme indiqué dans le tableau ci-dessus), nous vous recommandons de ne **pas** ajouter l’utilisateur nommé avec une ACL par défaut. La configuration d’un accès utilisateur nommé utilisant des ACL par défaut se traduit par l’attribution de 770 autorisations pour l’utilisateur propriétaire, le groupe propriétaire et les autres. La valeur par défaut de 770 ne retire pas les autorisations de l’utilisateur propriétaire (7) ni du groupe propriétaire (7), mais elle retire toutes les autorisations des autres (0). Cela entraîne un problème connu avec un cas d’usage particulier qui est décrit en détail dans la section [Problèmes connus et solutions de contournement](#known-issues-and-workarounds).

## <a name="known-issues-and-workarounds"></a>Problèmes connus et solutions de contournement

Cette section répertorie les problèmes connus liés à l’utilisation de HDInsight avec Data Lake Storage, ainsi que leurs solutions de contournement.

### <a name="publicly-visible-localized-apache-hadoop-yarn-resources"></a>Ressources Apache Hadoop YARN localisées visibles publiquement

Quand un compte Azure Data Lake Storage est créé, le répertoire racine est configuré automatiquement avec les bits d’autorisation des listes ACL d’accès définis sur 770. L’utilisateur propriétaire du dossier racine est défini sur l’utilisateur qui a créé le compte (l’administrateur Data Lake Storage), tandis que le groupe propriétaire est défini sur le groupe principal de l’utilisateur qui a créé le compte. Aucun accès n’est fourni pour les « autres ».

Ces paramètres sont connus pour affecter un cas d’usage HDInsight particulier capturé dans [YARN 247](https://hwxmonarch.atlassian.net/browse/YARN-247). L’envoi de tâches peut échouer avec un message d’erreur semblable à celui-ci :

```output
Resource XXXX is not publicly accessible and as such cannot be part of the public cache.
```

Comme indiqué dans le YARN JIRA associé précédemment, lors de la localisation des ressources publiques, le localisateur valide le fait que toutes les ressources demandées sont bien publiques en vérifiant leurs autorisations sur le système de fichiers distant. Les éléments LocalResource qui ne correspondent pas à cette condition sont rejetés pour la localisation. La vérification des autorisations inclut l’accès en lecture au fichier pour les « autres ». Ce scénario ne fonctionne pas instantanément lors de l’hébergement de clusters HDInsight sur Azure Data Lake, car Azure Data Lake refuse tout accès aux « autres » au niveau du dossier racine.

#### <a name="workaround"></a>Solution de contournement

Définissez les autorisations d’exécution en lecture pour les **autres** dans la hiérarchie, par exemple, au niveau **/** , **/clusters** et **/clusters/finance** comme indiqué dans le tableau ci-dessus.

## <a name="see-also"></a>Voir aussi

- [Démarrage rapide : Configurer des clusters dans HDInsight](./hdinsight-hadoop-provision-linux-clusters.md)
- [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](hdinsight-hadoop-use-data-lake-storage-gen2.md)