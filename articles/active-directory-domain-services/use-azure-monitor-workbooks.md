---
title: Utiliser les classeurs Azure Monitor avec Azure AD Domain Services | Microsoft Docs
description: Découvrez comment utiliser Azure Monitor classeurs pour examiner les audits de sécurité et comprendre les problèmes d’un domaine managé Azure Active Directory Domain Services.
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 3067388265fb69b916fe0e179cb896401b7c2dc4
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618159"
---
# <a name="review-security-audit-events-in-azure-active-directory-domain-services-using-azure-monitor-workbooks"></a>Examiner les événements d’audit de sécurité dans Azure Active Directory Domain Services à l’aide d’Azure Monitor Workbooks

Pour mieux comprendre l’état de votre domaine managé Azure Active Directory Domain Services (Azure AD DS), vous pouvez activer des événements d’audit de sécurité. Il est alors possible d’examiner ces événements d’audit de sécurité à l’aide des classeurs Azure Monitor qui combinent du texte, des requêtes analytiques et des paramètres dans des rapports interactifs enrichis. Azure AD DS comprend des modèles de classeurs pour avoir une vue d’ensemble de la sécurité et l’activité des comptes et vous permettre ainsi d’explorer les événements d’audit et de gérer votre environnement.

Cet article explique comment utiliser des classeurs Azure Monitor pour examiner les événements d’audit de sécurité dans Azure AD DS.

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement, synchronisé avec un annuaire local ou un annuaire cloud uniquement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Un domaine managé Azure Active Directory Domain Services activé et configuré dans votre locataire Azure AD.
    * Si nécessaire, suivez le tutoriel pour [créer et configurer un domaine managé Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Événements d’audit de sécurité activés pour votre domaine managé qui diffusent en continu des données vers un espace de travail Log Analytics.
    * Si nécessaire, [activer les audits de sécurité pour Azure AD DS][enable-security-audits].

## <a name="azure-monitor-workbooks-overview"></a>Vue d’ensemble des classeurs Azure Monitor

Lorsque les événements d’audit de sécurité sont activés dans Azure AD DS, il peut être difficile d’analyser et d’identifier les problèmes dans le domaine managé. Azure Monitor vous permet d’agréger ces événements d’audit de sécurité et d’interroger les données. Les classeurs Azure Monitor vous permettent de visualiser ces données afin d’accélérer et de faciliter l’identification des problèmes.

Les modèles de classeur sont des rapports organisés conçus pour une réutilisation flexible par plusieurs utilisateurs et équipes. Lorsque vous ouvrez un modèle de classeur, les données de votre environnement Azure Monitor sont chargées. Vous pouvez utiliser des modèles sans incidence sur les autres utilisateurs de votre organisation, et vous pouvez enregistrer vos classeurs basés sur le modèle.

Azure AD DS comprend les deux modèles de classeurs suivants :

* Rapport Vue d’ensemble de la sécurité
* Rapport d’activité de compte

Pour plus d’informations sur la modification et la gestion des classeurs, consultez [Vue d’ensemble des classeurs Azure Monitor](../azure-monitor/platform/workbooks-overview.md).

## <a name="use-the-security-overview-report-workbook"></a>Utiliser le classeur de rapport Vue d’ensemble de la sécurité

Pour vous aider à mieux comprendre l’utilisation et à identifier les menaces de sécurité potentielles, le rapport Vue d’ensemble de la sécurité résume les données de connexion et identifie les comptes que vous souhaitez peut-être vérifier. Vous pouvez afficher les événements dans une plage de dates particulière et descendre dans la hiérarchie des événements de connexion spécifiques, tels que les tentatives de mot de passe incorrectes ou l’endroit où le compte a été désactivé.

Pour accéder au modèle de classeur pour le rapport de vue d’ensemble de la sécurité, procédez comme suit :

1. Recherchez et sélectionnez **Azure Active Directory Domain Services** dans le Portail Microsoft Azure.
1. Sélectionnez votre domaine managé, par exemple *aaddscontoso.com*.
1. Dans le menu de gauche, choisissez **Supervision > Classeurs**

    ![Capture d’écran qui indique où sélectionner le rapport de vue d’ensemble de la sécurité et le rapport d’activité de compte.](./media/use-azure-monitor-workbooks/select-workbooks-in-azure-portal.png)

1. Choisissez le **Rapport Vue d’ensemble de la sécurité**.
1. Dans les menus déroulants en haut du classeur, sélectionnez votre abonnement Azure, puis un espace de travail Azure Monitor.

    Choisissez un **intervalle de temps**, par exemple *7 derniers jours*, comme indiqué dans la capture d’écran suivante :

    ![Sélectionnez l’option de menu Classeurs dans le Portail Microsoft Azure.](./media/use-azure-monitor-workbooks/select-query-filters.png)

    Vous pouvez également modifier les options de l’affichage **Mosaïque** et de l’affichage **Graphique** pour analyser et visualiser les données selon vos besoins.

1. Pour descendre dans la hiérarchie des types d’événement, sélectionnez l’une des cartes **Résultat de connexion** par exemple *Compte verrouillé*, comme illustré dans l’exemple suivant :

    ![Exemple de données du rapport Vue d’ensemble de la sécurité visualisées dans les classeurs Azure Monitor](./media/use-azure-monitor-workbooks/example-security-overview-report.png)

1. La partie inférieure du rapport Vue d’ensemble de la sécurité figure sous le graphique décompose ensuite le type d’activité sélectionné. Sur le côté droit, vous pouvez filtrer par nom d’utilisateur impliqué, comme indiqué dans l’exemple de rapport suivant :

    [![Détails des verrouillages de compte dans les classeurs Azure Monitor.](./media/use-azure-monitor-workbooks/account-lockout-details-cropped.png)](./media/use-azure-monitor-workbooks/account-lockout-details.png#lightbox)

## <a name="use-the-account-activity-report-workbook"></a>Utiliser le classeur du rapport d’activité de compte

Pour vous aider à résoudre les problèmes liés à un compte d’utilisateur spécifique, le rapport d’activité du compte distingue les informations détaillées du journal des événements d’audit. Vous pouvez vérifier si un nom d’utilisateur ou un mot de passe incorrect a été fourni lors de la connexion, ainsi que la source de la tentative de connexion.

Pour accéder au modèle de classeur pour le rapport d’activité du compte, procédez comme suit :

1. Recherchez et sélectionnez **Azure Active Directory Domain Services** dans le Portail Microsoft Azure.
1. Sélectionnez votre domaine managé, par exemple *aaddscontoso.com*.
1. Dans le menu de gauche, choisissez **Supervision > Classeurs**
1. Choisissez le **rapport d’activité de compte**.
1. Dans les menus déroulants en haut du classeur, sélectionnez votre abonnement Azure, puis un espace de travail Azure Monitor.

    Choisissez une **Période**, par exemple *Les 30 derniers jours*, puis la manière dont vous souhaitez que l’affichage **Mosaïque** représente les données.

    Vous pouvez filtrer par **Nom d’utilisateur du compte**, par exemple *felix*, comme indiqué dans l’exemple de rapport suivant :

    [![Rapport d’activité de compte dans les classeurs Azure Monitor.](./media/use-azure-monitor-workbooks/account-activity-report-cropped.png)](./media/use-azure-monitor-workbooks/account-activity-report.png#lightbox)

    La zone située sous le graphique affiche les événements de connexion individuels, ainsi que des informations telles que le résultat de l’activité et la station de travail source. Ces informations peuvent aider à identifier les sources répétées d’événements de connexion qui peuvent provoquer le verrouillage de compte ou indiquer une attaque potentielle.

Comme pour le rapport Vue d’ensemble de la sécurité, vous pouvez accéder aux différentes vignettes en haut du rapport pour visualiser et analyser les données en fonction des besoins.

## <a name="save-and-edit-workbooks"></a>Enregistrer et modifier les classeurs

Les deux classeurs de modèles fournis par Azure AD DS constituent un excellent point de départ pour effectuer votre analyse de données. Si vous avez besoin d’une plus grande précision dans les requêtes et investigations de données, vous pouvez enregistrer vos classeurs et modifier les requêtes.

1. Pour enregistrer une copie de l’un des modèles de classeur, sélectionnez **Modifier > Enregistrer sous > Rapports partagés**, puis entrez un nom et enregistrez-le.
1. À partir de votre copie du modèle, sélectionnez **Modifier** pour passer en mode d’édition. Vous pouvez choisir le bouton bleu **Modifier** en regard d’une partie du rapport afin de la modifier.

Tous les graphiques et tables des classeurs Azure Monitor sont générés à l’aide de requêtes Kusto. Pour plus d’informations sur la création de vos requêtes, consultez [Requêtes de journal Azure Monitor][azure-monitor-queries] et [Didacticiel sur les requêtes Kusto][kusto-queries].

## <a name="next-steps"></a>Étapes suivantes

Si vous devez ajuster les stratégies de mot de passe et de verrouillage, consultez la rubrique [Stratégies de mot de passe et de verrouillage de compte sur les domaines managés][password-policy].

Pour les problèmes liés aux utilisateurs, découvrez résoudre les [problèmes de connexion de compte][troubleshoot-sign-in] et les [problèmes de verrouillage de compte][troubleshoot-account-lockout].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[enable-security-audits]: security-audit-events.md
[password-policy]: password-policy.md
[troubleshoot-sign-in]: troubleshoot-sign-in.md
[troubleshoot-account-lockout]: troubleshoot-account-lockout.md
[azure-monitor-queries]: /azure/data-explorer/kusto/query/
[kusto-queries]: /azure/kusto/query/tutorial?pivots=azuredataexplorer