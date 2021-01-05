---
title: Serveur Azure MFA et Active Directory – Azure Active Directory
description: Comment intégrer le serveur Microsoft Azure Multi-Factor Authentication à Active Directory afin de pouvoir synchroniser les annuaires.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.custom: seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8b3778ea68edf1fbbb41efb899749e6f35b39bae
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742287"
---
# <a name="directory-integration-between-azure-mfa-server-and-active-directory"></a>Intégration d'annuaires entre le serveur Azure MFA et Active Directory

Utilisez la section Intégration de répertoires du serveur Azure MFA pour l’intégrer à Active Directory ou un autre répertoire LDAP. Vous pouvez configurer des attributs pour correspondre au schéma du répertoire et configurer la synchronisation automatique des utilisateurs.

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisés avec l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

## <a name="settings"></a>Paramètres

Par défaut, le serveur Azure Multi-Factor Authentication (MFA) est configuré pour importer ou synchroniser des utilisateurs à partir d’Active Directory.  L’onglet Intégration de répertoires vous permet d’annuler le comportement par défaut et d’effectuer la liaison à un autre répertoire LDAP, un répertoire ADAM ou un contrôleur de domaine Active Directory spécifique.  Il permet également l'utilisation de l'authentification LDAP sur un proxy LDAP ou une liaison LDAP comme cible RADIUS, la pré-authentification pour l'authentification IIS ou l'authentification principale pour le portail de l'utilisateur.  Le tableau suivant décrit les paramètres individuels.

![Modifier la configuration LDAP du serveur MFA](./media/howto-mfaserver-dir-ad/dirint.png)

> [!NOTE]
> Il n’est pas garanti que l’intégration d’annuaire fonctionne correctement avec des répertoires autres que Microsoft Active Directory Domain Services.

| Fonctionnalité | Description |
| --- | --- |
| Utiliser Active Directory |Sélectionnez cette option pour utiliser Active Directory pour l'importation et la synchronisation.  Il s'agit du paramètre par défaut. <br>Remarque : Pour que l’intégration Active Directory fonctionne correctement, joignez l’ordinateur à un domaine et connectez-vous avec un compte de domaine. |
| Inclure des domaines approuvés |Cochez la case **Inclure des domaines approuvés** pour que l’agent tente de se connecter à des domaines approuvés par le domaine actuel, un autre domaine de la forêt ou des domaines impliqués dans une approbation de forêt.  Lorsque vous n'importez pas ou ne synchronisez pas d'utilisateurs à partir des domaines approuvés, désactivez la case à cocher pour améliorer les performances.  La case est cochée par défaut. |
| Utiliser une configuration LDAP spécifique |Sélectionnez l'option Utiliser LDAP pour utiliser les paramètres LDAP spécifiés pour l'importation et la synchronisation. Remarque : Lorsque l’option Utiliser LDAP est sélectionnée, les références de l’interface utilisateur passent d’Active Directory à LDAP. |
| Bouton Modifier |Le bouton Modifier permet de modifier les paramètres de la configuration LDAP. |
| Utiliser les requêtes d'étendue d'attribut |Indique si les requêtes d'étendue d'attribut doivent être utilisées.  Les requêtes d'étendue d'attribut donnent des recherches d'annuaire efficaces incluant des enregistrements basés sur les entrées d'attribut d'un autre enregistrement.  Le serveur Azure Multi-Factor Authentication utilise les requêtes d'étendue d'attribut pour interroger efficacement les utilisateurs qui sont membres d'un groupe de sécurité.   <br>Remarque :  Dans certains cas, les requêtes d’étendue d’attribut sont prises en charge, mais ne doivent pas être utilisées.  Par exemple, Active Directory peut rencontrer des problèmes avec les requêtes d'étendue d'attribut lorsqu'un groupe de sécurité contient des membres issus de plusieurs domaines. Dans ce cas, désactivez la case à cocher. |

Le tableau suivant décrit les paramètres de la configuration LDAP.

| Fonctionnalité | Description |
| --- | --- |
| Serveur |Entrez le nom d'hôte ou l’adresse IP du serveur exécutant l'annuaire LDAP.  Il est également possible de spécifier un serveur de sauvegarde, en le séparant par un point-virgule. <br>Remarque : Lorsque le type de liaison est SSL (TLS), un nom d’hôte complet est nécessaire. |
| Nom unique de base |Entrez le nom unique de l’objet de répertoire de base à partir duquel toutes les requêtes de répertoire sont lancées.  Par exemple, dc = abc, dc = com. |
| Type de liaison - Requêtes |Sélectionnez le type de liaison approprié à utiliser lors de la liaison pour rechercher l'annuaire LDAP.  Cette option est utilisée pour les importations, la synchronisation et la résolution de nom d'utilisateur. <br><br>  Anonyme - Une liaison anonyme est effectuée.  Le nom unique de liaison et le mot de passe de liaison ne sont pas utilisés.  Cela fonctionne uniquement si le répertoire LDAP autorise une liaison anonyme et si les autorisations permettent l’interrogation des enregistrements appropriés et des attributs.  <br><br> Simple - Le nom unique de liaison et le mot de passe de liaison sont transmis en texte brut pour la liaison au répertoire LDAP.  Cela sert à des fins de test, pour vérifier que le serveur peut être atteint et que le compte de liaison dispose de l’accès approprié. Une fois que le certificat approprié a été installé, utilisez SSL.  <br><br> SSL - Le nom unique de liaison et le mot de passe de liaison sont chiffrés à l’aide de SSL pour la liaison au répertoire LDAP.  Installez en local un certificat approuvé par le répertoire LDAP.  <br><br> Windows - Le nom d’utilisateur de liaison et le mot de passe de liaison sont utilisés pour la connexion sécurisée à un contrôleur de domaine Active Directory ou un répertoire ADAM.  Si le champ Nom d’utilisateur de liaison n’est pas renseigné, le compte de l’utilisateur connecté est utilisé pour la liaison. |
| Type de liaison - Authentifications |Sélectionnez le type de liaison approprié à utiliser lors de l'authentification de liaison LDAP.  Consultez les descriptions de type de liaison sous Type de liaison - Requêtes.  Cela permet, par exemple, d'utiliser la liaison anonyme pour les requêtes alors que la liaison SSL est utilisée pour sécuriser les authentifications de liaison LDAP. |
| Nom unique de liaison ou nom d'utilisateur de liaison |Entrez le nom unique de l'enregistrement d'utilisateur pour le compte à utiliser lors de la liaison à l'annuaire LDAP.<br><br>Le nom unique de liaison est utilisé uniquement lorsque le Type de liaison est Simple ou SSL.  <br><br>Entrez le nom d'utilisateur du compte Windows à utiliser lors de la liaison à l'annuaire LDAP lorsque le Type de liaison est Windows.  Si ce champ n’est pas renseigné, le compte de l’utilisateur connecté est utilisé pour la liaison. |
| Mot de passe de liaison |Entrez le mot de passe de liaison pour le nom unique de liaison ou le nom d'utilisateur utilisé pour la liaison à l'annuaire LDAP.  Pour configurer le mot de passe du service AdSync du serveur MFA, activez la synchronisation et assurez-vous que le service s’exécute sur l’ordinateur local.  Le mot de passe est enregistré dans les noms d’utilisateur et mots de passe stockés de Windows et associé au compte sous lequel le service AdSync du serveur MFA est exécuté.  Le mot de passe est également associé au compte sous lequel l’interface utilisateur du serveur MFA est exécutée ainsi qu’au compte sous lequel le service du serveur MFA est exécuté.  <br><br>Comme le mot de passe est uniquement enregistré dans les noms d’utilisateur et mots de passe stockés de Windows sur le serveur local, répétez cette étape sur chaque serveur MFA ayant besoin d’accéder au mot de passe. |
| Limite de taille de requête |Spécifiez la taille limite pour le nombre maximal d’utilisateurs qu’une recherche dans le répertoire peut renvoyer.  Cette limite doit correspondre à la configuration de l'annuaire LDAP.  Pour les recherches volumineuses où la pagination n’est pas prise en charge, les tâches d’importation et de synchronisation tentent de récupérer les utilisateurs par lots.  Si la taille limite spécifiée ici est supérieure à la limite configurée sur l'annuaire LDAP, certains utilisateurs risquent d'être omis. |
| Bouton de test |Cliquez sur **Test** pour tester la liaison au serveur LDAP.  <br><br>Vous n’avez pas besoin de sélectionner l’option **Utiliser LDAP** pour tester la liaison. Cela permet de tester la liaison avant d’utiliser la configuration LDAP. |

## <a name="filters"></a>Filtres

Les filtres vous permettent de définir des critères pour qualifier les enregistrements lorsque vous effectuez une recherche dans l'annuaire.  En définissant le filtre, vous pouvez étendre les objets que vous souhaitez synchroniser.  

![Configurer le filtrage d’annuaire du serveur MFA](./media/howto-mfaserver-dir-ad/dirint2.png)

Azure Multi-Factor Authentication comporte les trois options de filtre suivantes :

* **Filtre conteneur** : spécifiez les critères de filtre utilisés pour qualifier les enregistrements de conteneur lorsque vous effectuez une recherche dans l'annuaire.  Pour Active Directory et ADAM, (|(objectClass=organizationalUnit)(objectClass=container)) est généralement utilisé.  Pour d’autres répertoires LDAP, utilisez les critères de filtre qualifiant chaque type d’objet de conteneur en fonction du schéma du répertoire.  <br>Remarque :  Si ce champ n’est pas renseigné, ((objectClass=organizationalUnit)(objectClass=container)) est utilisé par défaut.
* **Filtre de groupes de sécurité** : spécifiez les critères de filtre utilisés pour qualifier les enregistrements de groupes de sécurité lorsque vous effectuez une recherche dans l'annuaire.  Pour Active Directory et ADAM, (&(objectCategory=group)(groupType:1.2.840.113556.1.4.804:=-2147483648)) est généralement utilisé.  Pour d’autres répertoires LDAP, utilisez les critères de filtre qualifiant chaque type d’objet de groupe de sécurité en fonction du schéma du répertoire.  <br>Remarque :  Si ce champ n’est pas renseigné, (&(objectCategory=group)(groupType:1.2.840.113556.1.4.804:=-2147483648)) est utilisé par défaut.
* **Filtre d'utilisateur** : spécifiez les critères de filtre utilisés pour qualifier les enregistrements utilisateur lorsque vous effectuez une recherche dans l'annuaire.  Pour Active Directory et ADAM, (&(objectClass=user)(objectCategory=person)) est généralement utilisé.  Pour d’autres répertoires LDAP, utilisez (objectClass=inetOrgPerson) ou autre paramètre similaire en fonction du schéma du répertoire. <br>Remarque :  Si ce champ n’est pas renseigné, (&(objectCategory=person)(objectClass=user)) est utilisé par défaut.

## <a name="attributes"></a>Attributs

Vous pouvez personnaliser les attributs pour un répertoire spécifique.  Cela vous permet d’ajouter des attributs personnalisés et de régler avec précision la synchronisation uniquement avec les attributs dont vous avez besoin. Utilisez le nom de l’attribut, tel que défini dans le schéma de l’annuaire pour la valeur de chaque champ d’attribut. Le tableau suivant fournit des informations supplémentaires sur chaque fonctionnalité.

Les attributs peuvent être entrés manuellement et ne doivent pas nécessairement correspondre à un attribut de la liste d’attributs.

![Personnaliser les attributs d’intégration d’annuaire du serveur MFA](./media/howto-mfaserver-dir-ad/dirint3.png)

| Fonctionnalité | Description |
| --- | --- |
| Identificateur unique |Entrez le nom de l'attribut qui sert d'identificateur unique pour le conteneur, le groupe de sécurité et les enregistrements utilisateur.  Dans Active Directory, il s'agit généralement de objectGUID. Les autres implémentations LDAP peuvent utiliser entryUUID ou autre paramètre similaire.  La valeur par défaut est objectGUID. |
| Type d'identificateur unique |Sélectionnez le type de l'attribut d'identificateur unique.  Dans Active Directory, l'attribut objectGUID est du type GUID. Les autres implémentations LDAP peuvent utiliser Chaîne ou Tableau d’octets ASCII.  La valeur par défaut est GUID. <br><br>Il est important de définir correctement ce type dans la mesure où les éléments de synchronisation sont référencés par leur identificateur unique. Le type d’identificateur unique est utilisé pour trouver directement l’objet dans le répertoire.  La définition de ce type sur Chaîne alors que le répertoire stocke la valeur sous forme de tableau d’octets de caractères ASCII empêche la synchronisation de fonctionner correctement. |
| Nom unique |Entrez le nom de l'attribut qui contient le nom unique de chaque enregistrement.  Dans Active Directory, il s'agit généralement de distinguishedName. Les autres implémentations LDAP peuvent utiliser entryDN ou autre paramètre similaire.  La valeur par défaut est distinguishedName. <br><br>Si un attribut contenant seulement le nom unique n’existe pas, l’attribut ads path peut être utilisé.  La partie « LDAP://\<server\>/ » du chemin d’accès est automatiquement supprimée. Seul le nom unique de l’objet est conservé. |
| Nom du conteneur |Entrez le nom de l'attribut qui contient le nom d'un enregistrement de conteneur.  La valeur de cet attribut s’affiche dans la hiérarchie de conteneur lors de l’importation à partir d’Active Directory ou l’ajout d’éléments de synchronisation.  La valeur par défaut est name. <br><br>Si des conteneurs différents utilisent des attributs différents pour leurs noms, séparez les attributs de nom de conteneur par des points-virgules.  Le premier attribut de nom de conteneur trouvé sur un objet de conteneur est utilisé pour afficher son nom. |
| Nom de groupe de sécurité |Entrez le nom de l'attribut qui contient le nom d'un enregistrement de groupe de sécurité.  La valeur de cet attribut s’affiche dans la liste des groupes de sécurité lors de l’importation à partir d’Active Directory ou lors de l’ajout d’éléments de synchronisation.  La valeur par défaut est name. |
| Nom d’utilisateur |Entrez le nom de l'attribut qui contient le nom d'utilisateur dans un enregistrement utilisateur.  La valeur de cet attribut est utilisée comme nom d’utilisateur du serveur MFA.  Un deuxième attribut peut être spécifié en tant que sauvegarde.  Le deuxième attribut n’est utilisé que si le premier attribut ne contient pas de valeur pour l’utilisateur.  Les valeurs par défaut sont userPrincipalName et sAMAccountName. |
| Prénom |Entrez le nom de l'attribut qui contient le prénom dans un enregistrement utilisateur.  La valeur par défaut est givenName. |
| Nom |Entrez le nom de l'attribut qui contient le nom dans un enregistrement utilisateur.  La valeur par défaut est sn. |
| Adresse de messagerie |Entrez le nom de l'attribut qui contient l'adresse de messagerie dans un enregistrement utilisateur.  L’adresse de messagerie est utilisée pour envoyer des e-mails de bienvenue et de mise à jour à l’utilisateur.  La valeur par défaut est mail. |
| Groupe d'utilisateurs |Entrez le nom de l'attribut qui contient le groupe d'utilisateurs dans un enregistrement utilisateur.  Le groupe d'utilisateurs peut être utilisé pour filtrer les utilisateurs sur l'agent et sur les rapports dans le portail de gestion du serveur MFA. |
| Description |Entrez le nom de l'attribut qui contient la description dans un enregistrement utilisateur.  La description est utilisée uniquement pour la recherche.  La valeur par défaut est description. |
| Langue des appels téléphoniques |Entrez le nom de l'attribut qui contient le nom court de la langue à utiliser pour les appels vocaux pour l'utilisateur. |
| Langue des SMS |Entrez le nom de l'attribut qui contient le nom court de la langue à utiliser pour les messages texte SMS pour l'utilisateur. |
| Langue de l’application mobile |Entrez le nom de l'attribut qui contient le nom court de la langue à utiliser pour les messages texte de l'application téléphonique pour l'utilisateur. |
| Langue du jeton OATH |Entrez le nom de l'attribut qui contient le nom court de la langue à utiliser pour les messages texte du jeton OATH pour l'utilisateur. |
| Téléphone professionnel |Entrez le nom de l'attribut qui contient le numéro de téléphone professionnel dans un enregistrement utilisateur.  La valeur par défaut est telephoneNumber. |
| Téléphone personnel |Entrez le nom de l'attribut qui contient le numéro de téléphone fixe dans un enregistrement utilisateur.  La valeur par défaut est homePhone. |
| Récepteur de radiomessagerie |Entrez le nom de l'attribut qui contient le numéro du récepteur de radiomessagerie dans un enregistrement utilisateur.  La valeur par défaut est pager. |
| Téléphone mobile |Entrez le nom de l'attribut qui contient le numéro de téléphone mobile dans un enregistrement utilisateur.  La valeur par défaut est mobile. |
| Fax |Entrez le nom de l'attribut qui contient le numéro du fax dans un enregistrement utilisateur.  La valeur par défaut est facsimileTelephoneNumber. |
| Téléphone IP |Entrez le nom de l'attribut qui contient le numéro de téléphone IP dans un enregistrement utilisateur.  La valeur par défaut est ipPhone. |
| Custom |Entrez le nom de l’attribut qui contient un numéro de téléphone personnalisé dans un enregistrement utilisateur.  La valeur par défaut est vide. |
| Extension |Entrez le nom de l'attribut qui contient le numéro de téléphone de l'extension dans un enregistrement utilisateur.  La valeur du champ d’extension est utilisée comme extension du numéro de téléphone principal uniquement.  La valeur par défaut est vide. <br><br>Si l’attribut Extension n’est pas spécifié, les extensions peuvent être incluses dans le cadre de l’attribut de téléphone. Dans ce cas, faites précéder l’extension d’un « x » afin qu’il soit correctement analysé.  Par exemple, 555-123-4567 x890 pourrait conduire à 555-123-4567 comme numéro de téléphone et 890 comme extension. |
| Bouton Paramètres par défaut |Cliquez sur **Paramètres par défaut** pour restaurer la valeur par défaut des attributs.  Les valeurs par défaut doivent fonctionner correctement avec le schéma Active Directory ou ADAM normal. |

Pour modifier les attributs, cliquez sur **Modifier** dans l’onglet Attributs.  Cette action ouvre une fenêtre dans laquelle vous pouvez modifier les attributs. Sélectionnez **...** en regard d’un attribut pour ouvrir une fenêtre dans laquelle vous pouvez choisir les attributs à afficher.

![Modifier le mappage d’attribut d’annuaire du serveur MFA](./media/howto-mfaserver-dir-ad/dirint4.png)

## <a name="synchronization"></a>Synchronization

La synchronisation maintient la base de données utilisateur Azure MFA synchronisée avec les utilisateurs dans Active Directory ou un autre répertoire LDAP (Lightweight Directory Access Protocol). Le processus est similaire à l'importation manuelle d'utilisateurs à partir d'Active Directory, mais il procède à des interrogations régulières pour connaître les modifications apportées aux utilisateurs et groupes de sécurité Active Directory à traiter.  De même, il désactive ou supprime les utilisateurs qui ont été supprimés d’un conteneur, d’un groupe de sécurité ou d’Active Directory.

Le service Multi-Factor Auth ADSync est un service Windows qui interroge régulièrement Active Directory.  Il n'est pas à confondre avec Azure AD Sync ou Azure AD Connect.  Multi-Factor Auth ADSync, bien que basé sur une base de code similaire, est spécifique au serveur Azure Multi-Factor Authentication.  Il est installé dans un état Arrêté et est démarré par le service du serveur MFA lorsqu'il est configuré pour s'exécuter.  Si vous avez une configuration de serveur MFA, Multi-Factor Auth ADSync ne peut être exécuté que sur un seul serveur.

Le service Multi-Factor Auth ADSync utilise l'extension de serveur LDAP DirSync fournie par Microsoft pour rechercher efficacement les modifications.  Cet appelant de contrôle DirSync doit disposer de l'autorisation « directory get changes » et du droit d'accès de contrôle étendu DS-Replication-Get-Changes.  Par défaut, ces droits sont affectés aux comptes Administrateur et LocalSystem sur des contrôleurs de domaine.  Le service Multi-Factor Auth AdSync est configuré pour s'exécuter en tant que LocalSystem par défaut.  Par conséquent, il est plus simple d'exécuter le service sur un contrôleur de domaine.  Si vous configurez le service pour toujours effectuer une synchronisation complète, il peut s’exécuter en tant que compte doté d’autorisations moindres.  Cela est moins efficace, mais nécessite moins de privilèges.

Si le répertoire LDAP prend en charge le contrôle DirSync et est configuré pour l’utiliser, alors l’interrogation pour connaître les modifications des utilisateurs et des groupes de sécurité fonctionnera de la même manière qu’avec Active Directory.  Si le répertoire LDAP ne prend pas en charge le contrôle DirSync, alors une synchronisation complète est effectuée lors de chaque cycle.

![Synchronisation d’objets annuaire avec le serveur MFA](./media/howto-mfaserver-dir-ad/dirint5.png)

Le tableau suivant contient des informations supplémentaires sur chaque paramètre de l’onglet Synchronisation.

| Fonctionnalité | Description |
| --- | --- |
| Activer la synchronisation avec Active Directory |Lorsque cette option est activée, le service de serveur MFA interroge régulièrement Active Directory pour connaître les modifications. <br><br>Remarque : Un élément de synchronisation au moins doit être ajouté et l’opération Synchroniser maintenant doit être effectuée avant que le service du serveur MFA ne commence à traiter les modifications. |
| Synchroniser toutes les |Spécifiez combien de temps le service du serveur MFA doit attendre entre l'interrogation et le traitement des modifications. <br><br> Remarque : L’intervalle spécifié correspond au temps entre le début de chaque cycle.  Si la durée de traitement des modifications dépasse l’intervalle, l’interrogation redémarre immédiatement. |
| Supprimer les utilisateurs n'étant plus dans Active Directory |Lorsque cette option est activée, le service du serveur MFA traite les objets tombstone des utilisateurs supprimés dans Active Directory et supprime l'utilisateur connexe. |
| Toujours effectuer une synchronisation complète |Lorsque cette option est activée, le service du serveur MFA effectue toujours une synchronisation complète.  Lorsqu'elle est désactivée, le service du serveur MFA effectue une synchronisation incrémentielle en interrogeant uniquement les utilisateurs qui ont été modifiés.  Par défaut, cette option est désactivée. <br><br>Quand elle est désactivée, le serveur Azure MFA procède à une synchronisation incrémentielle uniquement si le répertoire prend en charge le contrôle DirSync et que le compte utilisé pour la liaison au répertoire possède les autorisations nécessaires pour effectuer des requêtes incrémentielles DirSync.  Si le compte ne dispose pas des autorisations appropriées ou si plusieurs domaines sont impliqués dans la synchronisation, le serveur Azure MFA effectue une synchronisation complète. |
| Exiger l'approbation de l'administrateur lorsque le nombre d'utilisateurs désactivés ou supprimés dépasse X |Les éléments de synchronisation peuvent être configurés pour désactiver ou supprimer les utilisateurs qui ne font plus partie du groupe de sécurité ou du conteneur de l'élément.  Par précaution, il est possible d'exiger l'approbation de l'administrateur lorsque le nombre d'utilisateurs à désactiver ou à supprimer dépasse un seuil.  Lorsque cette option est activée, l’approbation est requise pour le seuil spécifié.  La valeur par défaut est 5 et la plage va de 1 à 999. <br><br> L'approbation est facilitée par l'envoi au préalable d'une notification par courrier électronique aux administrateurs. La notification par courrier électronique fournit des instructions sur la révision et l'approbation de la désactivation et la suppression des utilisateurs.  Lorsque l'interface utilisateur du serveur MFA est lancée, elle demande l'approbation. |

Le bouton **Synchroniser maintenant** permet d'exécuter une synchronisation complète des éléments de synchronisation spécifiés.  Une synchronisation complète est nécessaire à chaque fois que des éléments de synchronisation sont ajoutés, modifiés, supprimés ou réorganisés.  Elle est également nécessaire pour que le service Multi-Factor Auth AdSync soit opérationnel, dans la mesure où elle définit le point de départ à partir duquel le service interroge les modifications incrémentielles.  Si des modifications ont été apportées aux éléments de synchronisation mais qu’aucune synchronisation complète n’a été effectuée, vous devrez synchroniser immédiatement.

Le bouton **Supprimer** permet à l'administrateur de supprimer un ou plusieurs éléments de synchronisation dans la liste des éléments de synchronisation du serveur MFA.

> [!WARNING]
> Lorsqu'un enregistrement d'élément de synchronisation a été supprimé, il ne peut pas être récupéré. Vous devrez de nouveau ajouter cet enregistrement en cas de suppression par erreur.

L'élément ou les éléments de synchronisation ont été supprimés du serveur MFA.  Le service du serveur MFA ne traitera plus les éléments de synchronisation.

Les boutons Déplacer vers le haut et Déplacer vers le bas permettent à l'administrateur de modifier l'ordre des éléments de synchronisation.  L'ordre est important étant donné qu'un même utilisateur peut être membre de plusieurs éléments de synchronisation (par exemple, un conteneur et un groupe de sécurité).  Les paramètres appliqués à l'utilisateur pendant la synchronisation proviennent du premier élément de synchronisation dans la liste à laquelle l'utilisateur est associé.  Par conséquent, les éléments de synchronisation doivent être placés dans l'ordre de priorité.

> [!TIP]
> Une synchronisation complète doit être effectuée après la suppression d'éléments de synchronisation.  Une synchronisation complète doit être effectuée après la réorganisation d'éléments de synchronisation.  Cliquez sur **Synchroniser maintenant** pour effectuer une synchronisation complète.

## <a name="multi-factor-authentication-servers"></a>Serveurs Multi-Factor Authentication

Il est possible de configurer des serveurs MFA supplémentaires en tant que proxy RADIUS de secours, proxy LDAP ou pour l’authentification IIS. La configuration de la synchronisation est partagée par tous les agents. Toutefois, le service du serveur MFA ne peut s’exécuter que sur un seul agent. Cet onglet vous permet de sélectionner le serveur MFA qui doit être activé pour la synchronisation.

![Serveurs Multi-Factor Authentication associés](./media/howto-mfaserver-dir-ad/dirint6.png)
