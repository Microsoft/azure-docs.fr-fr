---
title: Fichier include
description: Fichier include
services: active-directory
author: msaburnley
ms.service: active-directory
ms.topic: include
ms.date: 03/30/2020
ms.author: ajburnle
ms.custom: include file
ms.openlocfilehash: b7029c5cdd004910caa83425536bbe481e4b01f3
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95556885"
---
## <a name="for-users-in-your-directory"></a>Pour les utilisateurs dans votre répertoire

Suivez ces étapes si vous souhaitez autoriser les utilisateurs de votre annuaire à demander ce package d’accès. Lors de la définition de la stratégie de demandes, vous pouvez spécifier des utilisateurs individuels ou plus souvent des groupes d’utilisateurs. Par exemple, votre organisation a peut-être déjà un groupe tel que **Tous les employés**.  Si ce groupe est ajouté dans la stratégie pour des utilisateurs qui peuvent demander l’accès, tout membre de ce groupe peut ensuite demander l’accès.

1. Dans la section **Utilisateurs pouvant demander l’accès**, cliquez sur **Pour les utilisateurs dans votre annuaire**.

    Quand vous sélectionnez cette option, de nouvelles options s’affichent pour vous permettre de préciser qui dans votre annuaire peut demander ce package d’accès.

    ![Package d’accès - Demandes - Pour les utilisateurs de votre annuaire](./media/active-directory-entitlement-management-request-policy/for-users-in-your-directory.png)

1. Sélectionnez l’une des options suivantes :

    |  |  |
    | --- | --- |
    | **Utilisateurs et groupes spécifiques** | Choisissez cette option si vous souhaitez que seuls les utilisateurs et les groupes de votre annuaire que vous spécifiez puissent demander ce package d’accès. |
    | **Tous les membres (à l’exclusion des invités)** | Choisissez cette option si vous souhaitez que tous les utilisateurs membres de votre annuaire puissent demander ce package d’accès. Cette option n’inclut pas les utilisateurs que vous avez invités dans votre annuaire. |
    | **Tous les utilisateurs (y compris les invités)** | Choisissez cette option si vous souhaitez que tous les utilisateurs membres et les utilisateurs invités de votre annuaire puissent demander ce package d’accès. |

    Les utilisateurs invités font référence à des utilisateurs externes qui ont été invités dans votre annuaire avec [Azure AD B2B](../articles/active-directory/external-identities/what-is-b2b.md). Pour plus d’informations sur les différences entre les utilisateurs membres et les utilisateurs invités, consultez [Quelles sont les autorisations utilisateur par défaut dans Azure Active Directory ?](../articles/active-directory/fundamentals/users-default-permissions.md).

1. Si vous avez sélectionné **Utilisateurs et groupes spécifiques**, cliquez sur **Ajouter des utilisateurs et des groupes**.

1. Dans le volet Sélectionnez des utilisateurs et des groupes, sélectionnez les utilisateurs et les groupes à ajouter.

    ![Package d’accès - Demandes - Sélectionner des utilisateurs et des groupes](./media/active-directory-entitlement-management-request-policy/select-users-groups.png)

1. Cliquez sur **Sélectionner** pour ajouter les utilisateurs et les groupes.

1. Passez à la section [Approbation](#approval) plus bas.

## <a name="for-users-not-in-your-directory"></a>Pour les utilisateurs qui ne sont pas dans votre répertoire

 **Les utilisateurs absents de votre annuaire** sont des utilisateurs qui se trouvent dans un autre annuaire ou domaine Azure AD. Ces utilisateurs n’ont peut-être pas encore été invités dans votre annuaire. Les annuaires Azure AD doivent être configurés pour autoriser les invitations dans **Restrictions de collaboration**. Pour plus d’informations, consultez [Permettre une collaboration B2B externe et gérer les utilisateurs autorisés à en inviter d’autres](../articles/active-directory/external-identities/delegate-invitations.md).

> [!NOTE]
> Un compte d’utilisateur invité sera créé pour un utilisateur qui n’est pas encore dans votre répertoire et dont la demande est approuvée ou approuvée automatiquement. L’invité est invité, mais ne recevra pas d’invitation par e-mail. Il recevra un e-mail une fois l’attribution au package d’accès fournie. Par défaut, quand cet utilisateur invité n’a plus aucune attribution de package d’accès (pour cause d’expiration ou d’annulation), son compte ne peut plus se connecter et est supprimé par la suite. Si vous voulez que des utilisateurs invités demeurent indéfiniment dans votre répertoire, même s’ils n’ont plus d’attribution de package d’accès, vous pouvez modifier les paramètres de votre configuration de gestion des droits d’utilisation. Pour plus d’informations sur l’objet utilisateur invité, consultez [Propriétés d’un utilisateur Azure Active Directory B2B Collaboration](../articles/active-directory/external-identities/user-properties.md).

Suivez ces étapes si vous souhaitez autoriser les utilisateurs absents de votre annuaire à demander ce package d’accès :

1. Dans la section **Utilisateurs pouvant demander l’accès**, cliquez sur **Pour les utilisateurs dans votre annuaire**.

    Quand vous sélectionnez cette option, de nouvelles options s’affichent.

    ![Package d’accès - Demandes - Pour les utilisateurs qui ne sont pas dans votre annuaire](./media/active-directory-entitlement-management-request-policy/for-users-not-in-your-directory.png)

1. Sélectionnez l’une des options suivantes :

    |  |  |
    | --- | --- |
    | **Organisations connectées spécifiques** | Choisissez cette option si vous souhaitez sélectionner parmi une liste d’organisations que votre administrateur a ajoutées précédemment. Tous les utilisateurs des organisations sélectionnées peuvent demander ce package d’accès. |
    | **Toutes les organisations connectées** | Choisissez cette option si tous les utilisateurs de toutes les organisations connectées peuvent demander ce package d’accès. |
    | **Tous les utilisateurs (toutes les organisations connectées + tous les nouveaux utilisateurs externes)** | Choisissez cette option si tous les utilisateurs de toutes vos organisations connectées peuvent demander ce package d’accès et que les paramètres de la liste verte ou d’exclusion B2B doivent être prioritaires pour tout nouvel utilisateur externe. |

    Une organisation connectée est un domaine ou un annuaire Azure AD externe avec lequel vous collaborez.

1. Si vous avez sélectionné **Organisations connectées spécifiques**, cliquez sur **Ajouter des annuaires** pour effectuer une sélection parmi une liste d’organisations connectées que votre administrateur a ajoutées précédemment.

1. Tapez le nom ou le nom de domaine pour rechercher une organisation précédemment connectée.

    ![Package d’accès - Demandes - Sélectionner des annuaires](./media/active-directory-entitlement-management-request-policy/select-directories.png)

    Si l’organisation avec laquelle vous souhaitez collaborer ne figure pas dans la liste, vous pouvez demander à votre administrateur de l’ajouter en tant qu’organisation connectée. Pour plus d’informations, voir [Ajouter une organisation connectée](../articles/active-directory/governance/entitlement-management-organization.md).

1. Une fois que vous avez sélectionné toutes les organisations connectées, cliquez sur **Sélectionner**.

    > [!NOTE]
    > Tous les utilisateurs des organisations connectées sélectionnées pourront demander ce package d’accès. Cela comprend les utilisateurs Azure AD de tous les sous-domaines associés à l’organisation, à moins que ces domaines soient bloqués par la liste d’autorisation ou de refus d’Azure B2B. Pour plus d’informations, consultez [Autoriser ou bloquer des invitations aux utilisateurs B2B à partir d’organisations spécifiques](../articles/active-directory/external-identities/allow-deny-list.md).

1. Passez à la section [Approbation](#approval) plus bas.

## <a name="none-administrator-direct-assignments-only"></a>Aucune (attributions direct administrateur uniquement)

Effectuez ces étapes si vous souhaitez contourner les demandes d’accès et autoriser les administrateurs à attribuer directement des utilisateurs spécifiques à ce package d’accès. Les utilisateurs n’auront pas à demander le package d’accès. Vous pouvez toujours définir des paramètres de cycle de vie, mais il n’y a aucun paramètre de demande.

1. Dans la section **Utilisateurs pouvant demander l’accès**, cliquez sur **Aucune (attributions direct administrateur uniquement)** .

    ![Package d’accès - Demandes - Aucun (attributions directes d’administrateur uniquement)](./media/active-directory-entitlement-management-request-policy/none-admin-direct-assignments-only.png)

    Après avoir créé le package d’accès, vous pouvez directement attribuer des utilisateurs internes et externes spécifiques à ce package d’accès. Si vous spécifiez un utilisateur externe, un compte d’utilisateur invité est créé dans votre répertoire. Pour plus d’informations sur l’attribution directe d’un utilisateur, voir [Afficher, ajouter et supprimer les attributions pour un package d’accès](../articles/active-directory/governance/entitlement-management-access-package-assignments.md).

1. Passez à la section [Activer les demandes](#enable-requests) plus bas.

## <a name="approval"></a>Approbation

Dans la section Approbation, vous spécifiez si une approbation est nécessaire quand les utilisateurs demandent ce package d’accès. Les paramètres d’approbation fonctionnent de la façon suivante :

- La demande n’a besoin d’être approuvée que par un seul des approbateurs sélectionnés ou approbateurs de secours pour une approbation en une étape. 
- Seul l’un des approbateurs sélectionnés de chaque étape doit approuver une demande d’approbation en 2 étapes.
- L’approbateur peut être un gestionnaire, un sponsor interne ou un sponsor externe, en fonction de la personne dont la stratégie régit l’accès.
- L’approbation de chaque approbateur sélectionné n’est pas nécessaire pour une approbation en une ou 2 étapes.
- La décision d’approbation repose sur le premier approbateur à consulter la demande.

Pour une démonstration de l’ajout d’approbateurs à une stratégie de demande, regardez la vidéo suivante :

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4cZfg]

Pour obtenir une démonstration de l’ajout d’une approbation en plusieurs étapes à une stratégie de demande, regardez la vidéo suivante :

>[!VIDEO https://www.microsoft.com/videoplayer/embed/RE4d1Jw]

Pour spécifier les paramètres d’approbation pour les demandes de package d’accès, procédez comme suit :

1. Pour exiger une approbation pour des demandes des utilisateurs sélectionnés, définissez **Exiger une approbation** sur **Oui**. Ou bien, pour que les demandes soient approuvées automatiquement, définissez cette valeur sur **Non**.

1. Pour exiger des utilisateurs une justification pour demander le package d’accès, affectez la valeur **Oui** à **Exiger la justification du demandeur**.
    
1. À présent, déterminez si les demandes nécessitent une approbation en une ou 2 étapes. Définissez **Nombre d’étapes** sur **1** pour l’approbation en une étape, ou sur **2** pour l’approbation en 2 étapes.

    ![Package d’accès - Demandes - Paramètres d’approbation](./media/active-directory-entitlement-management-request-policy/approval.png)

Suivez les étapes ci-dessous pour ajouter des approbateurs après avoir sélectionné le nombre d’étapes dont vous avez besoin : 

### <a name="single-stage-approval"></a>Approbation en une seule étape

1. Ajoutez le **Premier approbateur** :
    
    Si la stratégie est définie pour régir l’accès des utilisateurs dans votre annuaire, vous pouvez sélectionner **Gestionnaire comme approbateur**. Vous pouvez également ajouter un utilisateur spécifique en cliquant sur **Ajouter des approbateurs** après avoir sélectionné Choisir des approbateurs spécifiques dans le menu déroulant.
    
    ![Package d’accès – Demandes – Pour les utilisateurs qui sont dans votre annuaire – Premier approbateur](./media/active-directory-entitlement-management-request-policy/approval-single-stage-first-approver-manager.png)

    Si cette stratégie est définie pour régir l’accès des utilisateurs qui ne sont pas dans votre annuaire, vous pouvez sélectionner **Sponsor externe** ou **Sponsor interne**. Vous pouvez également ajouter un utilisateur spécifique en cliquant **Ajouter des approbateurs** ou des groupes sous Choisir des approbateurs spécifiques.
    
    ![Package d’accès – Demandes – Pour les utilisateurs qui ne sont pas dans votre annuaire – Premier approbateur](./media/active-directory-entitlement-management-request-policy/out-directory-first-approver.png)
    
1. Si vous avez sélectionné **Gestionnaire** en tant qu’approbateur, cliquez sur **Ajouter un secours** pour sélectionner un ou plusieurs utilisateurs ou groupes dans votre annuaire en tant qu’approbateurs de secours. Les approbateurs de secours reçoivent la demande si la gestion des droits d’utilisation ne peut pas trouver le gestionnaire de l’utilisateur demandant l’accès.

    La gestion des droits d’utilisation trouve le gestionnaire à l’aide de l’attribut **Manager**. L’attribut se trouve dans le profil de l’utilisateur dans Azure AD. Pour plus d’informations, consultez [Ajouter ou mettre à jour les informations du profil utilisateur avec Azure Active Directory](../articles/active-directory/fundamentals/active-directory-users-profile-azure-portal.md).

1. Si vous avez sélectionné **Choisir des approbateurs spécifiques**, cliquez sur **Ajouter des approbateurs** pour sélectionner dans votre annuaire un ou plusieurs utilisateurs ou groupes devant assumer la fonction d’approbateurs.

1. Dans la zone sous **La décision doit être prise dans combien de jours ?** , spécifiez le nombre de jours dont dispose un approbateur pour examiner une demande pour ce package d’accès.

    Si une demande n’est pas approuvée dans le délai imparti, elle est refusée automatiquement. L’utilisateur devra envoyer une autre demande pour le package d’accès.

1. Pour exiger des approbateurs une justification de leur décision, définissez Exiger la justification de l’approbateur sur **Oui**.

    La justification est visible par les autres approbateurs et le demandeur.

### <a name="2-stage-approval"></a>Approbation en deux phases

Si vous avez sélectionné l’approbation en 2 étapes, vous devez ajouter un deuxième approbateur.

1. Ajoutez le **Deuxième approbateur** : 
    
    Si les utilisateurs se trouvent dans votre annuaire, ajoutez un utilisateur spécifique en tant que deuxième approbateur en cliquant **Ajouter des approbateurs** sous Choisir des approbateurs spécifiques.

    ![Package d’accès – Demandes – Pour les utilisateurs qui sont dans votre annuaire – Deuxième approbateur](./media/active-directory-entitlement-management-request-policy/in-directory-second-approver.png)

    Si les utilisateurs ne se sont pas dans votre annuaire, sélectionnez **Sponsor interne** ou **Sponsor externe** comme deuxième approbateur. Après avoir sélectionné l’approbateur, ajoutez les approbateurs de secours.

    ![Package d’accès – Demandes – Pour les utilisateurs qui ne sont pas dans votre annuaire – Deuxième approbateur](./media/active-directory-entitlement-management-request-policy/out-directory-second-approver.png) 

1. Spécifiez le nombre de jours pendant lesquels le deuxième approbateur doit approuver la demande dans la zone sous **La décision doit être prise dans combien de jours ?** . 

1. Définissez le commutateur Exiger la justification de l’approbateur sur **Oui** ou **Non**.

### <a name="alternate-approvers"></a>Approbateurs de substitution

Vous pouvez spécifier des approbateurs substitution, comme vous avez spécifié les premier et deuxième approbateurs qui peuvent approuver les demandes. Le fait de disposer d’approbateurs de substitution garantit que les demandes sont approuvées ou refusées avant leur expiration (délai d’expiration). Vous pouvez répertorier des approbateurs de substitution, le premier approbateur et le deuxième approbateur pour l’approbation en 2 étapes. 

En spécifiant des approbateurs de substitution, si les premier et deuxième approbateurs n’ont pas été en mesure d’approuver ou de refuser la demande, la demande en attente est transférée aux approbateurs de substitution, conformément à la planification de transfert que vous avez spécifiée lors de la configuration de la stratégie. Ils reçoivent un e-mail pour approuver ou refuser la demande en attente.

Une fois la demande transférée aux approbateurs de substitution, les premier et deuxième approbateurs peuvent toujours l’approuver ou la refuser. Pour approuver ou refuser la demande en attente, les approbateurs de substitution utilisent le même site Mon Accès.

Nous pouvons répertorier les personnes ou groupes de personnes qui doivent être approbateurs et approbateurs de substitution. Veillez à répertorier des groupes de personnes différents pour les premier et deuxième approbateurs ainsi que les approbateurs de substitution.
Par exemple, si vous avez mentionné Alice et Bob comme premier et deuxième approbateurs, mentionnez Carol et Dave comme approbateurs de substitution. Pour ajouter des approbateurs de substitution à un package d’accès, procédez comme suit :

1. Sous Premier approbateur, Deuxième approbateur, ou les deux, cliquez sur **Afficher les paramètres de demande avancés**.

    ![Package d’accès – Stratégie – Afficher les paramètres de demande avancés](./media/active-directory-entitlement-management-request-policy/alternate-approvers-click-advanced-request.png)

1. Définissez l’option **En l’absence d’action, transférer à d’autres approbateurs ?** sur **Oui**.

1. Cliquez sur **Ajouter des approbateurs de substitution**, puis sélectionnez les approbateurs de substitution dans la liste.

    ![Package d’accès – Stratégie – Ajouter des approbateurs de substitution](./media/active-directory-entitlement-management-request-policy/alternate-approvers-add.png)

1. Dans la zone **Passer aux autres approbateurs après combien de jours ?** , entrez le nombre de jours dont les approbateurs disposent pour approuver ou refuser une demande. Si aucun approbateur n’a approuvé ou refusé la demande pendant la durée de validité de celle-ci, la demande expire (délai d’expiration) et l’utilisateur doit soumettre une autre demande pour le package d’accès. 

    Les demandes ne peuvent être transférées à d’autres approbateurs qu’une journée après que la demande a atteint la moitié de sa durée de vie, et le délai d’expiration de la décision de l’approbateur principal doit être d’au moins quatre jours. Si le délai d’expiration de la demande est inférieur ou égal à trois jours, il n’y a pas assez de temps pour transférer la demande à d’autres approbateurs. Dans cet exemple, la durée de la demande est de 14 jours. Ainsi, la durée de la demande atteint sa demi-vie le jour 7. La demande ne peut donc pas être transférée avant le jour 8. Par ailleurs, les demandes ne peuvent pas être transférées le dernier jour de leur durée. Ainsi, dans l’exemple, l’ultime limite pour le transfert de la demande est le jour 13.

## <a name="enable-requests"></a>Activer les demandes

1. Si vous souhaitez que le package d’accès soit immédiatement disponible afin d’être demandé par les utilisateurs dans la stratégie de demandes, déplacez le bouton bascule Activer sur **Oui**.

    Vous pouvez toujours l’activer plus tard, après avoir créé le package d’accès.

    Si vous avez sélectionné **Aucun (attributions directes d’administrateur uniquement)** et que vous affectez la valeur **No** à Activer, les administrateurs ne pourront pas attribuer directement ce package d’accès.

    ![Capture d’écran montrant l’option permettant d’activer de nouvelles demandes et affectations.](./media/active-directory-entitlement-management-request-policy/enable-requests.png)

1. Cliquez sur **Suivant**.

## <a name="add-requestor-information-preview-to-an-access-package"></a>Ajout d’informations sur le demandeur (préversion) à un package d'accès

1. Accédez à l’onglet **Informations sur le demandeur**, puis cliquez sur le sous-onglet **Questions**.
 
1. Tapez ce que vous souhaitez demander au demandeur, également appelé « chaîne d’affichage », pour la question dans la zone **Question**.

    ![Package d’accès – Stratégie – Activation du paramètre Informations sur le demandeur](./media/active-directory-entitlement-management-request-policy/add-requestor-info-question.png)

1. Si vous souhaitez ajouter vos propres options de localisation, cliquez sur **Ajouter une localisation**.
    1. Une fois dans le volet **Ajouter des localisations pour les questions**, sélectionnez le **code de la langue** dans laquelle vous localisez la question.
    1. Dans la langue que vous avez configurée, tapez la question dans la zone **Texte localisé**.
    1. Une fois que vous avez ajouté toutes les localisations nécessaires, cliquez sur **Enregistrer**.

    ![Package d’accès – Stratégie – Configuration du texte localisé](./media/active-directory-entitlement-management-request-policy/add-localization-question.png)

1. Sélectionnez le **Format de la réponse** dans lequel vous souhaitez que les demandeurs répondent : *texte court*, *choix multiples* et *texte long*.
 
    ![Package d’accès – Stratégie – Sélection du bouton Afficher et modifier le format de réponse Choix multiples](./media/active-directory-entitlement-management-request-policy/answer-format-view-edit.png)
 
1. Si vous sélectionnez Choix multiples, cliquez sur le bouton **Afficher et modifier** pour configurer les options de réponse.
    1. Une fois que vous avez sélectionné Afficher et modifier, le volet **Afficher/modifier la question** s’ouvre.
    1. Tapez les options de réponse que vous souhaitez donner au demandeur lorsqu’il répond à la question dans les zones **Valeurs de réponse** .
    1. Tapez le nombre de réponses dont vous avez besoin, puis cliquez sur **Enregistrer**.
    
    ![Package d’accès – Stratégie –Saisie des options de choix multiples](./media/active-directory-entitlement-management-request-policy/answer-multiple-choice.png)
  
1. Pour exiger que les demandeurs répondent à cette question lorsqu’ils demandent l’accès à un package d’accès, cochez la case sous **Obligatoire**.

1. Cliquez sur Suivant.