---
title: Déterminer à quel moment un utilisateur spécifique pourra accéder à une application
description: Comment déterminer à quel moment un utilisateur très important pourra accéder à une application que vous avez configurée pour l’approvisionnement des utilisateurs avec Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 09/03/2019
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 307a97b71fe453c89617a86a88063e60fcf28fa3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994007"
---
# <a name="check-the-status-of-user-provisioning"></a>Vérifier l’état de l’approvisionnement d’utilisateurs

Le service d’approvisionnement d’Azure AD exécute un cycle d’approvisionnement initial sur le système source et le système cible, suivi de cycles incrémentiels périodiques. Lorsque vous configurez l’approvisionnement d’une application, vous pouvez vérifier l’état actuel du service d’approvisionnement et voir quand un utilisateur sera en mesure d’accéder à une application.

## <a name="view-the-provisioning-progress-bar"></a>Afficher la barre de progression de l’approvisionnement

 Sur la page **Approvisionnement** d’une application, vous pouvez afficher l’état du service d’approvisionnement d’Azure AD. La section **État actuel** en bas de la page indique si le cycle a démarré l’approvisionnement de comptes d’utilisateurs. Vous pouvez surveiller la progression du cycle, voir combien d’utilisateurs et de groupes ont été approvisionnés et combien de rôles sont créés.

Lorsque vous configurez pour la première fois l’approvisionnement automatique, la section **État actuel** au bas de la page affiche l’état initial du cycle d’approvisionnement. Cette section est mise à jour à chaque exécution d’un cycle incrémentiel. Les détails suivants sont affichés :
- Le type de cycle d’approvisionnement (initial ou incrémentiel) en cours d’exécution ou qui a été complété.
- Une **barre de progression** qui montre l’accomplissement de l’approvisionnement en pourcentage. Le pourcentage reflète le nombre de pages approvisionnées. Remarque : chaque page peut contenir plusieurs utilisateurs ou groupes. Le pourcentage ne reflète donc pas directement le nombre d’utilisateurs, de groupes ou de rôles approvisionnés.
- Un bouton **Actualiser** que vous pouvez utiliser pour mettre à jour la vue.
- Le nombre d’**utilisateurs** et de **groupes** dans le magasin de données du connecteur. Le nombre augmente dès qu’un objet est ajouté à l’étendue de l’approvisionnement. Le nombre ne diminue pas lors de la suppression réversible ou définitive d’un utilisateur, étant donné que cela ne supprime pas l’objet du magasin de données du connecteur. Le nombre est recalculé lors de la première synchronisation qui suit la [réinitialisation](/graph/api/synchronization-synchronizationjob-restart?tabs=http&view=graph-rest-beta) du CDS. 
- Un lien **Afficher les journaux d’audit**, qui ouvre les journaux de provisionnement Azure AD pour plus d’informations sur toutes les opérations exécutées par le service de provisionnement d’utilisateurs, notamment l’état de provisionnement d’utilisateurs individuels (voir la section [Utiliser des journaux de provisionnement](#use-provisioning-logs-to-check-a-users-provisioning-status) ci-dessous).

À la fin d’un cycle d’approvisionnement, la section **Statistiques à ce jour** affiche le nombre cumulatif d’utilisateurs et de groupes qui ont été approvisionnés à ce jour, ainsi que la date d’achèvement et la durée du dernier cycle. L’**ID d’activité** identifie uniquement le cycle d’approvisionnement le plus récent. L’**ID de tâche** est un identificateur unique de la tâche d’approvisionnement et est spécifique à l’application dans votre client.

La progression de l’approvisionnement peut être consultée dans le portail Azure, sous l’onglet **Azure Active Directory &gt; Applications d’entreprise &gt; \[Nom de l’application\] &gt;Approvisionnement**.

![Barre de progression sur la page de l’approvisionnement](./media/application-provisioning-when-will-provisioning-finish-specific-user/provisioning-progress-bar-section.png)

## <a name="use-provisioning-logs-to-check-a-users-provisioning-status"></a>Utiliser les journaux de provisionnement pour vérifier l’état de provisionnement d’un utilisateur

Pour consulter l’état de provisionnement d’un utilisateur sélectionné, consultez les [Journaux de provisionnement (préversion)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) dans Azure AD. Toutes les opérations effectuées par le service de provisionnement d’utilisateurs sont enregistrées dans les journaux de provisionnement Azure AD. Cela comprend toutes les opérations de lecture et d’écriture effectuées sur les systèmes sources et cibles, ainsi que les données utilisateur qui ont été lues ou écrites à chaque opération.

Vous pouvez accéder aux journaux d’approvisionnement dans le portail Azure en sélectionnant **Azure Active Directory** &gt; **Applications d’entreprise** &gt; **Journaux de provisionnement (préversion)** dans la section **Activité**. Vous pouvez rechercher les données de provisionnement en fonction du nom de l’utilisateur ou de l’identificateur dans le système source ou le système cible. Pour plus d’informations, consultez [Journaux de provisionnement (préversion)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context). 

Les journaux de provisionnement enregistrent toutes les opérations effectuées par le service de provisionnement, y compris :

* Interrogation d’Azure AD concernant les utilisateurs assignés qui se trouvent dans l’étendue de l’approvisionnement
* Interrogation de l’application cible concernant l’existence de ces utilisateurs
* Comparaison des objets utilisateur du système
* Ajout, mise à jour ou désactivation du compte d’utilisateur dans le système cible en fonction de la comparaison

Pour plus d’informations sur la lecture des journaux de provisionnement dans le portail Azure, consultez le [guide de création de rapports sur le provisionnement](check-status-user-account-provisioning.md).

## <a name="how-long-will-it-take-to-provision-users"></a>Combien de temps faut-il pour approvisionner des utilisateurs ?
Lorsque vous utilisez le provisionnement automatique des utilisateurs avec une application, Azure AD provisionne et met à jour automatiquement les comptes d’utilisateur dans une application selon différents éléments comme l’[affectation d’utilisateurs et de groupes](../manage-apps/assign-user-or-group-access-portal.md) et selon un intervalle régulier planifié (généralement toutes les 40 minutes).

Le temps de provisionnement d’un utilisateur donné dépend principalement de votre tâche de provisionnement, si elle est en cours d’exécution d’un cycle initial ou incrémentiel.

- Pour le **cycle initial**, le temps nécessaire dépend de plusieurs facteurs, notamment le nombre d’utilisateurs et de groupes dans l’étendue du provisionnement, ainsi que le nombre total d’utilisateurs et de groupes dans le système source. La première synchronisation entre Azure AD et une application peut prendre de 20 minutes à plusieurs heures, en fonction de la taille de l’annuaire Azure AD et du nombre d’utilisateurs dans l’étendue de l’approvisionnement. Une liste complète des facteurs affectant les performances du cycle initial est présentée plus loin dans cette section.

- Pour les **cycles incrémentiels** qui suivent le cycle initial, les durées des tâches ont tendance à être plus rapides (10 minutes, par exemple), car le service de provisionnement stocke les filigranes qui représentent l’état des deux systèmes après le cycle initial, ce qui améliore les performances des synchronisations suivantes. Le temps nécessaire dépend du nombre de modifications détectées dans ce cycle d’approvisionnement. S’il existe moins de 5 000 utilisateurs ou les changements d’appartenance au groupe, le travail peut finir dans un cycle d’approvisionnement incrémentiel. 

Le tableau suivant récapitule les temps de synchronisation des scénarios d’approvisionnement courants. Dans ces scénarios, le système source est Azure AD et le système cible est une application SaaS. Les temps de synchronisation sont dérivés d’une analyse statistique des travaux de synchronisation pour les applications SaaS ServiceNow, Workplace, Salesforce et G Suite.


| Configuration d’étendue | Utilisateurs, groupes et membres dans l’étendue | Durée du cycle initial | Durée du cycle incrémentiel |
| -------- | -------- | -------- | -------- |
| Synchroniser les utilisateurs et groupes assignés uniquement |  < 1 000 |  < 30 minutes | < 30 minutes |
| Synchroniser les utilisateurs et groupes assignés uniquement |  1 000 - 10 000 | 142 - 708 minutes | < 30 minutes |
| Synchroniser les utilisateurs et groupes assignés uniquement |   10 000 - 100 000 | 1 170 - 2 340 minutes | < 30 minutes |
| Synchroniser l'ensemble des utilisateurs et groupes dans Azure AD |  < 1 000 | < 30 minutes  | < 30 minutes |
| Synchroniser l'ensemble des utilisateurs et groupes dans Azure AD |  1 000 - 10 000 | < 30 - 120 minutes | < 30 minutes |
| Synchroniser l'ensemble des utilisateurs et groupes dans Azure AD |  10 000 - 100 000  | 713 - 1 425 minutes | < 30 minutes |
| Synchroniser l’ensemble des utilisateurs dans Azure AD|  < 1 000  | < 30 minutes | < 30 minutes |
| Synchroniser l’ensemble des utilisateurs dans Azure AD | 1 000 - 10 000  | 43 - 86 minutes | < 30 minutes |

En ce qui concerne la configuration **Synchroniser uniquement les utilisateurs et groupes assignés**, vous pouvez utiliser les formules suivantes et déterminer les temps minimum et maximum attendus pour le **cycle initial** :

- Nombre minimal de minutes = 0,01 x [nombre d’utilisateurs, de groupes et de membres de groupe affectés]
- Nombre maximal de minutes = 0,08 x [nombre d’utilisateurs, de groupes et de membres de groupe affectés]

Récapitulatif des facteurs ayant une incidence sur le temps nécessaire à l’accomplissement d’un **cycle initial** :

- Nombre total d'utilisateurs et de groupes concernés par l'approvisionnement.

- Nombre total d'utilisateurs, de groupes et de membres de groupe présents dans le système source (Azure AD).

- Si les utilisateurs dans l’étendue pour l’approvisionnement sont mis en correspondance ou non avec des utilisateurs existants dans l’application cible, ou doivent être créés la première fois. Les travaux de synchronisation pour lesquels tous les utilisateurs sont créés pour la première fois prennent environ *deux fois plus* de temps que les travaux de synchronisation pour lesquels tous les utilisateurs sont mis en correspondance avec des utilisateurs existants.

- Nombre d’erreurs dans les [journaux de provisionnement](check-status-user-account-provisioning.md). Les performances sont ralenties s'il y a beaucoup d'erreurs et que le service d'approvisionnement est passé en quarantaine. 

- Limites de taux de requêtes et limitation implémentées par le système cible. Certains systèmes cible implémentent des limites de taux de requêtes et une limitation qui peuvent affecter les performances lors d’opérations de synchronisation de grande envergure. Dans ces conditions, une application qui reçoit trop rapidement un trop grand nombre de requêtes risque d’afficher un taux de réponse plus faible ou d’interrompre la connexion. Pour améliorer les performances, le connecteur doit ajuster ce taux en évitant d’envoyer les requêtes d’application plus rapidement que ce que l’application est capable de traiter. Les connecteurs d’approvisionnement intégrés à Microsoft permettent d’effectuer cet ajustement. 

- Le nombre et la taille des groupes affectés. La synchronisation des groupes affectés prend plus de temps que la synchronisation des utilisateurs. Le nombre et la taille des groupes affectés ont un impact sur les performances. Si une application comporte des [mappages activés pour la synchronisation des objets de groupe](customize-application-attributes.md#editing-group-attribute-mappings), les propriétés de groupe, telles que les noms de groupe et les appartenances, sont synchronisées en plus des utilisateurs. Ces synchronisations supplémentaires prendront plus temps que la seule synchronisation des objets utilisateur.

- Si les performances deviennent un problème et que vous essayez de provisionner la majorité des utilisateurs et des groupes dans votre locataire, utilisez des filtres d’étendue. Les filtres d’étendue vous permettent d’affiner les données que le service d’approvisionnement extrait d’Azure AD en filtrant les utilisateurs en fonction d’attributs spécifiques. Pour plus d’informations sur les filtres d’étendue, consultez [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

## <a name="next-steps"></a>Étapes suivantes
[Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure Active Directory](user-provisioning.md)