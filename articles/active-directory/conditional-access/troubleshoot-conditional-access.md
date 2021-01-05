---
title: Résolution des problèmes de connexion avec l’accès conditionnel - Azure Active Directory
description: Cet article décrit la procédure à suivre lorsque vos stratégies d’accès conditionnel entraînent des résultats inattendus
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: troubleshooting
ms.date: 10/16/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, martinco
ms.collection: M365-identity-device-management
ms.openlocfilehash: 12f722977329bd5d79d4d0e410a29c730faf00c5
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92145100"
---
# <a name="troubleshooting-sign-in-problems-with-conditional-access"></a>Résolution des problèmes de connexion avec l’accès conditionnel

Les informations contenues dans cet article peuvent être utilisées pour résoudre des problèmes de connexion inattendus liés à l’accès conditionnel à l’aide des messages d’erreur et du journal des connexions Azure AD.

## <a name="select-all-consequences"></a>Sélectionner « toutes » les conséquences

L’infrastructure d’accès conditionnel vous offre une souplesse de configuration exceptionnelle. Toutefois, une grande souplesse signifie également que vous devez examiner soigneusement chaque stratégie de configuration avant de la mettre en œuvre afin d’éviter des résultats indésirables. Dans ce contexte, prêtez une attention particulière à l’affectation de jeux complets comme par exemple **tous les utilisateurs / groupes / applications cloud** .

Les organisations doivent éviter les configurations suivantes :

**Pour tous les utilisateurs, toutes les applications cloud :**

- **Bloquer l’accès**  : cette configuration bloque toute votre organisation.
- **Exiger que l’appareil soit marqué comme conforme**  : pour les utilisateurs qui n’ont pas encore inscrit leurs appareils, cette stratégie bloque tout accès, notamment l’accès au portail Intune. Si vous êtes un administrateur sans appareil inscrit, cette stratégie vous bloque et vous ne pouvez pas retourner dans le portail Azure pour modifier la stratégie.
- **Exiger un appareil joint au domaine Azure AD Hybride**  : ce blocage d’accès par stratégie peut également bloquer l’accès pour tous les utilisateurs de votre organisation si vous n’avez pas encore d’appareil avec jointure hybride Azure AD.
- **Exiger une stratégie de protection des applications**  : ce blocage d’accès par stratégie peut également bloquer l’accès pour tous les utilisateurs de votre organisation si vous n’avez pas encore de stratégie Intune. Si vous êtes administrateur sans application cliente dotée d’une stratégie de protection des applications Intune, cette stratégie vous empêche de revenir aux portails comme Intune et Azure.

**Pour tous les utilisateurs, toutes les applications cloud, toutes les plates-formes d’appareils :**

- **Bloquer l’accès**  : cette configuration bloque toute votre organisation.

## <a name="conditional-access-sign-in-interrupt"></a>Interruption de connexion avec l’accès conditionnel

Vous devez tout d’abord consulter le message d’erreur qui s’affiche. Pour les problèmes de connexion lors de l’utilisation d’un navigateur web, la page d’erreur elle-même contient des informations détaillées. Ces informations peuvent décrire le problème et suggérer une solution.

![Erreur de connexion - Appareil conforme requis](./media/troubleshoot-conditional-access/image1.png)

Dans l’erreur ci-dessus, le message indique que l’application est accessible uniquement à partir d’appareils ou d’applications clientes qui respectent la stratégie de gestion des appareils mobiles de l’entreprise. Dans le cas présent, l’application et l’appareil ne sont pas conformes à cette stratégie.

## <a name="azure-ad-sign-in-events"></a>Événements de connexion Azure AD

La deuxième méthode permettant d’obtenir des informations détaillées sur l’interruption de connexion consiste à passer en revue les événements de connexion Azure AD pour savoir quelles stratégies d’accès conditionnel ont été appliquées et pourquoi.

Pour plus d’informations sur le problème, cliquez sur **Plus de détails** dans la page d’erreur initiale. Cliquez sur **Plus de détails** pour afficher des informations de dépannage utiles lors de la recherche de l’événement d’échec spécifique que l’utilisateur a vu dans les événements de connexion Azure AD ou lors de l’ouverture d’un incident de support auprès de Microsoft.

![Plus d’informations sur la connexion interrompue d’un navigateur web avec l’accès conditionnel.](./media/troubleshoot-conditional-access/image2.png)

Procédez comme suit pour trouver quelles stratégies d’accès conditionnel ont été appliquées et pourquoi.

1. Connectez-vous au **portail Azure** en tant qu’administrateur général, administrateur de la sécurité ou lecteur général.
1. Accédez à **Azure Active Directory** > **Connexions** .
1. Recherchez l’événement de connexion à vérifier. Ajoutez ou supprimez des filtres et des colonnes pour filtrer les informations inutiles.
   1. Ajoutez des filtres pour limiter l’étendue :
      1. **ID de corrélation** lorsque vous avez un événement spécifique à examiner.
      1. **Accès conditionnel** pour voir l’échec et la réussite de la stratégie. Pour limiter les résultats, restreignez votre filtre afin de n’afficher que les échecs.
      1. **Nom d’utilisateur** pour afficher des informations relatives à des utilisateurs spécifiques.
      1. **Date** limitée au laps de temps en question.

   ![Sélection du filtre d’accès conditionnel dans le journal des connexions](./media/troubleshoot-conditional-access/image3.png)

1. Une fois que l’événement de connexion qui correspond à l’échec de connexion de l’utilisateur a été trouvé, sélectionnez l’onglet **Accès conditionnel** . L’onglet Accès conditionnel affiche la ou les stratégies spécifiques qui ont abouti à l’interruption de la connexion.
   1. Les informations de l’onglet **Dépannage et support** peuvent indiquer clairement pourquoi une connexion a échoué, par exemple un appareil ne respectant pas les exigences de conformité.
   1. Pour approfondir vos recherches, explorez la configuration des stratégies en cliquant sur **Nom de la stratégie** . Cliquez sur **Nom de la stratégie** pour afficher l’interface utilisateur de configuration de la stratégie pour la stratégie sélectionnée à des fins de révision et de modification.
   1. **L’utilisateur client** et les **détails sur l’appareil** qui ont été utilisés pour l’évaluation de la stratégie d’accès conditionnel sont également disponibles dans les onglets **Informations de base** , **Emplacement** , **Informations sur l’appareil** , **Détails d’authentification** et **Détails supplémentaires** de l’événement de connexion.

### <a name="policy-details"></a>Détails de la stratégie

La sélection des points de suspension à droite de la stratégie dans un événement de connexion permet d’afficher les détails de la stratégie. Les administrateurs peuvent ainsi obtenir des informations supplémentaires sur la raison pour laquelle une stratégie a été correctement appliquée ou non.

   ![Événement de connexion - Onglet Accès conditionnel](./media/troubleshoot-conditional-access/image5.png)

   ![Détails de la stratégie (préversion)](./media/troubleshoot-conditional-access/policy-details.png)

Le côté gauche fournit les détails collectés lors de la connexion et le côté droit indique si ces détails répondent aux exigences des stratégies d’accès conditionnel appliquées. Les stratégies d’accès conditionnel s’appliquent uniquement lorsque toutes les conditions sont satisfaites ou non configurées.

Si les informations de l’événement ne suffisent pas à comprendre les résultats de la connexion ou à ajuster la stratégie pour obtenir les résultats souhaités, il est possible d’ouvrir un incident de support. Accédez à l’onglet **Dépannage et support** de cet événement de connexion, puis sélectionnez **Créer une demande de support** .

![Onglet Dépannage et support de l’événement de connexion](./media/troubleshoot-conditional-access/image6.png)

Lors de l’envoi de l’incident, fournissez l’ID de la demande, ainsi que l’heure et la date de l’événement de connexion dans les détails d’envoi de l’incident. Ces informations permettent au support Microsoft de trouver l’événement qui vous intéresse.

### <a name="conditional-access-error-codes"></a>Codes d’erreur pour l’accès conditionnel

| Code d’erreur de connexion | Chaîne d’erreur |
| --- | --- |
| 53000 | DeviceNotCompliant |
| 53001 | DeviceNotDomainJoined |
| 53002 | ApplicationUsedIsNotAnApprovedApp |
| 53003 | BlockedByConditionalAccess |
| 53004 | ProofUpBlockedDueToRisk |

## <a name="what-to-do-if-you-are-locked-out-of-the-azure-portal"></a>Que faire si votre accès au portail Azure est verrouillé ?

Si vous ne pouvez pas accéder au portail Azure en raison d’un paramètre incorrect dans une stratégie d’accès conditionnel :

- Vérifiez s’il existe d’autres administrateurs dans votre organisation dont l’accès n’a pas encore été verrouillé. Un administrateur ayant accès au portail Azure peut désactiver la stratégie qui entrave votre connexion. 
- Si aucun des administrateurs de votre organisation ne peut mettre à jour la stratégie, soumettez une demande de support. Le support Microsoft peut vérifier et, sur confirmation, mettre à jour les stratégies d’accès conditionnel qui empêchent l’accès.

## <a name="next-steps"></a>Étapes suivantes

- [Rapports d’activité de connexion dans le portail Azure Active Directory](../reports-monitoring/concept-sign-ins.md)
- [Résolution des problèmes d’accès conditionnel à l’aide de l’outil What If](troubleshoot-conditional-access-what-if.md)
