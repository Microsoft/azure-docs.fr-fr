---
title: Utilisateurs non approvisionnés dans mon application
description: Résolution des problèmes courants rencontrés lorsque les utilisateurs ne figurent pas dans une application de la galerie Azure AD que vous avez configurée pour l’approvisionnement d’utilisateurs avec Azure AD
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 12/03/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 89a5838524daa1959ecf6b4fe3c17d6175ca8553
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96571774"
---
# <a name="no-users-are-being-provisioned"></a>Aucun utilisateur n’est en cours d’approvisionnement 
>[!NOTE]
>À partir du 16/04/2020, nous avons modifié le comportement des utilisateurs assignés au rôle d’accès par défaut. Pour plus d’informations, consultez la section ci-dessous. 
>
Une fois que l’approvisionnement automatique a été configuré pour une application (avec notamment la vérification des informations d’identification de l’application fournies à Azure AD pour la connexion à l’application), les utilisateurs et/ou groupes sont approvisionnés sur l’application. L’approvisionnement est déterminé par les éléments suivants :

-   Quels utilisateurs et groupes ont été **affectés** à l’application. Notez que le provisionnement des groupes imbriqués n’est pas pris en charge. Pour plus d’informations sur l’affectation, consultez [Affecter un utilisateur ou un groupe à une application d’entreprise dans Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md).
-   Les **mappages d’attributs** sont-ils activés et configurés pour synchroniser les attributs valides d’Azure AD avec l’application. Pour plus de détails sur les mappages d’attributs, consultez [Personnalisation des mappages d’attributs d’approvisionnement d’utilisateurs pour les applications SaaS dans Azure Active Directory](customize-application-attributes.md).
-   Un **filtre d’étendue** permet-il de filtrer les utilisateurs en fonction de valeurs d’attribut spécifiques. Pour plus d’informations sur les filtres d’étendue, consultez [Approvisionnement d’applications basé sur les attributs avec filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
  
  
Si vous constatez que des utilisateurs ne sont pas provisionnés, consultez les [Journaux de provisionnement (préversion)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context) dans Azure AD. Recherchez les entrées de journal d’un utilisateur spécifique.

Vous pouvez accéder aux journaux d’approvisionnement dans le portail Azure en sélectionnant **Azure Active Directory** &gt; **Applications d’entreprise** &gt; **Journaux de provisionnement (préversion)** dans la section **Activité**. Vous pouvez rechercher les données de provisionnement en fonction du nom de l’utilisateur ou de l’identificateur dans le système source ou le système cible. Pour plus d’informations, consultez [Journaux de provisionnement (préversion)](../reports-monitoring/concept-provisioning-logs.md?context=azure/active-directory/manage-apps/context/manage-apps-context). 

Les journaux de provisionnement enregistrent toutes les opérations effectuées par le service de provisionnement, y compris l’interrogation d’Azure AD concernant les utilisateurs attribués qui se trouvent dans l’étendue de provisionnement, l’interrogation de l’application cible concernant l’existence de ces utilisateurs, et la comparaison des objets utilisateur du système. Ajoutez ensuite, mettez à jour ou désactivez le compte d’utilisateur dans le système cible en fonction de cette comparaison.

## <a name="general-problem-areas-with-provisioning-to-consider"></a>Problèmes généraux d’approvisionnement à prendre en compte
Voici une liste des problèmes généraux que vous pouvez explorer si vous savez par où commencer.

- [Le service d’approvisionnement ne semble pas démarrer](#provisioning-service-does-not-appear-to-start)
- [Les journaux de provisionnement indiquent que des utilisateurs sont ignorés et non provisionnés, bien qu’ils soient attribués](#provisioning-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned)

## <a name="provisioning-service-does-not-appear-to-start"></a>Le service d’approvisionnement ne semble pas démarrer
Si vous définissez le paramètre **État de l’approvisionnement** sur **Activé** dans la section **Azure Active Directory &gt; Applications d’entreprise &gt; \[Nom de l’application\] &gt;Approvisionnement** du portail Azure. Cependant, si aucune autre information d’état n’est affichée sur cette page après de nouveaux chargements, il est probable que le service est en cours d’exécution et qu’il n’a pas encore achevé le cycle initial. Vérifiez les **Journaux de provisionnement (préversion)** décrits ci-dessus pour déterminer les opérations effectuées par le service, et la présence éventuelle d’erreurs.

>[!NOTE]
>Un cycle initial peut prendre de 20 minutes à plusieurs heures, en fonction de la taille de l’annuaire Azure AD et du nombre d’utilisateurs présents dans l’étendue du provisionnement. Les synchronisations qui suivent le cycle initial sont plus rapides, car le service de provisionnement stocke les filigranes qui représentent l’état des deux systèmes après le cycle initial. Le cycle initial améliore les performances des synchronisations suivantes.
>


## <a name="provisioning-logs-say-users-are-skipped-and-not-provisioned-even-though-they-are-assigned"></a>Les journaux de provisionnement indiquent que les utilisateurs sont ignorés et non provisionnés, bien qu’ils soient attribués

Lorsqu’un utilisateur apparaît comme « ignoré » dans les journaux de provisionnement, il est important d’examiner l’onglet **Étapes** du journal pour en déterminer la raison. Voici les raisons les plus courantes et les solutions correspondantes :

- **Un filtre d’étendue a été configuré,** **qui exclut l’utilisateur en fonction d’une valeur d’attribut**. Pour plus d’informations sur les filtres d’étendue, consultez [Filtres d’étendue](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
- **L’utilisateur n’est pas « autorisé de manière effective ».** Ce message d’erreur indique un problème concernant l’enregistrement d’affectation d’utilisateurs stocké dans Azure AD. Pour résoudre ce problème, supprimez l’assignation de l’utilisateur (ou du groupe) de l’application, puis réassignez-le. Pour plus d’informations sur l’assignation, consultez [Affecter un utilisateur ou un groupe à une application d’entreprise dans Azure Active Directory](../manage-apps/assign-user-or-group-access-portal.md).
- **Un attribut requis manque ou n’est pas indiqué pour un utilisateur.** Lors de la configuration de l’approvisionnement, il est important de vérifier et configurer les mappages d’attributs et les workflows qui définissent les propriétés de l’utilisateur (ou du groupe) passant d’Azure AD à l’application. Cette configuration inclut la définition d’une « propriété correspondante » réservée à l’identification et à la mise en correspondance des utilisateurs/groupes entre les deux systèmes. Pour plus de détails sur ce processus important, consultez [Personnalisation des mappages d’attributs d’approvisionnement d’utilisateurs pour les applications SaaS dans Azure Active Directory](customize-application-attributes.md).
- **Mappage d’attributs pour les groupes :** Approvisionnement du nom du groupe et des détails du groupe, en plus des membres, si la prise en charge est effective pour certaines applications. Vous pouvez activer ou désactiver cette fonctionnalité en activant ou désactivant le **mappage** pour les objets de groupe affichés dans l’onglet **Approvisionnement**. Si les groupes d’approvisionnement sont activés, veillez à passer en revue les mappages d’attributs afin de vous assurer qu’un champ approprié est utilisé pour l’« ID correspondant ». L’ID correspondant peut être le nom d’affichage ou l’alias de messagerie. Le groupe et ses membres ne sont pas approvisionnés si la propriété correspondante est vide ou n’est pas renseignée pour un groupe dans Azure AD.
## <a name="provisioning-users-assigned-to-the-default-access-role"></a>Approvisionnement des utilisateurs assignés au rôle d’accès par défaut
Le rôle par défaut sur une application de la galerie est appelé le rôle « accès par défaut ». Historiquement, les utilisateurs assignés à ce rôle ne sont pas approvisionnés et sont marqués comme étant ignorés dans les [journaux d’approvisionnement](../reports-monitoring/concept-provisioning-logs.md) parce qu’ils ont le statut « non autorisés de manière effective ». 

**Comportement pour l’approvisionnement des configurations créées après le 16/04/2020 :** Les utilisateurs assignés au rôle d’accès par défaut sont évalués de la même façon que tous les autres rôles. Un utilisateur auquel est attribué l’accès par défaut n’est pas ignoré pour le motif « non autorisé de manière effective ». 

**Comportement pour l’approvisionnement des configurations créées avant le 16/04/2020 :** Au cours des trois prochains mois, le comportement se poursuivra comme aujourd’hui. Les utilisateurs ayant le rôle d’accès par défaut seront ignorés, car non autorisés de manière effective. Après juillet 2020, le comportement sera uniforme pour toutes les applications. Nous n’allons pas ignorer l’approvisionnement des utilisateurs ayant le rôle d’accès par défaut, pour le motif « non autorisés de manière effective ». Cette modification sera effectuée par Microsoft, sans aucune intervention du client. Si vous souhaitez vous assurer que ces utilisateurs continuent d’être ignorés, même après cette modification, appliquez les filtres d’étendue appropriés ou supprimez l’attribution de l’utilisateur de l’application pour qu’ils ne sont pas inclus.  

Pour toute question sur ces modifications, contactez provisioningfeedback@microsoft.com.
## <a name="next-steps"></a>Étapes suivantes

[Synchronisation Azure AD Connect : présentation du provisionnement déclaratif](../hybrid/concept-azure-ad-connect-sync-declarative-provisioning.md)
