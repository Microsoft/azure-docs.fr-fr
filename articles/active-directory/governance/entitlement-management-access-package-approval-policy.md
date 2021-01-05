---
title: Modifier les paramètres d'approbation d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure AD - Azure Active Directory
description: Apprenez à modifier les paramètres d'approbation et d'informations sur le demandeur d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 09/16/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2e36b2d4576b43544bec89efd326363344b35be9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90992714"
---
# <a name="change-approval-and-requestor-information-preview-settings-for-an-access-package-in-azure-ad-entitlement-management"></a>Modifier les paramètres d'approbation et d'informations sur le demandeur (préversion) d'un package d'accès dans la fonctionnalité de gestion des droits d'utilisation Azure AD

En tant que gestionnaire de packages d'accès, vous pouvez à tout moment modifier les paramètres d'approbation et d'informations sur le demandeur d'un package d'accès en modifiant une stratégie existante ou en ajoutant une nouvelle stratégie.

Cet article explique comment modifier les paramètres d'approbation et d'informations sur le demandeur d'un package d'accès existant.

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


## <a name="change-approval-settings-of-an-existing-access-package"></a>Modifier les paramètres d'approbation d'un package d'accès existant

Pour spécifier les paramètres d’approbation pour les demandes de package d’accès, procédez comme suit :

**Rôle prérequis :** Administrateur général, administrateur d’utilisateurs ou gestionnaire de package d’accès

1. Dans le portail Azure, cliquez sur **Azure Active Directory**, puis sur **Identity Governance**.

1. Dans le menu de gauche, cliquez sur **Packages d’accès**, puis ouvrez le package d'accès.

1. Sélectionnez une stratégie à modifier ou ajoutez une nouvelle stratégie au package d'accès.
    1. Cliquez sur **Stratégies**, puis sur **Ajouter une stratégie** si vous souhaitez créer une nouvelle stratégie.
    1. Cliquez sur la stratégie que vous souhaitez modifier, puis sur **Modifier**.

1. Accédez à l'onglet **Requête**.

1. Pour exiger une approbation pour des demandes des utilisateurs sélectionnés, définissez **Exiger une approbation** sur **Oui**. Ou bien, pour que les demandes soient approuvées automatiquement, définissez cette valeur sur **Non**.

1. Pour exiger des utilisateurs une justification pour demander le package d’accès, affectez la valeur **Oui** à **Exiger la justification du demandeur**.
    
1. À présent, déterminez si les demandes nécessitent une approbation en une ou 2 étapes. Définissez **Nombre d’étapes** sur **1** pour l’approbation en une étape, ou sur **2** pour l’approbation en 2 étapes.

    ![Package d’accès - Demandes - Paramètres d’approbation](./media/entitlement-management-access-package-approval-policy/approval.png)


Suivez les étapes ci-dessous pour ajouter des approbateurs après avoir sélectionné le nombre d’étapes dont vous avez besoin : 

### <a name="single-stage-approval"></a>Approbation en une seule étape

1. Ajoutez le **Premier approbateur** :
    
    Si la stratégie est définie pour régir l’accès des utilisateurs dans votre annuaire, vous pouvez sélectionner **Gestionnaire comme approbateur**. Vous pouvez également ajouter un utilisateur spécifique en cliquant sur **Ajouter des approbateurs** après avoir sélectionné Choisir des approbateurs spécifiques dans le menu déroulant.
    
    ![Package d’accès – Demandes – Pour les utilisateurs qui sont dans votre annuaire – Premier approbateur](./media/entitlement-management-access-package-approval-policy/approval-single-stage-first-approver-manager.png)

    Si cette stratégie est définie pour régir l’accès des utilisateurs qui ne sont pas dans votre annuaire, vous pouvez sélectionner **Sponsor externe** ou **Sponsor interne**. Vous pouvez également ajouter un utilisateur spécifique en cliquant **Ajouter des approbateurs** ou des groupes sous Choisir des approbateurs spécifiques.
    
    ![Package d’accès – Demandes – Pour les utilisateurs qui ne sont pas dans votre annuaire – Premier approbateur](./media/entitlement-management-access-package-approval-policy/out-directory-first-approver.png)
    
1. Si vous avez sélectionné **Gestionnaire** en tant qu’approbateur, cliquez sur **Ajouter un secours** pour sélectionner un ou plusieurs utilisateurs ou groupes dans votre annuaire en tant qu’approbateurs de secours. Les approbateurs de secours reçoivent la demande si la gestion des droits d’utilisation ne peut pas trouver le gestionnaire de l’utilisateur demandant l’accès.

    La gestion des droits d’utilisation trouve le gestionnaire à l’aide de l’attribut **Manager**. L’attribut se trouve dans le profil de l’utilisateur dans Azure AD. Pour plus d’informations, consultez [Ajouter ou mettre à jour les informations du profil utilisateur avec Azure Active Directory](../fundamentals/active-directory-users-profile-azure-portal.md).

1. Si vous avez sélectionné **Choisir des approbateurs spécifiques**, cliquez sur **Ajouter des approbateurs** pour sélectionner dans votre annuaire un ou plusieurs utilisateurs ou groupes devant assumer la fonction d’approbateurs.

1. Dans la zone sous **La décision doit être prise dans combien de jours ?** , spécifiez le nombre de jours dont dispose un approbateur pour examiner une demande pour ce package d’accès.

    Si une demande n’est pas approuvée dans le délai imparti, elle est refusée automatiquement. L’utilisateur devra envoyer une autre demande pour le package d’accès.

1. Pour exiger des approbateurs une justification de leur décision, définissez Exiger la justification de l’approbateur sur **Oui**.

    La justification est visible par les autres approbateurs et le demandeur.

### <a name="2-stage-approval"></a>Approbation en deux phases

Si vous avez sélectionné l’approbation en 2 étapes, vous devez ajouter un deuxième approbateur.

1. Ajoutez le **Deuxième approbateur** : 
    
    Si les utilisateurs se trouvent dans votre annuaire, ajoutez un utilisateur spécifique en tant que deuxième approbateur en cliquant **Ajouter des approbateurs** sous Choisir des approbateurs spécifiques.

    ![Package d’accès – Demandes – Pour les utilisateurs qui sont dans votre annuaire – Deuxième approbateur](./media/entitlement-management-access-package-approval-policy/in-directory-second-approver.png)

    Si les utilisateurs ne se sont pas dans votre annuaire, sélectionnez **Sponsor interne** ou **Sponsor externe** comme deuxième approbateur. Après avoir sélectionné l’approbateur, ajoutez les approbateurs de secours.

    ![Package d’accès – Demandes – Pour les utilisateurs qui ne sont pas dans votre annuaire – Deuxième approbateur](./media/entitlement-management-access-package-approval-policy/out-directory-second-approver.png) 

1. Spécifiez le nombre de jours pendant lesquels le deuxième approbateur doit approuver la demande dans la zone sous **La décision doit être prise dans combien de jours ?** . 

1. Définissez le commutateur Exiger la justification de l’approbateur sur **Oui** ou **Non**.

### <a name="alternate-approvers"></a>Approbateurs de substitution

Vous pouvez spécifier des approbateurs substitution, comme vous avez spécifié les premier et deuxième approbateurs qui peuvent approuver les demandes. Le fait de disposer d’approbateurs de substitution garantit que les demandes sont approuvées ou refusées avant leur expiration (délai d’expiration). Vous pouvez répertorier des approbateurs de substitution, le premier approbateur et le deuxième approbateur pour l’approbation en 2 étapes. 

En spécifiant des approbateurs de substitution, si les premier et deuxième approbateurs n’ont pas été en mesure d’approuver ou de refuser la demande, la demande en attente est transférée aux approbateurs de substitution, conformément à la planification de transfert que vous avez spécifiée lors de la configuration de la stratégie. Ils reçoivent un e-mail pour approuver ou refuser la demande en attente.

Une fois la demande transférée aux approbateurs de substitution, les premier et deuxième approbateurs peuvent toujours l’approuver ou la refuser. Pour approuver ou refuser la demande en attente, les approbateurs de substitution utilisent le même site Mon Accès.

Nous pouvons répertorier les personnes ou groupes de personnes qui doivent être approbateurs et approbateurs de substitution. Veillez à répertorier des groupes de personnes différents pour les premier et deuxième approbateurs ainsi que les approbateurs de substitution.
Par exemple, si vous avez mentionné Alice et Bob comme premier et deuxième approbateurs, mentionnez Carol et Dave comme approbateurs de substitution. Pour ajouter des approbateurs de substitution à un package d’accès, procédez comme suit :

1. Sous Premier approbateur, Deuxième approbateur, ou les deux, cliquez sur **Afficher les paramètres de demande avancés**.

    ![Package d’accès – Stratégie – Afficher les paramètres de demande avancés](./media/entitlement-management-access-package-approval-policy/alternate-approvers-click-advanced-request.png)

1. Définissez l’option **En l’absence d’action, transférer à d’autres approbateurs ?** sur **Oui**.

1. Cliquez sur **Ajouter des approbateurs de substitution**, puis sélectionnez les approbateurs de substitution dans la liste.

    ![Package d’accès – Stratégie – Ajouter des approbateurs de substitution](./media/entitlement-management-access-package-approval-policy/alternate-approvers-add.png)

1. Dans la zone **Passer aux autres approbateurs après combien de jours ?** , entrez le nombre de jours dont les approbateurs disposent pour approuver ou refuser une demande. Si aucun approbateur n’a approuvé ou refusé la demande pendant la durée de validité de celle-ci, la demande expire (délai d’expiration) et l’utilisateur doit soumettre une autre demande pour le package d’accès. 

    Les demandes ne peuvent être transférées à d’autres approbateurs qu’une journée après que la demande a atteint la moitié de sa durée de vie, et le délai d’expiration de la décision de l’approbateur principal doit être d’au moins quatre jours. Si le délai d’expiration de la demande est inférieur ou égal à trois jours, il n’y a pas assez de temps pour transférer la demande à d’autres approbateurs. Dans cet exemple, la durée de la demande est de 14 jours. Ainsi, la durée de la demande atteint sa demi-vie le jour 7. La demande ne peut donc pas être transférée avant le jour 8. Par ailleurs, les demandes ne peuvent pas être transférées le dernier jour de leur durée. Ainsi, dans l’exemple, l’ultime limite pour le transfert de la demande est le jour 13.

## <a name="enable-requests"></a>Activer les demandes

1. Si vous souhaitez que le package d’accès soit immédiatement disponible afin d’être demandé par les utilisateurs dans la stratégie de demandes, déplacez le bouton bascule Activer sur **Oui**.

    Vous pouvez toujours l’activer plus tard, après avoir créé le package d’accès.

    Si vous avez sélectionné **Aucun (attributions directes d’administrateur uniquement)** et que vous affectez la valeur **No** à Activer, les administrateurs ne pourront pas attribuer directement ce package d’accès.

    ![Package d’accès - Stratégie - Paramètre Activer la stratégie](./media/entitlement-management-access-package-approval-policy/enable-requests.png)

1. Cliquez sur **Suivant**.

## <a name="collect-additional-requestor-information-for-approval-preview"></a>Collecter des informations supplémentaires sur le demandeur pour approbation (préversion)

Pour veiller à ce que les utilisateurs aient accès aux packages d'accès appropriés, vous pouvez demander aux demandeurs de compléter un champ de texte personnalisé ou de répondre à des questions à choix multiples au moment de la demande. La limite est fixée à 20 questions par stratégie et à 25 réponses pour les questions à choix multiples. Les questions seront ensuite présentées aux approbateurs pour les aider à prendre une décision.

1. Accédez à l'onglet **Informations sur le demandeur**, puis cliquez sur le sous-onglet **Questions**.
 
1. Entrez la question que vous souhaitez poser au demandeur, également appelée « chaîne d'affichage », dans la zone **Question**.

    ![Package d'accès - Stratégie - Activation du paramètre Informations sur le demandeur](./media/entitlement-management-access-package-approval-policy/add-requestor-info-question.png)

1. Si les utilisateurs qui auront besoin d'accéder au package d'accès ne parlent pas tous la même langue, vous pouvez améliorer leur expérience sur myaccess.microsoft.com. Pour ce faire, vous pouvez fournir des chaînes d'affichage alternatives dans différentes langues. Par exemple, si le navigateur web d'un utilisateur est défini sur Espagnol et que vous avez configuré des chaînes d'affichage en espagnol, ces chaînes apparaîtront sur l'écran de l'utilisateur demandeur. Pour configurer la localisation des demandes, cliquez sur **Ajouter une localisation**.
    1. Une fois dans le volet **Ajouter des localisations pour les questions**, sélectionnez le **code de la langue** dans laquelle vous localisez la question.
    1. Dans la langue que vous avez configurée, entrez la question dans la zone **Texte localisé**.
    1. Une fois que vous avez ajouté toutes les localisations nécessaires, cliquez sur **Enregistrer**.

    ![Package d'accès - Stratégie - Configuration du texte localisé](./media/entitlement-management-access-package-approval-policy/add-localization-question.png)

1. Sélectionnez le **Format de réponse** dans lequel vous souhaitez que les demandeurs répondent : *texte court*, *choix multiples* et *texte long*.
 
    ![Package d'accès - Stratégie - Sélection du bouton Afficher et modifier le format de réponse Choix multiples](./media/entitlement-management-access-package-approval-policy/answer-format-view-edit.png)
 
1. Si vous sélectionnez Choix multiples, cliquez sur le bouton **Afficher et modifier** pour configurer les options de réponse.
    1. Une fois que vous avez sélectionné Afficher et modifier, le volet **Afficher/modifier la question** s'ouvre.
    1. Entrez les options de réponse que vous souhaitez proposer au demandeur lorsqu'il répond à la question dans les zones **Valeurs de réponse**.
    1. Entrez le nombre de réponses dont vous avez besoin, puis cliquez sur **Enregistrer**.
    
    ![Package d'accès - Stratégie - Saisie des options de choix multiples](./media/entitlement-management-access-package-approval-policy/answer-multiple-choice.png)
  
1. Pour exiger que les demandeurs répondent à cette question lorsqu'ils demandent l'accès à un package d'accès, cochez la case située sous **Obligatoire**.

1. Complétez les autres onglets (par exemple, Cycle de vie) en fonction de vos besoins.

Après avoir configuré les informations relatives au demandeur dans votre stratégie d'accès, vous pouvez consulter les réponses de celui-ci. Pour obtenir des conseils sur l'affichage des informations relatives au demandeur, consultez la section [Afficher les réponses du demandeur (préversion)](entitlement-management-request-approve.md#view-requestors-answers-to-questions-preview).

## <a name="next-steps"></a>Étapes suivantes
- [Changer les paramètres de cycle de vie pour un package d’accès](entitlement-management-access-package-lifecycle-policy.md)
- [Afficher les requêtes d’un package d’accès](entitlement-management-access-package-requests.md)
