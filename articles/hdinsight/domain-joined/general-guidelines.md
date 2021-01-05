---
title: Recommandations générales sur la sécurité d’entreprise dans Azure HDInsight
description: Quelques bonnes pratiques permettant de faciliter le déploiement et la gestion du Pack Sécurité Entreprise.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 02/13/2020
ms.openlocfilehash: 8d97886232eecc369746e33df484cbfb9d40da72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87530263"
---
# <a name="enterprise-security-general-information-and-guidelines-in-azure-hdinsight"></a>Informations et recommandations générales sur la sécurité d’entreprise dans Azure HDInsight

Lors du déploiement d’un cluster HDInsight sécurisé, certaines bonnes pratiques peuvent faciliter le déploiement et la gestion des clusters. Des informations générales et des instructions sont abordées ici.

## <a name="use-of-secure-cluster"></a>Utilisation d’un cluster sécurisé

### <a name="recommended"></a>Recommandé

* Le cluster sera utilisé simultanément par plusieurs utilisateurs.
* Les utilisateurs ont différents niveaux d’accès aux mêmes données.

### <a name="not-necessary"></a>Pas nécessaire

* Vous allez exécuter uniquement des travaux automatisés (comme un compte d’utilisateur unique). Un cluster standard suffit.
* Vous pouvez effectuer l’importation de données à l’aide d’un cluster standard et utiliser le même compte de stockage sur un autre cluster sécurisé où les utilisateurs peuvent exécuter des travaux d’analytique.

## <a name="use-of-local-account"></a>Utilisation d’un compte local

* Si vous utilisez un compte d’utilisateur partagé ou un compte local, il sera difficile d’identifier qui a utilisé le compte pour changer la configuration ou le service.
* L’utilisation de comptes locaux pose problème quand les utilisateurs ne font plus partie de l’organisation.

## <a name="ranger"></a>Ranger

### <a name="policies"></a>Stratégies

* Par défaut, Ranger utilise **Refuser** comme stratégie.

* Quand l’accès aux données s’effectue par le biais d’un service où l’autorisation est activée :
  * Le plug-in d’autorisation Ranger est appelé et le contexte de la requête lui est indiqué.
  * Ranger applique les stratégies configurées pour le service. Si les stratégies Ranger échouent, le contrôle d’accès est déféré au système de fichiers. Certains services comme MapReduce vérifient uniquement si le fichier/dossier appartient au même utilisateur que celui qui envoie la requête. Les services comme Hive vérifient la correspondance de propriété ou les autorisations de système de fichiers appropriées (`rwx`).

* Pour Hive, en plus de disposer des autorisations nécessaires pour créer/mettre à jour/supprimer, l’utilisateur doit disposer d’autorisations `rwx` sur le répertoire du stockage et sur tous les sous-répertoires.

* Les stratégies peuvent être appliquées à des groupes (option préférable) plutôt qu’à des personnes.

* À chaque requête, l’agent d’autorisation Ranger évalue toutes les stratégies Ranger pour ce service. Cette évaluation peut avoir un impact sur le temps nécessaire à l’acceptation du travail ou de la demande.

### <a name="storage-access"></a>Accès au stockage

* Si le type de stockage est WASB, aucun jeton OAuth n’est impliqué.
* Si Ranger a effectué l’autorisation, l’accès au stockage se produit à l’aide de l’identité managée.
* Si Ranger n’a effectué aucune autorisation, l’accès au stockage se produit à l’aide du jeton OAuth de l’utilisateur.

### <a name="hierarchical-name-space"></a>Espace de noms hiérarchique

Quand l’espace de noms hiérarchique n’est pas activé :

* Il n’y a pas d’autorisations héritées.
* La seule autorisation de système de fichiers qui fonctionne est le rôle Azure **Données de stockage XXXX**, à attribuer à l’utilisateur directement dans le portail Azure.

### <a name="default-hdfs-permissions"></a>Autorisations HDFS par défaut

* Par défaut, les utilisateurs n’ont pas accès au dossier **/** sur HDFS. (Ils doivent se trouver dans le rôle propriétaire d’objets blob de stockage pour que l’accès aboutisse).
* Concernant le répertoire de préproduction pour MapReduce et d’autres applications, un répertoire spécifique à l’utilisateur est créé et des autorisations `sticky _wx` lui sont attribuées. Les utilisateurs peuvent créer des fichiers et des dossiers dessous, mais ils ne peuvent pas consulter d’autres éléments.

### <a name="url-auth"></a>Authentification d’URL

Si l’authentification d’URL est activée :

* La configuration contient les préfixes couverts dans l’authentification d’URL (par exemple `adl://`).
* Si l’accès est destiné à cette URL, Ranger vérifie si l’utilisateur figure dans la liste verte.
* Ranger ne vérifie aucune des stratégies affinées.

## <a name="resource-groups"></a>Groupes de ressources

Utilisez un nouveau groupe de ressources pour chaque cluster afin de pouvoir faire la distinction entre les ressources de cluster.

## <a name="nsgs-firewalls-and-internal-gateway"></a>Groupes de sécurité réseau, pare-feu et passerelle interne

* Utilisez les groupes de sécurité réseau (NSG) pour verrouiller les réseaux virtuels.
* Utilisez le pare-feu pour gérer les stratégies d’accès sortant.
* Utilisez la passerelle interne qui n’est pas ouverte sur l’Internet public.

## <a name="azure-active-directory"></a>Azure Active Directory

[Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md) (Azure AD) est le service Microsoft basé sur le cloud qui gère les identités et les accès.

### <a name="policies"></a>Stratégies

* Désactivez la stratégie d’accès conditionnel à l’aide de la stratégie basée sur les adresses IP. Cela nécessite l’activation des points de terminaison de service sur les réseaux virtuels sur lesquels les clusters sont déployés. Si vous utilisez un service externe pour l’authentification multifacteur (autre qu’AAD), la stratégie basée sur les adresses IP ne fonctionne pas

* La stratégie `AllowCloudPasswordValidation` est nécessaire pour les utilisateurs fédérés. Étant donné que HDInsight utilise le nom d’utilisateur/mot de passe directement pour obtenir des jetons à partir d’Azure AD, cette stratégie doit être activée pour tous les utilisateurs fédérés.

* Activez les points de terminaison de service si vous avez besoin de contourner l’accès conditionnel à l’aide d’adresses IP approuvées.

### <a name="groups"></a>Groupes

* Déployez toujours les clusters avec un groupe.
* Utilisez Azure AD pour gérer les appartenances aux groupes (plus faciles que d’essayer de gérer les services individuels dans le cluster).

### <a name="user-accounts"></a>Comptes d'utilisateurs

* Utilisez un compte d’utilisateur unique pour chaque scénario. Par exemple, utilisez un compte pour l’importation, et un autre compte pour les travaux de requête ou les autres travaux de traitement.
* Utilisez des stratégies Ranger basées sur un groupe plutôt que des stratégies individuelles.
* Élaborez un plan pour gérer les utilisateurs qui ne doivent plus avoir accès aux clusters.

## <a name="azure-active-directory-domain-services"></a>Azure Active Directory Domain Services

[Azure Active Directory Domain Services](../../active-directory-domain-services/overview.md) (Azure AD DS) fournit des services de domaine managés, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP (Lightweight Directory Access Protocol) et l’authentification Kerberos/NTLM entièrement compatible avec Windows Server Active Directory.

Azure AD DS est nécessaire pour que des clusters sécurisés rejoignent un domaine.
HDInsight ne peut pas dépendre de contrôleurs de domaine locaux ou de contrôleurs de domaine personnalisés, car cela introduit un trop grand nombre de points d’erreur, un partage des informations d’identification, des autorisations DNS, etc. Pour plus d’informations, consultez [Questions fréquentes (FAQ) sur Azure AD DS](../../active-directory-domain-services/faqs.md).

### <a name="azure-ad-ds-instance"></a>Instance Azure AD DS

* Créez l’instance avec le `.onmicrosoft.com domain`. Ainsi, il n’y aura pas plusieurs serveurs DNS traitant le domaine.
* Créez un certificat auto-signé pour le protocole LDAP sécurisé et chargez-le dans Azure AD DS.
* Utilisez un réseau virtuel appairé pour le déploiement de clusters. (C’est utile quand vous avez un certain nombre d’équipes qui déploient des clusters ESP HDInsight.) Cela garantit que vous n’avez pas besoin d’ouvrir des ports (NSG) sur le réseau virtuel avec contrôleur de domaine.
* Configurez correctement le DNS pour le réseau virtuel. (Le nom de domaine Azure AD DS doit être résolu sans aucune entrée de fichier d’hôte.)
* Si vous limitez le trafic sortant, vérifiez que vous avez consulté la [prise en charge du pare-feu dans HDInsight](../hdinsight-restrict-outbound-traffic.md).

### <a name="properties-synced-from-azure-ad-to-azure-ad-ds"></a>Propriétés synchronisées entre Azure AD et Azure AD DS

* Azure AD Connect effectue une synchronisation entre l’environnement local et Azure AD.
* Azure AD DS effectue une synchronisation à partir d’Azure AD.

Azure AD DS synchronise régulièrement les objets d’Azure AD. Le panneau Azure AD DS du portail Azure affiche l’état de la synchronisation. À chaque étape de la synchronisation, des propriétés uniques peuvent entrer en conflit et être renommées. Soyez attentif au mappage de propriétés d’Azure AD vers Azure AD DS.

Pour plus d’informations, consultez [Remplissage de UserPrincipalName dans Azure AD](../../active-directory/hybrid/plan-connect-userprincipalname.md) et [Fonctionnement de la synchronisation Azure AD DS](../../active-directory-domain-services/synchronization.md).

### <a name="password-hash-sync"></a>Synchronisation de hachage de mot de passe

* Les mots de passe sont synchronisés différemment des autres types d’objets. Seuls les hachages de mot de passe non réversibles sont synchronisés dans Azure AD et Azure AD DS
* La synchronisation de l’environnement local avec Azure AD doit être activée par le biais d’AD Connect
* La synchronisation d’Azure AD avec Azure AD DS est automatique (les latences sont inférieures à 20 minutes).
* Les hachages de mot de passe sont synchronisés uniquement en cas de changement du mot de passe. Quand vous activez la synchronisation du hachage de mot de passe, tous les mots de passe existants ne sont pas automatiquement synchronisés, car ils sont stockés de manière irréversible. Quand vous changez le mot de passe, les hachages de mot de passe sont synchronisés.

### <a name="computer-objects-location"></a>Emplacement d’objets ordinateur

Chaque cluster est associé à une seule unité d’organisation. Un utilisateur interne est provisionné dans l’unité d’organisation. Tous les nœuds sont joints à un domaine dans la même unité d’organisation.

### <a name="active-directory-administrative-tools"></a>Outils d’administration Active Directory

Pour savoir comment installer les outils d’administration Active Directory sur une machine virtuelle Windows Server, consultez [Installer des outils d’administration](../../active-directory-domain-services/tutorial-create-management-vm.md).

## <a name="troubleshooting"></a>Dépannage

### <a name="cluster-creation-fails-repeatedly"></a>Echecs répétés de création du cluster

Raisons les plus courantes :

* La configuration DNS n’est pas correcte ; la jonction de domaine des nœuds de cluster échoue.
* Les groupes de sécurité réseau (NSG) sont trop restrictifs, ce qui empêche la jonction de domaine.
* L’identité managée ne dispose pas des autorisations suffisantes.
* Le nom du cluster n’est pas unique sur les six premiers caractères (soit en raison d’un autre cluster actif, soit en raison d’un cluster supprimé).

## <a name="authentication-setup-and-configuration"></a>Instauration et configuration de l’authentification

### <a name="user-principal-name-upn"></a>Nom d’utilisateur principal (UPN)

* Veuillez utiliser des minuscules pour tous les services. Les UPN ne respectent pas la casse dans les clusters ESP, mais
* Le préfixe UPN doit correspondre au deux valeurs SAMAccountName dans Azure AD-DS. La correspondance avec le champ mail n’est pas obligatoire.

### <a name="ldap-properties-in-ambari-configuration"></a>Propriétés LDAP dans la configuration Ambari

Pour obtenir la liste complète des propriétés Ambari qui affectent la configuration de votre cluster HDInsight, consultez [Configuration de l’authentification LDAP Ambari](https://ambari.apache.org/1.2.1/installing-hadoop-using-ambari/content/ambari-chap2-4.html).

## <a name="next-steps"></a>Étapes suivantes

* [Configuration d’un Pack Sécurité Entreprise avec Azure Active Directory Domain Services dans HDInsight](./apache-domain-joined-configure-using-azure-adds.md)

* [Synchroniser des utilisateurs Azure Active Directory avec un cluster HDInsight](../hdinsight-sync-aad-users-to-cluster.md)
