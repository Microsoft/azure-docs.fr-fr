---
title: 'Rapports de réinitialisation de mot de passe libre-service : Azure Active Directory'
description: Rapports sur les événements de réinitialisation de mot de passe en libre-service Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 02/01/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rhicock
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6a764538ebc73927b1d274b2538e123ec90ce60e
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96741556"
---
# <a name="reporting-options-for-azure-ad-password-management"></a>Options de création de rapports pour la gestion des mots de passe Azure AD

Après le déploiement, de nombreuses organisations souhaitent savoir comment ou si la réinitialisation de mot de passe en libre-service (SSPR) est réellement utilisée. La fonctionnalité de création de rapports fournie par Azure Active Directory (Azure AD) vous aide à répondre aux questions à l’aide de rapports prédéfinis. Si vous disposez d’une licence appropriée, vous pouvez également créer des requêtes personnalisées.

![Génération des rapports sur SSPR à l’aide des journaux d'audit dans Azure AD][Reporting]

Les questions suivantes peuvent trouver réponse dans les rapports qui existent dans le [portail Azure](https://portal.azure.com/) :

> [!NOTE]
> Vous devez être [un administrateur global](../roles/permissions-reference.md) et vous devez vous abonner pour que ces données soient collectées pour votre organisation. Pour vous abonner, vous devez consulter l’onglet **Rapports** ou les journaux d’audit au moins une fois. Tant que cela n’est pas fait, les données ne sont pas collectées pour votre organisation.
>

* Combien de personnes se sont inscrites pour la réinitialisation des mots de passe ?
* Qui s’est inscrit pour la réinitialisation des mots de passe ?
* Quelles sont les données inscrites par ces personnes ?
* Combien de personnes ont réinitialisé leurs mots de passe au cours des sept derniers jours ?
* Quelles sont les méthodes les plus courantes employées par les utilisateurs ou les administrateurs pour réinitialiser leurs mots de passe ?
* Quels sont les problèmes courants rencontrés par les utilisateurs ou les administrateurs lors des tentatives d’utilisation de la fonctionnalité de réinitialisation des mots de passe ?
* Quels sont les administrateurs qui réinitialisent fréquemment leurs mots de passe ?
* Y a-t-il des activités suspectes en rapport avec la réinitialisation des mots de passe ?

## <a name="how-to-view-password-management-reports-in-the-azure-portal"></a>Comment visualiser les rapports de gestion des mots de passe dans le portail Azure

Dans l’interface du portail Azure, nous avons amélioré la méthode d’affichage de l’activité de réinitialisation de mot de passe et d’inscription de réinitialisation de mot de passe. Utilisez les étapes suivantes pour trouver les événements de réinitialisation de mot de passe et d’inscription de réinitialisation de mot de passe :

1. Accédez au [portail Azure](https://portal.azure.com).
2. Sélectionnez **Tous les services** dans le volet gauche.
3. Recherchez **Azure Active Directory** dans la liste des services, puis sélectionnez-le.
4. Dans la section Gérer, sélectionnez **Utilisateurs**.
5. Sélectionnez **Journaux d’audit** dans le panneau **Utilisateurs**. Vous affichez ainsi tous les événements d’audit qui ont concerné tous les utilisateurs de votre annuaire. Vous pouvez filtrer cette vue pour voir tous les événements associés aux mots de passe.
6. Dans le menu **Filtrer** situé en haut du volet, sélectionnez la liste déroulante **Service** et remplacez-la par le type de service **Gestion des mots de passe en libre-service**.
7. Si vous le souhaitez, vous pouvez affiner le filtre de la liste en choisissant l’**Activité** spécifique qui vous intéresse.

### <a name="combined-registration"></a>Inscription combinée

Si vous avez activé l’[inscription combinée](./concept-registration-mfa-sspr-combined.md), les informations relatives à l’activité des utilisateurs dans les journaux d’audit se trouvent sous **Sécurité** > **Méthodes d’authentification**.

## <a name="description-of-the-report-columns-in-the-azure-portal"></a>Description des colonnes du rapport dans le portail Azure

La liste suivante décrit chacune des colonnes du rapport dans le portail Azure en détail :

* **Utilisateur** : utilisateur ayant tenté d’effectuer une opération d’inscription de réinitialisation de mot de passe.
* **Rôle** : rôle de l’utilisateur dans l’annuaire.
* **Date et heure** : date et heure de la tentative.
* **Données inscrites** : données d’authentification fournies par l’utilisateur lors de l’inscription de réinitialisation de mot de passe.

## <a name="description-of-the-report-values-in-the-azure-portal"></a>Description des valeurs du rapport dans le portail Azure

Le tableau suivant décrit les différentes valeurs que vous pouvez définir pour chaque colonne dans le portail Azure :

| Colonne | Valeurs autorisées et leur signification |
| --- | --- |
| Données inscrites |**Autre adresse e-mail** : l’utilisateur a utilisé une autre adresse e-mail ou une adresse e-mail d’authentification pour s’authentifier.<p><p>**Téléphone de bureau** : l’utilisateur a utilisé un numéro professionnel pour s’authentifier.<p>**Téléphone mobile** : l’utilisateur a utilisé un téléphone mobile ou un téléphone d’authentification pour s’authentifier.<p>**Questions de sécurité** : l’utilisateur a utilisé des questions de sécurité pour s’authentifier.<p>**Toute combinaison des méthodes précédentes, par exemple, autre adresse e-mail + téléphone mobile** : se produit lorsqu’une stratégie à deux verrous est spécifiée et indique les deux méthodes employées par l’utilisateur pour authentifier sa demande de réinitialisation de mot de passe. |

## <a name="self-service-password-management-activity-types"></a>Types d’activités de gestion des mots de passe en libre-service

Les types d’activités suivants s’affichent dans la catégorie d’événement d’audit **Self-Service Password Management** (Gestion des mots de passe en libre-service) :

* [Blocked from self-service password reset](#activity-type-blocked-from-self-service-password-reset) (Blocage suite à une réinitialisation de mot de passe en libre-service) : indique qu’un utilisateur a tenté de réinitialiser un mot de passe, d’utiliser un verrou spécifique ou de valider un numéro de téléphone plus de cinq fois au total en l’espace de 24 heures.
* [Change password (self-service)](#activity-type-change-password-self-service) (Modification du mot de passe (libre-service)) : indique qu’un utilisateur a effectué une modification de mot de passe volontaire ou forcée (en raison d’une expiration).
* [Reset password (by admin)](#activity-type-reset-password-by-admin) (Réinitialisation du mot de passe (par l’administrateur)) : indique qu’un administrateur a effectué une réinitialisation de mot de passe pour le compte d’un utilisateur à partir du portail Azure.
* [Reset password (self-service)](#activity-type-reset-password-self-service) (Réinitialisation du mot de passe (libre-service)) : indique qu’un utilisateur a correctement réinitialisé son mot de passe à partir du [portail de réinitialisation de mot de passe Azure AD](https://passwordreset.microsoftonline.com).
* [Self serve password reset flow activity progress](#activity-type-self-serve-password-reset-flow-activity-progress) (Progression de l’activité du flux de réinitialisation du mot de passe en libre-service) : indique chaque étape spécifique effectuée par un utilisateur, comme le passage d’un verrou d’authentification de réinitialisation de mot de passe spécifique, dans le cadre du processus de réinitialisation de mot de passe.
* [Unlock user account (self-service)](#activity-type-unlock-a-user-account-self-service) (Déverrouillage du compte d’utilisateur (libre-service)) : indique qu’un utilisateur a correctement déverrouillé son compte Active Directory sans réinitialiser son mot de passe à partir du [portail de réinitialisation de mot de passe Azure AD](https://passwordreset.microsoftonline.com) à l’aide de la fonctionnalité Active Directory de déverrouillage de compte sans réinitialisation.
* [User registered for self-service password reset](#activity-type-user-registered-for-self-service-password-reset) (Utilisateur inscrit pour la réinitialisation du mot de passe en libre-service) : indique qu’un utilisateur a inscrit toutes les informations nécessaires pour pouvoir réinitialiser son mot de passe conformément à la stratégie de réinitialisation de mot de passe du locataire actuellement spécifiée.

### <a name="activity-type-blocked-from-self-service-password-reset"></a>Type d’activité : Blocage suite à une réinitialisation de mot de passe en libre-service

La liste suivante explique cette activité en détail :

* **Activity description** (Description de l’activité) : indique qu’un utilisateur a tenté de réinitialiser un mot de passe, d’utiliser un verrou spécifique ou de valider un numéro de téléphone plus de cinq fois au total en l’espace de 24 heures.
* **Activity actor** (Acteur de l’activité) : l’utilisateur qui a été empêché d’effectuer des opérations supplémentaires de réinitialisation de mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity target** (Cible de l’activité) : l’utilisateur qui a été empêché d’effectuer des opérations supplémentaires de réinitialisation de mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity status** (État de l’activité) :
  * _Réussite_ : indique qu’un utilisateur a été empêché d’effectuer des réinitialisations supplémentaires, d’essayer des méthodes d’authentification supplémentaires ou de valider des numéros de téléphone supplémentaires pendant les prochaines 24 heures.
* **Activity status failure reason** (Motif de l’état de l’activité Failure) : Non applicable.

### <a name="activity-type-change-password-self-service"></a>Type d’activité : Modifier un mot de passe (libre-service)

La liste suivante explique cette activité en détail :

* **Activity description** (Description de l’activité) : indique qu’un utilisateur a effectué une modification de mot de passe volontaire ou forcée (en raison d’une expiration).
* **Activity actor** (Acteur de l’activité) : l’utilisateur qui a modifié son mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity target** (Cible de l’activité) : l’utilisateur qui a modifié son mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity statuses** (États de l'activité) :
  * _Réussite_ : indique qu’un utilisateur a modifié son mot de passe avec succès.
  * _Échec_ : indique qu’un utilisateur n’a pas réussi à modifier son mot de passe. Vous pouvez sélectionner la ligne pour voir la catégorie **Activity status reason** (Motif de l’état de l’activité) et en savoir plus sur la cause de l’échec.
* **Activity status failure reason** (Motif de l’état de l’activité Failure) :
  * _FuzzyPolicyViolationInvalidPassword_ : l’utilisateur a sélectionné un mot de passe qui a été automatiquement interdit, car les fonctionnalités de détection de mots de passe interdits de Microsoft l’ont jugé trop banal ou trop faible.

### <a name="activity-type-reset-password-by-admin"></a>Type d’activité : Réinitialiser un mot de passe (par l’administrateur)

La liste suivante explique cette activité en détail :

* **Activity description** (Description de l’activité) : indique qu’un administrateur a effectué une réinitialisation de mot de passe pour le compte d’un utilisateur à partir du portail Azure.
* **Activity actor** (Acteur de l’activité) : l’administrateur qui a effectué la réinitialisation de mot de passe pour le compte d’un autre utilisateur ou d’un administrateur. Il doit s'agir d'un administrateur de mots de passe, d'un administrateur d'utilisateurs ou d'un administrateur du support technique.
* **Activity target** (Cible de l’activité) : l’utilisateur dont le mot de passe a été réinitialisé. L’utilisateur peut être un utilisateur final ou un autre administrateur.
* **Activity statuses** (États de l'activité) :
  * _Réussite_ : indique qu’un administrateur a correctement réinitialisé le mot de passe d’un utilisateur.
  * _Échec_ : indique qu’un administrateur n’a pas réussi à modifier le mot de passe d’un utilisateur. Vous pouvez sélectionner la ligne pour voir la catégorie **Activity status reason** (Motif de l’état de l’activité) et en savoir plus sur la cause de l’échec.

### <a name="activity-type-reset-password-self-service"></a>Type d’activité : Réinitialiser un mot de passe (libre-service)

La liste suivante explique cette activité en détail :

* **Activity description** (Description de l’activité) : indique qu’un utilisateur a correctement réinitialisé son mot de passe à partir du [portail de réinitialisation de mot de passe Azure AD](https://passwordreset.microsoftonline.com).
* **Activity actor** (Acteur de l’activité) : l’utilisateur qui a réinitialisé son mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity target** (Cible de l’activité) : l’utilisateur qui a réinitialisé son mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity statuses** (États de l'activité) :
  * _Réussite_ : indique qu’un utilisateur a correctement réinitialisé son propre mot de passe.
  * _Échec_ : indique qu’un utilisateur n’a pas réussi à réinitialiser son propre mot de passe. Vous pouvez sélectionner la ligne pour voir la catégorie **Activity status reason** (Motif de l’état de l’activité) et en savoir plus sur la cause de l’échec.
* **Activity status failure reason** (Motif de l’état de l’activité Failure) :
  * _FuzzyPolicyViolationInvalidPassword_ : l’administrateur a sélectionné un mot de passe qui a été automatiquement interdit, car les fonctionnalités de détection de mots de passe interdits de Microsoft l’ont jugé trop banal ou trop faible.

### <a name="activity-type-self-serve-password-reset-flow-activity-progress"></a>Type d’activité : Progression de l’activité du flux de réinitialisation de mot de passe en libre-service

La liste suivante explique cette activité en détail :

* **Activity description** (Description de l’activité) : indique chaque étape spécifique effectuée par un utilisateur (comme le passage d’un verrou d’authentification de réinitialisation de mot de passe spécifique) dans le cadre du processus de réinitialisation de mot de passe.
* **Activity actor** (Acteur de l’activité) : l’utilisateur qui a effectué une partie du flux de réinitialisation de mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity target** (Cible de l’activité) : l’utilisateur qui a effectué une partie du flux de réinitialisation de mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity statuses** (États de l'activité) :
  * _Réussite_ : indique qu’un utilisateur a correctement effectué une étape spécifique du flux de réinitialisation de mot de passe.
  * _Échec_ : indique qu’une étape spécifique du flux de réinitialisation de mot de passe a échoué. Vous pouvez sélectionner la ligne pour voir la catégorie **Activity status reason** (Motif de l’état de l’activité) et en savoir plus sur la cause de l’échec.
* **Activity status reasons** (Motif de l’état de l’activité) :   consultez le tableau suivant pour connaître [tous les motifs des états de l’activité de réinitialisation autorisés](#description-of-the-report-columns-in-the-azure-portal).

### <a name="activity-type-unlock-a-user-account-self-service"></a>Type d’activité : Déverrouiller un compte d’utilisateur (libre-service)

La liste suivante explique cette activité en détail :

* **Activity description** (Description de l’activité) : indique qu’un utilisateur a correctement déverrouillé son compte Active Directory sans réinitialiser son mot de passe à partir du [portail de réinitialisation de mot de passe Azure AD](https://passwordreset.microsoftonline.com) à l’aide de la fonctionnalité Active Directory de déverrouillage de compte sans réinitialisation.
* **Activity actor** (Acteur de l’activité) : l’utilisateur qui a déverrouillé son compte sans réinitialiser son mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity target** (Cible de l’activité) : l’utilisateur qui a déverrouillé son compte sans réinitialiser son mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Allowed activity statuses** (États de l’activité autorisés) :
  * _Réussite_ : indique qu’un utilisateur a correctement déverrouillé son propre compte.
  * _Échec_ : indique qu’un utilisateur n’a pas réussi à déverrouiller son propre compte. Vous pouvez sélectionner la ligne pour voir la catégorie **Activity status reason** (Motif de l’état de l’activité) et en savoir plus sur la cause de l’échec.

### <a name="activity-type-user-registered-for-self-service-password-reset"></a>Type d’activité : Utilisateur enregistré pour la réinitialisation de mot de passe en libre-service

La liste suivante explique cette activité en détail :

* **Activity description** (Description de l’activité) : indique qu’un utilisateur a inscrit toutes les informations nécessaires pour pouvoir réinitialiser son mot de passe conformément à la stratégie de réinitialisation de mot de passe du locataire actuellement spécifiée. 
* **Activity actor** (Acteur de l’activité) : l’utilisateur qui s’est inscrit pour la réinitialisation de mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Activity target** (Cible de l’activité) : l’utilisateur qui s’est inscrit pour la réinitialisation de mot de passe. L’utilisateur peut être un utilisateur final ou un administrateur.
* **Allowed activity statuses** (États de l’activité autorisés) :
  * _Réussite_ : indique qu’un utilisateur s’est inscrit pour la réinitialisation de mot de passe conformément à la stratégie actuelle. 
  * _Échec_ : indique qu’un utilisateur n’a pas réussi à s’inscrire pour la réinitialisation de mot de passe. Vous pouvez sélectionner la ligne pour voir la catégorie **Activity status reason** (Motif de l’état de l’activité) et en savoir plus sur la cause de l’échec.

     >[!NOTE]
     >Un échec ne signifie pas qu’un utilisateur ne peut pas réinitialiser son propre mot de passe. Cela signifie que processus d’inscription n’est pas terminé. Si son compte contient des données correctes non vérifiées (par exemple, un numéro de téléphone non validé), même s’il n’a pas vérifié ce numéro de téléphone, il peut toujours l’utiliser pour réinitialiser son mot de passe.

## <a name="next-steps"></a>Étapes suivantes

* [Rapport d’utilisation et d’insights SSPR et MFA](howto-authentication-methods-usage-insights.md)
* [Comment réussir le lancement de la réinitialisation de mot de passe en libre-service ?](howto-sspr-deployment.md)
* [Réinitialisez ou modifiez votre mot de passe](../user-help/active-directory-passwords-update-your-own-password.md).
* [Inscrivez-vous pour la réinitialisation du mot de passe en libre-service](../user-help/active-directory-passwords-reset-register.md).
* [Vous avez une question relative à la licence ?](concept-sspr-licensing.md)
* [Quelles données sont utilisées par la réinitialisation de mot de passe en libre-service et quelles données vous devez renseigner pour vos utilisateurs ?](howto-sspr-authenticationdata.md)
* [Quelles méthodes d'authentification sont accessibles aux utilisateurs ?](concept-sspr-howitworks.md#authentication-methods)
* [Quelles sont les options de stratégie disponibles avec la réinitialisation de mot de passe en libre-service ?](concept-sspr-policy.md)
* [Quelle est l’écriture différée de mot de passe et pourquoi dois-je m’y intéresser ?](./tutorial-enable-sspr-writeback.md)
* [Quelles sont toutes les options disponibles dans la réinitialisation de mot de passe en libre-service et que signifient-elles ?](concept-sspr-howitworks.md)
* [Je pense qu’il y a une panne quelque part. Comment puis-je résoudre les problèmes de la réinitialisation de mot de passe en libre-service ?](./troubleshoot-sspr.md)
* [J’ai une question à laquelle je n’ai pas trouvé de réponse ailleurs](active-directory-passwords-faq.md)

[Reporting]: ./media/howto-sspr-reporting/sspr-reporting.png "Exemple de journaux d’audit d’activité SSPR dans Azure AD"