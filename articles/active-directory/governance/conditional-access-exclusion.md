---
title: Gérer les utilisateurs exclus des stratégies d’accès conditionnel - Azure AD
description: Découvrez comment utiliser les révisions d’accès Azure Active Directory (Azure AD) pour gérer les utilisateurs exclus des stratégies d’accès conditionnel
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: compliance
ms.date: 12/23/2020
ms.author: barclayn
ms.reviewer: mwahl
ms.collection: M365-identity-device-management
ms.openlocfilehash: 426e28048ae370919529ea710717a3a3867d999d
ms.sourcegitcommit: 6e2d37afd50ec5ee148f98f2325943bafb2f4993
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97746250"
---
# <a name="use-azure-ad-access-reviews-to-manage-users-excluded-from-conditional-access-policies"></a>Utiliser les révisions d’accès Azure AD pour gérer les utilisateurs exclus des stratégies d’accès conditionnel

Idéalement, tous les utilisateurs respectent les stratégies d’accès pour sécuriser l’accès aux ressources de votre organisation. Toutefois, certains scénarios métiers nécessitent que vous fassiez des exceptions. Cet article présente des exemples de situations où des exclusions peuvent être nécessaires. En votre qualité d’administrateur informatique, vous pouvez gérer cette tâche, éviter d’avoir à superviser les exceptions de stratégie et fournir aux auditeurs la preuve de l’examen régulier de ces exceptions à l’aide des révisions d’accès Azure Active Directory (Azure AD).

>[!NOTE]
> L’utilisation des révisions d’accès Azure AD requiert une licence payante ou d’essai Azure AD Premium P2 ou Enterprise Mobility + Security E5 valide. Pour plus d’informations, consultez la page [Éditions d’Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="why-would-you-exclude-users-from-policies"></a>Pourquoi exclure certains utilisateurs des stratégies ?

Supposons qu’en tant qu’administrateur, vous décidiez d’utiliser un [accès conditionnel Azure AD](../conditional-access/concept-conditional-access-policy-common.md) pour exiger une authentification multifacteur (MFA) et limiter les demandes d’authentification à des réseaux ou appareils spécifiques. Au cours de la planification du déploiement, vous vous rendez compte que certaines utilisateurs ne peuvent pas satisfaire ces exigences. Par exemple, vous pouvez avoir des utilisateurs qui travaillent à partir de bureaux distants ne faisant pas partie de votre réseau interne. Vous devez peut-être aussi prendre en charge des utilisateurs qui se connectent à l’aide d’appareils non pris en charge en attendant le remplacement de ceux-ci. En bref, l’entreprise ayant besoin que ces utilisateurs se connectent pour accomplir leur travail, vous les excluez des stratégies d’accès conditionnel.

Il se peut également que vous utilisiez des [emplacements nommés](../conditional-access/location-condition.md) dans l’accès conditionnel afin de spécifier un ensemble de pays et régions à partir desquels vous ne voulez pas que des utilisateurs soient autorisés à accéder à leur locataire.

![Emplacements nommés dans l’accès conditionnel](./media/conditional-access-exclusion/named-locations.png)

Malheureusement, certains utilisateurs peuvent avoir encore une raison valable de se connecter à partir de ces pays/régions bloqués. Par exemple, des utilisateurs en déplacement professionnel peuvent avoir besoin d’accéder aux ressources de l’entreprise. Dans ce cas, la stratégie d’accès conditionnel visant à bloquer ces pays/régions pourrait utiliser un groupe de sécurité cloud pour les utilisateurs exclus de la stratégie. Les utilisateurs qui nécessitent un accès dans le cadre de leurs déplacements peuvent alors s’ajouter à ce groupe à l’aide de la fonctionnalité de [gestion de groupes en libre-service Azure AD](../enterprise-users/groups-self-service-management.md).

Vous avez peut-être aussi une stratégie d’accès conditionnel qui [bloque l’authentification héritée pour la grande majorité de vos utilisateurs](https://cloudblogs.microsoft.com/enterprisemobility/2018/06/07/azure-ad-conditional-access-support-for-blocking-legacy-auth-is-in-public-preview/). Cependant, si certains de vos utilisateurs doivent pouvoir utiliser des méthodes d’authentification héritées pour accéder à vos ressources par le biais de clients Office 2010 ou IMAP/SMTP/POP, vous pouvez les exclure de la stratégie bloquant les méthodes d’authentification héritées.

>[!NOTE]
>Microsoft vous recommande vivement de bloquer l’utilisation des protocoles hérités dans votre locataire afin de renforcer vos mesures de sécurité.

## <a name="why-are-exclusions-challenging"></a>Pourquoi les exclusions sont-elles complexes ?

Dans Azure AD, vous pouvez restreindre une stratégie d’accès conditionnel à un ensemble d’utilisateurs. Vous pouvez également configurer des exclusions en sélectionnant des rôles Azure AD, des utilisateurs individuels ou des invités. Gardez à l’esprit que, lorsque des exclusions sont configurées, il n’est pas possible d’appliquer l’intention de la stratégie à des utilisateurs exclus. Si des exclusions sont configurées à l’aide d’une liste d’utilisateurs ou de groupes de sécurité locaux hérités, la visibilité des exclusions est limitée. Par conséquent :

- Des utilisateurs peuvent ignorer qu’ils sont exclus.

- Des utilisateurs peuvent rejoindre le groupe de sécurité pour contourner la stratégie.

- Des utilisateurs qui étaient éligibles pour l’exclusion avant l’application de la stratégie ne le sont peut-être plus.

Souvent, lorsque vous configurez une exclusion pour la première fois, il existe une courte liste d’utilisateurs auxquels la stratégie ne s’applique pas. Au fil du temps, de plus en plus d’utilisateurs sont ajoutés à l’exclusion, et la liste s’allonge. À un moment donné, vous devez examiner la liste et confirmer que chacun des utilisateurs qui y sont répertoriés est toujours éligible pour l’exclusion. Si la gestion de cette liste d’exclusion peut sembler relativement simple d’un point de vue technique, vous devez déterminer qui prend les décisions à ce sujet et comment vous assurer qu’elles sont toutes vérifiables. Toutefois, si vous configurez l’exclusion à l’aide d’un groupe Azure AD, vous pouvez utiliser des révisions d’accès en guise de contrôle compensatoire afin d’apporter de la visibilité et de réduire le nombre d’utilisateurs exclus.

## <a name="how-to-create-an-exclusion-group-in-a-conditional-access-policy"></a>Création d’un groupe d’exclusions dans une stratégie d’accès conditionnel

Suivez la procédure ci-après pour créer un groupe Azure AD et une stratégie d’accès conditionnel qui ne s’applique pas à ce groupe.

### <a name="create-an-exclusion-group"></a>Créer un groupe d’exclusions

1. Connectez-vous au portail Azure.

2. Dans le volet de navigation gauche, cliquez sur **Azure Active Directory**, puis sur **Groupes**.

3. Dans le menu supérieur, cliquez sur **Nouveau groupe** pour ouvrir le volet de groupe.

4. Dans la liste **Type de groupe**, sélectionnez **Sécurité**. Spécifiez un nom et une description.

5. Veillez à définir le type **Appartenance** sur **Attribuée**.

6. Sélectionnez les utilisateurs qui doivent faire partie de ce groupe d’exclusions, puis cliquez sur **Créer**.

![Volet de nouveau groupe d’Azure Active Directory](./media/conditional-access-exclusion/new-group.png)

### <a name="create-a-conditional-access-policy-that-excludes-the-group"></a>Créer une stratégie d’accès conditionnel excluant le groupe

Vous pouvez à présent créer une stratégie d’accès conditionnel qui utilise ce groupe d’exclusions.

1. Dans le volet de navigation gauche, cliquez sur **Azure Active Directory**, puis cliquez sur **Accès conditionnel** pour ouvrir le panneau **Stratégies**.

2. Cliquez sur **Nouvelle stratégie** pour ouvrir le volet **Nouveau**.

3. Spécifiez un nom.

4. Sous Affectations, cliquez sur **Utilisateurs et groupes**.

5. Dans l’onglet **Inclure**, sélectionnez **Tous les utilisateurs**.

6. Sous l’onglet **Exclure**, cochez la case **Utilisateurs et groupes**, puis cliquez sur **Sélectionner les utilisateurs exclus**.

7. Sélectionnez le groupe d’exclusions que vous avez créé.

   > [!NOTE] 
   > Une bonne pratique consiste à exclure au moins un compte d’administrateur de la stratégie lors du test pour vous assurer que votre accès à votre locataire n’est pas verrouillé.

8. Continuez à configurer la stratégie d’accès conditionnel en fonction des exigences de votre organisation.

![Sélectionner le volet des utilisateurs exclus dans l’accès conditionnel](./media/conditional-access-exclusion/select-excluded-users.png)
  
Examinons deux exemples dans lesquels vous pouvez utiliser les révisions d’accès pour gérer les exclusions dans les stratégies d’accès conditionnel.

## <a name="example-1-access-review-for-users-accessing-from-blocked-countriesregions"></a>Exemple 1 : Révision d’accès relative aux utilisateurs accédant au locataire à partir de pays/régions bloqués

Supposons que vous disposiez d’une stratégie d’accès conditionnel qui bloque l’accès à partir de certains pays/régions. Vous avez créé un groupe qui est exclu de la stratégie. Voici une révision d’accès recommandée qui passe en revue les membres du groupe.

> [!NOTE] 
> La création des révisions d’accès requiert un rôle Administrateur général ou un rôle Administrateur des utilisateurs.

1. La révision a lieu chaque semaine.

2. Elle ne prend jamais fin pour garantir que ce groupe d’exclusions reste à jour.

3. La révision porte sur tous les membres de ce groupe.

4. Chaque utilisateur devant lui-même attester qu’il a toujours besoin d’un accès à partir des pays/régions bloqués, il doit toujours être membre du groupe.

5. Si l’utilisateur ne répond pas à la demande de révision, il est automatiquement supprimé du groupe et ne peut plus accéder au locataire quand il est en déplacement dans ces pays/régions.

6. Activez les notifications par e-mail pour informer les utilisateurs du début et de la fin de la révision d’accès.

    ![Volet Créer une révision d’accès pour l’exemple 1](./media/conditional-access-exclusion/create-access-review-1.png)

## <a name="example-2-access-review-for-users-accessing-with-legacy-authentication"></a>Exemple 2 : Révision d’accès relative aux utilisateurs accédant au locataire avec l’authentification héritée

Supposons que vous disposez d’une stratégie d’accès conditionnel qui bloque l’accès des utilisateurs qui utilisent une authentification héritée et des versions de client plus anciennes, et qui spécifie un groupe exclu de la stratégie. Voici une révision d’accès recommandée qui passe en revue les membres du groupe.

1. Cette révision doit être récurrente.

2. La révision doit porter sur tous les membres du groupe.

3. La révision peut être configurée pour répertorier les propriétaires de division en tant que réviseurs sélectionnés.

4. Appliquez automatiquement les résultats et supprimez les utilisateurs qui ne sont pas autorisés à poursuivre l’utilisation des méthodes d’authentification héritées.

5. Il peut être avantageux d’activer les suggestions de manière à faciliter la prise de décisions des réviseurs de groupes volumineux.

6. Activez les notifications par e-mail afin que les utilisateurs soient avertis du démarrage et de l’achèvement de la révision d’accès.

    ![Volet Créer une révision d’accès pour l’exemple 2](./media/conditional-access-exclusion/create-access-review-2.png)

>[!IMPORTANT] 
>Si vous disposez de nombreux groupes d’exclusions et devez donc créer plusieurs révisions d’accès, nous proposons désormais dans le point de terminaison bêta de Microsoft Graph une API permettant de créer et de gérer ces révisions par programmation. Pour commencer, consultez les [informations de référence sur les API des révisions d’accès Azure AD](/graph/api/resources/accessreviews-root?view=graph-rest-beta), ainsi que l’article [Example of retrieving Azure AD access reviews via Microsoft Graph](https://techcommunity.microsoft.com/t5/Azure-Active-Directory/Example-of-retrieving-Azure-AD-access-reviews-via-Microsoft/td-p/236096) (Exemple de récupération de révisions d’accès Azure AD par le biais de Microsoft Graph).

## <a name="access-review-results-and-audit-logs"></a>Résultats des révisions d’accès et journaux d’audit

Une fois que vous avez mis en place tous les éléments requis (groupe, stratégie d’accès conditionnel et révisions d’accès), vous devez surveiller et suivre les résultats de ces révisions.

1. Dans le portail Azure, ouvrez le panneau **Révisions d’accès**.

2. Ouvrez le contrôle et le programme que vous avez créés pour gérer le groupe d’exclusions.

3. Cliquez sur **Résultats** pour découvrir les utilisateurs qui ont été autorisés à rester dans la liste et ceux qui ont été supprimés.

    ![Les résultats des révisions d’accès affichent les utilisateurs approuvés](./media/conditional-access-exclusion/access-reviews-results.png)

4. Ensuite, cliquez sur **Journaux d’audit** pour visualiser les actions qui ont été exécutées durant cette révision.

    ![Journaux d’audit des révisions d’accès répertoriant les actions](./media/conditional-access-exclusion/access-reviews-audit-logs.png)

En tant qu’administrateur informatique, vous savez que la gestion des groupes d’exclusions de vos stratégies se révèle parfois inévitable. Toutefois, les révisions d’accès Azure AD peuvent faciliter la tenue à jour de ces groupes, leur examen à intervalles réguliers par le propriétaire de l’entreprise ou par les utilisateurs eux-mêmes, ainsi que l’audit de ces modifications.

## <a name="next-steps"></a>Étapes suivantes

- [Créer une révision d’accès de groupes ou d’applications](create-access-review.md)
- [Qu’est-ce que l’accès conditionnel dans Azure Active Directory ?](../conditional-access/overview.md)