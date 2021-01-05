---
title: Facteurs affectant les performances d’Azure AD Connect
description: Ce document décrit les différents facteurs qui impactent les performances du moteur de provisionnement Azure AD Connect. Les organisations doivent tenir compte de ces facteurs pour planifier leur déploiement Azure AD Connect conformément à leurs exigences de synchronisation.
services: active-directory
author: billmath
manager: daveba
tags: azuread
ms.service: active-directory
ms.subservice: hybrid
ms.topic: conceptual
ms.workload: identity
ms.date: 10/06/2018
ms.reviewer: martincoetzer
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 15bcb0f7ca30c343072da396abeac8d08dee03a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90087007"
---
# <a name="factors-influencing-the-performance-of-azure-ad-connect"></a>Facteurs affectant les performances d’Azure AD Connect

Azure AD Connect gère la synchronisation entre Active Directory et Azure AD. Ce serveur est un composant essentiel de la migration des identités d’utilisateur vers le cloud. Le tableau suivant répertorie les principaux facteurs qui impactent les performances d’Azure AD Connect :

| **Facteur de conception**| **Définition** |
|:-|-|
| Topologie| Distribution des points de terminaison et des composants qu’Azure AD Connect doit manager sur le réseau. |
| Scale| Nombre d’objets, tels que les utilisateurs, les groupes et les unités d’organisation, qui seront managés par Azure AD Connect. |
| Matériel| Matériel (physique ou virtuel) utilisé pour Azure AD Connect et chaque composant matériel dont la capacité impacte les performances, notamment l’UC, la mémoire, le réseau et la configuration du disque dur. |
| Configuration| Mode de traitement des annuaires et des informations par Azure AD Connect. |
| Load| Fréquence des modifications d’objets. Les charges peuvent varier au cours d’une heure, d’un jour ou d’une semaine. En fonction du composant, vous devrez peut-être prévoir des pics de charge ou une charge moyenne. |

L’objectif de ce document est de décrire les facteurs qui impactent les performances du moteur de provisionnement Azure AD Connect. Les organisations complexes ou de grande taille (celles devant provisionner plus de 100 000 objets) peuvent suivre les suggestions pour optimiser leur implémentation d’Azure AD Connect, en particulier si elles rencontrent certains problèmes de performances décrits dans ce document. Les autres composants d’Azure AD Connect, comme [Azure AD Connect Health](how-to-connect-health-agent-install.md) et les agents ne sont pas abordés ici.

> [!IMPORTANT]
> Microsoft ne prend pas en charge la modification ou l’utilisation d’Azure AD Connect en dehors des actions qui sont documentées de façon formelle. Ces actions peuvent entraîner un état de synchronisation Azure AD Connect incohérent ou non pris en charge. Par conséquent, Microsoft ne peut pas fournir de support technique pour ces déploiements.

## <a name="azure-ad-connect-component-factors"></a>Facteurs impactant les composants d’Azure AD Connect

Le diagramme ci-dessous illustre l’architecture générale d’un moteur de provisionnement connecté à une seule forêt, mais la connexion à plusieurs forêts est également prise en charge. Cette architecture montre les interactions entre les divers composants.

![Le diagramme montre comment les annuaires connectés et le moteur de provisionnement Azure AD Connect interagissent, y compris l’espace de connecteur et les composants Metaverse dans une base de données SQL. ](media/plan-connect-performance-factors/AzureADConnentInternal.png)

Le moteur de provisionnement se connecte à chaque forêt Active Directory et à Azure AD. Importer est le processus consistant à obtenir des informations de chaque annuaire. Exporter fait référence à la mise à jour des annuaires à partir du moteur de provisionnement. Synchroniser est l’opération qui évalue les règles de transfert des objets au sein du moteur de provisionnement. Pour approfondir vos connaissances, consultez [Azure AD Connect Sync : présentation de l'architecture](./concept-azure-ad-connect-sync-architecture.md).

Azure AD Connect utilise les zones de transit, les règles et les processus suivants pour effectuer la synchronisation entre Active Directory et Azure AD :

* **Espace connecteur (CS)** : les objets de chaque annuaire connecté (CD), les annuaires réels, sont d’abord mis en transit dans cet espace en attendant d’être traités par le moteur de provisionnement. Azure AD a son propre espace connecteur, tout comme chaque forêt à laquelle vous vous connectez.
* **Métaverse (MV)** : les objets à synchroniser sont créés ici selon les règles de synchronisation définies. Les objets doivent avoir été créés dans le métaverse afin de pouvoir ensuite remplir des objets et des attributs dans les autres annuaires connectés. Il y a un seul métaverse.
* **Règles de synchronisation** : elles déterminent quels objets seront créés (projetés) ou quels objets seront connectés (joints) aux objets dans le métaverse. Les règles de synchronisation déterminent également quelles valeurs d’attribut seront copiées ou transformées vers et depuis les annuaires.
* **Profils d’exécution** : ils regroupent les étapes du processus de copie des objets et de leurs valeurs d’attribut, selon les règles de synchronisation définies, entre les zones de transit et les annuaires connectés.

Il existe plusieurs profils d’exécution pour optimiser les performances du moteur de provisionnement. La plupart des organisations utilisent les planifications et profils d’exécution par défaut pour les opérations habituelles. Toutefois, certaines organisations peuvent avoir besoin de [modifier la planification](./how-to-connect-sync-feature-scheduler.md) ou de déclencher d’autres profils d’exécution pour faire face à des situations particulières. Les profils d’exécution suivants sont disponibles :

### <a name="initial-sync-profile"></a>Profil de synchronisation initiale

Le profil de synchronisation initiale est le processus consistant à obtenir les informations des annuaires connectés (comme une forêt Active Directory) pour la première fois. Il effectue ensuite une analyse de toutes les entrées dans la base de données du moteur de synchronisation. Le cycle initial crée des objets dans Azure AD ; il prend plus de temps si vos forêts Active Directory sont de grande taille. La synchronisation initiale comprend les étapes suivantes :

1. Importation complète sur tous les connecteurs
2. Synchronisation complète sur tous les connecteurs
3. Exportation sur tous les connecteurs

### <a name="delta-sync-profile"></a>Profil de synchronisation différentielle

Pour accélérer le processus de synchronisation, ce profil d’exécution traite uniquement les modifications (créations, suppressions et mises à jour) ayant été apportées aux objets dans les annuaires connectés depuis la dernière synchronisation. Par défaut, le profil de synchronisation différentielle s’exécute toutes les 30 minutes. Les organisations doivent s’efforcer de maintenir la durée d’exécution à moins de 30 minutes afin de tenir Azure AD toujours à jour. Pour superviser l’intégrité d’Azure AD Connect, utilisez [l’agent de supervision de l’intégrité](how-to-connect-health-sync.md), qui vous aide à détecter les éventuels problèmes avec le processus. Le profil de synchronisation différentielle comprend les étapes suivantes :

1. Importation différentielle sur tous les connecteurs
2. Synchronisation différentielle sur tous les connecteurs
3. Exportation sur tous les connecteurs

Voici un scénario de synchronisation différentielle standard dans une entreprise :

- Environ 1 % des objets sont supprimés
- Environ 1 % des objets sont créés
- Environ 5 % des objets sont modifiés

Le taux de modification peut varier selon la fréquence à laquelle votre organisation met à jour les utilisateurs dans Active Directory. Par exemple, le taux de modification peut être plus élevé durant les périodes de forte augmentation ou réduction du personnel.

### <a name="full-sync-profile"></a>Profil de synchronisation complète

Un cycle de synchronisation complète est nécessaire si vous avez effectué une ou plusieurs des modifications de configuration suivantes :



- Vous avez élargi l’étendue des objets ou des attributs à importer des annuaires connectés. Par exemple, vous avez ajouté un domaine ou une unité d’organisation à l’étendue d’importation.
- Vous avez changé les règles de synchronisation. Par exemple, vous avez créé une règle supplémentaire pour remplir la fonction des utilisateurs dans Azure AD avec l’attribut extension_attribute3 défini dans Active Directory. Pour appliquer ce changement, le moteur de provisionnement doit réexaminer tous les utilisateurs existants pour mettre à jour leurs fonctions.

Le profil de synchronisation complète comprend les étapes suivantes :

1. Importation complète sur tous les connecteurs
2. Synchronisation complète/différentielle sur tous les connecteurs
3. Exportation sur tous les connecteurs

> [!NOTE]
> Une planification soigneuse est nécessaire quand vous modifiez en bloc de nombreux objets dans Active Directory ou Azure AD. Après une mise à jour en bloc, l’importation par le processus de synchronisation différentielle prend plus de temps dans la mesure où beaucoup d’objets ont été modifiés. Des importations longues peuvent avoir lieu même si la mise à jour en bloc n’influe pas sur le processus de synchronisation. Par exemple, l’attribution de licences à de nombreux utilisateurs dans Azure AD déclenche un long cycle d’importation depuis Azure AD, sans entraîner de modifications d’attributs dans Active Directory.

### <a name="synchronization"></a>Synchronization

L’exécution du processus de synchronisation présente les caractéristiques de performances suivantes :

* La synchronisation est à thread unique, ce qui signifie que le moteur de provisionnement ne traite pas en parallèle les profils d’exécution des différents annuaires, objets ou attributs connectés.
* La durée de l’importation augmente de manière linéaire en fonction du nombre d’objets à synchroniser. Par exemple, si l’importation de 10 000 objets prend 10 minutes, l’importation de 20 000 objets prendra environ 20 minutes sur le même serveur.
* L’exportation suit également un schéma linéaire.
* La synchronisation augmente de manière exponentielle en fonction du nombre d’objets référençant d’autres objets. Les appartenances aux groupes et les groupes imbriqués ont le plus fort impact sur les performances, car leurs membres référencent des objets utilisateur ou d’autres groupes. Ces références doivent être définies et référencer des objets réels dans le métaverse pour permettre le cycle de synchronisation.

### <a name="filtering"></a>Filtrage

La taille de la topologie Active Directory à importer est le facteur qui impacte le plus les performances et la durée d’exécution des composants internes du moteur de provisionnement.

Utilisez le [filtrage](./how-to-connect-sync-configure-filtering.md) pour réduire le nombre d’objets à synchroniser. Vous éviterez ainsi que des objets soient inutilement traités et exportés vers Azure AD. Dans l’ordre de préférence, voici les méthodes de filtrage à votre disposition :



- **Filtrage par domaine** : utilisez cette option pour sélectionner des domaines spécifiques à synchroniser dans Azure AD. Vous devez ajouter et supprimer des domaines au niveau de la configuration du moteur de synchronisation lorsque vous apportez des modifications à votre infrastructure locale après avoir installé la synchronisation Azure AD Connect.
- **Filtrage par unité d’organisation** : utilisez des unités d’organisation pour cibler des objets spécifiques dans les domaines Active Directory en vue de leur provisionnement dans Azure AD. Le filtrage par unité d’organisation est la deuxième méthode de filtrage recommandée, car elle utilise des requêtes d’étendue LDAP simples pour importer un sous-ensemble plus petit d’objets à partir d’Active Directory.
- **Filtrage d’attributs par objet** : utilisez les valeurs d’attribut des objets pour déterminer quels objets spécifiques dans Active Directory doivent être provisionnés dans Azure AD. Le filtrage des attributs permet de définir des filtres plus précis, quand le filtrage par domaine ou par unité d’organisation ne répond pas à vos besoins. Le filtrage des attributs n’accélère pas l’importation, mais peut réduire les durées d’exportation et de synchronisation.
- **Filtrage par groupe** : utilisez l’appartenance au groupe pour déterminer quels objets doivent être provisionnés dans Azure AD. Le filtrage par groupe convient aux environnements de test uniquement, mais n’est pas recommandé pour les environnements de production. En effet, cette méthode donne lieu à une charge supplémentaire pour vérifier l’appartenance au groupe durant le cycle de synchronisation.

La présence d’un grand nombre [d’objets déconnecteurs](concept-azure-ad-connect-sync-architecture.md#relationships-between-staging-objects-and-metaverse-objects) persistants dans votre espace connecteur Active Directory risque d’augmenter les temps de synchronisation, car le moteur de provisionnement doit réévaluer chaque objet déconnecteur pour permettre la reconnexion durant le cycle de synchronisation. Pour résoudre ce problème, envisagez l’une des suggestions suivantes :



- Retirez les objets déconnecteurs de l’étendue d’importation à l’aide du filtrage par domaine ou unité d’organisation.
- Projetez/joignez les objets dans le métaverse et définissez l’attribut [cloudFiltered](how-to-connect-sync-configure-filtering.md#negative-filtering-do-not-sync-these) sur True pour empêcher le provisionnement de ces objets dans l’espace connecteur Azure AD.

> [!NOTE]
> Le filtrage d’un trop grand nombre d’objets peut être complexe pour les utilisateurs ou entraîner des problèmes d’autorisations d’application. Par exemple, dans une implémentation Exchange hybride Online, les utilisateurs ayant des boîtes aux lettres locales verront davantage d’utilisateurs dans leur liste d’adresses globale que les utilisateurs de boîtes aux lettres Exchange Online. Dans d’autres cas, un utilisateur pourra vouloir accorder l’accès à une application cloud à un autre utilisateur qui n’est pas inclus dans l’étendue des objets filtrés.

### <a name="attribute-flows"></a>Flux de valeurs d’attribut

Les flux de valeurs d’attribut désignent le processus de copie ou de transformation des valeurs d’attribut des objets d’un annuaire connecté dans un autre annuaire connecté. Ils sont définis dans les règles de synchronisation. Par exemple, quand le numéro de téléphone d’un utilisateur est changé dans votre annuaire Active Directory, ce numéro de téléphone est également mis à jour dans Azure AD. Les organisations peuvent [modifier les flux de valeurs d’attribut](./how-to-connect-sync-change-the-configuration.md) selon leurs besoins. Il est recommandé de faire une copie des flux de valeurs d’attribut existants avant de les modifier.

Les redirections simples, comme passer une valeur d’attribut à un autre attribut, n’impactent pas les performances matérielles. Un exemple de redirection est le transfert d’un numéro de téléphone mobile dans Active Directory vers le numéro de téléphone fixe dans Azure AD.

La transformation des valeurs d’attribut peut avoir un impact sur les performances au moment du processus de synchronisation. Transformer des valeurs d’attribut inclut la modification, le reformatage, la concaténation ou la soustraction de valeurs d’attribut.

Les organisations peuvent empêcher le transfert de certains attributs vers Azure AD, mais cela n’impacte pas les performances du moteur de provisionnement.

> [!NOTE]
> Ne supprimez pas les flux de valeurs d’attribut inutiles dans vos règles de synchronisation. Il est préférable de les désactiver, étant donné que les règles supprimées sont recréées lors des mises à niveau d’Azure AD Connect.

## <a name="azure-ad-connect-dependency-factors"></a>Facteurs de dépendance pour Azure AD Connect

Les performances d’Azure AD Connect sont dépendantes des performances des annuaires connectés importés et exportés. Par exemple, elles dépendent de la taille de l’importation Active Directory ou de la latence du réseau pour accéder au service Azure AD. La base de données SQL utilisée par le moteur de provisionnement a également un impact sur les performances globales du cycle de synchronisation.

### <a name="active-directory-factors"></a>Facteurs liés à Active Directory

Comme nous l’avons mentionné précédemment, le nombre d’objets à importer impacte considérablement les performances. L’article [Conditions préalables pour Azure AD Connect](how-to-connect-install-prerequisites.md) présente la configuration matérielle requise en fonction de la taille du déploiement. Azure AD Connect prend uniquement en charge les topologies spécifiques décrites dans [Topologies pour Azure AD Connect](plan-connect-topologies.md). Nous ne proposons pas de solutions ni de suggestions d’optimisation des performances pour les topologies non prises en charge.

Assurez-vous que le serveur Azure AD Connect est conforme à la configuration matérielle requise pour la taille de votre importation Active Directory. Le processus d’importation peut être ralenti si la connectivité réseau est lente ou mauvaise entre le serveur Azure AD Connect et vos contrôleurs de domaine Active Directory.

### <a name="azure-ad-factors"></a>Facteurs liés à Azure AD

Azure AD applique la limitation de requêtes pour protéger le service cloud contre les attaques par déni de service (DoS). Actuellement, Azure AD a une limite de 7 000 requêtes d’écriture toutes les 5 minutes (84 000 par heure). Par exemple, les opérations suivantes peuvent être limitées :



- Exportations d’Azure AD Connect vers Azure AD.
- Mises à jour par des applications ou scripts PowerShell directement dans Azure AD, y compris en arrière-plan, comme les appartenances de groupe dynamiques.
- Mises à jour par les utilisateurs de leurs propres enregistrements d’identité, comme l’inscription auprès du service MFA (authentification multifacteur) ou SSPR (réinitialisation du mot de passe libre-service).
- Opérations au sein de l’interface graphique utilisateur.

Planifiez les tâches de déploiement et de maintenance pour vous assurer que la limitation des requêtes n’impacte pas le cycle de synchronisation Azure AD Connect. Par exemple, si votre organisation connaît une forte période de recrutement qui nécessite la création de milliers d’identités d’utilisateur, cela peut entraîner de nombreuses mises à jour des appartenances de groupe dynamiques, des attributions de licences et des inscriptions au service de réinitialisation de mot de passe libre-service. Il est alors préférable d’échelonner ces écritures sur plusieurs heures voire quelques jours.

### <a name="sql-database-factors"></a>Facteurs liés à la base de données SQL

La taille de votre topologie Active Directory source impacte les performances de votre base de données SQL. Respectez la [configuration matérielle requise](how-to-connect-install-prerequisites.md) pour le serveur de la base de données SQL et prenez en compte les suggestions suivantes :



- Les organisations de plus de 100 000 utilisateurs peuvent réduire les latences du réseau en installant la base de données SQL et le moteur de provisionnement sur le même serveur.
- En raison de la grande quantité d’entrées et de sorties (E/S) nécessaires pour le processus de synchronisation, utilisez des disques SSD pour la base de données SQL du moteur de provisionnement afin d’avoir des résultats optimaux. Si cela n’est pas possible, utilisez des disques RAID 0 ou RAID 1.
- Ne lancez pas de synchronisation complète de façon préventive, car cela entraîne une surcharge inutile et augmente les temps de réponse.

## <a name="conclusion"></a>Conclusion

Pour optimiser les performances de votre implémentation d’Azure AD Connect, tenez compte des suggestions suivantes :



- Respectez la [configuration matérielle recommandée](how-to-connect-install-prerequisites.md) pour le serveur Azure AD Connect en fonction de la taille de votre implémentation.
- Quand vous effectuez une mise à niveau d’Azure AD Connect dans des déploiements à grande échelle, utilisez la [méthode de migration « swing »](./how-to-upgrade-previous-version.md#swing-migration) pour garantir un temps d’arrêt minimum et une fiabilité optimale. 
- Utilisez un disque SSD pour la base de données SQL afin d’optimiser les performances d’écriture.
- Filtrez l’étendue Active Directory pour inclure uniquement les objets nécessitant d’être provisionnés dans Azure AD, à l’aide du filtrage par domaine ou unité d’organisation, ou du filtrage des attributs.
- Si vous devez modifier les règles de flux de valeurs d’attribut par défaut, copiez la règle, modifiez la copie et désactivez la règle d’origine. N’oubliez pas de réexécuter une synchronisation complète.
- Prévoyez suffisamment de temps pour le profil d’exécution de la synchronisation complète initiale.
- Efforcez-vous de planifier un cycle de synchronisation différentielle de moins de 30 minutes. Si le profil de synchronisation différentielle s’exécute en plus de 30 minutes, changez la fréquence de synchronisation par défaut pour inclure un cycle complet de synchronisation différentielle.
- Supervisez l’agent [Azure AD Connect Health pour la synchronisation](how-to-connect-health-agent-install.md) dans Azure AD.

## <a name="next-steps"></a>Étapes suivantes
En savoir plus sur l’ [intégration de vos identités locales avec Azure Active Directory](whatis-hybrid-identity.md).