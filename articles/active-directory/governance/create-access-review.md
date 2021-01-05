---
title: Créer une révision d’accès des groupes et applications - Azure AD
description: Découvrez comment créer une révision d’accès pour les membres de groupes ou pour l’accès aux applications dans les révisions d’accès Azure Active Directory.
services: active-directory
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 12/07/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: b12eb95a7840bdbb902701fc644eee30ffe9900f
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96778535"
---
# <a name="create-an-access-review-of-groups-and-applications-in-azure-ad-access-reviews"></a>Créer une révision d’accès des groupes et applications dans les révisions d’accès Azure AD

L’accès aux groupes et aux applications pour les employés et les invités change au fil du temps. Afin de réduire les risques associés aux affectations d’accès obsolètes, les administrateurs peuvent utiliser Azure Active Directory (Azure AD) pour créer des révision des accès pour les membres de groupes ou pour l’accès aux applications. Si vous devez régulièrement passer en revue les accès, vous pouvez aussi créer des révisions d’accès périodiques. Pour plus d’informations sur ces scénarios, consultez [Gérer l’accès des utilisateurs](manage-user-access-with-access-reviews.md) et [Gérer l’accès des invités](manage-guest-access-with-access-reviews.md).

Vous pouvez regarder une courte vidéo sur l’activation des révisions d’accès :

>[!VIDEO https://www.youtube.com/embed/X1SL2uubx9M]

Cet article explique comment créer une ou plusieurs révisions d’accès pour les membres de groupes ou pour l’accès aux applications.

## <a name="prerequisites"></a>Prérequis

- Azure AD Premium P2
- Administrateur général ou Administrateur d’utilisateurs

Pour plus d’informations, consultez [Exigences des licences](access-reviews-overview.md#license-requirements).

## <a name="create-one-or-more-access-reviews"></a>Créer une ou plusieurs révisions d’accès

1. Connectez-vous au portail Azure et ouvrez la page [Identity Governance](https://portal.azure.com/#blade/Microsoft_AAD_ERM/DashboardBlade/).

2. Dans le menu de gauche, cliquez sur **Révisions d’accès**.

3. Cliquez sur **Nouvelle révision d’accès** pour créer une révision d’accès.

    ![Volet des révisions d’accès dans Gouvernance des identités](./media/create-access-review/access-reviews.png)

4. À l’**Étape 1 : Sélectionner les éléments à réviser**, sélectionnez la ressource que vous souhaitez réviser.

    ![Créer une révision d’accès - Nom et description de la révision](./media/create-access-review/select-what-review.png)

5. Si vous avez sélectionné **Équipes + groupes** à l’étape 1, vous avez deux options à l’étape 2.
   - **Tous les groupes Microsoft 365 avec des utilisateurs invités.** Sélectionnez cette option si vous souhaitez créer des révisions périodiques sur tous vos utilisateurs invités dans l’ensemble des équipes Microsoft et des groupes M365 de votre organisation. Vous pouvez choisir d’exclure certains groupes en cliquant sur « Sélectionner le(s) groupe(s) à exclure ».
   - **Sélectionner les équipes + groupes.** Sélectionnez cette option si vous souhaitez spécifier un ensemble limité d’équipes et/ou de groupes à réviser. Après avoir cliqué sur cette option, vous verrez une liste de groupes à droite parmi lesquels vous pourrez choisir.

     ![Équipes et groupes](./media/create-access-review/teams-groups.png)

     ![Option Équipes et groupes choisie dans l’interface utilisateur](./media/create-access-review/teams-groups-detailed.png)

6. Si vous avez sélectionné **Applications** à l’étape 1, vous pouvez sélectionner une ou plusieurs applications à l’étape 2.

    >[!NOTE]
    > La sélection de plusieurs groupes et/ou applications entraîne la création de plusieurs révisions d’accès. Par exemple, si vous sélectionnez cinq groupes à réviser, cela donnera lieu à cinq révisions d’accès distinctes.

   ![Interface affichée si vous avez choisi des applications plutôt que des groupes](./media/create-access-review/select-application-detailed.png)

7. Ensuite, à l’étape 3, vous pouvez sélectionner l’étendue de la révision. Vos options sont les suivantes :
   - **Utilisateurs invités uniquement.** En sélectionnant cette option, vous limitez la révision d’accès aux seuls utilisateurs invités B2B d’Azure AD dans votre répertoire.
   - **Tout le monde.** La sélection de cette option permet d’étendre la révision d’accès à tous les objets utilisateur associés à la ressource.

    >[!NOTE]
    > Si vous avez sélectionné Tous les groupes Microsoft 365 avec des utilisateurs invités à l’étape 2, la seule option consiste à réviser les utilisateurs invités à l’étape 3.

8. Cliquez sur Suivant : Révisions
9. Dans la section **Sélectionner les réviseurs**, sélectionnez une ou plusieurs personnes pour effectuer les révisions d’accès. Vous pouvez choisir :
    - **Propriétaire(s) de groupe**. (Disponible uniquement lors de la révision d’une équipe ou d’un groupe).
    - **Utilisateur(s) ou groupe(s) sélectionné(s)** .
    - **Les utilisateurs révisent leur propre accès**.
    - **(Préversion) Managers des utilisateurs.**
    Si vous choisissez **Managers des utilisateurs** ou **Propriétaires de groupes**, vous avez également la possibilité de spécifier un réviseur de secours. Les réviseurs de secours sont invités à effectuer une révision lorsque l’utilisateur n’a pas de manager spécifié dans le répertoire ou que le groupe n’a pas de propriétaire.

    ![Nouvelle révision d’accès](./media/create-access-review/new-access-review.png)

10. Dans la section **Spécifier la périodicité de la révision**, vous pouvez spécifier une fréquence **hebdomadaire, mensuelle, trimestrielle, semestrielle ou annuelle**. Vous spécifiez ensuite une **durée**, qui définit la durée pendant laquelle une révision sera ouverte à la contribution des réviseurs. Par exemple, la durée maximale d’une révision mensuelle est de 27 jours, ce qui permet d’éviter le chevauchement des révisions. Vous pouvez raccourcir cette durée pour vous assurer que la contribution de vos réviseurs est prise en compte plus tôt. Ensuite, vous pouvez sélectionner une **date de début** et une **date de fin**.

    ![Choisir la fréquence de la révision](./media/create-access-review/frequency.png)

11. Cliquez sur le bouton **Next: Paramètres** en bas de la page.
12. Dans **Paramètres une fois l’opération terminée**, vous pouvez spécifier ce qui se produit une fois la révision terminée.

    ![Créer une révision d’accès : paramètres une fois l’opération terminée](./media/create-access-review/upon-completion-settings-new.png)

Si vous voulez supprimer automatiquement l’accès pour les utilisateurs qui ont été refusés, définissez Appliquer automatiquement les résultats à la ressource sur Activer. Si vous voulez appliquer manuellement les résultats quand la révision est terminée, cliquez sur Désactiver.
Utilisez la liste Si les réviseurs ne répondent pas pour spécifier ce qui se passe pour les utilisateurs qui ne sont pas vérifiés par le réviseur au cours de la révision. Ce paramètre n’affecte pas les utilisateurs qui ont été vérifiés manuellement par les réviseurs. Si la décision finale du réviseur est Refuser, l’accès de l’utilisateur est supprimé.

- **Aucune modification** : laisser l’accès de l’utilisateur inchangé
- **Supprimer l’accès** : supprimer l’accès de l’utilisateur
- **Approuver l’accès** : approuver l’accès de l’utilisateur
- **Accepter les recommandations** : accepter la recommandation du système sur le refus ou l’approbation de la prolongation de l’accès de l’utilisateur

    ![Options de paramètres une fois l’opération terminée](./media/create-access-review/upon-completion-settings-new.png)

Utilisez l’action à appliquer aux utilisateurs **invités** refusés pour spécifier ce qui arrive aux utilisateurs invités s’ils sont refusés.
- Supprimer l’appartenance de l’utilisateur à la ressource supprimera l’accès de l’utilisateur refusé au groupe ou à l’application en cours de révision ; il sera toujours en mesure de se connecter au locataire.
- Empêcher l’utilisateur de se connecter pendant 30 jours, puis supprimer l’utilisateur du locataire empêchera les utilisateurs non autorisés de se connecter au locataire, même s’ils ont accès à d’autres ressources. En cas d’erreur ou si un administrateur décide de réactiver l’accès, cela est possible dans les 30 jours suivant la désactivation de l’utilisateur. Si aucune action n’est effectuée sur les utilisateurs désactivés, ils seront supprimés du locataire.

Pour en savoir plus sur les meilleures pratiques pour la suppression des utilisateurs invités qui n’ont plus accès aux ressources de votre organisation, lisez l’article intitulé [Utiliser Azure AD Identity Governance pour examiner et supprimer les utilisateurs externes qui n’ont plus accès aux ressources](access-reviews-external-users.md).

   >[!NOTE]
   >L’action à appliquer sur les utilisateurs invités refusés n’est pas configurable sur les révisions dont la portée est plus large que celle des utilisateurs invités. Elle n’est pas non plus configurable pour les révisions de **Tous les groupes M365 avec des utilisateurs invités.** Lorsqu’elle n’est pas configurable, l’option par défaut de suppression de l’appartenance de l’utilisateur à la ressource est utilisée sur les utilisateurs refusés.

13. Dans **Activer l’assistance aux décisions de révision**, indiquez si vous souhaitez que votre réviseur reçoive des recommandations au cours du processus de révision.

    ![Activer les options d’assistance à la décision](./media/create-access-review/helpers.png)

14. Dans la section **Paramètres avancés**, vous pouvez choisir les options suivantes :
    - Définissez **Justification obligatoire** sur **Activer** afin d’exiger que le réviseur indique un motif d’approbation.
    - Définissez **Notifications par e-mail** sur **Activer** pour qu’Azure AD envoie des notifications par e-mail aux réviseurs quand une révision d’accès commence et aux administrateurs quand une révision est terminée.
    - Définissez **Rappels** sur **Activer** pour qu’Azure AD envoie des rappels concernant les révisions d’accès en cours aux réviseurs qui n’ont pas terminé leur révision. Ces rappels sont automatiquement envoyés à mi-parcours de la révision.
    - Le contenu de l’e-mail envoyé aux réviseurs est généré automatiquement en fonction des détails de la révision, comme le nom de la révision, le nom de la ressource, la date d’échéance, etc. Si vous souhaitez communiquer des informations supplémentaires telles que des instructions ou des coordonnées, vous pouvez les ajouter dans la section **Contenu supplémentaire pour l’e-mail du réviseur**. Les informations que vous entrez sont incluses dans les e-mails d’invitation et de rappel envoyés aux réviseurs désignés. La section mise en surbrillance dans l’image ci-dessous indique où ces informations sont affichées.


      ![Contenu supplémentaire pour le réviseur](./media/create-access-review/additional-content-reviewer.png)

15. Cliquez sur **Suivant : Réviser + créer** pour passer à la page suivante.
16. Nommez la révision d’accès. Si vous le souhaitez, vous pouvez fournir une description de cette révision. Les réviseurs ont accès au nom et à la description de la révision.
17. Vérifiez les informations, puis sélectionnez **Créer**.

       ![Écran de création d’une révision](./media/create-access-review/create-review.png)

## <a name="start-the-access-review"></a>Démarrer la révision d’accès

Une fois que vous avez spécifié les paramètres pour une révision d’accès, cliquez sur **Démarrer**. La révision d’accès s’affiche dans votre liste, avec un indicateur de son état.

![Liste des révisions d’accès et de leur état](./media/create-access-review/access-reviews-list.png)

Par défaut, Azure AD envoie un e-mail aux réviseurs peu de temps après le démarrage de la révision. Si vous ne souhaitez pas qu’Azure AD envoie cet e-mail, veillez à informer les réviseurs qu’une révision d’accès leur a été assignée. Vous pouvez leur montrer les instructions relatives à la [révision d’accès aux groupes ou aux applications](perform-access-review.md). Si votre révision s’adresse à des invités qui doivent réviser leur propre accès, donnez-leur des instructions sur la méthode à suivre pour [réviser leur accès à des groupes ou à des applications](review-your-access.md).

Si vous avez attribué le rôle de réviseur à des clients invités, mais que ceux-ci ont refusé l’invitation, ils ne recevront pas d’e-mails des révisions d’accès, car il est nécessaire d’avoir accepté l’invitation pour pouvoir commencer la révision.

## <a name="access-review-status-table"></a>Tableau de l’état de la révision d’accès

| Statut | Définition |
|--------|------------|
|NotStarted | La révision a été créée, la détection des utilisateurs est en attente de démarrage. |
|Initialisation   | La détection des utilisateurs est en cours pour identifier tous les utilisateurs faisant partie de la révision. |
|Démarrage en cours | La révision démarre. Si les notifications par e-mail sont activées, des e-mails sont envoyés aux réviseurs. |
|InProgress | La révision a démarré. Si les notifications par e-mail sont activées, des e-mails ont été envoyés aux réviseurs. Les réviseurs peuvent soumettre des décisions jusqu’à la date d’échéance. |
|Completing | La révision est terminée et les e-mails sont envoyés au propriétaire de la révision. |
|Révision automatique | La révision se fait dans le cadre d’une phase de vérification du système. Le système enregistre des décisions pour les utilisateurs qui n’ont pas été révisés en fonction de recommandations ou de décisions préconfigurées. |
|Révisé automatiquement | Les décisions ont été enregistrées par le système pour tous les utilisateurs qui n’ont pas été révisés. La révision est prête à passer à l’étape **Applying** si l’application automatique est activée. |
|Applying | L’accès n’est pas modifié pour les utilisateurs qui ont été approuvés. |
|Applied | Les utilisateurs refusés, le cas échéant, ont été supprimés de la ressource ou du répertoire. |
|Échec | La révision n'a pas pu se poursuivre. Cette erreur peut être liée à la suppression du locataire, à une modification des licences ou à d'autres changements internes au niveau du locataire. |

## <a name="create-reviews-via-apis"></a>Créer des révisions via des API

Vous pouvez également créer des révisions d’accès avec des API. Ce que vous faites pour gérer les révisions d’accès des groupes et des utilisateurs d’applications dans le portail Azure peut également être effectué avec les API Microsoft Graph. Pour en savoir plus, voir [Informations de référence sur les API des révisions d’accès d’Azure AD](/graph/api/resources/accessreviews-root?view=graph-rest-beta). Pour obtenir un exemple de code, voir [Exemple de récupération de révisions d’accès Azure AD via Microsoft Graph](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-of-retrieving-Azure-AD-access-reviews-via-Microsoft/m-p/236096).

## <a name="next-steps"></a>Étapes suivantes

- [Réviser l’accès à des groupes ou à des applications](perform-access-review.md)
- [Réviser son accès à des groupes ou à des applications](review-your-access.md)
- [Effectuer une révision d’accès de groupes ou révisions d’accès des applications dans Azure AD](complete-access-review.md)