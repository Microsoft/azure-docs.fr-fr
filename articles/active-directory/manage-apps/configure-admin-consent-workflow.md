---
title: Configurer le flux de travail de consentement de l’administrateur-Azure Active Directory | Microsoft Docs
description: Découvrez comment configurer un moyen pour les utilisateurs finaux de demander l’accès aux applications qui requièrent le consentement de l’administrateur.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 10/29/2019
ms.author: kenwith
ms.reviewer: luleon
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7fc688ca0981024e5e08c9fb20ecb6704efbd7f2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997492"
---
# <a name="configure-the-admin-consent-workflow-preview"></a>Configurer le workflow du consentement administrateur (préversion)

Cet article explique comment activer la fonctionnalité de workflow du consentement de l’administrateur, qui donne aux utilisateurs finaux la possibilité de demander l’accès aux applications qui requièrent le consentement de l’administrateur.

Sans un workflow de consentement de l’administrateur, un utilisateur dans un locataire où le consentement de l’utilisateur est désactivé est bloqué lorsqu’il tente d’accéder à une application qui requiert des autorisations pour accéder aux données de l’organisation. L’utilisateur voit un message d’erreur générique qui indique qu’il n’est pas autorisé à accéder à l’application et qu’il doit demander de l’aide à son administrateur. Mais souvent, l’utilisateur ne sait pas qui il doit contacter, et il peut donc abandonner ou créer un nouveau compte local dans l’application. Même lorsqu’un administrateur est averti, il n’existe pas toujours un processus rationalisé pour l’aider à accorder l’accès et à informer ses utilisateurs.
 
Le workflow de consentement de l’administrateur offre aux administrateurs une méthode sécurisée pour accorder l’accès aux applications qui requièrent l’approbation de l’administrateur. Lorsqu’un utilisateur tente d’accéder à une application, mais qu’il n’est pas en mesure de donner son consentement, il peut envoyer une demande d’approbation de l’administrateur. La requête est envoyée par courrier électronique aux administrateurs qui ont été désignés comme réviseurs. Un réviseur entreprend une action sur la demande et l’utilisateur est averti de l’action.

Pour approuver des demandes, un réviseur doit être un administrateur général, un administrateur d’application cloud ou un administrateur d’application. Le réviseur doit déjà avoir un de ces rôles d’administrateur attribués : leur simple désignation en tant que réviseur n’élève pas ses privilèges.

## <a name="enable-the-admin-consent-workflow"></a>Activer le workflow de consentement de l’administrateur

Pour activer le workflow de consentement de l’administrateur et choisir les réviseurs :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant qu’administrateur.
2. Cliquez sur **All services** (Tous les services) en haut du menu de navigation de gauche. Le panneau **Extension Azure Active Directory** apparaît.
3. Dans la zone de recherche de filtre, tapez « **Azure Active Directory** », puis sélectionnez l’élément **Azure Active Directory**.
4. Dans le menu de navigation, cliquez sur **Applications d’entreprise**. 
5. Sous **Gérer**, sélectionnez **Paramètres utilisateur**.
6. Sous **Demandes de consentement d’administrateur (préversion)** , définissez **Les utilisateurs peuvent demander le consentement d’administrateur pour les applications qu’ils ne peuvent pas accepter** sur **Oui**.

   ![Configurer les paramètres de workflow de consentement administrateur](media/configure-admin-consent-workflow/admin-consent-requests-settings.png)
 
6. Configurez les paramètres suivants :

   * **Sélectionner les utilisateurs pour passer en revue les demandes de consentement d’administrateur**. Sélectionnez les réviseurs pour ce workflow parmi un ensemble d’utilisateurs qui possèdent les rôles d’administrateur général, d’administrateur d’applications cloud et d’administrateur d’applications.
   * **Les utilisateurs sélectionnés recevront des notifications par e-mail pour les demandes**. Activez ou désactivez les notifications par courrier électronique envoyées aux réviseurs lorsqu’une demande est effectuée.  
   * **Les utilisateurs sélectionnés recevront des rappels d’expiration de demande**. Activez ou désactivez les notifications de rappel par courrier électronique envoyées aux réviseurs lorsqu’une demande est sur le point d’expirer.  
   * **La demande de consentement expire après (jours)** . Spécifiez la durée pendant laquelle les demandes restent valides.

7. Sélectionnez **Enregistrer**. L’activation de la fonctionnalité peut prendre jusqu’à une heure.

> [!NOTE]
> Vous pouvez ajouter ou supprimer des réviseurs pour ce flux de travail en modifiant la liste **Select admin consent requests reviewers** (Sélectionner les réviseurs des demandes de consentement administrateur). Il est à noter que l’une des limites actuelles de cette fonctionnalité est que les réviseurs peuvent conserver la possibilité d’examiner les requêtes qui ont été présentées pendant qu’ils étaient désignés en tant que réviseurs.

## <a name="how-users-request-admin-consent"></a>Comment les utilisateurs demandent le consentement de l’administrateur

Une fois le workflow de consentement de l’administrateur activé, les utilisateurs peuvent demander l’approbation de l’administrateur pour une application à laquelle ils ne sont pas autorisés à donner leur consentement. Les étapes suivantes décrivent l’expérience de l’utilisateur lors de la demande d’approbation. 

1. L’utilisateur tente de se connecter à l’application.

2. Le message **Approbation requise** s’affiche. L’utilisateur indique pourquoi il a besoin d’accéder à l’application, puis sélectionne la **requête d’approbation**.

   ![Capture d’écran affichant une boîte de dialogue Approbation requise dans laquelle vous pouvez demander une approbation.](media/configure-admin-consent-workflow/end-user-justification.png)

3. Un message **Demande envoyée** confirme que la demande a été envoyée à l’administrateur. Si l’utilisateur envoie plusieurs requêtes, seule la première est envoyée à l’administrateur.

   ![Capture d’écran affichant la confirmation Demande envoyée.](media/configure-admin-consent-workflow/end-user-sent-request.png)

 4. L’utilisateur reçoit une notification par courrier électronique lorsque sa requête est approuvée, refusée ou bloquée. 

## <a name="review-and-take-action-on-admin-consent-requests"></a>Examiner et entreprendre des actions sur les demandes de consentement de l’administrateur

Pour passer en revue les demandes de consentement de l’administrateur et prendre les mesures suivantes :

1. Connectez-vous au [portail Azure](https://portal.azure.com) en tant que réviseur inscrit du workflow de consentement administrateur.
2. Sélectionnez **Tous les services** en haut du menu de navigation de gauche. Le panneau **Extension Azure Active Directory** apparaît.
3. Dans la zone de recherche de filtre, tapez « **Azure Active Directory** », puis sélectionnez l’élément **Azure Active Directory**.
4. Dans le menu de navigation, cliquez sur **Applications d’entreprise**.
5. Sous **Activité**, sélectionnez **Demandes de consentement d’administrateur (préversion)** .

   > [!NOTE]
   > Les réviseurs ne verront que les demandes d’administrateur qui ont été créées après qu’ils ont été désignés réviseurs.

1. Sélectionnez l’application demandée.
2. Passez en revue les détails de la requête :  

   * Pour voir qui demande l’accès et pourquoi, sélectionnez l’onglet **Demandé par**.
   * Pour voir les autorisations demandées par l’application, sélectionnez **Passer en revue les autorisations et le consentement**.

8. Évaluez la demande et prenez les mesures appropriées :

   * **Approuvez la demande**. Pour approuver une demande, accordez le consentement de l’administrateur à l’application. Une fois qu’une demande est approuvée, tous les demandeurs sont avertis qu’un accès leur a été accordé.  
   * **Refusez la demande**. Pour refuser une demande, vous devez fournir une justification qui sera fournie à tous les demandeurs. Une fois qu’une demande est refusée, tous les demandeurs sont avertis de l’accès à l’application. Le refus d’une demande n’empêchera pas les utilisateurs de demander le consentement de l’administrateur à l’application à l’avenir.  
   * **Bloquez la demande**. Pour bloquer une demande, vous devez fournir une justification qui sera fournie à tous les demandeurs. Une fois qu’une demande est bloquée, tous les demandeurs sont avertis de l’accès à l’application. Le blocage d’une demande crée un objet principal de service pour l’application dans votre client à l’état désactivé. À l’avenir, les utilisateurs ne seront pas en mesure de demander le consentement administrateur à l’application.
 
## <a name="email-notifications"></a>Notifications par e-mail
 
S’il est configuré, tous les réviseurs reçoivent des notifications par courrier électronique dans les cas suivants :

* Une demande a été créée
* La demande a expiré
* Une demande est proche de sa date d’expiration  
 
Les demandeurs recevront des notifications par courrier électronique dans les cas suivants :

* Ils envoient une nouvelle demande d’accès
* Leur demande a expiré
* Leur demande a été refusée ou bloquée
* Leur demande a été approuvée
 
## <a name="audit-logs"></a>Journaux d’audit 
 
Le tableau ci-dessous décrit les scénarios et les valeurs d’audit disponibles pour le flux de travail de consentement de l’administrateur. 

> [!NOTE]
> Le contexte utilisateur de l’intervenant d’audit est actuellement manquant dans tous les scénarios. Il s’agit d’une limitation connue dans la préversion.


|Scénario  |Audit Service  |Catégorie d’audit  |Activité d’audit  |Intervenant d’audit  |Limites du journal d’audit  |
|---------|---------|---------|---------|---------|---------|
|Activation par l’administrateur du workflow de demande de consentement        |Révisions d’accès           |UserManagement           |Créer un modèle de stratégie de gouvernance          |Contexte de l’application            |Actuellement, le contexte utilisateur est introuvable            |
|Administrateur désactivant le workflow de demande de consentement       |Révisions d’accès           |UserManagement           |Supprimer un modèle de stratégie de gouvernance          |Contexte de l’application            |Actuellement, le contexte utilisateur est introuvable           |
|Administrateur mettant à jour les configurations du workflow de consentement        |Révisions d’accès           |UserManagement           |Mettre à jour un modèle de stratégie de gouvernance          |Contexte de l’application            |Actuellement, le contexte utilisateur est introuvable           |
|Utilisateur final créant une demande de consentement administrateur pour une application       |Révisions d’accès           |Stratégie         |Créer une demande           |Contexte de l’application            |Actuellement, le contexte utilisateur est introuvable           |
|Réviseurs approuvant une demande de consentement administrateur       |Révisions d’accès           |UserManagement           |Approuver toutes les requêtes dans le workflow d’entreprise          |Contexte de l’application            |Pour l’heure, vous ne pouvez pas trouver le contexte de l’utilisateur ou l’ID de l’application qui a reçu le consentement de l’administrateur.           |
|Réviseurs refusant une demande de consentement administrateur       |Révisions d’accès           |UserManagement           |Approuver toutes les requêtes dans le workflow d’entreprise          |Contexte de l’application            | Pour l’heure, vous ne pouvez pas trouver le contexte utilisateur de l’acteur qui a refusé une demande de consentement administrateur.          |

## <a name="faq"></a>Questions fréquentes (FAQ) 

**J’ai activé ce flux de travail, mais lors du test de la fonctionnalité, pourquoi ne puis-je pas voir la nouvelle invite « Approbation requise » me permettant de demander l’accès ?**

Une fois la fonctionnalité activée, il peut s’écouler jusqu’à 60 minutes avant que la mise à jour ne soit visible pour les utilisateurs finaux. Vous pouvez vérifier que la configuration a pris effet correctement en affichant la valeur **EnableAdminConsentRequests** dans l’API `https://graph.microsoft.com/beta/settings`.

**En tant que réviseur, pourquoi ne puis-je pas voir toutes les requêtes en attente ?**

Les réviseurs ne peuvent voir que les demandes d’administrateur qui ont été créées après qu’ils ont été désignés réviseurs. Par conséquent, si vous avez récemment été ajouté en tant que réviseur, vous ne verrez pas les demandes qui ont été créées avant votre assignation.

**En tant que réviseur, pourquoi est-ce que je vois plusieurs demandes pour la même application ?**
  
Si un développeur d’applications a configuré son application pour utiliser le consentement statique et dynamique afin de demander l’accès aux données de l’utilisateur final, vous verrez deux demandes de consentement administrateur. Une demande représente les autorisations statiques, tandis que l’autre représente les autorisations dynamiques.

**En tant que demandeur, puis-je vérifier l’état de ma demande ?**  

Non, pour l’instant, les demandeurs peuvent uniquement obtenir des notifications par courrier électronique.

**En tant que réviseur, est-il possible d’approuver l’application, mais pas pour tout le monde ?**
 
Si vous souhaitez accorder le consentement de l’administrateur et permettre à tous les utilisateurs du locataire d’utiliser l’application, nous vous recommandons de refuser la demande. Octroyez ensuite manuellement le consentement de l’administrateur en restreignant l’accès à l’application en demandant l’attribution de l’utilisateur et en affectant des utilisateurs ou des groupes à l’application. Pour plus d'informations, voir [Méthodes d'affectation d'utilisateurs et de groupes](./assign-user-or-group-access-portal.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le consentement des applications, consultez [Infrastructure de consentement d’Azure Active Directory](../develop/consent-framework.md).

[Configurer le consentement de l’utilisateur final pour une application](configure-user-consent.md)

[Accorder le consentement de l’administrateur au niveau locataire à une application](grant-admin-consent.md)

[Autorisations et consentement dans la plateforme d’identités Microsoft](../develop/v2-permissions-and-consent.md)

[Azure AD sur StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)