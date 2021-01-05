---
title: Vue d’ensemble de la sécurité dans Azure Data Lake Storage Gen1 | Microsoft Docs
description: Découvrez les fonctionnalités de sécurité d’Azure Data Lake Storage Gen1, notamment l’authentification, l’autorisation, l’isolement réseau, la protection des données et l’audit.
services: data-lake-store
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: twooley
ms.openlocfilehash: 240018381a3139a6378141d78514e43ae469de5d
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "92146321"
---
# <a name="security-in-azure-data-lake-storage-gen1"></a>Sécurité dans Azure Data Lake Storage Gen1

Un grand nombre d’entreprises tirent parti de l’analyse du Big Data pour bénéficier d’informations métier afin de prendre des décisions éclairées. Une organisation peut évoluer dans un environnement complexe et réglementé, avec un nombre croissant d’utilisateurs divers. Il est essentiel pour une entreprise de garantir que les données métier critiques sont stockées de manière plus sécurisée, avec le niveau d’accès adéquat pour chaque utilisateur. Azure Data Lake Storage Gen1 vous permet de respecter ces exigences de sécurité. Dans cet article, découvrez les fonctionnalités de sécurité de Data Lake Storage Gen1, notamment :

* Authentification
* Autorisation
* Isolement réseau
* Protection de données
* Audit

## <a name="authentication-and-identity-management"></a>Authentification et gestion des identités

L’authentification est le processus par lequel l’identité d’un utilisateur est vérifiée lorsque celui-ci interagit avec Data Lake Storage Gen1 ou tout service qui se connecte à Data Lake Storage Gen1. Pour l’authentification et la gestion des identités, Data Lake Storage Gen1 utilise [Azure Active Directory](../active-directory/fundamentals/active-directory-whatis.md), une solution complète pour la gestion des identités et des accès dans le cloud qui simplifie la gestion des utilisateurs et des groupes.

Chaque abonnement Azure peut être associé à une instance d’Azure Active Directory. Seuls les utilisateurs et les identités de service qui sont définis dans votre service Azure Active Directory peuvent accéder à votre compte Data Lake Storage Gen1, à l’aide du portail Azure, des outils de ligne de commande, ou via les applications clientes que votre organisation crée avec le Kit de développement logiciel (SDK) Data Lake Storage Gen1. Les principaux avantages de l’utilisation d’Azure Active Directory en tant que mécanisme de contrôle d’accès centralisé sont les suivants :

* Gestion du cycle de vie des identités simplifiée. L’identité d’un utilisateur ou d’un service (une identité de principal du service) peut être rapidement créée et révoquée en supprimant ou en désactivant simplement le compte dans le répertoire.
* Authentification multifacteur [L’authentification multifacteur](../active-directory/authentication/concept-mfa-howitworks.md) fournit une couche supplémentaire de sécurité pour les connexions et les transactions utilisateur.
* Authentification à partir de n’importe quel client via un protocole ouvert standard, tel qu’OAuth ou OpenID.
* Fédération avec les services de répertoire d’entreprise et les fournisseurs d’identité cloud.

## <a name="authorization-and-access-control"></a>Autorisation et contrôle d’accès

Une fois qu’un utilisateur est authentifié par Azure Active Directory pour accéder à Data Lake Storage Gen1, les contrôles d’autorisation accèdent aux autorisations pour Data Lake Storage Gen1. Data Lake Storage Gen1 sépare l’autorisation pour les activités relatives aux comptes et aux données de la manière suivante :

* [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/overview.md) pour la gestion des comptes
* ACL POSIX pour accéder aux données dans le magasin

### <a name="azure-rbac-for-account-management"></a>Azure RBAC pour la gestion des comptes

Par défaut, quatre rôles de base sont définis pour Data Lake Storage Gen1. Les rôles permettent d’exécuter différentes opérations sur un compte Data Lake Storage Gen1 via le portail Azure, les applets de commande PowerShell et les API REST. Les rôles Propriétaire et Collaborateur peuvent effectuer un large éventail de fonctions d’administration sur le compte. Vous pouvez assigner le rôle Lecteur aux utilisateurs qui voient uniquement les données de gestion des comptes.

![Rôles Azure](./media/data-lake-store-security-overview/rbac-roles.png "Rôles Azure")

Notez que, même si les rôles sont attribués pour la gestion des comptes, certains rôles affectent l’accès aux données. Vous devez utiliser les ACL pour contrôler l’accès aux opérations qu’un utilisateur peut effectuer sur le système de fichiers. Le tableau suivant présente un résumé des droits d’accès aux données et aux droits de gestion pour les rôles par défaut.

| Rôles | Droits de gestion | Droits d’accès aux données | Explication |
| --- | --- | --- | --- |
| Aucun rôle affecté |None |Régi par ACL |L’utilisateur ne peut pas utiliser le portail Azure ou les applets de commande Azure PowerShell pour parcourir Data Lake Storage Gen1. L’utilisateur peut uniquement utiliser les outils de ligne de commande. |
| Propriétaire |Tous |Tous |Le rôle Propriétaire est un super utilisateur. Ce rôle peut tout gérer et bénéficie d’un accès total aux données. |
| Lecteur |Lecture seule |Régi par ACL |Le rôle Lecteur peut tout afficher en matière de gestion des comptes, notamment quel utilisateur est affecté à quel rôle. Le rôle Lecteur ne permet pas d’effectuer des modifications. |
| Contributeur |Tout, sauf ajouter et supprimer des rôles |Régi par ACL |Le rôle Collaborateur peut gérer certains aspects d’un compte, tels que les déploiements et la création et la gestion des alertes. Le rôle Collaborateur ne peut pas ajouter ou supprimer des rôles. |
| Administrateur de l'accès utilisateur |Ajouter et supprimer des rôles |Régi par ACL |Le rôle Administrateur de l’accès utilisateur peut gérer l’accès des utilisateurs aux comptes. |

Pour obtenir des instructions, consultez [Affecter les utilisateurs ou les groupes de sécurité aux comptes Azure Data Lake Storage Gen1](data-lake-store-secure-data.md#assign-users-or-security-groups-to-data-lake-storage-gen1-accounts).

### <a name="using-acls-for-operations-on-file-systems"></a>Utilisation des ACL pour les opérations sur les systèmes de fichiers

Data Lake Storage Gen1 est un système de fichiers hiérarchique comme Hadoop HDFS Distributed File System (HDFS), et il prend en charge les [ACL POSIX](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsPermissionsGuide.html#ACLs_Access_Control_Lists). Elle contrôle la lecture (r), l’écriture (w) et exécute(x) les autorisations des ressources pour le rôle Propriétaire, le groupe Propriétaires et d’autres utilisateurs et groupes. Dans Data Lake Storage Gen1, les ACL peuvent être activées au niveau du dossier racine, des sous-dossiers et des fichiers individuels. Pour plus d’informations sur le fonctionnement des ACL dans le contexte de Data Lake Storage Gen1, consultez [Contrôle d’accès dans Data Lake Storage Gen1](data-lake-store-access-control.md).

Nous vous recommandons de définir des ACL pour plusieurs utilisateurs à l’aide des [groupes de sécurité](../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Ajoutez des utilisateurs dans un groupe de sécurité, puis affectez les ACL d’un fichier ou dossier à ce groupe de sécurité. Cela est utile lorsque vous souhaitez fournir des autorisations assignées, parce que vous êtes limité à un maximum de 28 entrées pour les autorisations assignées. Pour plus d’informations sur comment améliorer la sécurisation des données stockées dans Data Lake Storage Gen1 à l’aide de groupes de sécurité Azure Active Directory, consultez [Affecter des utilisateurs ou un groupe de sécurité comme ACL au système de fichiers Data Lake Storage Gen1](data-lake-store-secure-data.md#filepermissions).

![Lister les autorisations d’accès](./media/data-lake-store-security-overview/adl.acl.2.png "Lister les autorisations d’accès")

## <a name="network-isolation"></a>Isolement réseau

Utilisez Data Lake Storage Gen1 pour contrôler l’accès à votre magasin de données au niveau du réseau. Vous pouvez activer des pare-feu et définir une plage d’adresses IP pour vos clients approuvés. Avec une plage d’adresses IP, seuls les clients qui ont une adresse IP dans la plage définie peuvent se connecter à Data Lake Storage Gen1.

![Paramètres de pare-feu et adresse IP](./media/data-lake-store-security-overview/firewall-ip-access.png "Paramètres de pare-feu et adresse IP")

Les réseaux virtuels Azure prennent en charge les balises de service pour Data Lake Gen 1. Une balise de service représente un groupe de préfixes d’adresses IP d’un service Azure donné. Microsoft gère les préfixes d’adresse englobés par la balise de service et met à jour automatiquement la balise de service quand les adresses changent. Pour plus d’informations, consultez [Présentation des balises de Service Azure](../virtual-network/service-tags-overview.md).

## <a name="data-protection"></a>Protection de données

Data Lake Storage Gen1 protège vos données tout au long de leur cycle de vie. Pour les données en transit, Data Lake Storage Gen1 utilise le protocole normalisé TLS (Transport Layer Security) 1.2 pour sécuriser les données sur le réseau.

![Chiffrement dans Data Lake Storage Gen1](./media/data-lake-store-security-overview/adls-encryption.png "Chiffrement dans Data Lake Storage Gen1")

Data Lake Storage Gen1 assure également le chiffrement des données stockées dans le compte. Vous pouvez choisir de chiffrer vos données ou non. Si vous optez pour le chiffrement, les données stockées dans Data Lake Storage Gen1 sont chiffrées avant d’être stockées sur un support permanent. Dans ce cas, Data Lake Storage Gen1 chiffre automatiquement les données avant qu’elles soient rendues persistantes et les déchiffre avant qu’elles soient récupérées ; ainsi, ces opérations sont totalement transparentes vis-à-vis du client qui accède aux données. Aucune modification de code n’est nécessaire côté client pour chiffrer/déchiffrer les données.

Concernant la gestion des clés, Data Lake Storage Gen1 met à votre disposition deux modes pour gérer les clés de chiffrement principales, qui sont nécessaires pour déchiffrer les données stockées dans le Data Lake Storage Gen1. Vous pouvez soit laisser Data Lake Storage Gen1 gérer les clés de chiffrement principales, soit choisir de conserver la propriété des clés de chiffrement principales en utilisant votre compte Azure Key Vault. Vous spécifiez le mode de gestion de clés lors de la création du compte Data Lake Storage Gen1. Pour savoir comment fournir une configuration avec chiffrement, consultez [Prise en main d’Azure Data Lake Storage Gen1 à l’aide du Portail Azure](data-lake-store-get-started-portal.md).

## <a name="activity-and-diagnostic-logs"></a>Journaux d’activités et de diagnostic

Vous pouvez utiliser les journaux d’activité ou de diagnostic selon si vous recherchez des journaux d’activité sur des activités liées à la gestion ou des activités liées aux données.

* Les activités liées à la gestion des comptes utilisent les API Azure Resource Manager et sont affichées dans le Portail Azure via les journaux d’activité.
* Les activités liées aux données utilisent les API REST WebHDFS et sont affichées dans le portail via les journaux de diagnostic.

### <a name="activity-log"></a>Journal d’activité

Pour respecter les réglementations, une organisation peut avoir besoin de pistes d’audit adéquates des activités de gestion des comptes si elle doit examiner en profondeur des incidents spécifiques. Data Lake Storage Gen1 est doté de fonctionnalités intégrées d’analyse et journalise toutes les activités de gestion des comptes.

Pour les pistes d’audit de gestion des comptes, affichez et choisissez les colonnes que vous souhaitez journaliser. Vous pouvez également exporter des journaux d’activité vers le Stockage Azure.

![Journal d’activité](./media/data-lake-store-security-overview/activity-logs.png "Journal d’activité")

Pour plus d’informations sur l’utilisation des journaux d’activité, consultez [Afficher les journaux d’activité pour auditer les actions sur les ressources](../azure-resource-manager/management/view-activity-logs.md).

### <a name="diagnostics-logs"></a>Journaux de diagnostics

Vous pouvez activer l’audit d’accès aux données et la journalisation des diagnostics dans le Portail Azure et envoyer des journaux à un compte de stockage Blob Azure, à Event Hub ou à des journaux Azure Monitor.

![Journaux de diagnostics](./media/data-lake-store-security-overview/diagnostic-logs.png "Journaux de diagnostics")

Pour plus d’informations sur l’utilisation des journaux de diagnostic avec Data Lake Storage Gen1, consultez [Accès aux journaux de diagnostic pour Data Lake Storage Gen1](data-lake-store-diagnostic-logs.md).

## <a name="summary"></a>Résumé

Les clients professionnels exigent une plateforme cloud d’analyse des données sécurisée et facile à utiliser. Data Lake Storage Gen1 est conçu pour répondre à ces exigences par le biais de la gestion des identités et l’authentification via l’intégration Azure Active Directory, l’autorisation basée sur les ACL, l’isolement réseau, le chiffrement des données en transit et au repos et l’audit.

Si vous souhaitez voir de nouvelles fonctionnalités de Data Lake Storage Gen1, envoyez-nous vos commentaires sur notre [forum UserVoice Data Lake Storage Gen1](https://feedback.azure.com/forums/327234-data-lake).

## <a name="see-also"></a>Voir aussi

* [Vue d’ensemble d’Azure Data Lake Storage Gen1](data-lake-store-overview.md)
* [Prise en main de Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* [Sécuriser les données dans Data Lake Storage Gen1](data-lake-store-secure-data.md)